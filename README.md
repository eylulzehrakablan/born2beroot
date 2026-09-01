# Downloading VirtualBox
first of all i started by downloading **Oracle VirtualBox** (i prefered downloading via a package manager)

installing VirtualBox via the package manager provides several advantages over downloading a .deb package manually through a web browser. the package manager automatically select the exact build for our specific (Linux Mint) distribution and kernel version. and package manager itself automatically identifies, downloads and configures all required background libraries withut requiring manual tracking.

i already had VirtualBox on my system and i needed to remove it so that the new and old VirtualBox packages don't conflict:
1. `sudo apt remove --purge virtualbox virtualbox-qt virtualbox-dkms`
**--purge:** completely wipes the software along with all associated system-wide configuration files
**virtualbox virtualbox-qt virtualbox-dkms:** the specific package names causing conflicts (the main virtualization engine, the graphical user interface, and the dynamic kernel module support package)

2. `sudo apt autoremove -y && sudo apt clean`
**autoremove:** scans the system for leftover dependencies
**-y:** automatically answers "yes" to all system confirmation prompts.
**clean:** deletes downloaded .deb installer files from the local package cache to free up disk space

to fetch the lastest list of available software packages and their versions from the official remote linux mint and ubuntu repositories:
3. `sudo apt update`

4. `sudo apt install virtualbox virtualbox-ext-pack -y`
**virtualbox-ext-pack:** the official Oracle extension pack
**virtualbox:** the core hypervisor binary and user interface.

---

# Downloading Debian ISO file
continued by downloading the latest stable version of Debian.

search engine -> Debian -> other downloads -> complete installation image (under the 'Download an installation image' header) -> Download USB/CD/DVD images using HTTP -> Official USB/CD/DVD images of the "stable" release -> architecture selection (amd64 for me) -> download debian-13.6.0-amd64-netinst.iso

## How to find your system architecture?
you can verify your host machine's architecture using the command lines:

`uname -m` (for linux)

Output: x86_64

**-Breakdown of the output-**

x86 : derived from intel's early CPUs ending in '86' (8086, 386, 486). identifies the fundamental x86 processor instruction family.
64 : represents 64-bit memory addressing and register size (max amount of data). 
    --> register size determines the chunk size of data the cpu can handle at once.

---

# Creating the virtual machine

<img src="images/VirtualBoxMenu.png" alt="VirtualBox menu screenshot" width="500">

Oracle VM VirtualBox Manager -> New

<img src="images/vm_name_os.png" alt="Virtual Machine name and os" width="500">

1. Name the virtual machine to your liking. i prefered born2beroot.
2. Folder specifies the destination directory where VirtualBox saves the virtual machine’s configuration files and virtual hard disk drives. Any local directory with at least 10-15 gb of free space is ok. (goinfre)
3. ISO Image part specifies the downloaded Debian network installer ISO
(after choosing the ISO, Oracle VM VirtualBox Manager detected my computer's OS type as Debian 64-bit by itself. so didn't have to select for edition, type and version. You might need to select manually.)
4. select 'Skip Unattended Installation'
    The subject strictly requires a manual installation process to set up at least 2 encrypted partitions using LVM. We are supposed to manage the disk manually. If we don't skip Unattended Installation, VirtualBox quickly perform a basic Debian installation in the background without letting you access the LVM encryption and user creation screens. This situtation leads to -42.

<img src="images/hardware_config.png" alt="Hardware configuration page" width="500">

in the next page (hardware), in order to maintain a minimal server (as required by the project), these are the recommended values:

RAM: 1 GB (enough) or 2 GB (for a better performance -> to speed up package installation and overall responsiveness)
Processors: 1 or 2 CPU

since born2beroot is a minimal server environment without a GUI and we are supposed to use lightweight background services like OpenSSH, Cron/shell scripts, IPTable etc (a few mb of ram total), 1 gb ram and 1 processor is more than enough. Additionally, the base linux kernel uses roughly 100-150 mb of ram at idle. Leaving 850+ mb free for general operations (e.g. active ssh sessions, shell instnces, package operations, running monitoring.sh)

do not select 'Enable EFI (special OSes Only)'

**EFI (Extensible Firmware Interface) or UEFI:** is the modern system firmware interface designed to replace the legacy PC BIOS. Should be enabled when setting up virtual disks larger than 2 TB. Minimal linux servers don't require enabling EFI.

