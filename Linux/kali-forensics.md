# Kali Linux and Tools
> A reference guide to Kali Linux command line interface tools for digital forensics.

**Category:** Linux  
**Last Updated:** 2026-09-29    
**Author:** Brady Genik

---

## Tools

| Term | What It Does |
|------|-----------|
| **Chroot** | |
| **Autopsy** | |
| **Holehe** | |  
| **nmap** | |
---

## Overview
The Kali Linux CLI is the primary tool for digital forensics. Mastering these tools is essential for working as a digital forensics investigator.

## Chroot Connection
I have a bootable drive that runs Kali live boot and on a computer if I forget the password to my only account I need to change the password without deleting my data   
You will need to mount the drive from the host machine to the kali machine in the bootable drive using these following steps

### Proxmox Machine
1. Boot Kali Live, open a terminal, become root:

```bash
sudo su
```
2. See the disk layout:
```bash
lsblk
```
3. Look for a volume group named pve. If you see LVM, activate it:

```bash
vgchange -ay
lvscan
```
>The Proxmox root is almost always /dev/pve/root. It may vary depending on the distro or OS

4. Mount it and bind the system dirs (needed for passwd to work in chroot):

```bash
mkdir /mnt/pve
mount /dev/pve/root /mnt/pve
for d in dev proc sys run; do mount --bind /$d /mnt/pve/$d; done
```
5. Chroot in and change the password:

```bash
chroot /mnt/pve
```
6. Now that you can change any account using the following commands below
```bash
# Create 
useradd -m -s /bin/bash username                # -m makes a home dir, -s sets the login shell
passwd username                                 # REQUIRED after useradd — account is locked until a password is set

# Passwords
passwd root                                     # change the root account's password
passwd username                                 # change a specific user's password
passwd -d username                              # remove the password entirely (empty password — use with care)
passwd -u username                              # unlock a password-locked account
chage -l username                               # view password expiry/aging info
chage -E -1 username                            # remove an account-expiration date (revives an "expired" account)

# Modify 
usermod -l newname oldname                      # change login name
usermod -d /home/newname -m username            # move/change home directory
usermod -s /bin/bash username                   # change shell
usermod -L username                             # lock the account (disables login)
usermod -U username                             # unlock it

# Groups 
usermod -aG sudo username                       # add to sudo group (grants admin rights)
usermod -aG groupname username                  # add to any group
gpasswd -d username groupname                   # remove user from a group
groupadd groupname                              # create a group
groupdel groupname                              # delete a group
groups username                                 # list a user's groups
# NOTE: always use -aG (append the group). 'usermod -G sudo username' REPLACES all
# the user's groups with only sudo, kicking them out of every other group.

# Inspect 
cat /etc/passwd                                 # every account (name:x:UID:GID:...:home:shell)
cat /etc/shadow                                 # password hashes and expiry (root-only)
cat /etc/group                                  # groups and their members
id username                                     # a specific user's UID, GID, and groups

# Delete
userdel username                                # remove the user, leave their home dir
userdel -r username                             # remove the user AND home dir + mail spool
```
make sure you edit the chroot mode by using the exit command
```bash
exit
```

7. Clean up and reboot:

```bash
for d in run sys proc dev; do umount /mnt/pve/$d; done
umount /mnt/pve
reboot
```
>Pull the Kali drive during reboot.
---
## Autopsy
---
## Holehe
---
## nmap
---