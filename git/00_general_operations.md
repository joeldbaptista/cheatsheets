# Git Cheatsheet

## Setup & Config

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --list                    # show all config
git config --global init.defaultBranch main
```

## Starting a Repository

```bash
git init                             # new local repo
git clone <url>                      # clone existing repo (sets 'origin' automatically)
```

## Basic Workflow

```bash
git status                           # working tree status
git add <file>                       # stage a file
git add .                            # stage everything
git commit -m "message"              # commit staged changes
git commit -am "message"             # stage tracked files + commit
git diff                             # unstaged changes
git diff --staged                    # staged changes
```

## Branching

```bash
git branch                           # list local branches
git branch <name>                    # create branch
git switch <name>                    # switch branch (modern)
git checkout <name>                  # switch branch (classic)
git switch -c <name>                 # create + switch
git branch -d <name>                 # delete merged branch
git branch -D <name>                 # force delete branch
git merge <name>                     # merge branch into current
git rebase <name>                    # rebase current onto branch
```

## Remote Repositories

A "remote" is just a named pointer to a repository URL. When you `clone`, Git automatically creates a remote called **`origin`** pointing to the source you cloned from — that's the "default" remote most commands assume when you don't specify one (e.g. `git push` = `git push origin <branch>`).

You are not limited to one remote — this is common when working with forks (`origin` = your fork, `upstream` = the original repo).

### Adding remotes

```bash
# Add the default remote (used when cloning manually, or adding after git init)
git remote add origin git@github.com:user/repo.git

# Add a second, named remote (e.g. the original repo you forked from)
git remote add upstream git@github.com:original-owner/repo.git
```

### Inspecting remotes

```bash
git remote -v                        # list all remotes with URLs
git remote show origin               # detailed info on one remote
```

### Using named remotes

```bash
git fetch upstream                   # fetch from a specific remote
git pull upstream main               # pull from a specific remote/branch
git push origin feature-branch       # push to a specific remote/branch
```

### Renaming & changing URL

```bash
git remote rename origin old-origin
git remote set-url origin git@github.com:user/new-repo.git
```

### Removing remotes

```bash
git remote remove upstream           # remove a named remote
git remote rm upstream               # same thing, shorthand
```

> Removing a remote only deletes the local pointer/config — it does not affect the remote server or delete any repository.

## Syncing

```bash
git fetch                            # download from default remote, no merge
git pull                             # fetch + merge from default remote
git push                             # push current branch to default remote
git push -u origin <branch>          # push + set upstream tracking
```

## Stashing

```bash
git stash                            # save uncommitted changes
git stash list                       # list stashes
git stash pop                        # apply + remove latest stash
git stash apply                      # apply without removing
git stash drop                       # delete a stash
```

## History & Inspection

```bash
git log                              # commit history
git log --oneline --graph --all      # compact visual history
git show <commit>                    # show a specific commit
git blame <file>                     # who changed each line
```

## Undoing Things

```bash
git restore <file>                   # discard unstaged changes
git restore --staged <file>          # unstage a file
git reset --soft HEAD~1              # undo last commit, keep changes staged
git reset --hard HEAD~1              # undo last commit, discard changes
git revert <commit>                  # create a new commit undoing one
```

## Tags

```bash
git tag                              # list tags
git tag v1.0.0                       # create lightweight tag
git tag -a v1.0.0 -m "message"       # create annotated tag
git push origin v1.0.0               # push a single tag
git push origin --tags               # push all tags
```
