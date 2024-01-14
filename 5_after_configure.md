# After configure

References:

- https://mutschler.dev/linux/pop-os-post-install/
- https://blog.zachinachshon.com/storage-volume/
- https://github.com/unixorn/fzf-zsh-plugin
- https://github.com/bobthecow/git-flow-completion
- https://medium.com/@kjdeluna/upgrade-your-terminal-experience-with-zsh-oh-my-zsh-and-powerlevel10k-d2aabf145112
- https://unix.stackexchange.com/questions/657256/autocompletion-of-makefile-with-makro-in-zsh-not-correct-works-in-bash
- https://unix.stackexchange.com/questions/111718/command-history-in-zsh


## Permanently mount additional disk

```sh
sudo blkid
/dev/mapper/cryptdata: UUID="TVajpY-EKAZ-jJ6s-39nQ-CtRg-GdkX-qQSk6s" TYPE="LVM2_member"
/dev/mapper/data-root: UUID="8062bdbc-fb55-4f0d-8426-033241b86402" UUID_SUB="dfda8dd3-1810-47f4-b971-21f756d5a0ea" BLOCK_SIZE="4096" TYPE="btrfs"
/dev/sdb2: UUID="3340-2759" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="recovery" PARTUUID="b8803837-5c66-4f4d-844b-5caa44195fc4"
/dev/sdb3: UUID="f5587dee-374e-4ab9-afdd-b6a4f5530254" TYPE="crypto_LUKS" PARTUUID="d98ed4d8-2f66-4bb7-98e4-cf4c224135cf"
/dev/sdb1: UUID="3340-27C4" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="397e9bb5-11ae-460c-b5f7-b13890914655"
/dev/sda1: UUID="52164e25-3e48-42f1-93ba-b14058c03ab4" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="data" PARTUUID="33d3efb9-57fe-47a6-ad69-16d567b2a2fd"

echo "UUID=$(blkid -s UUID -o value /dev/sda1)  /vol ext4 defaults,auto,users,rw,nofail,x-systemd.device-timeout=30 0 0" | sudo tee -a /etc/fstab

sudo mkdir /vol

sudo mount -av
/boot/efi                : already mounted
/recovery                : already mounted
/                        : ignored
/home                    : already mounted
/swap                    : already mounted
none                     : ignored
/vol                     : successfully mounted

sudo chown -R $USER:$USER /vol
```

## Set hostname

If you need some creativity: https://namingschemes.com/Main_Page

```sh
hostnamectl set-hostname meanmachine
```

## Set locales and get rid of unnecessary languages

```sh
# List locales to pick
locale -a

sudo locale-gen en_US.UTF.8
sudo locale-gen es_AR.UTF.8
sudo locale-gen pt_BR.UTF.8
sudo update-locale LANG=en_US.UTF-8
```

In Region Settings open “Manage Installed Languages”, do not update these, but first remove the unnecessary ones. Then reopen “languages” and update these.

## Install updates and reboot

```sh
sudo apt update
sudo apt upgrade
sudo apt dist-upgrade
sudo apt autoremove
sudo apt autoclean
sudo fwupdmgr get-devices
sudo fwupdmgr get-updatesgit-flow-completion.bash
sudo fwupdmgr update
flatpak update
sudo pop-upgrade recovery upgrade from-release # this updates the recovery partition
sudo reboot now
```

## Enable snap support

```sh
sudo apt install snapd
```

## Some system utilities

### Caffeine

```sh
sudo apt install -y caffeine
```

Run caffeine indicator.

### GParted

```sh
sudo apt install -y gparted
```

### nautilus-admin

Right-click context menu in nautilus for admin

```sh
sudo apt install -y nautilus-admin
```

### Nautilus open in Vscode

```sh
wget -qO- https://raw.githubusercontent.com/harry-cpp/code-nautilus/master/install.sh | bash
```

### Git related packages

```sh
sudo apt install -y git git-lfs
git-lfs install
```

### Visual Studio Code

```sh
sudo apt install -y code
```

### Tools

```sh
sudo apt install -y meld
sudo apt install -y htop
sudo apt install -y tmux
sudo apt install -y tree
sudo apt install -y imagemagick-6.q16hdri libmagickcore-6.q16hdri-6-extra
```

### Install with Pop Shop

* Chrome
* KeePassXC

## Tweaks

* Set Tile windows.

