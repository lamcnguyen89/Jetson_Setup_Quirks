# How to install ROS 2 for Ubuntu 24

The targeted version of ROS 2 for **Ubuntu 24.04 LTS (Noble Numbat)** is **ROS 2 Jazzy Jalisco**.

Follow these steps to install ROS 2 Jazzy Desktop using `apt` debian packages:

1. **Set UTF-8 Locale:** System Configuration.
Ensure your locale supports UTF-8 to prevent string encoding issues in ROS 2:

```bash
sudo apt update && sudo apt install locales -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

```

*Verification:* Run `locale` in your terminal and verify that `LANG=en_US.UTF-8` is displayed.


2. **Enable Repositories & Install Sources:** Package Manager.
Enable the Ubuntu Universe repository and install the official ROS 2 GPG keys and repository configuration:

```bash
sudo apt install software-properties-common curl -y
sudo add-apt-repository universe -y

# Download and install the ros2-apt-source configuration package
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F'"' '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}_$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb

```

*Verification:* Check if `/etc/apt/sources.list.d/ros2.sources` (or equivalent file) now exists.


3. **Update Apt Caches & Install ROS 2 Packages:** Installation.
Update your system repository index and install the recommended Desktop installation (includes ROS 2 core, RViz, and demo tools):

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ros-jazzy-desktop ros-dev-tools -y

```

*(Note: If you only need a headless server or embedded installation, replace `ros-jazzy-desktop` with `ros-jazzy-ros-base`).*

*Verification:* Run `dpkg -l | grep ros-jazzy-desktop` to confirm the installation completed without errors.


4. **Environment Setup:** Shell Configuration.
Source the setup script to make ROS 2 commands available in your terminal:

```bash
source /opt/ros/jazzy/setup.bash

```

To automatically source ROS 2 every time you open a new terminal session, add it to your `.bashrc` file:

```bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc

```

*Verification:* Run `ros2 --help`. If the ROS 2 CLI options display, your path environment is correctly configured.


5. **Test the Installation:** Smoke Test.
Test that the publisher/subscriber nodes can communicate properly.

1. Open **Terminal 1** and run a C++ talker node:

```bash
ros2 run demo_nodes_cpp talker

```

2. Open **Terminal 2** and run a Python listener node:

```bash
ros2 run demo_nodes_py listener

```

*Verification:* Terminal 2 should begin printing messages (`[INFO] [listener]: I heard: [Hello World: ...]`) received from Terminal 1.