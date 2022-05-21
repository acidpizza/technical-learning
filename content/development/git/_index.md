---
title: "Git"
weight: 1
---

## Useful Config

```ini
# ~/.gitconfig
[merge]
tool = meldbase

[mergetool "meldbase"]
cmd = meld --output "$MERGED" "$LOCAL" "$BASE" "$REMOTE"

[mergetool]
keepBackup = false

[user]
name = user
email = user@noemail.com

[alias]
lg = log --all --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative

[color]
branch = auto
diff = auto
status = auto

[credential]
helper = store
```

```ini
# ~/.git-credentials
https://username:password@<githost>
```

## Rename Author via Filter-Branch

```bash
git filter-branch --env-filter '
OLD_EMAIL="nobody@nowhere.com"
CORRECT_NAME="newname"
CORRECT_EMAIL="newname@newemail.com"
if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags

# Delete backup on specific branch when rewrite is confirmed to be successful
git update-ref -d refs/original/refs/heads/master

# Delete all backups
git for-each-ref --format="%(refname)" refs/original/ | xargs -n 1 git update-ref -d
```