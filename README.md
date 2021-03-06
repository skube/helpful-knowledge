# Helpful Knowledge 🎓

A general repository for stuff I always keep forgetting how to do

# Creating and Using SSH Keys for Passwordless Experience

Some well-written and thorough SSH information can be found at [ssh.com](https://www.ssh.com/ssh/command/)

## Concept
Using SSH Keys enables secure logins easily.

Generally concerned with two files:
- `id_rsa.pub` (public and shared on remote system)
- `id_rsa`(private on local system)

> The files—usually referred to as keys—can be likened to a physical padlock 🔒 (.pub) and key 🔑

These files are usually kept inside a hidden directory in your home directory (i.e. `~/.ssh`)

```sh
$ ls ~/.ssh/
id_rsa     id_rsa.pub
```
Like a 🔑, the `id_rsa` file is **not** to be public or shared.
Conversely, the 🔒can be on multiple systems.

> There's nothing particualarily special with the name `id_rsa`. 
> You can rename when generating keys (e.g. `id_rsa_home`, `my_coolsite.pub`)

## Generating Keys

To create a matching pair of keys 🔐(i.e. padlock & key), on Linux/MacOS use the `ssh-keygen` utility:

```sh
$ ssh-keygen -t rsa -C "note to help id the pub file entry"

Generating public/private rsa key pair.

Enter file in which to save the key (/Users/skube/.ssh/id_rsa): [enter some descriptive note] name_of_key_file

Enter passphrase (empty for no passphrase):
Enter same passphrase again:

Your identification has been saved in /Users/skube/.ssh/name_of_key_file
Your public key has been saved in     /Users/skube/.ssh/name_of_key_file.pub
```

> Note: It is recommended to use a passphrase (e.g. _orange you glad I didn't say banana_)
> Note: Passphrase can be temporarily saved/bypassed using a utility (`ssh-agent`)

> Info: 
> - The contents of [] are optional, a note will automatically be appended to the *.pub file as _user@machine_ 
> - `-t rsa` is default tells `ssh-keygen` to create an RSA type encrpytion
> - `-C` overrides the default comment on the `*.pub` file (why?)

## Authorized Keys

Now you have two files `id_rsa.pub` 🔒 and `id_rsa` 🔑. The 🔒 must be placed on the server and appended to the `authorized_keys` file.

### Method 1

One way to easily copy the `*.pub` file to a remote server is to use the `ssh-copy-id` util:

```sh
# If util available, supply the _key_ if not using the default `id_rsa`

$ ssh-copy-id [-i name_of_key_file] user@remotehost
```
> `-i` if you want to override the default identity file of `id_rsa`

### Method 2

Another way is with a long, complicated but impressive looking shell command:

```sh
# Fancy unix one-liner

$ cat ~/.ssh/id_rsa.pub | ssh user@remoteserver 'mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys'
```
> (Info: the `-p` flag on `mkdir` creates intermediate directories as required, not erroring if the directory already exists)

### Method 3

Otherwise, you can simply manually copy the 🔒contents and paste through an online form

```sh
# Manually copy to clipboard 
$ pbcopy < ~/.ssh/id_rsa.pub
```

You can safely now delete the `*.pub` file as it has been appended to the `authorize_keys` file.

## Using SSH

Now you can SSH into your remote server
```sh
# Method 1: 
ssh user@remoteserver -i ~/.ssh/id_rsa_<note>
```

> Info: the `-i` is to specifcy a path to the proper key when not using the the default `id_rsa`

```sh
# Method 2: If not using multiplie keys (i.e. only one `id_rsa` key)
ssh user@remoteserver
```

## Managing Multiple Keys

It's not uncommon to use multiple keys (not to mention it's easier to use a _friendly-name_). To facilitate this, you can use a `~/.ssh/config` file. 

```sh
# contents of ~/.ssh/config (replace <...>) 
Host <friendly-name>
    User <user>
    HostName <remoteserver.com>
    IdentityFile ~/.ssh/id_rsa_<note>

# macOS Sierra no longer persists keys
Host *
  AddKeysToAgent yes
```

That's it! You should* be able to SSH simply by into a remote server by: `ssh friendly-name`

## Bypassing Passphrase with `ssh-agent`

If you supplied a passphrase, you _may_ find it super annoying entering it each time.

The `ssh-agent` manages your SSH keys **and** remembers your passphrases.

To use `ssh-agent` and `ssh-add`, follow the steps below:

#### 1. At the Unix prompt, enter:

```sh
eval `ssh-agent`
```
> Note: make sure you use the backquote (\`) rather than the single quote (`'`).

#### 2. Add the key to the agent:

```sh
ssh-add -K ~/.ssh/id_rsa_<custom>
```
> Info: `-K` adds to keychain for persistance (tho may not work after macOS Sierra)

#### 3. Kill the current agent:

```sh
ssh-agent -k
```

## *Troubleshooting

### Problem 1

Your local and remote systems _may_ have different `ssh` versions. In that case, you need to set permissions on the `.ssh` directory and `authorized_keys` file of your remote system.

```sh
# method 1: one line
ssh user@remotehost 'chmod 700 ~/.ssh; chmod 640 ~/.ssh/authorized_keys'
```

```sh
# method 2: multi-line 
ssh user@remotehost
chmod 700 ~/.ssh
chmod 640 ~/.ssh/authorized_keys
```
> (note: 421 corresponds to rwx, so `chmod 640` means `chmod u=rw,g=r,o-a`)

### Problem 2:

If you see a message "Agent admitted failure to sign using the key" then add your RSA or DSA identities to the authentication agent ssh-agent then execute the following command:

```sh
ssh-add
```

## Using rsync with `ssh` identity file 

Setting up the `rsync` command (trailing slashes seem to be important)
```sh
rsync --progress -avz -e "ssh -i ~/.ssh/id_rsa_<note>" /path/to/local/dir/ <user>@<hostname>:/path/to/remote/dir/
```

Create a shell script for the synchronization

```sh
vi ~/bin/some-super-cool-name.sh
```

Add to the shell script file's contents:

```sh
#!/bin/bash <copy-the above rsync line here>
```

Add proper permissions

```sh
chmod 700 ~/bin/some-super-cool-name.sh
```
