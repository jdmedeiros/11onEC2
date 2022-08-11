Go here https://www.microsoft.com/software-download/windows11 and in the section Download Windows 11 Disk Image (ISO) select the Windows 11 multi edition ISO 64-bit edition. You should end up with the Win11_English_x64v1.iso file.


Create a new virtual machine with VMware Workstation/Fusion or Virtualbox; I am using Fusion. For boot firmware make sure you specify **UEFI**. Adjust memory and disk size to your preferences; I would recommend 4/64. For disk type specify SATA (I have not tested NVMe, SCSI or IDE). Also, I keep it to one file.


Start the virtual machine; make sure you select Windows 11 PRO. If you get the **"This PC can't run Windows 11"** message, click on the top right red X to quit. Select the **"Repair your computer"** option. Click on **Troubleshoot**, **Command Prompt** and run **regedit**. 


While in  **regedit** select _HKEY_LOCAL_MACHINE_, _SYSTEM_, _Setup_ and create a new key with the name **LabConfig** [no you may not use another name for the key]. Inside the **LabConfig** create 3 new DWORD (32 bit) value fields with the following names: 
- **BypassTPMCheck**
- **BypassRAMCheck**
- **BypassSecureBootCheck**

making sure the value for each of those fields is 1. Some of those values may be optional and depending on your hardware you may need need some additional _Bypass_ parameter. Exit **regedit** and in the command prompt run **setup** to continue.

When the system boots, proceed normally to complete the installation choosing "Set up for personal use" and "offline account". The other options you may choose as you like.

Update the system. When done, shut it down and export the VM to a single **.ova** or **.vhd** file.
