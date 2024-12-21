# How to Passtrhough an AMD GPU to a Windows 11 VM

## Credits

This guide is mainly based on the [Ultimate Beginner's Guide to GPU Passthrough on Reddit](https://www.reddit.com/r/homelab/comments/b5xpua/the_ultimate_beginners_guide_to_gpu_passthrough/). Thanks to the original author for the invaluable resource!

### Disclaimer

In no way, shape, or form does this guide claim to work for all instances of Proxmox/GPU configurations. Use at your own risk. I am not responsible if you blow up your server, your home, or yourself.

## Let's Get Started (Pre-configuration Checklist)

It's important to make note of all your hardware/software setup before we begin the GPU passthrough. For reference, I will list what I am using for hardware and software. This guide may or may not work the same on any given hardware/software configuration, and it is intended to help give you an overall understanding and basic setup of GPU passthrough for Proxmox only.

Your hardware should, at the very least, support: VT-d, interrupt mapping, and UEFI BIOS.

### My Hardware Configuration:

The main system is a [Minisforum MS-01](https://store.minisforum.de/products/ms-01) with the following configuration:

- **CPU**: Intel Core [i5 12600H](https://www.intel.de/content/www/de/de/products/sku/96156/intel-core-i512600h-processor-18m-cache-up-to-4-50-ghz/specifications.html)
- **Memory**: [32GB DDR5](https://www.amazon.de/dp/B09RVNMGFH?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1)
- **GPU**: 1x AMD Radeon [RX 6400](https://www.amazon.de/XFX-Speedster-SWFT105-Grafikkarte-RX-64XL4SFG2/dp/B09Y7358KJ/?_encoding=UTF8&pd_rd_w=8N6Zj&content-id=amzn1.sym.16038c01-cfea-4f09-a119-c9f8c051c46c%3Aamzn1.symc.fc11ad14-99c1-406b-aa77-051d0ba1aade&pf_rd_p=16038c01-cfea-4f09-a119-c9f8c051c46c&pf_rd_r=BNCCBXX7QJFESM0CX5VY&pd_rd_wg=VM7m9&pd_rd_r=82033626-c11e-4552-8955-92dd5c3dd5e5&ref_=pd_hp_d_atf_ci_mcx_mr_ca_hp_atf_d&th=1)

## Strange Behavior with the MS-01

I've noticed some strange behavior with my MS-01 when inserting GPUs. If you're working with the same model, make sure to enable both 1Gb ports before inserting the GPU. On my machine, this causes one of the adapters to change its name, and a new one is created. 

If you're connected to only one adapter, it’s possible that the Ethernet adapter may no longer be recognized, causing the machine to lose its internet connection. By enabling both ports, you can switch between them to find one that works. Typically, one of them will always function. In the GUI, you can activate the newly detected network port.

A similar issue occurs with the 10Gb ports, but in this case, both ports are renamed. If you’re using or have configured these ports, make sure to update your configuration accordingly. If anyone has insight into the cause of this behavior, feel free to reach out.

Configuration without GPU:
![image](https://github.com/user-attachments/assets/19037910-d38a-42aa-bcc2-7bca03062497)

Configuration with GPU:
![image](https://github.com/user-attachments/assets/a07a5f08-fbcd-4306-8d1b-f6d874f8d654)

### My Software Configuration:

- **Latest Proxmox Build**: 8.3 as of this writing
- **Windows 11 Pro** (Virtual Machine)

### Notes:

1. It is not recommended to build in the GPU before reaching [Step 5](https://github.com/techbygiusi/Guides/blob/main/Proxmox%20AMD%20GPU%20Passthrough/README.md#step-5-adding-gpu-to-vfio) on an MS-01. If so the iGPU is beeing deactivated and only the display output through the GPU will work. 😅
2. Any Windows 11 installation ISO should work, however, try to stick to the latest available ISO from Microsoft.

## Configuring Proxmox

This guide assumes you already have at the very least, installed Proxmox on your server and are able to login to the WebGUI and have access to the server node's Shell terminal. If you need some guides, i can recommend to watch the following videos:

- [5 Things I Would Do On Fresh Install Of ProxMox](https://www.youtube.com/watch?v=xD9Xyt2mdSI&t=273s)
- [Proxmox VE Install and Setup Tutorial](https://www.youtube.com/watch?v=7OVaWaqO2aU)

## Step 1: Configuring the GRUB Bootloader

1. Open the GRUB configuration file using a text editor:
   ```shell
   nano /etc/default/grub
   ```

2. Locate the following line:
   ```shell
   GRUB_CMDLINE_LINUX_DEFAULT="quiet"
   ```

3. Replace it with:
   ```shell
   GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
   ```

4. Save the file by pressing `CTRL + X`, then confirm with `Y` to write the changes.

5. Update the GRUB configuration:
   ```shell
   update-grub
   ```

## Step 2: VFIO Modules

1. Open the modules file:
   ```shell
   sudo nano /etc/modules
   ```

2. Add the following lines to the file:
   ```shell
   vfio
   vfio_iommu_type1
   vfio_pci
   vfio_virqfd
   ```

## Step 3: IOMMU Interrupt Remapping

1. Create the configuration file for unsafe interrupts:
   ```shell
   echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
   ```

2. Create the configuration file to ignore MSRs:
   ```shell
   echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf
   ```

## Step 4: Blacklisting Drivers

1. Blacklist the following drivers by appending to the blacklist file:
   ```shell
   echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
   echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
   echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
   echo "blacklist amdgpu" >> /etc/modprobe.d/blacklist.conf
   ```

## Step 5: Adding GPU to VFIO

1. **Shut Down the Machine**:
   - Power off the machine completely. You can use the following command to shut it down:
     ```shell
     shutdown now
     ```

2. **Open the Machine**:
   - Once the machine is powered off, physically open the case.

3. **Install the GPU**:

4. **Start the Machine Again**:
   - Once the built-in GPU is installed and connected, power the machine back on.

5. **Verify the GPU is Active**:
   - After the system boots back up, check if the built-in GPU is recognized by running:
     ```shell
     lspci | grep VGA
     ```
   - This should display the integrated GPU as well as any other attached GPUs.
      ![image](https://github.com/user-attachments/assets/70abf6f6-7678-4bec-bae8-bfe2e5d19c52)

6. Check the list of PCI devices:
   ```shell
   lspci -v
   ```
   ![image](https://github.com/user-attachments/assets/3a3e7ed3-4f63-4180-bc44-c96ddc1fca6e)


7. Get detailed information for the specific device (replace `03:00` with the correct PCI address):
   ```shell
   lspci -n -s 03:00
   ```
   <img width="226" alt="image" src="https://github.com/user-attachments/assets/0e7cbedc-f096-4bc2-8651-784bcd61b241" />


8. Add the GPU to VFIO by creating a configuration file:
   ```shell
   echo "options vfio-pci ids=1002:743f,1002:ab28 disable_vga=1" >> /etc/modprobe.d/vfio.conf
   ```

9. Update the initial RAM filesystem:
   ```shell
   update-initramfs -u
   ```

10. Reset the system:
    ```shell
    reset
    ```

## Step 6: Configuring the VM (Windows 11)

Now comes the 'fun' part. It took me many, many different configuration attempts to get things just right. Hopefully my pain will be your gain, and help you get things done right, the first time around.

1. Create a VM

   Creating a Virtual Machine is pretty easy and self-explanatory, but if you're having issues, I suggest looking up the official Proxmox Wiki and How-To guides.
   For this guide, you'll need a Windows ISO for your Virtual Machine and the VirtIO Drivers.

   - Windows 11 Image:  [Download Windows 11](https://www.microsoft.com/en-us/software-download/windows11)
   - VirtIO Drivers: [Download Windows VirtIO Drivers](https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers)

4. Enable OMVF (UEFI) for the VM
   
   Under your VM's **OS Tab**, configure the following settings:
   
   - **Storage**: local
   - **ISO Image**: Win11_24H2_German (or your downloaded ISO file)
   - **Type**: Microsoft Windows
   
   #### Add an additional drive for VirtIO drivers:
   - **Storage**: local
   - **ISO Image**: virtio-win.iso
   
   ![image](https://github.com/user-attachments/assets/4a81f5de-3b0b-4622-8ad9-d604b96c05ea)
   
   Next, go to the **System Tab** and configure the following:
   
   - **Graphic Card**: Default (After the last GPU configuration, this can be set to `none`)
   - **Machine**: q35
   - **SCSI Controller**: VirtIO SCSI Single
   - **BIOS**: OVMF (UEFI)

   ![OS Tab Configuration](https://github.com/user-attachments/assets/dcd6d37d-bf50-4635-b3d5-a3eb0850c7e2)
   
   Everything else can be set according to your preference. After that, you can start the VM and proceed with the Windows 11 installation.
   If you want to bypass the Microsoft login, use the `SHIFT` + `F10` combo to open the command prompt and enter the following command:

   ```shell
   OOBE\BYPASSNRO
   ```

   But pssst, don't tell Microsoft. 😏
   **Important**: Ensure that the network card of the VM is deactivated before performing this step.
   After completing this, you can continue with the rest of the guide.

6. Edit the VM Config File

   Going back to the Shell window, we need to edit `/etc/pve/qemu-server/<vmid>.conf`, where `<vmid>` is the VM ID Number you used during the VM creation (General Tab).

   ```shell
   nano /etc/pve/qemu-server/<vmid>.conf
   ```

   In the editor, add these command lines (it doesn't matter where you add them, as Proxmox will reorganize them after you save):

   ```shell
   machine: q35
   cpu: host,hidden=1,flags=+pcid
   args: -cpu 'host,+kvm_pv_unhalt,+kvm_pv_eoi,hv_vendor_id=NV43FIX,kvm=off'
   ```
7. Add PCI Devices (Your GPU) to VM

   ![image](https://github.com/user-attachments/assets/24820e5d-fb42-452a-ae6f-84df7255c5d6)

   Under the VM's **Hardware Tab/Window**, click on the **Add** button towards the top. Then, in the drop-down menu, click **PCI Device**.
   
   Look for your GPU in the list, and select it. On the PCI options screen, you should configure it as follows:

   ![image](https://github.com/user-attachments/assets/0c4b8a7f-be3d-4c73-aaf7-cdf104eb611d)
   
   - **All Functions**: YES
   - **Rom-Bar**: YES
   - **Primary GPU**: NO
   - **PCI-Express**: YES (requires `machine: q35` in the VM config file)

8. Reboot the VM

   After rebooting the VM, the new configuration must be set. In conclusion, you can download the [AMD Driver Software](https://www.amd.com/en/support/download/drivers.html) and install it. If everything works correctly, the GPU should be recognized by the installer.

   ![AMD Driver Software](https://github.com/user-attachments/assets/b4dd9931-961e-4996-8829-836f8abbf711)

   After installing the driver, the GPU should also be correctly shown in the **Task Manager** of the Windows VM.

   ![GPU in Task Manager](https://github.com/user-attachments/assets/9cb05bfa-d742-41fa-acbd-3d726d0014ca)

   


