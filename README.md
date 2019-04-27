# Multiple SSH Keys For Separate GitHub Accounts

As an example, let's say you have two GitHub accounts:

1. One for personal projects: `jsore@email.com`
2. One for work related projects: `jsore.work@email.com`

You have one machine to develop on but want to use separate SSH keys for each account

This is building on GitHub's articles for [setting up GitHub SSH access on a Mac](https://help.github.com/en/articles/checking-for-existing-ssh-keys)

<br>





## Create the keys

Run `ssh-keygen`, supplying your GitHub account's email address as a comment ( `-C` )

```
( running cd with no arguments moves you $HOME )
~ $ cd
~ $ ssh-keygen -t rsa -b 4096 -C "jsore@email.com"
> Generating public/private rsa key pair.
```

<br>



You'll be asked what to name the keys and where to create them ( include the
whole path with the name )

The names can be anything, just make sure you can tell them apart

```
> Enter file in which to save the key (/Users/jsore/.ssh/id_rsa):
>     /Users/jsore/.ssh/github_personal_rsa
```

<br>



Repeat for however many GitHub accounts you want SSH keys for

```
~ $ ssh-keygen -t rsa -b 4096 -C "jsore.work@email.com"
> Generating public/private rsa key pair.
> Enter file in which to save the key (/Users/jsore/.ssh/id_rsa):
>     /Users/jsore/.ssh/github_work_rsa
```

<br>



I also went ahead and built a key pair to be a 'default' key for single-use VM's

```
~ $ ssh-keygen -t rsa -b 4096 -C "jsore.private@email.com"
> Generating public/private rsa key pair.
> Enter file in which to save the key (/Users/jsore/.ssh/id_rsa):
>     /Users/jsore/.ssh/default_ssh_rsa
```

<br>



Your keys can be checked with `ssh-add -l` and deleted with `ssh-add -D`

<br>





## Dealing with the `ssh-agent`

For macOS 10.12.2 or later, you'll need to update the `~/.ssh/config` file

<br>



Start the agent then open the config file, `touch ~/.ssh/config` first if it doesn't exist

```
~ $ eval "~ $(ssh-agent -s)"
~ $ open .ssh/config "Sublime Text"
```

<br>



This is the config GitHub mentions in their SSH article

```bash
Host *
    AddKeysToAgent yes    # auto-load keys into ssh-agent
    UseKeychain yes       # store the passphrases ( keys ) in your keychain
    IdentityFile ~/.ssh/id_rsa    # <-- equivalent to my github_personal_rsa
```

<br>



Instead, don't default all SSH connections to our GitHub keys, define specific hosts

```bash
Host *
    AddKeysToAgent yes
    UseKeychain yes
    #IdentityFile ~/.ssh/id_rsa
    IdentityFile ~/.ssh/default_ssh_rsa    # for my temporary VM's

# will be used in conjunction with git clone Host variable
# the -Host should be the same as your GitHub account's username
Host github.com-jsore
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_personal_rsa    # this is the private key, not .pub

Host github.ibm.com-sorensen
    HostName github.ibm.com
    User git
    IdentityFile ~/.ssh/github_work_rsa
```

<br>



Add the private keys to the `ssh-agent` and use mac-specific `-K` flag for keychain storage

```
~ $ ssh-add -K ~/.ssh/github_personal_rsa
> Identity added: /Users/justin/.ssh/github_personal_rsa (jsore@email.com)
~ $ ssh-add -K ~/.ssh/github_work_rsa
> Identity added: /Users/justin/.ssh/github_work_rsa (jsore.work@email.com)
~ $ ssh-add -K ~/.ssh/default_ssh_rsa
> Identity added: /Users/justin/.ssh/default_ssh_rsa (jsore.private@email.com)
```

<br>



## Configuring your repos

Add your **public** keys to each of your GitHub accounts - **NEVER** share the private

```
( copy each to your clipboard )
~ $ pbcopy < ~/.ssh/github_personal_rsa.pub
( then paste in github.com/settings/keys >> New SSH key )
```

<br>



Now, I only have a couple work-related repos - the majority of my repos are on
my personal account - so, I have a `--global` config setting to use my personal
account's login info and SSH key ( defined in the `.ssh/config` Host ) and set
manual definitions for my couple of work repos

```
~ $ git config --global user.name "jsore"
~ $ git config --global user.email "jsore@email.com"
~ $ cd some/dir/for/work
```

<br>



Let's assume I'm making a new pointer to an existing repo in my work GitHub account
and need to `git clone` down to my local machine: I'll need to use the -Host variable
I mentioned in the comments in `.ssh/config`

```
work $ git clone git@github.ibm.com-sorensen:some/work/project.git DirectoryName
work $ cd DirectoryName
```

<br>

The `-sorensen` portion is the `Host` portion, it specifies which `Host` to use from `.ssh/config`

<br>

Just set local username options and you're good to go using your normal dev flow

```
work/DirectoryName $ git config user.name "sorensen"
work/DirectoryName $ git config user.email "jsore.work@email.com"

( make some changes to something )

DirectoryName $ git add .
DirectoryName $ git commit -m "some comments"
DirectoryName $ git push -u origin master

( pushes without bugging for a password because auth happens with key pair )
```

<br>

You can check out your local git config options

```
work/DirectoryName $ cat .git/config
> ...
> [remote "origin"]
>     url = git@github.ibm.com-sorensen:some/work/project.git
>     fetch = +refs/heads/*:refs/remotes/origin/*
> ...
> [user]
>     name = sorensen
>     email = jsore.work@email.com
> ...
```

<br>

And compare them with your global settings

```
work/DirectoryName $ cd
~ $ git config --list
> ...
> user.name=Justin Sorensen
> user.email=jsore@email.com
> ...
~ $ cat .gitconfig
> [user]
>     name = sorensen
>     email = jsore.work@email.com
```


<br>

Change to a new project that isn't strictly isolated to work

```
DirectoryName $ cd some/dir/for/projects
projects $ git init

( start a new README )

projects $ git add README.md
projects $ git commit -m "some comments"
projects $ git push -u origin master
```









Host *
    UseKeychain yes
    AddKeysToAgent yes
    IdentityFile ~/.ssh/id_rsa



    /*----------  credential-osxkeychain helper  ----------*/
~ $ git credential-osxkeychain
    > usage: git credential-osxkeychain <get|store|erase>
    ~ $ git config --global credential.helper osxkeychain
( attempt to clone a repo )




    git clone https://github.com/jsore/Snippets.git
    ( OSX asks for login password in a popup )
    ( github auth will fail, do the clone again )
Cloning into 'Snippets'...
remote: Invalid username or password.
fatal: Authentication failed for 'https://github.com/jsore/Snippets.git/'

git clone https://github.com/jsore/Snippets.git
Cloning into 'Snippets'...
Username for 'https://github.com': jsore
Password for 'https://jsore@github.com':
remote: Enumerating objects: 53, done.
remote: Counting objects: 100% (53/53), done.
remote: Compressing objects: 100% (41/41), done.
remote: Total 53 (delta 19), reused 45 (delta 11), pack-reused 0
Unpacking objects: 100% (53/53), done.

( enter github username & password when prompted )
    ( all subsequent HTTPS push/pulls gets login data from keychain now )
    git add usefulSnippets.js
    git commit -m "Testing OSX keychain, ignore"
    git push -u origin master