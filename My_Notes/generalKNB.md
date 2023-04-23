# General Knowledge Base

## BASH Debug Ability
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


## BASH Static Analysis Tool
- Install a VS Code extension (ShellCheck)
- To ingore some of its warnings:
    ```
    # shellcheck disable=SC2034
    ```

- Which will help find faults in BASH scripts.

    >https://marketplace.visualstudio.com/items?itemName=timonwong.shellcheck


</br>
</br>


## BASH Strict Mode

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


</br>
</br>


- ### References
    > http://redsymbol.net/articles/unofficial-bash-strict-mode/

    > https://dev.to/banks/stop-ignoring-errors-in-bash-3co5

    > https://github.com/tests-always-included/wick/blob/master/doc/bash-strict-mode.md

<br>
<br>

## Linux Network Namespaces

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