on the next page, we configure the virtual hardk disk. according to the lsblk output in the project subject, an 8 gb disk is sufficient for the required partitions. however, to avoid running out of space during package installation and logging, setting it to 10-15 gb seems safer.

leaving 'Pre-allocate Full Size' disabled so that the host won't take up the entire specified size right away. Now the host storage drive can start small and grows as we use it.

<img src="images/vir_hard_disk.png" alt="Virtual Hard Disk" width="500">

on the next (Summary) page, select 'Finish'

# Setting up the virtual machine

> **REMEMBER :** you can pause the VirtualBox where you are and resume later without losing any installer progress.
1. Close the VirtualBox window by clicking the X in the top-right corner.
2. Select "Save the machine state" and click OK.
3. VirtualBox will freeze the RAM and current installer state onto your computer's disk. When you re-open VirtualBox and click Start, it will resume at this exact screen.

and then we can Start our virtual machine by selecting freshly created vm and Start

<img src="images/vm_start.png" alt="vm start" width="500">

after launching, we are greeted by the installer menu.

the project subject strictly forbids the GUI usage, so continue with 'Install'

<img src="images/installer_menu.png" alt="installer_menu" width="500">

select the language, country and keymap

<img src="images/lang.png" alt="lang" width="500">

<img src="images/country.png" alt="country" width="500">

<img src="images/keymap.png" alt="keymap" width="500">

after a moment, we are represented with a screen where we need to enter a hostname.
the project subject tells 'the hostname of your virtual machine must be your login ending with 42'

<img src="images/host_name.png" alt="host name" width="500">

i didn't see any requirement for domain name on the project subject pdf. since our debian vm is a standalone local server, an external domain name is kinda unnecessary and the project subject only requires a specific hostname. so i left domain name empty.

<img src="images/domain_name.png" alt="domain name" width="500">

in the next page, the installer wants us to set a root password. i was told not to forget this password since we will be using this password later on. in case, note these kind of stuff all the time.

<img src="images/root_password.png" alt="root_password" width="500">

<img src="images/root_password_verify.png" alt="root_password_verify" width="500">

on the next pages, we are setting up users and passwords.

according to the project subject, user with our login as the username has to be present (we will make this user a member of user42 and sudo groups)

for the new user, i preferred leaving the 'Full name for the new user' as it is. not as necessary as the username.

<img src="images/full_name_user.png" alt="full_name_user" width="500">

<img src="images/username.png" alt="username" width="500">

set a password for this user too. of course, do not forget this password either.

<img src="images/user_password.png" alt="user_password" width="500">

<img src="images/user_password_verify.png" alt="user_password_verify" width="500">

selecting the time zone doesn't seem that important to me rn, i chose Central for now. can change the zone later if necassary.

<img src="images/clock.png" alt="time_zone" width="500">

---

# Manuel Disk Partitioning

since we were told (in the project subject) that we need to determine the appropriate size for each partition to ensure proper operation while avoiding unnecessary disk usage, we will use the manual partitioning method:

<img src="images/partitioning_method.png" alt="partitioning_method" width="500">

and then we are greeted by a disk management dashboard. we will choose which hard drive we want to modify. we are supposed to choose the middle one here since we are doing a %100 manual disk setup. 

by selecting 'SCSI3 (0,0,0) (sda) - 16.1 GB ATA VBOX HARDDISK', we are turning of the automatic installer to build the partition table, the /boot partition, the encryption layer and the LVM volymes ourselves.

'SCSI3 (0,0,0) (sda) - 16.1 GB ATA VBOX HARDDISK' represents our raw, empty VirtualBox hard drive. it currently has no partitions, no file systems and no data on it.

<img src="images/partition_options.png" alt="partition_options" width="500">

**-breakdown-** 

**SCSI3 (Small Computer System Interface Generation 3):** the virtual controller interface protocol VirtualBox uses to communicate with the drive

**(0,0,0):** the hardware address numbers (adapter, bus, target/LUN ID) assigned to this disk on the virtual controller
    - Adapter: the virtual storage controller card plugged into your virtual motherboard??
    we only have one primary storage controller configured in our VM settings, so it gets index 0. if we added a secondary PCI storage controller card, its drives would start with Adapter 1
    - Bus: the internal data pathway (or channel) attached to that specific controller adapter.
    - Target / LUN ID: the specific device port number on the bus
        - LUN (Logical Unit Number): a sub-address used when a single physical storage target hosts multiple logical drives.
    virtualBox assigns your virtual disk (.vdi) to the very first storage port (Port 0) on the virtual controller. that's why the third value is 0

