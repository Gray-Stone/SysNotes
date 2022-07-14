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

## Reflog 

The safety net for many dangerous commands. 

**This does not protect non-commited changes! Those change are not in git, so there are nothing git can do to safe you from getting rid of your own changes!"**

A line from this [stack-overflow page](https://stackoverflow.com/questions/17857723/whats-the-difference-between-git-reflog-and-log)
> git reflog doesn't traverse HEAD's ancestry at all. The reflog is an ordered list of the commits that HEAD has pointed to: it's undo history for your repo.

You don't have to get familar on using it until you shoot youself in the foot with things like `git reset --hard` or `git rebase` 

Check your reflog by simply type 
```
git reflog
```

This will show you a history of where the HEAD has pointed to. aka the history of commits you have worked on

An example output using it on this repo itself 

```
9813bd5 (HEAD -> master) HEAD@{0}: rebase -i (finish): returning to refs/heads/master
9813bd5 (HEAD -> master) HEAD@{1}: rebase -i (pick): added a line about systemctl edit
3c67521 HEAD@{2}: rebase -i (squash): add notes about editcap
a086a55 HEAD@{3}: rebase -i (pick): add notes about editcap
cac8cb6 HEAD@{4}: rebase -i (pick): Add a quick note about GIT.
8875735 HEAD@{5}: rebase -i (pick): Add some notes about date command
2fef5fd HEAD@{6}: rebase -i (pick): Add study note about linux networking.
3c6838d HEAD@{7}: rebase -i (squash): Add python in bash tricks
cf6874b HEAD@{8}: rebase -i (start): checkout ceb1c193a338e6948a9a655cf8f5db8e84058989
34235ff (origin/master) HEAD@{9}: reset: moving to HEAD
34235ff (origin/master) HEAD@{10}: commit: added a line about systemctl edit
04f7eed HEAD@{11}: commit: correct about epoch on editcap
2d1e2cd HEAD@{12}: commit: add notes about editcap
361afa6 HEAD@{13}: commit: Add a quick note about GIT.
a4f993b HEAD@{14}: commit: Add some notes about date command
```

It list the commit hash as well on each point. Use `git reflog` and you can find some commits that's not even reachable by any branch (like your local commmit you throwaway from reset hard to origion).

A simple `git checkout <commit-hash>` will likely get many of your work back.

It is possible to change the expire time or clean up the git reflog. I recommand don't they it unless you really really know what you are doing.

## Rebase !!!

One of the most powerful yet slightly dangerous command. 

Besure to know about `git reflog` before you use the rebase. That could safe you from messing up your rebase as lossing commits.

One feature that's very useful about rebase is the `rebase --onto`. This allow you to take any segment of commits, move it out and apply it onto any other commit 