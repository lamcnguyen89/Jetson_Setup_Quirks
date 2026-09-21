# Connecting to UCF Wifi Network Using the Treminal

Because SSH is using the direct Ethernet link, you can configure Wi‑Fi without dropping the session. UCF’s main-campus SSID is `UCF_WPA2`, using PEAP with MSCHAPv2 and your NID credentials. [UCF’s Linux instructions](https://ucfsandbox.service-now.com/kb?id=kb_article_view&sysparm_article=KB0010807)

Run:

```bash
nmcli general status
nmcli device status

sudo nmcli radio wifi on
nmcli device wifi rescan
nmcli device wifi list
```

Identify the Wi‑Fi interface from `nmcli device status`; it will usually be `wlan0`. Then create the connection:

```bash
read -rp "UCF NID: " UCF_NID

sudo nmcli connection add \
  type wifi \
  ifname wlx289401631f06  \
  con-name "UCF-WiFi-JetsonNX" \
  ssid "UCF_WPA2"

sudo nmcli connection modify "UCF-WiFi-JetsonNX" \
  802-11-wireless-security.key-mgmt wpa-eap \
  802-1x.eap peap \
  802-1x.phase2-auth mschapv2 \
  802-1x.identity "$UCF_NID" \
  802-1x.system-ca-certs yes \
  802-1x.password-flags 2 \
  ipv4.method auto \
  connection.autoconnect yes
```

Replace `wlan0` if your interface has another name. Connect using:

```bash
sudo nmcli --ask connection up "UCF-WiFi-JetsonNX"
```

Enter your NID password when prompted. `password-flags 2` tells NetworkManager to request it rather than permanently storing it.

Verify the connection:

```bash
nmcli connection show --active
ip -br address show wlx289401631f06
ping -c 3 1.1.1.1
```

If authentication fails specifically because of certificate validation, UCF’s general setup page currently tells clients to choose “Don’t validate.” You can reproduce that with:

```bash
sudo nmcli connection modify "UCF-WiFi-Jetson" \
  802-1x.system-ca-certs no \
  802-1x.ca-cert ""

sudo nmcli --ask connection up "UCF-WiFi-Jetson"
```

That fallback is less secure, so try the system certificate configuration first. UCF confirms that `UCF_WPA2` requires your NID and NID password. [UCF Wi‑Fi guidance](https://ucfsandbox.service-now.com/kb/en?id=kb_article_view&sysparm_article=KB0010286)