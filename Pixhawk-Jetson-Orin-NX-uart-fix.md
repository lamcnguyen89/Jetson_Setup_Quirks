# Fixing MAVProxy BAD_DATA on a Jetson Orin NX connected to a Pixhawk

## Result

MAVProxy received corrupted MAVLink data from an older mRo Pixhawk connected to the UART pins of a Jetson Orin NX on a Yahboom carrier board. The same flight controller setup worked on a Jetson Orin Nano.

Disabling DMA for the NX's UART and using PIO (interrupt-driven I/O) resolved the problem. The user confirmed the procedure below worked. The MAVProxy baud rate remained **57600**.

This documents the successful fix for this particular system. It does not establish that every Orin NX or Jetson Linux R36.5.2 installation has the same problem.

## Affected setup

| Component | Details |
| --- | --- |
| Companion computer | NVIDIA Jetson Orin NX |
| Carrier board | Yahboom; exact product model was not recorded |
| Flight controller | Older mRo Pixhawk; exact model and firmware were not recorded |
| Connection | Direct UART pins |
| Linux serial device | `/dev/ttyTHS1` |
| Baud rate | `57600` |
| Jetson Linux | R36.5.2, associated with JetPack 6.2.3 |
| Kernel | `5.15.199-tegra` |
| UART device-tree node | `/bus@0/serial@3100000` |
| Installed DTB | `/boot/dtb/kernel_tegra234-p3768-0000+p3767-0000-nv.dtb` |

Both Jetsons were described as running JetPack 6, but the Nano's exact release was not collected.

The connection command was:

```bash
sudo mavproxy.py --master=/dev/ttyTHS1 --baudrate=57600
```

The baud rate must match the flight controller's telemetry-port configuration; the controller's age alone does not determine it.

## Symptoms and diagnosis

MAVProxy repeatedly reported errors such as:

```text
MAV error: BAD_DATA {Bad prefix, data:['0', '0', '0', '0', ...]}
MAV error: BAD_DATA {invalid MAVLink CRC in msgID 0 0x0020 should be 0x8e16, data:['fd', '0', '0', '0', '0', '0', '0', '0', '0', '0', '20', '0']}
```

