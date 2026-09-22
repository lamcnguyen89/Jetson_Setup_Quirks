# Build OpenCV with CUDA on the Jetson Orin Nano

This guide builds OpenCV 4.13.0 with CUDA and cuDNN support for the Jetson used
by this project. The detected hardware is:

```text
Board:              NVIDIA Jetson Orin NX Developer Kit Super
GPU:                Orin
Compute capability: 8.7 (sm_87)
CUDA toolkit:       12.9
Python:             3.10
```

The CUDA architecture setting must match the physical GPU. An earlier build
used `CUDA_ARCH_BIN=10.0`, which produced an OpenCV installation that could see
the Orin GPU but could not run DNN inference on it. It failed with:

```text
OpenCV was not built to work with the selected device.
Please check CUDA_ARCH_PTX or CUDA_ARCH_BIN in your build configuration.
```

For an Orin Nano, Orin NX, or AGX Orin, use `CUDA_ARCH_BIN=8.7`. Do not use the
Blackwell architecture values intended for Jetson Thor.

## 1. Confirm the hardware and toolchain

Check the Jetson model and GPU compute capability:

```bash
tr -d '\0' </proc/device-tree/model
echo
nvidia-smi --query-gpu=name,compute_cap --format=csv,noheader
```

The GPU query on this Jetson should include:

```text
Orin (nvgpu), 8.7
```

Check the selected CUDA toolkit:

```bash
readlink -f /usr/local/cuda
/usr/local/cuda/bin/nvcc --version
```

The commands in this guide assume that `/usr/local/cuda` resolves to CUDA 12.9.
If it points to a different toolkit, update the CUDA paths in the CMake command.

## 2. Remove conflicting OpenCV packages

Ubuntu's `python3-opencv` package normally lacks the Jetson-specific CUDA build
needed here. Remove it and its development packages if they are installed:

```bash
sudo apt remove -y python3-opencv libopencv-dev libopencv-contrib-dev
sudo rm -rf opencv opencv_contrib
```

Remove PyPI OpenCV wheels from the system Python and from any virtual or Conda
environment used by the project:

```bash
python3 -m pip uninstall -y \
    opencv-python \
    opencv-contrib-python \
    opencv-python-headless \
    opencv-contrib-python-headless
```

PyPI wheels can shadow the custom `/usr/local` installation and generally do
not contain the CUDA support built for this Jetson.

## 3. Install build dependencies

```bash
sudo apt update
sudo apt install -y \
    build-essential \
    cmake \
    git \
    pkg-config \
    libjpeg-dev \
    libpng-dev \
    libtiff-dev \
    libavcodec-dev \
    libavformat-dev \
    libswscale-dev \
    libgtk-3-dev \
    libcanberra-gtk3-dev \
    python3-dev \
    python3-numpy \
    python3-pip \
    libgstreamer1.0-dev \
    libgstreamer-plugins-base1.0-dev \
    libv4l-dev \
    v4l-utils \
    libopenjp2-7-dev
```

Verify that the CUDA 12 cuDNN runtime and development packages are installed:

```bash
dpkg -l | grep libcudnn9
```

The CMake summary must ultimately report `cuDNN: YES`. Avoid installing a
different cuDNN meta-package over the JetPack-compatible packages already on
the system, since mixed CUDA/cuDNN package versions can conflict.

## 4. Download matching OpenCV sources

The `opencv` and `opencv_contrib` versions must match:

```bash
cd "$HOME"
git clone --depth 1 --branch 4.13.0 \
    https://github.com/opencv/opencv.git
git clone --depth 1 --branch 4.13.0 \
    https://github.com/opencv/opencv_contrib.git
```

If those repositories already exist at version 4.13.0, reuse them. The CUDA
13.2 contrib patch previously mentioned in this guide is not required for this
Jetson's CUDA 12.9 build.

## 5. Configure a fresh Orin build

Use a new build directory. Reusing a directory previously configured for
architecture `10.0` can leave incompatible cached settings or object files.

Deactivate Conda first if it is active:

```bash
conda deactivate
```

Configure OpenCV from the source directory:

```bash
cd "$HOME/opencv"

cmake -S . -B build-orin \
    -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D OPENCV_EXTRA_MODULES_PATH="$HOME/opencv_contrib/modules" \
    -D CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-12.6 \
    -D CUDA_NVCC_EXECUTABLE=/usr/local/cuda-12.6/bin/nvcc \
    -D WITH_CUDA=ON \
    -D CUDA_ARCH_BIN=8.7 \
    -D CUDA_ARCH_PTX="" \
    -D WITH_CUBLAS=ON \
    -D WITH_CUDNN=ON \
    -D OPENCV_DNN_CUDA=ON \
    -D ENABLE_FAST_MATH=ON \
    -D CUDA_FAST_MATH=ON \
    -D WITH_GSTREAMER=ON \
    -D WITH_V4L=ON \
    -D BUILD_opencv_python3=ON \
    -D OPENCV_GENERATE_PKGCONFIG=ON \
    -D BUILD_EXAMPLES=OFF \
    -D BUILD_TESTS=OFF \
    -D BUILD_PERF_TESTS=OFF \
    -D Python3_EXECUTABLE=/usr/bin/python3 \
    -D PYTHON3_EXECUTABLE=/usr/bin/python3 \
    -D PYTHON3_INCLUDE_DIR=/usr/include/python3.10 \
    -D PYTHON3_LIBRARY=/usr/lib/aarch64-linux-gnu/libpython3.10.so \
    -D PYTHON3_NUMPY_INCLUDE_DIRS=/usr/lib/python3/dist-packages/numpy/core/include \
    -D PYTHON3_PACKAGES_PATH=/usr/local/lib/python3.10/dist-packages
```

Before compiling, read the CMake summary and confirm at least:

```text
NVIDIA CUDA:                   YES
NVIDIA GPU arch:               87
cuDNN:                         YES
Python 3 interpreter:          /usr/bin/python3
```

Do not continue if the summary still reports GPU architecture `100`.

## 6. Compile and install

The detected Jetson has approximately 8 GB of RAM. Two parallel compilation
jobs are conservative and help avoid the compiler being killed due to memory
pressure:

```bash
cd "$HOME/opencv"
cmake --build build-orin --parallel 4
sudo cmake --install build-orin
sudo ldconfig
```

## 7. Confirm the installed build

First verify which `cv2` module Python imports and inspect its CUDA settings:

```bash
python3 - <<'PY'
import cv2

print("OpenCV:", cv2.__version__)
print("Module:", cv2.__file__)
print("CUDA devices:", cv2.cuda.getCudaEnabledDeviceCount())

for line in cv2.getBuildInformation().splitlines():
    if any(item in line for item in (
        "NVIDIA CUDA",
        "NVIDIA GPU arch",
        "NVIDIA PTX archs",
        "cuDNN",
    )):
        print(line.strip())

cv2.cuda.printShortCudaDeviceInfo(0)
PY
```

The output must show one CUDA device, GPU architecture `87`, and an Orin device
with `sm_87`. A positive device count by itself is insufficient: OpenCV can
detect a GPU even when its compiled kernels target the wrong architecture.

