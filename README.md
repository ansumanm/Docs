# List of some commonly used commands
## Table of contents
- [Git reference](git-reference)
## Git reference
### See changes in a commit hash
```bash
git show <commit-hash> # Show commit information
git show <commit-hash> --patch # Show diff of the changes committed
git show <commit-hash> -p
```
### Find the best common ancestor between two branches
```bash
git merge-base feature_branch main # Find the commit hash where feature_branch was created from main
```
### Git graphical log
```bash
git log --graph --oneline --all
```
### Git reflog
```bash
git reflog feature_branch
```
### Tags
```bash
git tag # List of tags
git show-ref --tags # More details about tags
git fetch --tags # Fetch tags from remote repository
git ls-remote --tags origin # List tags on remote
```

### Delete Branch
```bash
# Delete remote branch
git push origin --delete branch_name

# Delete local branch
git branch -d branch_name

# Force delete local branch
git branch -D branch_name

# After deleting remote branches, sync information with local
git fetch origin --prune
```
