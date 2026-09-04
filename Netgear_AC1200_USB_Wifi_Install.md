# How to Install the Netgear AC1200 Wifi USB Driver

Yes. For a Jetson—especially your AGX Thor—the main difference from normal Ubuntu is that you generally **should not install generic Ubuntu kernel headers**. Jetson uses NVIDIA's `-tegra` kernel, so you need NVIDIA's matching headers before compiling the Realtek module. NVIDIA explicitly provides `nvidia-l4t-kernel-headers` for building external modules. ([NVIDIA Docs][1])

### 1. Confirm the adapter and kernel

Plug in the RTL8812BU and run:

```bash
lsusb
uname -r
uname -m
```

You're looking for something resembling:

```text
ID 0bda:b812 Realtek Semiconductor Corp.
```

`0bda:b812` is one of the standard RTL8812BU/RTL8822BU IDs. ([GitHub][2])

On your AGX Thor, I would expect something roughly like:

```text
6.8.12-tegra
aarch64
```

### 2. See if the existing kernel driver already works

Try:

```bash
sudo modprobe rtw88_8822bu
```

Then:

```bash
ip link
```

and:

```bash
iw dev
```

If you suddenly get something such as:

```text
wlan1
```

or:

```text
wlx0013ef......
```

you may already be done.

You can also check:

```bash
lsmod | grep rtw
```

The upstream Linux `rtw88` family supports RTL8812BU/RTL8822BU, though support in older kernels such as 6.8 isn't as mature as it is in newer kernels. The current Realtek driver maintainer specifically recommends the in-kernel `rtw88` implementation on kernel 6.12+. ([GitHub][3])

### 3. If it doesn't work, install the Jetson build dependencies

On Jetson, do:

```bash
sudo apt update
sudo apt install -y \
    build-essential \
    dkms \
    git \
    iw \
    rfkill \
    nvidia-l4t-kernel-headers
```

Then verify that the build directory exists:

```bash
ls -l /lib/modules/$(uname -r)/build
```

You want it to point somewhere under `/usr/src/`.

For example:

```text
/lib/modules/6.8.12-tegra/build -> /usr/src/linux-headers-...
```

This is important. **Don't blindly use:**

```bash
sudo apt install linux-headers-$(uname -r)
```

on Jetson. Ubuntu's repositories often won't contain headers for NVIDIA's custom `-tegra` kernel. NVIDIA's `nvidia-l4t-kernel-headers` package is intended for precisely this purpose. ([NVIDIA Docs][1])

### 4. Install the RTL8812BU driver

A well-maintained driver for this chipset is `morrownr/88x2bu-20210702`. It explicitly supports **RTL8812BU**, **aarch64/ARM64**, Ubuntu 24.04/kernel 6.8, DKMS, monitor mode, AP mode, and packet injection. ([GitHub][3])

Run:

```bash
cd ~
git clone https://github.com/morrownr/88x2bu-20210702.git
cd 88x2bu-20210702
```

Then:

```bash
sudo ./install-driver.sh
```

If it isn't executable:

```bash
chmod +x install-driver.sh
sudo ./install-driver.sh
```

The installer uses DKMS when available, meaning the driver can be rebuilt when your Jetson kernel gets updated. ([GitHub][4])

Then reboot:

```bash
sudo reboot
```

### 5. Verify after reboot

Run:

```bash
lsmod | grep 88x2bu
```

You should see something similar to:

```text
88x2bu
```

Then:

```bash
iw dev
```

and:

```bash
nmcli device
```

You should have an additional Wi-Fi interface, for example:

```text
DEVICE            TYPE      STATE
wlP1p1s0          wifi      connected
wlx1cbfce64a99c   wifi      disconnected
```

You can scan with:

```bash
nmcli dev wifi list
```

### 6. One Jetson-specific check if compilation fails

If `install-driver.sh` gives an error involving:

```text
/lib/modules/6.8.12-tegra/build
```

or:

```text
No such file or directory
```

run these and send me the output:

```bash
uname -a
dpkg -l | grep nvidia-l4t-kernel
ls -l /lib/modules/$(uname -r)/
ls -l /usr/src/
```

That would almost certainly indicate a **Jetson kernel/header mismatch**, rather than a problem with the RTL8812BU itself.

Also send me:

```bash
lsusb
```

because I can verify the exact USB vendor/device ID against the driver's supported list. There are quite a few RTL8812BU adapters sold under different brands and USB IDs. ([GitHub][2])

For your likely **AGX Thor + Ubuntu 24.04 + 6.8.x-tegra** configuration, I'd use the `88x2bu` DKMS approach above rather than fighting the older in-kernel `rtw88` implementation. Once NVIDIA eventually moves that Jetson to a sufficiently recent kernel (6.12+), I'd switch back to the standard in-kernel `rtw88` driver. ([GitHub][3])

[1]: https://docs.nvidia.com/jetson/archives/r38.4/DeveloperGuide/SD/SoftwarePackagesAndTheUpdateMechanism.html?utm_source=chatgpt.com "Software Packages and the Update Mechanism — NVIDIA Jetson Linux Developer Guide"
[2]: https://github.com/morrownr/88x2bu-20210702/blob/main/supported-device-IDs?utm_source=chatgpt.com "88x2bu-20210702/supported-device-IDs at main · morrownr/88x2bu-20210702 · GitHub"
[3]: https://github.com/morrownr/88x2bu-20210702?utm_source=chatgpt.com "GitHub - morrownr/88x2bu-20210702: Linux Driver for USB WiFi Adapters that are based on the RTL8812BU and RTL8822BU Chipsets - v5.13.1 · GitHub"
[4]: https://github.com/morrownr/88x2bu-20210702/blob/main/install-driver.sh?utm_source=chatgpt.com "88x2bu-20210702/install-driver.sh at main · morrownr/88x2bu-20210702 · GitHub"
