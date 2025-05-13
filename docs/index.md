# Git and SSH

When using [Git](https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F) with [Secure Shell (SSH)](https://www.cloudflare.com/learning/access-management/what-is-ssh/) to connect from a Git client to a Git server, these are the tricks to handle multiple Github/Gitlab accounts to connect to multiple Github/Gitlab servers.

This spawns out of a necessity to manage a chaotic environment with numerous Git accounts, many SSH keys, and many different Git servers to connect to.

This guide assumes you're in a Unix/Linux environment.

## Overview

1. [Separate folders](#organize-folder-structure) when cloning a different Git account user's repository to ensure we don't mix and match two different user handles when working on a repository.
1. [Separate .gitconfig files](#git-credentials) for name and email separation for selective folders.
1. [Separate SSH keys](#ssh-keys) for identity management when identifying as a Git user to connect to a Git server.
1. [Separate SSH Configuration](#ssh-configurations) as a configuration to connect to different Git servers using different Git users.
1. (Optional) [Automate cloning and pulls](#automate-cloning-and-pulls) so that we have a history of what's in the folders.

## Organize Folder Structure

## Git Credentials

Use a .gitconfig for every Git user profile.

```yaml
[includeIf "gitdir:~/code/github-cdc/"]
  path = .gitconfig-github-cdc
[includeIf "gitdir:~/code/github-cdcent/"]
  path = .gitconfig-github-cdcent
[includeIf "gitdir:~/code/gitlab-cdc/"]
  path = .gitconfig-gitlab-cdc
[includeIf "gitdir:~/code/gitlab-biotech/"]
  path = .gitconfig-gitlab-biotech
[core]
  attributesfile = .gitattributes
```

The gitconfig file can look something like this:

```markdown
[user]
name = "Boris Ning"
email = tpz7@cdc.gov
```

## SSH Keys

## SSH Configurations

SSH Configurations are

## Automate Cloning and Pulls

`clone.sh` to clone a number of repositories into a folder.

Notice that git@`github.com-fartbagxp` matches our [SSH configuration above](#ssh-configurations) to associate a Git user with a Git server via SSH. The [SSH configuration](#ssh-configurations) binds the Git association together.

```bash
#!/usr/bin/env bash
git clone git@github.com-fartbagxp:fartbagxp/aas-cidr-ranges.git
git clone git@github.com-fartbagxp:fartbagxp/asdf-dolt.git
git clone git@github.com-fartbagxp:fartbagxp/asdf-oauth2c.git
```

`pull.sh` to update all repositories in a folder.

```bash
#!/usr/bin/env bash

for d in */ ; do
  echo $d
  git -C $d pull
done
```
