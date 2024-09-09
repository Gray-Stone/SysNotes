
SSH have the ability to do X11 forwarding, which allow the app to be displayed on the remote client when ssh into the server.

The trick is all in setting the `DISPLAY` env-var.

```
export DISPLAY=:0
```

Will make all following GUI apps launch in primory monitor of the remote(server) machine.

or for just one app 

````
DISPLAY=:0 gnome-panel
```

https://unix.stackexchange.com/questions/193827/what-is-display-0
