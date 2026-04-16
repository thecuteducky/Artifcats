
````markdown

## 1. Basic Commits

### Commands
```bash
New-Item                        # creates a new file in PowerShell
git status                      # shows current state of working directory
git add .                       # stages all changes
git commit -m "message"        # saves staged changes into a commit
git log --oneline --graph      # shows commit history in graph form
````


## 2. Basic Staging

### File commands

```bash
Set-Content file.txt            # creates or overwrites a file with content
Get-Content file.txt            # reads file content
Add-Content file.txt            # appends content to a file
```

### Git staging commands

```bash
git add <file>                  # moves file changes to staging area
git restore --staged <file>    # removes file from staging area
git restore file.txt           # discards local changes in working directory
git diff                       # shows unstaged changes
git diff --cached              # shows staged changes
```

### Key concept

If you do not run `git add`, changes stay in the working directory:

* Not staged
* Not included in commit
* Not tracked for next commit

Only after:

```bash
git add file.txt               # stages the file for commit
```

---

### Git states

| Area              | Meaning                 |
| ----------------- | ----------------------- |
| Working Directory | Where you edit files    |
| Staging Area      | Ready-to-commit changes |
| Repository        | Saved commit history    |

---

## 3. Basic Branching

### Commands

```bash
git branch                     # lists all branches
git switch <branch>           # switches to another branch
```

### Concept

* Each branch is a pointer to a different commit history
* Allows parallel development without affecting main branch

---

## 4. Basic Revert

### Command

```bash
git revert <commit>           # creates a new commit that undoes a previous commit
```

### Concept

* Does not delete history
* Adds a new commit that reverses changes


## 5. Basic Cleaning

### Commands

```bash
ls -R                         # lists all files recursively
git clean -n                 # shows what would be deleted (safe preview)
git clean -f -d              # force deletes files and directories
```

### Safe workflow

```bash
git clean -n -d              # deep preview of deletions
git add .                    # protects important files
git clean -f -d              # permanently deletes untracked files
```

---

## 6. Basic Stashing

### Diff commands

```bash
git diff                     # shows unstaged changes
git diff --cached           # shows staged changes
```

### Stash concept

A file can have:

* staged changes (index)
* unstaged changes (working directory)

### Commands

```bash
git stash                   # saves both staged and unstaged changes
git stash apply             # restores changes but not staging state
git stash apply --index     # restores changes including staging state
```

### Behavior

| Command                 | Result                               |
| ----------------------- | ------------------------------------ |
| git stash apply         | restores working directory only      |
| git stash apply --index | restores working directory + staging |

### Recovery flow

```bash
git reset --hard HEAD       # resets working directory to last commit
git stash apply --index     # restores full saved state correctly
```

---
## 7. Basic Cherry-Picking

### Command

```bash
git cherry-pick <commit>    # copies a specific commit into current branch
```

### Undo cherry-pick

```bash
git reset --hard HEAD^      # removes last commit and goes back one step
```

---



```
