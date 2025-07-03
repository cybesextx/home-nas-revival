# home-nas-revival
This project documents reviving an old Ubuntu-based NAS, including regaining admin access, 
troubleshooting display issues, and preparing for backup tasks.


## Table of Contents
- [Background](#background)
- [Password Reset](#password-reset)
- [Display Troubleshooting](#display-troubleshooting)
- [Lessons Learned](#lessons-learned)
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
mount -o remount,rw/
4. **Reset the Password:**
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
reboot

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
 sudo reboot

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

## Next Steps

- Set up backup scripts to regularly save important documents and data.
- Secure the NAS with strong passwords and regular software updates.
- Plan for offsite or cloud backups for redundancy.
- Continue documenting any new configurations, scripts, or troubleshooting steps as the project evolves.

---

## Resources

- [Ubuntu Community Help Wiki: Resetting Password](https://help.ubuntu.com/community/LostPassword)
- [Ubuntu Documentation: Xorg vs Wayland](https://wiki.ubuntu.com/Wayland)
- [xrandr Manual](https://www.x.org/releases/X11R7.6/doc/man/man1/xrandr.1.xhtml)

---

*This project is ongoing. I will update this documentation as I add backup automation, document management, and further improvements to my home NAS setup.*
