# Password Recovery via GRUB

> **What it is:** If you forget the root password or get locked out, you can reset it by booting into single-user mode or emergency mode via GRUB. This requires physical or console access to the server.

## Method 1: RHEL/CentOS 7+ and Fedora (rd.break)

```bash
# Step 1: Reboot the system
reboot

# Step 2: At GRUB menu, press 'e' to edit boot entry

# Step 3: Find the line starting with 'linux' or 'linux16'
# Example:
# linux16 /vmlinuz-... root=/dev/mapper/... ro crashkernel=auto ...

# Step 4: Add 'rd.break' to the END of that line:
# linux16 /vmlinuz-... root=/dev/mapper/... ro rd.break

# Step 5: Press Ctrl+X or F10 to boot

# Step 6: Remount filesystem as read-write:
mount -o remount,rw /sysroot

# Step 7: Chroot into the system:
chroot /sysroot

# Step 8: Reset the root password:
passwd root

# Step 9: If SELinux is enabled, relabel on next boot:
touch /.autorelabel

# Step 10: Exit and reboot:
exit
exit
# or: reboot -f
```

## Method 2: Ubuntu/Debian

```bash
# Step 1: Reboot and hold Shift to get GRUB menu

# Step 2: Select your kernel and press 'e' to edit

# Step 3: Find line starting with 'linux' and change:
#   ro   -->   rw init=/bin/bash

# Step 4: Press Ctrl+X or F10 to boot

# Step 5: Reset password:
passwd root

# Step 6: Sync and reboot:
sync
reboot -f
```

## Method 3: Using Live CD/USB

```bash
# Step 1: Boot from Live CD/USB

# Step 2: Find and mount root partition:
lsblk
mount /dev/sda2 /mnt
# For LVM:
vgscan
vgchange -ay
mount /dev/mapper/rhel-root /mnt

# Step 3: Chroot into the system:
mount --bind /dev /mnt/dev
mount --bind /proc /mnt/proc
mount --bind /sys /mnt/sys
chroot /mnt

# Step 4: Change password:
passwd root

# Step 5: Exit and reboot:
exit
umount /mnt/dev /mnt/proc /mnt/sys /mnt
reboot
```

## GRUB Boot Parameters

| Parameter | Description |
|-----------|-------------|
| `rd.break` | Break before pivot (RHEL/CentOS, recommended) |
| `init=/bin/bash` | Boot directly to bash shell |
| `single` | Single user mode |
| `emergency` | Emergency mode |
| `systemd.unit=rescue.target` | Rescue target (systemd) |
| `1` | Runlevel 1 (SysV init) |

## Quick Reference

```bash
# After getting shell:
mount -o remount,rw /sysroot    # Remount as read-write
chroot /sysroot                  # Enter the real system
passwd root                      # Change password
touch /.autorelabel             # SELinux relabel (RHEL)
reboot -f                        # Force reboot
```

---

*Part of the [Linux Documentation](README.md)*

