# ORB-SLAM3 Installation Guide for NVIDIA Jetson AGX Xavier

This guide is designed to help you install ORB-SLAM3 on a NVIDIA Jetson AGX Xavier board running Jetpack (which is based on Ubuntu 20.04). The Xavier AGX platform has an ARM architecture, so make sure to use compatible libraries and packages.

## Prerequisites

Before you begin, ensure your Xavier board is running the latest version of Jetpack. You can download and install it from the [NVIDIA official website](https://developer.nvidia.com/embedded/jetpack).

## Step 1: Install Dependencies

First, update your package list and install the necessary dependencies:

```sh
sudo apt update && sudo apt upgrade -y
sudo apt-get install build-essential cmake git libgtk-3-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install python3-dev python3-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev
sudo apt-get install libeigen3-dev libglew-dev libboost-all-dev libssl-dev
```

## Step 2: Install OpenCV

ORB-SLAM3 requires OpenCV. Let's build OpenCV from source:

```sh
# Create a directory for the OpenCV build
cd ~
mkdir -p Dev && cd Dev
mkdir opencv_build && cd opencv_build

# Clone the OpenCV's and OpenCV contrib repositories
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git

# Checkout a stable version, for example, 4.5.0
cd opencv && git checkout 4.5.0
cd ../opencv_contrib && git checkout 4.5.0

# Prepare the build
cd ../opencv
mkdir build && cd build

# Configure the OpenCV build with CMake
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules \
      -D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \
      -D WITH_CUDA=ON \
      -D CUDA_ARCH_BIN=7.2 \
      -D CUDA_ARCH_PTX="" \
      -D WITH_CUDNN=ON \
      -D WITH_CUBLAS=1 \
      -D ENABLE_FAST_MATH=1 \
      -D CUDA_FAST_MATH=1 \
      -D ENABLE_NEON=ON \
      -D WITH_QT=OFF \
      -D WITH_OPENCL=OFF \
      -D BUILD_TESTS=OFF \
      -D BUILD_PERF_TESTS=OFF \
      -D BUILD_EXAMPLES=OFF ..

# Build OpenCV
make -j$(nproc)
sudo make install
```

## Step 3: Install Pangolin

```sh
cd ~/Dev

# Clone the Pangolin repository
git clone https://github.com/stevenlovegrove/Pangolin.git

# Enter the Pangolin directory
cd Pangolin

# Checkout the specific commit for Pangolin if required
git checkout 86eb4975fc4fc8b5d92148c2e370045ae9bf9f5d

# Create a build directory for Pangolin and enter it
mkdir build && cd build

# Configure the Pangolin build with CMake for release
cmake .. -DCMAKE_BUILD_TYPE=Release

# Compile Pangolin using the available cores on Xavier AGX (use 'nproc' to determine and use all available cores)
make -j$(nproc)

# Install Pangolin on your system
sudo make install
```

## Step 4: Clone and build ORB-SLAM3

```sh
cd ~/Dev
git clone https://github.com/UZ-SLAMLab/ORB_SLAM3.git ORB_SLAM3
cd ORB_SLAM3
git submodule update --init --recursive
chmod +x build.sh
./build.sh
```

## Step 5: Running ORB-SLAM3 Live with RealSense D435 Camera

To run ORB-SLAM3 with a RealSense D435 camera, follow these steps:

1. Plug in the RealSense D435 camera to the AGX board.
2. Navigate to the ORB-SLAM3 directory:
    ```sh
    cd ~/ORB_SLAM3
    ```
3. Execute the following command:
    ```sh
    ./Examples/Stereo/stereo_realsense_D435i Vocabulary/ORBvoc.txt ./Examples/Stereo/RealSense_D435i.yaml
    ```

### Handling Segmentation Fault Error

If you encounter a segmentation fault error on Step 5, such as:

```
SLAM settings:
    -Camera 1 parameters (Pinhole): [ 382.613 382.613 320.183 236.455 ]
Segmentation fault (core dumped)
```

You can resolve it by commenting out a specific line in the `ORB_SLAM3/src/System.cc` file. Open the file and comment out the following line:

```cpp
// cout << (*settings_) << endl;
```

Then recompile ORB-SLAM3:

```sh
cd ~/Dev/ORB_SLAM3
./build.sh
```

and go back to Step 5.
