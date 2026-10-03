# How to Connect to Wifi using Terminal

## Normal Wifi

To connect your Ubuntu host computer to Wi-Fi while SSHed into it over Ethernet, you can use **Netplan** (standard on modern Ubuntu Server/Desktop) or **NetworkManager** (`nmcli`).

Here is the step-by-step process using `nmcli`, which is the most reliable and immediate method over SSH.

---

### Step 1: Identify Your Wi-Fi Interface Name

Run the following command to list your network interfaces:

```bash
ip link show

```

Look for your wireless interface, usually starting with `wlan0`, `wlp2s0`, or `wlp3s0`.

*Example verification:* Verify the interface name is listed and not set to `DOWN` state.

---

### Step 2: Ensure Wi-Fi Is Enabled

Check if radio transmitters are blocked:

```bash
nmcli radio wifi

```

If it returns `disabled`, turn it on:

```bash
nmcli radio wifi on

```

---

### Step 3: Scan for Available Wi-Fi Networks

Scan for surrounding access points to confirm your target network is visible:

```bash
nmcli device wifi list

```

---

### Step 4: Connect to Your Wi-Fi Network

Run the following command, replacing `Your_SSID` and `Your_Password` with your network details:

```bash
sudo nmcli device wifi connect "Your_SSID" password "Your_Password"

```

*Verification:* If successful, `nmcli` will output:
`Device 'wlan0' successfully activated with '...'`

---

### Step 5: Verify Your Connection and IP Address

To verify that the Wi-Fi connection is active and has received an IP address, run:

```bash
ip addr show dev wlan0

```

*(Replace `wlan0` with your specific wireless interface name if different).*

You should see an `inet` entry with a valid IP address assigned by your Wi-Fi router. You can also test internet connectivity directly from the host:

```bash
ping -c 3 8.8.8.8

```

---

### Alternative Method: Using Netplan (YAML Configuration)

If `nmcli` is not available or you are using Ubuntu Server without NetworkManager running:

1. Locate your Netplan configuration file:
```bash
ls /etc/netplan/

```


2. Edit the configuration file (e.g., `50-cloud-init.yaml` or `01-netcfg.yaml`):
```bash
sudo nano /etc/netplan/01-netcfg.yaml

```


3. Add your Wi-Fi interface settings under `network.wifis`:
```yaml
network:
  version: 2
  renderer: networkd
  wifis:
    wlan0:
      dhcp4: true
      access-points:
        "Your_SSID":
          password: "Your_Password"

```


4. Apply the configuration:
```bash
sudo netplan apply

```

## School/ Institutional Wifi

To connect to `UCF_WPA2` directly from the Ubuntu terminal, you can use **`nmcli`** (NetworkManager CLI) or configure **Netplan / `wpa_supplicant**`. `nmcli` is the fastest and most reliable method over terminal.

---

### Method 1: Using `nmcli` (Recommended)

#### 1. Identify Your Wireless Interface

```bash
ip link show

```

Look for your Wi-Fi interface name (typically `wlan0`, `wlp2s0`, or `wlp3s0`).

#### 2. Ensure Wi-Fi Radio is Enabled

```bash
nmcli radio wifi on

```

#### 3. Connect to `UCF_WPA2`

Run the following command, replacing `wlan0` with your Wi-Fi interface, and `YOUR_NID` / `YOUR_NID_PASSWORD` with your UCF credentials:

```bash
sudo nmcli device wifi connect "UCF_WPA2" \
  ifname wlan0 \
  username "YOUR_NID" \
  password "YOUR_NID_PASSWORD" \
  802-1x.eap peap \
  802-1x.phase2-auth mschapv2

```

*Verification:* If successful, `nmcli` will return:
`Device 'wlan0' successfully activated with '...'`

---

### Method 2: Using Netplan (Ubuntu Server / Headless)

If you are on Ubuntu Server without `nmcli`, edit your Netplan configuration file:

#### 1. Open Netplan Config File

```bash
sudo nano /etc/netplan/01-netcfg.yaml

```

#### 2. Add `UCF_WPA2` Under `wifis`

Replace `wlan0`, `YOUR_NID`, and `YOUR_NID_PASSWORD` with your details:

```yaml
network:
  version: 2
  renderer: networkd
  wifis:
    wlan0:
      dhcp4: true
      access-points:
        "UCF_WPA2":
          auth:
            key-management: wpa-eap
            eap-method: peap
            identity: "YOUR_NID"
            password: "YOUR_NID_PASSWORD"
            phase2-auth: mschapv2

```

#### 3. Apply the Configuration

```bash
sudo netplan apply

```

---

### Method 3: Connecting to `eduroam` via Terminal

If you prefer using `eduroam` over terminal:

```bash
sudo nmcli device wifi connect "eduroam" \
  ifname wlan0 \
  username "YOUR_NID@ucf.edu" \
  password "YOUR_NID_PASSWORD" \
  802-1x.eap peap \
  802-1x.phase2-auth mschapv2

```

---

### Step 4: Verify Connection & Internet

To confirm you have received an IP address and are connected:

```bash
ip addr show dev wlan0
ping -c 3 8.8.8.8

```