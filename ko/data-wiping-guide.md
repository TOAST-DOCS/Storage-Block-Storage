
## Storage > Block Storage > 데이터 완전 삭제 가이드

인스턴스에 연결된 블록 스토리지 및 NAS의 데이터를 완전히 삭제하는 방법을 소개합니다.
KISA 데이터 완전 삭제 기준인 DoD 5220.22-M 패턴을 적용할 수 있는 scrub(Linux), Disk Wipe(Windows)를 이용해 데이터를 완전히 삭제할 수 있습니다.

## Linux 인스턴스

1. scrub 패키지를 설치합니다.
    * Debian, Ubuntu 인스턴스

            $ sudo apt-get install scrub

    * CentOS 인스턴스

            $ sudo yum install scrub


2. 데이터를 완전히 삭제할 스토리지 디바이스를 확인합니다.

        $ df -h
 
3. 데이터를 완전히 삭제합니다.
    * (블록 스토리지의 경우) dod 패턴 적용

            $ sudo scrub -p dod /dev/vdb1
 
    * (NAS의 경우) 마운트된 디렉터리에 dod 패턴 적용

            $ sudo scrub -p dod -X /home/ubuntu/nas/scrup


## Windows 인스턴스

1. [https://www.diskwipe.org](https://www.diskwipe.org)에서 Disk Wipe를 다운로드합니다.
2. 관리자 권한으로 Disk Wipe를 실행하여 데이터를 완전히 삭제할 드라이브를 선택한 후, 왼쪽 상단의 **Wipe Disk** 버튼을 클릭합니다.
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_01.png)
3. **File System** 항목에서 **NTFS**를 선택합니다.
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_02.png)
4. **Erasing Pattern** 항목에서 **US Department of Defense DoD 5220.22-M(E) (3 passes - slow)**를 선택합니다.
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_03.png)
5. `ERASE ALL`을 입력한 후 **Finish**(완료) 버튼을 클릭합니다.
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_04.png)
6. 작업을 완료한 후 드라이브의 데이터를 확인합니다.