**(sda):** the linux device name.
    - sd: SCSI disk (standart linux prefix for SCSI drives)
    - a: the first storage drive detected by the system. (a second drive would be sdb)

**ATA (Advanced Technology Attachment):** the virtual storage bus type emulation.

**VBOX HARDDISK:** The hardware device model name generated by VirtualBox.

**Other selections:**
- Guided partitioning: a shortcut back to the automated wizard screen you just left.
- Configure iSCSI volumes: an advanced tool to connect storage over a network interface. we are configuring a local virtual hard disk (.vdi) inside VirtualBox. not remote network storage. so skip this one too.
- Undo changes to partitions: a reset button that discards any changes, partition creations, or formatting choices made during the current installation session.
- Finish partitioning and write changes to disk: the final confirmation button used to save your completed partition scheme. selecting this now will cause an error because no partitions or file systems have been created yet.

on the next page the installer asks to create new empty partition table. the whole time, the purpose was to create an empty table. choose yes.

<img src="images/partition_table.png" alt="partition_table" width="500">

selecting pri/log (FREE SPACE) here launches the partition creation wizard.

<img src="images/partition_selection.png" alt="partition_selection" width="500">

select 'Create a new partition'

<img src="images/create_new_partition.png" alt="create_new_partition" width="500">

let's take a look at the given partition table example and fill out the new partition sizes according to the project subject:

> lsblk (list block devices) command displays detailed information about all available block devices such as hard drives, SSDs, USB drives and their respective partitions.

<img src="images/example_lsblk.png" alt="example lsblk output" width="500">

sda1 (/boot):

<img src="images/partition_1.0.png" alt="partition 1.0" width="500">

### Primary and Logical types for partitioning

<img src="images/partition_1_type.png" alt="partition 1 type" width="500">

when using partition table on a hard drive, linux divides the disk using these two types:

- **Primary:** is the main partition on the disk. 
> an MBR (msdos) disk can have a max of 4 primary partitions.
> primary type is used for critical system startup files (like /boot). the BIOS and bootloader (GRUB) can read Primary partitions directly to start the operating system.

<a id="logical"></a>

- **Logical:** A sub-partition created inside a special Primary partition called an Extended Partition.
> it bypasses the 4-partition limit, allowing you to create extra partitions.
> used for extra data storage or additional system drives when you run out of primary slots.

Setting /boot as a Primary partition at the Beginning of the disk ensures the system can locate and read the boot files cleanly before any complex drivers are loaded.

<img src="images/partition_1_location.png" alt="partition 1 location" width="500">

choosing 'Beginning' places /boot  at the front of the drive so that GRUB (bootloader) can immediately find it.

'End' is typically used for non-urgent partitions.

following screen shows us the details of the partition. We will modify the mount point according to the project subject.

> What is 'mounting'
    The process of attaching a disk partition (or USB drive) to the operating system so Linux can read and write data to it

> What is 'mount point'
    the specific folder used as the entry door to access that mounted disk space.

-Simplified Analogy-
- Disk / Partition: a room full of storage boxes
- Mounting: unlocking the door and connecting that room to your hallway
- Mount Point: the label on the door (such as /boot or /home) that you open to step inside that room

select 'Mount point'

<img src="images/partition_1_settings.png" alt="partition 1 settings" width="500">

set the mount point to /boot

<img src="images/p1_mount_point.png" alt="partition 1 mountpoint" width="500">

we are not going to modify any other setting:

- _**Use as:** Ext4 journaling file system :_ defines how data is formatted and organized on the partition, Ext4 is the standart for linux file system, GRUB and linux kernel fully support reading Ext4 without needing any extra drivers, so leave it as it is.

- _**Mount options:** defaults :_ sets system permissions and behaviors (read-write access, executing binaries) when mounting the partition. _defaults_ includes all basic r/w flags required.

- _**Label:** none :_ an optional text name assigned to the drive volume for identification. Linux uses device names (/dev/sda1) internally, custom labels are for nothing but styling it. unnecessary here.

