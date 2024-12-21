# Ultimate Beginner's Guide to Proxmox GPU Passthrough

Welcome all, to the first installment of my Idiot Friendly tutorial series! I'll be guiding you through the process of configuring GPU Passthrough for your Proxmox Virtual Machine Guests. This guide is aimed at beginners to virtualization, particularly for Proxmox users. It is intended as an overall guide for passing through a GPU (or multiple GPUs) to your Virtual Machine(s). It is not intended as an all-exhaustive how-to guide; however, I will do my best to provide you with all the necessary resources and sources for the passthrough process, from start to finish. If something doesn't work properly, please check /r/Proxmox, /r/Homelab, r/VFIO, or /r/linux4noobs for further assistance from the community.

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
