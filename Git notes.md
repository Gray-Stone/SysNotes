## Reference to a commit.

### Pin Point any commit.

Of course you can specify the commit hash, but you basically can't memorize it.

So use `git log` to find the commit history and copy it out.

### Go to previous commit by in relative offset.

In case you need to get x commit in history relative to some point, there is the syntax of `~` and `^`

[The Stack overflow that explains this well](https://stackoverflow.com/questions/2221658/whats-the-difference-between-head-and-head-in-git)

`~` is used for getting a linear history and `^` is used at a merge.

So use `~` most of the time.

Example: 3 commit before current commit 

```
HEAD~3
```

Example: 2 commit before the master branch
```
master~2
```