- _**Reserved blocks:** 5% :_ keeps 5% (25 mb for this specific partition) of the partition's space locked for the root user to prevent complete system lockup if a disk fills up to 100%. it is a built-in kernel safety net. this setting ensures that the root user can still log in and run emergency cleanup commands if a normal user or daemon accidentally fills 100% of the available user disk space.
    > **daemon:** deamons start automatically when the system boots and run quietly behind the scenes without opening a terminal/user interface. in Linux, daemons usually end with a 'd' (crond, systemd, sshd etc.). daemons can handle system monitoring, listen for incoming network connections, write logs, or manage hardware devices.

- _**Typical usage:** standart :_ adjusts how many files can exist on the disk. standart is optimized for normal file and directories. /boot only holds a handful of large kernel image files, standart works for us here.

- _**Bootable flag:** off :_ a legacy MBR flag used by older PC systems to decide which partitions to read first. modern GRUB bootloaders on linux ignore this flag because GRUB is installed directly into the Master Boot Record (MBR) sector at the front of sda

once we are done with the partition settings, the partition will appear on the partition table.

<img src="images/p1_done.png" alt="p1_done" width="500">

<img src="images/ptable_after_p1.png" alt="ptable_after_p1" width="500">

The born2beRoot subject requirement states: "You must create at least 2 encrypted partitions using LVM."

> **An encrypted partition** is a physical or logical disk area where all raw data is mathematically scrambled using a cryptographic algorithm (LUKS/AES in Linux). Without entering the correct password at boot to decrpyt it, the data remains unreadable to anyone pulling the hard drive out.

> **LVM (Logical Volume Manager)** is a storage abstraction layer in Linux that allows you to manage disk space flexibly. in traditional partitioning, we devide a physical hard drive directly into fixed partitions (like sda1 in my project), which are difficult to resize or extend. LVM places a management layer between physical disks and the file systems, letting you treat disk space flexibly.

one more time, let's take a closer look at the given lsblk output example:
- sda5 is the physical partition on disk
- sda_crypt is the decrypted container unlocked by your password
- wil--vg-root, wil--vg-swap_1, and wil--vg-home are individual encrypted logical volumes inside that single decrypted container.

<img src="images/example_lsblk.png" alt="example lsblk output" width="500">

since all three stands inside the sda5_crypt container, all three are encrypted. i will use the example from subject for my project.

selected the FREE SPACE so that i can create a new partition.

<img src="images/remaining_free_space.png" alt="remaining free space" width="500">

<img src="images/create_new_partition.png" alt="create_new_partition" width="500">

as in the example, out of the total disk size (8GB), around 500 mb (487) was allocated to /boot (sda1), and the remaining 7.5GB (max) was given to sda5. i will do the same partitioning.

<img src="images/reserve_remaining_disk.png" alt="reserving remaining disk" width="500">

