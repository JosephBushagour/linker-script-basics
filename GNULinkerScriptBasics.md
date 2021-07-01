# GNU Linker Script Basics

> Note: This document assumes general familiarity with object files, executables, and the concept of a linker

The GNU linker (`ld`) is an incredibly powerful tool that you can generally avoid messing with. Sometimes you do need to intervene and manually describe the link, this document acts as a quick, but **not exhasutive**, reference for those situations.


## What is a Linker Script?
 - Linker scripts describe the memory layout and the mapping of input sections to an executable 
 - Every link is described by a linker script
   - If you don't explicitly write one, a default script will be used
 - Linker scripts are a series of commands written in "Linker Command Language"


## Linker Command Language Basics
 - Linker Command Langauge is compact, weird, and C-like
 - Comments are written like C block comments (`/* Comment Here */`)
 - Semicolons are ignored after commands but required after expressions

## Symbol Assignment
 - Syntax: `symbol = expression;`
   - After definition, you can use C-like assignment operators: `+=`, `-=`, `*=`, `/=`, `<<=`, `>>=`, `&=`, and `|=`
 - Expression syntax is just like C
   - This includes the ternary if (`expression ? if_nonzero : if_zero`) 
 - Constants are assumed to be 32 or 64 bits and can be prefixed with `0x` (hex) and `0` (octal)
 - ❗ User defined symbols are visible to the programs being linked ❗

## `SECTIONS` Command
 - The most important, and only required command in a linker script
 - `SECTIONS` describes the mapping of input sections to output sections in the executable
 - Syntax:
   ```C
    SECTIONS
    {
      /*
       * Populated with one or more of any of the following:
       * - Output section description
       * - Symbol assignment
       * - ENTRY command
       * - Overlay description [not covered]
       */
       ...
    }
   ```
 - The magic symbol `.` is defined inside of a `SECTIONS` command to mean the current output location counter
   - Assigning to this symbol moves where the next input section will be linked

### Output Section Description
 - Syntax:
   ```C
    /*
     * Output Section Description Parameters:
     * - section_name : required, the name of the output section (ex: .text, .data, ...)
     * - [address]    : optional, the VMA (Virtual Memory Address) of the section
     * - [(type)]     : optional, if "NOLOAD" the section won't be loaded into memory
     * - [AT(lma)]    : optional, the LMA (Load Memory Address) where the section will be loaded
     *     into memory. It's up to you to copy this data from the LMA to the VMA at runtime
     * - [>region]    : optional, assigns the section to a region of memory (MEMORY command)
     * - [:phdr ...]  : optional, assigns the section to a program segment (PHDRS command [not covered])
     * - [=fillexp]   : optional, sets fill pattern for the etnire section
     */
    section_name [address] [(type)] : [AT(lma)]
    {
      /*
       * Populated with one or more of any of the following:
       * - Input section description
       * - Literal values
       * - Symbol assignment
       * - Output section keywords [not covered]
       */
       ...
    } [>region] [:phdr ...] [=fillexp]
   ```
 - The section name `'/DISCARD/'` is magic and will discard any input sections in it 

#### Input Section Description
 - Directs the linker on which sections to include
 - Syntax: `filename(.section_name ...)`
   - Will include `.section_name` from `filename`
   - The filename and section name can contain unix-style patterns: `*`, `?`, `[char set]`
   - ex: `*(.text)` includes the `.text` section from all linked files
 - The linker will link sections to the first matching pattern and won't repeat any sections

#### Literal Values
 - The following functions can be used to include literal values in an output section:
    | Function           | Size (bytes) |
    |--------------------|--------------|
    |`BYTE(expression)`  | 1            |
    |`SHORT(expression)` | 2            |
    |`LONG(expression)`  | 4            |
    |`QUAD(expression)`  | 8            |

### `ENTRY` command
 - Sets the entry point of the code, can be overrided with the `-e` command line flag
 - Syntax: `ENTRY(symbol)`
 - If not used, will default to the magic symbol `start` if defined, otherwise the first byte of `.text`


## `MEMORY` Command
 - Optional, if not included the linker will assume all memory is usable
 - Syntax:
   ```C
    MEMORY 
    {
        /*
         * One or more memory descriptions (format following):
         * region_name : ORIGIN = literal_constant, LENGTH = literal_constant
         */
        ...
    }
   ```
 - Regions defined in the `MEMORY` command can be used in the `SECTIONS` command

## Builtin Functions
 - `ADDR(section)`: Returns the VMA of `section`
 - `ALIGN(expression)`: Returns the location counter (`.`) aligned to the next `expression` byte boundary
 - `DEFINED(symbol)`: Returns 1 if the symbol is defined
 - `LOADADDR(section)`: Returns the LMA of `section`
 - `MAX(expression_1, expression_2)`: Returns the max of `expression_1` and `expression_2`
 - `MIN(expression_1, expression_2)`: Returns the min of `expression_1` and `expression_2`
 - `SIZEOF(section)`: Returns the size of `section` in bytes

## Miscellaneous Commands
 - `INCLUDE filename`: Includes the linker script `filename` in this linker script
   - Can be nested up to 10 levels
 - `INPUT(filename, ....)`: Includes `filename` in the link
 - `OUTPUT(filename)`: Names the output file
 - `STARTUP(filename)`: Specifies `filename` as the first file linked
 - `ASSERT(expression, message)`: Asserts `expression != 0`
 - `PROVIDE(symbol = expression)`: Defines `symbol` only if it is referenced and not defined by the code being linked
