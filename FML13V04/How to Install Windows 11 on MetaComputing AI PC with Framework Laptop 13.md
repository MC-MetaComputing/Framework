## Preparation
- Download the [Windows 11 for Arm ](https://www.microsoft.com/en-us/software-download/windows11arm64) Image.
- A USB drive of at least 16GB.
- An 8852BE WiFi network card (for internet access after installation).
- A set of external keyboard and mouse (built-in keyboard and touchpad won't work until drivers are installed).

## Create Windows 11 Installation USB
1. Download the disk imaging tool [Rufus](https://github.com/pbatard/rufus/releases/download/v4.14/rufus-4.14.exe).
2. Select the Windows 11 for Arm image to burn.
<img width="686" height="1017" alt="image" src="https://github.com/user-attachments/assets/442bbd07-bf43-430d-b557-de1f5bebccaa" />

3. Start the burning process. When the dialog appears, select "Remove requirement for an online Microsoft account" as shown below.
<img width="833" height="465" alt="image" src="https://github.com/user-attachments/assets/98bd7bba-b407-4cbb-a5fe-51b1c15b5aae" />

4. Wait for the burning process to complete.

## Install Windows 11
1. Insert the Windows 11 installation USB into the MetaComputing AI PC with Framework Laptop 13 device.
2. Configure the device BIOS settings by following the steps below:
- Power on the device and press the ESC key repeatedly to enter the BIOS screen.
<img width="1280" height="960" alt="image" src="https://github.com/user-attachments/assets/b860cf0f-a9aa-47c5-a4c2-db87f0bdda31" />

- Select "MetaComputing System Manager" to enter the following page.
<img width="1280" height="960" alt="image" src="https://github.com/user-attachments/assets/436ef968-cd04-4f2b-8ba9-1256bb366929" />

- Select "SoC Configuration".
<img width="1280" height="960" alt="image" src="https://github.com/user-attachments/assets/855733fd-dceb-429f-9458-36039e8be7b4" />

- Select "CPU Configuration".
<img width="1280" height="960" alt="image" src="https://github.com/user-attachments/assets/3498fa81-07fa-4aad-878d-43357bab445c" />

- Set LPI to LP10 or Disabled, then press F10 to save and exit.
- Return to the "MetaComputing System Manager" page.
<img width="1280" height="1009" alt="image" src="https://github.com/user-attachments/assets/0eee1d62-add3-4f1f-893b-937481df74d2" />

- Enter the "Platform Configuration" page, change the running mode to "Work Mode", then save and exit.
<img width="1280" height="1241" alt="image" src="https://github.com/user-attachments/assets/690ec453-431c-4f0d-b8c5-0480ef4317ff" />


3. Set the boot device to the USB drive in BIOS.
- Select Boot Manager.
<img width="1280" height="960" alt="image" src="https://github.com/user-attachments/assets/7a6bbd41-5d37-4a23-8416-483b4c0cbc12" />

- Select the USB drive, for example: UEFI HIKSEMI, and press Enter to start the Windows installation.
<img width="1280" height="852" alt="image" src="https://github.com/user-attachments/assets/ce16e7f4-2785-4aa7-b504-061064a7ef5e" />

4. Follow the on-screen steps to install Windows and wait for the installation to complete.

## Install Drivers
- After successfully installing Windows 11, you need to install the relevant drivers to use features such as WiFi, Bluetooth, built-in keyboard and touchpad, etc.
- After installing drivers, you need to restart the system for them to take effect.
### Install 8852BE Network Card Driver
- After entering the system, open a terminal as Administrator and run the command bcdedit /set testsigning on to enter test mode, then restart the system.
<img width="1064" height="590" alt="image" src="https://github.com/user-attachments/assets/0799df77-2e10-4533-89ca-11a6574e0b02" />

- Download the WiFi and Bluetooth drivers, then refer to the driver installation manual to install them.
  - WiFi driver: https://drive.google.com/file/d/1NUaoHi2TP4waqmTvkpA6nREsjGTjGvgF/view?usp=drive_link
  - Bluetooth driver: https://drive.google.com/file/d/1BnfDK4iQo1Qqaa0VEIfPq75Of4WgGoCn/view?usp=sharing
- Download and extract the RTWlanPCIE file.Open the folder, right-click the installer and run it as shown below:
<img width="1188" height="714" alt="image" src="https://github.com/user-attachments/assets/7bdbb710-da69-4fff-896d-6c14918de021" />
- After the driver is installed, restart the system for it to take effect.

### Install Framework Keyboard and Touchpad Driver
- Download and install the driver: https://drive.google.com/file/d/17zyqoTgW_vx7Endmdq0BQGpDuMSN4FmP/view?usp=drive_link
- For keyboard and touchpad, you only need to install the GPIO and I2C drivers.
<img width="547" height="396" alt="image" src="https://github.com/user-attachments/assets/ff227ee3-05bb-4074-bc18-36175951fa37" />

- After the driver is installed, restart the system for it to take effect.


### Driver Installation Reference Manual
- https://drive.google.com/file/d/1hP0F-PHFAIORIu_S9N24R4PV_qnRXoKB/view?usp=sharing
