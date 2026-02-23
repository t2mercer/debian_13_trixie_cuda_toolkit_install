# Debian 13 (Trixie) Cuda 13.1 Toolkit and Driver Install Recipe

My step by step recipe to create a clean install of the CUDA Toolkit on an updated Debian 13 (Trixie) linux OS - NO 32 bit compatibility included. The official CUDA Installation Guide for Linux and NVIDIA Driver Installation Guide can be found at the following 2 URLs. They covers all supported OS versions and modes of installation. This guide provides one that works for my Debian 13 (Trixie) box - if you want 32 bit compatibility or other OS/versions please the official guides.

> **CUDA Installation Guide for Linux**
>
> <https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#>
>
> **NVIDIA Driver Installation Guide**
>
> <https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/index.html#>

Ensure you do the following as a NON-ROOT user that has sudo rights. Do **_NOT_** do the following as ROOT.

Always important to ensure your system has been updated.

    ~$ sudo apt update && sudo apt upgrade

## Prep For Clean Install

Now we want to remove the NVIDIA Drivers first (before removing previously installed CUDA Toolkit). If it is done the other way around it does not appear to create a full clean up. If you have installed the 32 bit compatibility libraries are installed along with the main architecture follow the instructions here:

> <https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/removing-the-driver.html>

otherwise the following will do the trick to remove the driver packages.

    ~$ sudo apt remove --autoremove --purge -V \
        cuda-compat\* \
        cuda-drivers\* \
        cuda-mps\* \
        firmware-nvidia-gsp\* \
        libcuda1\* \
        libcudadebugger1\* \
        libegl-nvidia0\* \
        libgles-nvidia1\* \
        libgles-nvidia2\* \
        libglx-nvidia0\* \
        libnvcuvid1\* \
        libnvidia-allocator1\* \
        libnvidia-api1\* \
        libnvidia-cfg1\* \
        libnvidia-eglcore\* \
        libnvidia-encode1\* \
        libnvidia-fbc1\* \
        libnvidia-glcore\* \
        libnvidia-glvkspirv\* \
        libnvidia-gpucomp\* \
        libnvidia-ml1\* \
        libnvidia-ngx1\* \
        libnvidia-nscq\* \
        libnvidia-nvvm\* \
        libnvidia-opticalflow1\* \
        libnvidia-pkcs11-openssl3\* \
        libnvidia-present\* \
        libnvidia-ptxjitcompiler1\* \
        libnvidia-rtcore\* \
        libnvidia-sandboxutils\* \
        libnvidia-tileiras\* \
        libnvidia-vksc-core\* \
        libnvoptix1\* \
        libnvsdm\* \
        libxnvctrl\* \
        nvidia-alternative\* \
        nvidia-detect\* \
        nvidia-driver\* \
        nvidia-egl-common\* \
        nvidia-egl-icd\* \
        nvidia-fabricmanager\* \
        nvidia-imex\* \
        nvidia-kernel\* \
        nvidia-legacy\* \
        nvidia-modprobe\* \
        nvidia-open\* \
        nvidia-opencl\* \
        nvidia-persistenced\* \
        nvidia-powerd\* \
        nvidia-settings\* \
        nvidia-smi\* \
        nvidia-support\* \
        nvidia-suspend-common\* \
        nvidia-vdpau-driver\* \
        nvidia-vulkan\* \
        nvidia-xconfig\* \
        xserver-xorg-video-nvidia\*

Now to remove the Cuda ToolKit packages.

    ~$ sudo apt remove --autoremove --purge "*cuda*" "*cublas*" "*cufft*" "*cufile*" "*curand*" "*cusolver*" "*cusparse*" "*gds-tools*" "*npp*" "*nvjpeg*" "nsight*" "*nvvm*"

The OS needs to be rebooted as per NVIDIA's guidance.

    ~$ sudo reboot

## Check To See What System Has

Now a check of the system to see if you have devices that can make use of the CUDA Toolkit / NVIDIA drivers.

The following should have you a list of compatible devices on your system

    ~$ lspci | grep -i nvidia

Ideally, you will see something like the following - shows you the GPU you have installed.

>     01:00.0 VGA compatible controller: NVIDIA Corporation AD106 [GeForce RTX 4060 Ti 16GB] (rev a1)
>     01:00.1 Audio device: NVIDIA Corporation AD106M High Definition Audio Controller (rev a1)

Check the os and kernal you are running if you are unsure:

    ~$ hostnamectl

