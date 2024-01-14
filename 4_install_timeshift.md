# Install timeshift

1. Install timeshift and configure it directly via the GUI:

```sh
sudo apt install -y timeshift
sudo timeshift-gtk
```

Select “btrfs” as the “Snapshot Type”; continue with “Next”
Choose your btrfs system partition as “Snapshot Location”; continue with “Next”
“Select Snapshot Levels” (type and number of snapshots that will be
automatically created and managed/deleted by timeshift), my recommendations:
        Activate “Monthly” and set it to 3
        Activate “Weekly” and set it to 3
        Activate “Daily” and set it to 5
        Deactivate “Hourly”
        Activate “Boot” and set it to 5
        Activate “Stop cron emails for scheduled tasks”
Continue with “Next”

I do not include the @home subvolume (which is not selected by default). Note
that when you restore a snapshot with timeshift you get to choose whether you
want to restore @home as well (which in most cases you actually don’t want to
do!).

Activate “Enable BTRFS qgroups (recommended)”. There are some issues on GitHub
that there MIGHT be some performance issues with this (if you manually
deactivate quotas as well), but I’ve never had any issues, so I stick with the
recommendation to enable it. Click “Finish”

“Create” a manual first snapshot, add a comment “clean install” to it & exit
timeshift.

In the terminal you will see an ERROR: can't list qgroups: quotas not enabled.
Simply ignore this as this error comes only the first time you run timeshift.

2. Check the backup:

```sh
ls /run/timeshift/backup/timeshift-btrfs/snapshots/2024-01-11_07-07-23/@
```

Note that /run/timeshift/backup/@ is your / folder and
/run/timeshift/backup/@home your /home folder. Your snapshots are accessible via
timeshift-btrfs.

3. Install timeshift-autosnap-apt:

```sh
sudo apt install -y git make
mkdir /home/$USER/.deb
git clone https://github.com/wmutschl/timeshift-autosnap-apt.git /home/$USER/.deb/timeshift-autosnap-apt
cd /home/$USER/.deb/timeshift-autosnap-apt
sudo make install
```

Make changes to the configuration file:

```sh
sudo nano /etc/timeshift-autosnap-apt.conf
```

For example, as we don’t have a dedicated /boot partition, we can set
snapshotBoot=false in the timeshift-autosnap-apt-conf file such that the /boot
directory is not rsync’ed to /boot.backup. Note that the EFI partition will be
rsynced into /boot.backup/efi. So if something goes wrong with the EFI
partition, you always have a backup of it as well. Moreover, POP!_OS does not
use GRUB so we can set updateGrub=false.

```sh
cat /etc/timeshift-autosnap-apt.conf
#
# /etc/timeshift-autosnap.conf
#

# snapshotBoot defines if /boot folder should be cloned into /boot.backup before the call to timeshift.
# Default value is true.
snapshotBoot=false

# snapshotEFI defines if /boot/efi folder should be cloned into /boot.backup/efi before the call to timeshift.
# Default value is true.
snapshotEFI=true

# skipAutosnap defines if timeshift-autosnap execution should be skipped.
# Default value is false.
skipAutosnap=false

# deleteSnapshots defines if old snapshots should be deleted.
# Default value is true.
deleteSnapshots=true

# maxSnapshots defines how much old snapshots script should left.
# Only positive whole numbers can be used.
# Default value is 3.
maxSnapshots=5

# updateGrub defines if grub entries should be auto-generated.
# If grub-btrfs package is not installed grub won't be generated.
# Default value is true.
updateGrub=false

# snapshotDescription defines value used to distinguish snapshots created using timeshift-autosnap
# Default value is "{timeshift-autosnap} {created before upgrade}".
snapshotDescription={timeshift-autosnap-apt} {created before call to APT}
```

Check if everything is working:

```sh
sudo timeshift-autosnap-apt
Rsyncing /boot/efi into the filesystem before the call to timeshift.
Using system disk as snapshot device for creating snapshots in BTRFS mode

/dev/dm-1 is mounted at: /run/timeshift/backup, options: rw,relatime,compress=zstd:1,ssd,discard=async,space_cache=v2,subvolid=5,subvol=/

Creating new backup...(BTRFS)
Saving to device: /dev/dm-1, mounted at path: /run/timeshift/backup
Created directory: /run/timeshift/backup/timeshift-btrfs/snapshots/2024-01-11_08-42-39
Created subvolume snapshot: /run/timeshift/backup/timeshift-btrfs/snapshots/2024-01-11_08-42-39/@
Created control file: /run/timeshift/backup/timeshift-btrfs/snapshots/2024-01-11_08-42-39/info.json
BTRFS Snapshot saved successfully (5s)
Tagged snapshot '2024-01-11_08-42-39': ondemand
------------------------------------------------------------------------------
```

Now, if you run sudo apt install|remove|upgrade|dist-upgrade,
timeshift-autosnap-apt will create a snapshot of your system with timeshift.
