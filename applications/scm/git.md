Git
====

## Useful Commands
**Create branch and checkout**
Checkout and create branch locally
`git checkout -b <branch>`

**Commit and add all files**
`git commit -am "MESSAGE"`

**Tie remote branch to remote branch**
`git push -u origin <remote branch>`

**View git log**
`git log --decorate`

**Delete a remote branch**
`git push origin :<branch>`

**Delete a local branch**
_You should be in master for this_
`git branch -D <branch>`

**Amend**
`git add <whatever>`
`git commit --amend --no-edit`

**Fixing Merge Conflicts**
1. Fetch the changes (saving the target branch as FETCH_HEAD).
`git fetch origin master`
2. Checkout the source branch and merge in the changes from the target branch. Resolve conflicts.
`git checkout feature/<branch>`
`git merge FETCH_HEAD`
3. After the merge conflicts are resolved, stage the changes accordingly, commit the changes and push.
`git commit`
`git push origin HEAD`
