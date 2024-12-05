# Implementation of RAID 1 with mdadm on Debian

<br>

## RAID 1 Introduction
*RAID 1 (Mirroring) is a storage configuration that duplicates data across two or more disks. This provides high redundancy, as an identical copy of the data is stored on each disk in the array. If one disk fails, the data remains accessible on the other disk(s), ensuring continuous operation without data loss. RAID 1 is primarily used for critical applications where data safety is paramount.*

## Key Features:

- Data Mirroring: All data written to one disk is simultaneously written to another disk.
- Fault Tolerance: Provides high availability by allowing the system to continue running even if a disk fails.
- Performance: Read performance is enhanced since data can be read from any disk in the array, but write performance is generally the same as a single disk due to mirroring.

### Use Case: RAID 1 is ideal for systems where data integrity and uptime are crucial, such as file servers, database servers, and high-availability systems.



## Steps to Set Up RAID 1 with mdadm on Debian

### 1. Install mdadm and Prerequisites
  - First, ensure that mdadm is installed on your Debian system:

```yml
sudo apt update
sudo apt install mdadm
```

### 2. Create RAID 1 Array
  - To create a RAID 1 array with two disks (assuming /dev/sdb and /dev/sdc):

```yml
sudo mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

```
- /dev/md1: Name of the RAID device.
- --level=1: Specifies RAID 1 (mirroring).
- --raid-devices=2: Indicates the number of devices in the RAID array.
- /dev/sdb, /dev/sdc: The two disks to use for the RAID array.

### 3.Monitor the progress using:

```yml
sudo cat /proc/mdstat
```
### 4. Save RAID Configuration to mdadm.conf
  - Once the RAID array is created, save the configuration to /etc/mdadm/mdadm.conf:

```yml
sudo mdadm --detail --scan /dev/md1 >> /etc/mdadm/mdadm.conf
```
  - Update the initramfs to include the RAID configuration:
```yml

sudo update-initramfs -u
```

### 5. . Format the RAID Array
  - Next, format the RAID device (/dev/md1) with a filesystem:

```yml
sudo mkfs.ext4 /dev/md1
```
  - You can replace ext4 with another filesystem if needed (e.g., xfs, btrfs).

### 5. Mount the RAID Array and Add a Test File
  - Create a mount point for the RAID array:

```yml
sudo mkdir /mnt/raid_1
```
  - Now, mount the RAID array:

```yml
sudo mount /dev/md1 /mnt/raid_1/
```

  - Create a test file to verify that the array is working properly:

```yml
echo "This is a test file for RAID 1" | sudo tee /mnt/raid_1/testfile.txt
```

  - Verify the file:

```yml
ls /mnt/raid_1

cat /mnt/raid_1/testfile.txt

```

### 7. Verify the RAID Array
  - To check the RAID array's status, use:

```yml

sudo mdadm --detail /dev/md1
```

  - This will provide detailed information about the array, including the status of the devices and the array itself.

### 8. Remove One Disk from the RAID Array
  - To simulate a disk failure, fail one of the disks in the array:

```yml

sudo mdadm --manage /dev/md1 --fail /dev/sdb

```

  - Check the array status:

```yml
sudo mdadm --detail /dev/md1
```

  - The failed disk (/dev/sdb) will be marked as "failed" or "removed," but the array should still function.

### 9. Verify the File is Still Accessible
  - Despite the disk failure, the data remains mirrored on the other disk. Verify the test file is still accessible:

```yml
cat /mnt/raid_1/testfile.txt
```

### 10. Reassemble the RAID Array
  - To reassemble the RAID array:

1. Stop the RAID array:
```yml
sudo mdadm --stop /dev/md1
```

2. Reassemble the RAID array:
```yml
sudo mdadm --assemble /dev/md1
```

3. Check the RAID array status:
```yml
sudo mdadm --detail /dev/md1
```

### 11. Add a New Disk to the RAID 1 Array
  - To add a new disk (/dev/sdd) to the RAID 1 array and rebuild it:

1. Add the new disk to the array:
```yml
sudo mdadm --add /dev/md1 /dev/sdd
```

2. Monitor the rebuild process:
```yml
sudo cat /proc/mdstat
```

3. After the rebuild completes, verify the RAID status:
```yml
sudo mdadm --detail /dev/md1
```
### 12. Verify the File After Adding the New Disk
  - Verify that the test file is still intact after adding the new disk and rebuilding the array:

```yml
cat /mnt/raid_1/testfile.txt
```
  - The file should still be accessible, confirming that the RAID array is functioning properly and that the data is mirrored across all disks.

## 13. Save the RAID Configuration Again (Post-Modification)
  - If changes were made to the RAID array (e.g., adding new disks), update the mdadm.conf file:

```yml
sudo mdadm --detail --scan /dev/md1 >> /etc/mdadm/mdadm.conf
```

  - Then, update the initramfs again to ensure changes persist across reboots:

```yml
sudo update-initramfs -u
```

<br>
<br>


## Conclusion

*You now have a fully functional RAID 1 array on Debian, complete with steps for creating the array, verifying data, and simulating disk failure and rebuild. This implementation ensures our RAID 1 setup is working as expected, with data mirrored across disks. By adding and rebuilding disks, we can effectively recover from disk failure and maintain data redundancy.*







