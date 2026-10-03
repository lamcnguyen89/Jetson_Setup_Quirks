The issue depends on which host OS you are plugged into. Batocera formats its main storage partition (`SHARE`) as **ext4** or **BTRFS** by default, which Windows cannot write to natively, and Linux hosts mount with strict root ownership.

Here is how to resolve it based on your setup:

---

### Method 1: If You are on Windows or macOS

Windows cannot natively write to Linux file systems (ext4/BTRFS) and will either prompt to format or block writes.

1. **Use WSL2 or a Third-Party Driver (Windows):**
* Install **Paragon Linux File Systems for Windows** or **BTRFS for Windows** (depending on the filesystem used on the `SHARE` partition).


2. **Best Alternative (Network Transfer):**
* Boot the NVMe drive in its target system/console.
* Connect it to your local network via Ethernet or Wi-Fi.
* Open Windows File Explorer or macOS Finder and navigate to `\\BATOCERA` (or `smb://batocera.local`).
* Drag and drop your ROMs directly into the `roms` subdirectories.



---

### Method 2: If You are on Linux (Ubuntu/Debian)

When plugging an external ext4/BTRFS drive into Linux, the files are owned by `root`, preventing regular user writes.

#### Option A: Change Ownership of the Mounted Partition

Find where your system mounted the drive (usually under `/media/$USER/SHARE` or `/run/media/$USER/SHARE`) and take ownership:

```bash
# Identify where the partition is mounted
df -h | grep -i share

# Fix ownership so your user account can write freely
sudo chown -R $USER:$USER /path/to/mounted/SHARE

```

#### Option B: Remount as Read-Write

If the filesystem dirty flag was set or mounted read-only due to an unclean unmount:

```bash
# Check block devices to find the partition (e.g., /dev/nvme0n1p2 or /dev/sdb2)
lsblk

# Remount with explicit read-write permissions
sudo mount -o remount,rw /dev/sdX2 /path/to/mounted/SHARE

```

---

### Method 3: Transferring Directly on the Batocera System

If you boot into Batocera with the NVMe inside the device, you can use Batocera's built-in file manager:

1. Press **F1** on a connected keyboard while in the Batocera main menu to launch PCManFM (the built-in file manager).
2. Use the left sidebar to access local drives or external USB storage containing your ROMs.
3. Drag and drop the game files directly into `/userdata/roms/`.