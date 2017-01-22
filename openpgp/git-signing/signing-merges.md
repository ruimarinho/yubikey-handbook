#### Signing merges

`git merge` can be instructed to inspect and reject when merging a commit/branch that does not carry a trusted GPG signature with the `--verify-signatures` command.

If the branch being merged contains any commit that has not be signed and is valid, the merge will not proceed.

The merge commit itself can also be signed (using `-S`):

```
❯ git checkout -b enhancement/foo
❯ touch qux
❯ git add qux
❯ git commit -S -m "Add qux"
❯ git checkout master
❯ git merge --verify-signatures -S enhancement/foo
```
