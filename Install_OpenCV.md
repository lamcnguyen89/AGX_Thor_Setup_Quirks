To install OpenCV with CUDA acceleration on the **NVIDIA Jetson AGX Thor**, you must **build OpenCV from source**. Pre-built apt binaries (`python3-opencv`) ship as CPU-only versions because CUDA architectures are target-specific. This guide is for Cuda 13.2 on the AGX Thor

The Jetson AGX Thor uses the **Blackwell GPU architecture**, which targets **CUDA Compute Capability `11.0**` (`sm_110`) under CUDA 13+ or `10.0`/`10.1` under CUDA 12.x.

---

## Step 1: Clean Up Previous Installs & Setup Environment

1. **Remove default CPU-only OpenCV installations** to prevent Python path conflicts:
```bash
sudo apt remove -y python3-opencv libopencv-dev libopencv-contrib-dev
sudo apt autoremove -y

```


2. **Install required build tools and dependencies**:
```bash
sudo apt update
sudo apt install -y build-essential cmake git pkg-config \
    libjpeg-dev libpng-dev libtiff-dev \
    libavcodec-dev libavformat-dev libswscale-dev \
    libgtk-3-dev libcanberra-gtk3-dev \
    python3-dev python3-numpy python3-pip \
    libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev \
    libv4l-dev v4l-utils \
    nvidia-cudnn \
    libopenjp2-7-dev

```



---

## Step 2: Download OpenCV and OpenCV Contrib Repositories

Download matching release versions of `opencv` and `opencv_contrib` (e.g., OpenCV 4.10.0 or 4.11.0):

```bash
cd ~
git clone --depth 1 --branch 4.13.0 https://github.com/opencv/opencv.git
git clone --depth 1 --branch 4.13.0 https://github.com/opencv/opencv_contrib.git

```
Apply patch to OpenCV contrib because there is an error with Cuda 13.2

```bash
cd ~/opencv_contrib

curl -L https://github.com/opencv/opencv_contrib/pull/4097.patch \
  -o /tmp/opencv-cuda-13.2.patch

git apply --check /tmp/opencv-cuda-13.2.patch
git apply /tmp/opencv-cuda-13.2.patch
```


---

## Step 3: Configure CMake with CUDA Flags for AGX Thor

1. Create a build directory:
```bash
cd ~/opencv
mkdir build && cd build
conda deactivate

```


2. Run `cmake` specifying the Blackwell architecture for AGX Thor (`CUDA_ARCH_BIN=11.0` for CUDA 13 / JetPack 7, or `10.0` if using CUDA 12.x):
```bash
conda deactivate # First Deactivate any anaconda environments. 
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
      -D WITH_CUDA=ON \
      -D CUDA_ARCH_BIN="11.0" \
      -D WITH_CUDNN=ON \
      -D OPENCV_DNN_CUDA=ON \
      -D ENABLE_FAST_MATH=ON \
      -D CUDA_FAST_MATH=ON \
      -D WITH_GSTREAMER=ON \
      -D WITH_LIBV4L=ON \
      -D BUILD_opencv_python3=ON \
      -D OPENCV_GENERATE_PKGCONFIG=ON \
      -D BUILD_EXAMPLES=OFF \
      -D BUILD_TESTS=OFF \
      -D BUILD_PERF_TESTS=OFF .. \
      -D Python3_EXECUTABLE=/usr/bin/python3 \
      -D PYTHON3_EXECUTABLE=/usr/bin/python3 \
      -D PYTHON3_INCLUDE_DIR=/usr/include/python3.12 \
      -D PYTHON3_LIBRARY=/usr/lib/aarch64-linux-gnu/libpython3.12.so \
      -D PYTHON3_NUMPY_INCLUDE_DIRS=/usr/lib/python3/dist-packages/numpy/core/include \
      -D PYTHON3_PACKAGES_PATH=/usr/local/lib/python3.12/dist-packages

```


3. **Verify the CMake output summary before compiling**. Check that the following lines appear in the output:
* `NVIDIA CUDA: YES`
* `NVIDIA GPU arch: 110` (or `100`)
* `cuDNN: YES`


---

## Step 4: Compile and Install

Run `make` using all available CPU threads on your AGX Thor, then install the binaries:

```bash
make -j$(nproc)
sudo make install
sudo ldconfig

```

---

## Step 5: Verify CUDA Acceleration in Python

Run this quick test script to confirm that OpenCV detects the Thor Blackwell GPU:

```python
import cv2

# Check CUDA device count
cuda_count = cv2.cuda.getCudaEnabledDeviceCount()
print(f"CUDA Enabled Devices: {cuda_count}")

if cuda_count > 0:
    # Print CUDA build information
    build_info = cv2.getBuildInformation()
    cuda_idx = build_info.find("NVIDIA CUDA")
    print("\nCUDA Configuration:")
    print(build_info[cuda_idx:cuda_idx+250])

    # Test CUDA matrix operation on GPU
    import numpy as np
    src_cpu = np.ones((1080, 1920, 3), dtype=np.uint8) * 255
    
    gpu_mat = cv2.cuda_GpuMat()
    gpu_mat.upload(src_cpu)
    gpu_gray = cv2.cuda.cvtColor(gpu_mat, cv2.COLOR_BGR2GRAY)
    result = gpu_gray.download()
    
    print(f"\nGPU upload/download matrix test successful! Output shape: {result.shape}")
else:
    print("CUDA device not found by OpenCV. Check your CMake flags and CUDA installation.")

```