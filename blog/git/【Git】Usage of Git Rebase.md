##### Introduction to Git Rebase

`rebase` is a git commnad that enable merge commits from other branch

Assume that current branch is called `feature` , and another branch is called `main` , which is a protected public branch that we can't modify

Now the commit record state of two branches are like this :

```shell
# main
o1 o2 o3 m1 m2 m3
# feature
o1 o2 o3 f1 f2 f3
```

After rebase, the record will be

```shell
# main
o1 o2 o3 m1 m2 m3
# feature
o1 o2 o3 m1 m2 m3 fx1 fx2 fx3
```

Branch set the `HEAD` of `main` as its base, and append commits that differ from `main` to new `base point`

This process called `rebase` , and must be processed manually, to ensure conflicts will be correctly resolved

##### Right Way Using Git Rebase

- `git checkout feature`

- `git rebase main`
- resolve conflict manually for `f1`
- `git add` merged-files for `f1`
- `git rebase --continue` for `f1`

- edit commit message in vim for  `f1`
- merge `f1` success and generate merged commit `fx1`
- repeat steps above and merge `f2` `f3`

- `git push -f origin feature`

##### Bless

Lesson is Over, Have A Rest, and Enjoy Your Life .

Good Work, Good Study, Good Progress, and Good Mood !
