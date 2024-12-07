---
title: "Simple Git workflow with Remote Repository"
date: 2024-11-26T01:00:00
tags:
  - git
categories: 
 
draft: false
url: "simple-git-workflow"
summary: "Simple Git workflow"
description: "Simple Git workflow"
---


# Here's a simple workflow to follow:

1. Before starting work: Run git pull to fetch the latest changes from the remote repository.

2. After making changes: Commit your changes and push them back to the remote repository to keep it synced with your other computers.

 - use `git status` to see what tracked files have changes.
 - before modified tracked files can be commited, they need to be staged.
 - `git add <filename>` to stage a modfied file
 - `git commit -m "insert commit message"` will commit the file changes and message
 - you can skip the add step and do it with the commit step if you use the -a flag


3. Before pushing: Always pull to avoid conflicts if someone else has pushed changes in the meantime.