NAME
====

bfu - brainfuck command line debugger

SYNOPSIS
========

bfu \[OPTIONS\] \[FILES\] \...

Options:

      -n          use numbers in i/o
      -i          be interpreter
      -s FLOAT    show a slide show

      -p          preprocess like awib bfpp
      -d          preprocess program and data for dbfi

      -P          preprocess only, don't run
      -F          print formatted program

      -h          show help
      -v          show version
      -m          show man page

With no FILES, reads program from STDIN. Input is reading after \'!\'
character in program, and then from STDIN.

COMMANDS
========

Syntax
------

### EXPR

EXPR is simple c-style expression with numbers and symbols.

\`@\' and \`\#\' characters have special meanings: \`\@EXPR\' expands to
value of cell number EXPR (or current cell, if EXPR is omitted), \`\#\'
expands to number of current cell.

If there is \`\$\' before \`@\' or \`\#\' characters, then its value
will not be expanded immediately, but every time when bfu will need its
value (for example, after every command in breakpoint condition).

### COND

COND is condition to check in breakpoints. It can be used for checking
command or memory pointer. Possible forms are:

\>EXPR, \<EXPR, =EXPR, etc \-- means that value we are checking should
be greater, lesser, etc then EXPR. =EXPR is equal to EXPR;

?\[EXPR\] \-- if EXPR isn\'t omitted, true if EXPR is true, with no EXPR
true if cell with number of checking value is non-zero;

\[EXPR1 ; EXPR2\] \-- means value should be between EXPR1 and EXPR2.

### CMND

CMND is one of eight brainfuck commands or \`\*\' which means any
command that modifies memory.

Commands
--------

### , \[EXPR1\] EXPR2

Assign value of EXPR2 to cell number EXPR1 or current cell if it is
omitted.

### . EXPR

Print value of EXPR.

### \^ \[EXPR\]

Jump to line number EXPR (i.e. continue execution until line number will
be equal to EXPR). Jump to the next line if EXPR is omitted.

### @\[!\]\[CMND\] \[COND\]

Continue until cell number will satisfy COND. If CMND isn\'t omitted,
command also should satisfy CMND (or should \_not\_ satisfy if \`!\' is
not omitted). If CMND is not omitted, COND can be omitted to jump to any
cell when command will satisfy CMND.

### } and {

Are the same as \`@\' command, but conformably to instruction pointer
instead of data pointer. \`}\' jumps forward and \`{\' jumps backwards,
so you can easily undo and redo any commands. If all operands are
omitted, run only one command.

### }} and {{

Change direction to forward or backward. It affects on action of **empty
command**.

### \] \[EXPR\]

Continue until exit from current cycle if EXPR is omitted or nearest
parent cycle from which current is of depth EXPR if not.

### \> \[EXPR\] and \< \[EXPR\]

Shift visible frame of memory array right or left to EXPR positions (or
a few positions if EXPR is omitted). Use \`\<\<\' to return to the
begining.

### !

Quit debugger.

### empty command

Just press enter to run next command (or previous if \`{{\' was called).

EXAMPLES
========

### From debugger

\$ **,3**

:   \$mem \[\$cur\] = 3

\$ **,(\#+8)@**

:   \$mem \[\$cur + 8\] = \$mem \[\$cur\]

\$ **@ \[\#+1;\#+5\]**

:   continue until data pointer will between (\$cur + 1) and (\$cur + 5)

\$ **@!\*? \$@ \> 3**

:   continue until current command will not modify memory and \'\$mem
    \[\$cur\] \> 3\' will be true

\$ **{\< \<10**

:   change direction to reversed and continue undoing commands until
    current command will be \`\<\' and its number in the line will be
    lesser then 10

### From shell

     % echo ',++.' > example.b
     % bfu -n -i example.b
     123
     125

     % bfu
     + bfu 0.2
     + brainfuck debugger
     +++++[->+++++[->+++++<]<]>>----[->+>+<<]>-----------------.---.>.
     !
     [0] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
     {1:0} +++++[->+++++[->+++++<]<]>>----[->+>+<<]>-----------------.---.>.
     > hey
     $ }65

PREPROCESSING
=============

With **-p** option, program will be preprocessed like bfpp from awib
project does it. It is useful for big programs and also it is necessary
for correct line numbers while debugging.

With **-d** option, bfu will format program for dbfi. It is useful if
you want to use dbfi in pipes, like this:

            echo 'my data' | bfu -d program.b | dbfi

It is equal to

            echo "`cat program.b`\!my data" | dbfi

except \`bfu -d\` will also preprocess program and concatenate data
after \`!\' in program.b with STDIN (if any).

AUTHOR
======

Victor Gaydov \<victor\@enise.org\>, 2009.
