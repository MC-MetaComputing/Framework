## Arm Linux Kernel Compilation Guide

### Prerequisites
- Recommended Operating System: Ubuntu 20.04/22.04/24.04 aarch64
- Required additional software packages:
```
sudo apt-get update
sudo apt-get install make build-essential libncurses-dev bison flex libssl-dev libelf-dev dpkg-dev -y
```

### Step 1: Pull Kernel Source Code and Enter Directory
```
git clone https://github.com/MC-MetaComputing/fml13v04-linux.git 
cd fml13v04-linux
```

### Step 2: Configure Kernel
1. View device configuration files in the arch/arm64/configs directory:
```
ls arch/arm64/configs/
# Output:
cix.config*  cix_cloudbook.config  cix_debug.config*  cix_emu.config  cix_fml13v04_defconfig  cix_fpga.config  cix_redroid.config  defconfig  virt.config
```

2. Execute the following command to load the configuration:
```
make cix_fml13v04_defconfig
```

### Step 3: Compile Kernel (Multi-threading Acceleration)
```
make -j$(nproc)
```
- The compilation process may take 10-30 minutes (depending on CPU performance). Upon completion, the kernel image (arch/arm64/boot/Image) and kernel modules (.ko files) will be generated.
- To verify compilation completion, check if the exit status is 0:
```
echo $status
0
```

### Step 4: Install Kernel Modules and Image
1. Install kernel modules to the system directory:
```
sudo make modules_install
```

2. Install kernel image and automatically update boot configuration:
```
sudo make install
```
- This command will automatically copy the kernel image to the /boot directory, generate the initramfs root filesystem image, and update the GRUB boot menu.


### Step 5: Restart and Verify  New Kernel
```
sudo reboot
```

- After the system restarts, select the newly compiled kernel from the GRUB menu to boot. Execute the following command to verify the kernel version:
```
uname -r
```

- The output version number should match the kernel version you compiled, confirming successful compilation and deployment.


### Appendix
- If errors occur during compilation, refer to the following solutions.
- Error keyword hint:
```
drivers/regulator/fwnode_regulator.c:96:20: error: passing argument 1 of ‘IS_ERR’ makes pointer from 
integer without a cast [-Wint-conversion] 96 | if (IS_ERR(n_phandles)) 
```
- Solution:
```
nano drivers/regulator/fwnode_regulator.c 

#Press Ctrl+W to search for n_phandles and locate the erroneous code around line 96:

if (IS_ERR(n_phandles)) 

#Change it directly to the correct way to write it below：

if (n_phandles < 0) 

#Press Ctrl+O and Enter to save, then press Ctrl+X to exit the editor. Re-execute make -j$(nproc) to compile again—this error will be resolved.
```
