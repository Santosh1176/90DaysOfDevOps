# Variables in Linux

Linux environment variables help define your Linux shell experience. Many programs and scripts use environment variables to obtain system
information and store temporary data as well as configuration information. The Bash shell uses a feature called **environment variables** to store information about the shell session and the working environment (thus the name environment variables). This allows the storage of data in memory which can be easily accessed by the shell script. The **system environment variables** almost always use all capital letters to differentiate them from user-defined variables.

## Global variables

Global environment variables are visible from the shell session and from any spawned child subshells. The Linux system sets several global environment variables when you start your Bash session. To view global environment variables, use the `env` or the `printenv`.

```bash
santosh@90DaysOfDevOps*(main)$:env   # You get the same result when `printenv` is used
SHELL=/bin/bash
SESSION_MANAGER=local/santoshdts:@/tmp/.ICE-unix/3195,unix/santoshdts:/tmp/.ICE-unix/3195
QT_ACCESSIBILITY=1
COLORTERM=truecolor
XDG_CONFIG_DIRS=/etc/xdg/xdg-ubuntu:/etc/xdg
SSH_AGENT_LAUNCHER=gnome-keyring
<snip>
.
.
.
<snip>
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus
NVM_BIN=/home/santosh/.nvm/versions/node/v16.14.0/bin
GIO_LAUNCHED_DESKTOP_FILE_PID=24363
GIO_LAUNCHED_DESKTOP_FILE=/var/lib/snapd/desktop/applications/code_code.desktop
GOPATH=/home/santosh/go
TERM_PROGRAM=vscode
_=/usr/bin/env
```
However, to display an individual environment variable's value, you can use the `printenv` command, but not the `env` command.

```bash
santosh@90DaysOfDevOps*(main)$:printenv GOPATH   # echo $GOPTAH yields same result
/home/santosh/go                                 
santosh@90DaysOfDevOps*(main)$:printenv GITHUB_USER  #  # echo $GITHUB_USER yields same result
Santosh1176
```
You can use `echo` command to obtain the same result but we need to prefix `$` before the variable. `echo $GOPATH` or `echo $GITHUB_USER` in our example. Apart from printing the value when used with `echo` command. The dollar sign before a variable name allows the variable to be passed as a parameter to various other commands. 

```bash
santosh@90DaysOfDevOps*(main)$:ls $HOME
agent-policy-document.json   coredns       git-prompt.sh            minikube-linux-amd64   root.key
agent-trust-policy.json      create.sh     go                       Music                  santoshdts_accessKeys.csv
<snip>
 ```



## Local Variables


Local variables are available only in the shell that creates them. Even though they are local, they are just as important as global environment variables. Apart from some Linux-defined Local variables, we can also define your own local variables. These, as you would assume, are called `user-defined local variables`. Unfortunately, there isn't a command that displays only these variables. The `set` command displays all variables defined for a specific process, including both local and global environment variables as well as user-defined
variables.

## Setting Local Variables

You can assign either a numeric or a string value to an environment variable by assigning the variable to a value using the equal sign:
```bash
santosh@90DaysOfDevOps*(main)$:my_variable=Hello
santosh@90DaysOfDevOps*(main)$:echo $my_variable
Hello
```
Here we defined a single word ( without space), If we need to define a local variable with white spaces, we need to encapsulate the string value in **quotes**, like so:

```bash
santosh@90DaysOfDevOps*(main)$:my_variable="Hello World!"
santosh@90DaysOfDevOps*(main)$:echo $my_variable
Hello World!
```

> Note:It is extremely important to note that we do not use spaces between the variable name, the equal sign, and the value.

To remove an existing local environment variable, we use `unset` command:
`unset my_variable`

## Variable arrays

A really cool feature of environment variables is that they can be used as arrays. An array is a variable that can hold multiple values. Values can be referenced either individually or as a whole for the entire array. For example:

```bash
santosh@90DaysOfDevOps*(main)$:my_variable=(one two three four)  # Setting variable as arras
santosh@90DaysOfDevOps*(main)$:echo ${my_variable[1]}  # Accesing specific variable from an array
two
santosh@90DaysOfDevOps*(main)$:my_variable[1]=five   # Modifying an value with the array
santosh@90DaysOfDevOps*(main)$:echo ${my_variable[1]}
five
santosh@90DaysOfDevOps*(main)$:${my_variable[*]}  # To display an entire array variable, you use the asterisk (*) wildcard character
one five three four
```