This will show something like the following:

>       Static hostname: YOUR_MACHINE_NAME
>             Icon name: computer-desktop
>               Chassis: desktop 🖥️
>            Machine ID: 328xxxxfac5649xxxxxxxx52657a9622
>               Boot ID: 712xxxxb7c4e49xxxxxxxx931e6e1583
>      Operating System: Debian GNU/Linux 13 (trixie)
>                Kernel: Linux 6.12.73+deb13-amd64
>          Architecture: x86-64
>       Hardware Vendor: Dell Inc.
>        Hardware Model: XPS 8930
>      Firmware Version: 1.1.31
>         Firmware Date: Tue 2023-11-21
>          Firmware Age: 2y 3month 4d

As per NVIDIA, you need the current version of gcc with the current kernal headers.

    ~$ gcc --version

This will show something like the following:

>     gcc (Debian 14.2.0-19) 14.2.0
>     Copyright (C) 2024 Free Software Foundation, Inc.
>     This is free software; see the source for copying conditions.  There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

If not, install gcc.

    ~$ sudo apt install gcc

## Make Sure ENV Variables Include CUDA Paths

When Cuda 13.1 is install it will land here:

    /usr/local/cuda-13.1/

Make sure your PATH env includes this path, check by echoing it out.

    ~$ echo $PATH

should show something like this - yours may look different but should have the /usr/local/cuda-13.1/bin path included.

    /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/cuda-13.1/bin

if not, do the following:

    ~$ export PATH=${PATH}:/usr/local/cuda-13.0/bin

Make sure your LD_LIBRARY_PATH env includes this path, check by echoing it out.

    ~$ echo $LD_LIBRARY_PATH

should show something like this - yours may look different but should have the /usr/local/cuda-13.1/lib64 path included.

    :/usr/local/cuda-13.1/lib64

if not, do the following:

    ~$ export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/cuda-13.1/lib64

## ADD Repos To Package Manager

We need to get the repo references which we use wget to do, so check if you have that.

    ~$ wget --version

This will show something like the following:

>     GNU Wget 1.25.0 built on linux-gnu.

Ok, now we need a place to keep a few things we will download. This can be a path of your choice but we will use ~/cuda.

    ~$ cd ~/
    ~$ mkdir cuda
    ~$ cd cuda

Now we get the nvidia repo references

    wget https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/cuda-keyring_1.1-1_all.deb

You should now have a file cuda-keyring_1.1-1_all.deb in your ~/cuda$ folder. Add them to the package managers and then go back to home folder.

    ~/cuda$ sudo dpkg -i cuda-keyring_1.1-1_all.deb
    cd ~

Refresh you package environment. Doing the following should only refresh the packages as you already did this at the beginning of the recipe and have upgraded your system appropriately before starting the recipe.

    ~$ sudo apt update && sudo apt upgrade

You should a line something like this in the repo list:

    Hit:7 https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64  InRelease

## Install NVIDIA Driver First

Installing the drivers first before the CUDA Toolkit appears to work better and consistently provides a stable complete install.

You need to have the correct linux-headers.

    ~$ sudo apt install linux-headers-$(uname -r)

