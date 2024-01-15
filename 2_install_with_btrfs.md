# Install with btrfs

References:

- [Pop!_OS 22.04: installation guide with btrfs, luks encryption and auto snapshots with timeshift | mutschler.dev](https://mutschler.dev/linux/pop-os-btrfs-22-04/)
- [Suspending to Disk in Ubuntu-based Distro with BTRFS filesystem | Chuma's Journal](https://journal.chumaumenze.com/entries/suspending-to-disk-in-ubuntu-based-distro-with-btrfs-filesystem)
- [Enabling hibernation with full disk encryption on Pop!_OS 21.04. · GitHub](https://gist.github.com/dawaltconley/8cb4c3cfac7da394a58fab363628bf63)
- [[HowTo] Enable and configure hibernation with BTRFS - Tutorials - Manjaro Linux Forum](https://forum.manjaro.org/t/howto-enable-and-configure-hibernation-with-btrfs/51253)
- [Swapfile — BTRFS  documentation](https://btrfs.readthedocs.io/en/latest/Swapfile.html)

1. Do mutschler_pop-os-btrfs-22-04 Step 1 complete.

**IF VIRTUAL MACHINE**: Install vm-tools so is easier to copy and paste:

```sh
sudo apt install open-vm-tools-desktop
```

**IF BARE METAL**: Open `gparted`, delete the swap partition and resize the main
partition to get the new free space.

2. Do mutschler_pop-os-btrfs-22-04 Step 2 complete.

3. Start mutschler_pop-os-btrfs-22-04 Step 3.

```sh
sudo -i

# Mount the btrfs top-level root filesystem with zstd compression
cryptsetup luksOpen /dev/sdb3 cryptdata
# Enter passphrase for /dev/sda3
mount -o subvolid=5,defaults,compress=zstd:1,discard=async /dev/mapper/data-root /mnt

# Create btrfs subvolumes @ and @home
btrfs subvolume create /mnt/@
# Create subvolume '/mnt/@'
cd /mnt
ls | grep -v @ | xargs mv -t @
# this moves all files that don't have "@" in the name to the @/ folder
ls -a /mnt
# . .. @

btrfs subvolume create /mnt/@home
# Create subvolume '/mnt/@home'
mv /mnt/@/home/* /mnt/@home/
ls -a /mnt/@/home
# . ..
ls -a /mnt/@home
# . .. wmutschl

btrfs subvolume list /mnt
# ID 264 gen 339 top level 5 path @
# ID 265 gen 340 top level 5 path @home
```

4. **IF BARE METAL**: We do some steps from chumaumenze_suspending-to-disk:

```sh
# Create a @swap sub-volume
btrfs subvolume create /mnt/@swap

# List sub-volumes on the root level
btrfs subvolume list /mnt
```

This tutorial and many other are using btrfs version 6.1 and above, it has
specific commands for creating swapfiles and also for getting the offset that we
need to update the kernel.

At this moment (2023-01-09) the last stable btrfs version is: v5.16.2, so we
need to get the create the swapfile manually and get the offset in another
creative way.

```sh
free -h
# See the actual memory, here 16Gi, we create the swap with 33Gi
# so we are ready to add more memory to the notebook later

# Manually create the swapfile.
truncate -s 0 /mnt/@swap/swapfile
chattr +C /mnt/@swap/swapfile
fallocate -l 33G /mnt/@swap/swapfile
chmod 0600 /mnt/@swap/swapfile
mkswap /mnt/@swap/swapfile
```

5. **IF BARE METAL**: We do some modified steps from dawaltconley_using-a-swapfile:

```sh
# Comment out the current swap from fstab
sed -i 's!^/dev/mapper/cryptswap!# &!' /etc/fstab
# Optionally, comment out the current swap from crypttab
sed -i 's!^cryptswap!# &!' /etc/crypttab
```

6. **IF BARE METAL**: We found the swapfile offset following some steps from
   manjaro_howto-enable-and-configure-hibernation-with-btrfs:

```sh
curl -s "https://raw.githubusercontent.com/osandov/osandov-linux/master/scripts/btrfs_map_physical.c" > bmp.c
gcc -O2 -o bmp bmp.c

SWAP_UUID=$(blkid -s UUID -o value /dev/mapper/data-root)
SWAP_OFFSET=$(echo "$(./bmp /mnt/@swap/swapfile | egrep "^0\s+" | cut -f9) / $(getconf PAGESIZE)" | bc)
KERNEL_OPTS="resume=UUID=$SWAP_UUID resume_offset=${SWAP_OFFSET/../}"
echo $KERNEL_OPTS
# we are going to copy and paste the $KERNEL_OPTS value later...
```

7. We continue mutschler_pop-os-btrfs-22-04 from "Changes to fstab"":

```sh
sed -i 's/btrfs  defaults/btrfs  defaults,subvol=@,compress=zstd:1,discard=async/' /mnt/@/etc/fstab
echo "UUID=$(blkid -s UUID -o value /dev/mapper/data-root)  /home  btrfs  defaults,subvol=@home,compress=zstd:1,discard=async   0 0" >> /mnt/@/etc/fstab
```

8. **IF BARE METAL**: We do some modified steps from chumaumenze_suspending-to-disk:

```sh
echo "UUID=$(blkid -s UUID -o value /dev/mapper/data-root)  /swap btrfs defaults,subvol=@swap,nodatacow,noatime,nospace_cache,compress=no 0 0" >> /mnt/@/etc/fstab
echo "/swap/swapfile none swap defaults 0 0" >> /mnt/@/etc/fstab

cat /mnt/@/etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system>  <mount point>  <type>  <options>  <dump>  <pass>
PARTUUID=397e9bb5-11ae-460c-b5f7-b13890914655  /boot/efi  vfat  umask=0077  0  0
PARTUUID=b8803837-5c66-4f4d-844b-5caa44195fc4  /recovery  vfat  umask=0077  0  0
UUID=8062bdbc-fb55-4f0d-8426-033241b86402  /  btrfs  defaults,subvol=@,compress=zstd:1,discard=async  0  1
UUID=8062bdbc-fb55-4f0d-8426-033241b86402  /home  btrfs  defaults,subvol=@home,compress=zstd:1,discard=async   0 0
UUID=8062bdbc-fb55-4f0d-8426-033241b86402  /swap btrfs defaults,subvol=@swap,nodatacow,noatime,nospace_cache,compress=no 0 0
/swap/swapfile none swap defaults 0 0
```

9. We continue mutschler_pop-os-btrfs-22-04 from "Changes to crypttab":

```sh
sed -i 's/luks/luks,discard/' /mnt/@/etc/crypttab

cat /mnt/@/etc/crypttab
cryptdata UUID=f5587dee-374e-4ab9-afdd-b6a4f5530254 none luks,discard
```

Adjust configuration of kernelstub. Here you need to add rootflags=subvol=@ to 
the "user" kernel options:

```sh
nano /mnt/@/etc/kernelstub/configuration
# {
#   "default": {
#     "kernel_options": [
#       "quiet",
#       "splash"
#     ],
#     "esp_path": "/boot/efi",
#     "setup_loader": false,
#     "manage_mode": false,
#     "force_update": false,
#     "live_mode": false,
#     "config_rev":3
#   },
#   "user": {
#     "kernel_options": [
#       "quiet",
#       "loglevel=0",
#       "systemd.show_status=false",
#       "splash",
#       "rootflags=subvol=@"
#     ],
#     "esp_path": "/boot/efi",
#     "setup_loader": true,
#     "manage_mode": true,
#     "force_update": false,
#     "live_mode": false,
#     "config_rev":3
#   }
# }
```

VERY IMPORTANT: Don’t forget to put a comma after "splash"

Adjust configuration of systemd bootloader:

```sh
mount /dev/sdb1 /mnt/@/boot/efi

sed -i 's/splash/splash rootflags=subvol=@/' /mnt/@/boot/efi/loader/entries/Pop_OS-current.conf
cat /mnt/@/boot/efi/loader/entries/Pop_OS-current.conf
# title Pop!_OS
# linux /EFI/Pop_OS-UUID_of_data-root/vmlinuz.efi
# initrd /EFI/Pop_OS-UUID_of_data-root/initrd.img
# options root=UUID=UUID_of_data-root ro quiet loglevel=0 systemd.show_status=false splash rootflags=subvol=@
```

Optionally, I like to add a timeout to the systemd boot menu in order to easily access the recovery partition:

```sh
echo "timeout 3" >> /mnt/@/boot/efi/loader/loader.conf
cat /mnt/@/boot/efi/loader/loader.conf 
# default Pop_OS-current
# timeout 3
```

Create a chroot environment and update initramfs:

```sh
echo $KERNEL_OPTS
cd /
umount -l /mnt
mount -o subvol=@,defaults,compress=zstd:1,discard=async /dev/mapper/data-root /mnt
ls /mnt
# bin boot dev etc home lib lib32 lib64 libx32 media mnt opt proc recovery root run sbin srv sys tmp usr var

for i in /dev /dev/pts /proc /sys /run; do mount -B $i /mnt$i; done
chroot /mnt
```

Now we are inside the chroot.

10. **IF BARE METAL**: We create the mount point for the swap:

```sh
mkdir swap
```

11. We mount everything:

```sh
mount -av
```

12. **IF BARE METAL**: We do some modified steps from chumaumenze_suspending-to-disk:

```sh
# Copy and paste the previous $KERNEL_OPTS
KERNEL_OPTS="<value>"
echo $KERNEL_OPTS
kernelstub -a "$KERNEL_OPTS"
echo "$KERNEL_OPTS" | tee -a /etc/initramfs-tools/conf.d/resume

cat /etc/initramfs-tools/conf.d/resume
resume=UUID=8062bdbc-fb55-4f0d-8426-033241b86402 resume_offset=902840
```

13. We update the initramfs to make it aware of our changes to the kernelstub and
   continue mutschler_pop-os-btrfs-22-04 from "Step 4: Reboot, some checks, and
   system updates":

```sh
update-initramfs -c -k all

# Exit the chroot.
exit
```

Close the terminal and finally hit Reboot Device on the installer app. Cross
your fingers! If all went well you should see a passphrase prompt, where you
enter the luks passphrase and your system should boot.

> Note: THe ouput of the following commands will vary depending if we make
> changes to swap or not.

```sh
sudo mount -av
/boot/efi                : already mounted
/recovery                : already mounted
/                        : ignored
/home                    : already mounted
/swap                    : already mounted
none                     : ignored

sudo mount -v | grep /dev/mapper
/dev/mapper/data-root on / type btrfs (rw,relatime,compress=zstd:1,ssd,discard=async,space_cache=v2,subvolid=256,subvol=/@)
/dev/mapper/data-root on /home type btrfs (rw,relatime,compress=zstd:1,ssd,discard=async,space_cache=v2,subvolid=257,subvol=/@home)
/dev/mapper/data-root on /swap type btrfs (rw,noatime,compress=zstd:1,ssd,discard=async,space_cache=v2,subvolid=258,subvol=/@swap)

sudo swapon
NAME           TYPE SIZE USED PRIO
/swap/swapfile file  33G   0B   -2

free -h
               total        used        free      shared  buff/cache   available
Mem:            15Gi       2.3Gi        10Gi       447Mi       3.0Gi        11Gi
Swap:           32Gi          0B        32Gi

sudo btrfs filesystem show /
Label: none  uuid: 8062bdbc-fb55-4f0d-8426-033241b86402
    Total devices 1 FS bytes used 40.77GiB
    devid    1 size 467.92GiB used 51.02GiB path /dev/mapper/data-root

sudo btrfs subvolume list /
ID 256 gen 600 top level 5 path @
ID 257 gen 600 top level 5 path @home
ID 258 gen 288 top level 5 path @swap

sudo systemctl enable fstrim.timer

cat /etc/lvm/lvm.conf | grep issue_discards
#   # Configuration option devices/issue_discards.
#   issue_discards = 1

# If all look’s good, let’s update and upgrade the system:
sudo apt update
sudo apt upgrade
sudo apt dist-upgrade
sudo apt autoremove
sudo apt autoclean
flatpak update
```

14. Reboot.
