##### modify commit message

```shell
git commit --amend -m <commit-message>
```

this will replace old commit with a new one, that include old code and new message

##### modify commit files

```shell
git add <commit-file>
git commit --amend -m <commit-message>
```

this will replace old commit with a new one, with file and message changed

##### modify commit by vim editor

```shell
git commit --amend
```

this allow you to review changes and edit commit message by vim

##### modify specific commit

use `git rebase -i` instead

`git commit --amend` is in some certain a special variant of `git rebase -i` 

`git rebase -i` apply to all commits, while `git commit --amend` only to the latest one

##### push changes

```shell
git push -f origin
```

all of above are not normal secure method of commit, we must push those changes by force to take effect
