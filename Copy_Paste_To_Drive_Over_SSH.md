# Checking for External USB Drives and Copying Files to Them Over SSH

Run the commands in sections 1, 2 and 4 on the **remote machine** (e.g. the Jetson) after `ssh user@host`.
Section 3 shows how to copy from your **local machine**.

## 1. Check for available external USB drives

```bash
lsblk -f                                   # list block devices, filesystems, labels, mount points
lsblk -o NAME,SIZE,TYPE,FSTYPE,LABEL,MOUNTPOINT,TRAN,MODEL   # TRAN=usb marks USB drives
df -h                                      # mounted filesystems and free space
lsusb                                      # USB devices the system can see
sudo fdisk -l                              # partition tables (look for /dev/sda, /dev/sdb, ...)
sudo dmesg | tail -30                      # kernel messages right after plugging the drive in
```

- An external drive normally shows up as `sda`, `sdb`, ... with partitions such as `sda1`.
- If the partition has a `MOUNTPOINT` (often `/media/<user>/<LABEL>`), it is already mounted and you can skip to section 3.
- If the drive appears in `lsusb` but has no partitions or filesystem in `lsblk`, see [Fixing_External_Drive_Issue.md](./Fixing_External_Drive_Issue.md) (UAS quirk).

## 2. Mount the drive (if it is not mounted)

```bash
sudo mkdir -p /mnt/external
sudo mount /dev/sda1 /mnt/external

# NTFS / exFAT (install support first if needed: sudo apt install ntfs-3g exfatprogs)
sudo mount -t ntfs-3g /dev/sda1 /mnt/external
sudo mount -t exfat /dev/sda1 /mnt/external

# Make the drive writable by your user (needed for exFAT/NTFS/FAT)
sudo mount -o uid=$(id -u),gid=$(id -g) /dev/sda1 /mnt/external

# Confirm it is mounted and has free space
df -h /mnt/external
```

Replace `/dev/sda1` with the partition found in section 1. For ext4 drives, fix write access with `sudo chown -R $USER:$USER /mnt/external`.

## 3. Copy files and folders to the drive over SSH

### From your local machine to the remote drive

```bash
# Single file
scp file.txt user@host:/mnt/external/

# Entire folder
scp -r my_folder user@host:/mnt/external/

# rsync (recommended: resumable, skips unchanged files, shows progress)
rsync -avh --progress my_folder/ user@host:/mnt/external/my_folder/
```

### From the remote drive back to your local machine

```bash
scp user@host:/mnt/external/file.txt .
scp -r user@host:/mnt/external/my_folder .
rsync -avh --progress user@host:/mnt/external/my_folder/ ./my_folder/
```

### While already logged into the remote machine

```bash
cp file.txt /mnt/external/
cp -r ~/my_folder /mnt/external/
rsync -avh --progress ~/my_folder/ /mnt/external/my_folder/
```

Notes:
- A trailing `/` on the rsync source copies the folder's *contents*; without it, the folder itself is copied.
- For a non-default SSH port use `scp -P 2222 ...` or `rsync -e "ssh -p 2222" ...`.
- FAT/exFAT/NTFS drives don't store Linux permissions. Use `rsync -rvh --progress --no-perms --no-owner --no-group` to avoid errors.
- For long transfers run inside `tmux` or `screen` so a dropped SSH connection doesn't kill the copy.
- A path on the remote drive with spaces needs quoting, e.g. `scp file.txt "user@host:/media/user/My Drive/"`.

## 4. Verify and safely eject

```bash
ls -lh /mnt/external                       # confirm the files arrived
du -sh /mnt/external/my_folder             # compare size with the source
sync                                       # flush cached writes to the drive
sudo umount /mnt/external                  # unmount before unplugging
```

If `umount` says the target is busy, find the culprit with `lsof +f -- /mnt/external` (or `cd ~` if your shell is inside the drive).
