# Installing ROS 2 Humble on Ubuntu 22

To install ROS 2 Humble Hawksbill on Ubuntu 22.04 LTS (Jammy Jellyfish), follow these step-by-step instructions via the official `apt` repository.

1. **Ensure UTF-8 Locale:** Prerequisite.
Make sure your system environment supports UTF-8. Run the following commands to set it up:

```bash
sudo apt update && sudo apt install locales -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

```

Verify that the output of `locale` shows `en_US.UTF-8`.


2. **Enable Ubuntu Universe Repository:** Prerequisite.
Ensure the Ubuntu Universe repository is enabled on your system:

```bash
sudo apt install software-properties-common -y
sudo add-apt-repository universe -y

```


3. **Add the ROS 2 Apt Repository:** Setup GPG key & sources list.
First, add the official ROS 2 GPG key:

```bash
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

```

Next, add the repository to your sources list:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

```


4. **Install ROS 2 Packages:** Desktop Installation.
Update your `apt` package index and upgrade existing packages before installing ROS 2:

```bash
sudo apt update && sudo apt upgrade -y

```

Install ROS 2 Desktop (includes ROS, RViz, demos, and tutorials):

```bash
sudo apt install ros-humble-desktop python3-argcomplete -y

```

*(Optional)* If you are building ROS packages, install development tools and `colcon`:

```bash
sudo apt install ros-dev-tools python3-colcon-common-extensions -y

```


5. **Environment Setup:** Source ROS 2 script.
Source the setup script in your current terminal session:

```bash
source /opt/ros/humble/setup.bash

```

To automatically source ROS 2 whenever you open a new terminal window:

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc

```


6. **Verify Installation:** Run demo publisher & subscriber.
Open **Terminal 1** and run a C++ talker node:

```bash
ros2 run demo_nodes_cpp talker

```

Open **Terminal 2** and run a Python listener node:

```bash
ros2 run demo_nodes_py listener

```

If you see messages being sent in Terminal 1 and received in Terminal 2, ROS 2 Humble is installed successfully.