# opencv-cuda-ubuntu2404
## Compile OpenCV with the latest NVIDIA GPU CUDA driver support under Ubuntu 24.04 in a virtual environment

**Note:** These instructions can be used for a minimal Ubuntu installation option using Python 3.12.

### Step 1a: Remove any NVIDIA drivers

* `sudo dpkg -P $(dpkg -l | grep nvidia | awk '{print $2}')`

* `sudo apt autoremove`

### Step 1b: Remove any outdated repositories in the sources.list

* `ls /etc/apt/sources.list.d`

* `sudo apt autoremove`

### Step 2: Install the CUDA Toolkit and NVIDIA Drivers according to [NVIDIA CUDA Toolkit Installation](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=24.04&target_type=deb_network)

Install CUDA Toolkit 12.8 (OpenCV does not compile currently with 12.9) using:

* `wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb`
* `sudo dpkg -i cuda-keyring_1.1-1_all.deb`
* `sudo apt-get update`
* `sudo apt-get install cuda-toolkit-12-8`

Install NVIDIA Open Source Drivers:

* `sudo apt-get install nvidia-open`

Optional: Enable persistence mode for the GPU to reduce power draw at idle:

* `sudo nvidia-smi --persistence-mode=ENABLED`

### Step 2b (Optional): Install NVIDIA GPU DirectStorage (GDS) Drivers according to [NVIDIA GPUDirect Storage Installation and Troubleshooting Guide](https://docs.nvidia.com/gpudirect-storage/troubleshooting-guide/index.html)

**Note:** Refer to website for pre-installation instructions, including IOMMU adjustments.

* `sudo apt install nvidia-gds`

### Step 3: Install latest CUDNN drivers 

* `sudo apt-get install cudnn9-cuda-12` (at the time of writing cuDNN 12 is the latest)

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

### Step 6: Install prerequisites to compile OpenCV from source code, adopted from [Installing OpenCV 4 with CUDA in Ubuntu 22.04](https://towardsdev.com/installing-opencv-4-with-cuda-in-ubuntu-20-04-fde6d6a0a367)

* `sudo apt install cmake`

* `sudo apt install python3-numpy` (may not be needed)

* `sudo apt install libavcodec-dev libavformat-dev libswscale-dev`

* `sudo apt install libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev`

* `sudo apt install libgtk-3-dev`

* `sudo apt install libpng-dev libjpeg-dev libopenexr-dev libtiff-dev libwebp-dev`

* `sudo apt install git`

### Step 7: Compile and install OpenCV

* `git clone https://github.com/opencv/opencv.git`

* `git clone https://github.com/opencv/opencv_contrib.git`

* `cd opencv; mkdir build`

* Ensure your Python environment is activated

* cd opencv/build

* Compile with options below:
```
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D WITH_CUDA=ON -D WITH_CUDNN=ON -D WITH_CUBLAS=ON -D WITH_TBB=ON -D OPENCV_DNN_CUDA=ON -D OPENCV_ENABLE_NONFREE=ON -D CUDA_ARCH_BIN={compute capability number in the form of x.x} -D OPENCV_EXTRA_MODULES_PATH=$HOME/opencv_contrib/modules -D BUILD_EXAMPLES=OFF -D HAVE_opencv_python3=ON -D ENABLE_FAST_MATH=1 -D cuda_toolkit_root_dir=/usr/local/cuda -D CUDNN_INCLUDE_DIR=/usr/include/ -D CUDNN_LIBRARY=/usr/lib/x86_64-linux-gnu/libcudnn.so.9 -D PYTHON3_PACKAGES_PATH=/usr/local/lib/python3.12/dist-packages ..
```

* `make -j$(nproc)`

* `sudo make install`

* `sudo ldconfig` (may not be needed)

* `sudo ln -s /usr/local/lib/python3.12/dist-packages/cv2 {environment path and name}/lib/python3.12/site-packages/cv2`

### Step 7: Test installation

* Open Python interpreter: `python3`

* `import numpy, cv2`

* The following command should indicate that the GPU was located: `cv2.cuda.printCudaDeviceInfo(0)`


