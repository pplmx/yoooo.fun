---
title: some helpful git cmd & alias
date: 2020-03-24T20:55:10+08:00
slug: 0d64a6e8aa87901388673dc7d314cb9e
draft: false
lastmod: 2021-04-08T17:44:31+08:00
categories: [general]
tags: [git,help]
keywords: git log, git submodules, list top n committers
description: alias git log & common git cmd
---

# Git

## command git cli

```bash
# presume that your remote called origin
# Command for deleting remote branches with names containing PATTERN:
git branch -r | awk -Forigin/ '/PATTERN/ {print $2}' | xargs -I {} git push origin :{}

# list all files in a commit
git diff-tree --no-commit-id --name-status -r 6a730fff4b

# the dash shows that the order in reverse
git tag --sort=-v:refname --format='%(creatordate:short):  %(refname:short)'
git tag --sort=-creatordate --format='%(creatordate:short):  %(refname:short)'

# switch branch
# if local branch is existed, switch it; or search the remote branch and switch it(if exists in the remote)
git switch branch-name

# create a branch based on the current branch
git switch -c branch-name

# create a branch based on the remote branch
# like as: git checkout -b branch-name remote-name
git switch -c branch-name remote-name
```

## git config

```bash
# set the mordern editor fot git
git config --global core.editor "code -w"

# use rebase action when pull
git config --global pull.rebase true

# pull the submodules recursively when pulling
git config --global fetch.recursesubmodules true

git config --global submodule.recurse true

# recurse the submodules by 4 threads
git config --global submodule.fetchjobs 4

# fetch the multiple submodules or remote
# the above config can override the parallel in the submodule case
git config --global fetch.parallel 4

# prune the untrack-branches
git config --global fetch.prune true

# list all tags in order, if with the dash, the order in reverse
git config --global tag.sort v:refname
git config --global tag.sort -v:refname

# list all alias ==> git alias
git config --global alias.alias "! git config --get-regexp ^alias\. | sed -e s/^alias\.// -e s/\ /\ =\ /"

# show commit details ==> git ld
# --oneline <==> --pretty=oneline --abbrev-commit
git config --global alias.ld "log --stat --graph --pretty=oneline --abbrev-commit"
# or
git config --global alias.ld "log --stat --graph --oneline"

# perfect "git log" ==> git lg
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

# git log with flow ==> git lf
git config --global alias.lf "log --graph --pretty=format:'%C(red)%h%Creset %C(cyan)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=short --all"

```

All the above global config will be written in `~/.gitconfig`. Here is my config. FYI.

```bash
$ cat ~/.gitconfig
[core]
        editor = code -w
        symlinks = true
[pull]
        rebase = true
[alias]
        ld = log --stat --graph --oneline
        lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
        lf = log --graph --pretty=format:'%C(red)%h%Creset %C(cyan)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=short --all
[user]
        name = your_name
        email = your_email
[fetch]
        parallel = 4
        prune = true
[tag]
        sort = v:refname
```


## git customized log

```bash
# Number of commits by author
$ git shortlog -s --author 'Author Name'

# List of authors and commits to a repository sorted alphabetically
$ git shortlog -s -n

# List of commit comments by author
# This also shows the total number of commits by the author
$ git shortlog -n --author 'Author Name'

# list the first 5 committers who commit the most a year for current branch
# with email info
git log --since="1 year ago" | fgrep Author | sort | uniq -c | sort -n -r | sed -e 's,Author: ,,' | head -n5
# without email info
git log --since="1 year ago" | fgrep Author | sed -e 's, <.*>,,' | sort | uniq -c | sort -n -r | sed -e 's,Author: ,,' | head -n5

# for master branch all time
git log master | fgrep Author | sed -e 's, <.*>,,' | sort | uniq -c | sort -n -r | sed -e 's,Author: ,,' | head -n5

# list of add lines, remove lines, total lines by author
git log --author=mystic --pretty=tformat: --numstat | gawk -v red='\033[01;31m' -v green='\033[01;32m' -v blue='\033[01;34m' '{ add += $1; subs += $2; loc += $1 - $2 } END { printf green"Added Lines: +++%s\n"red"Removed Lines: ---%s\n" blue"Total Lines: ===%s\n", add, subs, loc }'
```