`Bad prefix` means the received bytes could not be parsed as a valid packet start. A CRC error means a possible packet failed its checksum check. `fd` is the MAVLink 2 start marker, but finding that byte alone does not prove the surrounding data is valid. See the [MAVLink packet format](https://mavlink.io/en/guide/serialization.html).

The repeated zeros suggested corruption in the receive path. NVIDIA had discussed a UART DMA issue on R36.5 and described disabling DMA as a workaround, along with a proposed IOMMU device-tree correction. That made DMA a candidate rather than an immediate diagnosis. See [NVIDIA's UART DMA discussion](https://forums.developer.nvidia.com/t/solved-uart-serial-port-not-working-after-upgradint-to-jetpack-6-2-2-orin-nano-nx/363837/7).

On this NX, the active UART configuration contained:

```text
status: okay
compatible: nvidia,tegra194-hsuart
dma-names: rx, tx
dmas: 00 00 00 ed 00 00 00 08 00 00 00 ed 00 00 00 08
iommus: absent
```

This showed DMA configured for RX and TX without an `iommus` property on that UART node. Those findings, followed by the successful PIO workaround, strongly implicated the UART DMA path. They do not independently prove the precise underlying driver or IOMMU defect on R36.5.2.

The [R36.5.2 release notes](https://docs.nvidia.com/jetson/archives/r36.5.2/ReleaseNotes/Jetson_Linux_Release_Notes_r36.5.2.pdf) did not explicitly identify this UART corruption issue as fixed or outstanding when checked.

## Diagnostic commands

Check the exact release rather than relying on the general label “JetPack 6”:

```bash
cat /etc/nv_tegra_release
uname -r
```

With MAVProxy stopped, inspect the UART and check for competing processes:

```bash
sudo dmesg | grep -Ei 'ttyTHS|3100000|serial|dma'
sudo fuser -v /dev/ttyTHS1
readlink -f /sys/class/tty/ttyTHS1/device/of_node
```

The NX reported:

```text
3100000.serial: ttyTHS1 at MMIO 0x3100000 ... is a TEGRA_UART
/sys/firmware/devicetree/base/bus@0/serial@3100000
```

No output from `fuser` means it found no process using that port at the time of the check. A log entry creating the `serial-getty` service group alone does not establish a serial-console conflict.

Inspect the active device tree with this read-only script:

```bash
python3 - <<'PY'
from pathlib import Path

node = Path('/sys/firmware/devicetree/base/bus@0/serial@3100000')
for name in ('status', 'compatible', 'dma-names', 'dmas', 'iommus'):
    p = node / name
    if not p.exists():
        print(f'{name}: absent')
        continue
    data = p.read_bytes()
    if name in ('status', 'compatible', 'dma-names'):
        value = data.rstrip(b'\x00').replace(b'\x00', b', ').decode()
        print(f'{name}: {value}')
    else:
        print(f'{name}: {data.hex(" ")}')
PY
```

Inspect the boot configuration and installed board identity:

```bash
cat /boot/extlinux/extlinux.conf
ls -l /boot/dtb/
tr '\0' '\n' < /proc/device-tree/model
tr '\0' '\n' < /proc/device-tree/compatible
```

This system reported `nvidia,p3768-0000+p3767-0000`, matching the installed DTB filename. Its original `primary` boot entry had no explicit `FDT` line.

## Working fix: disable UART DMA with a boot overlay

This procedure adds an overlay and a separate boot entry. It preserves the existing `primary` entry and does not overwrite the installed DTB. The overlay follows the [JetsonHacks UART workaround](https://github.com/jetsonhacks/jetson-orin-uart), using NVIDIA's `delete_prop` overlay mechanism to remove the UART DMA properties.

The paths below match the system diagnosed here. On another board, verify the UART node and installed DTB first. Preserve that board's existing `APPEND` line, especially its root partition identifier. Have access to the boot menu through a local console so you can select the original entry if the test fails to boot.

### 1. Create and compile the overlay

```bash
sudo apt install device-tree-compiler
mkdir -p ~/uart-pio-test
cd ~/uart-pio-test

cat > disable-uart1-dma.dts <<'EOF'
/dts-v1/;
/plugin/;

/ {
    overlay-name = "Disable UART1 DMA";
    fragment@0 {
        target-path = "/bus@0/serial@3100000";
        delete_prop = "dmas", "dma-names";

        __overlay__ {
            status = "okay";
        };
    };
};
EOF

dtc -@ -I dts -O dtb -o disable-uart1-dma.dtbo disable-uart1-dma.dts
sudo install -m 644 disable-uart1-dma.dtbo /boot/disable-uart1-dma.dtbo
```

If compilation fails, stop before modifying the boot configuration.

### 2. Back up the boot configuration

Run this before the first modification. The `-n` option prevents overwriting an existing backup if repeated:

```bash
sudo cp -an /boot/extlinux/extlinux.conf /boot/extlinux/extlinux.conf.before-uart-pio
sudo nano /boot/extlinux/extlinux.conf
```

If the system already has the working fix installed, do not create another test entry or treat its current configuration as the original backup.

### 3. Add a separate test boot entry

Leave the original `LABEL primary` entry intact. Append the following entry **once**. The `APPEND` value below is the original value from this specific NX; on another installation, copy its own original `APPEND` line instead. Keep the entire `APPEND` directive on one line.

```text
LABEL uart-pio
MENU LABEL UART PIO test
LINUX /boot/Image
INITRD /boot/initrd
FDT /boot/dtb/kernel_tegra234-p3768-0000+p3767-0000-nv.dtb
OVERLAYS /boot/disable-uart1-dma.dtbo
APPEND ${cbootargs} root=PARTUUID=5b5c49a6-2385-4d62-847c-39620b67d90c rw rootwait rootfstype=ext4 mminit_loglevel=4 console=ttyTCU0,115200 firmware_class.path=/etc/firmware fbcon=map:0 video=efifb:off console=tty0 efi=runtime pci=pcie_bus_perf nvme.use_threaded_interrupts=1 nv-auto-config
```

Change the existing default selection near the top from:

```text
DEFAULT primary
```

to:

```text
DEFAULT uart-pio
```

Save the file and reboot:

```bash
sudo reboot
```

### 4. Verify PIO and retry MAVProxy

After reboot:

```bash
sudo dmesg | grep -Ei '3100000|PIO'
```

Look for messages indicating:

```text
RX in PIO mode
TX in PIO mode
```

You can also rerun the active device-tree inspection script. The overlay is intended to remove `dmas` and `dma-names` from the UART node.

Retry the original connection:

```bash
sudo mavproxy.py --master=/dev/ttyTHS1 --baudrate=57600
```

Confirm that MAVProxy receives valid telemetry and that the repeated `BAD_DATA` errors have stopped. The user reported that this procedure worked; separate post-fix logs were not collected in the discussion.

## Rollback

If the test entry fails to boot, select **primary kernel** from the boot menu. Once logged in, restore the original configuration:

```bash
sudo cp -a /boot/extlinux/extlinux.conf.before-uart-pio /boot/extlinux/extlinux.conf
sudo reboot
```

Alternatively, edit `extlinux.conf` and change `DEFAULT uart-pio` back to `DEFAULT primary`. The overlay file can remain in `/boot`; the preserved original entry does not reference it.

## If the errors return

- Check whether an update or a Jetson-IO change selected another boot entry or removed the overlay reference. Recheck PIO mode before changing MAVProxy settings.
- If PIO is active but corruption persists, check the baud rate at both ends, TX/RX wiring, shared ground, carrier-board voltage compatibility, and other processes opening the serial port.
- A USB connection to the Pixhawk can help distinguish the Jetson header UART path from broader flight-controller or MAVProxy issues.

PIO changes how the CPU handles UART traffic. This workaround was successful at 57600 baud in this setup; performance at higher rates was not tested here.
