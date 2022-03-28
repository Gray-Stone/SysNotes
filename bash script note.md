# Notes to make bash scripts

Bash double array for loop 
https://stackoverflow.com/questions/17403498/iterate-over-two-arrays-simultaneously-in-bash

Good thing about how to kill all process launched from a script
https://unix.stackexchange.com/questions/124127/kill-all-descendant-processes


## Heredoc

Here doc `<<` is a way to quickly generate some file on the fly then pipe it into something (combined with cat or echo)

### Mixing Evaluating and Not

One of the feature of heredoc is the quotes around the delimiter ` cat <<EOF ` vs ` cat <<'EOF' ` When the delimiter is quoted, nothing in the heredoc is expended. This allows you to put any strange character into it and not worry about colliding with reserved keyword/symbols. Without the quotes, variables and expressions are evaluated.

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