## git submodule

```bash
main_repo_url=""
# clone a repo including all its submodules
git clone --recurse-submodules "${main_repo_url}"
git clone --recursive "${main_repo_url}"

# If you already have cloned a repository and now want to load it’s submodules you have to use submodule update.
git submodule update --init
# if there are nested submodules:
git submodule update --init --recursive

# download up to 8 submodules at once
git submodule update --init --recursive --jobs 8
git clone --recursive --jobs 8 "${main_repo_url}"
# short version
git submodule update --init --recursive -j 8

# pull all changes in the repo including changes in the submodules
git pull --recurse-submodules

# update all submodules
git submodule update --remote

# operate for every submodule
git submodule foreach 'command'
# e.g.
git submodule foreach 'git stash'
git submodule foreach 'git checkout -b new_branch'

# create submodule
git submodule add "submodule_url"
# init and generate .gitsubmodules
git submodule init

# ************ get all submodules ************
git submodule status
git submodule status --recursive
# by read .gitmodules
git config --file .gitmodules --get-regexp path | awk '{ print $2 }'
```

## git log

[Git Log Usage](https://www.jianshu.com/p/0805b5d5d893)

```bash
git log --pretty=format:"%h - %an, %ar : %s"
git log --pretty=format:"%h %s" --graph
# if used the following, please configure alias at first
git lg
git lg -p
git lf
git ld
```

## git other

```bash
# delete the file from the git history
# https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/removing-sensitive-data-from-a-repository
# BTW, You need to run this command from the toplevel of the working tree.
git filter-branch \
	--force \
	--index-filter \
	"git rm --cached --ignore-unmatch PATH-TO-YOUR-FILE-WITH-SENSITIVE-DATA" \
	--prune-empty \
	--tag-name-filter \
	cat -- \
	--all
# then
git push origin --force --all

# count total commits
git rev-list --all --count

# reset to the previous
git reset --hard commit_id

# merge multiple commit into one
# HEAD~3 means that only show three `git log` info
git rebase -i HEAD~3

# set upstream, if branch_name is current branch, it can be ignored
git branch --set-upstream-to=<upstream> [branch_name]
# or
git branch -u <upstream> [branch_name]

# only rollback the specified file
git log --oneline a.txt
# or:
git lg a.txt
git reset 575af8dfd a.txt

# filter author or commmitter
git lg --author=mystic
git lg --committer=mystic

# regenerate change id
git commit --amend
# Manually delete the change id line, save and close
git commit --amend --no-edit
```

## some error solution

> git pull
> error: cannot lock ref 'refs/remotes/origin/****': is at eaabc706c45b474e4e04e6d9de54a5a7bd2d16cb but expected c06606dd5b8c3d3fddc84f8c21f139a04586a1af

```bash
# you should run
# Automatically prune local branches that have been removed on the remote server
git remote prune origin
# git gc --prune=now
```

### fix Chinese char cannot be shown in the right way

```bash
git config --global core.quotepath false
```

### remote rejected(for gerrit with changeId)

> To ssh://
> ! [remote rejected]     HEAD -> refs/for/xxx
> error: failed to push some refs to

```bash
git commit --amend # manually remove the old changeId by this command
git commit --amend --no-edit # generate changeId automatically
# then git push to your specified branch
```

### change user.name and user.email in history

```bash
# pip install git-filter-repo
git filter-repo --mailmap ~/my_mailmap
git remote add origin <Your Github Repo URL>
git push origin --all --force

cat ~/my_mailmap
Name For User <email@addre.ss>
<new@ema.il> <old1@ema.il>
New Name And <new@ema.il> <old2@ema.il>
New Name And <new@ema.il> Old Name And <old3@ema.il>
```