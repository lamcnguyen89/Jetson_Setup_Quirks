Your installed DTB filename matches the board IDs reported by Linux. We can now **test disabling UART DMA**, using that existing file and preserving your current boot entry.

This is a diagnostic workaround, not yet a confirmed fix for your R36.5.2 installation. The overlay below uses the same approach as the [JetsonHacks UART workaround](https://github.com/jetsonhacks/jetson-orin-uart).

1. **Create and compile the overlay** on the NX:

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

   If compilation fails, stop before changing the boot configuration.

2. **Back up and edit the boot configuration:**

   ```bash
   sudo cp -a /boot/extlinux/extlinux.conf /boot/extlinux/extlinux.conf.before-uart-pio
   sudo nano /boot/extlinux/extlinux.conf
   ```

   Leave the existing `LABEL primary` entry intact. Append this new entry, keeping its entire `APPEND` line on one line:

   ```text
   LABEL uart-pio
   MENU LABEL UART PIO test
   LINUX /boot/Image
   INITRD /boot/initrd
   FDT /boot/dtb/kernel_tegra234-p3768-0000+p3767-0000-nv.dtb
   OVERLAYS /boot/disable-uart1-dma.dtbo
   APPEND ${cbootargs} root=PARTUUID=5b5c49a6-2385-4d62-847c-39620b67d90c rw rootwait rootfstype=ext4 mminit_loglevel=4 console=ttyTCU0,115200 firmware_class.path=/etc/firmware fbcon=map:0 video=efifb:off console=tty0 efi=runtime pci=pcie_bus_perf nvme.use_threaded_interrupts=1 nv-auto-config
   ```

   Change the top line `DEFAULT primary` to:

   ```text
   DEFAULT uart-pio
   ```

3. **Reboot and verify:**

   Have local boot-console access available so you can select **primary kernel** if the test entry fails to boot.

   ```bash
   sudo reboot
   ```

   After reboot:

   ```bash
   sudo dmesg | grep -Ei '3100000|PIO'
   ```

   Look for **RX in PIO mode** and **TX in PIO mode**, then retry:

   ```bash
   sudo mavproxy.py --master=/dev/ttyTHS1 --baudrate=57600
   ```

If PIO is active and the errors disappear, that strongly implicates the UART DMA path. If errors remain, send the new log output.

To undo the boot change:

```bash
sudo cp -a /boot/extlinux/extlinux.conf.before-uart-pio /boot/extlinux/extlinux.conf
sudo reboot
```