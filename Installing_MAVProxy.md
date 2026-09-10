Installing MAVProxy system-wide directly on your Jetson without isolated virtual environments requires installing system C/C++ build dependencies via `apt`, followed by installing MAVProxy using `--break-system-packages` (for JetPack 6 / Ubuntu 24.04/22.04 with Python 3.10+) or user-level pip flags.

1. **Install System Dependencies:** Required C libraries, GUI bindings, and Python development headers.
Update apt repositories and install all necessary underlying dependencies:

```bash
sudo apt update
sudo apt install -y python3-dev python3-pip python3-opencv python3-wxgtk4.0 \
                    python3-matplotlib python3-lxml python3-pygame libxml2-dev \
                    libxslt1-dev git

```

> **Note:** If you are running a headless Jetson configuration (no monitor/GUI attached over SSH), you can omit `python3-wxgtk4.0`.


2. **Install MAVProxy and PyMAVLink System-Wide:**
Depending on the JetPack/Ubuntu version running on your Jetson, Python's PEP 668 may block global `pip` installations.

**For JetPack 6 (Ubuntu 22.04/24.04):**
Use the `--break-system-packages` flag to allow system-wide PIP modifications:

```bash
sudo python3 -m pip install PyYAML pymavlink mavproxy --break-system-packages
sudo /usr/bin/python3 -m pip install future --break-system-packages

```

**For JetPack 5 (Ubuntu 20.04) or User-Local Install:**
Alternatively, install to your local user directory without needing `sudo`:

```bash
python3 -m pip install PyYAML pymavlink mavproxy --user
sudo /usr/bin/python3 -m pip install future --break-system-packages

```


3. **Configure PATH and Serial Permissions:** Ensures executable visibility and raw UART/USB access.
Add the user binary directory to your `PATH` if using `--user`, and grant your user access to telemetry ports (`/dev/ttyTHS*` or `/dev/ttyUSB*`) without requiring `sudo`:

```bash
# Add user bin path to bashrc
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.bashrc
source ~/.bashrc

# Add user to dialout group for serial/UART communication
sudo usermod -a -G dialout $USER

# Disable modemmanager to prevent serial port lockouts
sudo apt remove -y modemmanager

```

*Reboot or log out and back in for group permission changes to apply.*


4. **Verify Installation:**
Confirm MAVProxy is accessible from your global environment:

```bash
mavproxy.py --version

```