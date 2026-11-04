# Can you help me figure out why my External Hard drive is not showing up in Ubuntu? It shows up fine on Windows and MacOS

I'll start by checking whether the system detects the drive.

Ran terminal command: lsblk -f 2>&1; echo ---; lsusb 2>&1; echo ---; dmesg 2>&1 | tail -30; echo ---; journalctl -k --no-pager 2>&1 | grep -iE "usb|sd[a-z]|ntfs|exfat" | tail -20

The drive is detected as `sda` but shows no partitions or filesystem. I'll dig further.

Ran terminal command: journalctl -k --no-pager 2>&1 | grep -iE "sd 0|sda|uas|usb-storage|2-1.1" | tail -25; echo ---; lsblk -b -o NAME,SIZE,TYPE,PTTYPE,MODEL /dev/sda; echo ---; sudo -n fdisk -l /dev/sda 2>&1 | head -20; sudo -n blkid /dev/sda 2>&1; udevadm info /dev/sda 2>&1 | grep -E "ID_FS|ID_PART|ID_MODEL=|ID_BUS"

Ubuntu sees the drive's hardware, but it isn't reading the contents. I can't run privileged commands (sudo needs a password), so I can't go further than this.

**What I found**
- The USB enclosure is detected: an ASMedia bridge, with a 512 GB disk (`/dev/sda`, model M303C) connected at SuperSpeed+.
- It loaded with the `uas` driver and no errors.
- `lsblk` shows no partition table and no filesystem on `sda`, so the problem is at the partition and filesystem level, not the cable or port.
- This is an ARM/Tegra (Jetson-style) system. That matters for the options below.

**Likely causes**
1. **The partition scheme isn't recognised.** If the drive was formatted on a Mac, it may use APFS or HFS+, which Ubuntu can't read by default. That doesn't fit with it also working on Windows, though, unless Windows reads it through a driver.
2. **UAS quirk.** Some ASMedia bridges misbehave under UAS on Linux. The kernel then reads the disk but returns bad data, which would explain the blank partition table.
3. **The kernel lacks exFAT or NTFS support.** Custom Tegra kernels sometimes omit it. This would normally still show a filesystem type in `lsblk`, though.

**Run these and send me the output**
```bash
sudo fdisk -l /dev/sda
sudo blkid /dev/sda /dev/sda1
sudo dmesg | grep -iE "sda|uas|sd 0:0"
cat /proc/filesystems | grep -iE "exfat|ntfs|apfs|hfs"
```

**Quick fix to try for the UAS issue**

Disable UAS for this device (the ID is `174c:55aa`), then replug the drive:
```bash
echo "options usb-storage quirks=174c:55aa:u" | sudo tee /etc/modprobe.d/disable-uas.conf
sudo update-initramfs -u && sudo reboot
```

**If it turns out to be exFAT or NTFS missing**
```bash
sudo apt install exfatprogs ntfs-3g
```

Please send the output of the `fdisk` and `dmesg` commands. They should show which cause it is.

## The UAS fix worked. Can you explain this problem more?

Glad it worked. Here's what was going on.

**What UAS is**
- USB Attached SCSI (UAS) is a newer protocol for USB storage. It supports command queuing and parallel transfers, so it's faster than the older "Bulk-Only Transport" (BOT) protocol, which the `usb-storage` driver handles.
- When you plug in a drive, the Linux kernel asks the enclosure which protocols it supports. If it advertises UAS, the kernel prefers it.

**Why it broke**
- Your enclosure uses an ASMedia bridge chip (`174c:55aa`, the ASM1051E/1053E/1153 family). Some firmware revisions of these chips advertise UAS but implement it badly.
- Windows and macOS either use their own UAS stacks, which tolerate the quirks, or fall back to BOT for these chips. Linux trusts the advertisement and runs into the bugs.
- The log matched this: the kernel logged `scsi host0: uas` and detected a 512 GB disk. After that, reads of the partition table failed or timed out, so `lsblk` showed no partitions or filesystem.
- Your Tegra-based system may also have made it worse. Its USB controller driver (`tegra-xusb`) and the bridge don't always get along on UAS stream handling.

**What the fix did**
- `options usb-storage quirks=174c:55aa:u` tells the kernel to ignore UAS for that specific vendor and product ID. It then binds the drive to the plain `usb-storage` driver (BOT) instead.
- It's slower in theory, with no command queuing. In practice, for a single spinning disk or SATA SSD behind a bridge like this, the difference is small, and a working drive beats a fast one that doesn't show up.
- The `u` flag is the "ignore UAS" quirk. Writing it to `/etc/modprobe.d/` makes it persist across reboots, and `update-initramfs -u` ensures it's applied early at boot.

**Good to know**
- The quirk applies only to that bridge ID, so other USB drives are unaffected.
- To confirm which driver is in use now, run `lsusb -t`. It should show `Driver=usb-storage` rather than `uas`.
- If you replace the enclosure or cable with a different chipset, you can delete `/etc/modprobe.d/disable-uas.conf` and rebuild the initramfs to go back to the default.
- Checking for a firmware update for the enclosure is another option, but ASMedia firmware is rarely user-updatable.