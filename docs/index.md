# Git and SSH

When using [Git](https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F) with [Secure Shell (SSH)](https://www.cloudflare.com/learning/access-management/what-is-ssh/) to connect from a Git client to a Git server, these are the tricks to handle multiple Github/Gitlab accounts to connect to multiple Github/Gitlab servers.

This spawns out of a necessity to manage a chaotic environment with

1. numerous Git (ex. Github, Gitlab, Bitbucket) accounts,
1. many SSH keys associated with different identities,
1. and many different Git servers to connect to.

This guide assumes you're in a Unix/Linux environment or a Linux like environment such as [cygwin](https://www.cygwin.com/).

## Overview

1. [Separate folders](#organize-folder-structure) when cloning a different Git account user's repository to ensure we don't mix and match two different user handles when working on a repository.
1. [Separate .gitconfig files](#git-config-identity) to separate Git identity - name and email separation for selective folders.
1. [Separate SSH keys](#ssh-keys) for identity management when identifying as a Git user to connect to a Git server.
1. [Separate SSH Configuration](#ssh-configurations) as a configuration to connect to different Git servers using different Git users.
1. (Optional) [Automate cloning and pulls](#automate-cloning-and-pulls) so that we have a history of what's in the folders.

## Organize Folder Structure

I favor creating a single `code` folder for all code related artifacts. Within the folder, I create subfolders aligned to Git servers and Git profiles I intend to associate with. For example, image `user1` is the name of the user.

We can have multiple folder representing different organizations and projects underneath. Each **organization** tends to associate with a Git server (ex. Github, Gitlab, etc) or an organization within Github or Gitlab (ex. [https://github.com/google](https://github.com/google)). Each **project** underneath is a Git repository.

Once we organize our folder structure in this manner, we can setup [Git identity](#git-config-identity) to associate our Git identity (username, email) to each of the organization folders.

```bash
/home/user1/code
├── org1
│ ├── project1
│ ├── project2
├── org2
│ ├── project1
│ ├── project2
├── org3
│ ├── projectz
├── org4
│ ├── project9
│ ├── projectb
```

## Git Config Identity

Most people set their Git identity manually using the command line once globally because Git commits requires it. Git server providers such as GitHub also [verifies Git commit via Git config email](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address#about-commit-email-addresses) to put a Verified badge on the commit.

The most common way done manually is via the command line:

```bash
# To set your name and email address globally, use the following commands in your terminal:
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

!!! note

    GitHub recommends using [a noreply email](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address#about-no-reply-email) they provide to avoid getting spammed by soliciters seeking to profit off your Git commit emails.

To automate `git config` across the [folder structure](#organize-folder-structure) setup previously, we create a [gitconfig file](https://git-scm.com/book/ms/v2/Getting-Started-First-Time-Git-Setup) that is stored the root of our directory. This file has no file extension and is a simple text file.

For example, my .gitconfig file is located at `~/.gitconfig` or `/home/user1/.gitconfig`.

Next, we include an `includeIf` clause to point at each of the [folder structure](#organize-folder-structure) created earlier, the path after `gitdir:` must end in `/` to represent a directory. This instructs Git that when looking in the path of `~/code/org/`, to use the configuration specified in the path, `.gitconfig-org1`.

```toml
[includeIf "gitdir:~/code/org1/"]
  path = .gitconfig-org1
[includeIf "gitdir:~/code/org2/"]
  path = .gitconfig-org2
[includeIf "gitdir:~/code/github-org3/"]
  path = .gitconfig-github-org3
[includeIf "gitdir:~/code/gitlab-org4/"]
  path = .gitconfig-org4
[core]
  attributesfile = .gitattributes
```

The `.gitconfig-org1` file includes a configuration of the user's name and email address.

!!! note

    If you decide to keep your email private for [Github](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address#about-commit-email-addresses), go to [Github email settings](https://github.com/settings/emails) and navigate to the section of the Github provided email you should use.

```toml
[user]
name = "user1"
email = 1234556+user1@users.noreply.github.com
```

## SSH Keys

SSH Keys serves as another way other than passwords for a SSH client to authenticate to a SSH server. Typing in passwords to login every time is rather annoying. The security tradeoffs [are often debated](https://serverfault.com/a/334478) between using passwords and using SSH keys for authentication.

To generate SSH keypairs of public and private key as a client, I prefer ED25519 based keys, as they're faster and smaller than RSA based keys.

```bash
ssh-keygen -t ed25519 -o -a 100 -C "email@example.com" -f ~/.ssh/example_network_id_ed25519
```

Otherwise, to generate a RSA 4096 bits keypair:

!!! note

    RSA 2048 bit keys are still widely in use until [standards phases it out in 2030](https://keylength.com). It is about [4x faster than RSA 4096 bit keys to use](https://www.fastly.com/blog/key-size-for-tls) while RSA 4096 bit keys [have slightly better encryption strength](https://crypto.stackexchange.com/questions/1978/how-big-an-rsa-key-is-considered-secure-today).

```bash
ssh-keygen -t rsa -b 4096 -C "email@example.com" -f ~/.ssh/example_network_id_rsa
```

(Optional) You may choose to add a passphrase for extra security and to avoid having to type the password again, it's best to have it cached once with an SSH agent.

To run SSH agent, run the following:

```bash
eval "$(ssh-agent -s)"
```

(Optional) Add your SSH key to the SSH agent.

```bash
ssh-add ~/.ssh/example_network_id_ed25519
```

The SSH convention for keys is that the private key file has no extensions while the public key file has a **.pub** extension.

## SSH Configurations

SSH Configurations determines what SSH keypair and identity to use when a Git client tries to connect to a Git server; it binds the usage of the keypair with the SSH client's identity when connecting to a Git server listening for SSH connections.

The SSH configuration is a single file `config` in the directory path `~/.ssh/config`.

I modify the config file to target a specific HostName (ex. github.com, gitlab.com) to use a particular User (ex. git) and prefer to use the authentication method of publickey instead of password and select the SSH private key path as the IdentityFile. The config file can include numerous SSH configurations for the different Hosts (ex. Github, Gitlab, Bitbucket) to target.

```bash
Host github.com-org1
  HostName github.com
  User git
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/git/github_org1_ed25519
  IdentitiesOnly yes

Host github.com-org2
  HostName github.com
  User git
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/git/github_org2_ed25519
  IdentitiesOnly yes

Host gitlab.com-org1
  HostName gitlab.com
  User git
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/git/gitlab_org1_ed25519
  IdentitiesOnly yes
```

## Automate Cloning and Pulls

`clone.sh` to clone a number of repositories into a folder.

!!! note

    Notice below that git@**github.com-org1** matches our [HostName field in the SSH configuration](#ssh-configurations) so that the Git client can associate a Git user with a Git server. The [SSH configuration](#ssh-configurations) binds the Git association together, to select a particular Git user with a particular SSH key for a particular Git server target.

```bash
#!/usr/bin/env bash
git clone git@github.com-org1:fartbagxp/aas-cidr-ranges.git
git clone git@github.com-org1:fartbagxp/aas-cidr-ranges.git
git clone git@github.com-org1:fartbagxp/asdf-oauth2c.git
```

Following the [folder structure](#organize-folder-structure) we setup, we may have another `clone.sh` script for another organization and the configuration binds the Git user, the unique SSH key, and the Git server target together.

```bash
#!/usr/bin/env bash
git clone git@gitlab.com-org1:dwt1/dotfiles.git
git clone git@gitlab.com-org1:wireshark/wireshark.git
```

(Optional) To sync repositories without verifying, `pull.sh` updates all repositories in a folder. This will cause git conflicts if there are unmerged changes!

```bash
#!/usr/bin/env bash

for d in */ ; do
  echo $d
  git -C $d pull
done
```
