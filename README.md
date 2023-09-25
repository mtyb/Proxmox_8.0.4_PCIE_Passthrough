# Proxmox_8.0.4_PCIE_Passthrough
Guide how to enable PCIE Passthrough for AMD GPU on Proxmox 8.0.4. Tested on AMD RX5700 and Intel i9 10900T.

- Edit grub (nano /etc/default/grub):

  replace ``` GRUB_CMDLINE_LINUX_DEFAULT="quiet" ``` with ```GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on" ```

- Update GRUB:

  ``` update-grub ```

- Edit modules file (nano /etc/modules):

```
  vfio
  vfio_iommu_type1
  vfio_pci
  vfio_virqfd
```

- Run below commands:

```
  echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
  echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf
```

- Backlist GPU drivers:

```
  echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
  echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
  echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
```

- Add GPU to VFIO (to check video id run: ```lspci -v```):

  ``` echo "options vfio-pci ids=VENDOR_ID1,VENDOR_ID2 disable_vga=1"> /etc/modprobe.d/vfio.conf ```

- Update and restart node:

  ```
  update-initramfs -u
  reset
  ```

- Fix AMD vendor reset:

   ```
  apt install pve-headers-$(uname -r)
  apt install git dkms build-essential
  git clone https://github.com/gnif/vendor-reset.git
  cd vendor-reset
  dkms install .
  echo "vendor-reset" >> /etc/modules
  update-initramfs -u
  shutdown -r now
   ```

  create service:

  ```
  cat << EOF >>  /etc/systemd/system/vreset.service
  [Unit]
  Description=AMD GPU reset method to 'device_specific'
  After=multi-user.target
  [Service]
  ExecStart=/usr/bin/bash -c 'echo device_specific > /sys/bus/pci/devices/VENDOR_ID1/reset_method'
  [Install]
  WantedBy=multi-user.target
  EOF
  systemctl enable vreset.service && systemctl start vreset.service
  ```

  ## VM Config:

  ![image](https://github.com/mtyb/Proxmox_8.0.4_PCIE_Passthrough/assets/50377736/32b1294a-583f-4ca8-8117-0a3d77500028)


  ## Sources:
  [Proxmox GPU passthrough | Configuration setup](https://bobcares.com/blog/proxmox-gpu-passthrough/)
  
  [PCI/GPU Passthrough on Proxmox VE 8 : Installation and configuration](https://forum.proxmox.com/threads/pci-gpu-passthrough-on-proxmox-ve-8-installation-and-configuration.130218/)
