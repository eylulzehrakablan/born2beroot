*This project has been created as part of the 42 curriculum by ekablan.*

# Born2beRoot

## Description
The goal of **Born2beRoot** is to learn the basics of system administration, virtualization, and security. In this project, i created a secure virtual machine that works as a minimal server. 

By doing this project, i gained practical experience in setting up a Linux operating system from scratch. I also learned how to set up strict security rules, manage disk partitions with LVM and LUKS encryption, secure network connections (SSH), and automate system monitoring using Bash scripts and Cron jobs.

---

### Project Description & Technical Choices

#### Operating System Choice: Debian
For this project, a chose **Debian**. 
* **Advantages:** Debian is known for being very stable. it has a large software repository (`apt`) and lots of documentation online. It doesn't use much RAM or CPU, so it is perfect for a minimal server. Also, it uses AppArmor by default, which is easier for beginners to learn than other MAC (Mandatory Access Control) systems.
* **Disadvantages:** The updates are a bit slow compared to newer distributions. This means the packages are not always the newest versions, but this is okay because it makes the server more stable.

#### System Architecture & Design Choices
* **Partitioning:** I partitioned the disk manually using **LVM (Logical Volume Manager)** inside a **LUKS** encrypted container (`sda5_crypt`). This makes sure the data on the disk is encrypted and cannot be read without the password. I created logical volumes for `root`, `swap`, and `home` to separate system files from user files.
* **Security Policies:** 
  * I set strict password rules in `/etc/login.defs` (passwords expire in 30 days, minimum 2 days before changing).
  * I used `libpam-pwquality` for password complexity (minimum 10 characters, specific character types needed, no usernames in passwords).
  * `sudo` is very restricted: it allows only 3 wrong password attempts, requires a TTY terminal, limits where commands can run, and logs everything to `/var/log/sudo/`.
* **User Management:** I created a standard user account (`ekablan`) and added it to the `sudo` and `user42` groups. Direct `root` login over SSH is completely disabled to stop brute-force attacks.
* **Services Installed:** I kept the system very minimal. The main services are `sshd` (it only listens on port `4242`), `ufw` for the firewall, and `cron` for running the monitoring script.

#### Technical Comparisons

##### Debian vs. Rocky Linux
* **Debian:** Uses the `apt` package manager (.deb packages). It is a community project and is very famous for being stable and widely used.
* **Rocky Linux:** Uses the `dnf` package manager (.rpm packages). It is a clone of Red Hat Enterprise Linux (RHEL) and is mostly used in corporate companies.

##### AppArmor vs. SELinux
* **AppArmor:** A Mandatory Access Control (MAC) system that uses file paths for security. It is the default on Debian and is generally easier to learn.
* **SELinux:** Another MAC system created by the NSA. It uses labels instead of paths. It is very powerful but also very hard to learn. It is the default on Rocky Linux.

##### UFW vs. firewalld
* **UFW (Uncomplicated Firewall):** The default on Debian. It is a very simple command-line tool to manage the firewall (for example, `ufw allow 4242`).
* **firewalld:** The default on Rocky Linux. It manages firewall rules using "zones" (like public or home) and you can change rules without dropping active connections.

##### VirtualBox vs. UTM
* **VirtualBox:** A hypervisor made by Oracle for x86/amd64 computers. It is very popular, strong, and has many settings.
* **UTM:** A virtual machine app for macOS. It is necessary if you use Apple Silicon (M1/M2/M3 MacBooks) because it can run ARM-based VMs or emulate x86.

---

## Instructions

### 1. Prerequisites
* **Oracle VirtualBox** installed on your host machine.
* The `.vdi` virtual disk file of this project.

### 2. Setup & Execution
1. Import the virtual machine into VirtualBox.
2. Ensure the VM Network settings are configured to **NAT**.
3. Set up Port Forwarding in VirtualBox:
   * **Host Port:** `4243` (can be any port that is not in use)
   * **Guest Port:** `4242`
4. Start the Virtual Machine.
5. When prompted by the GRUB bootloader, enter the LUKS encryption passphrase to decrypt the disk.
6. The machine will boot into a command-line interface.

### 3. Accessing via SSH
To securely connect to the virtual machine from your host terminal, run:

```bash
ssh -p 4243 ekablan@127.0.0.1
```

4. System Monitoring

A bash script (monitoring.sh) is located in /usr/local/bin/. It automatically shows system information (CPU load, RAM usage, LVM status, active connections) to all logged-in terminals every 10 minutes using a cron job.

## Resources

### Documentation & Tutorials:

- Debian Official Documentation

- LVM Administrator's Guide

- UFW Manual (man ufw)

- Sudoers Manual (man sudoers & man sudo)

- Using PAM (Pluggable Authentication Modules) - Red Hat Documentation

### Use of Artificial Intelligence (AI):

- Leveraged AI as a interactive study guide to help me prepare for this project (to deepen my understanding of topics such as D-Bus, LVM, and bash scripts).