Pin to the current version so you do not have to do this so often and the package manager updates should work correctly - though that does not appear to always be the case. For more details about pinning, see the official install guide: [Pinning - Selecting a Branch or a Specific Driver version](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/debian.html#selecting-a-branch-or-a-specific-driver-version).

    ~$ sudo apt install nvidia-driver-pinning-590

Below, the Proprietary Kernel Modules are installed, if you wish to install the Open Kernel Modules, see the offical documentation: [Driver Installation](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/debian.html#driver-installation)

    ~$ sudo apt -V install cuda-drivers

The OS needs to be rebooted as per NVIDIA's guidance.

    ~$ sudo reboot

After the reboot we need to restart nvidia-persistenced

    ~$ sudo systemctl restart nvidia-persistenced

The following:

    ~$ cat /proc/driver/nvidia/version

should produce something like this:

    NVRM version: NVIDIA UNIX x86_64 Kernel Module  590.48.01  Mon Dec  8 11:22:45 UTC 2025
    GCC version:  gcc version 14.2.0 (Debian 14.2.0-19)

## Ready To Install CUDA Toolkit

For completness sake:

    ~$ sudo apt update && sudo apt upgrade

Now install the CUDA Toolkit

    ~$ sudo apt install cuda-toolkit

The OS needs to be rebooted as per NVIDIA's guidance.

    ~$ sudo reboot

## Testing Installation

### LM Studio

You may be wanting to play with LLMs on your local linux install with LM Studio.  This is probably the quickest way to test to see if everything is installed and working as expected.

Get the latest appImage.

> <https://lmstudio.ai/download/latest/linux/x64?format=AppImage>

We created the cuda folder at ~/cuda.  Download the latest to that folder.  At the time of writing it was version 0.4.4 build 1.  Change to allow your use to execute.

    ~/cuda$ chmod 744 LM-Studio-0.4.4-1-x64.AppImage

The ~/cuda$ folder should now have something like this:

    total 1031892
    drwxrwxr-x  2 your_user your_user         78 Feb 23 15:55 ./
    drwx------ 33 your_user your_user       4096 Feb 23 15:19 ../
    -rw-rw-r--  1 your_user your_user       4182 Jan 14 03:46 cuda-keyring_1.1-1_all.deb
    -rwxr--r--  1 your_user your_user 1056644513 Feb 23 15:54 LM-Studio-0.4.4-1-x64.AppImage*

You can now execute LM-Studio

    ~/cuda$ ./LM-Studio-0.4.4-1-x64.AppImage

Inside LM-Studio press Ctrl Shift h.  This should open the hardware dialogue box and you should see your GPU in the section "GPU detected with CUDA"

### NVIDIA Sample Code

To test the install, NVIDIA provides a full set of [samples](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#verify-the-installation) that are build ready and be run to test your setup.  

We created the cuda folder at ~/cuda.  Git clone the project to into this folder.

    ~$ cd ~/cuda
    ~$ git clone https://github.com/nvidia/cuda-samples

Following the instructions here to [Build CUDA Samples](https://github.com/nvidia/cuda-samples?tab=readme-ov-file#building-cuda-samples-1)

When you have completed the build, you can run the deviceQuery app.

    ~$ ./cuda/cuda-samples/build/Samples/1_Utilities/deviceQuery/deviceQuery

This should produce an output something like this:

    CUDA Device Query (Runtime API) version (CUDART static linking)

    Detected 1 CUDA Capable device(s)

    Device 0: "NVIDIA GeForce RTX 4060 Ti"
    CUDA Driver Version / Runtime Version          13.1 / 13.1
    CUDA Capability Major/Minor version number:    8.9
    Total amount of global memory:                 15948 MBytes (16722821120 bytes)
    (034) Multiprocessors, (128) CUDA Cores/MP:    4352 CUDA Cores
    GPU Max Clock rate:                            2550 MHz (2.55 GHz)
    Memory Clock rate:                             9001 Mhz
    Memory Bus Width:                              128-bit
    L2 Cache Size:                                 33554432 bytes
    Maximum Texture Dimension Size (x,y,z)         1D=(131072), 2D=(131072, 65536), 3D=(16384, 16384, 16384)
    Maximum Layered 1D Texture Size, (num) layers  1D=(32768), 2048 layers
    Maximum Layered 2D Texture Size, (num) layers  2D=(32768, 32768), 2048 layers
    Total amount of constant memory:               65536 bytes
    Total amount of shared memory per block:       49152 bytes
    Total shared memory per multiprocessor:        102400 bytes
    Total number of registers available per block: 65536
    Warp size:                                     32
    Maximum number of threads per multiprocessor:  1536
    Maximum number of threads per block:           1024
    Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
    Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
    Maximum memory pitch:                          2147483647 bytes
    Texture alignment:                             512 bytes
    Concurrent copy and kernel execution:          Yes with 2 copy engine(s)
    Run time limit on kernels:                     Yes
    Integrated GPU sharing Host Memory:            No
    Support host page-locked memory mapping:       Yes
    Alignment requirement for Surfaces:            Yes
    Device has ECC support:                        Disabled
    Device supports Unified Addressing (UVA):      Yes
    Device supports Managed Memory:                Yes
    Device supports Compute Preemption:            Yes
    Supports Cooperative Kernel Launch:            Yes
    Device PCI Domain ID / Bus ID / location ID:   0 / 1 / 0
    Compute Mode:
        < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

    deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 13.1, CUDA Runtime Version = 13.1, NumDevs = 1
    Result = PASS

It is a lot of work to get it up and running but worth the effort if you have a decent GPU and want to play with for LLM or general programming.

Hope this has helped.  Have pulled my hair out too many times trying to get the recipe to work consistently.  Let me know if it can be improved.

Live Long and Prosper.