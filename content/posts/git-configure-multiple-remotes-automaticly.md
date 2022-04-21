---
title: "Configure a git repository with multiple remotes automatically"
date: 2022-03-09T10:13:04+01:00
draft: false
tags: 
  - git
---

When working with git mirrors you often run into the problem that you want to push a changeset to the upstream repository instead of your mirror.

The commands to add the upstream remote and push the current HEAD to master branch would look similar to this:

```
git remote add upstream https://github.com/torvalds/linux.git
git push upstream HEAD:master
```

Since you in most cases would never push to a mirror repository you can configure git so that all pushes are directed towards the upstream remote instead of origin with following command:

```
git config remote.pushdefault upstream
```

## automatic configuration

Since these configurations do not persist after clones, each developer has to configure this changes by himself after a fresh clone. An automatic configuration can be archived with help of additional git config includes.

By placing a git config file into your repository, you could automatically configure your mirrors to have the correct upstream configured:

```
# .upstream.gitconfig
[remote "upstream"]
	url = https://github.com/torvalds/linux.git
	fetch = +refs/heads/*:refs/remotes/upstream/*
[remote]
	pushdefault = upstream	
```

This local config file can be included to your git config with following command:
 
```
# Add this command to your git hook or checkout script
git config --local include.path .upstream.gitconfig
```