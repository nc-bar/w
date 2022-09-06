
    $ ed filename   # open the file 'filename'
    

the contents of the file are copied to a buffer; if no file is provided, the
buffer is empty. changes are always applied to the buffer. if changes are not
saved, they are lost when closing the editor.

there are several versions of ed, with slightly different arguments and
commands; the `-G` argument runs gnu's `ed` in compatibility mode.

interacting with `ed` is similar to interacting with a shell. a prompt is
displayed while it reads lines from standar input. if the input doesn't match
with a command, an error is returned.

the default prompt is an empty string, but it can be changed using the `-p`
option:

    
    
    $ ed -p-  # use '-' as the prompt
    

### addresses

the user enters commands in the standard input. most commands consist of only
one line and are of the form:

    
    
    [address[,address]]command[parameters]
    

the commands are applied to the lines given by the addresses. an address can
be:

  * a positive number
    
        1p  prints the first line of the buffer
    0p  error: 0 can't be printed
    1,4p    print lines 1 through 4
    2   set current 2 as the current line
    

  * a negative number counts backwards from the last line (so `-1` is `$-1`, that is, the previous line to the last; `-0` is `$`)
    
        # '-' is not the prompt, there is no prompt
    -0p print the last line
    -1p print the second-to-last
    

  * `.` represents the current address; it is tracked by ed
    
        .p  print the current line
    .   print the current line
    .,3p    print the lines between the current address and the third line
    d   delete the line in the current address
    

  * `0` is a virtual line above all others. it's not posible to read or write into this address. it is useful, for example, to write a line before the first one (that is, bellow `0`)
    
        0p  error: 0 can't be printed
    0a  add text at the beggining of the buffer (the previous first line will be the second line.
    

  * `$` is the last line in the buffer
    
        1,$p    print the entire file
    1,$d    erase the content in the buffer
    

two addresses separatad by a comma represent the range of lines between them,
with both ends included:

    
    
        1,2 first and second lines
        1,$ all lines in the buffer
    

the second address has to be bigger than the first:

    
    
        3,1 error: 3 > 1
        1,1 first line
        .,3 lines between the current address and the third line.
            if '.' is bigger than 3, then the address is invalid
    

  * the symbols `+n` and `-n` refer to the nth line after and prior to the current address respectivly.
    
        .n  # print the current line and its number
    3   # current address is 3
    +2  # advance the current address two places
    5   # now current address is 5
    -2  # go back two places
    3   # now current address is 3
    -   # equivalent to -1
    2   # now current address is 2
    +   # equivalent to +1
    3   # now current address is 3
    

. `/re/` search the line for a match to the regular expression `re`. the
search starts with the inmediate address after `.`, and wraps to the first
line if it reaches the end of the buffer. if a match is found, the current
address is set to the matched line.

    
    
        a       # adding some lines to show /re/ working
        uno
        dos
        tres
        cuatro      # the name of the line equals its address
        .
        2
        dos     # '.' is set to 2
        /atr/       # search for a line with the string 'atr'
        cuatro      # the 4th line contains 'atr'
        .n      # print current address along with the line address
        4   cuatro  # '.' was set to 4, the matching line
    
        /o/     # now search using /o/
                # the search reaches the end and wraps
                # to the first line
        uno     # the line 'uno' matches /o/
        .n      # print current address
        1   uno # current address is set to 1
    
    `/re/` matches the first line it finds and stops so, in the example above, the search stoped at line 1 even though line 2 (dos) 
    

also matches the pattern.

  * `?re?` the same as `/re/` but the search is backwards, wrapping to the last line when the fist line is reached. it also stops at the first match
    
        a       # adding some lines to show ?re? working
    uno
    dos
    tres
    cuatro      # the name of the line equals its address
    .
    2
    dos         # '.' is set to 2
    ?atr?       # search for a line with the string 'atr'
    cuatro      # the 4th line contains 'atr'
    .n          # print current address along with the line address
    4   cuatro  # '.' was set to 4, the matching line
    ?o?     # search backwards for the pattern 'o'
    dos     # second line matches the pattern
    .n
    2   dos # now the matched line is the 2nd,
            # because the search started at the 4th line
            # and proceeded backwards
    

  * it's possible to mark lines in ed, so they can be referenced in future commands. the marked lines are also valid addresses, `'lc` refers to the line marked with `lc`, where `lc` is a lowercase letter.
    
        a       # adding some lines to show ?re? working
    uno
    dos
    tres
    cuatro      # the name of the line equals its address
    .
    2
    dos     # '.' is set to 2
    ka      # mark line 2 using a as a key
    'an     # print line and address of the line marked as 'a'
    2   dos # the line with key a is the second
            # one, so '.' is set to 2
    'ad     # delete the line marked with a
    ,p      # print file to show the line was deleted
    uno
    dos
    tres
    

### commands

commands cause ed to perform an action upon one or more lines. if the user
enters (at the prompt) an address followed by a letter matching a command,
then the commands is executed on the addressed lines. most commands have a
default line to use if no address is provided.

    
    
        $ cat file
        first line
        second line
        $ ed -G -p- file
        39
        -,p     # ',' is the entire file, and 'p' is the print command
                # it prints the lines especified by the address
        first line
        second line
        -,n     # print the lines and the line numbers
        1   first line
        2   second line
    

in the list of commands below default lines are between `()`; for example
`(.)p` means that the command `p` applies to the current line if no address is
specified. also, the prompt is set to `-`, when there is one, and the shell
prompt is `$`.

  * `(.)a` adds lines entered by the user (after the command is entered) after the line addressed. the lines are read from the standard input. to go back to the prompt write a line containing only `.`. the current address is set to the last line added.
    
        -,p
    line 1
    line 2
    line 4
    -.p
    line 4
    -a      # add lines belw 3
            # a accepts lines from stdin
            # and adds them after the line in the address
            # a '.' in a line by itself comes back to the prompt
    line 5
    line 6
    -.n     # print current address
            # the current address is set to the last line entered
    5   line 6
    -,p     # print entire file
    line 1
    line 2
    line 4
    line 5
    line 6
    -2a     # add lines after '2'
    line 2
    'a' accepts more than one line
    .
    -,n
    1   line 1
    2   line 2
    3   line 3
    4   'a' accepts more than one line
    5   line 4
    6   line 5
    

  * `(.)d` deletes the addressed lines from the buffer. the current address is set to the line bellow the last deleted line, if one doesn't exist, it is set to the las line above the deleted lines.
    
        -,p     # print all lines in the file
    first line
    second line
    third line
    fourth line
    -1d     # delete first line
    -,p
    second line
    third line
    fourth line
    -3d     # delete line number 3
    -.p     # print current address
    third line  # the current address is one above the deleted range
            # because there is no line after it
    -,p
    second line
    third line
    

  * `(.)i` reads lines from stdin and adds them above the address. the current line is set to the last one added. entering a line with `.` by itself takes ed back to the prompt. it behaves similarly to the `a` command, but inserts the lines above the address.

  * `(.,+)j` replaces the addressed lines by the line resulting of their concatenation. the current address is set to the resulting line.
    
        -,p
    uno
    dos
    tres
    -2
    dos
    -j  # join the default lines:
    -p  # the current address and the next (.,+)
    uno
    dostres # lines 2 and 3 were replaced by the line
        # resulting of the join of line 2 and 3
    

  * `(.)klc` sets a mark identified by the lowercase character `lc`. marked lines can be refered to by `lc` in, for example, further commands.

  * `(.)i` prints the addressed lines raw
    
        -,p
    año
    -,l
    a\303\261o
    

  * `(.,.)m(.)` moves the addressed lines above the right hand side address.

  * `(.,.)n` prints the lines in the address with the line numbers. the current line is set to the last line printed.

  * `(.,.)p` prints the lines in the address. the current line is set to the last line printed.

  * `(.,.)s/re/replacement/x` searches in the addressed lines for the `re` pattern one at a time and replaces it with `replacement`. if `x` is `g` then every match is replaced; if `x` is a positive integer `n`, then only the `n`th match is replaced; if `x` is not specified, only the first match is changed. finding no matches is an error. the current line is the last one affected.

an unscaped `&` in `replacement` is changed to the current match. if
`replacement` is `%` the one from the last substitution is used. for
`replacement` to contain a newline, it must be scaped with `\`.

    
    
        -,n
        1   bla bla bla match match match bla
        2   bla bla bla match match match bla
        3   bla bla bla match match match bla
        4   bla bla bla match match match bla
        5   bla bla bla match match match bla
        -1,2s/match/changed/
        -,n
        1   bla bla bla changed match match bla
        2   bla bla bla changed match match bla
        3   bla bla bla match match match bla
        4   bla bla bla match match match bla
        5   bla bla bla match match match bla
        -3,4s/match/changed/g
        -,n
        1       bla bla bla changed match match bla
        2       bla bla bla changed match match bla
        3       bla bla bla changed changed changed bla
        4       bla bla bla changed changed changed bla
        5       bla bla bla match match match bla
        -5s/match/changed/3
        -,n
        1       bla bla bla changed match match bla
        2       bla bla bla changed match match bla
        3       bla bla bla changed changed changed bla
        4       bla bla bla changed changed changed bla
        5       bla bla bla match match match bla
    

  * `(.,.)s` repeats the last substitution in the addressed lines. the command may be modified by the sufixes `n`, `r`, `g` and `p`. the suffix `n` makes the substitution happen only on the `n`th match. the other three can be used in any combination: `r` uses the regular expression of the last seearch instead of the last substitution; `g` toogles the global mode in the last substitution; and `p` toogles the print mode of the last substitution. the current address is set to the last one affected
    
        -,n
    1   bla bla bla match match match bla
    2   bla bla bla match match match bla
    3   bla bla bla match match match bla
    4   bla bla bla match match match bla
    5   bla bla bla match match match bla
    -2,4s/match/changed/3
    -,n
    1   bla bla bla match match match bla
    2   bla bla bla match match changed bla
    3   bla bla bla match match changed bla
    4   bla bla bla match match changed bla
    5   bla bla bla match match match bla
    -1s     # repeat last substitution on line 1
    -,n
    1   bla bla bla match match changed bla
    2   bla bla bla match match changed bla
    3   bla bla bla match match changed bla
    4   bla bla bla match match changed bla
    5   bla bla bla match match match bla
    -5sg        # on line 5 repeat the last substitution but
            # toogle the global flag: last one didn't have
            # the global flag so this time it will
    -,n
    1   bla bla bla match match changed bla
    2   bla bla bla match match changed bla
    3   bla bla bla match match changed bla
    4   bla bla bla match match changed bla
    5   bla bla bla changed changed changed bla
    

  * `(.,.)t(.)` copies the addressed lines to after the right hand side address. the current address is set to the last line copied.
    
        -,n
    1   line 1
    2   line 2
    3   line 3
    4   line 4
    -2,3t0
    -,n
    1   line 2
    2   line 3
    3   line 1
    4   line 2
    5   line 3
    6   line 4
    

  * `u` undoes the last command and sets the current line to the one previous to the last command. the commands g, G, v, V are treated as one command. `u` can undo itself.

  * `(1,$)g/re/command-list` it works as a for loop over all the lines matching the `re` regular expression. the current address is set to each matching line in turn and then the command list is executed. each command must be in a different line. `G`, `g`, `V` and `v` can't be on the command list. if the list is empty, `p` is executed. any change in line numbers (for example if a line is deleted) takes place before the next match is processed. if there were no matches, the current address doesn't change.
    
        ,n
    1   a uno
    2   b dos
    3   c tres
    4   d uno
    5   e dos
    6   f dos
    7   g tres
    g/dos/.n\
    d\
    .n
            # first match (b dos)
    2   b dos   # .n prints the current line
            # d deletes current line
    2   c tres  # .n prints current line (2)
            # --------------------------
            # second match (e dos)
    4   e dos   # .n: 'e dos' is now line 4
            # d deletes line 4: e dos
            # .n prints line 4: f dos
            # --------------------------
            # third match (f dos)
    4   f dos   # .n: 'f dos' is now line 4
            # d deletes line 4: f dos
    4   g tres  # .n prints line 4: g tres
            # no more matches
    

  * `(1,$)G/re/command-list` similar to the `g` command, but interactive. for each line matching the regular expression `re` the current address is set, the line is printed and the user can supply a list of commands, in the same format as with `g`. arter the last matching line was proccessed, the current line is set to the last one modified. `&` repeats the last non-empty command.
    
        ,p
    match 1
    pass
    pass
    match 2
    pass
    match 3
    pass
    G/match/
    match 1    # match found, printed line, set as current address
    d          # delete the line
    match 2    # match found
    a\         # add a line below the match
    newline    # \ used for multiline command list and arguments
    match 3
    .c\        # replace matching line
    changed
    ,n
    pass
    pass
    match 2
    newline
    pass
    changed
    pass
    

  * `(1,$)v/re/command-list` same as `g` but it applies the commands to lines not matching `re`.

  * `(1,$)V/re/command-list` same as `G` but it applies the commands to lines not matching `re`.

  * `(1,$)w file` writes addressed lines to `file`. if no file is specified the default is used; lines are writen to a file if one is specified, also it's set as the defaul. this command will erase the data in the file without warning. the current address is not changed.

  * `(1,$)wq file` saves the addressed lines into `file` and then issues a `q` command. if `q` triggers a warning, the current line is one after the addressed lines; if the last line in the buffer was among the lines copied, then current address is set to the last line in the buffer.
    
        $ ls
    $ ed -G -p- ~/some/existing/file
    -,p
    line 1
    line 2
    line 3
    .
    -w
    21
    -1,2w test
    14
    $ cat test
    line 1
    line 2
    

  * `(1,$)w !command` executes `command` and passes the addressed lines to the stdin of the command. the current address doesn't change.
    
        -,p
    how many words?
    -w !wc -w
    3       # the result of echo "how many words?" | wc -w
    16      # number of characters passed to the command
    -a
    now how many?
    what about now?
    .
    -,p
    how many words?
    now how many?
    what about now?
    -2,3w !wc -w
    6
    30
    

  * `(1,$)W file` copies the lines in the address to the end of `file`. the current address remains unchanged.
    
        $ ls
    $ ed -G -p-
    -a
    line1
    line2
    .
    -,W test
    12
    -q
    $ ls
    test
    $ cat test
    line1
    line2
    $ ed -G -p-
    -a
    W appends at the end of the file
    it does not overwrites it
    .
    -,W test
    59
    -q
    $ cat test
    line1
    line2
    W appends at the end of the file
    it does not overwrites it
    $ ed -G -p-
    -a
    it's also possible to append
    some lines, not just the entire file
    for example, the first and last lines of this buffer
    won't be appended
    .
    - 2,3W test
    137
    -q
    $ cat test
    line1
    line2
    W appends at the end of the file
    it does not overwrites it
    some lines, not just the entire file
    for example, the first and last line
    

  * `(+)zn` scrolls `n` lines at a time starting at the current address. if n is not specified the current window size is used. the current address is set to the last line printed. adding a `n` to the end prints the line numbers also, otherwise it just prints the lines.
    
        -z10    # prints 10 lines starting whith the current address
    -z10n   # prints the same but it adds the line numbers
    -10z20  # prints 20 lines from line number 10 (included)
    -10z    # prints an entire window of lines from 10
    

the command `z` is useful for scrolling: repeated `z` commands prints
succesive windows, effectivly navigating the buffer one screen at a time
(which is called scrolling).

  * `($)= i` ??.

  * `(+)` an address without a command prints the addressed line and sets the current address to that line. if no address is provided, the result is equivalent to a `+` address (the next line).
    
        -,n # print line and line number
    1   one
    2   two
    3   three
    -1n
    1   one
    -+
    two
    -+
    three
    -1n
    1   one
    -++ # cumulative effect
    three   # advanced two lines fordward
    1
    one
    -
    two # a line by itself is equivalent to +
    -
    three
    

  * `f file` sets the default file to `file`. if the file is not specified it will print the default filename.

  * `E file` is the same as `e file` but it won't print a warning if the buffer contains changes not saved to disk.

  * `e file` edits 'file', and sets the default filename to `file`. if `file` is not specified, the default file is used. the buffer prior to the command is erased and then the contents of `file` are copied to the buffer. the current address is set to the last line copied.

  * `H` toogles the automatic printing of an explanation of the last error. by default, the explanation is not printed.

  * `h` prints an explanation of the last error.

  * `P` toogles the command prompt on or off (off by default unless the `-p` option is used).

  * `q` quits ed. if there are changes not saved to disk, the first `q` prints a warning, the second `q` closes the file without saving.

  * `Q` quits ed, even if there are changes not saved to disk.

  * `($)r file` copies the contents of `file` after the addressed line. if a `file` is not specified, the default is used. if there is not a default file set, `file` is used and set as default. the default file is not changed if there was one before the command was issued. the current address `.` is set to the last line copied. it's similar to `r !command` but with the file contents instead of the output of a command.

  * `($)r !command` copies the output of `!command` to after the line addressed. the current address is set to the last line that was copied.
    
        $ ls
    file1
    file2
    $ ed -G -p-
    -r !ls
    14
    -,p
    file1
    file2
    -0r !ls
    14
    file1
    file2
    file1
    file2
    

  * `!command` executes `command` in the shell. if the first character of `command` is `!` then it is replaced by text of the previous `!command`.
    
        $ ls
    file1
    file2
    $ ed -G -p-
    -!ls
    file1
    file2
    !
    -!!
    ls
    file1
    file2
    !
    

### using ed

good properties:

  * the startup and editing is extremely fast.

  * adding, deleting and changing lines is very confortable and fast.

  * navigating the buffer is not that bad, but it isn't very confortable either; reading lines surrounding a specific line, for example the function containing a compiler error, requires thinking how many lines to read. most others editors make this easier.

  * scrolling is easy with the `z` command.

  * the command syntax is very compact and elegant.

  * no configuration, other than the prompt. it works out of the box, with a 

### references

  * ed man from openbsd

  * ed man from gnu ed

