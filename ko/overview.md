## Storage > Block Storage > 개요

인스턴스 루트 블록 스토리지 외에 추가 블록 스토리지를 사용할 수 있습니다.

- 연결된 인스턴스를 삭제하더라도 블록 스토리지는 삭제되지 않습니다.
- 블록 스토리지는 여러 개의 인스턴스에서 동시에 연결하여 사용할 수 없습니다.
- 연결이 해제된 블록 스토리지는 다른 인스턴스에 연결하여 사용할 수 있습니다.
- 블록 스토리지는 같은 가용성 영역(availability zone) 안에 있는 인스턴스에만 연결할 수 있습니다.

블록 스토리지는 여러 상황에서 사용할 수 있습니다.

- 루트 블록 스토리지의 저장 공간이 부족할 때 블록 스토리지를 추가로 장착하여 인스턴스의 저장 공간을 늘릴 수 있습니다.
- 인스턴스를 삭제하기 전에 인스턴스의 루트 블록 스토리지에 있는 데이터를 영구 보관하기 위해 블록 스토리지를 장착하고 데이터를 복사할 수 있습니다.

블록 스토리지를 사용하려면 다음 과정을 진행해야 합니다.

1. [블록 스토리지를 생성](/Storage/Block%20Storage/ko/console-guide/#_1)합니다.
2. 생성한 블록 스토리지를 [대상 인스턴스에 연결](/Storage/Block%20Storage/ko/console-guide/#_4)합니다.
3. 블록 스토리지 [파티션 작업, 포맷, 마운트](#_1)하여 사용합니다.

블록 스토리지는 인스턴스 실행 중에도 연결할 수 있습니다. 연결된 블록 스토리지는 빈 장치이므로 사용하기 전에 인스턴스의 운영체제에 따라 파티션 작업, 포맷, 마운트 작업을 직접 진행해야 합니다.


## 빈 블록 스토리지 사용
### Linux

인스턴스에 접속한 후 아래의 과정을 진행합니다.

> [참고] 이 예제에서 나오는 모든 명령은 반드시 `root` 권한으로 실행해야 합니다.

#### 파티션 생성
블록 스토리지가 인스턴스에 연결되면 빈 장치로 등록됩니다. 등록된 블록 스토리지 목록은 리눅스의 `lsblk` 명령어로 확인할 수 있습니다.
```
# lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  20G  0 disk
├─vda1 253:1    0   2G  0 part [SWAP]
└─vda2 253:2    0  18G  0 part /
vdb    253:16   0  10G  0 disk
```
위의 예제는 루트 블록 스토리지인 `vda`와 추가 블록 스토리지인 `vdb`가 연결되어 있음을 나타냅니다. `lsblk`의 출력 결과를 보면 `vda`에 파티션이 생성되어 있지만 `vdb`는 비어 있는 것을 알 수 있습니다.

> [참고] 블록 스토리지 장치의 이름은 `vda`, `vdb`, `vdc`... 와 같이 인스턴스에 블록 스토리지를 연결한 순서대로 알파벳 문자가 하나씩 올라가게 됩니다. 위의 예제에서 `vdb`는 두 번째로 연결된 디스크를 의미합니다. 장치의 이름은 콘솔의 블록 스토리지 목록 화면에서 확인할 수 있습니다.

먼저 빈 장치인 `vdb`에 파티션을 생성합니다. 다음과 같이 `fdisk` 유틸리티를 이용하여 블록 스토리지 전체를 파티션 하나로 만듭니다. 필요에 따라 블록 스토리지 하나를 여러 파티션으로 나눌 수도 있습니다.
```
# fdisk /dev/vdb
Command (m for help): n

Partition number (1-4): 1
First cylinder (1-20805, default 1): 1
Last cylinder, +cylinders or +size{K,M,G} (1-20805, default 20805): 20805
Command (m for help): w
```
셸에서 `fdisk /dev/{장치 이름}`을 입력하면 장치의 파티션 관리 명령을 입력할 수 있는 프롬프트가 나옵니다. 이 프롬프트에서 사용할 수 있는 명령 목록을 보려면 `m`을 입력합니다. 이 예제에서는 새로운 파티션을 생성할 것이므로 'New Partition'을 의미하는 `n`을 입력합니다. 그러면 아래와 같이 생성할 파티션의 타입을 물어봅니다. 이 예제에서는 'Primary'를 의미하는 `p`를 입력합니다. 파티션과 관련한 보다 자세한 정보는 [마스터 부트 레코드](https://ko.wikipedia.org/wiki/%EB%A7%88%EC%8A%A4%ED%84%B0_%EB%B6%80%ED%8A%B8_%EB%A0%88%EC%BD%94%EB%93%9C)를 참고합니다.
```
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
```
파티션의 타입을 결정하면 생성할 파티션의 갯수를 물어봅니다. 이 예제에서는 파티션 한 개를 생성할 것이므로 `1`을 입력합니다.
```
Partition number (1-4, default 1): 1
```
이제 파티션의 크기를 결정합니다. 생성한 블록 스토리지의 크기에 따라 입력할 수 있는 범위가 달라집니다. 이 예제에서는 전체 블록 스토리지를 사용하는 파티션을 생성할 것이므로 기본값을 선택합니다.
```
First sector (2048-20971519, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519):
Using default value 20971519
Partition 1 of type Linux and of size 10 GiB is set
```
기본적인 파티션 설정이 끝났습니다. 지금까지 입력한 설정을 블록 스토리지에 반영하기 위해 `w`를 입력합니다.
```
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```
이것으로 파티션 생성 과정이 끝났습니다.

#### 파티션 포맷
생성한 파티션을 사용하려면 포맷을 해야 합니다. `lsblk` 명령으로 포맷할 파티션을 찾습니다.
```
[root@host-192-168-0-67 ~]# lsblk /dev/vdb
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vdb    253:16   0  10G  0 disk
└─vdb1 253:17   0  10G  0 part /mnt/vdb
```
위의 예제에서 `vdb`에 `vdb1`이라는 파티션이 생성된 것을 확인할 수 있습니다. 일반적으로 Linux에서 파티션의 이름은 '장치이름 + 숫자' 형태입니다.

이제 `vdb1` 파티션을 포맷합니다. Linux에서는 `mkfs` 명령어를 사용합니다. 파티션을 포맷할 때 반드시 사용할 파일 시스템을 지정해야 합니다. 이 예제에서는 널리 쓰이는 Linux 파일 시스템 중 하나인 `xfs`를 이용하여 포맷합니다. Linux에서 사용 가능한 파일 시스템에 대해서는 [파일 시스템](https://ko.wikipedia.org/wiki/%ED%8C%8C%EC%9D%BC_%EC%8B%9C%EC%8A%A4%ED%85%9C#%EB%A6%AC%EB%88%85%EC%8A%A4%EC%9D%98_%ED%8C%8C%EC%9D%BC_%EC%8B%9C%EC%8A%A4%ED%85%9C)을 참고합니다.
```
# mkfs -t xfs /dev/vdb1
```

#### 블록 스토리지 마운트
파일 시스템까지 만든 블록 스토리지는 마운트 과정을 거쳐야 접근할 수 있습니다. 간편하게 `mount` 명령어로 블록 스토리지를 마운트 할 수도 있으나 인스턴스가 재부팅되면 마운트 해제(unmount)가 됩니다. 이 예제에서는 `/etc/fstab` 파일에 마운트 할 블록 스토리지를 추가하여 인스턴스 부팅 과정에서 자동으로 마운트 하는 방법을 설명합니다.

아래는 `/etc/fstab` 파일 내용을 출력한 것입니다.
```
# /etc/fstab
# Created by anaconda on Tue Nov 17 18:37:50 2015
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=3d9cc015-610e-4482-9071-fbf998d68121 /                       xfs     defaults,nodev,noatime        1 1
```
블록 스토리지 하나가 이미 등록된 것을 확인할 수 있습니다. 등록된 블록 스토리지는 인스턴스의 루트 블록 스토리지입니다.

블록 스토리지를 등록하려면 장치 고유 ID가 필요합니다. 장치 고유 아이디는 아래와 같이 `blkid` 명령어로 확인할 수 있습니다.
```
# blkid /dev/vdb1
/dev/vdb1: UUID="5a4004f4-3ba6-4484-9459-7c2b321b727f" TYPE="xfs"
```
출력 내용 중 `UUID`에 해당하는 항목이 바로 장치 고유 ID입니다.

마운트 대상 디렉터리를 생성합니다. 마운트 대상 디렉터리는 어떠한 것이라도 상관없습니다. 이 예제에서는 `/mnt/vdb`로 합니다.
```
mkdir -p /mnt/vdb
```
마운트 대상 디렉터리가 준비되었으면 다음과 같이 블록 스토리지를 등록합니다.
```
# echo "UUID=5a4004f4-3ba6-4484-9459-7c2b321b727f /mnt/vdb xfs defaults,nodev,noatime,nofail 1 2" >> /etc/fstab
```

> [주의] 볼륨 마운트에 실패하더라도 부팅이 될 수 있도록 `nofail` 옵션을 추가하는 것을 권장합니다.

마지막으로 `/etc/fstab`의 내용을 반영해야 합니다. `mount -a` 명령어로 `/etc/fstab`에 등록된 모든 블록 스토리지를 마운트 합니다.
```
# mount -a
```
`df` 명령어로 정상적으로 마운트됐는지 확인합니다.
```
# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        18G  1.3G   17G   7% /
devtmpfs        912M  4.0K  912M   1% /dev
tmpfs           921M     0  921M   0% /dev/shm
tmpfs           921M   89M  832M  10% /run
tmpfs           921M     0  921M   0% /sys/fs/cgroup
/dev/vdb1        10G   33M   10G   1% /mnt/vdb
```
새 파티션이 마운트되어 있는 것을 확인할 수 있습니다.

각 명령어에 대한 자세한 설명은 Linux의 `man` 명령으로 확인할 수 있습니다.

> [참고] 위의 과정을 한 번에 처리하려면 아래의 스크립트를 참고하시기 바랍니다.
> 아래의 스크립트는 CentOS 7에서 테스트한 것입니다.

```
#!/bin/bash

DEVICES=(`lsblk -s -d -o name,type | grep disk | awk '{print $1}'`)

for DEVICE_NAME in ${DEVICES[@]}
do
   MOUNT_DIR=/mnt/$DEVICE_NAME
   FS_TYPE=xfs

   mkdir -p $MOUNT_DIR

   echo -e "n\np\n1\n\n\nw" | fdisk /dev/$DEVICE_NAME
   PART_NAME="/dev/${DEVICE_NAME}1"
   mkfs -t $FS_TYPE $PART_NAME > /dev/null

   UUID=`blkid $PART_NAME -o export | grep "^UUID=" | cut -d'=' -f 2`
   echo "UUID=$UUID $MOUNT_DIR $FS_TYPE defaults,nodev,noatime,nofail 1 2" >> /etc/fstab

   mount -a
done
```


> [참고] CentOS6, Debian, Ubuntu는 기본 파일 시스템이 ext4 입니다.
> 따라서 아래의 스크립트를 사용해야 합니다.

```
#!/bin/bash

DEVICES=(`lsblk -s -d -o name,type | grep disk | awk '{print $1}'`)

for DEVICE_NAME in ${DEVICES[@]}
do
   MOUNT_DIR=/mnt/$DEVICE_NAME
   FS_TYPE=ext4

   mkdir -p $MOUNT_DIR

   echo -e "n\np\n1\n\n\nw" | fdisk /dev/$DEVICE_NAME
   PART_NAME="/dev/${DEVICE_NAME}1"
   mkfs -t $FS_TYPE $PART_NAME > /dev/null

   UUID=`blkid $PART_NAME -o export | grep "^UUID=" | cut -d'=' -f 2`
   echo "UUID=$UUID $MOUNT_DIR $FS_TYPE defaults,nodev,noatime,nofail 1 2" >> /etc/fstab

   mount -a
done
```

### Windows

Windows에서 볼륨을 추가하는 방법은 크게 두 가지입니다. 첫 번째는 GUI 기반의 **서버 관리자**를 사용하는 것이고, 두 번째는 CLI 기반의 **PowerShell**을 사용하는 것입니다. 이 문서에서는 각각의 방법을 간략하게 소개합니다.

#### **서버 관리자**를 사용하여 볼륨 추가
Windows 인스턴스에 연결된 블록 스토리지는 오프라인 상태의 디스크로 표시됩니다. 이 디스크를 사용하려면 온라인 상태로 변경하고 볼륨을 생성해야 합니다. 온라인 상태로 전환하는 과정은 다음과 같습니다.

1. 인스턴스에 접속한 후 시작 화면에서 **서버 관리자**를 클릭합니다.
2. **서버 관리자 > 대시보드** 화면에서 **파일 및 저장소 서비스**를 선택합니다.
3. 파일 및 저장소 서비스의 서버 화면에서 **볼륨 > 디스크**를 선택합니다.
4. 오프라인 상태의 새 디스크를 마우스 오른쪽 버튼으로 클릭한 후 **온라인 상태로 전환**을 선택합니다.
5. 온라인 상태로 전환을 묻는 메시지가 나타나면 **예**를 클릭합니다.
6. 디스크 목록을 새로 고침하여 새 디스크가 온라인 상태로 전환되었는지 확인합니다.

디스크가 온라인 상태로 전환되면 새 볼륨을 만들 수 있습니다. 새 볼륨을 만드는 과정은 다음과 같습니다.

1. 디스크 목록에서 새 디스크를 마우스 오른쪽 버튼으로 클릭한 후 **새 볼륨**을 선택합니다.
2. **새 볼륨 마법사** 대화 상자에서 볼륨을 생성할 디스크를 선택합니다.
3. 생성할 볼륨의 크기를 지정합니다.
4. 드라이브 문자를 선택합니다.
5. 볼륨의 파일 시스템을 선택합니다.
6. 마지막으로 설정한 항목들을 확인한 후 **만들기**를 선택합니다.

생성된 볼륨은 **디스크 관리** 프로그램에서 사용할 수 있게 설정합니다.

1. **시작** 버튼을 마우스 오른쪽 버튼으로 클릭한 후 **디스크 관리**를 선택합니다.
2. **디스크 관리**의 디스크 목록에서 볼륨을 생성한 디스크를 마우스 오른쪽 버튼으로 클릭한 후 **드라이브 문자 및 경로 변경**을 선택합니다.
3. 새 볼륨의 드라이브 문자 및 경로를 추가합니다.

이제 Windows 탐색기에서 디스크가 추가된 것을 확인할 수 있습니다. 디스크 관리에 관한 보다 자세한 설명은 [새 디스크 초기화 | Microsoft Docs](https://docs.microsoft.com/ko-kr/windows-server/storage/disk-management/initialize-new-disks)를 참고합니다.

> [참고] Windows의 디스크 형식은 두 가지입니다.
> * MBR: 예전부터 사용되는 디스크 형식이며, 2TB 이하의 디스크에서 사용합니다.
> * GPT: 2TB 이상의 디스크에서 사용하는 새로운 디스크 형식입니다.
> **서버 관리자**에서 설정한 디스크는 기본적으로 GPT 형식입니다.

#### PowerShell을 사용하여 볼륨 추가
Windows에서 제공하는 PowerShell로도 볼륨을 추가할 수 있습니다. **시작**을 클릭한 후 **Windows PowerShell**을 클릭합니다.

`Get-Disk` 명령으로 현재 인스턴스에 연결된 디스크 목록을 출력합니다. 아래의 결과에서 `RAW`로 표시된 디스크가 새로 추가할 디스크입니다.
```
PS C:\Users\Administrator> Get-Disk
Number Friendly Name                            OperationalStatus                    Total Size Partition Style
------ -------------                            -----------------                    ---------- ---------------
0      Red Hat VirtIO SCSI Disk Device          Online                                    50 GB MBR
1      Red Hat VirtIO SCSI Disk Device          Offline                                   10 GB RAW
```

`Initialize-Disk` 명령으로 디스크를 초기화합니다. 각 옵션 설명은 다음과 같습니다.

* Number: 초기화할 디스크의 번호를 지정합니다.
* PartitionStyle: 디스크 형식을 지정합니다. 이 예제에서는 **서버 관리자**에서의 경우와 마찬가지로 디스크 형식을 GPT로 지정합니다.

```
PS C:\Users\Administrator> Initialize-Disk -Number 1 -PartitionStyle GPT
Number Friendly Name                            OperationalStatus                    Total Size Partition Style
------ -------------                            -----------------                    ---------- ---------------
1      Red Hat VirtIO SCSI Disk Device          Offline                                   10 GB RAW
```

`New-Partition` 명령으로 파티션을 생성합니다. 각 옵션 설명은 다음과 같습니다.

* DiskNumber: 파티션을 생성할 디스크 번호를 선택합니다.
* AssignDriveLetter: 생성한 파티션에 드라이브 문자를 자동으로 할당하도록 설정합니다.
* UseMaximumSize: 파티션의 크기로 디스크 가용 용량 전체를 선택합니다.

```
PS C:\Users\Administrator> New-Partition -DiskNumber 1 -AssignDriveLetter -UseMaximumSize
   Disk Number: 1
PartitionNumber  DriveLetter Offset                                        Size Type
---------------  ----------- ------                                        ---- ----
2                D           34603008                                   9.97 GB Basic
```

`Format-Volume` 명령으로 파티션을 포맷합니다. 각 옵션 설명은 다음과 같습니다.

* FileSystem: 사용할 파일 시스템 형식을 지정합니다. 이 예제에서는 NTFS로 지정합니다.
* Confirm: 사용자 확인을 위한 프롬프트 출력 여부를 지정합니다. 이 예제에서는 묻지 않도록 false로 설정합니다.

```
PS C:\Users\Administrator> Format-Volume -DriveLetter D -FileSystem NTFS -Confirm:$false

DriveLetter       FileSystemLabel  FileSystem       DriveType        HealthStatus        SizeRemaining             Size
-----------       ---------------  ----------       ---------        ------------        -------------             ----
D                                  NTFS             Fixed            Healthy                   9.92 GB          9.97 GB
```

이제 Windows 탐색기에서 디스크가 추가된 것을 확인할 수 있습니다. PowerShell에 대한 보다 자세한 설명은 [PowerShell Module Browser | Microsoft Docs](https://docs.microsoft.com/ko-kr/powershell/module/?view=win10-ps)를 참고합니다.

> [참고] 위의 과정을 한 번에 처리하려면 아래의 스크립트를 참고하시기 바랍니다. 아래의 스크립트는 PowerShell 3.0 이상을 지원하는 Windows 2012, 2016에서 테스트한 것입니다.
```
Get-Disk |
 Where PartitionStyle -eq 'RAW' |
 Initialize-Disk -PartitionStyle GPT -PassThru |
 New-Partition -AssignDriveLetter -UseMaximumSize |
 Format-Volume -FileSystem NTFS -Confirm:$false
```

> [참고] PowerShell 2.0까지 지원하는 Windows 2008에서는 아래의 스크립트를 사용하여 디스크를 추가할 수 있습니다.
```
$newdisk = gwmi -Query "SELECT * FROM Win32_diskdrive where partitions=0"
$index = $newdisk.index
$Scriptblock=@"
select disk=$index
clean
convert GPT
Create Partition Primary
Format fs=ntfs quick
assign
"@
$Scriptblock | diskpart
```

## 블록 스토리지 스냅숏

블록 스토리지 스냅숏 기능을 이용하면 사용자가 직접 블록 스토리지의 데이터를 복사하는 것보다 빠르게 블록 스토리지를 백업할 수 있습니다. 블록 스토리지가 인스턴스에 연결되어 있는 상태에서도 블록 스토리지 스냅숏을 생성할 수 있지만, 데이터의 정합성과 안정성을 보장하기 위해 인스턴스에서 연결을 해제하고 블록 스토리지 스냅숏을 생성하기를 권장합니다. 안정성 향상을 위해 블록 스토리지를 인스턴스에서 연결 해제하기 전에 마운트 해제(unmount)를 합니다.

블록 스토리지의 스냅숏은 읽기 전용으로 되어 있으므로 인스턴스에 직접 연결하여 사용할 수 없습니다. 스냅숏을 사용하려면 스냅숏으로부터 블록 스토리지를 생성하고, 생성된 블록 스토리지를 인스턴스에 연결합니다.

> [주의]
블록 스토리지 스냅숏을 갖는 블록 스토리지는 삭제할 수 없습니다. 블록 스토리지를 삭제하려면 그 블록 스토리지의 모든 스냅숏을 삭제하시기 바랍니다.

### 과금

블록 스토리지는 생성한 순간부터 요금이 부과되며, 블록 스토리지 생성 시 설정된 `블록 스토리지 크기`에 따라 요금이 부과됩니다. 스냅숏의 경우 원본 블록 스토리지의 크기에 따라 요금이 부과됩니다.
