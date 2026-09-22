# How to Install the Netgear AC1200 Wifi USB Driver

Netgear AC1200 USB Wi-Fi adapters (such as the A6150 or A6210) rely on Realtek chipsets—most commonly the **RTL8812AU** or **RTL8812BU**. Out of the box, Linux (and Jetson Linux/L4T) lacks native kernel drivers for these chips, which is why the dongle won't function immediately upon plugging it in.

To activate the adapter on a Jetson Orin running JetPack (Ubuntu), you need to compile and install DKMS drivers built for arm64 (`aarch64`).

---

### Step 1: Identify Your Chipset

Before compiling drivers, verify the vendor and product ID of your USB dongle:

1. Plug the Netgear AC1200 dongle into an available USB port.
2. Open a terminal and check `lsusb`:
```bash
lsusb

```


3. Look for your Netgear device entry.
* If the ID contains `0bda:8812` or `0846:9052` / `0846:9053`, it uses the **RTL8812AU** chipset.
* If the ID contains `0846:9055` or `0bda:b812`, it uses the **RTL8812BU** chipset.



---

### Step 2: Install Build Dependencies & Kernel Headers

Ensure you have build tools, `dkms`, and the correct L4T kernel headers installed.

```bash
sudo apt update
sudo apt install -y build-essential dkms git linux-headers-$(uname -r)

```

---

### Step 3: Clone and Install the Driver

Select the section below corresponding to your chipset ID from Step 1. Both repository maintainers (`morrownr`) provide stable DKMS drivers optimized for Linux kernels used across JetPack versions.

#### Option A: For RTL8812AU Chipsets

```bash
# Clone the out-of-tree RTL8812AU driver repo
git clone 
cd 8812au-20210629

# Run the automatic DKMS installer script
sudo ./install-driver.sh

```

#### Option B: For RTL8812BU Chipsets

```bash
# Clone the out-of-tree RTL8812BU driver repo
git clone https://github.com/morrownr/88x2bu-20210702.git
cd 8812bu-20210629

# Run the automatic DKMS installer script
sudo ./install-driver.sh

```

*During installation, the script will prompt you whether to enable driver options like concurrent AP/STA or edit configuration flags. Default settings work fine for general Wi-Fi connectivity.*

---

### Step 4: Load the Driver Module & Test

1. Reboot your Jetson Orin to load the newly compiled module:
```bash
sudo reboot

```


2. Once rebooted, verify that the module is active:
```bash
lsmod | grep 8812

```


3. Check if the wireless interface is created:
```bash
ip a

```


*(You should see a `wlan0` or `wlan1` interface).*
4. You can now connect to Wi-Fi networks using `nmcli` or the Ubuntu desktop network GUI:
```bash
sudo nmcli device wifi list
sudo nmcli device wifi connect "YOUR_SSID" password "YOUR_PASSWORD"

```



---