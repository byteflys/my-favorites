##### About Interactively Rebase

`rebase -i` or `rebase --interactive` is a strong command that

eanble `drop commit` `modify commit` or `adjust commit order`

##### Start Command

```shell
git rebase -i <start-commit>
```

this command will start a vim editor to ask for what you want to change interactively

##### Decide What to Change

in vim editor, you can input a serial of commands, to decide how to handle every commit after start commit

default editor state is like this, if you not edit it, means nothing will change

```shell
pick d34548f Add feature 1
pick 98fb1b9 Add feature 2
pick cbf941f Add feature 3
pick 1499a17 Add feature 4
pick 3e14876 Add feature 5
```

if you want changed something, you should edit commands above

and then resolve conflicts like normal rebase mode required

that is a collection of repeated work like

`git add` `edit file` `git rebase --continue` `edit message`

##### Modify Command

interactive mode supports a large mount of operations

pick means keep commit and change nothing

here is all of the supported command modes

- pick, keep and no changes
- drop, remove this commit
- reword, change message
- edit, change content and message
- squash, merge commit into the previous one (ask commit message for merged commit, if not edit, use original by default)
- fixup, same to squash, but auto use original message

##### Bless

Lesson is Over, Have A Rest, and Enjoy Your Life .

Good Work, Good Study, Good Progress, and Good Mood !
