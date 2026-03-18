## BIOS Update Manual for MetaComputing AI PC with Framework Laptop 13

### Update Preparation
- A portable USB drive (no less than 2GB and formatted in FAT32)
- The latest BIOS file
- The MetaComputing AI PC to be updated

### BIOS Update
1. Download the BIOS file and tool, then save them to the USB drive
- BIOS file：http://120.92.155.32:8082/artifactory/virtOS/cix/20260130/cix_flash_all_fml13v04-q3-1.bin
- Update tool:https://drive.google.com/file/d/1r1IKts7_C1TzNc_t1oIPBnAkyX-DEHH-/view?usp=sharing


2. Insert the USB drive into the MetaComputing AI PC to be updated


3. Press the Esc key while the device is booting to enter the BIOS environment, as shown in the figure below 
<img width="1280" height="1086" alt="image" src="https://github.com/user-attachments/assets/c3ba6afc-992f-4eca-b8bd-44fbc8caf1e0" />


4. Select the Boot Manager option to access the following interface.
<img width="1280" height="1093" alt="image" src="https://github.com/user-attachments/assets/35559585-c698-4708-92a0-08f3dd727aaf" />


5. Select the UEFI Shell option to enter the following interface, then locate the mounted directory of the USB drive.
<img width="1280" height="983" alt="image" src="https://github.com/user-attachments/assets/90811efe-2854-495a-87c7-96cd6a3e9c58" />


6. Select the USB drive's mount point in the Shell environment. As shown in the figure below, enter FS1: in the Shell, then input the command FlashUpdate.efi -f cix_flash_all_fml13u04-g3-1.bin_ to start the BIOS update.
<img width="1280" height="1175" alt="image" src="https://github.com/user-attachments/assets/967cf946-3ddd-49b2-b647-2e5c858e45b1" />

7. Wait for the update to complete, and the system will restart automatically
