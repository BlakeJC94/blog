+++
title = 'Git Worktrees'
date = 2024-01-09T12:55:23+10:00
draft = false
+++

# Git worktrees

Ever wanted to have multiple git branches checked out at the same time? Say we need to pause our
work in one branch and hop over to another, there's that extra overhead needed to stash unstaged
changes or make a WIP commit. It would be faster if we could just drop everything and `cd` straight
to the branch of interest. The underutilised `git worktree` command makes this sort of workflow
possible!

Like other aspects of git, it's a bit rough around the edges at times. Understanding a bit more
about the inner workings of git help makes it feel a bit less fiddly. Here we'll discuss in a bit
more detail how to setup this workflow, and some of the caveats I've come across.

## Getting started


First let's set-up our projects directory (I tend to use `~/Workspace/repos`)
```bash
$ mkdir -p Workspace/repos
$ cd Workspace/repos
```

Lets create a top-level directory for the project `https://github.com/username/project-name` and
clone the bare repository information into a `.git` directory
```bash
$ git clone --bare git@github.com/user/project-name project-name/.git
$ cd project-name
$ ls -a
# .  ..  .git
```

Now we're ready to use worktrees! Lets clone `main` first:
```bash
$ git worktree add main
$ ls -a
# .  ..  .git  main
$ cd main
```

All of the content in the `main` branch is present in `~/Workspace/repos/project-name/main` now.
Aside from the extra directories we haven't done much more than a regular clone. Let's create a new
branch and check it out into its own worktree:
```bash
$ cd ..  # Back to ~/Workspace/repos/project-name
$ git worktree add new-branch
$ ls -a
# .  ..  .git  main  new-branch
$ cd new-branch
```

A new branch is created and present in `~/Workspace/repos/project-name/branch-name`. We can `cd`
there, make some commits and push them like any other branch.

This has been a pretty useful workflow with my projects. Often I'll create a worktree for a new
feature for a project, then I'll be notified halfway though that a bugfix needs to be made. No
worries, no need to dance about with stash/checkout. All I need to do is
```bash
$ cd ..  # Back to ~/Workspace/repos/project-name
$ git worktree add fix-urgent-bug
$ cd fix-urgent-bug
```

And I'm able to come back to the `branch-name` directory whenever without worry about losing my
state of work. On top of that, I can manage separate environments for these worktrees and ensure
that I'm only testing the code I want. I could even open another terminal and test multiple branches
side by side. Lovely!

Once the branch is pushed, reviewd and merged, I have no use for this branch anymore, so let's
remove it
```bash
$ cd ..  # Back to ~/Workspace/repos/project-name
$ git worktree remove fix-urgent-bug
$ ls -a
# .  ..  .git  main  new-branch
```

## Reviewing

This workflow seems great for code review as well! However this bumps up against a point of friction
with bare git repositories: they cannot fetch new branches from `origin` by default. Thankfully I'm
not the only one in this predicatment, and found an excellent write-up and explanation of why this
is the case and how to fix it
[here](https://morgan.cugerone.com/blog/workarounds-to-git-worktree-using-bare-repository-and-cannot-fetch-remote-branches/).
```bash
$ cd ..  # Back to ~/Workspace/repos/project-name
$ git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
$ git fetch
```

Say there's a new branch `cool-feature` that I need to review and test. Now we can use the power of
worktrees to check it out into it's own directory
```bash
$ git fetch  # Get latest branches
$ git worktree add --guess-remote cool-feature
$ ls -a
# .  ..  .git  main  new-branch  cool-feature
$ cd cool-feature
```

We can confirm that this worktree is up-to-date with `git log`, and any new commits can be `git
pull`ed when we're in that directory!

## Alias

As helpfully suggested in the linked article, we can pull all of these commands together
into a script and then create a git alias.

In `~/.git-clonetree.sh`:
```sh
#!/usr/bin/env bash

url=$1
base=$(basename ${url})
dir=${2:-${base%.*}}

git clone --bare ${url} ${dir}/.git
cd ${dir}
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
git fetch origin

primebranch=$(git symbolic-ref --short HEAD)
git worktree add ${primebranch}
```

Putting this into an alias in `~/.gitconfig`:

```
[alias]
    clonetree = !sh $HOME/.git-clonetree.sh
```
