# Ubuntu 22.04 Notes
Thinkpad Extreme Gen 3 (Intel Core I9 10th gen, Nvidia GTX 1650)
Python Machine Learning Setup notes for pytorch/tensorflow.

Nvidia has dedicated resources to building/maintaining linux drivers. Nvidia linux forums
are your best resource for troubleshooting. For professional use (development or ML)
prefer Nvidia drivers to other open source drivers (i.e. Nouveau). Always use the latest
stable Nvidia driver. Dont use the latest Cuda version.


## Links
* [Lenovo Support Thinkpad Extreme 3g](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-x-series-laptops/thinkpad-x1-extreme-3rd-gen-type-20tk-20tl/)
* [Nvidia Driver Search](https://www.nvidia.com/en-us/geforce/drivers/)
* [Nvidia Linux Forums](https://forums.developer.nvidia.com/c/gpu-graphics/linux/148)
* [Cuda Releases](https://developer.nvidia.com/cuda-toolkit-archive)
* [Pyenv Build Dependencies](https://github.com/pyenv/pyenv/wiki#suggested-build-environment)

## Setup Bios
1. Ensure bios version is 1.19 or newer (Current version 1.25)
2. Set display device setting to "Discrete" instead of "Hybrid"


## Clean System
Ensure all packages are pristine and system state is fresh

Clean leftover nvidia and unused packages
```bash
$ sudo apt-get remove --purge $(dpkg -l | grep "nvidia" | awk '{print $2}')
$ sudo apt-get remove --purge $(dpkg -l | awk '/^rc/{print $2}')
$ reboot
```

If upgraded from 20.04 or 21.04 check/cleanup:
```bash
/etc/modprobe.d  # Cleanup old module loading/blacklist
/etc/apt/sources.list  # Remove any old sources/repos
/etc/apt/sources.list.d  # Remove any old sources/repos
/etc/default/grub  # Set bootloader options
```

Ensure all packages updated
```bash
$ sudo apt update
$ sudo apt full-upgrade
$ sudo apt autoremove
$ sudo apt autoclean --yes
$ reboot
```

Install basic c/cpp build deps
```bash
sudo apt update; sudo apt install make build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```


## Check Hardware

Ensure video card is detected
```bash
$ sudo lspci | grep -i nvidia
01:00.0 VGA compatible controller: NVIDIA Corporation TU117M [GeForce GTX 1650 Ti Mobile] (rev a1)
01:00.1 Audio device: NVIDIA Corporation Device 10fa (rev a1)
$ lshw -class video
  *-display                 
       description: VGA compatible controller
       product: TU117M [GeForce GTX 1650 Ti Mobile]
       vendor: NVIDIA Corporation
       physical id: 0
       bus info: pci@0000:01:00.0
       version: a1
       width: 64 bits
       clock: 33MHz
       capabilities: pm msi pciexpress vga_controller bus_master cap_list rom
       configuration: driver=nvidia latency=0
       resources: irq:201 memory:ed000000-edffffff memory:c0000000-cfffffff memory:d0000000-d1ffffff ioport:3000(size=128) memory:ee080000-ee0fffff
  *-graphics
       product: EFI VGA
       physical id: 2
       logical name: /dev/fb0
       capabilities: fb
       configuration: depth=32 resolution=3840,2160

```


## Nvidia Install
Do not use ubuntu nvidia packages

Go to [Cuda Releases Page](https://developer.nvidia.com/cuda-toolkit-archive)
* Choose `11.6.2`
* Operating System: `Linux`
* Architecture: `x86_64`
* Distribution: `Ubuntu`
* Version: `20.04`  (Note: works for 22.04)
* Installer Type: `deb [network]`

Run Install commands 
Warning: dont copy paste from this doc, copy paste from Nvidia Cuda page
```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/3bf863cc.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
sudo apt-get update
sudo apt-get -y install cuda
```

When install succeeds, reboot computer

## Verify Nvidia Install
Nvidia-smi will succeed if driver is working
```bash
$ nvidia-smi
Tue Nov 15 11:32:55 2022       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 520.61.05    Driver Version: 520.61.05    CUDA Version: 11.8     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  On   | 00000000:01:00.0  On |                  N/A |
| N/A   46C    P0    10W /  N/A |    736MiB /  4096MiB |      2%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      2288      G   /usr/lib/xorg/Xorg                546MiB |
|    0   N/A  N/A      2426      G   /usr/bin/gnome-shell               61MiB |
|    0   N/A  N/A      3225      G   ...AAAAAAAAA= --shared-files      127MiB |
+-----------------------------------------------------------------------------+
```


## Pyenv Install
Get pyenv:
```bash
$ git clone https://github.com/pyenv/pyenv $HOME/.pyenv
$ export PATH="$PYENV_ROOT/bin:$PATH"' >> $HOME/.bashrc
$ echo 'eval "$(pyenv init -)"' >> $HOME/.bashrc
$ source $HOME/.bashrc
```

Install a python:
```bash
$ pyenv install 3.11.0
```

Set a default python for your shell:
```bash
$ echo 'pyenv global 3.11.0' >> $HOME/.bashrc
$ echo 'pyenv shell 3.11.0' >> $HOME/.bashrc
```

Install basic global packages for your terminal environment:
```bash
$ pip install virtualenv requests ipython
```

## Test pytorch
```bash
$ cd testpytorch/
$ virtualenv env
$ source env/bin/activate
$ pip install torch numpy
```

Verify pytorch Cuda
```bash
# Gpu available in pytorch
$ python -c 'import torch; print(torch.cuda.is_available())'
True
```

Verify pytorch gpu tensor
```bash
$ python -c 'import torch; print(torch.rand((2,4), device=torch.device("cuda")))'
tensor([[0.2024, 0.5186, 0.8549, 0.4732],
        [0.5454, 0.0432, 0.2677, 0.9631]], device='cuda:0')
```

Verify pytorch gpu tensor copy
```
$ python -c 'import torch; print(torch.rand((2,4), device=torch.device("cuda")).cpu())'
tensor([[0.4169, 0.4663, 0.8001, 0.2875],
        [0.8941, 0.8595, 0.9689, 0.7710]])
```
