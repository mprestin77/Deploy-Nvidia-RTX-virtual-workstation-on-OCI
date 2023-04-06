--
duration: PT0H30M0S
description: Learn how to deploy Nvidia RTX virtual workstation on OCI
level: Advanced
roles: Devops;Developer;
lab-id:
products: en/cloud/oracle-cloud-infrastructure/oci;
keywords: OKE
inject-note: true
---

# Deploy NVIDIA RTX Virtual Workstation on OCI

## Introduction

NVIDIA RTX Virtual Workstation software enables users to run high-performance simulations, graphic rendering, and design workloads on cloud, with a native workstation-like performance. It unlocks powerful rendering capabilities provided by graphic APIs such as OpenGL or DirectX, bringing breakthrough graphics performance to the cloud. This article explains how to leverage RTX and NVIDIA Virtual GPU technology using NVIDIA A10 GPU-enabled compute shapes on Oracle Cloud Infrastructure (OCI)


### Task 1: Provision a compute instance on OCI for Nvidia RTX virtual workstation.

To create a vitual cloud network (VCN) and launch a compute instance on OCI see
[Create a VCN](https://docs.oracle.com/en/learn/oci-basics-tutorial/index.html#create-a-vcn)
[Launch COmpute Instance](https://docs.oracle.com/en/learn/oci-basics-tutorial/index.html#launch-compute-instance)

Choose one of available GPU.A10 shapes:

VM.GPU.A10.1
VM.GPU.A10.2
BM.GPU.A10.4

Currently OCI GPU.A10 compute shapes support Oracle Linux, Ubuntu and Rocky Linux. Windows is supported by VM shapes only.

       > **Note:** Rocky Linux is not officially supported by NVIDIA

When provisioning a compute instance on OCI use a standard OS image.  Do not use GPU enabled images because the installed NVIDIA GPU driver does not support RTX virtual workstation (vWS) that requires NVIDIA vGPU driver to be installed.

![Image1](https://user-images.githubusercontent.com/54962742/230504669-1f34055f-47d7-45cc-87d8-652a729b22aa.png)


### Task 2: Download and install NVIDIA vGPU driver.

Download NVIDIA vGPU driver as described in [Downloading NVIDIA vGPU software](https://docs.nvidia.com/grid/latest/grid-software-quick-start-guide/index.html#redeeming-pak-and-downloading-grid-software). If you don’t have an enterprise account with NVIDIA you can register for trial at [Virtual GPU (vGPU) Software Free 90Days Trial - NVIDIA](https://www.nvidia.com/en-us/data-center/resources/vgpu-evaluation).

Log in NVIDIA Enterprise Application HUB using your NVIDIA Enterprise account. Open NVIDIA Licensing Portal and select Software Downloads. Apply the following filters:

•       Product Family: "VGPU"
•       Platform: “Linux KVM"

![Image2](https://user-images.githubusercontent.com/54962742/230505291-440b61f5-3fe4-423a-bb87-b7f0658dc753.png)

Sort by Release Date and download the package with the latest vGPU drivers. For example, currently the latest vGPU version is 15.1. Unzip the file and go to Guest_Drivers folder. There you’ll find vGPU driver installation files for Windows and Linux.

### Task 3: Install NVIDIA vGPU driver on Linux

- Oracle Linux 8

     Copy NVIDIA Linux driver NVIDIA-Linux-x86_64-xxx.xx.xx-grid.run to the provisioned compute instance.
     
     If you are using Oracle Linux 8.7 or later Oracle Linux image, prior to installing NVIDIA driver enable gcc-toolset-11 by running

     scl enable gcc-toolset-11 bash

     You’ll also need to disable nouveau driver that has a conflict with NVIDIA driver. Check if nouveau driver is loaded by running

     lsmod | grep nouveau
     
     If it shows nouveau driver in the output of the command, you’ll need to disable it first. To disable nouveau driver on Oracle Linux create the /etc/modprobe.d/blacklist-nouveau.conf file and add the content below:
     
     blacklist nouveau
     options nouveau modeset=0
     
     Save the file and re-generate initramfs.
     sudo dracut --force

     After disabling the driver reboot the server
     sudo reboot
     
     Install NVIDIA vGPU driver by running:
     sudo bash ./NVIDIA-Linux-x86_64-xxx.xx.xx-vgpu-kvm.run
     Ignore warnings and hit OK to continue with the installation.
     
     Reboot the server
     sudo reboot 




