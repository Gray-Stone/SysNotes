# GDB is great

## Use gdb in a non-interactive way. 

```
gdb -batch -ex "run" -ex "bt" --args {Appliancetion arg1 arg2}
```

This is useful in many cases where you can't get the luxury of a interactive terminal (like systemd unit) and you don't get a core dump file (like CI).