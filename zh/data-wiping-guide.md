
## Storage > Block Storage > Data Wiping Guide

This guide describes how to wipe out the data on block storage or NAS attached to an instance.
You can wipe out data by using scrub (Linux) or Disk Wipe (Windows) that can apply the DoD 5220.22-M pattern, which is the KISA data wiping standard.

## Linux Instance

1. Install the scrub package.
    * Debian or Ubuntu instance

            $ sudo apt-get install scrub

    * CentOS instance

            $ sudo yum install scrub


2. Identify the storage device from which data will be wiped out.

        $ df -h
 
3. Wipe out the data.
    * (For block storage) Apply the dod pattern

            $ sudo scrub -p dod /dev/vdb1
 
    * (For NAS) Apply the dod pattern to the mounted directory

            $ sudo scrub -p dod -X /home/ubuntu/nas/scrup


## Windows Instance

1. Download Disk Wipe from [https://www.diskwipe.org](https://www.diskwipe.org).
2. Run Disk Wipe as administrator, select the drive from which data will be wiped out, and click the **Wipe Disk** button in the upper left corner.
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_01.png)
3. Select **NTFS** in **File System**.
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_02.png)
4. Select **US Department of Defense DoD 5220.22-M(E) (3 passes - slow)** in **Erasing Pattern**.
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_03.png)
5. Enter `ERASE ALL` and click the **Finish** button.
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_04.png)
6. After completing the operation, check the data on the drive.
