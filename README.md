# home-nas-revival
This project documents reviving an old Ubuntu based NAS, including regaining admin access, 
troubleshooting display issues, and preparing for backup tasks.


## Table of Contents
- [Background](#background)
- [Password Reset](#password-reset)
- [Display Troubleshooting](#display-troubleshooting)
- [Lessons Learned](#lessons-learned)
- [SSH Setup and Remote Access](#ssh-setup-and-remote-access)
- [Samba Share Access and User Profile Setup](#samba-share-access-and-user-profile-setup)
-  [Firewall Configuration, Automated Backups, and Email Notifications](#firewall-configuration-automated-backups-and-email-notifications)
- [Next Steps](#next-steps)

## Password Reset Process

**Problem:**  
I did not remember the admin password, so I couldn’t log in or make system changes.

**Solution:**  
1. **Boot into Recovery Mode:**
   - Restarted the PC and held `Shift` during boot to access the GRUB menu.
   - Selected "Advanced options for Ubuntu" > a "recovery mode" entry.
2. **Drop to Root Shell:**
   - Chose the "root – Drop to root shell prompt" option.
3. **Remount the Filesystem as Writable:**
```
   mount -o remount,rw/
```
5. **Reset the Password:**
- Listed users with:
  ```
  ls /home
  ```
- Reset the password for my user:
  ```
  passwd <your-username>
  ```
- Entered a new password when prompted.
5. **Rebooted:**
```
reboot
```
6. **Logged in successfully with the new password!**

---

## Display Troubleshooting

**Problem:**  
After logging in, the display was stuck in vertical orientation and labeled "unknown." Rotation was not possible via the GUI or `xrandr`. The only output was "default" and rotation commands failed.

**Troubleshooting Steps:**
- Tried to rotate the display in Ubuntu’s Settings > Displays, but the monitor was labeled "unknown" and changes had no effect.
- Ran `xrandr` in the terminal, but it only showed "default" and gave errors like:
output default cannot use rotation "right" reflection "none"

- Suspected the system was using the Wayland display server, which limits certain features and tools like `xrandr`.

**Solution:**  
1. **Checked for Xorg/Wayland Session:**
 - Noticed there was no gear icon on the login screen to switch to Xorg.
2. **Manually Disabled Wayland:**
 - Edited the GDM3 configuration file:
   ```
   sudo nano /etc/gdm3/custom.conf
   ```
 - Found the line:
   ```
   #WaylandEnable=false
   ```
 - Uncommented it so it read:
   ```
   WaylandEnable=false
   ```
 - Saved and closed the file.

3. **Restarted GDM3 or Rebooted:**
 ```
sudo reboot
```

4. **Result:**
- System booted into Xorg.
- The display was now recognized by name (e.g., HDMI-1, VGA-1).
- Rotation worked in both the GUI and with `xrandr`.

---

## Lessons Learned

- **Physical access is invaluable** for deep troubleshooting, especially when remote access isn’t possible.
- **Ubuntu defaults to Wayland** in recent versions, which can restrict certain display features and tools.
- **Disabling Wayland and switching to Xorg** can resolve compatibility issues with older hardware and software utilities.
- **Documenting each step** (including failed attempts) is helpful for future reference and for sharing solutions with others.

- ---

## SSH Setup and Remote Access

**Goal:**  
Enable secure, remote command line access to the NAS from another computer, eliminating the need to use a local monitor and keyboard.

### Steps and Commands Tried

1. **Checked if SSH Was Installed and Running**
   - On the NAS, checked the SSH service status:
     ```
     sudo systemctl status ssh
     ```
   - If SSH was not running, installed and started it:
     ```
     sudo apt update
     sudo apt install openssh-server
     sudo systemctl enable ssh
     sudo systemctl start ssh
     ```
   - Confirmed SSH was active:
     ```
     sudo systemctl status ssh
     ```

2. **Found the NAS IP Address**
   - Ran the following to get the local IP address:
     ```
     hostname -I
     ```
   - Alternatively:
     ```
     ip a
     ```
   - Noted the IP address (e.g., `192.168.1.100`) for remote access.

3. **Tested SSH Connection from Main PC**
   - On the main PC, opened a terminal and connected:
     ```
     ssh yourusername@192.168.1.100
     ```
   - Accepted the host key and entered the NAS password when prompted.
   - Successfully accessed the NAS remotely via SSH.

4. **(Optional) Allowed SSH Through the Firewall**
   - If UFW firewall was enabled, allowed SSH traffic:
     ```
     sudo ufw allow ssh
     sudo ufw reload
     ```

5. **Created a Dedicated User for SSH Access (if needed)**
   - Checked if the intended user existed:
     ```
     id yourusername
     ```
     (If the user did not exist, created it:)
     ```
     sudo adduser yourusername
     ```
   - Set a strong password for the new user and ensured they could log in via SSH.

---

## Lessons Learned

- **SSH is essential** for headless server management and remote administration.
- **Ensuring the SSH service is enabled and running** is the first step before disconnecting the monitor and keyboard.
- **Knowing the server’s IP address** is critical for remote access—static IPs or DHCP reservations are recommended for reliability.
- **Creating a dedicated user** for SSH access improves security and allows for better access control.
- **Firewall configuration** may be necessary to allow SSH connections, especially if UFW or another firewall is enabled.


---

## Samba Share Access and User Profile Setup

**Goal:**  
Enable file sharing between the Ubuntu NAS and a Windows PC using Samba, and resolve authorization issues by creating a dedicated user profile for Samba access.

### Steps and Commands Tried

1. **Installed and Verified Samba**
   - Updated package lists and installed Samba:
     ```
     sudo apt update
     sudo apt install samba samba-common-bin
     ```
   - Verified installation:
     ```
     samba -V
     sudo systemctl status smbd
     ```

2. **Created a Shared Directory**
   - Made a directory to share and set permissions:
     ```
     sudo mkdir -p /srv/samba/share
     sudo chown nobody:nogroup /srv/samba/share
     sudo chmod 777 /srv/samba/share
     ```
   - (You can use a different path if desired.)

3. **Configured the Samba Share**
   - Edited the Samba configuration file:
     ```
     sudo nano /etc/samba/smb.conf
     ```
   - Added to the end:
     ```
     [NASShare]
     path = /srv/samba/share
     browseable = yes
     read only = no
     guest ok = yes
     ```
   - Restarted Samba to apply changes:
     ```
     sudo systemctl restart smbd nmbd
     ```

4. **Encountered Authorization Issue on Windows**
   - Tried to access the share from Windows File Explorer:
     ```
     \\192.168.1.100\NASShare
     ```
   - Received error:  
     *"You can't access this shared folder because your organization's security policies block unauthenticated guest access."*
   - Realized Windows blocks guest (unauthenticated) access by default for security reasons.

5. **Created a Samba User Profile for Authenticated Access**
   - Checked if the intended user existed:
     ```
     id yourusername
     ```
     (If not, created the user:)
     ```
     sudo adduser yourusername
     ```
   - Set a Samba password for the user:
     ```
     sudo smbpasswd -a yourusername
     ```
   - Updated the share in `/etc/samba/smb.conf` to require authentication:
     ```
     [NASShare]
     path = /srv/samba/share
     browseable = yes
     read only = no
     valid users = yourusername
     guest ok = no
     ```
   - Restarted Samba:
     ```
     sudo systemctl restart smbd nmbd
     ```

6. **Accessed the Share with Credentials**
   - On Windows, accessed:
     ```
     \\192.168.1.100\NASShare
     ```
   - Entered the Samba username and password when prompted.
   - Successfully accessed the NAS share.

7. **Pinned the Share for Quick Access**
   - Right clicked the NAS share in File Explorer.
   - Selected **Pin to Quick access** for easy sidebar access.

8. **(Optional) Mapped as a Network Drive**
   - In File Explorer > This PC, clicked **Map network drive**.
   - Chose a drive letter and entered the share path.
   - Checked **Reconnect at sign in** for persistent mapping.

---

## Lessons Learned

- **Authenticated Samba access is required** for compatibility with modern Windows security policies guest access is often blocked by default.
- **Creating a dedicated Linux user and Samba profile** is necessary for authenticated access.
- **Pinning or mapping the share** streamlines workflow and makes the NAS feel like a local drive.
- **Permissions and access control** are essential for both usability and security.

## Firewall Configuration, Automated Backups, and Email Notifications

### Firewall Setup

To secure the NAS and limit access to only required services, I installed and configured UFW (Uncomplicated Firewall):

```
sudo apt update
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow samba
sudo ufw enable
sudo ufw status verbose
```

- **SSH and Samba** are allowed so I can manage the server remotely and access shared files from my other devices.
- All other incoming connections are denied by default for security.

---

### Automated Backups

To ensure important data is regularly saved, I created a backup script and scheduled it with cron:

**Backup script example (`backup.sh`):**
```
#!/bin/bash

SOURCE="/home/yourusername/Documents/"
DEST="/srv/samba/share/backup/"

mkdir -p "$DEST"
rsync -aAX --delete "$SOURCE" "$DEST"

echo "Backup completed at $(date)" | mail -s "NAS Backup Complete" your@email.com
```
Add a line to run the backup daily at a time when the NAS is idle (example: 10:30 AM):
```
30 10 * * * /home/yourusername/backup.sh

```

- The script uses `rsync` to copy files from the source to the backup directory.
- The `--delete` flag ensures the backup mirror matches the source.
- After completion, the script sends an email notification.

**Scheduling with cron:**
```
crontab -e

```


---

### Automated Email Notifications

- Installed `mailutils` to enable sending emails from the server:
```
sudo apt install mailutils
```


- The backup script sends an email upon completion, so I’m immediately notified of backup status.

---

### Lessons Learned

- Enabling and configuring a firewall is essential for securing any server exposed to a network.
- Automating backups with `rsync` and cron ensures data is protected without manual intervention.
- Email notifications provide peace of mind and immediate feedback if something goes wrong.
- Scheduling backups during idle hours avoids performance issues during active use.

---

## Next Steps

- Plan for offsite or cloud backups for redundancy.
- Continue documenting any new configurations, scripts, or troubleshooting steps as the project evolves.

---

## Resources

- [Ubuntu Community Help Wiki: Resetting Password](https://help.ubuntu.com/community/LostPassword)
- [Ubuntu Documentation: Xorg vs Wayland](https://wiki.ubuntu.com/Wayland)
- [xrandr Manual](https://www.x.org/releases/X11R7.6/doc/man/man1/xrandr.1.xhtml)
- [Ubuntu Documentation: OpenSSH Server](https://help.ubuntu.com/community/SSH/OpenSSH/Configuring)
- [Ubuntu Manpage: systemctl](https://manpages.ubuntu.com/manpages/focal/en/man1/systemctl.1.html)
- [Ubuntu Community Help Wiki: UFW](https://help.ubuntu.com/community/UFW)
- [Ubuntu Documentation: adduser](https://manpages.ubuntu.com/manpages/focal/en/man8/adduser.8.html)
- [Ubuntu Tutorial: Install and Configure Samba](https://ubuntu.com/tutorials/install-and-configure-samba)[1]
- [Ubuntu Documentation: Set up Samba as a file server](https://documentation.ubuntu.com/server/how-to/samba/file-server/)[2]
- [Pimylifeup: Setting up a Simple NAS On Ubuntu using Samba](https://pimylifeup.com/ubuntu-nas/)[5]
- [PhoenixNAP: How to Install Samba on Ubuntu](https://phoenixnap.com/kb/ubuntu-samba)[3]
- [Ubuntu Community Help Wiki: Samba](https://help.ubuntu.com/community/Samba)
- [How to use Markdown for writing documentation - Adobe](https://experienceleague.adobe.com/en/docs/contributor/contributor-guide/writing-essentials/markdown)[3]
- [Getting Started | Markdown Guide](https://www.markdownguide.org/getting-started/)[2]
- [UFW Essentials: Common Firewall Rules and Commands](https://help.ubuntu.com/community/UFW)
- [Automating Backups with rsync and cron](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories-on-a-vps)
- [Mailutils Documentation](https://mailutils.org/docs.html)
  

---

*This project is ongoing. I will update this documentation as I add backup automation, document management, and further improvements to my home NAS setup.*
