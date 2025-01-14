# opencv-cuda-ubuntu2404
## Compile OpenCV with NVIDIA GPU CUDA support under Ubuntu 24.04 in a virtual environment

**Note:** These instructions can be used for a minimal Ubuntu installation option using Python 3.12.

### Step 1: Remove any NVIDIA drivers

* `sudo dpkg -P $(dpkg -l | grep nvidia | awk '{print $2}')`

* `sudo apt autoremove`

### Step 2: Install CUDA Toolkit and NVIDIA Drivers according to [NVIDIA CUDA Toolkit 12.6 Installation](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=24.04&target_type=deb_network)

**Note:** This installation has been tested using the NVIDIA Open v560 drivers.

Install the latest CUDA Toolkit using:

* `wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb`
* `sudo dpkg -i cuda-keyring_1.1-1_all.deb`
* `sudo apt-get update`
* `sudo apt-get -y install cuda-toolkit-12-6`

Install NVIDIA Open Source Drivers:

* `sudo apt-get install -y nvidia-open`

Enable persistence mode for the GPU to reduce power draw at idle:

* `sudo nvidia-smi --persistence-mode=ENABLED`

### Step 2b (Optional): Install NVIDIA GPU DirectStorage (GDS) Drivers according to [NVIDIA GPUDirect Storage Installation and Troubleshooting Guide](https://docs.nvidia.com/gpudirect-storage/troubleshooting-guide/index.html)

**Note:** Refer to website for pre-installation instructions, including IOMMU adjustments.

* `sudo apt install nvidia-gds`

### Step 3: Install latest CUDNN drivers according to Step 3 of [Installing CUDA and cuDNN in Ubuntu 22.04 for deep learning](https://medium.com/@juancrrn/installing-cuda-and-cudnn-in-ubuntu-20-04-for-deep-learning-dad8841714d6)

### Step 4: Find Compute Capability for the GPU from [Your GPU Compute Capability](https://developer.nvidia.com/cuda-gpus)

### Step 5: Prepare to compile OpenCV from source code, adopted from [Installing OpenCV 4 with CUDA in Ubuntu 22.04](https://towardsdev.com/installing-opencv-4-with-cuda-in-ubuntu-20-04-fde6d6a0a367)

* `sudo apt install cmake`

* `sudo apt install python3-numpy`

* `sudo apt install libavcodec-dev libavformat-dev libswscale-dev`

* `sudo apt install libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev`

* `sudo apt install libgtk-3-dev`

* `sudo apt install libpng-dev libjpeg-dev libopenexr-dev libtiff-dev libwebp-dev`

* `sudo apt install git`

* `git clone https://github.com/opencv/opencv.git`

* `git clone https://github.com/opencv/opencv_contrib.git`

* `cd opencv; mkdir build; cd build`

* `cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D WITH_CUDA=ON -D WITH_CUDNN=ON -D WITH_CUBLAS=ON -D WITH_TBB=ON -D OPENCV_DNN_CUDA=ON -D OPENCV_ENABLE_NONFREE=ON -D CUDA_ARCH_BIN={compute capability number in the form of x.x} -D OPENCV_EXTRA_MODULES_PATH=$HOME/opencv_contrib/modules -D BUILD_EXAMPLES=OFF -D HAVE_opencv_python3=ON -D ENABLE_FAST_MATH=1 -D cuda_toolkit_root_dir=/usr/local/cuda-{xx.x} -D CUDNN_INCLUDE_DIR=/usr/include/ -D CUDNN_LIBRARY=/usr/lib/x86_64-linux-gnu/libcudnn.so.8 ..`

* `make -j {number of CPU cores}`

* `sudo make install`

* `sudo ldconfig`

### Step 6: Create Python 3.12 Virtual Environment using pipx

* `sudo apt install python3-pip`

* `python3 -m pip install --user pipx`

* `python3 -m pipx ensurepath`

* Close and reopen terminal

* `pipx install virtualenv`

* `virtualenv --python=python3.12 {name of environment}`

* `sudo ln -s /usr/local/lib/python3.12/dist-packages/cv2 {environment path and name}/lib/python3.12/site-packages/cv2`

### Step 7: Test installation

* Open Python interpreter: `python3`

* `import numpy, cv2`

* The following command should indicate that the GPU was located: `cv2.cuda.printCudaDeviceInfo(0)`