* Add Floating Windows Exceptions:
    * Settings
    * Calculator

* Reorder Favorites on Dock (Terminal, Nautilus, Vscode, Firefox)
* Remove Pop Show from Dock
* Remove Settings from Dock

* Hit META+A and uninstall: Contacts, Calendar, Geary, Weather.

* In nautilus add bookmark to /vol

* History search in terminal using page up and page down:

```sh
sudo nano /etc/inputrc
# Uncomment the following lines
# "\e[5~": history-search-backward
# "\e[6~": history-search-forward
```

## Settings

* Turn off bluetooth

Desktop:
    Desktop Options:
        * Show Maximize Button.
    Background:
        * Change Background.
    Dock:
        * Deactivate Extend dock to the edges of the screen.
        * Deactivate Show Launcher Incon in Dock.
        * Deactivate Show Workspaces Icon in Dock.
        * Deactivate Show Applications Icon in Dock.
        * Dock visibility: intelligently hide
        * Show Dock on Display: All Displays
        * Dock Size: Small
    Workspaces:
        * Set Fixed Number of Workspaces.
Privacy:
    File History & Trash:
        * Automatically delete file history in 30 days
        * Automatically delete temporary files and trash in 30 days
    Screen:
        * Turn of screen after 15 min with automatic screen lock.
Sound:
    * Mute mic
Power:
    * Disable Automatic Suspend.
    * Turn on hibernate for power button.
    * Enable Show Battery Percentage.
Displays:
    * Turn on night mode Manual Schedule 20:00 to 06:00
Mouse & Touchpad:
    * Turn on natural scroll for mouse touchpad.
    * Go through keyboard shortcuts and adapt:
        * Remove super+l super+h
        * Assign additional super+l to Lock screen
        * Assign additional super+w to Show workspaces
        * Custom Shortcut: xkill ctrl+alt+backspace
User:
    * Change avatar image.
Date & TIme:
    * Change clock to 24h format

## Gnome-tweaks

```sh
sudo apt install gnome-tweaks
```

Make the following changes:

General:

* Disable “Suspend when laptop lid is closed” in General.

Top Bar:

* Enable “Weekday” and “Date”.
* Enable Calendar “Week Numbers”.

## Gnome Extension

* Change Lock screen background, using GNOME-Shell extension: https://extensions.gnome.org/extension/1476/unlock-dialog-background/

* Show Numlock and Capslock status on the panel, using GNOME-Shell extension: https://extensions.gnome.org/extension/1532/lock-keys/

* Unblank lock screen, using GNOME-Shell extension: https://extensions.gnome.org/extension/1414/unblank/

## SSH keys

If I want to create a new SSH key, I run e.g.:

```sh
ssh-keygen -t ed25519 -C "my-new-key"
```

Usually, however, I restore my .ssh folder from my backup. Either way, afterwards, one needs to add the file containing your key, usually id_rsa or id_ed25519, to the ssh-agent:

```sh
# First set the right permission to the restored private keys
chmod 600 ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_ed25519

eval "$(ssh-agent -s)" #works in bash
ssh-add ~/.ssh/id_ed25519
```

## bash git flow completion

```sh
sudo apt install -y bash-completion

mkdir -p ~/.bash_completion
wget -nv https://raw.githubusercontent.com/bobthecow/git-flow-completion/master/git-flow-completion.bash -O ~/.bash_completion/git-flow-completion.bash
```

## zsh and oh-my-zsh

```sh
sudo apt install -y zsh

wget -O- -nv https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh \
&& git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/.oh-my-zsh/custom/themes/powerlevel10k \
&& rm -rf ~/.oh-my-zsh/custom/themes/powerlevel10k/.git* \
&& mkdir -p ~/.cache/gitstatus \
&& wget -O- -nv https://github.com/romkatv/gitstatus/releases/download/v1.3.1/gitstatusd-linux-x86_64.tar.gz | tar -xz -C ~/.cache/gitstatus gitstatusd-linux-x86_64

git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions \
&& rm -rf ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions/.git*
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting \
&& rm -rf ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting/.git*

git clone --depth 1 https://github.com/unixorn/fzf-zsh-plugin.git ~/.oh-my-zsh/custom/plugins/fzf-zsh-plugin \
&& rm -rf ~/.oh-my-zsh/custom/plugins/fzf-zsh-plugin/.git*

git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
Do you want to enable fuzzy auto-completion? ([y]/n) y
Do you want to enable key bindings? ([y]/n) y
Do you want to update your shell configuration files? ([y]/n) n

git clone https://github.com/bobthecow/git-flow-completion ~/.oh-my-zsh/custom/plugins/git-flow-completion \
&& rm -rf ~/.oh-my-zsh/custom/plugins/git-flow-completion/.git*
```

