To build and install `realsense-viewer` on an **NVIDIA Jetson AGX Thor** running ARM64 architecture with **CUDA support enabled**, you need to compile `librealsense` from source.

Installing via standard `apt` repositories is typically limited to x86 architectures or CPU-only Linux packages, so building from source ensures full hardware acceleration on Thor's GPU.

---

## Prerequisites & Environment Setup

1. **Verify CUDA Installation:**
Make sure your CUDA path and compiler (`nvcc`) are accessible in your environment:
```bash
echo "export PATH=/usr/local/cuda/bin:$PATH" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH" >> ~/.bashrc
echo "export CUDACXX=/usr/local/cuda/bin/nvcc" >> ~/.bashrc
source ~/.bashrc

```


2. **Install Required Build Dependencies & Libraries:**
Install the standard build tools, USB headers, and OpenGL development packages:
```bash
sudo apt-get update
sudo apt-get install -y \
    build-essential \
    cmake \
    git \
    libssl-dev \
    libusb-1.0-0-dev \
    libudev-dev \
    pkg-config \
    libgtk-3-dev \
    libglfw3-dev \
    libgl1-mesa-dev \
    libglu1-mesa-dev \
    v4l-utils

```



---

## Build Procedure

1. **Configure Udev Rules:** Required before plugging in the camera.
Set up the RealSense `udev` rules so the system correctly permissions the camera USB endpoints:

```bash
git clone https://github.com/IntelRealSense/librealsense.git
cd librealsense
sudo ./scripts/setup_udev_rules.sh

```


2. **Configure CMake with CUDA & ARM Support:** Prepares build configuration for AGX Thor architecture.
Create a build directory and configure CMake. Enabling `-DBUILD_WITH_CUDA=true` accelerates point cloud generation and color space conversions on Thor's GPU.

```bash
mkdir build && cd build
cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_EXAMPLES=true \
  -DBUILD_GRAPHICAL_EXAMPLES=true \
  -DBUILD_WITH_CUDA=true \
  -DCMAKE_CUDA_ARCHITECTURES=native \
  -DFORCE_RSUSB_BACKEND=false

```


3. **Compile and Install:** Builds librealsense binaries and realsense-viewer.
Compile using multiple cores (adjust `-j` depending on your active memory allocations):

```bash
make -j$(($(nproc)-1))
sudo make install
sudo ldconfig

```


4. **Launch RealSense Viewer:** Testing hardware stream & interface.
Plug in your Intel RealSense camera (use a direct USB 3.2 port) and launch the viewer:

```bash
realsense-viewer

```


---

## 💡 Key AGX Thor Considerations

* **USB Bandwidth Allocation:** The RealSense depth stream consumes significant USB 3.2 bus bandwidth. On Jetson Thor hardware, plugging the camera directly into a high-speed Type-A port may tie up adjacent ports on the same controller. Connect peripherals (keyboard/mouse) via a separate USB-C hub if you encounter dropped USB packets.
* **Docker Option (Isaac ROS):** If you are running NVIDIA Isaac ROS workflows on Thor, `realsense-viewer` and the associated drivers come pre-packaged inside the `isaac-ros` Docker layer (`additional_image_keys: - realsense`).