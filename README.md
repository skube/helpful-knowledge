# Helpful Knowledge 🎓

A general repository for stuff I always keep forgetting how to do

# Using SSH Keys for Password-less Access

Logins are usually done with passwords. But really, who wants them though!?

SSH Keys allows for logging into to systems extremely sercurely without the need for remembering a (hopefully) complex password.

The idea is simple and can be likened to a physical padlock and key. 

A _padlock_ can be on anything you want to keep secure. The _key_ is used in combination with the padlock. The key is more private and can be shared (though shouldn't be)

With **SSH Keys** there are two* files: 
- the padlock here is the file `id_rsa.pub`
- and the key is the file `id_rsa`

> (Info: The files are usually referred to as keys)

These files are usually kept inside a hidden directory in your home directory (i.e. `~/.ssh`)

```sh
$ ls ~/.ssh/
id_rsa     id_rsa.pub
```
The id_rsa file is your private key. Keep this on your computer.

Just like in real life, you almost certainly need multiple keys & padlocks. There's nothing particualarily special with the name `id_rsa`. You can suffix a more descriptive word when generating **multiple** keys (e.g. `id_rsa_home`, `id_rsa_coolsite`)

To create a matching pair of keys (i.e. padlock & key), on Linux/MacOS use the `ssh-keygen` utility:

```sh
$ ssh-keygen [-t rsa -C "note to help id the pub file entry"]
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/skube/.ssh/id_rsa): [enter some descriptive note] id_rsa_<note>
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/skube/.ssh/id_rsa_<note>.
Your public key has been saved in /Users/skube/.ssh/id_rsa_<note>.pub.
```

> Note: It is strongly recommended to use a passphrase

> Info: 
> - The contents of [] are optional, a note will automatically be appended to the *.pub file as _user@machine_ 
> - `-t rsa` is default tells ssh-keygen to create an RSA type encrpytion
> - `-C` overrides the default comment on the *.pub file. This is for the  `authorized_keys` file as you can have several keys in that file

Now you have two files `id_rsa_<note>.pub` (e.g. _padlock_) and `id_rsa_<note>` (_key_)

Now the `*.pub` file needs to added to the server and appended to the `authorized_keys` file. 

This can be done in one step thanks to the magic of unix commands

```sh
# method 0 : copy to clipboard to pasted online
pbcopy < ~/.ssh/id_rsa_<note>.pub
```

```sh
# method 1: manually
cat ~/.ssh/id_rsa_<note>.pub | ssh user@remoteserver 'mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys'
```

```sh
# method 2: use ssh-copy-id util, if available
ssh-copy-id user@remotehost
```

> (Info: the `-p` flag on `mkdir` creates intermediate directories as required, not erroring if the directory already exists)

You can safely now delete the `*.pub` file as it has been appended to the `authorize_keys` file.

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

### Managing Multiple Keys

It's not uncommon to use multiple keys. To facilitate this, you can use a `~/.ssh/config` file. 

```sh
# contents of ~/.ssh/config
Host _friendly-name_
    User _user_
    HostName _remoteserver.com_
    IdentityFile ~/.ssh/id_rsa_<note>

# macOS Sierra no longer persists keys
Host *
  AddKeysToAgent yes
```

That's it! You should* be able to SSH simply by into a remote server by: `ssh friendly-name`

## Bypassing Passphrase

Now assuming you supplied a passphrase, you _may_ find entering it every time is annoying or perhaps some process requires automatic access. 

In this case, you can add your key to the SSH agent. The `ssh-agent` manages your SSH keys and remembers your passphrases.

To use ssh-agent and ssh-add, follow the steps below:

1. At the Unix prompt, enter:
```sh
eval `ssh-agent`
```
> Note: make sure you use the backquote (`\``), under the tilde (`~`), rather than the single quote (`'`).

2. Add the key to the agent:
```sh
ssh-add -K ~/.ssh/id_rsa_<custom>
```
> Info: `-K` adds to keychain for persistance (tho may not work after macOS Sierra)

3. Kill the current agent:
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
