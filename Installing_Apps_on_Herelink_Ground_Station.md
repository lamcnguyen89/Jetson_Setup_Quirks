# Installing Apps on the Herelink Ground Station

Sideloading applications onto a **CubePilot Herelink Ground Station Unit (GCS)** using the Android Debug Bridge (ADB) follows standard Android sideloading procedures.

---

### Step 1: Install ADB on Your Computer

If you do not already have `adb` installed on your host computer, set up the SDK Platform Tools:

* **Windows:** Download the [Android SDK Platform-Tools](https://developer.android.com/tools/releases/platform-tools) ZIP file. Extract it to a folder (e.g., `C:\platform-tools`) and optionally add it to your system PATH.
* **macOS:** Install via Homebrew:
```bash
brew install android-platform-tools

```


* **Linux (Debian/Ubuntu):** Install via `apt`:
```bash
sudo apt update && sudo apt install android-tools-adb

```



---

### Step 2: Enable Developer Mode & USB Debugging

1. Power on the Herelink Ground Station.
2. Open **Settings** on the unit.
3. Scroll down and select **About Phone** (or **About Device**).
4. Locate **Build Number** and tap it **7 times** until you see the prompt: *"You are now a developer!"*
5. Go back to the main **Settings** menu and enter **Developer options**.
6. Scroll to **USB debugging** and toggle it **ON**.

---

### Step 3: Connect and Verify Connection

1. Connect the Herelink Ground Station to your PC using a **Micro-USB data cable** (ensure it is a data cable, not power-only).
2. Open a terminal (Mac/Linux) or Command Prompt/PowerShell (Windows).
3. Verify that the Herelink is recognized by executing:
```bash
adb devices

```


4. Look at the Herelink screen. If a pop-up appears asking to **"Allow USB debugging?"**, check *Always allow from this computer* and tap **OK**.
5. Running `adb devices` again should display your device with the status `device` (rather than `unauthorized`).

---

### Step 4: Install the Application (APK)

1. Download the `.apk` file you wish to install onto your computer.
2. Install the app via ADB by running:
```bash
adb install /path/to/your_app.apk

```

**Useful Flags:**
* If you are updating an existing application, add the **`-r`** flag to keep existing application data:

```bash
    adb install -r /path/to/your_app.apk
```
* If you need to allow version downgrades, add the **`-d`** flag:

```bash
    adb install -r -d /path/to/your_app.apk 
``` 

3. Once the terminal displays **`Success`**, the application will appear in your Herelink app drawer.