[Click here to remind yourself tthe Logical partioning type](#logical)

on an MBR disk, primary partition numbers are reserved for slots 1 to 4 (sda1/2/3/4)

logical partitions are created inside an Extended partition (which takes up one of those primary slots, like sda2).
means:
- sda1 is our Primary (/boot) partition
- sda5 is our first Logical partition, placed right after sda1
- sda2 is the Extended container holding the logical space

We reserve and use Logical partitions for any scenario where you need more than 4 total partitions on a traditional MBR disk, or when you want to isolate non-critical user and system data away from the main primary boot files.

Continuted with Logical type

<img src="images/logical_p.png" alt="logical" width="500">

on the next screen, select 'Mount point':

<img src="images/sda5_mount.png" alt="sda5 mount point" width="500">

select 'Do not mount it':

sda5 is a raw container and its job is encryption. it will serve as the raw physical host for our encrypted container. the actual file systems and mount points will be created inside the LVM volumes that sit on top of this encrypted layer. (in the project subject, we can see that sda5 is not mounted there either)

<img src="images/sda5_dont_mount.png" alt="sda5 do not mount" width="500">

after setting the mount point to none, select 'Done setting up the partition' and then we will be able to see the new partition on the partition table.

<img src="images/sda5_done.png" alt="sda5 done" width="500">

we can go for creating the encrypted volumes, select 'Configure encrypted volumes'

<img src="images/configure_encrypted.png" alt="configure encrypted volumes" width="500">

select <Yes> in order to write the current partitioning scheme to the disk before going on with the encrypted volumes. 

i will be using LUKS (Linux Unified Key Setup) as the encryption tool. Linux kernel cannot build an encrypted LUKS container on top of a partition that does not technically exist on the physical drive yet.

<img src="images/write_to_disk.png" alt="write the partitioning scheme to disk" width="500">

now we can create the encrypted volumes:

<img src="images/create_enc_volumes.png" alt="create_enc_volumes" width="500">

select the partition you want to perform the encryption on. we better not select sda1 (/boot), because when our computer turns on and the bootloader is loaded, GRUB needs to read the Linux kernel and initial RAM disk from /boot to start the operating system

or

if /boot is encrypted, GRUB itself must contain the LUKS decryption drivers to prompt us for a password before the linux kernel is even loaded, which is too complicated to be worth it. 

and

standart Linux architecture requires the standart design pattern across Linux distibutions is to keep a _small_, _unencrypted_ /boot partition so that GRUB can easily hand control over the kernel. the kernel handles prompting for the passphrase and decrypting the rest of the disk (which will be sda5_crypt in my project)

and select 'Continue'
 
<img src="images/devices_to_enc.png" alt="devices_to_enc" width="500">

select 'Done setting up the partition'

leave the remaining settings as it is:
- Use as: physical volume for encryption (neat as day)
- Encryption method: Device-mapper (dm-crypt) --> is a tool in the Linux system that locks and hides data on hard drives so people cannot read it without the correct password. Often works with LUKS.
- Mount point: none --> an encrypted raw partition is simply a locked box (dm-crypt container). it doesn't contain a filesystem like Ext4 that Linux can attach to a folder directory. Leaving Mount point as none is mandatory so the installer knows to treat this partition purely as an encrypted container for LVM, rather than trying to mount it directly to the system
- Mount options: defaults --> this option applies to how filesystem flags (dev, auto, exec, async, suid, rw etc.) are passed when mounting. Because this raw partition isn't being mounted directly, altering mount options here has no functional effect on the setup.
- Encryption: aes, Key size: 256 --> AES-256 is the current standart for symmetric encription. provides maximum data protection.
- IV algorithm: xts-plain64 --> XTS is the recommended Cipher Block Chaining mode specifically designed for full-disk encryption. Selecting older initialization vector (IV) modes like cbc-essiv might lead to make the volume more vulnerable to targeted cryptographic attacks
- Encryption key: Passphrase --> this tells LUKS to prompt you for a password when booting up the virtual machine
- Erase data: yes --> keeping it as <yes> hides the real data volume. An attacker looking at the drive won't be able to tell where your actual files end and where empty space begins. it will all look like random noise. It erases any leftover traces of old files that were on the disk before. you can save about 1–2 minutes during installation. however, old data remnants might stay on the drive, and an attacker can see exactly how much real data you actually have stored inside the encrypted container.
- Bootable flag: off --> label that tells your computer's motherboard to start the computer from the specified partition. the system cannot directly start from and encrypted partition because it can't read the files inside without the password first. it won't work anyway if we set it to 'on'

<img src="images/done_enc_part.png" alt="done with the current partition" width="500">

selected 'Finish' since i want to follow the example on the project subject and don't want to create more encrypted volumes.

<img src="images/finish_enc_conf.png" alt="done with creating encrypted volumes" width="500">

The following screen is asking if you want to fill your new encrypted partition with random data before formatting it with LUKS. when preparing an encrypted volume, the installer offers to overwrite the entire partition space with random noise (0s and 1s). filling the partition with random noise prevents attackers from inspecting the raw drive later to tell the difference between actual encrypted files and unused empty space. 

might take a few minutes to finish

<img src="images/erase_sda5_data.png" alt="erase_sda5_data" width="500">

<img src="images/erasing_data.png" alt="erasing_data" width="500">

now we are supposed to enter a password but this time it will be the encryption passphrase, remember it.

<img src="images/encryption_passphrase.png" alt="encryption passphrase" width="500">

<img src="images/verify_enc_passphrase.png" alt="verify encryption passphrase" width="500">


## Configuring the Logical Volume Manager

we will configure the logical volume manager

<img src="images/conf_lvm.png" alt="configure logical volume manager" width="500">
