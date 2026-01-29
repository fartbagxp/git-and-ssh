# Git and SSH

When using [Git](https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F) with [SSH](https://www.cloudflare.com/learning/access-management/what-is-ssh/) to connect a Git client to a Git server, these are the tricks to handle multiple Github/Gitlab accounts and servers.

I wrote this because I got tired of managing a chaotic environment with

1. numerous Git accounts (Github, Gitlab, Bitbucket),
1. many SSH keys tied to different identities,
1. and many different Git servers to connect to.

This guide assumes you're in a Unix/Linux environment or something like [cygwin](https://www.cygwin.com/).

## Overview

1. [Separate folders](#organize-folder-structure) when cloning repos so you don't mix up user handles across accounts.
1. [Separate .gitconfig files](#git-config-identity) to set different Git identities (name, email) per folder.
1. [Separate SSH keys](#ssh-keys) for authenticating as different Git users.
1. [Separate SSH configuration](#ssh-configurations) to route connections to different Git servers with different keys.
1. (Optional) [Automate cloning and pulls](#automate-cloning-and-pulls) to keep a record of what's in each folder.

## Organize folder structure

I keep a single `code` folder for all code. Inside it, I create subfolders matching the Git servers and profiles I use. For example, say `user1` is the username.

You can have multiple folders for different organizations and projects. Each **organization** maps to a Git server (Github, Gitlab, etc.) or an organization within one (like [https://github.com/google](https://github.com/google)). Each **project** underneath is a Git repository.

Once the folders are set up, you can configure [Git identity](#git-config-identity) to tie your username and email to each organization folder.

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

## Git config identity

Most people set their Git identity once, globally, because Git commits require it. Git servers like GitHub also [verify commits via the config email](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address#about-commit-email-addresses) to show a Verified badge.

The usual way is via the command line:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

!!! note

    GitHub recommends using [a noreply email](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address#about-no-reply-email) they provide so spammers can't harvest your commit email.

To automate `git config` across the [folder structure](#organize-folder-structure) from earlier, create a [gitconfig file](https://git-scm.com/book/ms/v2/Getting-Started-First-Time-Git-Setup) at the root of your home directory. It has no file extension -- just a plain text file.

For example, my .gitconfig file is at `~/.gitconfig` or `/home/user1/.gitconfig`.

Then add `includeIf` clauses pointing at each folder. The path after `gitdir:` must end in `/` to match a directory. This tells Git: when working inside `~/code/org1/`, use the config file `.gitconfig-org1`.

```toml
[includeIf "gitdir:~/code/org1/"]
  path = .gitconfig-org1
[includeIf "gitdir:~/code/org2/"]
  path = .gitconfig-org2
[includeIf "gitdir:~/code/github-org1/"]
  path = .gitconfig-github-org1
[includeIf "gitdir:~/code/github-org2/"]
  path = .gitconfig-github-org2
[includeIf "gitdir:~/code/gitlab-org1/"]
  path = .gitconfig-org1
[core]
  attributesfile = .gitattributes
```

The `.gitconfig-org1` file just has the user's name and email:

!!! note

    If you want to keep your email private on [Github](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address#about-commit-email-addresses), go to [Github email settings](https://github.com/settings/emails) and find the noreply address they give you.

```toml
[user]
name = "user1"
email = 1234556+user1@users.noreply.github.com
```

## SSH keys

SSH keys are an alternative to passwords for authenticating to an SSH server. Typing passwords every time is annoying, and the security tradeoffs between passwords and keys [are debated](https://serverfault.com/a/334478).

To generate an SSH keypair, I prefer ED25519 keys -- they're faster and smaller than RSA keys.

```bash
ssh-keygen -t ed25519 -o -a 100 -C "email@example.com" -f ~/.ssh/example_network_id_ed25519
```

Otherwise, to generate an RSA 4096-bit keypair:

!!! note

    RSA 2048-bit keys are still widely used until [standards phase them out in 2030](https://keylength.com). They're about [4x faster than RSA 4096-bit keys](https://www.fastly.com/blog/key-size-for-tls), while 4096-bit keys offer [slightly better encryption strength](https://crypto.stackexchange.com/questions/1978/how-big-an-rsa-key-is-considered-secure-today).

```bash
ssh-keygen -t rsa -b 4096 -C "email@example.com" -f ~/.ssh/example_network_id_rsa
```

(Optional) You can add a passphrase for extra security. To avoid retyping it, cache it with an SSH agent.

To start the SSH agent:

```bash
eval "$(ssh-agent -s)"
```

(Optional) Add your SSH key to the agent:

```bash
ssh-add ~/.ssh/example_network_id_ed25519
```

By convention, the private key file has no extension and the public key file ends in **.pub**.

## SSH configurations

The SSH config file determines which keypair and identity to use when connecting to a Git server. It binds a specific key to a specific host.

The config file lives at `~/.ssh/config`.

I set each Host entry to target a specific HostName (github.com, gitlab.com), use `git` as the User, prefer publickey authentication, and point IdentityFile at the right private key. You can have as many Host entries as you need.

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

## Automate cloning and pulls

`clone.sh` clones a list of repositories into a folder.

!!! note

    Notice that git@**github.com-org1** matches the [Host field in the SSH config](#ssh-configurations). This is how the Git client knows which SSH key to use for which server. The SSH config ties the Git user, the SSH key, and the Git server together.

```bash
#!/usr/bin/env bash
git clone git@github.com-org1:fartbagxp/aas-cidr-ranges.git
git clone git@github.com-org1:fartbagxp/git-and-ssh.git
git clone git@github.com-org1:fartbagxp/asdf-oauth2c.git
```

Following the [folder structure](#organize-folder-structure), you might have another `clone.sh` for a different organization:

```bash
#!/usr/bin/env bash
git clone git@gitlab.com-org1:dwt1/dotfiles.git
git clone git@gitlab.com-org1:wireshark/wireshark.git
```

(Optional) To pull all repos in a folder, `pull.sh` updates everything. This will cause conflicts if there are unmerged changes!

```bash
#!/usr/bin/env bash

for d in */ ; do
  echo $d
  git -C $d pull
done
```

## Miscellaneous tricks

Cloning or pushing to a Git server can get tricky depending on how your client connects (sometimes through firewalls) for the first time.

Here are some debugging tips for both SSH and HTTPS connections.

### Debugging SSH on Git

The default `git clone <repository>` doesn't show useful error messages when SSH connections fail.

To get debug output, prepend **GIT_SSH_COMMAND**:

```bash
GIT_SSH_COMMAND="ssh -v" git clone git@github.com:github/training-kit.git
```

For even more detail:

```bash
GIT_SSH_COMMAND="ssh -vvv" git clone git@github.com:github/training-kit.git
```

### Debugging HTTPS on Git

If the Git server or a firewall blocks SSH, you'll need to use HTTPS instead.

Use `nc` and `curl` first to check connectivity.

Test whether you can reach the Git server on SSH port 22:

```bash
nc -vz -w 3 github.com 22
```

If this fails and your internet works otherwise, a firewall is probably blocking SSH, or the server won't accept SSH connections from you.

Test HTTPS connectivity:

```bash
curl -vv https://github.com
```

You should see a wall of HTML fetched from the site. If it fails or hangs, something is blocking HTTPS too, and there's not much else to try.

If HTTPS works, clone over it:

```bash
git clone https://username@github.com/username/repository.git
```

You can also inline the password (less secure -- it gets stored in shell history):

```bash
git clone https://username:password@github.com/username/repository.git
```

A third option is a [Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic), which most Git servers support:

```bash
git clone https://*token*@github.com/username/repository.git
```

### SOCKS proxy with Git

A [SOCKS proxy](https://fartbagxp.github.io/ssh-proxy-traffic/ssh-proxy/proxy-with-socks) can get around firewalls blocking HTTPS to the Git server.

Once you have [a SOCKS proxy running](https://fartbagxp.github.io/ssh-proxy-traffic/ssh-proxy/proxy-with-socks), clone through it like this. This example uses `127.0.0.1` (localhost) on port 8055:

```bash
ALL_PROXY=socks5h://127.0.0.1:8055 git clone https://username@github.com/username/repository.git
```
