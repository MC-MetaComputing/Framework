# How to Install an Image System on SSD (Take Ubuntu as an Example)
## Installation Preparation
- A SSD(M.2 2280 specification)
- An M.2 external SSD enclosure
- A PC/laptop for flashing the SSD
- Flashing software (e.g., balenaEtcher)
- System image: https://drive.google.com/file/d/1aqGTjbmBr8jSLScAU0QAVFaFSNWuI5Zz/view?usp=sharing

## Flash the Image to SSD
1. Download the image:
https://drive.google.com/file/d/1aqGTjbmBr8jSLScAU0QAVFaFSNWuI5Zz/view?usp=sharing

2. Visit etcher.balena.io to download and install balenaEtcher (available for Windows, Linux, or macOS).
<img width="1280" height="660" alt="image" src="https://github.com/user-attachments/assets/f8ca2637-2478-4486-b7f1-a32c50de345d" />

3. Insert the SSD: Insert the SSD into the M.2 external SSD enclosure, then connect the enclosure to an independent PC/laptop via USB.
<img width="1006" height="740" alt="image" src="https://github.com/user-attachments/assets/6c81ad64-bd62-4ea6-8120-e5305bd32e1b" />

4. Launch the flashing program: Run the installed balenaEtcher application, click "Flash from file", then browse and select the downloaded Ubuntu system image file.
<img width="1215" height="742" alt="image" src="https://github.com/user-attachments/assets/df803356-4714-46bb-b8df-b2752c8d918b" />

5. Select the target drive: Click "Select target".Important Note: Be sure to carefully select the drive corresponding to the external SSD enclosure. Selecting the wrong drive will result in permanent data loss on that drive.
<img width="1187" height="752" alt="image" src="https://github.com/user-attachments/assets/612402f6-9513-41d6-8dbc-39dfec2bbeb3" />
<img width="1196" height="756" alt="image" src="https://github.com/user-attachments/assets/b25aea8c-d2fd-4366-85cd-7554db8bec9e" />

6. Start flashing: After confirming the image and target drive, click "Flash!" (or equivalent button) to initiate the writing process.
<img width="1257" height="713" alt="image" src="https://github.com/user-attachments/assets/e906b502-d79b-427a-854b-f8f3b4ab355d" />

7. Wait for flashing completion: Keep waiting until the flashing process finishes completely; the application will display a completion prompt.
<img width="1186" height="742" alt="image" src="https://github.com/user-attachments/assets/a9756360-e4e1-4c2a-bede-ee3ddcd007e7" />


8. Safely remove the enclosure: Safely eject or disconnect the external SSD enclosure from the laptop.

9.  Install the SSD on the target device and boot: Physically reinstall the flashed SSD into the MetaComputing AI PC motherboard, then power on the device to boot up.


