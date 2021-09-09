---
layout: single
title:  "WSL2 + CUDA + Pytorch"
excerpt: ""
categories: tutorial
tags: WSL2 CUDA setup python machine_learning deep_learning
---

{% include toc title="Table of Contents" %}

I recently installed Windows on my laptop - Asus ROG Strix GL553VD (7700HQ, FHD, GTX 1050). As I do software development on both Windows and Ubuntu, I wanted to use both simultaneously instead of using the option of dual boot or VirtualBox. 
I wanted to build Deep Learning (Pytorch/Tensorflow) models and using a GPU with Pytorch requires an NVIDIA GPU and the CUDA toolkit installed.
Hence, below is summary of the setup.The following librabry/packages will be installed:
-WSL2
-CUDA 
-Windows Terminal
-Miniconda
-PyTorch
 


# Microsoft Windows Insider Preview OS Build
Register for the [Windows Insider Program](https://insider.windows.com/en-us/getting-started/#register) and choose dev channel as your [flighting channel](https://docs.microsoft.com/en-us/windows-insider/flighting#switching-between-channels) (previously fast rings).
Install the latest build from the [Dev Channel](https://insider.windows.com/en-us/getting-started#install).

# NVIDIA Drivers for CUDA
Download the NVIDIA Driver from the download section on the [CUDA on WSL](https://developer.nvidia.com/cuda/wsl) page. Choose the appropriate driver depending on the type of NVIDIA GPU in your system - GeForce and Quadro.
Install the driver using the (for.eg 510.06_gameready_win11_win10-dch_64bit_international.exe) executable on the Windows machine. The Windows Display Driver will install both the regular driver components for native Windows and for WSL support.
CUDA support enables GPU accelerated computing for data science, machine learning and inference solutions. DirectX support enables graphics on WSL2 by supporting DX12 APIs. 


# WSL2

## install WSL2
Install WSL 2 by following the instructions in the Microsoft documentation available [here](https://docs.microsoft.com/windows/wsl/install-win10).

Open PowerShell as Administrator and run:

### Step 1 - Enable the Windows Subsystem for Linux
You must first enable the "Windows Subsystem for Linux" optional feature before installing any Linux distributions on Windows.
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

### Step 2 - Enable Virtual Machine feature
Before installing WSL 2, you must enable the Virtual Machine Platform optional feature. Your machine will require virtualization capabilities to use this feature.
```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
Restart your machine to complete the WSL install and update to WSL 2.

### Step 3 - Download the Linux kernel update package
Download the latest package: WSL2 Linux kernel update package for x64 machines(https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)
Run the update package downloaded in the previous step. (Double-click to run - you will be prompted for elevated permissions, select ‘yes’ to approve this installation.)

### Step 4 - Set WSL 2 as your default version
Once the installation is complete, move on to the next step - setting WSL 2 as your default version when installing new Linux distributions.
```poweshell
wsl --set-default-version 2
```

### Step 5 - Install your Linux distribution of choice
Open the [Microsoft Store](https://aka.ms/wslstore) and select [Ubuntu 20.04 LTS](https://www.microsoft.com/store/apps/9n6svws3rx71) as the Linux distribution. 
From the distribution's page, select "Get". This wikk download and install Ubuntu to your system.
The first time you launch a newly installed Linux distribution, a console window will open and you'll be asked to wait for a minute or two for files to de-compress and be stored on your PC. All future launches should take less than a second.
You will then need to create a user account and password for your new Linux distribution.


## Windows Terminal (Optional) 
Windows Terminal enables multiple tabs (quickly switch between multiple Linux command lines, Windows Command Prompt, PowerShell, Azure CLI, etc), create custom key bindings (shortcut keys for opening or closing tabs, copy+paste, etc.), use the search feature, and custom themes (color schemes, font styles and sizes, background image/blur/transparency).
Install Windows Terminal(https://docs.microsoft.com/en-us/windows/terminal/get-started)


## Move WSL2 Safely to another location (Optional)
It is real pain when you have small SSD and Windows Subsystem for Linux (WSL) is growing exponentially in size. The following method can be used to move the WSL installation to another drive. 

Open PowerShell as Administrator and run:

### Step 1 - Check if the WSL 2 installation you are planning to move is is currently running/stopped.
```powershell
```

### Step 2 - If its running then you must stop the particular WSL distribution. (Ubuntu used as example)
```powershell
wsl -t Ubuntu
```

### Step 3 - Export to some folder. (Here exporting Ubuntu as ubuntu-ex.tar to Z:wsl2)
```powershell
wsl --export Ubuntu "G:\WSL2\ubuntu-ex.tar"
```

### Step 4 - Unregister previous WSL installation
```powershell
wsl --unregister Ubuntu
```

### Step 5 - Create a new folder and import your WSL installation to that folder
```powershell
wsl --import Ubuntu "G:\WSL2\wsl_ubuntu" "G:\WSL2\ubuntu-ex.tar"
```

### Step 6 - Check after import is complete
```powershell
wsl -l -v
```

### Step 7 - Mark one of your WSL distribution as (default)
```powershell
wsl -s Ubuntu
```

### Step 8 - After exporting your default user will be set as root , to change it to your desired username, run following command
```powershell
ubuntu config --default-user user
```

### Step 9 - Finally run wsl and you have successfully moved your WSL 2 installation to another drive
```powershell
wsl
```


## CUDA on WSL2

### Step 1 - Run WSL 2
```powershell
wsl
```

### Step 2 - Installing CUDA toolkit

You are installing CUDA toolkit from our regular package, note that for WSL 2, you should use the cuda-toolkit-<version> meta-package to avoid installing the NVIDIA driver that is typically bundled with the toolkit.
Execute each line below in WSL2 terminal

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.4.0/local_installers/cuda-repo-ubuntu2004-11-4-local_11.4.0-470.42.01-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu2004-11-4-local_11.4.0-470.42.01-1_amd64.deb
sudo apt-key add /var/cuda-repo-ubuntu2004-11-4-local/7fa2af80.pub
sudo apt-get update
```

Note: Do not choose the “cuda”, “cuda-11-0”, or “cuda-drivers” meta-packages under WSL 2 if you are installing the regular CUDA toolkit as these packages will result in an attempt to install the Linux NVIDIA driver under WSL 2.

```bash
apt-get install -y cuda-toolkit-11-4
```

### Step 3 - Testing the CUDA installation on WSL2
Build the CUDA samples available under /usr/local/cuda/samples from your installation of the CUDA Toolkit in the previous section. The BlackScholes application is located under /usr/local/cuda/samples/4_Finance/BlackScholes. 
Alternatively, you can transfer a binary built on Linux to WSL 2!

```bash
cd /usr/local/cuda-11.4/samples/4_Finance/BlackScholes

make BlackScholes

./BlackScholes
```

If everything is installed properly then a similar output will displayed on WSL2 terminal

```bash
[./BlackScholes] - Starting...
GPU Device 0: "Pascal" with compute capability 6.1

Initializing data...
...allocating CPU memory for options.
...allocating GPU memory for options.
...generating input data in CPU mem.
...copying input data to GPU mem.
Data init done.

Executing Black-Scholes GPU kernel (512 iterations)...
Options count             : 8000000
BlackScholesGPU() time    : 1.136816 msec
Effective memory bandwidth: 70.371963 GB/s
Gigaoptions per second    : 7.037196

BlackScholes, Throughput = 7.0372 GOptions/s, Time = 0.00114 s, Size = 8000000 options, NumDevsUsed = 1, Workgroup = 128
Reading back GPU results...
Checking the results...
...running CPU calculations.

Comparing the results...
L1 norm: 1.741792E-07
Max absolute error: 1.192093E-05

Shutting down...
...releasing GPU memory.
...releasing CPU memory.
Shutdown done.

[BlackScholes] - Test Summary

NOTE: The CUDA Samples are not meant for performance measurements. Results may vary when GPU Boost is enabled.

Test passed
```


## Miniconda
Anaconda is a full distribution of the central software in the PyData ecosystem, and includes Python itself along with the binaries for several hundred third-party open-source projects. 
Miniconda is essentially an installer for an empty conda environment, containing only Conda, its dependencies, and Python
In other words, Miniconda is a mini version of Anaconda. Miniconda ships with just the repository management system and a few packages. Whereas, with Anaconda, you have the distribution of some 150 built-in packages.

I will use Miniconda (because it is much more light weight than Anaconda) when I need to setup python programming environment 

### Step 1 - Download the latest shell script
```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

### Step 2 - Make the miniconda installation script executable
```bash
chmod +x Miniconda3-latest-Linux-x86_64.sh
```

### Step 3 - Run miniconda installation script
```bash
./Miniconda3-latest-Linux-x86_64.sh
```

### Step 4 - Create a conda environment
It’s a good idea to use a separate environment for different projects. Create a new conda environment for deep learning stuff:
```bash
conda create --name ENV_NAME
```

### Step 4 - Activate the conda environment
```bash
conda activate cnn
```


## Pytorch 

```bash
conda install pytorch torchvision torchaudio cudatoolkit=10.2 -c pytorch
```

## Verify that everything works
Start python in WSL2 and run the following command
```python
import torch
torch.cuda.is_available()
```


# References
https://docs.microsoft.com/en-us/windows/wsl/install-win10
https://docs.nvidia.com/cuda/wsl-user-guide/index.html#installing-nvidia-drivers
https://avinal.space/posts/development/wsl1.html#:~:text=%20Move%20WSL%202%20Safely%20to%20another%20Drive,to%20move%20is%20is%20currently%20running%2Fstopped.%20More%20
