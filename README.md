# cxmon

A command-line file manipulation tool and disassembler

Platform | CI Status
---------|:----------------
AmigaOS  | [� unclear �](https://github.com/emaculation/cxmon/issues/3)
BeOS     | [� unclear �](https://github.com/emaculation/cxmon/issues/3)
FreeBSD  | [� unclear �](https://github.com/emaculation/cxmon/issues/3)
Linux    | [![Linux Build Status](http://badges.herokuapp.com/travis/emaculation/cxmon?env=BADGE=linux&label=build&branch=master)](https://travis-ci.org/emaculation/cxmon)
OSX      | [![OSX Build Status](http://badges.herokuapp.com/travis/emaculation/cxmon?env=BADGE=osx&label=build&branch=master)](https://travis-ci.org/emaculation/cxmon)
Windows  | [Pending](https://github.com/emaculation/cxmon/issues/6)


Copyright (C) 1997-2007 Christian Bauer, Marc Hellwig

GNU binutils disassemblers Copyright (C) 1988, 89, 91, 93, 94, 95, 96, 97, 1998 Free Software Foundation, Inc.

## License

cxmon is available under the terms of the GNU General Public License. See the file "COPYING" that is included in the distribution for details.

## Overview

cxmon is an interactive command-driven file manipulation tool that is inspired by the "Amiga Monitor" by Timo Rossi. It has commands and features similar to a machine code monitor/debugger, but it lacks any functions for running/tracing code. There are, however, built-in PowerPC, 680x0, 80x86, x86-64, 6502 and Z80 disassemblers and special support for disassembling MacOS code. By default, cxmon operates on a fixed-size (but adjustable) memory buffer with addresses starting at 0.

## Installation

Please consult the file "INSTALL" for installation instructions.

## Usage

cxmon can be started from the Shell or from the Tracker (BeOS), but command line history doesn't work when started from the Tracker.

Options:
* `-m`  enables symbolic MacOS A-Trap and low memory globals display in the 680x0 disassembler
* `-r`  makes cxmon operate in real (virtual) memory space instead of an allocated buffer

If no additional command line arguments are given, cxmon enters interactive mode. Otherwise, all remaining arguments are interpreted and executed as cxmon commands.

The default buffer size is 1MB.

The cxmon command prompt looks like this:

```
  [00000000]->
```

The number in brackets is the value of `.` (the "current address", see the section on expressions). You can get a short command overview by entering `h`.

Commands that create a longer output can be interrupted with Ctrl-C.

To quit cxmon, enter the command `x`.

## Constants, variables and expressions

The default number base is hexadecimal. Decimal numbers must be prefixed with `\_`. Hexadecimal numbers may also be prefixed with `$` for clarity. Numbers can also be entered as ASCII characters enclosed in single quotes (e.g. `'BAPP'` is the same as `$42415050`). All numbers are 32-bit values (one word).

With the "set" command, variables can be defined that hold 32-bit integer values. A variable is referred to be its name. Variable names may be arbitrary combinations of digits and letters (they may also start with a digit) that are not also valid hexadecimal numbers. Names are case-sensitive.

cxmon accepts expressions in all places where you have to specify a number. The following operators are available and have the same meaning and precedence as in the C programming language:

Operator| Meaning
--------|:----------------
`~`     | complement
`+`     | unary plus
`-`     | unary minus
`*`     | multiplication
`/`     | integer division
`%`     | modulo
`+`     | addition
`-`     | subtraction
`<<`    | shift left
`>>`    | shift right
`&`     | bitwise AND
`^`     | bitwise exclusive OR
`|`     | bitwise inclusive OR


Parentheses may be used to change the evaluation order of sub-expressions.

There are two special symbols that can be used in expressions:

* `.`   represents the "current address" (the value of `.` is also displayed in the command prompt). What exactly the current address is, depends on the command last executed. The display commands set `.` to the address after the last address displayed, the "hunt" commands sets `.` to the address of the first found occurrence of the search string, etc.
* `:`   is used by the "apply" (`y`) command and holds the value of the byte/half-word/word at the current address.

The "modify" (`:`), "fill" (`f`) and "hunt" (`h`) commands require you to specify a byte string. Byte strings consist of an arbitrary number of byte values and ASCII strings separated by commas. Examples:

```
  "string"
  12,34,56,78,9a,bc,de,f0
  "this",0a,"is a string",0a,"with","newlines",\_10
```

## The buffer

Those cxmon commands that operate on "memory" operate on a buffer allocated by cxmon whose size is adjustable with the `@` command. The default buffer size is 1MB. The buffer is an array of bytes where each byte has a 32-bit integer address. Addresses start at 0 and are taken modulo the buffer size (i.e. for the default 1MB buffer, addresses `0` and `100000` refer to the same byte).

The buffer is the working area of cxmon where you load files into, manipulate them, and write files back from. Arbitrary portions of the buffer may be used as scratch space.

## Commands

The following commands are available in cxmon (`[]` marks a parameter that can be left out):

### Basic Commands

Cmd | Name | Description
--------|:-----|:-----------
`x`     | Quit cxmon | quits cxmon and returns to the shell.
`h`     | Show help text | displays a short overview of commands.
`??`    | Show list of commands |displays a short list of available commands.
`ver`   | Show version | shows the version number of cxmon.
`? expression` | Calculate `expression` | displays the value of the given expression in hex, decimal, and ASCII characters. If the value is negative, it is displayed as a signed and unsigned number.
`@ [size]` | Reallocate buffer | changes the size of the buffer to `size` bytes while preserving the contents of the buffer. If `size` is omitted, the current buffer size is displayed.
`: start string` | Modify memory | puts the specified byte string `string` at the address `start` into the buffer. The value of "." is set to the address after the last address modified.
`f start end string` | Fill memory | fill the buffer in the range from `start` to (and including) `end` with the given byte string `string`.
<code>y[b&#124;h&#124;w] start end expr</code> | Apply expression to memory | works like the "fill" (`f`) command, but it doesn't fill with a byte string but with the value of an expression that is re-evaluated for each buffer location to be filled. The command comes in three flavors: `y`/`yb` works on bytes (8-bit), `yh` on half-words (16-bit) and `yw` on words (32-bit). The value of `.` is the current address to be modified,  the value of `:` holds the contents of this address before modification.  See examples below.
`t start end dest` | Transfer memory | transfers the buffer contents from `start` to (and including) `end` to `dest`. Source and destination may overlap.
`c start end dest` | Compare memory | compares the buffer contents in the range from `start` to (and including) `end` with the contents at `dest`. The addresses of all different bytes and the total number of differences (decimal) are printed.
`h start end string` | Search for byte string | searches for the given byte string `string` in the buffer starting at `start` up to (and including) `end`. The addresses and the total number of occurrences are displayed. The value of `.` is set to the address of the first occurrence.
`\ "command"`| Execute shell command | executes the given shell command which must be enclosed in quotes.
`ls [args]` | List directory contents | works as the shell command `ls`.
`rm [args]` | Remove file(s) | works as the shell command `rm`.
`cp [args]` | Copy file(s) | works as the shell command `cp`.
`mv [args]` | Move file(s) | works as the shell command `mv`.
`cd directory` | Change current directory | works as the shell command `cd`. The name of the `directory` doesn't have to be enclosed in quotes.
`o ["file"]` | Redirect output | When a file name is specified by `file`, all following output is redirected to this file. The file name **must** be enclosed in quotation marks even if it contains no spaces. Entering `o` without parameters closes the file and directs the output into the terminal window again.
`[ start "file"` | Load data from file | loads the contents of the specified `file` into the buffer starting from address `start`. The file name **must** be enclosed in quotation marks even if it contains no spaces. The value of `.` is set to the address after the last address affected by the l | writes `size` number of bytes of the buffer from `start` to the specified file. The `file` name **must** be enclosed in quotation marks even if it contains no spaces.
`set [var[=value]]` | Set/clear/show variables | If no arguments are given, all currently defined variables are displayed. Otherwise, the value of `var` is set to the specified `value`. If `=value` is omitted, the variable `var` is cleared.
`cv` | Clear all variables | clears all currently defined variables.


### Memory Display Commands

Each of these commands takes the arguments `[start [end]]` representing the `start` and `end` indexes of the buffer contents.  Omitting the arguments is equivalent to specifying `.` as an argument (e.g. `i` without arguments is equivalent to `i .`).  The value of `.` is set to the address after the last address displayed.

Cmd | Name | Description
--------|:-----|:-----------
`i [start [end]]`     | ASCII memory dump | displays the buffer as ASCII characters.
`b [start [end]]`     | Binary memory dump | displays the buffer in a binary format.
`m [start [end]]`     | Hex/ASCII memory dump | displays the buffer as hex words and ASCII characters
`d [start [end]]`     | Disassemble PowerPC code | disassembles the buffer
`d65 [start [end]]`   | Disassemble 6502 code | disassembles the buffer
`d68 [start [end]]`   | Disassemble 680x0 code | disassembles the buffer
`d80 [start [end]]`   | Disassemble Z80 code | disassembles the buffer
`d86 [start [end]]`   | Disassemble 80x86 (32-bit) code | disassembles the buffer
`d8086 [start [end]]` | Disassemble 80x86 (16-bit) code | disassembles the buffer
`d8664 [start [end]]` | Disassemble x86-64 code | disassembles the buffer

## Examples

Here are some simple examples for what is possible with cxmon.


Command            | Effect
:------------------|:--------------
`yw 0 fff :<<8`    | shifts all words in the address range `0..fff` to the left by 8 bits (you can use this to convert bitmap data from ARGB to RGBA format, for example)
`y 0 1234 ~:`      | inverts all bytes in the address range `0..1234`
`yh 2 ff 20000/.`  | creates a table of the fractional parts of the reciprocals of `1..7f`
`[ 0 "file1"`<br />`[ . "file2"`<br />`] 0 . "file3"` | Join `file1` and `file2` to `file3`
`[ 0 "file"`<br />`] 18 .-18 "file"`| Remove the first 24 bytes (e.g. an unneeded header) of `file`
`[ 0 "cxmon"`<br />`h 0 . 60,00,00,00` | Load the `cxmon` executable and search for PowerPC `nop` commands
`[ 0 "cxmon"`<br />`set size=.`<br />`h 0 . "->"`<br />`: . " $"`<br />`] 0 size "cxmon1"` | Create a modified version of `cxmon` so that the prompt has ` $` instead of `->`
`[ 0 "file"`<br /><code>yh 0 .-1 :>>8&#124;:<<8</code><br />`] 0 . "file"` | Convert a binary `file` which contains 16-bit numbers in little-endian format to big-endian format (or vice-versa)
`[ 0 "bootnub.image"`<br />`d 100` | Load a BeBox boot ROM image and start disassembling the system reset handler


## Using cxmon in your own programs

cxmon provides a simple interface for integration in other programs. It can, for example, be used as a monitor/debugger for an emulator (it is used in Basilisk II in this way).

Here's how to do it (all functions are defined in the mon.h header file):

1. Link all the cxmon object files, except `main.o`, to your program.
1. In your program, call `mon_init()` before using any other cxmon functions.
1. After calling `mon_init()`, set the `mon_read_byte` and `mon_write_byte` function pointers to the routines used for accessing memory.
1. You can use `mon_add_command()` to add new commands to cxmon by specifying the command name, function and help text. From within your command function, you can use `mon_get_token()` and `mon_expression()` to parse the arguments and the `mon_read/write\_\*()` functions to access memory.
1. To enter cxmon, call the `mon()` function like this:

    ```c++
      const char \*args[3] = {"mon", "-r", NULL};
      mon(2, args);
    ```

1. If you're done with cxmon, call `mon_exit()`.

## History

Please consult the file "ChangeLog" for the release history.

Christian Bauer
www.cebix.net

Marc Hellwig
<Marc.Hellwig@uni-mainz.de>
