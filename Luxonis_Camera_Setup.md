# Install DepthAI on Jetson

1. APT Update/Upgrade

```bash
sudo apt update && sudo apt upgrade
sudo reboot now
```

2. SWAP Size change

```bash
# Disable ZRAM:
sudo systemctl disable nvzramconfig
# Create 4GB swap file
sudo fallocate -l 4G /mnt/4GB.swap
sudo chmod 600 /mnt/4GB.swap
sudo mkswap /mnt/4GB.swap

```

Try these if you have issues:

```bash 
sudo nano /etc/fstab
# Add this line at the bottom of the file
/mnt/4GB.swap swap swap defaults 0 0
# Reboot
sudo reboot now
```

3. Dependencies

```bash
sudo wget -qO- https://docs.luxonis.com/install_dependencies.sh | bash
```

4. DepthAI Repo

```bash
#Clone github repository
git clone https://github.com/luxonis/depthai-python.git
cd depthai-python
sudo apt install python3-venv
python3 -m venv depthai
source depthai/bin/activate
python3 examples/install_requirements.py
```

7. .bashrc

```bash
echo "export OPENBLAS_CORETYPE=ARMV8" >> ~/.bashrc
```

