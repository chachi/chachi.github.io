---
layout: post
title: Using CUDA on AWS
---

***Update: Includes additional CUDA packages that had been missed in original version.***

At my new position with [Replica Labs](http://replicalabs.com), we're setting up some cloud-based systems for some heavy GPU computations and are leveraging the GPU instances provided by Amazon Web Services. For the most part, these are just headless Ubuntu 14.04 instances. I thought I'd document getting my process of getting proprietary NVIDIA drivers and the CUDA SDK installed and running here.

*Disclaimer: These are the steps that I took, but they may not work for you. Information provided without warranty.*

1. Use `ubuntu-drivers` to check what GPU model your server is equipped with.

  Example output:

```bash
user@host:~$ sudo ubuntu-drivers devices
== /sys/devices/pci0000:00/0000:00:03.0 ==
vendor   : NVIDIA Corporation
model    : GK104GL [GRID K520]
modalias : ...
driver   : nvidia-331-updates - distro non-free
driver   : xserver-xorg-video-nouveau - distro free builtin
driver   : nvidia-331 - distro non-free recommended
```

  So we can see that our AWS instance has an NVIDIA GRID K520 GPU.

1. Find the recommended driver version from NVIDIA at [http://www.nvidia.com/Download/index.aspx?lang=en-us]()

1. Download and install the driver.
1. `sudo sh NVIDIA-Linux-x86_64-331.89.run` (insert your driver script name) to install the driver.
  - Don't worry if the "preinstall" script fails, just continue.
  - Yes, install DKMS module
  - Yes, install 32-bit compatibility libraries
  - Yes, run `nvidia-xconfig` to configure the X server.

1. Install the CUDA packages: `libcudart5.5`, `nvidia-cuda-dev` and `nvidia-cuda-toolkit`.
1. Reboot the server.
1. Profit.

I have tried various other means of installing drivers including:

- Various of Ubuntu's default repository drivers (331 and 331-updates)
- xorg-edgers PPA drivers (which said they were the correct version)

and none of them worked for me, unfortunately. I'd much rather install software through `apt-get`, but these drivers have forced me to do otherwise.

