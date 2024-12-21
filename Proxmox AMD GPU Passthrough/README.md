# How to Passtrhough an AMD GPU to a Windows 11 VM

## Credits

This guide is mainly based on the [Ultimate Beginner's Guide to GPU Passthrough on Reddit](https://www.reddit.com/r/homelab/comments/b5xpua/the_ultimate_beginners_guide_to_gpu_passthrough/). Thanks to the original author for the invaluable resource!

### Disclaimer

In no way, shape, or form does this guide claim to work for all instances of Proxmox/GPU configurations. Use at your own risk. I am not responsible if you blow up your server, your home, or yourself. Surgeon General Warning: do not operate this guide while under the influence of intoxicating substances. Do not let your cat operate this guide. You have been warned.

## Let's Get Started (Pre-configuration Checklist)

It's important to make note of all your hardware/software setup before we begin the GPU passthrough. For reference, I will list what I am using for hardware and software. This guide may or may not work the same on any given hardware/software configuration, and it is intended to help give you an overall understanding and basic setup of GPU passthrough for Proxmox only.

Your hardware should, at the very least, support: VT-d, interrupt mapping, and UEFI BIOS.

### My Hardware Configuration:

The main system is a [Minisforum MS-01](https://store.minisforum.de/products/ms-01) with the following configuration:

- **CPU**: Intel Core [i5 12600H](https://www.intel.de/content/www/de/de/products/sku/96156/intel-core-i512600h-processor-18m-cache-up-to-4-50-ghz/specifications.html)
- **Memory**: [32GB DDR5](https://www.amazon.de/dp/B09RVNMGFH?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1)
- **GPU**: 1x AMD Radeon [RX 6400](https://www.amazon.de/XFX-Speedster-SWFT105-Grafikkarte-RX-64XL4SFG2/dp/B09Y7358KJ/?_encoding=UTF8&pd_rd_w=8N6Zj&content-id=amzn1.sym.16038c01-cfea-4f09-a119-c9f8c051c46c%3Aamzn1.symc.fc11ad14-99c1-406b-aa77-051d0ba1aade&pf_rd_p=16038c01-cfea-4f09-a119-c9f8c051c46c&pf_rd_r=BNCCBXX7QJFESM0CX5VY&pd_rd_wg=VM7m9&pd_rd_r=82033626-c11e-4552-8955-92dd5c3dd5e5&ref_=pd_hp_d_atf_ci_mcx_mr_ca_hp_atf_d&th=1)

### My Software Configuration:

- **Latest Proxmox Build**: 8.3 as of this writing
- **Windows 11 Pro** (Virtual Machine)

### Notes:

1. It is not recommended build in the GPU before beeing on Step 5.
2. Any Windows 11 installation ISO should work, however, try to stick to the latest available ISO from Microsoft.

## Configuring Proxmox

This guide assumes you already have at the very least, installed Proxmox on your server and are able to login to the WebGUI and have access to the server node's Shell terminal. If you need some guides, i can recommend to watch the following videos:

- [5 Things I Would Do On Fresh Install Of ProxMox](https://www.youtube.com/watch?v=xD9Xyt2mdSI&t=273s)
- [Proxmox VE Install and Setup Tutorial](https://www.youtube.com/watch?v=7OVaWaqO2aU)

## Step 1: Configuring the GRUB Bootloader

1. Open the GRUB configuration file using a text editor:
   ```shell
   sudo nano /etc/default/grub
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
   sudo update-grub
   ```

##Step 2: VFIO Modules

nano /etc/modules
   vfio
   vfio_iommu_type1
   vfio_pci
   vfio_virqfd

##Step 3: IOMMU interrupt remapping

echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf

##Step 4: Blacklisting Drivers

echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
echo "blacklist amdgpu" >> /etc/modprobe.d/blacklist.conf

Shut Down - Build in GPU - Start

##Step 5: Adding GPU to VFIO

lspci -v
lspci -n -s 03:00

echo "options vfio-pci ids=1002:743f, 1002:ab28  disable_vga=1"> /etc/modprobe.d/vfio.conf

update-initramfs -u
reset
