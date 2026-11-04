# Copying Files to an External Drive on a Host Over SSH

## 1. Find and mount the drive (on the host)

```bash
ssh user@host
lsblk -f                      # find the partition, e.g. /dev/sda1
sudo mkdir -p /mnt/external
sudo mount /dev/sda1 /mnt/external
# NTFS/exFAT drives: sudo mount -t ntfs-3g|exfat /dev/sda1 /mnt/external
# Often auto-mounted already under /media/<user>/<LABEL> (check with `lsblk` / `df -h`)
# Ensure write permission: sudo mount -o uid=$(id -u),gid=$(id -g) /dev/sda1 /mnt/external
```

## 2. Copy from your local system to the host's drive (run on your local machine)

```bash
# Single file
scp file.txt user@host:/mnt/external/

# Directory
scp -r mydir user@host:/mnt/external/

# rsync (recommended: resumable, skips unchanged files, shows progress)
rsync -avh --progress mydir/ user@host:/mnt/external/mydir/
```

Notes: trailing `/` on the source copies contents; without it, the folder itself. 
Use `-P 2222` (scp) or `-e "ssh -p 2222"` (rsync) for non-default ports.
FAT/exFAT/NTFS drives don't keep Linux permissions; use `rsync -rvh --progress --no-perms --no-owner --no-group`.

## 3. Copy while already logged into the host

```bash
cp -r ~/mydir /mnt/external/
rsync -avh --progress ~/mydir/ /mnt/external/mydir/
```

For long transfers, use `tmux`/`screen` so an SSH drop doesn't kill it.

## 4. Verify and safely eject (on the host)

```bash
sync
ls -l /mnt/external
sudo umount /mnt/external
```
