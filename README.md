# zfs-root-notes-debian-buster
Some notes regarding zfs root on debian buster.

## Install the base system

* Using [debian-buster-zfs-root.sh](https://github.com/hn/debian-buster-zfs-root/blob/master/debian-buster-zfs-root.sh) to install the base system.


## Serial console configuration

1. Edit /etc/default/grub, modify these lines:
```
GRUB_CMDLINE_LINUX_DEFAULT="boot=zfs console=ttyS0,115200n8"
GRUB_CMDLINE_LINUX="root=ZFS=rpool/ROOT/debian-buster"
```
2. Update grub:

```
update-grub
grub-install --target=x86_64-efi --efi-directory=/boot/efi
```
3. Copy EFI system partitions to other disks if you're mirroring rpool.
```
umount /boot/efi
dd if=/dev/vda2 of=/dev/vdb2 bs=1M
mount /boot/efi
```
4. For each rpool member disk, create an EFI boot entry:
```
efibootmgr -c -g -d /dev/vda -p 2 -L "debian-disk0" -l '\EFI\debian\grubx64.efi'
efibootmgr -c -g -d /dev/vdb -p 2 -L "debian-disk1" -l '\EFI\debian\grubx64.efi'
```
5. (Optional) Disable swap in /etc/fstab
- Systemd might get stuck while enabling swap and I don't need that. Just disable that.

## Notes on EFI shell
If something failed and you've dropped onto EFI shell, you need to find and execute 'grubx64.efi'. The basic manipulations are:
- Find the device mapping
```
map
```
- Navigate the contents of the 1st ESP
```
ls fs1:
```
- Execute grubx64.efi, location might vary
```
fs0:\EFI\debian\grubx64.efi
```
- The EFI shell escape charactor is '^'


## Notes on grub CLI

If your grub fails to load its configfile, the basic manipulations are:

- Load grub modules:
```
insmod zfs
insmod linux
insmod initrd
```

- Locate grub.cfg, it shoud be in your rpool's /boot directory.
- Load grub configfile, again location might vary.
```
configfile (hd0,gpt3)/ROOT/debian-buster@/boot/grub/grub.cfg
```
- Invalid grub.cfg, you need to load kernel and initrd manually

```
linux $YOUR_KERNEL_LOCATION
initrd $YOUR_INITRD_LOCATION
```

## Credits

* https://github.com/hn/debian-buster-zfs-root
