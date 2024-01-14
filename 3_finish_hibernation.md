# Complete the hibernate setup

Reference: [Suspending to Disk in Ubuntu-based Distro with BTRFS filesystem | Chuma's Journal](https://journal.chumaumenze.com/entries/suspending-to-disk-in-ubuntu-based-distro-with-btrfs-filesystem/#test-and-enable-hibernation)

1. Disable the default Zram

Pop OS now has Zram enabled by default. With this module enabled, the kernel can
create a RAM device with active disk compression of files and or inactive memory
pages. The RAM device can be used for swap or as a disk for storage of temporary
files.

For hibernation, Zram is not a suitable option. The module may be used for swap
in RAM to speed up operation, however, contents in the RAM are volatile and do
not persist after a power loss.

```sh
# Remove Pop OS Zram settings
sudo apt purge pop-default-settings-zram

# Reboot the system
sudo reboot now
```

2. Ensure hibernation works by executing the following command.

```sh
systemctl hibernate
```

3. Install browser extension: <https://extensions.gnome.org/#>

4. Add Hibernate and Hybrid Sleep options to the power menu, using GNOME-Shell
   extension:
   <https://extensions.gnome.org/extension/755/hibernate-status-button/>

5. Add the following policy kit:

```sh
sudo tee /etc/polkit-1/localauthority/10-vendor.d/com.ubuntu.desktop.pkla << EOF
[Enable hibernate in upower]
Identity=unix-user:*
Action=org.freedesktop.upower.hibernate
ResultActive=yes

[Enable hibernate in logind]
Identity=unix-user:*
Action=org.freedesktop.login1.hibernate;org.freedesktop.login1.handle-hibernate-key;org.freedesktop.login1;org.freedesktop.login1.hibernate-multiple-sessions;org.freedesktop.login1.hibernate-ignore-inhibit
ResultActive=yes
EOF
```

6. Ensure hibernation works by using the Power Off/ Log Out menu.

Reboot.
