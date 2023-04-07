--
duration: PT0H30M0S
description: Learn how to deploy Nvidia RTX virtual workstation on OCI
level: Advanced
roles: Devops;Developer;
lab-id:
products: en/cloud/oracle-cloud-infrastructure/oci;
keywords: Nvidia GPU RTX 
inject-note: true
---

# Deploy NVIDIA RTX Virtual Workstation on OCI

## Introduction

NVIDIA RTX Virtual Workstation software enables users to run high-performance simulations, graphic rendering, and design workloads on cloud, with a native workstation-like performance. It unlocks powerful rendering capabilities provided by graphic APIs such as OpenGL or DirectX, bringing breakthrough graphics performance to the cloud. This article explains how to leverage RTX and NVIDIA Virtual GPU technology using NVIDIA A10 GPU-enabled compute shapes on Oracle Cloud Infrastructure (OCI)


### Provision a compute instance on OCI for Nvidia RTX virtual workstation.

To create a vitual cloud network (VCN) and launch a compute instance on OCI see

[Create a VCN](https://docs.oracle.com/en/learn/oci-basics-tutorial/index.html#create-a-vcn)

[Launch Compute Instance](https://docs.oracle.com/en/learn/oci-basics-tutorial/index.html#launch-compute-instance)

Choose one of available GPU.A10 shapes:
```
VM.GPU.A10.1

VM.GPU.A10.2

BM.GPU.A10.4
```
If your tenancy does not have service limit set for GPU.A10, these shapes will not be in the shape list. To check your tenancy limits in OCI Console, set the region where you are going to provision a GPU.A10 compute instance, open the navigation menu and click Governance & Administration. Under Tenancy Management select Limits, Quotas and Usage. Set the service to Compute, select one of availability domains in the scope field, and type GPU.A10 in the resource field. Select GPUs for A10 based VM and BM instances:

![Image7](https://user-images.githubusercontent.com/54962742/230644511-12752c7e-6331-40bb-89a5-e57d1322da89.png)

Compute limits are per availability domain. Check if the limit is set in any of availability domains of the region. If the service limit is set to 0 for all availability domains, you can click on request a service limit increase link and submit a request for limit increase for this resource.

     Note: in order to access Limits, Quotas and Usage you need to be a tenancy Admininstrator or to have a policy added for your user group to read LimitsAndUsageViewers 

For more information about service limits, see [Service Limits](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/servicelimits.htm#ViewingYourServiceLimitsQuotasandUsage). 
     
Currently OCI GPU.A10 compute shapes support Oracle Linux, Ubuntu and Rocky Linux. Windows is supported by VM shapes only.

     Note: Rocky Linux is not officially supported by NVIDIA

When provisioning a compute instance on OCI use a standard OS image.  Do not use GPU enabled images because the installed NVIDIA GPU driver does not support RTX virtual workstation (vWS) that requires NVIDIA vGPU driver to be installed.

![Image1](https://user-images.githubusercontent.com/54962742/230504669-1f34055f-47d7-45cc-87d8-652a729b22aa.png)


### Download and install NVIDIA vGPU driver.

Download NVIDIA vGPU driver as described in [Downloading NVIDIA vGPU software](https://docs.nvidia.com/grid/latest/grid-software-quick-start-guide/index.html#redeeming-pak-and-downloading-grid-software). If you don’t have an enterprise account with NVIDIA you can register for trial at [Virtual GPU (vGPU) Software Free 90Days Trial - NVIDIA](https://www.nvidia.com/en-us/data-center/resources/vgpu-evaluation).

Log in NVIDIA Enterprise Application HUB using your NVIDIA Enterprise account. Open NVIDIA Licensing Portal and select Software Downloads. Apply the following filters:

•       Product Family: "VGPU"
•       Platform: “Linux KVM"

![Image2](https://user-images.githubusercontent.com/54962742/230505291-440b61f5-3fe4-423a-bb87-b7f0658dc753.png)

Sort by Release Date and download the package with the latest vGPU drivers. For example, currently the latest vGPU version is 15.1. Unzip the file and go to Guest_Drivers folder. There you’ll find vGPU driver installation files for Windows and Linux.

### Install NVIDIA vGPU driver on Linux

***Oracle Linux 8***

  Copy NVIDIA Linux driver NVIDIA-Linux-x86_64-xxx.xx.xx-grid.run to the provisioned compute instance.
     
  If you are using Oracle Linux 8.7 or later Oracle Linux image, prior to installing NVIDIA driver enable gcc-toolset-11 by running
  ```
  scl enable gcc-toolset-11 bash
  ```
  You’ll also need to disable nouveau driver that has a conflict with NVIDIA driver. Check if nouveau driver is loaded by running
  ```
  lsmod | grep nouveau
  ```   
  If it shows nouveau driver in the output of the command, you’ll need to disable it first. To disable nouveau driver on Oracle Linux create the /etc/modprobe.d/blacklist-nouveau.conf file and add the content below:
  ```   
  blacklist nouveau
  
  options nouveau modeset=0
  ```  
  Save the file and re-generate initramfs:
  ``` 
  sudo dracut --force
  ``` 
  After disabling the driver reboot the server:
  ```   
  sudo reboot
  ```      
  Install NVIDIA vGPU driver by running:
  ``` 
  sudo bash ./NVIDIA-Linux-x86_64-xxx.xx.xx-vgpu-kvm.run
  ``` 
  Ignore warnings and hit OK to continue with the installation. Reboot the server:
  ``` 
  sudo reboot 
  ``` 
  
***Rocky Linux 9***

  Copy NVIDIA Linux driver NVIDIA-Linux-x86_64-xxx.xx.xx-grid.run to the provisioned compute instance.

  Install Linux headers matching the version of Linux kernel:
  ```
  yum dnf kernel-devel-$(uname -r)
  ```
  If it fails to find Linux headers matching the kernel version, upgrade Linux kernel and reboot the server:
  ```
  sudo dnf install kernel

  sudo reboot
  ```
  After reboot reinstall Linux headers to match Linux kernel version:
  ```
  sudo install kernel-devel -y   
  ```
  Check if nouveau driver is loaded by running:
  ```
  lsmod | grep nouveau
  ``` 
  If it shows nouveau driver in the output of the command, you’ll need to disable it first. To disable nouveau driver on Oracle Linux create the /etc/modprobe.d/blacklist-nouveau.conf file and add the content below:
  ```
  blacklist nouveau
  
  options nouveau modeset=0
  ```
  Save the file and re-generate initramfs:
  ```
  sudo dracut --force
  ```
  After disabling the driver reboot the server:
  ```
  sudo reboot
  ```
  Install NVIDIA vGPU driver by running:
  ```  
  sudo bash ./NVIDIA-Linux-x86_64-xxx.xx.xx-vgpu-kvm.run
  ```
  Ignore warnings and hit OK to continue with the installation. Reboot the server:
  ```
  sudo reboot 
  ```

***Ubuntu 22***

  Copy NVIDIA Linux driver NVIDIA-Linux-grid-xxx.xx.xx_amd64.deb to the provisioned compute instance.
  
  Check if nouveau driver is loaded by running
  ```
  lsmod | grep nouveau
  ```
  If it shows nouveau driver in the output of the command, you’ll need to disable it first. To disable nouveau driver on Oracle Linux create the /etc/modprobe.d/blacklist-nouveau.conf file and add the content below:
  ```
  blacklist nouveau
  
  options nouveau modeset=0
  ```  
  Save the file and re-generate initramfs:
  ```
  sudo dracut --force
  ```
  After disabling the driver reboot the server:
  ```
  sudo reboot
  ```
  Install NVIDIA vGPU driver by running:
  ```
  sudo bash ./NVIDIA-Linux-grid-xxx.xx.xx_amd64.deb
  ```
  Reboot the server
  ```
  sudo reboot 
  ```
  
### Verify that NVIDIA vGPU driver is installed

Verify NVIDIA vGPU driver installation using nvidia-smi command:

![Image3](https://user-images.githubusercontent.com/54962742/230516122-d74e38c2-57e7-49f5-a7da-56074ba5c347.png)


### Enable NVIDIA RTX Virtual Workstation

To enable NVIDIA RTX Virtual Workstation feature edit /etc/nvidia/gridd.conf 
```
sudo vi /etc/nvidia/gridd.conf 
```
and add a line:
```
FeatureType=2 
```
Save the changes and close the file. 

Check if GSP firmware is enabled:
```
nvidia-smi -q | grep GSP
```
If GSP firmware is enabled the command displays GSP firmware version:
```
 GSP Firmware Version                  : 525.85.05
```
If GSP firmware is enabled, disable it by setting the NVIDIA module parameter  NVreg_EnableGpuFirmware to 0. Set this parameter by editing /etc/modprobe.d/nvidia.conf file:
```
sudo vi /etc/modprobe.d/nvidia.conf  
```
adding the following line to it:
```
options nvidia NVreg_EnableGpuFirmware=0
```
If the /etc/modprobe.d/nvidia.conf file does not already exist, create it.

After disabling GSP you must reboot the server
```
sudo reboot
```
Download the client configuration token from NVIDIA Licensing Portal or DLS appliance. For information how register NVIDIA vGPU license, refer to **Registering with NVIDIA vGPU Software License Server**

Copy the client configuration token to the default location in /etc/nvidia/ClientConfigToken and set the file permissions to 744:
```
sudo chmod 744 /etc/nvidia/ClientConfigToken/client_configuration_token_*.tok 
```
  Note: If you want to store the client configuration token in a custom location, copy the token to the directory that you created and set ClientConfigTokenPath configuration parameter in /etc/nvidia/gridd.conf to point to this directory. 

Restart nvidia-gridd service:
```
sudo systemctl restart nvidia-gridd
```
Run "nvidia-smi -q" and check that the Product Brand is set to NVIDIA RTX and License Status displays "Licensed".

![Image4](https://user-images.githubusercontent.com/54962742/230515935-aed4e254-57f9-4073-a528-083dbe4f13c9.png)


If it fails to obtain the license and shows License Status “Unlicensed” check nvidia-gridd service log:
```
sudo grep gridd /var/log/messages
```

### Install NVIDIA vGPU driver on Windows
Copy the NVIDIA Windows driver package to the guest VM or physical host Run the uploaded NVIDIA vGPU driver installation *.exe file and use Custom Installation option. Make sure that both Graphics Driver and RTX Desktop Manager are selected (see the screenshot below)

OCI A10 GPU VM is configured with GPU passthrough, and therefore you must set the vGPU driver behavior via regedit. For more information see [Virtual GPU Client Licensing User Guide](https://docs.nvidia.com/grid/latest/grid-licensing-user-guide/index.html)

Add the FeatureType DWord (REG_DWORD) registry value to the Windows registry key:
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\nvlddmkm\Global\GridLicense
```
Set this value to 2 to enable NVIDIA RTX Virtual Workstation license. 
Download the client configuration token from NVIDIA Licensing Portal or DLS appliance. For information how register NVIDIA vGPU license, refer to Registering with NVIDIA vGPU Software License Server
Copy the client configuration token to the folder
```
%SystemDrive%:\Program Files\NVIDIA Corporation\vGPU Licensing\ClientConfigToken 
```
From a command line or powershell run "nvidia-smi -q" and check that the Product Brand is set to NVIDIA RTX and License Status displays "Licensed"

  Note: On Windows nvidia-smi.exe is installed by default in c:\Program Files\NVIDIA Corporation\NVSMI folder


If it fails to obtain the license and shows License Status “Unlicensed” check licensing messages in the log:
```
%SystemDrive%\Users\Public\Documents\NvidiaLogging\Log.NVDisplay.Container.exe.log
```




