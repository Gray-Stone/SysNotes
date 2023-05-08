
Z shell have built in for socket and tcp control. 
https://www.jianshu.com/p/05a68184379c



## Cache half typed command:

https://sgeb.io/posts/bash-zsh-half-typed-commands/

> push-input: kills the whole, potentially multiline, command and yanks it on the next fresh prompt. It comes with the nice side effect that a multiline command becomes editable again for all lines. In vanilla zsh this is bound to Ctrl-q (along with esoteric Esc-Q and Esc-q).

`ctrl q` is the way to go.

## Useful info on zsh line editor

https://sgeb.io/posts/zsh-zle-custom-widgets/





## Configure and Enable CD history.
source: https://unix.stackexchange.com/a/157773

```
setopt AUTO_PUSHD                  # pushes the old directory onto the stack
# PushHD Minus is a matter of persional taste, and I don't think this change anything on my machine.
# setopt PUSHD_MINUS                 # exchange the meanings of '+' and '-'
setopt CDABLE_VARS                 # expand the expression (allows 'cd -2/<subfolder>')
autoload -U compinit && compinit   # load + start completion
zstyle ':completion:*:directory-stack' list-colors '=(#b) #([0-9]#)*( *)==95=38;5;12'
```

## Config No glob match behavior.

I want it more like bash, pass the glob down.

Source: https://superuser.com/a/982399

This change the default NOMATCH to off, and pass the glob down instead of throwing error
```
setopt NO_NOMATCH 
```

