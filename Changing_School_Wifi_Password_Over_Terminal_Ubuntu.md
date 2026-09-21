# How to change UCF Wifi password in Ubuntu over the Terminal

To change the password for a saved Wi-Fi network in Ubuntu terminal, you can use **NetworkManager's command-line tool (`nmcli`)**.

School and university Wi-Fi networks typically use **WPA-Enterprise (802.1X)** authentication rather than standard WPA-Personal.

### Step 1: Find your saved connection name

List all saved network connections to find the exact name (SSID or Profile ID) of your school Wi-Fi:

```bash
nmcli connection show

```

*Look for your school network in the **NAME** column (e.g., `SchoolWiFi` or `eduroam`).*

---

### Step 2: Update the password

#### Option A: For Enterprise Networks (Most common for schools / 802.1X)

School networks usually require a username and password (EAP protocol). Update the 802.1X enterprise password setting:

```bash
nmcli connection modify "Your_Network_Name" 802-1x.password "Your_New_Password"

```

#### Option B: For Standard Wi-Fi Networks (WPA/WPA2/WPA3 Personal)

If your school network uses a simple shared Wi-Fi password/passphrase:

```bash
nmcli connection modify "Your_Network_Name" wifi-sec.psk "Your_New_Password"

```

*(Be sure to replace `"Your_Network_Name"` and `"Your_New_Password"` with your actual network name and new password, keeping the quotes).*

---

### Step 3: Apply the changes & reconnect

Reconnect to the network so NetworkManager attempts authentication with the new password:

```bash
nmcli connection down "Your_Network_Name" && nmcli connection up "Your_Network_Name"

```

### Verification

To check if the step was successful, verify that the connection status returns successfully connected or check your active connection status with:

```bash
nmcli connection show --active

```

If your school network appears under active connections, the authentication succeeded.


## Why is entering my password in the commands screwed up?
That error happens because double quotes (`" "`) in Bash still allow **history expansion**. If your password contains an exclamation mark (`!`), Bash interprets it as an attempt to re-run a previous terminal command (a history event) rather than a literal character.

### Solution 1: Use single quotes

Enclose your password in **single quotes (`' '`)**. Single quotes instruct Bash to treat every character inside literally, disabling history expansion and variable parsing.

```bash
nmcli connection modify "Your_Network_Name" 802-1x.password 'Your_New_Password!'

```

*(Note: If your password itself contains a single quote, use Solution 2 instead).*

---

### Solution 2: Escape the exclamation mark

If you prefer double quotes, add a backslash (`\`) right before any exclamation mark (`!`) to escape it:

```bash
nmcli connection modify "Your_Network_Name" 802-1x.password "Your_New_Password\!"

```

---

### Solution 3: Enter the password interactively (Most secure)

To avoid shell character issues entirely and prevent your password from appearing in your bash history file (`~/.bash_history`), use `nmcli`'s interactive ask mode:

```bash
nmcli --ask connection modify "Your_Network_Name" 802-1x.password

```

Terminal will prompt you to type the password securely without exposing special characters to the shell parser.

---

### Verification

Run the command to reconnect:

```bash
nmcli connection up "Your_Network_Name"

```

To verify it succeeded without errors, ensure the command prints `Connection successfully activated` and run:

```bash
nmcli connection show --active

```

If your network is listed in the output, the password was accepted and you are connected.