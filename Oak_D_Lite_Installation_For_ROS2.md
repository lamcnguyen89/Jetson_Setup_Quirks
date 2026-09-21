# How to Install Oak-D Lite Camera on Ubuntu and Connect it to ROS

To connect a Luxonis OAK-D Lite camera to ROS 2, you need to configure **USB permissions (udev rules)**, install the official **`depthai-ros` driver package**, and launch the node.

---

0. **Install DepthAI Package:** I don't think this part is necessary for using the Oak-D lite camera with ROS. This step is just there to make sure your Ubuntu OS can actually access and use the Oak-D Lite Depth Camera

```bash

git clone https://github.com/luxonis/depthai-core.git && cd depthai-core
python3 -m venv venv
source venv/bin/activate
# Installs library and requirements
python3 examples/python/install_requirements.py

```

Next run the example:

```bash
cd examples/python
# Run YoloV6 detection example
python3 DetectionNetwork/detection_network.py
# Display all camera streams
python3 Camera/camera_all.py
```

1. **Configure USB Permissions (udev Rules):** Prerequisite.
By default, Linux limits raw USB access. Add the Luxonis udev rules so ROS 2 can communicate with the camera without root privileges:

```bash
echo 'SUBSYSTEM=="usb", ATTRS{idVendor}=="03e7", MODE="0666"' | sudo tee /etc/udev/rules.d/80-movidius.rules
sudo udevadm control --reload-rules && sudo udevadm trigger

```

> **Check success:** Unplug the camera, plug it back into a **USB 3.0 (blue) port**, and run `lsusb`. You should see `Intel Movidius` or `MyriadX` listed.

2. Before going on to the next steps, make sure you have ROS2 installed.

3. **Install the depthai-ros Driver Package:** ROS 2 Binaries.
Luxonis provides pre-built binaries for supported ROS 2 distributions (replace `<ros-distro>` with `humble`, `iron`, `jazzy`, etc.):

```bash
sudo apt update
sudo apt install ros-<ros-distro>-depthai-ros

```

*(Optional)* If binaries are unavailable for your setup, clone and build from source:

```bash
mkdir -p ~/ros2_ws/src && cd ~/ros2_ws/src
git clone https://github.com/luxonis/depthai-ros.git
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash

```

> **Check success:** Run `ros2 pkg list | grep depthai` to verify that `depthai_ros_driver` is registered.


4. **Launch the Camera Node and Visualize:** Execution.
Start the ROS 2 camera driver. Luxonis includes launch arguments to bring up RViz automatically:

```bash
ros2 launch depthai_ros_driver driver.launch.py use_rviz:=true

```

If you only want basic RGB-D point cloud topics without neural network overlays, use:

```bash
ros2 launch depthai_ros_driver rgbd_pcl.launch.py

```

> **Check success:** Open a new terminal and run `ros2 topic list`. You should see camera topics active, such as `/oak/rgb/image_raw` and `/oak/stereo/image_raw`.