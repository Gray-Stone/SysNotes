# Notes to make bash scripts

Bash double array for loop
https://stackoverflow.com/questions/17403498/iterate-over-two-arrays-simultaneously-in-bash

Good thing about how to kill all process launched from a script
https://unix.stackexchange.com/questions/124127/kill-all-descendant-processes


## Subshell

WIP 

`sh -c` or `bash -c` or `zsh -c` actually doesn't create a subshell

https://stackoverflow.com/questions/23951346/is-there-a-subshell-created-when-i-run-sh-c-command-a-new-shell-or-none-of

However, PPID dose seems to change. So whe application want to detect weather it's a systemd will become tricky if it ever get wrapped in a subshell.


## Heredoc

Here doc `<<` is a way to quickly generate some file on the fly then pipe it into something (combined with cat or echo)

### Mixing Evaluating and Not

One of the feature of heredoc is the quotes around the delimiter `cat <<EOF` vs `cat <<'EOF'` When the delimiter is quoted, nothing in the heredoc is expended. This allows you to put any strange character into it and not worry about colliding with reserved keyword/symbols. Without the quotes, variables and expressions are evaluated.

There are cases when both behavior is needed, this could be very annony.

A quick solution for mixing small amount of special character within a evaluated heredoc is to use echo

```bash
#! /usr/bin/bash

set -euvx
cat <<EOF >> ./my_markdown.md 
This is a markdown file, here is the generated code snippit
$( echo '```' )
# This is the command you should run
${HOME}/somescript
$( echo '```' )
EOF

```

This reason this work is echo doesn't expend content quoted using single quote. And here doc will just evaluate the echo and place the output of echo into the doc.

The terminal output of running this script is:

```
$ ./script.sh
cat <<EOF >> ./my_markdown.md 
This is a markdown file, here is the generated code snippit
$( echo '```' )
# This is the command you should run
${HOME}/somescript
$( echo '```' )
EOF
+ cat
++ echo '```'
++ echo '```'
```

# Python in Bash

YES! write Python inside bash, not the other way around.

Python is good for writing quick string parsing, use some build in library for quick stuff. It is not designed to directly interact with system and other commands like bash. One prefect example is argument parsing. Python have this `argparse` library which is super quick and easy to write nice command line argument parsing.

## Basic syntax

To get the benefit of Python in a bash script, without keeping everything in a single file/script, you will need to use python with heredoc.

```bash
python3 <<EOF
import argparse
import pathlib

parser = argparse.ArgumentParser()
parser.add_argument("input_file" ,type=str,help="base file to operate")
parser.add_argument("--other" , help="extra options" , type=str ,default="")

args = parser.parse_known_args("$*".split(" "))[0] 
# This doesn't handle space in single argument at all. Read on for better design

solve_path = lambda p : pathlib.Path(p).resolve() if p else ""
print(f"""
FILE_TO_USE={solve_path(args.input_file)}
OTHER_OPTIONS={args.other}
""")
EOF
```

## Make the python script output something into bash

The hard part of using python is how to get the information back out. python is a subprocess run here, there are basically no way of sharing/setting variable from inside python and have outside bash script sees it.

 The easiest I have found is to use the stdout from python to print in bash syntax script, then run the stdout. I mostly use python for argument and string process, so I always output a string that define Bash variables.

Here is an example

```bash
python_arg_process(){
python3 <<EOF
# Some python program like above
print(f"""
ARG_1={python_var_from_above}
ARG_2={other_variable}
""")
EOF
}
for i in "$@"; do
	if [[ "$i" == "-h" ]]  ; then
		PYTHON_OUTPUT=$(python_arg_process -h)
		echo "$PYTHON_OUTPUT"
		exit 0
	fi
done


PYTHON_OUTPUT=$(python_arg_process "$@")
echo "$PYTHON_OUTPUT"
eval "${PYTHON_OUTPUT}" # run the stdout from python as if it's some commands

# rest of your shell script that use these.

```

Let me explain what's happening:

The bash loop looks for any option with -h, which is the default flag `argparse` look for when generating the help message. In case user has input -h, the stdout from python is just the help message, no longer the actual arguments. Thus we want to only print this and exit.

the python script is put into a function to be called, you don't have to do it this way. The output string is a list of variable definitions in bash format. The stdout from python is stored in a variable, this allowed me to first display these variables, then call `eval` on them to actually define the variable and make them usable

Another important thing to pay attention is the triple quote. This is a python syntax that allow you to type a "brick" of text. By doing a triple quote f-string, you can easily generate a brick of bash syntaxed script. What's better is this allow you to add quotes within the brick without escapes (something very important for bash)

## How to debug my shitty python code?

When the stdout is used to feed into bash, you can't just keep adding print statements to python script to debug it.

The easiest way is simply add a `exit` command in bash right after the python output is printed. This way you can see your python's output without worry your debug print statements gobble up and cause bad things in further bash process.

Anther way is to use stderr. You have two output pipe, stdout and stderr, with some smart bash tricks, you can use one for feeding things to bash (most likely stdout) and other other one to print debug messages (likely stderr)

## Python make a bash list/array

```python
# some arg parsing here 
for file in args.files:
	output_filename_str +=f'"{file}}" \t '

print(f"""
OUTPUT_FILE_ARRAY=({output_filename_str})
""")
```

Notice how I added double quote inside the single quote when constructing the string in the for loop. This way prevented globbing and word splitting see [SC2086](https://github.com/koalaman/shellcheck/wiki/SC2086)

## Handle spaces in cli arguments when passing it into python

Issue in pervious example is python is splitting the args using space. When using `$@`, bash know from where to where is one argument, however, when it is printed, this grouping is lost. Unexpected Word splitting happened during the transition from bash to python if any space exist.

So here we are re-generating this list from bash into python's syntax to preserve the grouping before python start processing it.

```
args = parser.parse_args([ $(for var in "$@" ; do  echo -n "'$var' , " ; done) ] )
```
