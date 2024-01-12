# After configure

References:

- https://mutschler.dev/linux/pop-os-post-install/
- https://blog.zachinachshon.com/storage-volume/

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
sudo fwupdmgr get-updates
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

### Git related packages

```sh
sudo apt install -y git git-lfs
git-lfs install
```

### Visual Studio Code

```sh
sudo apt install -y code
```

### Meld

```sh
sudo apt install -y meld
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

## zsh and oh-my-zsh

```sh
sudo apt install -y zsh

wget -O- -nv https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh \
&& git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/.oh-my-zsh/custom/themes/powerlevel10k \
&& rm -rf ~/.oh-my-zsh/custom/themes/powerlevel10k/.git* \
&& mkdir -p ~/.cache/gitstatus \
&& wget -O- -nv https://github.com/romkatv/gitstatus/releases/download/v1.3.1/gitstatusd-linux-x86_64.tar.gz | tar -xz -C ~/.cache/gitstatus gitstatusd-linux-x86_64


```