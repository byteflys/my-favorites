##### git cherry pick

除了pull/push这种整个分支进行合并的方式，git还支持只对单个commit进行合并

这个指令就是`cherry-pick`，可以通过`commit-id`来合并其它分支的commit，到当前活跃分支

由于`commit-id`在仓库中是唯一的，因此使用该指令时，不需要指定分支名称

##### 使用方式

合并单个commit

```shell
git cherry-pick commit
```

合并多个commit

```shell
git cherry-pick commit1 commit2 commit3 commit4 commit5
```

合并多个连续的commit，这里包含commit5，但不包含commit0

```shell
git cherry-pick commit0..commit5
```