Download these four ttf files:
- [MesloLGS NF Regular.ttf](
       https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf)
- [MesloLGS NF Bold.ttf](
       https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf)
- [MesloLGS NF Italic.ttf](
       https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf)
- [MesloLGS NF Bold Italic.ttf](
       https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf)

Double-click on each file and click "Install". This will make `MesloLGS NF` font available to all applications on your system.

Configure your terminal to use this font:

- **Visual Studio Code**: Open *File → Preferences → Settings* (PC) or
     *Code → Preferences → Settings* (Mac), enter `terminal.integrated.fontFamily` in the search box at the top of *Settings* tab and set the value below to `MesloLGS NF`.
     Consult [this screenshot](
       https://raw.githubusercontent.com/romkatv/powerlevel10k-media/389133fb8c9a2347929a23702ce3039aacc46c3d/visual-studio-code-font-settings.jpg)
     to see how it should look like or see [this issue](
       https://github.com/romkatv/powerlevel10k/issues/671) for extra information.
- **GNOME Terminal** (the default Ubuntu terminal): Open *Terminal → Preferences* and click on the selected profile under *Profiles*. Check *Custom font* under *Text Appearance* and select`MesloLGS NF Regular`.

Configure p10k:

```sh
zsh
p10k configure
# Options:
# - (1) Rainbow
# - (1) Unicode
# - (n) Do not show curren time
# - (1) Angled separator
# - (1) Sharp prompt head
# - (1) Flat prompt tail
# - (1) One line prompt height
# - (1) Compact prompt spacing
# - (1) Few Icons
# - (1) Concise prompt flow
# - (y) Enable transient prompt
# - (1) Verbose instant prompt mode
```

Add fzf-zsh-plugin to your plugin list - edit ~.zshrc and change plugins=(...) to plugins=(... fzf-zsh-plugin)

Make zsh the default shell:

```sh
chsh -s $(which zsh)
```

## kopia backup

References:

- https://kopia.io/docs/installation/#linux-installation-using-apt-debian-ubuntu
- https://kopia.io/docs/faqs/
- https://kopia.io/docs/getting-started/
- https://kopia.io/docs/advanced/compression/
- https://www.cyberciti.biz/faq/howto-linux-unix-test-disk-performance-with-dd-command/
- https://kopia.io/docs/advanced/kopiaignore/
- https://kopia.io/docs/mounting/

```sh
curl -s https://kopia.io/signing-key | sudo gpg --dearmor -o /etc/apt/keyrings/kopia-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kopia-keyring.gpg] http://packages.kopia.io/apt/ stable main" | sudo tee /etc/apt/sources.list.d/kopia.list\nsudo apt update
sudo apt install -y kopia
sudo apt install -y kopia-ui
```

Check disk throughput to select repository compression:

```sh
dd if=/dev/zero of=/vol/test1.img bs=1G count=1 oflag=dsync
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 3.82768 s, 281 MB/s

kopia benchmark compression --data-file=/vol/test1.img
NOTICE: The provided input file is too big, using first 134.2 MB.
     Compression                Compressed   Throughput   Allocs   Usage
------------------------------------------------------------------------------------------------
  0. s2-parallel-8              2.6 KB       6.8 GB/s     1097     17 MB
  1. s2-parallel-4              2.6 KB       6.8 GB/s     974      15.9 MB
  2. s2-default                 2.6 KB       4.8 GB/s     1116     18.1 MB
  3. pgzip                      188.7 KB     3.7 GB/s     1205     28 MB
  4. s2-better                  2.6 KB       3.5 GB/s     1053     22.3 MB
  5. pgzip-best-speed           188.8 KB     3.5 GB/s     1211     29.5 MB
  6. deflate-default            132.6 KB     3.2 GB/s     30       1.6 MB
  7. deflate-best-speed         132.7 KB     2.3 GB/s     30       1.3 MB
  8. zstd-fastest               26.6 KB      2.1 GB/s     6223     9.9 MB
  9. zstd                       14.4 KB      1.9 GB/s     3137     19.3 MB
 10. zstd-better-compression    14.4 KB      1.3 GB/s     3136     39 MB
 11. gzip-best-speed            162.9 KB     1.1 GB/s     37       1.7 MB
 12. gzip                       130.5 KB     357.4 MB/s   33       1.1 MB
 13. gzip-best-compression      130.5 KB     353.6 MB/s   33       1.1 MB
 14. pgzip-best-compression     140.9 KB     97.5 MB/s    1301     45.2 MB
 15. deflate-best-compression   138.2 KB     76.4 MB/s    31       1.7 MB

rm -v -i /vol/test1.img

dd if=/dev/zero of=./test1.img bs=1G count=1 oflag=dsync
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 0.943067 s, 1.1 GB/s

kopia benchmark compression --data-file=./test1.img
NOTICE: The provided input file is too big, using first 134.2 MB.
     Compression                Compressed   Throughput   Allocs   Usage
------------------------------------------------------------------------------------------------
  0. s2-parallel-4              2.6 KB       6.4 GB/s     985      18 MB
  1. s2-parallel-8              2.6 KB       5 GB/s       988      19.1 MB
  2. s2-default                 2.6 KB       4.8 GB/s     1109     19.1 MB
  3. s2-better                  2.6 KB       4.4 GB/s     1123     20.2 MB
  4. pgzip                      188.7 KB     3.8 GB/s     1185     24.7 MB
  5. deflate-default            132.6 KB     3.3 GB/s     31       1.6 MB
  6. pgzip-best-speed           188.8 KB     2.7 GB/s     1236     33.7 MB
  7. deflate-best-speed         132.7 KB     2.2 GB/s     30       1.3 MB
  8. zstd                       14.4 KB      2.1 GB/s     3151     19.3 MB
  9. zstd-fastest               26.6 KB      2.1 GB/s     6220     9.9 MB
 10. zstd-better-compression    14.4 KB      1.5 GB/s     3138     39 MB
 11. gzip-best-speed            162.9 KB     1.1 GB/s     36       1.7 MB
 12. gzip                       130.5 KB     346.7 MB/s   33       1.1 MB
 13. gzip-best-compression      130.5 KB     346.6 MB/s   33       1.1 MB
 14. pgzip-best-compression     140.9 KB     97.7 MB/s    1277     43 MB
 15. deflate-best-compression   138.2 KB     76.2 MB/s    31       1.7 MB

rm -v -i ./test1.img
```

As soon as the throughput of compression is higher than I/O, compression is no longer the bottleneck. Therefore, any higher compression basically comes as free.

In this example, for a repository in (/dev/sda) /vol we could use deflate-default for a balance betwenn speed/memory/compression.

For a repository in /dev/sdb we could use deflate-default for a balance betwenn speed/memory/compression.

We are going to set the compression globally:

```sh
kopia policy set --global --compression=deflate-default
```

Create a `.kopiaignore` file for the $HOME folder:

```
tee ~/.kopiaignore << END
# Ignore folders within the parent directory
/Downloads
/.cache
/.mozilla
/.local/state
/.local/share

# Ignore hot files and folders
/hot
.netrc

# Ignore files and folders provided by .dotfiles
/scripts
.bashrc
.zshrc
.gitconfig

# Ignore any .LOCK file
*.LOCK

# Ignore files
.sudo_as_admin_successful

# Ignore any folder beginning with _ within the parent directory
/[_]*

END
```

Inspect if there are more files or folders to ignore and edit `~/.kopiaignore` if needed.

Create a local repository in `/vol/_backup`.

Create a policy for doing manual snapshots of the `$HOME`.
This snapshots are just in case meanwhile we finish the machine setup.

Create an snapshot using the new policy.

Let's check if every thing was ok with the snapshot and if we need to ignore more files or folders, if that is the case, edit `~/.kopiaignore` acordingly, delete the snapshot, take a new one and do the verification again.

To mount a particular snapshot use the root id:

```sh
mkdir /tmp/mnt
kopia mount kcac3c69f604a9eeb3e7dbdd8c5135a17 /tmp/mnt &
umount /tmp/mnt
```

To mount the `latest` snapshots of the repository:

```sh
mkdir /tmp/mnt
kopia mount all /tmp/mnt &
umount /tmp/mnt
```

When the special path `all` is used, the whole repository with its latest snapshot version is mounted.
