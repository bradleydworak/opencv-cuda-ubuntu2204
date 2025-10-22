# compile-cuda-opencv-ffmpeg-ubuntu
## Compile OpenCV and FFMPEG with hardware accelarated video encoding using the NVIDIA CUDA Toolkit v13.0 and numpy2 v2.3.3 under Ubuntu 24.04

**Note:** These instructions can be used for a minimal Ubuntu installation option using Python 3.12.

### Step 1a: Remove any NVIDIA drivers

* `sudo dpkg -P $(dpkg -l | grep nvidia | awk '{print $2}')`

* `sudo apt autoremove`

### Step 1b: Remove any outdated repositories in the sources.list

* `ls /etc/apt/sources.list.d`

* `sudo apt autoremove`

### Step 2: Install the CUDA Toolkit and NVIDIA Drivers according to [NVIDIA CUDA Toolkit Installation](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=24.04&target_type=deb_network)

Install CUDA Toolkit 13.0:

* `wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb`
* `sudo dpkg -i cuda-keyring_1.1-1_all.deb`
* `sudo apt update`
* `sudo apt install cuda-13-0`

Install the latest NVIDIA Signed Drivers from Canonical (for optimization and security)

* `sudo apt install nvidia-open`

* `sudo apt install linux-nvidia` or for example for a specific kernel version `linux-nvidia-6.14`

Optional: Enable persistence mode for the GPU to reduce power draw at idle:

* `sudo nvidia-smi --persistence-mode=ENABLED`

### Step 2b (Optional): Install NVIDIA GPU DirectStorage (GDS) Drivers according to [NVIDIA GPUDirect Storage Installation and Troubleshooting Guide](https://docs.nvidia.com/gpudirect-storage/troubleshooting-guide/index.html)

**Note:** Refer to website for pre-installation instructions, including IOMMU adjustments.

* `sudo apt install nvidia-gds`

### Step 3: Install latest CUDNN drivers 

* `sudo apt-get install cudnn9-cuda-13` (at the time of writing cuDNN v9 is the latest)

### Step 4: Create a Python 3.12 Virtual Environment using pipx

* `sudo apt install python3-pip`

* `sudo apt install pipx`

* `pipx ensurepath`

* Close and reopen terminal

* `pipx install virtualenv`

* `virtualenv --python=python3.12 {name of environment}`

* `source {environment path and name}/bin/activate`

* `pip install numpy`

### Step 5: Find Compute Capability for the GPU from [Your GPU Compute Capability](https://developer.nvidia.com/cuda-gpus)

### Step 6: Install prerequisites to compile OpenCV from source code, 

* `sudo apt install cmake`

* `sudo apt install python3-numpy` (may not be needed)

* `sudo apt install libavcodec-dev libavformat-dev libswscale-dev`

* `sudo apt install libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev`

* `sudo apt install libgtk-3-dev`

* `sudo apt install libpng-dev libjpeg-dev libopenexr-dev libtiff-dev libwebp-dev`

* `sudo apt install git`

### Step 7: Install other prerequisites to compile FFMPEG from source code, 

* `sudo apt-get install build-essential yasm libtool libc6 libc6-dev unzip wget libnuma1 libnuma-dev`

### Step 8: Compile and install the latest FFMPEG, adopted from [Using FFmpeg with NVIDIA GPU Hardware Acceleration](https://docs.nvidia.com/video-technologies/video-codec-sdk/11.1/ffmpeg-with-nvidia-gpu/index.html)

* Clone and install ffnvcodec headers

`git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git`

`cd nv-codec-headers && sudo make install && cd –`

* Clone FFMpeg repository
  
`git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg/ ** cd ffmpeg`

* Configure, compile, and install source code using flags/options below. Be sure the paths point to your cuda folder...

```
./configure --prefix=/usr/local --extra-cflags='-I/usr/local/include -I/usr/local/cuda/include' --extra-ldflags='-L/usr/local/lib -L/usr/local/cuda/lib64' --extra-libs='-lpthread -lm' --enable-gpl --enable-nonfree --enable-cuda-nvcc --enable-cuda-llvm --enable-nvenc --enable-cuvid --disable-static --enable-shared`
```

`make -j$(nproc)`

`sudo make install`

### Step 9: Compile and install the latest OpenCV adopted from [Installing OpenCV 4 with CUDA in Ubuntu 22.04](https://towardsdev.com/installing-opencv-4-with-cuda-in-ubuntu-20-04-fde6d6a0a367)

* `git clone https://github.com/opencv/opencv.git`

* `git clone https://github.com/opencv/opencv_contrib.git`

* `cd opencv; mkdir build`

* Ensure your Python environment is activated

* cd opencv/build

* Compile with the flags/options below. Make sure that the FFMPEG and CUDNN library and Python packages path point to the correct destinations...

```
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D CMAKE_PREFIX_PATH=/usr/local -D FFMPEG_INCLUDE_DIR=/usr/local/include -D FFMPEG_LIBRARIES="/usr/local/lib/libavcodec.so;/usr/local/lib/libavformat.so;/usr/local/lib/libavutil.so;/usr/local/lib/libswscale.so" -D CMAKE_VERBOSE_MAKEFILE=ON -D WITH_CUDA=ON -D WITH_NVCUVID=OFF -D WITH_NVCUVENC=OFF -D WITH_CUDNN=ON -D WITH_CUBLAS=ON -D WITH_TBB=ON -D WITH_FFMPEG=ON -D BUILD_opencv_cudacodec=ON -D OPENCV_DNN_CUDA=ON -D OPENCV_ENABLE_NONFREE=ON -D CUDA_ARCH_BIN={compute capability number in the form of x.x} -D OPENCV_EXTRA_MODULES_PATH=$HOME/opencv_contrib/modules -D BUILD_EXAMPLES=OFF -D HAVE_opencv_python3=ON -D ENABLE_FAST_MATH=1 -D cuda_toolkit_root_dir=/usr/local/cuda -D CUDNN_INCLUDE_DIR=/usr/include/x86_64-linux-gnu -D CUDNN_LIBRARY=/usr/lib/x86_64-linux-gnu/libcudnn.so.9 -D CUDA_nvcuvid_LIBRARY=/usr/lib/x86_64-linux.gnu -D PYTHON3_PACKAGES_PATH=/usr/local/lib/python3.12/dist-packages ..
```

* Ensure that the Python3: numpy: is version 2.X.X in the cmake results and FFMPEG reports the correct versions of the libav*.so libraries

* `make -j$(nproc)`

Note: If cmake fails to compile due to missing cudnn header and library files, please refer to the following link for help: [How to install cudnn on Ubuntu](https://askubuntu.com/questions/767269/how-can-i-install-cudnn-on-ubuntu-16-04/767270#767270)

* `sudo make install`

* `sudo ldconfig` (may not be needed)

* `sudo ln -s /usr/local/lib/python3.12/dist-packages/cv2 {environment path and name}/lib/python3.12/site-packages/cv2`

### Step 10: Test installation

* Open Python interpreter: `python3`

* `import numpy, cv2`

* The following command should indicate that the GPU was located: `cv2.cuda.printCudaDeviceInfo(0)`
