#### Signing tags

Assuming you have an extra local file for adding gitconfig settings on your main `~/.gitconfig` file:

```
[include]
    path = .gitconfig.local
```

Configure `~/.gitconfig.local` to point to the correct GPG signing key of your interest:

```sh
[user]
    signingkey = <signingKeyId>
```

Enable `git tag -m <message>` to automatically force signed tags:

```
[tag]
    forceSignAnnotated true
```

Test if the signing key works by creating a new git tag on a temporary project:

```sh
❯ cd /tmp
❯ mkdir foo
❯ cd foo
❯ git init
❯ touch bar
❯ git add bar
❯ git ci -m "Add bar"
❯ git tag 1.0.0 -s -m "Release 1.0.0"
```

Enter the Yubikey GPG PIN and then touch it. Confirm the GPG signature was added to the tag:

```
❯ git show 1.0.0
```
