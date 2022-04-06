
## Storage > Block Storage > 데이터 완전 삭제 가이드

인스턴스에 연결된 블록 스토리지 및 NAS의 데이터를 완전 삭제하는 방법을 소개합니다.
KISA 데이터 완전 삭제 기준인 DoD 5220.22-M 패턴을 적용 가능한 도구로, scrub(Linux), diskwipe(Windows)를 이용하여 데이터를 완전 삭제할 수 있습니다.

## Linux 인스턴스

* scrub 패키지 설치
```
# Debian, Ubuntu 인스턴스
$ sudo apt-get install scrub

# CentOS 인스턴스
$ sudo yum install scrub
```

* 데이터 완전 삭제를 지정할 스토리지 디바이스 확인
```
$ df -h
```
 

* (블록 스토리지의 경우) dod 패턴을 적용하여 데이터 완전 삭제 수행
```
$ sudo scrub -p dod /dev/vdb1
```
 
* (NAS의 경우) 마운트된 디렉토리에 dod 패턴을 적용하여 데이터 완전 삭제 수행
```
$ sudo scrub -p dod -X /home/ubuntu/nas/scrup
```


## Windows 인스턴스

1. [https://www.diskwipe.org](https://www.diskwipe.org)에서 diskwipe 다운로드
2. `관리자 권한`으로 diskwipe를 실행하여 데이터 완전 삭제하고자 하는 드라이브를 선택 후, 왼쪽 상단의 `Wipe Disk` 버튼 클릭
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_01.png)
3. File System 항목에서 `NTFS` 선택
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_02.png)
4. Erasing Pattern 항목에서 `DoD 5220.22-M(E)` 선택
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_03.png)
5. `ERASE ALL` 입력 후 완료 버튼 클릭
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_04.png)
6. 작업 완료 후 드라이브 내의 데이터 확인
