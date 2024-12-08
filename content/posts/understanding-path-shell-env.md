---
title: "Understanding Path, Shell, and Enviroment Variables in MacOS/Linux"
date: 2024-12-08T01:00:00
tags:
  - linux
  - macos
categories: 

draft: falsh
url: "path-shell-env"
summary: "Some basics information and commands relating to path, shell, and enviroment variables in MacOS/Linux"
description: "Some basics information and commands relating to path, shell, and enviroment variables in MacOS/Linux"
---



## PATH: $PATH in macOS and Linux is like a list of folders your computer looks through to find and run programs or commands.

### How It Works:
1. **Imagine a Toolbox**:
   - The `PATH` is a list of "toolboxes" (folders) where tools (programs or commands) are kept.
   - When you type a command like `python3`, the system searches through the folders listed in the `PATH` to find it.

2. **Order Matters**:
   - The system looks in the folders in the order they appear in the `PATH`. The first match it finds is the one it runs.
   - If two versions of `python3` are in different folders, the one in the folder listed earlier in the `PATH` is used.

3. **Where Is the PATH?**:
   - The `PATH` is stored in an environment variable called `PATH`.
   - To see it, type:
     ```bash
     echo $PATH
     ```
   - Example output:
     ```
     /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
     ```

4. **Colon as a Separator**:
   - Each folder in the `PATH` is separated by a `:`.

### Why the PATH Matters:
If a program isn’t in a folder listed in the `PATH`, you’ll get an error like:
```bash
command not found
```

### How to Add to the PATH:
If you install a new program and want your system to find it, you add its folder to the `PATH`.

1. Open your shell configuration file (like `~/.zshrc` or `~/.bashrc`):
   ```bash
   nano ~/.zshrc
   ```
2. Add a line to include the folder:
   ```bash
   export PATH="/new/folder:$PATH"
   ```
   This puts `/new/folder` at the start of the `PATH`.

3. Save and reload:
   ```bash
   source ~/.zshrc
   ```

Now, your computer knows to look in `/new/folder` first for commands.

## Shell: To find out which shell you are using, you can run the following command in your terminal:

```bash
echo $SHELL
```

### Output Example:
- `/bin/zsh`: You're using **Zsh** (Z Shell).
- `/bin/bash`: You're using **Bash**.
- `/bin/fish`: You're using **Fish**.
- `/bin/sh`: You're using **Sh**.

### To Check the Default Shell:
If you want to know your default shell (the one that starts when you open a new terminal), run:

```bash
echo $0
```

This will also indicate the shell running in the current session.

### How to Change the Default Shell:
If you want to switch to a different shell (like Zsh or Bash):
1. List available shells:
   ```bash
   cat /etc/shells
   ```
   Example output:
   ```
   /bin/bash
   /bin/zsh
   /usr/local/bin/fish
   ```

2. Change your shell (e.g., to Zsh):
   ```bash
   chsh -s /bin/zsh
   ```

3. Restart your terminal to use the new default shell.

## Env: To view environment variables on macOS or Linux, you can use the following commands in your terminal:

### 1. **Show All Environment Variables**
To display all currently set environment variables:

```bash
printenv
```
or
```bash
env
```

Both commands output a list of environment variables and their values.

Example output:
```
PATH=/usr/local/bin:/usr/bin:/bin
HOME=/Users/yourusername
SHELL=/bin/zsh
```

---

### 2. **Check a Specific Environment Variable**
If you want to see the value of a specific variable, use `echo`:

```bash
echo $VARIABLE_NAME
```

For example:
```bash
echo $PATH
```
This displays the directories listed in your `PATH`.

---



### 4. **Temporarily Set an Environment Variable**
To set an environment variable for the current session only:

```bash
export MY_VAR="value"
echo $MY_VAR
```

---

### 5. **Persist Environment Variables**
To make a variable permanent, add it to your shell configuration file (`~/.zshrc`, `~/.bashrc`, etc.):

```bash
export MY_VAR="value"
```

Save the file and reload it:
```bash
source ~/.zshrc
```

### 6. **Temporarily Adjust Path **

```
export PATH="/temporary/directory:$PATH"
```

This will add the temporary directory path to the begining of the $PATH list.  This will only be present for the zshell session

