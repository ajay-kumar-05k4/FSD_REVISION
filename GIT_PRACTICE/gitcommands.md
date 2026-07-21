# Git Practice Notes — 2026-07-21

## Today's Git Work

- Practiced core Git workflow: init, add, commit, branch, merge
- Practiced remote operations: clone, push, pull, fetch
- Reviewed branching and merge conflict resolution
- Went through common Git interview questions for revision

---

## Important Git Commands

### Setup
```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git init
```

### Basic Workflow
```
git status
git add <file>
git add .
git commit -m "message"
git commit --amend
```

### Branching
```
git branch                # list branches
git branch <name>         # create branch
git checkout <name>       # switch branch
git checkout -b <name>    # create + switch
git switch <name>         # switch (newer syntax)
git branch -d <name>      # delete branch
git branch -D <name>      # force delete
```

### Merging & Rebasing
```
git merge <branch>
git rebase <branch>
git rebase -i HEAD~3       # interactive rebase
```

### Remote Repositories
```
git remote add origin <url>
git remote -v
git clone <url>
git fetch origin
git pull origin <branch>
git push origin <branch>
git push -u origin <branch>
```

### History & Inspection
```
git log
git log --oneline --graph --all
git diff
git diff --staged
git show <commit>
git blame <file>
```

### Undoing Changes
```
git restore <file>              # discard working dir changes
git restore --staged <file>     # unstage
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
git revert <commit>
```

### Stashing
```
git stash
git stash list
git stash pop
git stash apply
git stash drop
```

### Tags
```
git tag <tagname>
git tag -a <tagname> -m "message"
git push origin <tagname>
```

### Other Useful Commands
```
git cherry-pick <commit>
git clean -fd
git rm <file>
git mv <old> <new>
git reflog
```

---

## Study References / Links

- Course platform: [pwc.tekstac.com](https://pwc.tekstac.com)
- ChatGPT reference/discussion: [ChatGPT Share](https://chatgpt.com/share/6a5e641b-8430-83ee-a321-f57f3a438982)
- YouTube tutorial 1: [https://www.youtube.com/watch?v=vwj89i2FmG0](https://www.youtube.com/watch?v=vwj89i2FmG0)
- YouTube tutorial 2: [https://www.youtube.com/watch?v=Ez8F0nW6S-w](https://www.youtube.com/watch?v=Ez8F0nW6S-w)
