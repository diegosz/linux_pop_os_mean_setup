# Initial Setup

We could do the installation in bare metal or in a virtual machine.

## Bare Metal

In Bare Metal we will remove the swap partition and create a swap file inside a btrfs subvolume and enable hibernation.

Make sure that the BIOS is set to use **UEFI** and **Secure Boot** is disabled.

## Virtual Machine

For the virtual machine, we will NOT change the swap partition and we will NOT enable hibernation.

Create a new virtual machine, but before starting it, go to the virtual machine settings and in Options, Advanced, select **UEFI** and disable **Secure Boot**.
