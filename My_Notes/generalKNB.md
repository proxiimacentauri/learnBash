# General Knowledge Base


<h2 id="top"></h2>


| Topics      |
| :---        |
| [BASH Debug Ability](#BDA)      |
| [BASH Static Analysis Tool](#BSAT)      |
| [BASH Strict Mode](#BSM)      |
| [BASH Terminal Redirection Commands](#BTRC)      |
| [BASH Arrays](#BA)      |
| [BASH Conditional Evaluators `[]` `[[]]`](#BCE)      |
| [BASH Stdout Formatting](#BStdF)      |
| [BASH Variable Expansion and Manipulation](#BVEAM)      |
| [BASH Multiline Comments](#BMC)      |
| [BASH Misc Tools and Tricks](#BMTT)      |
| [BASH Globbing](#BG)      |
| [BASH Kill Background Processess](#BKBP)      |
| [BASH Coloured Output](#BCO)      |
| [BASH Stream Editor Shortcuts (sed)](#BSES)      |
| [BASH Regular Expression Shortcuts](#BREGEX)      |
| [BASH Python Path for Packages](#BPPFP)      |
| [Ubuntu Network Namespaces](#UNN)      |
| [Ubuntu Terminal Cheatsheet](#UTC)      |
| [Ubuntu Package Check and Install](#UPCI)      |
| [Ubuntu Force a USB 3.0 port to work in USB 2.0 mode](#UUSB)      |
| [Ubuntu `Systemd` service](#UDSL)      |
| [Ubuntu Cron Jobs](#UCJ)      |
| [Ubuntu CPU Stats](#UCPUS)      |
| [Ubuntu Set Permanent Naming Scheme for Serial Devices](#USPNSSD)      |
| [Ubuntu Real Time Disk Usage (ncdu)](#URTDU)      |
| [VIM Cheatsheet](#VC)      |
| [TMUX Cheatsheet](#TC)      |
| [MacOS Open Multiples of the Same Application](#MOMSA)      |
| [MacOS Reset Trial Period of Applications](#MRTPA)      |
| [Windows Sub-System for Linux (WSL)](#WSL)      |
| [git (Global Information Tracker)](#GIT)      |
| [git (Work and Personal Profiles)](#GWPP)      |
| [VS Code Shortcuts](#VSS)      |
| [](#)      |
| [](#)      |

</br></br></br></br></br></br></br></br></br>


<h2 id="BDA"></h2>

## BASH Debug Ability
[Back to Top](#top)
- Add this code to be able to DEBUG a file just based on a `_DEBUG` flag.
- Set debug commands.
  ```bash
    #!/bin/bash
    _DEBUG="on"
    export PS4=' +\033[38;5;36m ${BASH_SOURCE}:\033[0m\e[1;35m #${LINENO}:\e[0m\e[31m ${FUNCNAME[0]:+${FUNCNAME[0]}():}\e[0m       '
    DEBUG()
    {
        if [[ "$_DEBUG" == "on" ]]
        then
            "$@"
        fi
    }

    DEBUG echo 'Debugging'
    DEBUG set -x
    a=2
    b=3
    c=$(( $a + $b ))
    DEBUG set +x
    echo "$a + $b = $c"
  ```
- Check function args.
  ```bash
    #!/bin/bash
    _DEBUG="on"
    print_DEBUG()
    {
        if [[ "$_DEBUG" == "on" ]]
        then
            printf "\n \033[1;38;5;124m [ DEBUG ] -->\033[0m \033[1;38;5;121m %s \033[0m \n" "$1"
        fi
    }

    my_func()
    {
        print_DEBUG "Function Name: ${FUNCNAME}()"
        print_DEBUG "         Args: $*"

        echo "I am doing something in this function"
    }

    my_func "arg1" "arg2" "arg3" "arg4" "arg5" "arg6"
  ```

</br>
</br>


<h2 id="BSAT"></h2>

## BASH Static Analysis Tool
[Back to Top](#top)
- Install a VS Code extension (ShellCheck)
- To ingore some of its warnings:
    ```
    # shellcheck disable=SC2034
    ```

- Which will help find faults in BASH scripts.

    >https://marketplace.visualstudio.com/items?itemName=timonwong.shellcheck


</br>
</br>


<h2 id="BSM"></h2>

## BASH Strict Mode
[Back to Top](#top)
- ### `set -e`
    - Instructs bash to immediately exit if any command [1] has a non-zero exit status.
    - When you use command substitution in Bash, the -e setting is not applied
        ```bash
        #!/bin/bash
        set -e
        echo "This command will succeed."
        ls /tmp
        echo "This command will fail."
        ls /nonexistent/directory
        echo "This command will never be executed."
        ```

</br>
</br>


- ### `set -E`
    - It tells the shell to treat errors as `traps` and to execute any error-handling code before exiting.
    - The `-e` option takes precedence over the `-E` option.
    - This means that if both options are used, the shell will immediately exit on errors instead of running the `cleanup()` function.

        ```bash
        #!/bin/bash
        set -E
        EXIT_IF_CMD_FAIL_WITH_TRACE()
        {
            local error_code=$?
            printf "\n \033[1;34m ~~~~~~~~!!!! ERROR DETECTED !!!!~~~~~~~~ \033[0m \n\n"
            printf "\n \033[1;38;5;75m >>>>> Error Trace <<<<< \033[0m \n\n"
            printf " \033[38;5;36m ${BASH_SOURCE}:\033[0m\e[1;35m #${BASH_LINENO}: \e[0m\e[31m ${FUNCNAME[1]}():\e[0m  ${BASH_COMMAND}       \n"
            printf "\n \033[1;38;5;75m Exit Status ---> %s \033[0m \n" " $error_code"
            exit 1
        }

        trap EXIT_IF_CMD_FAIL_WITH_TRACE ERR
        echo "This command will succeed."
        ls /tmp
        echo "This command will fail."
        ls /nonexistent/directory
        echo "This command will NOT succeed and trap's EXIT_IF_CMD_FAIL_WITH_TRACE() will be executed"
        ```


</br>
</br>

- ### `set -u`
    - It affects variables.
    - When set, a reference to any variable you haven't previously defined - with the exceptions of $* and $@ - is an error, and causes the program to immediately exit.
        ```bash
        #!/bin/bash
        firstName="Aaron"
        fullName="$firstname Maxwell"
        echo "$fullName"
        ```

</br>


- ### `set -o pipefail`
    - This setting prevents errors in a pipeline from being masked.
    - If any command in a pipeline fails, that return code will be used as the return code of the whole pipeline.
    - By default, the pipeline's return code is that of the last command - even if it succeeds.

        ```bash
        % grep some-string /non/existent/file | sort
        grep: /non/existent/file: No such file or directory
        % echo $?
        0
        ```
    - `grep` has an exit code of 2, writes an error message to stderr, and an empty string to stdout.
    - This empty string is then passed through sort, which happily accepts it as valid input, and returns a status code of 0.
        ```bash
        % set -o pipefail
        % grep some-string /non/existent/file | sort
        grep: /non/existent/file: No such file or directory
        % echo $?
        2
        ```

</br>


- ### `Setting IFS`
    - The IFS variable - which stands for Internal Field Separator - controls what Bash calls word splitting.
    - When set to a string, each character in the string is considered by Bash to separate words.
    - For the first loop, IFS is a space, meaning that words are separated by a `space` character.
    - For the second loop, "words" are separated by a `newline`, which means bash considers the whole value of "items" as a single word.
    - If **IFS is more than one character**, splitting will be done on any of those characters.
        ```bash
        #!/bin/bash
        names=(
            "Aaron Maxwell"
            "Wayne Gretzky"
            "David Beckham"
        )

        echo "With default IFS value..."
        for name in ${names[@]}; do
            echo "$name"
        done

        With default IFS value...
        Aaron
        Maxwell
        Wayne
        Gretzky
        David
        Beckham

        echo "With strict-mode IFS value..."
        IFS=$'\n\t'
        for name in ${names[@]}; do
            echo "$name"
        done

        With strict-mode IFS value...
        Aaron Maxwell
        Wayne Gretzky
        David Beckham
        ```
    - Setting IFS to `$'\n\t'` means that word splitting will happen only:
        - Newlines
        - Tab characters.
    - This very often produces useful splitting behavior.
    - By default, bash sets this to `$' \n\t'` - space, newline, tab.

</br>




- ### `shopt -s inherit_errexit`
    - When you use command substitution in Bash, the -e setting is not applied.
        ```bash
        echo "Command output: $(false; date)"
        ```
    - The command would successfully output the result of date, even though the failing false command should have exited the script first.
    - Enabling `inherit_errexit` allows the command substitution to inherit our `-e` setting, so date will not be run.
    - **Note** that the error status of the command substitution is still ignored.
    - Even though the parenthesized expression returned a nonzero exit status, echo will still run successfully.

</br>
</br>

- ### `shopt -s nullglob failglob`
    - If there's no match, the glob expression isn't replaced, it's just passed to the command as-is.

        ```bash
        ls ./builds/bin-*/

        # contains one or more directories whose names start with bin-

        ls ./builds/bin-windows/ ./builds/bin-linux/

        #if there's no match, the glob expression isn't replaced, it's just passed to the command as-is
        ```
    - `nullglob` - will replace the glob expression with the empty string
    - `failglob` - will raise an error

</br>
</br>

- ### TL;DR
    ```bash
    #!/bin/bash
    IFS=$'\n\t'
    set -eEuo pipefail
    shopt -s inherit_errexit nullglob failglob
    EXIT_IF_CMD_FAIL_WITH_TRACE()
    {
        local error_code=$?
        printf "\n \033[1;34m ~~~~~~~~!!!! ERROR DETECTED !!!!~~~~~~~~ \033[0m \n\n"
        printf "\n \033[1;38;5;75m >>>>> Error Trace <<<<< \033[0m \n\n"
        printf " \033[38;5;36m ${BASH_SOURCE}:\033[0m\e[1;35m #${BASH_LINENO}: \e[0m\e[31m ${FUNCNAME[1]}():\e[0m  ${BASH_COMMAND}       \n"
        printf "\n \033[1;38;5;75m Exit Status ---> %s \033[0m \n" " $error_code"
        exit 1
    }

    trap EXIT_IF_CMD_FAIL_WITH_TRACE ERR
    ```


</br></br>

- ### References
    > http://redsymbol.net/articles/unofficial-bash-strict-mode/

    > https://dev.to/banks/stop-ignoring-errors-in-bash-3co5

    > https://github.com/tests-always-included/wick/blob/master/doc/bash-strict-mode.md

</br></br>

<h2 id="BTRC"></h2>

## BASH Terminal Redirection Commands
[Back to Top](#top)

|          | Terminal|          | File             |  | existing |
| Syntax   | StdOut  |  StdErr  |  StdOut  |  StdErr  |   file    |
| :---     |
|     `>`    |   no     |   yes    |   yes    |    no    | overwrite  |
|    `>>`    |   no     |   yes    |   yes    |    no    |  append    |
|   `2>`     |   yes    |    no    |    no    |   yes    | overwrite  |
|   `2>>`    |   yes    |    no    |    no    |   yes    |  append    |
|          |          |          |          |          |            |
|   `&>`     |    no    |    no    |   yes    |   yes    | overwrite  |
|   `&>>`    |    no    |    no    |   yes    |   yes    |  append    |
|          |          |          |          |          |            |
|   `tee`    |   yes    |   yes    |   yes    |    no    | overwrite  |
|   `tee -a` |   yes    |   yes    |   yes    |    no    |  append    |
|          |          |          |          |          |            |
| `& tee`    |   yes    |   yes    |   yes    |   yes    | overwrite  |
| `& tee -a` |   yes    |   yes    |   yes    |   yes    |  append    |

</br>

- `cmd > output.txt`
    - StdOut ----> file
    - NOT visible in the terminal
    - If file exists ----> overwritten.

</br>

- `cmd >> output.txt`
    - StdOut ----> file
    - NOT visible in the terminal
    - If file exists ----> append

</br>

- `cmd 2> output.txt`
    - StdOut ----> file
    - NOT be visible in the terminal
    - If file exists ----> overwritten.

</br>

- `cmd 2>> output.txt`
    - StdErr ----> file
    - NOT be visible in the terminal
    - If file exists ----> append

</br>

- `cmd &> output.txt`
    - *StdOut and StdErr* ----> file
    - NOT be visible in the terminal
    - If file exists ----> overwritten.

</br>

- `cmd &>> output.txt`
    - *StdOut and StdErr* ----> file
    - NOT be visible in the terminal.
    - If file exists ----> append

</br>

- `cmd | tee output.txt`
    - StdOut `copied` to the file
    - Visible in the terminal
    - If file exists ----> overwritten.

</br>

- `cmd | tee -a output.txt`
    - StdOut `copied` to the file
    - Visible in the terminal
    - If file exists ----> append

</br>

- `cmd |& tee output.txt`
    - *StdOut and StdErr* `copied` to the file
    - Visible in the terminal
    - If file exists ----> overwritten.

</br>

- `cmd |& tee -a output.txt`
    - *StdOut and StdErr* `copied` to the file
    - Visible in the terminal.
    - If file exists ----> append

</br>


- ### References
    > stackoverflow.com

</br></br></br></br>


<h2 id="BA"></h2>

## BASH Arrays
[Back to Top](#top)
- Associative/ Dictionary Arrray (`Unordered`)
    - Bash doesnt provide an ordered associative Array
    - `-A` - Associative Array
    - `-g` - Global Array
    ```bash
    declare -gA num_words=( [1]="one" [2]="two" [3]="three" )

    for num in "${!num_words[@]}"
    do
        echo "$num --- ${num_words[$num]}"
    done
    ```

- Associative/ Dictionary Arrray (`Ordered`)
    ```bash
    Num_words=("1=one" "2=two" "3=three")

    for num in "${Num_words[@]}"
    do
        letter="${num%=*}"
        word="${num#*=}"
        echo "Key: $letter, Value: $word"
    done
    ```
- Regular 1D Array
    ```bash
    words=("one" "two" "three")

    for w in "${words[@]}"
    do
        echo "Word: $w"
    done
    ```


- ### References
    > Self

</br></br></br></br>

<h2 id="BCE"></h2>

## BASH Conditional Evaluators `[]` `[[]]`
[Back to Top](#top)
- `[` is a bash Builtin
- `[[` is a bash Keyword
    - It evaluates the expression and indicate the result of the evaluation by its exit status

- Main difference is that special parsing rules apply to them
- Ex:
    ```bash
    $ [ a < b ]
    -bash: b: No such file or directory
    $ [[ a < b ]]
    ```
- The below table represents the differences:

| Feature       | `[[ ]]`     |  `[]`      | Example                                               |
| :---     |
| string        | >           | \>         | `[[ a > b ]] ||` echo "a does not come after b"       |
| comparison    | <           | \<         | `[[ az < za ]] &&` echo "az comes before za"          |
|               | = or ==     | =          | `[[ a = a ]] &&` echo "a equals a"                    |
|               | !=          | !=         | `[[ a != b ]] &&` echo "a is not equal to b"          |
|               | -gt         | -gt        | `[[ 5 -gt 10 ]] ||` echo "5 is not bigger than 10"    |
| | | | |
| integer       | -lt         | -lt        | `[[ 8 -lt 9 ]] &&` echo "8 is less than 9"                                                                            |
| comparison    | -ge         | -ge        | `[[ 3 -ge 3 ]] &&` echo "3 is greater than or equal to 3"                                                             |
|               | -le         | -le        | `[[ 3 -le 8 ]] &&` echo "3 is less than or equal to 8"                                                                |
|               | -eq         | -eq        | `[[ 5 -eq 05 ]] &&` echo "5 equals 05"                                                                                |
|               | -ne         | -ne        | `[[ 6 -ne 20 ]] &&` echo "6 is not equal to 20"                                                                       |
| | | | |
| conditional   | &&          | -a         | `[[ -n $var && -f $var ]] &&` echo "$var is a file"                                                                   |
| evaluation    | `||`    | -o         | `[[ -b $var pipepipe -c $var ]] &&` echo "$var is a device"                                                           |
| | | | |
| expression grouping   | (...)       | \(...\)    | `[[ $var = img* && ($var = *.png pipepipe $var = *.jpg) ]] &&` echo "$var starts with img and ends with .jpg or .png" |
| | | | |
| Pattern matching      | = or ==     | N/A        | `[[ $name = a* ]] ||` echo "name does not start with an 'a': $name"                                             |
| | | | |
| RegularExpression matching     | =~          | N/A        | `[[ $(date) =~ ^Fri\ ...\ 13 ]] &&` echo "It's Friday the 13th!"                                                      |
| | | | |
| entry (file or directory) exists    | -e          | N/A        | `[[ -e $config ]] &&` echo "config file exists: $config"                                                              |
| file is newer/older than other file | -nt / -ot   | N/A        | `[[ $file0 -nt $file1 ]] &&` echo "$file0 is newer than $file1"                                                       |
| two files are the same              | -ef         | N/A        | `[[ $input -ef $output ]] &&` { echo "will not overwrite input file: $input"; exit 1; }                               |
| negation                            | !           | N/A        | `[[ ! -u $file ]] &&` echo "$file is not a setuid file"                                                               |

- `-a` and `-o` operators, and `( ... )` grouping, are defined by POSIX but only for strictly limited case and are marked as deprecated.
- Use of these operators is discouraged; you should use multiple `[` commands instead.


- ### References
    > http://mywiki.wooledge.org/BashFAQ/031



</br></br></br></br>

<h2 id="BStdF"></h2>

## BASH Stdout Formatting
[Back to Top](#top)
- In order to print multiple new lines:
    ```bash
    $ num_lines=10
    $ printf "\n %.0s" $(seq 1 $num_lines)
    ```

- ### References
    > stackoverflow.com

</br></br></br></br>

<h2 id="BVEAM"></h2>

## BASH Variable Expansion and Manipulation
[Back to Top](#top)

- BASH offers a way to parse the value of a variable in the below way:
- `#` = match from the beginning.
- `%` = match from the end.
    - One instance means **shortest** `#` or `%`
    - Two instances means **longest** `##` or `%%`
    ```bash
    ${my_var#pattern}    # delete shortest match of pattern from the beginning
    ${my_var##pattern}   # delete longest match of pattern from the beginning
    ${my_var%pattern}    # delete shortest match of pattern from the end
    ${my_var%%pattern}   # delete longest match of pattern from the end

    # Example
    $ my_var="users/akshay/domain.example"
    $ echo ${MYVAR##*/}
    domain.example

    $ echo ${MYVAR%/*}
    users/akshay

    $ echo ${MYVAR##*.}
    example

    $ NAME=${MYVAR##*/}  # remove part before last slash
    $ echo ${NAME%.*}    # from the new var remove the part after the last period
    domain
    ```
- BASH also offers string manipulation:
    ```bash
    ${my_var:3}           # Remove the first three chars (leaving 4..end)
    ${my_var:4:}          # Return the characters except the first 4
    ${my_var::3}          # Return the first three characters
    ${my_var::-3}         # Return the characters except the last 3
    ${my_var:3:5}         # The next five characters after removing the first 3 (chars 4-9)
    ```
- BASH also offers a way to change the case.
    ```bash
    ${my_var^^}           # Changes the case of the value in my_var to UPPER Case
    ${my_var,,}           # Changes the case of the value in my_var to LOWER Case

    ${my_var/search/replace}
    ```
- BASH also allows setting the value of the variables through below conditions:
    ```bash
    ${my_var:-bash}      # expands to the value of my_var if it is set and not null, and to default otherwise
    ${my_var:=bash}      # expands to the value of my_var if it is set and not null, and to default otherwise.
                            # It also sets the value of my_var to default if it is unset or null.
    ${my_var:+word}      # expands to word if my_var is set and not null, and to an empty string otherwise
    ${my_var:?error}     # expands to the value of my_var if it is set and not null, and causes an error and prints error otherwise.
    ${#my_var}           # expands to the length of my_var
    ```
- ### References
    > https://stackoverflow.com/questions/19482123/extract-part-of-a-string-using-bash-cut-split/19482947

    > https://www.baeldung.com/linux/string-contains-substring

</br></br></br></br>



<h2 id="BMC"></h2>

## BASH Multiline Comments
[Back to Top](#top)
- There are 2 ways of doing this:
    - Use a bash in built command `:`
        - Def - `: [arguments]` No effect; the command does nothing beyond expanding arguments
        and performing any specified redirections. A zero exit code is
        returned.
        ```bash
        #!/bin/bash
        echo "I want this to be printed before."
        : '
        echo "This should not be printed."
        echo "This should not be printed."
        echo "This should not be printed."
        '
        echo "I want this to be printed after."
        ```
    - Use `<<`
        ```bash
        #!/bin/bash
        echo "I want this to be printed before."
        <<akshay
        echo "This should not be printed."
        echo "This should not be printed."
        echo "This should not be printed."
        akshay
        echo "I want this to be printed after."
        ```

- ### References
    > stackoverflow.com

</br></br></br></br>

<h2 id="BMTT"></h2>

## BASH Misc Tools and Tricks
[Back to Top](#top)
- Shell Shortcuts
    ```bash
    !$      # copies the last arg of the previous command
    !!      # copies the whole line before it
    !!:p    # It will print out the output of !!
    !foo    # It will run the command which starts with foo
    !*      # Run the previous command except the first word

    $?      # tells you the exit status of the previous command
    $@      # stores all the arguments in a list of string     {$1, $2, $3 ...}
    $*      # stores all the arguments as a single string      $1 $2 $3 ...
    $#      # stores the number of arguments
    $-      # stores the current option flags set builtin command, or those set by the shell itself
    $$      # stores the process ID of the shell.
            # In a () subshell, it stores process ID of the invoking shell, not the subshell
    $!      # stores the process  ID of the job most recently placed into the background,
            # whether executed as an asynchronous command or using the bg builtin
    $0      # stores the name of the shell or shell script
    $_      # stores the most recent parameter (or the abs path of the command to start the current shell immediately after startup).
    ```
- Gives you CPU Stats (Like a Task Manager in Windows)
    ```bash
    $ htop
    ```
- File Explorer and Estimates the size of all folder dirs.
    ```bash
    $ ncdu
    ```
- File Explorer
    ```bash
    $ mc
    ```
- Empty the contents of the file.txt
    ```bash
    $ truncate -s 0 file.txt
    ```
- History will be timestamped (add it to bashrc)
    ```bash
    $ HISTTIMEFORMAT="%Y-%B-%d %T      "
    ```
- Format data in a table
    ```bash
    $ mount | column -t
    ```
- Find a function in an environment:
    ```bash
    $ declare -F
    $ typeset -F
    $ set | grep " ()"
    $ compgen -A function
    ```
- Digital Loggers Programmable Power Strip: `https://dlidirect.com/products/new-pro-switch`
- Calculated the Disk usage of all folders in current dir.
    ```bash
    $ du -sch .[!.]* * | sort -h
    ```
- Speed Test on Command Line
    ```bash
    $ curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python -
    ```
- Shows a list of open files by a process
    ```bash
    # `+L1'' will select open files that have been unlinked.
    $ sudo lsof +L1
    ```

- If you want a numbered output for cat
    ```bash
    $ cat -n filename
    ```
- Refer to command line n, where (n=number in history)
    ```bash
    # history
    #   1255 vim ak.txt
    #   1256 ls -la
    #   1257 cat ak.txt
    #   1258 mv ak.txt ak1.txt
    #   1259 rm -f ak1.txt
    # Now if you want to execute the 1257 command

    $ !-3
    ```

- Scrollable numbered line text
    ```bash
    $ less -N filepath
    ```
- Shows full path of the filename
    ```bash
    $ which -a filename
    ```
- Find out the location of installation
    ```bash
    $ type -a <cmd>
    ```
- Shows the source of the binary
    ```bash
    $ whereis binaryname
    ```
- Edit `sudoers` using visudo will validate the syntax:
    ```bash
    $ export EDITOR=vim
    $ visudo
    ```

- Timestamps for commands:
    ```bash
    $ ls -la | ts [%b" "%d,%Y-%l":"%M:%.S]"   "
    ```
- To make sure that the script is only `sourced` and not Executed
    ```bash
    #!/bin/bash
    [[ "${BASH_SOURCE[0]}" == "${0}" ]] && { echo -e "\n\n\n\n The scriptName.sh script must be sourced, not executed.\n\n\n\n"; exit 1; }

    ```
- Delete a folder x no. of days OLD.
    ```bash
    $ find /home/akshayd -name "FolderName" -type d -mtime +2 -exec rm -rdf {} \;
    ```
- Create a new user in linux with home dir and get full `sudo` permissions.
    ```bash
    $ Uname="akshaydandgaval"
    $ sudo useradd -m -s /bin/bash $Uname
    $ sudo passwd $Uname
    $ sudo usermod -aG sudo $Uname
    $ sudo usermod --shell /bin/bash $Uname
    $ sudo visudo

    # Add the below to sudoers file
    username ALL=(ALL) NOPASSWD:ALL
    ```
- Detach from a terminal.
    ```bash
    $ disown -a && exit = Disown the terminal and keep process going
    ```
- Obtain a timestamp and store it in a variable.
    ```bash
    $ ts=$(date +"%d%b_%H_%M_%S")
    ```


- **Redirect Stream to `/dev/null/`**

    ```bash
    $ popd &> /dev/null 2>&1
    ```
- **Pipestatus**
    - bash
        ```bash
        false | true
        echo "${PIPESTATUS[0]} ${PIPESTATUS[1]}"
        ```
    - zsh - Array begins at index :1
        ```bash
        false | true
        echo "$pipestatus[1]" "$pipestatus[2]"
        ```
    - Check all pipes, you can also use `set -e` for the code you want to check and then unset it.
        ```bash
        true |  false | true
        if [[ $(echo "${PIPESTATUS[@]}" | tr -s ' ' + | bc) -ne 0 ]]
        then
            echo FAIL
        fi
        ```

- **File Structure in Linux**
    ```
        /---\ /---\ /---\
        d r w x r w x r - x             r - Read = 4
        |                               w - Write = 2
        |                               x - execute = 1
        ^
        d - directory
        - - file
        l - symbolic Link
        b - block special file
        c - character special file
        p - named pipe special file
        s - local socket special file
    ```

- **Double quotes**
    - literal strings with whitespace `"StackOverflow rocks!" "Steve's Apple"`
    - variable expansions `"$var"` `"${arr[@]}"`
    - command substitutions `"$(ls)" `"`ls``"
    - globs where directory path or file name part includes spaces `"/my dir/"*`
    - to protect single quotes `"single'quote'delimited'string"`
    - Bash parameter expansion `"${filename##*/}"`

- **Single quotes**
    - command names and arguments that have whitespace in them
    - literal strings that need interpolation to be suppressed `'Really costs $$!'` `'just a backslash followed by a t: \t'`
    - to protect double quotes `'The "crux"'`
    - `regex` literals that need interpolation to be suppressed
    - use shell quoting for literals involving special characters `$'\n\t'`
    - use shell quoting where we need to protect several single and double quotes `$'{"table": "users", "where": "first_name"=\'Steve\'}'`

- **No quotes**
    - around standard numeric variables `$$, $?, $# etc.`
    - in arithmetic contexts like `((count++)) "${arr[idx]}" "${string:start:length}"`
    - inside `[[ ]]` expression which is free from word splitting and globbing issues (this is a matter of style and opinions can vary widely)
    - where we `want` word splitting for word in `$words`
    - where we `want` globbing for txtfile in `*.txt; do ...`
    - where we `want` `~` to be interpreted as `$HOME` ---> `~/"some dir" but not "~/some dir"`



- ### References
    > Self

</br></br></br></br>

<h2 id="BG"></h2>

## BASH Globbing
[Back to Top](#top)
- `?`  - Match single character. It can be used for multiple times for matching multiple characters.
- `*`  - Match zero or more characters.
- `[]` - match the character from the range
- `^`  - **Outside []** to search contents of the file that starts with a given range of characters.
- `^`  - **Inside []** to show all content of the file by highlighting the lines start with a given range of characters
- `!`  - It works same as the use of ‘^’ symbol outside the range pattern
- `$`  - To define the ending characters
- `{}` - Can be used to match filenames with more than one globbing patterns.
- `|`  - Its also used for applying more than one condition on globbing pattern.

    ```bash
    $ ls -l best.???


    $ ls -l a*.*


    $ echo "Do you want to confirm?"
    $ read answer
    $ case $answer in
    $ [Yy]* )  echo "confirmed.";;
    $ [Nn]* )  echo "Not confirmed.";;
    $ *) echo "Try again.";;
    $ esac

    # [:upper:] or [A-Z]
    # [:lower:] or [a-z]
    # [:digit:] or [0-9]

    $ ls -l [p-s]*
    $ ls -l [1-5]*

    # list.txt
    #     Apple
    #     4000
    #     Banana
    #     700
    #     Orange
    #     850
    #     Pear
    #     9000
    #     Jackdruit

    $ grep '^[P-R]' list.txt       ## Search Lines that starts with ‘P’ or Q or R
    Pear

    $ grep '[^P-R]' list.txt       ## Highlight lines that starts with ‘P’ or Q or R
    Apple
    4000
    Banana
    700
    Orange
    850
    Pear
    9000
    Jackdruit

    $ grep [!P-R] list.txt          ## Show Lines that starts with ‘P’ or Q or R
    Pear

    $ grep a$ list.txt
    Banana

    $ grep 50$ list.txt
    850

    $ ls -l {?????.sh,*st.txt}
    $ rm {*.doc,*.docx}

    $ ls a*+(.bash | .sh)

    $ echo "Select any option from the menu:"
    $ read answer
    $ case $answer in
    $ 1 | S )  echo "Searching text";;
    $ 2 | R )  echo "Replacing text";;
    $ 3 | D )  echo "Deleting text";;
    $ *) echo "Try again.";;
    $ esac
    ```

- ### References
    > stackoverflow.com

</br></br></br></br>

<h2 id="BKBP"></h2>

## BASH Kill Background Processes
[Back to Top](#top)

- If there is a process in the bacground like the below and there are multiple instances of it:
    ```bash
    $ while true
    do
        sleep 60s
        echo 3
        break
    done &
    [1] 4796
    [2] 4799
    ```

- There are 2 ways to kill it:
    - We can get it into `foreground` by typing ==> `fg`
        - once you type fg on the terminal
        - It will get the code in the bacground on the terminal and you can press Ctrl+C
        ```bash
        $ fg
        # Press Ctrl + C
        ```

    - Use the cmd `jobs` and using the number inside the [] issue cmd ===> `kill %1`
        ```bash
        $ jobs
        [1]-  Running                 while true; do
            sleep 60s; echo 1; break;
        done &
        [2]+  Running                 while true; do
            sleep 60s; echo 3; break;
        done &
        $ kill %1
        $ kill %2
        ```

- ### References
    > stackoverflow.com

</br></br></br></br>

<h2 id="BCO"></h2>

## BASH Coloured Output
[Back to Top](#top)
- Below is a way to display various colours available in BASH:
    ```bash
    RESET="\033[0m"
    for (( j=0; j<=200; j++ ))
    do
        CLR="\033[1;38;5;${j}m"
        printf "\n \n ${CLR} ============================================================= ${RESET}"
        printf "\n ${CLR} ================ [ Colour Number: $j ] ===================== ${RESET}"
        printf "\n ${CLR} ============================================================= ${RESET} \n"
    done
    ```

- ### References
    > Self

</br></br></br></br>


<h2 id="BSES"></h2>

## BASH Stream Editor Shortcuts (sed)
[Back to Top](#top)
- You can concatenate multiple edits together:
    ```bash
    sed -i 's/\"//g;s/\,/\n/g;s/\s//g' input_file

    x="$(cat timestamp.txt  | sed 's#'\n'# #g')"
    echo $x
    timestamp_array=(${x//$'\n'/ })
    echo ${timestamp_array[10]}
    for i in ${timestamp_array[@]}
    do
        echo $i
    done
    ```

- List of valid delimiters for sed:
    ```bash
    $ sed 's/a/b/g' <<< "haha"
    hbhb

    $ sed 's_a_b_g' <<< "haha"
    hbhb

    $ sed 's#a#b#g' <<< "haha"
    hbhb

    $ sed 's$a$b$g' <<< "haha"
    hbhb

    $ sed 's?a?b?g' <<< "haha"
    hbhb

    $ sed 's*a*b*g' <<< "haha"
    hbhb

    $ sed 's-a-b-g' <<< "haha"
    hbhb

    $ sed 's.a.b.g' <<< "haha"
    hbhb

    $ sed 'sXaXbXg' <<< "haha"
    hbhb

    $ sed 'sxaxbxg' <<< "haha"
    hbhb

    $ sed 's1a1b1g' <<< "haha"
    hbhb
    ```
- ### References
    > stackoverflow.com

</br></br></br></br>


<h2 id="BREGEX"></h2>

## BASH Regular Expression Shortcuts
[Back to Top](#top)
- `[[ "$var" =~ ^[0-9]+$ ]]` - To check if the variable is a digit
- `[[ "$var" =~ ^[[:digit:]]{5}$ ]]` - To check if the variable is a 5 digit


- ### References
    > stackoverflow.com

</br></br></br></br>


<h2 id="BPPFP"></h2>

## BASH Python Path for Packages
[Back to Top](#top)

- Sometimes python is NOT able to find its packages:
    ```bash
    $ export PYTHONPATH=$PYTHONPATH:/usr/lib/python3/dist-packages
    $ echo $PYTHONPATH
    $ python -c "import sys; print(sys.path)"
    $ python -c "from scapy.all import *;Ether() / bytearray(10)"
    ```

- ### References
    > stackoverflow.com

</br></br></br></br>


<h2 id="UNN"></h2>

## Ubuntu Network Namespaces
[Back to Top](#top)
- ### `Custom Prompt`
    ```bash
    NS="ACU_P8"
    PS1_NS="\033[3;38;5;197m${NS}\033[3;38;5;15m@\033[3;38;5;179m$(hostname)\033[3;38;5;15m:$ \033[0m"
    sudo ip netns exec ${NS} /bin/bash --rcfile <(echo "PS1=${PS1_NS}")

    ^^^^ Work in Progress

    NS="ACU_P8"
    sudo ip netns exec ${NS} /bin/bash
    NS="ACU_P8"
    PS1="\033[3;38;5;197m${NS}\033[3;38;5;15m@\033[3;38;5;179m$(hostname)\033[3;38;5;15m:$ \033[0m"

    ^^^ Works
    ```

- ### References
    > https://gist.github.com/dpino/6c0dca1742093346461e11aa8f608a99

</br></br></br></br>

<h2 id="UTC"></h2>

## Ubuntu Terminal Cheatsheet
[Back to Top](#top)
### Shell
- A (possibly interactive) command interpreter, acting as a layer between the user and the system.
Bash: The Bourne Again Shell, a Bourne compatible shell.

- Traditionally, a shell prompt either ends with `$ %  #`
    - `$` = Indicates a shell that's compatible with the Bourne shell (such as a POSIX shell, or a Korn shell, or Bash)
    - `%` = Indicates a C shell (csh or tcsh)
    - `#` = Indicates that the shell is running as the system's superuser account (root)

- **Manual Pages**
    - `man man`     = contains the manual of all the commands (ex: man ssh)
    - `man apropos` = search the manual page names and descriptions

### Shell Shortcuts
```bash
    C-a     # Move cursor to beginning of line
    C-e     # Move cursor to end of line
    C-l     # Clear-screen
    C-r     # Reverse Search History
    C-p     # Walk back in History
    C-n     # Walk Forward in history
    C-k     # Kill-line forward of cursor
    C-u     # Kill-line forward of cursor
    C-xx    # Move between start of the command line and the current position
    C-z     # Suspend/stop the command (puts the current in background)

    Opt-R Arrow  # Move cursor forward 1 word
    Opt-L Arrow  # Move cursor backwards 1 word
```

- ### References
    > https://gist.github.com/dpino/6c0dca1742093346461e11aa8f608a99

</br></br></br></br>


<h2 id="UPCI"></h2>

## Ubuntu Package Check and Install
[Back to Top](#top)
- There is also a way to search for a particular package:
    ```bash
    $ apt search nvidia-graphics-driver
    ```
- Below is a helper function designed by me which will check for the requirements and install them if they are NOT installed.
    ```bash
    #!/bin/bash
    _installreq()
    {
        update_pkg_mgr()
        {
            echo -e "\n\n ${YELLOW}Updating apt-get package manager.${NC}\n\n"
            sudo apt update && sudo apt upgrade -y
        }

        apt_get_install()
        {
            sudo apt-get install -y $1 #&> /dev/null 2>&1
        }

        install_pyreq()
        {
            if [[ -f ./requirements.txt ]]
            then
                pip3 install -r requirements.txt
            else
                echo -e "${BLUE} The 'requirements.txt' is missing in the current directory.${NC} \n"
            fi
        }

        run_installer()
        {
            ## $1 - App to check
            ## $2 - Installer Program to Run <apt_get | dpkg>

            echo -e "${BLUE} $1 --> [NOT Installed].${NC} \n"
            echo -e "${YELLOW} Installing $1 using $2...${NC} \n"
            case $2 in
                apt_get|dpkg)
                    apt_get_install "$1"
                    ;;
                *)
                    echo -e "${RED} $1 [Installation NOT Supported] ${NC} \n"
                    ;;
            esac
            if [[ "$?" == "0" ]]
            then
                echo -e "\n ${GREEN} $1 [Installed] ${NC} \n"
            else
                echo -e "${BLUE} $1 --> [Failed Installation].${NC} \n"
            fi
        }

        check_program()
        {
            ## $1 - App to check
            ## $2 - Installer Program to Run <apt_get | dpkg>

            if [[ "$2" =~ ^(apt_get)$ ]]
            then
                if command -v "$1" &> /dev/null 2>&1
                then
                    echo -e "\n ${GREEN} $1 [Installed] ${NC} \n"
                else
                    run_installer "$1" "$2"
                fi

            elif [[ "$2" =~ ^(dpkg)$ ]]
            then
                if dpkg -s "$1" &> /dev/null 2>&1
                then
                    echo -e "\n ${GREEN} dpkg Module $1 [Installed] ${NC} \n"
                else
                    run_installer "$1" "$2"
                fi
            fi
        }

        local RED='\033[0;31m'
        local BLUE='\033[1;34m'
        local GREEN='\033[0;32m'
        local YELLOW='\033[1;33m'
        local NC='\033[0m'

        update_pkg_mgr

        install_apps=("python3=dpkg" "python3-pip=dpkg" "scapy=apt_get" "sshpass=apt_get" "net-tools=apt_get" "moreutils=dpkg")

        for element in "${install_apps[@]}"
        do
            app="${element%=*}"
            mgr="${element#*=}"
            #echo "Key: $app, Value: $mgr"

            check_program "$app" "$mgr"
        done

        install_pyreq
    }
    ```

- ### References
    > Self

</br></br></br></br>

<h2 id="UUSB"></h2>

## Ubuntu Force a USB 3.0 port to work in USB 2.0 mode
[Back to Top](#top)
- The controllers have a register XUSB2PR – xHC USB 2.0 Port Routing Register – at address 0xd0
- When the XUSB2PR register is set to 0, it routes all the corresponding USB 2.0 port pins to the EHCI controller and RMH #1
- The USB 2.0 port is masked from the xHC and the USB 2.0 port’s OC pin is routed to the EHCI controller. (Below Command does it)
    ```bash
    $ setpci -H1 -d @ d0.l=0
    ```
- setpci needs the vendor and device ID. So the first 2 lines find all USB controller’s IDs and pass them to xargs to invoke setpci.
    ```bash
    $ lspci -nn | grep USB

        00:14.0 USB controller [0c03]: Intel Corporation C610/X99 series chipset USB xHCI Host Controller [8086:8d31] (rev 05)
        00:1a.0 USB controller [0c03]: Intel Corporation C610/X99 series chipset USB Enhanced Host Controller #2 [8086:8d2d] (rev 05)
        00:1d.0 USB controller [0c03]: Intel Corporation C610/X99 series chipset USB Enhanced Host Controller #1 [8086:8d26] (rev 05)

    $ lspci -nn | grep USB | cut -d '[' -f3 | cut -d ']' -f1

        8086:8d31
        8086:8d2d
        8086:8d26
    $ sudo setpci -H1 -d 8086:8d31 d0.l=0
    $ sudo setpci -H1 -d 8086:8d2d d0.l=0
    $ sudo setpci -H1 -d 8086:8d26 d0.l=0

    # One line-shot
    $ lspci -nn | grep USB | cut -d '[' -f3 | cut -d ']' -f1 | xargs -t -I@ setpci -   H1 -d @ d0.l=0
    ```
- ### References
    > https://www.systutorials.com/how-to-force-a-usb-3-0-port-to-work-in-usb-2-0-mode-in-linux/

</br></br></br></br>

<h2 id="UDSL"></h2>

## Ubuntu `Systemd` service
[Back to Top](#top)
- Create a file named `serviceName.service` and include the following
- Syntax:
    ```
    [Unit]
    Description=<description about this service>

    [Service]
    User=<user e.g. root>
    WorkingDirectory=<directory_of_script e.g. /root>
    ExecStart=<script which needs to be executed>
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
- Example:
    ```
    [Unit]
    Description=key server for product and dev key generation
    After=network.target

    [Service]
    Type=simple
    WorkingDirectory=/home/akshayd/test/.key_server
    ExecStart=/home/akshayd/test/key_server &
    TimeoutStartSec=0

    [Install]
    WantedBy=default.target
    ```
- To Enable the new service:
    ```bash
    # Reload the service files to include the new service.
    $ sudo systemctl daemon-reload

    # Start your service
    $ sudo systemctl start serviceName.service

    # check the status of your service
    $ sudo systemctl status serviceName.service

    # enable your service on every reboot
    $ sudo systemctl enable serviceName.service

    # disable your service on every reboot
    $ sudo systemctl disable serviceName.service
    ```
- Any script that you add this the below folder location will be executed at the time of reboot `/etc/init.d`


- ### References
    > google.com

</br></br></br></br>


<h2 id="UCJ"></h2>

## Ubuntu Cron Jobs
[Back to Top](#top)

- Execute the command to open the file where we can add:
    ```bash
    # Choose the editor to open in if doing for the first time
    $ sudo crontab -e
    ```
- If you want some script to start on a periodic manner add it:
    ```bash
    # NOT every version of cron supports @reboot
    @reboot sh /home/Ak/setup_something.sh
    @yearly sh /home/Ak/setup_something.sh
    @monthly sh /home/Ak/setup_something.sh
    @weekly sh /home/Ak/setup_something.sh
    @daily sh /home/Ak/setup_something.sh
    @midnight sh /home/Ak/setup_something.sh
    @hourly sh /home/Ak/setup_something.sh
    ```
- If you want more granularity:
    ```bash
    # <minute> <hour> <day-of-month> <month> <day-of-week> <command>
    * * * * * sh /home/Ak/setup_something.sh

    # Run at 12 noon everyday
    0 12 * * ? sh /home/Ak/setup_something.sh
    ```

- ### References
    > google.com

</br></br></br></br>


<h2 id="UCPUS"></h2>

## Ubuntu CPU Stats
[Back to Top](#top)
- Cores = Cores per socket X Sockets
    ```bash
    $ echo "Cores = $(( $(lscpu | awk '/^Socket\(s\)/{ print $2 }') * $(lscpu | awk '/^Core\(s\) per socket/{ print $4 }') ))"
    ```

- CPUs are what you see when you run `htop`:
    ```bash
    $ lscpu | grep -E '^Thread|^Core|^Socket|^CPU\('
    ```

- The output of `nproc` corresponds to the CPU count from `lscpu`:
    ```bash
    $ nproc --all
    ```

- The cpu cores reported by `/proc/cpuinfo` corresponds to the **Core(s) per socket** reported by lscpu.
    ```bash
    $ grep -m 1 'cpu cores' /proc/cpuinfo
    ```

- Another useful utility is `dmidecode` which outputs per socket information:
    ```bash
    $ sudo dmidecode -t 4 | grep -E 'Socket Designation|Count'
    ```

- The `lscpu` command has a number of useful options that you may like to check out:
    ```bash
    lscpu --all --extended
    lscpu --all --parse=CPU,SOCKET,CORE | grep -v '^#'
    lshw | grep cpu:
    ```

- ### References
    > https://unix.stackexchange.com/questions/218074/how-to-know-number-of-cores-of-a-system-in-linux

</br></br></br></br>

<h2 id="USPNSSD"></h2>

## Ubuntu Set Permanent Naming Scheme for Serial Devices
[Back to Top](#top)

- When we connect a serial cables from a device:
    ```bash
    $ ls /dev/ttyUSB
    ttyUSB0    ttyUSB1    ttyUSB2
    ```

- When we wish to Log/Open the serial port to view the logs we will need to specify the port.
    ```bash
    $ sudo minicom -w -c on -D /dev/ttyUSB1
    ```

- The above can be stored in a file as a database. But when the hostmachine to which these cables are connected Reboots.

- The assignment of the ttyUSBx is random and cant be controlled.

- In order to solve this problem we need to bind the serial number of the cable to an alias: `ttyUSB1 ---> ttyUSBDev1`


- Find out the `ID_SERIAL_SHORT` of the UART cable connected to the host machine.
    ```bash
    # Replace the `x` with a number
    $ sudo udevadm info --query=property --name=/dev/ttyUSBx | grep -i "ID_SERIAL_SHORT"
    ID_SERIAL_SHORT=CIDYb116L16
    ```
- In case the `ID_SERIAL_SHORT=""` (empty) that means that the UART cable doesnt have a serial number.
- Please order a different UART cable which will have a serial number.

    https://www.amazon.com/gp/product/B07RFNHTL9/ref=ppx_yo_dt_b_asin_title_o08_s00?ie=UTF8&th=1


- Add the serial number to the below line:
    ```bash
    ACTION=="add",ENV{ID_BUS}=="usb",ENV{ID_SERIAL_SHORT}=="CIDYb116L16",SYMLINK+="ttyUSBp4lx1"
                                                            ^^^^^^^^^^             ^^^^^^^^^^
                                                        Add Serial No.      Set an Appro. Name
    ```
- Paste this line in the following file:
    ```bash
    $ vim /etc/udev/rules.d/99-usbserial.rules
    ```

- ### References
    > Self

</br></br></br></br>

<h2 id="URTDU"></h2>

## Ubuntu Real Time Disk Usage (ncdu)
[Back to Top](#top)

- Installation
    - Linux - `sudo apt install ncdu`
    - MacOS - `brew install ncdu`

</br>

- ### References
    > stackoverflow.com

</br></br></br></br>


<h2 id="VC"></h2>

## VIM Cheatsheet
[Back to Top](#top)
### Syntax

- Syntax of the Language is expressed as `Verb + Noun`
    ```
    Ex:
        dw = d - delete
             w - word
    ```
- All the changes in Vim are *<u>Repeatable</u> and <u>Undoable</u>*

    - `Repeat`:  To repeat any change you made just press `.` dot
    - `Undo`: To undo any change that you man just press `u`

- The `Verbs` in VIM
    - `d` - Delete
    - `c` - Change (delete and enter the Insert mode)
    - `>` - Indent
    - `<` - Outdent
    - `v` - Visually Select
    - `y` - Yank (Copy)

- The `Nouns` in VIM
    - `w` - forward word
    - `b` - backward word
    - `2j` - current line

- Examples:
    ```
    >0 will indent the current line
    >j will indent the current line and 1 line below
    >2j will indent the current line and 2 lines below
    ```
    ```
    :set number = set number
    dd          = deletes the entire line
    ndd         = n is a number, deletes nlines

    u           = undo
    Ctrl+r      = redo

    /<string>   = searches for a string
    N           = previous search
    n           = forward search
    :%s/<string to search>/<string to replace>/g

    g           = replaces all occurrences
    gc          = asks before replacing every occurrence

    gg          = cursor reaches the start of document
    G           = cursor reaches the end of document
    0           = reaches the front of line
    $           = reaches the end of line
    :$          = reaches the end of document

    Ctrl+b      = page up
    Ctrl+f      = page down
    Ctrl+y      = page up by 1 line
    Ctrl+e      = page down by 1 line

    :q          = to quit
    :q!         = to quit without saving
    :wq         = to write and quit
    :wq!        = to write and quit
                  even if file has only read permission
                  (if file does not have write permission: force write)

    :x          = to write and quit (similar to :wq, but only write if there are changes)
    :exit       = to write and exit (same as :x)
    :qa         = to quit all (short for :quitall)
    :cq         = to quit without saving and make Vim return non-zero error (i.e. exit with error)
    ```

- Write a READONLY File in Vim -->   `  :w !sudo tee %`

    - `:w` = Write a file.
    - `!sudo` = Call shell sudo command.
    - `tee` = The output of the vi/vim write command is redirected using tee.
    - `%` = Triggers the use of the current filename.
    - Simply put, the ‘tee’ command is run as sudo and follows the vi/vim command on the current filename given.
    - Press O and the file will be saved
    -  It remains open in vi/vim for more editing or reading and you can exit normally by typing :q! since the file is still open as read-only.



- ### References
    > https://vim.rtorr.com/

</br></br></br></br>






<h2 id="TC"></h2>

## TMUX Cheatsheet
[Back to Top](#top)
- Installation: `sudo apt-get install tmux`
- Shortcuts
    - `tmux ls`                            = lists all the sessions
    - `tmux a #`                           = attaches to last created session

    - `tmux new -s <name_of_session>`      = starts a session with new names
    - `tmux a -t <sess_name|sess_num>`     = attach to specified session

    - `tmux -CC new -s <name_of_session>`  = starts a session with new names
    - `tmux -CC a -t <sess_name|sess_num>` = attach via CC to specified session

    - `tmux kill-session -t <sess_name>`   = kills a particular session
    - `tmux kill-server`                   = kill all sessions

    - `tmux kill-session -a`               = kill all sessions except the current one

- Usage:
    ```
    ctrl+b = command mode

    :           = tmux prompt
    d           = detach a session.
    x           = kill a session.
    "           = split horizontally
    %           = split vertically
    <arrow_key> = move from pane to pane
    ```

- ### References
    > https://tmuxcheatsheet.com/

</br></br></br></br>


<h2 id="MOMSA"></h2>

## MacOS Open Multiples of the Same Application
[Back to Top](#top)
- To open more than 2 windows of the same applcication:
    ```
    $ open -n /Applications/Adobe\ Acrobat\ Reader.app
    ```

- ### References
    > stackoverflow.com

</br></br></br></br>


<h2 id="MRTPA"></h2>

## MacOS Reset Trial Period of Applications
[Back to Top](#top)
- There are some applications which mention that 45 days of trial are left.
- There is a handy way in which we can reset it back to 45 days.
    ```bash
    $ cd  /Users/akshaydandgaval/Library/Application\ Support/Beyond\ Compare/
    ```
    - Delete all the files in that dir.
        ```bash
        rm -f  /Users/akshaydandgaval/Library/Application\ Support/Beyond\ Compare/registry.dat
        ```

- ### References
    > stackoverflow.com

</br></br></br></br>



<h2 id="WSL"></h2>

## Windows Sub-System for Linux (WSL)
[Back to Top](#top)
- ### **WSL 1**
    - It allows windows users to:
        - Run a Linux terminal environment
        - Install Packages from Ubuntu Archive
        - Run Linux applications and workflows

    - The original WSL is now known as WSL1.
    - It is a compatibility layer for running (ELF) Linux Binary Executables natively on Win 10.
    - NO re-compilation and porting is required.
    - WSL1 provides a Linux-compatible kernel interface developed by Microsoft.
    - It executes ELF64 Linux Binaries.
    - > It works by operating the linux kernel interface on top of the Win 10 kernel.
    - It translates calls from the linux sys --> win system calls


- ### **WSL 2**
    - WSL2 was announced at Microsoft Build 2019.
    - It features a linux kernel runing inside of Win 10.
    - It is built on the core tech. of Hyper-V virtualization (Available only on Windows 10 Pro, Enterprise, and Education).
    - Open Power Shell as Admin
        ```powershell
        Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
        ```
    - Uses Virtualization technology to run a Linux kernel inside of a lightweight utility virtual machine (VM).
    - Increase file system performance.
    - Support full system call compatibility.
    - It does use a VM, it is managed and run behind the scenes. (Fast boot, small resource).
    - The full linux kernel is built by Microsoft, tuned for WSl.
    - File intensive operations like git clone, apt update, unzip tarball 20x faster and 2-5x faster for git clone.
    - You can run Docker inside of WSL and othe new apps.
    - Any updates to the linux kernel are ready for use.
    - It uses a VHD (Virtual Harddrive) to store the linux files.
        - It is represented as .vhdx on windows.
        - It has ext4 file system format.
        - It has a initial maximum size of 256GB and it automatically resizes to meet storage req.
    - WSL 2 CANT access IPv6-only addresses.
    - Linux file system root directory:
        ```powershell
        # To Find the root of a Distribution
        $ \\wsl$
        $ cd \\wsl$\Ubuntu-18.04\home

        # To know which windows version is running
        $ winver
        ```

    - It offers faster access to files mounted from windows.
    - Faster access to linux files from Windows applications.
    - Doesn't support serial port access.
    - WSL 2 has a virtualized ethernet adapter with its own unique IP address.
    - Currently, to enable this workflow you will need to go through the same steps as you would for a regular virtual machine.

        ```powershell
        # Connects it to port 4000 to the WSL 2 VM
        port proxy that listens on port 4000 on Host

        netsh interface portproxy set v4tov4 listenport=8888 listenaddress=0.0.0.0 connectport=8888 connectaddress=$(wsl hostname -I)
        ```

    - Examples:
        ```powershell
        $ winver
        $ wsl --install -d Ubuntu
        $ wsl -l -v
        $ wsl --set-default-version 1
        $ wsl hostname -I
        ```
- ### References
    > microsoft.com

</br></br></br></br>



<h2 id="GIT"></h2>

## git (Global Information Tracker)
[Back to Top](#top)
- It was written by Linux Torvalds in 2005

- ### References
    >  git, self, google.com

</br></br></br></br>

<h2 id="GWPP"></h2>

## git (Work and Personal Profiles)
[Back to Top](#top)

- Create a Dir which you want to be your personal directory.
    ```
    /myhome/
        |__.gitconfig
        |__.gitconfig_office
        |__.gitconfig_personal
        |__work/
        |__personal/
    ```

- We need to perform a few steps:
    - Create a NEW SSH key for laptop you are in:

        - Execute these commands:
            ```bash
            $ cd ~/.ssh
            $ ssh-keygen -t ecdsa -C "email@example.com" -b 521 -f "id_ecdsa_p"
            ```

        - Copy the contents of `id_ecdsa_p.pub` from `~/.ssh` and go to your GitHub account and add this SSH Key.

    - We need need to know what is the path of the folder where all files will be personal files. Ex: `/myhome/personal/`

    - Create a `.gitconfig_personal` file to switch to that SSH Key which is linked to Personal Git Account.

        - Now we want to switch to personal account when we push changes in the personal directory the `.gitconfig_personal` file will do exactly that.
            ```bash
            $ cd ~
            $ vim .gitconfig_personal
            ```
            - In that file add the the following:
            ```
            [user]
            email = email@example.com
            name = <name in Github>

            [github]
            user = "<Github Username>"

            [core]
            sshCommand = "ssh -i ~/.ssh/id_ecdsa_p"
            ```

        - The above file will tell git to use the above given details.

        - Now we need to add a condition that whenever git is used in `/myhome/personal/` we have to use `~/.gitconfig_personal`

        - To do that we need to edit `.gitconfig`, add this line to it.
            ```
            [includeIf "gitdir:~/personal/"]
        	path = ~/.gitconfig.p
            ```
        - This will tell git that when it is in "~/personal/" it needs to use "~/.gitconfig.p"

        - Execute the command:
            ```bash
            $ git config --list
            includeif.gitdir:~/personal/.path=~/.gitconfig.p
            ```

- ### References
    > https://blog.gitguardian.com/8-easy-steps-to-set-up-multiple-git-accounts/

</br></br></br></br>



<h2 id="VSS"></h2>

## VS Code Shortcuts
[Back to Top](#top)
- Line Wrap - `Option + Z`
- To Add/Remove a code book mark: `Cmd + Shift + <1 - 9>`
- To Go to that bookmark: `Command + <1 - 9>`
- ### References
    > microsoft.com

</br></br></br></br>




<h2 id=""></h2>

## T
[Back to Top](#top)

- ### References
    > Self

</br></br></br></br>

<h2 id=""></h2>

## T
[Back to Top](#top)

- ### References
    > Self

</br></br></br></br>


