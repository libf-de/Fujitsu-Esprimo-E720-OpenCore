# Fujitsu Esprimo E720 - OpenCore Configuation

[![macOS](https://img.shields.io/badge/macOS-Monterey-brightgreen.svg)](https://developer.apple.com/documentation/macos-release-notes)
[![macOS](https://img.shields.io/badge/macOS-Ventura-brightgreen.svg)](https://developer.apple.com/documentation/macos-release-notes)
[![OpenCore](https://img.shields.io/badge/OpenCore-0.9.6-blue)](https://github.com/acidanthera/OpenCorePkg)
[![License](https://img.shields.io/badge/license-MIT-purple)](/LICENSE)

<p align="center">
   <strong>Status: Maintained</strong>
   <br />
   <strong>Version: </strong>1.0
  </p>
</p>
</br>

## ⚠️ Disclaimer
This guide is only for the Fujitsu Esprimo E720 - although it might work on similar machines. I am NOT responsible for any harm you cause to your device. This guide is provided "as-is" and all steps taken are done at your own risk.

> This README was stolen from from [valnoxy](https://github.com/valnoxy/t480-oc/).

&nbsp;

## Introduction
<details>
<summary><strong>🖥️ My Hardware</strong></summary>
<br>
These are the Hardware component I use. But this OpenCore configuation <strong>should still work</strong> with your device, even if the components are not equal.

> **Note** Check if you have similar hardware (WiFi, dGPU) and remove non-existing hardware from config.plist.

| Category  | Component                              |
| --------- | -------------------------------------- |
| CPU       | Intel Core i7-4790                     |
| iGPU      | Intel HD Graphics 4600                 |
| dGPU      | Nvidia GeForce 1650 (WILL BE DISABLED!)|
| SSD       | SATA Crucial BX500 480GB               |
| Memory    | 24GB DDR3 1600Mhz                      |
| WiFi & BT | Intel Wireless AC 7265                 |

My Crucial P3 NVMe SSD where Linux is installed is detected, so may also boot from NVMe, but I've not tested that.

</details>  

&nbsp;

## Installation

<details>  
<summary><strong>📝 Requirements</strong></summary>
</br>

You must have the following items:
- Esprimo E720
- Monitor **NOT** connected via VGA to iGPU (this causes macOS installer to refuse to start for me!)
- Access to a working Windows, Linux or macOS machine with Python installed.
- A pendrive with more than 4 GB (Remember that during the preparation we will format the flash drive to create the installation media).
- an Internet connection (only tested via Ethernet).
- 1-2 hours of your time.

If you have an unsupported dGPU, you probably also want a monitor where you can switch inputs easily OR an external input switcher.
If you don't set your primary GPU to IGD, the BIOS will show via the dGPU and macOS will then switch to the iGPU output.

</details>

<details>  
<summary><strong>⚙️ Preperation</strong></summary>
</br>

### Create the install media

You can basically follow the official guide here: [Dortania OpenCore Guide - Creating the USB](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/)

### Configure and install OpenCore
Download the EFI folder from this repo, you will find the latest files under the release tab or just download the repo as it is. Move the folder to the root of your pendrive (e.g. J:\; /media/username/OPENCORE/), make sure it's called ```EFI```.

### Copy OCLP
I'd suggest putting [OpenCore Legacy Patcher](https://dortania.github.io/OpenCore-Legacy-Patcher/) on your USB drive, as for me it's almost impossible to use Safari without GPU acceleration, as most pages just display as white. Just [download](https://github.com/dortania/Opencore-Legacy-Patcher/releases) the latest `OpenCore-Patcher-GUI.app.zip` and unpack it to your USB drive.

##### ACPI patches
Please enable / disable the following patches depending on what is installed in your device.

| SSDT                 | Affected device            | Description                                                |
| -------------------- | -------------------------- | ---------------------------------------------------------- |
| SSDT-GPU-DISABLE.aml | Nvidia GeForce 1650        | Disable NVIDIA GPU (necessary if installed)                |

You might need to adjust the ACPI path of your GPU, for me it's `\_SB.PCI0.PEG0.PEGP`. Please check [HERE](https://dortania.github.io/Getting-Started-With-ACPI/Desktops/desktop-disable.html) how to do that.

#### GenSMBIOS
We need a script, called [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS), to create fake serial number, UUID and MLB numbers. **This step is essential to have working iMessage, so do not skip it!**

The process is the following:

- Download GenSMBIOS as a ZIP, then extract it.
- Start GenSMBIOS.bat and use option ```1``` to download MacSerial.
- Choose option ```2```, to select the path of the config.plist file. It will be located in ```EFI -> OC``` folder.
- Choose option ```3```, and enter ```iMac18,1``` as the machine type.
- Press ```Q``` to quit. Your config now should contain the requied serials.

#### Enter the proper ROM value
After adding serials to your config.plist, you have to add the computer's MAC address to the config.plist file. **This step is also essential to have a working iMessage, so do not skip it.** We need a Plist editior, to write the MAC address into the config.plist file. I used [ProperTree](https://github.com/corpnewt/ProperTree), since it works on Windows too. You have to change the MAC address value in the config.plist at

```PlatformInfo -> Generic -> ROM```

Delete the generic ```112233445566``` value, and enter your MAC address into the field, without any colons. Save the Plist file, and it is now ready to be written out to the EFI partition of your install media.

#### Default keyboard layout and language
The default keyboard layout and language is ```German```. To change the language, edit the value of ```NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> prev-lang:kbd``` to the value of your language. If your value contains an underscore "```_```", replace it with a hyphen "```-```". The value for English would be ```en-US:0```. You can find a list of all language values [here](https://github.com/acidanthera/OpenCorePkg/blob/master/Utilities/AppleKeyboardLayouts/AppleKeyboardLayouts.txt).

### Prepare BIOS
The bios must be properly configured prior to installing macOS.

Most default settings are fine, so if you have problems, try resetting the defaults and change settings below:

In Advanced menu, set the following options:
- `Graphics Configuration -> Internal Graphics`: must be **Enabled** if you have a dGPU
- `Graphics Configuration -> IGD Memory`: must be **64M**
(you can set `Graphics Configuration -> Primary Display` to **IGD** if you want)
- `Super IO Configuration -> Parallel Port Configuration`: must be **Disabled**

In Security menu, set the following options:
- `Secure Boot -> Secure Boot Control`: must be **Disabled**

In Boot menu, set the following options:
- `CSM Configuration -> Launch CSM`: must be **Disabled**
If you get a message that CSM can't be disabled as something is in Legacy Mode, set all OpRoms to UEFI, reboot to BIOS and try again.

The following settings are **FINE**, you **DON'T** have to disable them:
- `Advanced -> CPU Configuration -> VT-d`: *Enabled*

Now you can go through the install.ff

</details>

<details>  
<summary><strong>🚚 Installation</strong></summary>
</br>

### Install macOS
1. Boot from USB and select the USB drive inside of OpenCore ```"OPENCORE (DMG)" or similar```.
>  **Note:** It's normal that there is *a lot* of messages and that it seems to hang just before you get to the GUI - just wait.
2. Wait for the macOS Utilities screen.
3. Select Disk Utility, select your disk and click erase. Give a name and choose **APFS** with **GUID Partition Map**.
4. After erasing, go back and select **Reinstall macOS** and follow the steps on your screen. The installation make take up to **2 hours**.
>  **Note:** Your PC will restart multiple times. Just boot from USB and select your disk inside of OpenCore. (named macOS Installer or the disk name).
5. Once you see the `Region selection` screen, you are good to proceed.
6. Create your user accound and everything else.

</details>

<details>  
<summary><strong>♻️ Upgrade macOS / Switch EFI</strong></summary>
</br>

Upgrading macOS has not been tested yet.

</details>

&nbsp;

## Post-install (optional)
<details>  
<summary><strong>🩹 Use OpenCore Legacy Patcher</strong></summary>
</br>

1. [Download OCLP](https://github.com/dortania/Opencore-Legacy-Patcher/releases) if you haven't already
2. Open OCLP and apply root patches. If it works, continue with installing OpenCore to Hard drive. **If OCLP complains about SIP not being disabled**, do the following steps:
3. Reboot, and select "Recovery" in OpenCore.
4. Open the terminal and run the following commands:
    - `csrutil disable`
    - `csrutil authenticated-root disable`
5. Reboot to macOS and try again.

[source of steps 3-5](https://www.reddit.com/r/hackintosh/comments/z9flzx/cant_disable_sip_on_ventura/)
</details>


<details>  
<summary><strong>💾 Install OpenCore to Hard drive</strong></summary>
</br>

1. Press `ALT + SPACE` and open terminal. Type `sudo diskutil mountDisk disk0s1` (where disk0s1 corresponds to the EFI partition of the main disk)
2. Open Finder and copy the EFI folder of your USB device to the main disk's EFI partition.
3. Unplug the USB device and reboot your laptop. Now you can boot macOS without your USB device.

</details>

<details>  
<summary><strong>✏️ Create a offline install media (Optional)</strong></summary>
</br>

In case of reinstalling macOS, a offline install media can save some time. You also don't need an Ethernet connection for the installation.
To create a offline install media, you need the following stuff: 

- macOS Installer from the App Store.
- A 16 GB pendrive (Keep in mind, during the preperation we will format the disk to create the install media).

Press `ALT + SPACE` and open Disk utility. Select your USB device and click erase. Name it `MyUSB` and choose **Mac OS Extended** with **GUID Partition Map**. After erasing the USB device, close Disk utility.

Now press `ALT + SPACE` and open terminal. Type the following command:

Big Sur:
```sudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/MyUSB --downloadassets```

Monterey:
```sudo /Applications/Install\ macOS\ Monterey.app/Contents/Resources/createinstallmedia --volume /Volumes/MyUSB --downloadassets```

Ventura:
```sudo /Applications/Install\ macOS\ Ventura.app/Contents/Resources/createinstallmedia --volume /Volumes/MyUSB --downloadassets```

After creating the install media, copy your EFI folder to the EFI partition of your USB device.


</details>

&nbsp;

## Status

<details>  
<summary><strong>✅ What's working</strong></summary>
</br>
 
- [X] Intel WiFi and LAN works, Bluetooth seems to have problems
- [X] Audio (Audio Jack & Speaker)
- [X] USB Ports
- [X] Graphics Acceleration
</details>

<details>  
<summary><strong>⚠️ What's not working</strong></summary>
</br>

- [ ] Power management / Sleep
- [ ] SIP (must be disabled for OCLP)

</details>

<details>  
<summary><strong>🔄 Not tested</strong></summary>
</br>

- [ ] FaceTime / iMessage (iServices)
- [ ] Automatic OS updates
- [ ] Handoff / Universal Clipboard
- [ ] Sidecar (Cable) / AirPlay to Mac
- [ ] Other PCIe cards, supported dGPUs, ...

</details>

&nbsp;

## ⭐️ Feedback
Did you find any bugs or just have some questions? Feel free to provide your feedback using issues.

&nbsp;

## 📜 License

OpenCore is licensed under the [BSD 3-Clause License](https://github.com/acidanthera/OpenCorePkg/blob/master/LICENSE.txt).

<hr>
<h6 align="center">© 2023 libf.de</h6>