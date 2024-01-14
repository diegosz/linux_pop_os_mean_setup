# Recipes

## LUKS Encryption

References:

- [boot - LUKS: How can I add more password slots (or remove/change a password) - Ask Ubuntu](https://askubuntu.com/questions/1319688/luks-how-can-i-add-more-password-slots-or-remove-change-a-password)

### Add LUKS passphrase

```sh
sudo cryptsetup luksAddKey /dev/sdb3
Enter any existing passphrase:
Enter new passphrase for key slot:
Verify passphrase:

# Verify new passphrase
sudo cryptsetup --verbose open --test-passphrase /dev/sdb3
```

### Change LUKS passphrase

```sh
sudo cat /etc/crypttab
cryptdata UUID=f5587dee-374e-4ab9-afdd-b6a4f5530254 none luks,discard

# Test old passphrase
sudo cryptsetup --verbose open --test-passphrase /dev/sdb3
No usable token is available.
Enter passphrase for /dev/sdb3: 
Key slot 0 unlocked.
Command successful.

sudo cryptsetup luksChangeKey /dev/sdb3 -S 0
Enter passphrase to be changed:
Enter new passphrase:
Verify passphrase:

# Verify new passphrase
sudo cryptsetup --verbose open --test-passphrase /dev/sdb3
```

### Remove a password slot

Possibility 1: You have to enter the password which you want to delete (it will
automatically find the correct password slot)

```sh
sudo cryptsetup luksRemoveKey /dev/sda3
```

Possibility 2: This will delete password slot 2 (you have to enter the password
of any other password slot, but not of slot 2. This works even if you don't know
the password of slot 2.

```sh
sudo cryptsetup luksKillSlot /dev/sda3 2
```

## eCryptfs Encryption

eCryptfs stores cryptographic metadata in the header of each file written, so
that encrypted files can be copied between hosts; the file will be decryptable
with the proper key, and there is no need to keep track of any additional
information aside from what is already in the encrypted file itself.

Installation of eCryptfs:

```sh
sudo apt install ecryptfs-utils
```

Setup is easy. First, create your “private” directory that will contain the
encrypted files and sub-directories. For example:

```sh
mkdir ~/Documents/private
```

When this directory is not “mounted”, you can look at the contents of the files
in it, but you will see nothing meaningful, since everything will be encrypted.
To use (read & write) to the unencrypted version of it, you should “mount” this
directory. You can mount this directory over to itself like this:

```sh
sudo mount -t ecryptfs ~/Documents/private ~/Documents/private
```

Since this is the first time you try to mount this directory with eCryptfs, you
will answer a few questions like this:

- First, enter a passphrase that you will never forget.
- Cipher: aes (default)
- Key bytes: 32
- Plaintext passthrough: n (default)
- Filename encryption: n (default)

Now, the important warning:

WARNING: Based on the contents of [/root/.ecryptfs/sig-cache.txt], it looks like
you have never mounted with this key before. This could mean that you have typed
your passphrase wrong.

Would you like to proceed with the mount (yes/no)? :

Since this is the first time you mount this directory, you will answer yes so
that a new file called ~root/.ecryptfs/sig-cache.txt will be created, which will
contain a “signature” of the passphrase.

Would you like to append sig [xxxxxxxxxxxx] to [/root/.ecryptfs/sig-cache.txt]
in order to avoid this warning in the future (yes/no)? : 

Answer also yes to this question so the file ~root/.ecryptfs/sig-cache.txt will
be populated.

Later, when we re-mount this directory we should not get this warning, unless we
enter the wrong passphrase.

```sh
mkdir /vol/home_back
sudo mount -t ecryptfs /vol/home_back /vol/home_back

export BACKUP=/vol/home_back/fresh
mkdir $BACKUP/.local/share/
sudo rsync -avuP ~/Desktop $BACKUP/
sudo rsync -avuP ~/Documents $BACKUP/
sudo rsync -avuP ~/Downloads $BACKUP/
sudo rsync -avuP ~/Music $BACKUP/
sudo rsync -avuP ~/Pictures $BACKUP/
sudo rsync -avuP ~/Public $BACKUP/
sudo rsync -avuP ~/Templates $BACKUP/
sudo rsync -avuP ~/Videos $BACKUP/
sudo rsync -avuP ~/.bash_logout $BACKUP/
sudo rsync -avuP ~/.bashrc $BACKUP/
sudo rsync -avuP ~/.config $BACKUP/
sudo rsync -avuP ~/.deb $BACKUP/
sudo rsync -avuP ~/.gitconfig $BACKUP/
sudo rsync -avuP ~/.local/share/applications $BACKUP/.local/share/
sudo rsync -avuP ~/.mozilla $BACKUP/
sudo rsync -avuP ~/.pki $BACKUP/
sudo rsync -avuP ~/.profile $BACKUP/
sudo rsync -avuP ~/.vscode $BACKUP/
```

## SSH Keys

Change comment of ssh key:

```sh
ssh-keygen -c -C "some clever comment" -f ~/.ssh/my_private_key
```

Show key in file:

```sh
ssh-keygen -lf ~/.ssh/my_private_key
```

Restart ssh-agent:

```sh
killall ssh-agent; eval "$(ssh-agent)"
```

## History

Clean history in bash:

```sh
history -cw
```

Clean history in zsh:

```sh
# history -p
truncate -s 0 ~/.zsh_history
```
