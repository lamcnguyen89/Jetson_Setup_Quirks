To dual boot Raspberry Pi OS from a micro SD card and Batocera from an NVMe drive on a Raspberry Pi 5, the cleanest approach is to let the Pi's hardware bootloader handle target selection based on **device detection priority** using the NVMe Base/HAT and the Pi 5 EEPROM settings.

---

## Prerequisites & EEPROM Setup

Ensure your Raspberry Pi 5 EEPROM is updated to support NVMe boot and configured with the correct boot order.

1. **Boot into Pi OS on the SD Card.**
2. Open a terminal and run the configuration tool:
```bash
sudo raspi-config

```


3. Navigate to **Advanced Options** → **Boot Order** → Select **NVMe/USB Boot**.
4. Alternatively, edit the EEPROM config directly:
```bash
sudo rpi-eeprom-config --edit

```


Ensure the `BOOT_ORDER` variable includes both `1` (microSD) and `6` (NVMe):
```text
BOOT_ORDER=0xf416

```


* *`6` corresponds to NVMe, and `1` corresponds to microSD. Execution runs right-to-left (`6` first, then `1`).*



---

## Dual-Boot Behavior

Because of how the Pi 5 bootloader processes `BOOT_ORDER`:

### Mode A: Automatic Switching (Physical Media Insertion)

* **Boot Batocera (NVMe):** Power on the Pi with the NVMe attached and the SD card **removed** (or keep NVMe prioritized as `6`).
* **Boot Pi OS (SD Card):** If `BOOT_ORDER=0xf614` is set instead, the Pi looks for a microSD card first. Inserting the SD card boots Raspberry Pi OS; removing it forces the Pi to failover to the NVMe drive to boot Batocera.

---

### Mode B: On-Screen Boot Menu (PINN / NOOBS Style)

If you want an on-screen menu at startup without physically removing the SD card:

1. Flash **PINN** (an expanded version of NOOBS designed for Raspberry Pi) to your microSD card.
2. Boot into PINN with the NVMe drive connected via PCIe.
3. Use the PINN installer to target and install:
* **Raspberry Pi OS** onto the **microSD card** (`/dev/mmcblk0`).
* **Batocera** onto the **NVMe drive** (`/dev/nvme0n1`).


4. At every power-on, PINN displays an interactive menu to choose between OS options before loading the respective kernel.

---

## PCIe Gen 3 Speed Optimization

By default, the Pi 5 runs PCIe at Gen 2 speeds. To enable full NVMe performance for Batocera and Pi OS:

Add the following line to `config.txt` on the boot partition of both drives:

```ini
dtparam=pciex1_no_max_speed=1

```
