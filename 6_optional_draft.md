# Optional Draft

## Restore from Backup

I mount my luks encrypted backup storage drive using nautilus and use rsync to
copy over my files and important configuration scripts:

```sh
export BACKUP=/media/$USER/UUIDOFBACKUPDRIVE/@home/$USER/
sudo rsync -avuP $BACKUP/Pictures ~/
sudo rsync -avuP $BACKUP/Documents ~/
sudo rsync -avuP $BACKUP/Downloads ~/
sudo rsync -avuP $BACKUP/dynare ~/
sudo rsync -avuP $BACKUP/Images ~/
sudo rsync -avuP $BACKUP/Music ~/
sudo rsync -avuP $BACKUP/Desktop ~/
sudo rsync -avuP $BACKUP/SofortUpload ~/
sudo rsync -avuP $BACKUP/Videos ~/
sudo rsync -avuP $BACKUP/Templates ~/
sudo rsync -avuP $BACKUP/Work ~/
sudo rsync -avuP $BACKUP/.config/Nextcloud ~/.config/
sudo rsync -avuP $BACKUP/.gitkraken ~/
sudo rsync -avuP $BACKUP/.gnupg ~/
sudo rsync -avuP $BACKUP/.local/share/applications ~/.local/share/
sudo rsync -avuP $BACKUP/.matlab ~/
sudo rsync -avuP $BACKUP/.ssh ~/
sudo rsync -avuP $BACKUP/.dynare ~/
sudo rsync -avuP $BACKUP/.gitconfig ~/

sudo chown -R $USER:$USER /home/$USER
```

## Virtual machines: Quickemu and other stuff

I used to set up KVM, Qemu, virt-manager and gnome-boxes as this is much faster
as VirtualBox. However, I have found a much easier tool for most tasks:
Quickqemu which uses the snap package Qemu-virgil:

```sh
git clone https://github.com/wmutschl/quickemu ~/quickemu
sudo apt install snapd bsdgames wget
sudo snap install qemu-virgil
sudo snap connect qemu-virgil:kvm
sudo snap connect qemu-virgil:raw-usb
sudo snap connect qemu-virgil:removable-media
sudo snap connect qemu-virgil:audio-record
sudo ln -s ~/quickemu/quickemu /home/$USER/.local/bin/quickemu
# Note that I keep my virtual machines on an external SSD
```

In case I need the old stuff:

```sh
sudo apt install -y qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virt-manager libvirt-daemon ovmf gnome-boxes
sudo adduser $USER libvirt
sudo adduser $USER libvirt-qemu
# run gnome-boxes
# run libvirt add user session
# As I use btrfs I need to change compression of images to no:
sudo chattr +C ~/.local/share/gnome-boxes
sudo chattr +C ~/.local/share/libvirt
```

## Networking
OpenSSH Server

I sometimes access my linux machine via ssh from other machines, for this I
install the OpenSSH server:

sudo apt install openssh-server

Then I make some changes to

sudo nano /etc/ssh/sshd_config

to disable password login and to allow for X11forwarding.
Nextcloud

I have all my files synced to my own Nextcloud server, so I need the sync
client:

```sh
sudo apt install -y nextcloud-desktop
```

Open Nextcloud and set it up. Recheck options.
OpenConnect and OpenVPN

```sh
sudo apt install -y openconnect network-manager-openconnect network-manager-openconnect-gnome
sudo apt install -y openvpn network-manager-openvpn network-manager-openvpn-gnome
```

Go to Settings-Network-VPN and add openconnect for my university VPN and openvpn
for ProtonVPN, check connections.

## Security steps with Yubikey

I have two Yubikeys and use them as second-factor for all admin/sudo tasks to
unlock my luks encrypted partitions for my private GPG key

For this I need to install several packages:

```sh
sudo apt install -y yubikey-manager yubikey-personalization # some common packages
# Insert the yubikey
ykman info # your key should be recognized
# Device type: YubiKey 5 NFC
# Serial number: 
# Firmware version: 5.1.2
# Form factor: Keychain (USB-A)
# Enabled USB interfaces: OTP+FIDO+CCID
# NFC interface is enabled.
# 
# Applications	USB    	NFC     
# OTP     	Enabled	Enabled 	
# FIDO U2F	Enabled	Enabled 	
# OpenPGP 	Enabled	Enabled 	
# PIV     	Enabled	Disabled	
# OATH    	Enabled	Enabled 	
# FIDO2   	Enabled	Enabled 	

sudo apt install -y libpam-u2f # second-factor for sudo commands
sudo apt install -y yubikey-luks  # second-factor for luks
sudo apt install -y gpg scdaemon gnupg-agent pcscd gnupg2 # stuff for GPG
```

Make sure that OpenPGP and PIV are enabled on both Yubikeys as shown above.

## Yubikey: two-factor authentication for admin/sudo password

Let’s set up the Yubikeys as second-factor for everything related to sudo using
the common-auth pam.d module:

```sh
pamu2fcfg > ~/u2f_keys # When your device begins flashing, touch the metal contact to confirm the association. You might need to insert a user pin as well
pamu2fcfg -n >> ~/u2f_keys # Do the same with your backup device
sudo mv ~/u2f_keys /etc/u2f_keys
# Make this required for common-auth
echo "auth    required                        pam_u2f.so nouserok authfile=/etc/u2f_keys cue" | sudo tee -a /etc/pam.d/common-auth
```

Before you close the terminal, open a new one and check whether you can do sudo
echo test.

## Yubikey: two-factor authentication for luks partitions

Let’s set up the Yubikeys as second-factor to unlock the luks partitions. If you
have brand new keys, then create a new key on them:

```sh
ykpersonalize -2 -ochal-resp -ochal-hmac -ohmac-lt64 -oserial-api-visible #BE CAREFUL TO NOT OVERWRITE IF YOU HAVE ALREADY DONE THIS
```

Now we can enroll both yubikeys to the luks partition:

```sh
export LUKSDRIVE=/dev/nvme0n1p4
#insert first yubikey
sudo yubikey-luks-enroll -d $LUKSDRIVE -s 7 # first yubikey
#insert second yubikey
sudo yubikey-luks-enroll -d $LUKSDRIVE -s 8 # second yubikey
export CRYPTKEY="luks,keyscript=/usr/share/yubikey-luks/ykluks-keyscript"
sudo sed -i "s|luks|$CRYPTKEY|" /etc/crypttab
cat /etc/crypttab #check whether this looks okay
sudo update-initramfs -u
```

## Yubikey: private GPG key

Let’s use the private GPG key on the Yubikey (a tutorial on how to put it there
is taken from Heise or YubiKey-Guide). My public key is given in a file called
/home/$USER/.gnupg/public.asc:

```sh
sudo systemctl enable pcscd
sudo systemctl start pcscd
# Insert yubikey
gpg --card-status
# If this did not find your Yubikey, then try to first reboot.
# If it still does not work, then put
# echo 'reader-port Yubico YubiKey' >> ~/.gnupg/scdaemon.conf
# reboot and try again. Make sure to enable pcscd.
cd ~/.gnupg
gpg --import public.asc #this is my public key, my private one is on my yubikey
export KEYID=91E724BF17A73F6D
gpg --edit-key $KEYID
  trust
  5
  y
  quit
echo "This is an encrypted message" | gpg --encrypt --armor --recipient $KEYID -o encrypted.txt
gpg --decrypt --armor encrypted.txt
# If this did not trigger to enter the Personal Key on your Yubikey, then try to put
# echo 'reader-port Yubico YubiKey' >> ~/.gnupg/scdaemon.conf
# reboot and try again. Make sure to enable pcscd.
```
