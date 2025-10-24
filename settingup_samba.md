# 📁 Sharing a Seagate External Drive from Linux Mint to Windows 11 via Samba Server 

This guide walks through setting up a Samba server on Linux Mint Cinnamon to share a Seagate external drive with a Windows 11 Pro PC over a local Wi-Fi network.

---

## 🖥️ System Setup

- **Linux Host**: Linux Mint Cinnamon
- **Windows Client**: Windows 11 Pro
- **Drive**: Seagate external HDD (auto-mounted at `/media/temuzin/Seagate Hub`)
- **Network**: Both systems connected to the same Wi-Fi and you should be knowing your Linux connected ip address . 

---

## 🔧 Step 1: Install Samba on Linux Mint

```bash
$:sudo apt update
$:sudo apt install samba
```

## 📂 Step 2: Add the Share in the conf file
Edit the Samba configuration file: 
```bash
$: sudo nano /etc/samba/smb.conf
````

Add the following block at the end of the file:
```ini
[SeagateHub]
   path = "/media/temuzinusername_linux/Seagate Hub"
   browseable = yes
   writable = yes
   guest ok = yes
   public = yes
   force user = username_linx
```   


Replace username_linux with your actual Linux username.
Replace Seagate Hub with actual drive name that appears when the drive is connected and mounted  on Linux. 

Save and exit (Ctrl+O, Enter, Ctrl+X).

## 🔄 Step 3: Restart Samba Service , enable auto start at the boot up
```bash
$ sudo systemctl restart smbd nmbd
$ sudo systemctl enable smbd nmbd 
```

# 🔐 Step 4: Optional — Adjust Folder Permissions
Ensure the drive is accessible by the Samba user:
````bash
ls -ld "/media/temuzin/Seagate Hub"
````

If needed, adjust permissions:
```bash
sudo chmod -R 755 "/media/temuzin/Seagate Hub"
```

## 🪟 Step 5: Connect from Windows 11
- Open File Explorer right click on the Network icon, and select Map Network Drive
- In the opened pop up window give  \\ <linux ip>\ Linux ip \\

- To know your linux ip from linux terminal 
```bash 
$ hostname -I
```
This will show 
- Example:
\\192.168.1.100\
With this you should get connected to the IP and below that you should see your drive as a folder. Select this Click "Finish" . 

⚠️ Troubleshooting Tips
- If you see “network name cannot be found”:
- Double-check the share name and path
- Ensure the drive is mounted and accessible
- Use quotes or escape spaces in the path ("Seagate Hub" or Seagate\040Hub)
- If guest access fails:
- Enable insecure guest logons in Windows:
- Run gpedit.msc
- Navigate to:
Computer Configuration > Administrative Templates > Network > Lanman Workstation
- Enable:
Enable insecure guest logons
- If using credentials:
net use Z: \\192.168.1.100\SeagateHub /user:temuzin yourpassword



✅ Result
Your Windows 11 PC can now access the Seagate external drive connected to your Linux Mint system as a network drive.

🙌 Credits
Tested and documented by Sree
Linux Mint Cinnamon + Windows 11 Pro
October 2025

---




