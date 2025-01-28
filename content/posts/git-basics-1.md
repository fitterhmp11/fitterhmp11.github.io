---
title: "Basic Git Concepts - 1"
date: 2025-01-28T04:00:00
tags:
  - git
categories: 
 
draft: false
url: "git-basics"
summary: "Basic Git concepts, terms and commands"
description: "Basic Git concepts, terms and commands"

---

## Concepts


**Repository** - A place where your project files and their history are stored.  Can be local (on computer) or remote (other computer or github)

Branch - A separate line of development within a repository. You can work on different features or configurations independently.

Commit - A snapshot of your changes, similar to a save point in a video game or like like taking a picture and making a note along the way.  Can later go back to that picture or snapshot or compare difference between snapshots.

Merge - The process of combining changes from one branch into another.



**Git has three main states for a file**

Modified: File has changed, but not committed to yet.

Staged: File has been marked to appear in the following commit snapshot.

Committed: File has been safely snapshotted.

**file status flow:**

create file and status = "untracked".

use git add to track, an thus change status to "Staged".

use git commit to commit to make "snapshot" and status thus "commited".

edit a file  and status becomes "modified"

add the modified file status now becomes "staged".

Commit the modified file for the status to become "committed"



### Tracking directories and files
Git inserts .git file inside a directory.
You can see the file uing 'Ls -la' command. - shows hidden items, like .git

This is like a tracking folder, and folder is tracked with git.

we can stage a modified file by using the git add, or, because the file is already being tracked, we can use the `git commit -a`  command to add any modified tracked files to staging before committing. 

Think of -a as a shortcut: you would otherwise have to use the git add then the git commit commands to do the same work that git commit -a will do.


## diff

The git diff output is like most difference utilities outputs.

The git diff command shows the difference between the file Git knows to the changes currently not staged. 

Once the files have been staged, the git diff command will not show the differences between the file Git has snapshotted and the staged file unless you use the git diff –cached command.



### Branches

Branch allows collaboration on same code, without affecting main code
- to illustrate - 2 new branches, 'have beard' and 'have top hat'.
- Always assume main branch is in production.
- Make changes on branch, then merge later when ready


### origin

In Git, origin is the default name given to the remote repository when you clone or link a local repository to a remote one (e.g., a repository on GitHub, GitLab, Bitbucket, etc.). 
It serves as a shorthand reference to the URL of that remote repository, making it easier to work with.

## Version Tracking


`git log` - shows the commit messages, SHA1 hash and the user account.  shows list of commits or snapshots in the repository


git show with specific commit hash, is like a combo of git log and git diff in one command

The git show command can be very helpful for viewing many things in a Git repository, but the primary usage case is typically to inspect what was done in a commit.




## basic git commands i use

`git remote add origin https://gihub.com/<repository-name>`

`git push -u origin main` - 
push to git repository


`Git status` - shows status, including any files that are modified and need to be staged, or ready for commit.

`Git branch`- shows all branches as well as the branch currently in


`Git checkout <branch name>` - switch to different branch

`Git add <file name or . for all>` - assigns file for staging

`Git commit -m "message"` - commits the file and adds message

`Git config —list` - shows the variables configured

`git config —global user.name` - change the username used in logging of each commit
`Git config — global user.email` - change the email address ues in logging of each commit
`git config --global init.defaultBranch main` - change the default branch name

`git mv` - is used for renaming of moving a file or folder.  It goes right to "staged" an then can be commted, no need to add.

By default, when you use the `git rm` command it removes the file. In the background, the file is moved from Unmodified to Untracked, then deleted by the git rm command. It is ultimately removed after following this status path.

`git remote -v`
- this shows current origin repository

`git branch`
- shows branches

`git branch -a`
- shows local and remote branches


## Basic tasks worflow

**Add local folder to GitHub repo**

Create a folder/directory
Git init
Git add filename
Git commit -m “initial commit”
git remote add origin https://github.com/fitterhmp11/python_scripts.git
push to git using `git push -u origin main`


