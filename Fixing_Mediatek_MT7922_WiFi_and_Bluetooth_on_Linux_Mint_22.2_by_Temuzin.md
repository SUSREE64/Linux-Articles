# Fixing Mediatek MT7922 WiFi and Bluetooth on Linux Mint 22.2 (MSI B550M PRO-VDH WiFi)

**By Temuzin**

---

## Overview

After installing **Linux Mint 22.2 (Zara)** on a new desktop with an **MSI B550M PRO-VDH WiFi** motherboard and **AMD Ryzen 5 5600GT**, I encountered a familiar problem for Linux users — **WiFi and Bluetooth were not working**.  

The system detected the Mediatek MT7922 chip, but no driver was loaded. This post explains how I identified the issue and fixed it by installing the correct firmware.

---

## System Setup

- **OS:** Linux Mint 22.2 Zara (Cinnamon Edition)  
- **Base:** Ubuntu 24.04 LTS (Noble)  
- **Kernel:** 6.14.0-33-generic  
- **Motherboard:** MSI B550M PRO-VDH WiFi  
- **Wireless Chipset:** Mediatek MT7922 802.11ax PCIe  
- **CPU:** AMD Ryzen 5 5600GT  

---

## Step 1. Confirm Hardware Detection

```bash
lspci | grep -i mediatek
```

Expected output:
```
29:00.0 Network controller: MEDIATEK Corp. MT7922 802.11ax PCI Express Wireless Network Adapter
```

If the adapter appears here, the hardware is recognized but likely missing a functioning driver.

---

## Step 2. Check Driver and Module Status

```bash
sudo lspci -vnnk | grep -A3 7922
```

You may see:
```
Kernel driver in use: N/A
Kernel modules: mt7921e
```

If no driver is loaded, try to manually probe the module:

```bash
sudo modprobe mt7921e
dmesg | grep mt79
```

A typical error looks like:
```
mt7921e 0000:29:00.0: driver own failed
mt7921e 0000:29:00.0: probe with driver mt7921e failed with error -5
```

This indicates a firmware loading problem.

---

## Step 3. Inspect the Firmware Folder

```bash
ls /lib/firmware/mediatek/ | grep MT7922
```

In Linux Mint 22.2 (and Ubuntu 24.04), you’ll likely see **compressed `.zst` firmware files**:
```
BT_RAM_CODE_MT7922_1_1_hdr.bin.zst
WIFI_MT7922_patch_mcu_1_1_hdr.bin.zst
WIFI_RAM_CODE_MT7922_1.bin.zst
```

These compressed files are part of the standard `linux-firmware` package, but some kernels fail to load them correctly.

---

## Step 4. Replace the Compressed Firmware Files

To fix the issue, download the **uncompressed `.bin` firmware** from the official Linux kernel firmware repository and replace the compressed versions.

### 1. Download the uncompressed `.bin` files

Go to:  
[Official Linux Firmware Repository – Mediatek Folder](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/mediatek)

Search for and download these files:
```
BT_RAM_CODE_MT7922_1_1_hdr.bin
WIFI_RAM_CODE_MT7922_1.bin
WIFI_MT7922_patch_mcu_1_1_hdr.bin
```

### 2. Copy the files to your firmware directory

```bash
sudo cp BT_RAM_CODE_MT7922_1_1_hdr.bin WIFI_RAM_CODE_MT7922_1.bin WIFI_MT7922_patch_mcu_1_1_hdr.bin /lib/firmware/mediatek/
```

### 3. Verify file permissions

```bash
sudo chmod 644 /lib/firmware/mediatek/BT_RAM_CODE_MT7922_1_1_hdr.bin
sudo chmod 644 /lib/firmware/mediatek/WIFI_*.bin
```

### 4. Reboot your system

```bash
sudo reboot
```

---

## Step 5. Verify the Fix

```bash
nmcli device
```

Expected output should show both WiFi (`wlp...`) and Ethernet adapters:
```
DEVICE      TYPE      STATE         CONNECTION
wlp41s0     wifi      connected     <Your Network Name>
enp42s0     ethernet  connected     Wired connection 1
```

Then confirm Bluetooth:
```bash
bluetoothctl list
```

You should now see your Bluetooth adapter listed and operational.

---

## Step 6. (Optional) Keep Firmware Updated

```bash
sudo apt update
sudo apt install --reinstall linux-firmware
```

---

## Credits

A big thanks to **morrownr**, whose GitHub documentation helped identify the firmware issue:  
[How to Install Firmware for Mediatek-based WiFi Adapters](https://github.com/morrownr/USB-WiFi/blob/main/home/How_to_Install_Firmware_for_Mediatek_based_USB_WiFi_adapters.md)

---

## Conclusion

The problem stemmed from **compressed firmware files (.zst)** that the kernel failed to unpack at runtime.  
Replacing them with the uncompressed `.bin` versions allowed the **mt7921e driver** to load correctly — enabling both WiFi and Bluetooth on the **Mediatek MT7922** chipset.

This simple fix highlights a broader issue: Linux hardware vendors often ship incomplete support for embedded components.  
Fortunately, the open-source community continues to provide effective workarounds.

If you’re setting up a new Linux desktop with Mediatek WiFi, check your firmware directory first — the solution might already be there, just in the wrong format.

---

✍️ **Authored by Temuzin**  
Credits : https://github.com/morrownr

