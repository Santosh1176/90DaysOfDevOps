# Frequently used Linux Commands

Before we start lets get comfortable with the manuals provided by the Linux dirstribution for various commands the terminal provides. We can access details of any command by prefixing `man` to that command for ex: `man ls`.
For a beginner the output provided by the `man`, of any command, though in detail but can be bit overwhelming.
We can liik for the shortest possible description of what a command does by passing a `-f` flag to the command. for ex: `
```bash
santosh@~*$:man -f ls
ls (1)               - list directory contents
``` 
So, lets start with the basic obe `ls` as we've seen above it *lists all the contents in the current working directory.*

`ls <path name` lists the content of the specified directory.
```bash
santosh@~*$:ls ~/west
emoji.yml  kustomization.yml  kustomize  ns.yml  vote-bot.yml  voting.yml  web.yml
```      

