## Storage > Block Storage > 概要

インスタンスルートブロックストレージ以外に追加ブロックストレージを使用できます。

- 接続されたインスタンスを削除してもブロックストレージは削除されません。
- ブロックストレージは、複数個のインスタンスから同時に接続して使用できません。
- 接続が解除されたブロックストレージは、他のインスタンスに接続して使用できます。
- ブロックストレージは、同じアベイラビリティーゾーン(availability zone)内にあるインスタンスにのみ接続できます。

ブロックストレージは様々な状況で使用できます。

'- ルートブロックストレージの記憶領域が不足した時、ブロックストレージを追加で装着してインスタンスの記憶領域を増やすことができます。
'- インスタンスを削除する前に、インスタンスのルートブロックストレージにあるデータを永久保存するために、ブロックストレージを装着してデータをコピーできます。

ブロックストレージを使用するには、次の手順を進行する必要があります。

1. [ブロックストレージを作成](/Storage/Block%20Storage/ja/console-guide/#_1)します。
2. 作成したブロックストレージを[対象インスタンスに接続](/Storage/Block%20Storage/ja/console-guide/#_4)します。
3. ブロックストレージ[パーティション作業、フォーマット、マウント](#_1)して使用します。

ブロックストレージはインスタンス実行中にも接続できます。接続されたブロックストレージは空のデバイスなので、使用する前にインスタンスのオペレーションシステムに応じてパーティション作業、フォーマット、マウント作業を直接進行する必要があります。

## 空のブロックのストレージの使用

### Linux

インスタンスに接続した後、下記の手順を進行します。

> [参考]この例に登場する全てのコマンドは、必ず`root`権限で実行する必要があります。

#### パーティションの作成

ブロックストレージがインスタンスに接続されると、空のブロックストレージデバイスとして登録されます。登録されたブロックストレージリストは Linux の`lsblk`コマンドで確認できます。

```
# lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  20G  0 disk
├─vda1 253:1    0   2G  0 part [SWAP]
└─vda2 253:2    0  18G  0 part /
vdb    253:16   0  10G  0 disk
```

上記の例はルートブロックストレージの`vda`と追加ブロックストレージの`vdb`が接続されていることを表しています。 `lsblk`の出力結果を見ると`vda`にパーティションが生成されていますが、 `vdb`は空であることがわかります。

> [参考]ブロックストレージの名前は`vda`、 `vdb`、 `vdc`…というように、インスタンスにブロックストレージを接続した順にアルファベット文字が 1つずつ変わります。上の例で`vdb`は 2 番目に接続されたディスクを意味します。デバイスの名前はコンソールのブロックストレージリスト画面で確認できます。

まず空のブロックストレージデバイスの`vdb`にパーティションを作成します。次のように`fdisk`ユーティリティを利用してブロックストレージ全体を 1つのパーティションで作成します。必要に応じて 1つのブロックストレージを複数のパーティションに分割することもできます。

```
# fdisk /dev/vdb
Command (m for help): n

Partition number (1-4): 1
First cylinder (1-20805, default 1): 1
Last cylinder, +cylinders or +size{K,M,G} (1-20805, default 20805): 20805
Command (m for help): w
```

シェルで`fdisk /dev/{デバイス名}`を入力すると、デバイスのパーティション管理コマンドを入力できるプロンプトが現れます。このプロンプトで使用できるコマンドリストを見るには`m`を入力します。この例では、新たなパーティションを作成するので、'New Partition'を意味する`n`を入力します。すると下記のように作成するパーティションのタイプを尋ねられます。この例では'Primary'を意味する`p`を入力します。パーティション関連のより詳細な情報については[マスターブートレコード](https://ja.wikipedia.org/wiki/%E3%83%9E%E3%82%B9%E3%82%BF%E3%83%BC%E3%83%96%E3%83%BC%E3%83%88%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89)を参照してください。

```
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
```

パーティションのタイプを決定すると、作成するパーティションの個数を尋ねられます。この例ではパーティションを 1 つ作成するので`1`を入力します。

```
Partition number (1-4, default 1): 1
```

次にパーティションのサイズを決定します。作成したブロックストレージのサイズによって入力できる範囲が異なります。この例ではブロックストレージ全体を使用するパーティションを作成するので基本値を使用します。

```
First sector (2048-20971519, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519):
Using default value 20971519
Partition 1 of type Linux and of size 10 GiB is set
```

基本的なパーティション設定が終わりました。これまで入力した設定をブロックストレージに反映するために`w`を入力します。

```
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

これでパーティション作成手順が終わりました。

#### パーティションのフォーマット

作成したパーティションを使用するには、フォーマットする必要があります。`lsblk`コマンドでフォーマットするパーティションを探します。

```
[root@host-192-168-0-67 ~]# lsblk /dev/vdb
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vdb    253:16   0  10G  0 disk
└─vdb1 253:17   0  10G  0 part /mnt/vdb
```

上の例で`vdb`に`vdb1`というパーティションが作成されたのを確認できます。一般的にLinuxでパーティションの名前は'デバイス名 + 数字'の形式です。

次に`vdb1`パーティションをフォーマットします。 Linux では`mkfs`コマンドを使用します。パーティションをフォーマットする時、必ず使用するファイルシステムを指定する必要があります。この例では広く使われている Linux ファイルシステムの 1 つである`xfs`を利用してフォーマットします。Linux で使用可能なファイルシステムについては[ファイルシステム](https://wiki.gentoo.org/wiki/Filesystem/ja)を参照してください。

```
# mkfs -t xfs /dev/vdb1
```

#### ブロックストレージのマウント

ファイルシステムまで作成したブロックストレージには、マウントするとアクセスできます。簡単に`mount`コマンドでブロックストレージをマウントすることもできますが、インスタンスが再起動されるとマウントが解除(unmount)されます。この例では`/etc/fstab`ファイルにマウントするブロックストレージを追加して、インスタンス起動時に自動的にマウントする方法を説明します。

下記は`/etc/fstab`ファイルの内容を出力したものです。

```
# /etc/fstab
# Created by anaconda on Tue Nov 17 18:37:50 2015
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=3d9cc015-610e-4482-9071-fbf998d68121 /                       xfs     defaults,nodev,noatime        1 1
```

ブロックストレージ 1つが既に登録されていることを確認できます。登録されたブロックストレージはインスタンスのルートブロックストレージです。

ブロックストレージを登録するには、ブロックストレージのデバイス固有 ID が必要です。デバイス固有 ID は下記のように`blkid`コマンドで確認できます。

```
# blkid /dev/vdb1
/dev/vdb1: UUID="5a4004f4-3ba6-4484-9459-7c2b321b727f" TYPE="xfs"
```

出力内容の中で、`UUID`に該当する項目がデバイス固有 ID です。

マウント対象ディレクトリを作成します。マウント対象ディレクトリはなんでも構いません。この例では`/mnt/vdb`で行います。

```
mkdir -p /mnt/vdb
```

マウント対象ディレクトリが準備されている場合は、次のようにブロックストレージを登録します。

```
# echo "UUID=5a4004f4-3ba6-4484-9459-7c2b321b727f /mnt/vdb xfs defaults,nodev,noatime 1 2" >> /etc/fstab
```

最後に`/etc/fstab`の内容を反映する必要があります。 `mount -a`コマンドで`/etc/fstab`に登録された全てのブロックストレージをマウントします。

```
# mount -a
```

`df`コマンドで正常にマウントできたか確認します。

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

新しいパーティションがマウントされていることを確認できます。

各コマンドの詳細については Linux の`man`コマンドで確認できます。

> [参考]上記の手順を一気に処理するには、下記のスクリプトを参照してください。
> 下記のスクリプトは CentOS でテストしたものです。

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
    mkfs -t $FS_TYPE -f $PART_NAME > /dev/null

    UUID=`blkid $PART_NAME -o export | grep UUID | cut -d'=' -f 2`
    echo "UUID=$UUID $MOUNT_DIR $FS_TYPE defaults,nodev,noatime 1 2" >> /etc/fstab

    mount -a
done
```

### Windows

Windows でボリュームを追加する方法は大きく 2 つです。1 つ目は GUI ベースの**サーバー管理者**を使用する方法で、 2 つ目は CLI ベースの**PowerShell**を使用する方法です。この文書では、それぞれの方法を簡単に紹介します。

#### **サーバー管理者**を使用してボリューム追加

Windows インスタンスに接続されたブロックストレージは、オフライン状態のディスクとして表示されます。このディスクを使用するには、オンライン状態に変更してボリュームを作成する必要があります。オンライン状態に切り替える手順は次のとおりです。

1. インスタンスに接続した後、起動画面で**サーバー管理者**をクリックします。
2. **サーバー管理者 > ダッシュボード** 画面で**ファイル及びストレージサービス**を選択します。
3. ファイル及びストレージサービスのサーバー画面で**ボリューム > ディスク**を選択します。
4. オフライン状態の新しいディスクを右クリックして、 **オンライン状態に転換**を選択します。
5. オンライン状態に転換するか尋ねるメッセージが表示されたら**はい**をクリックします。
6. ディスクリストを更新し、新しいディスクがオンライン状態になっているか確認します。

ディスクがオンライン状態に転換されていれば、新しいボリュームを作成できます。新しいボリュームを作成する手順は次のとおりです。

1. ディスクリストから新しいディスクを右クリックした後、 **新しいボリューム**を選択します。
2. **新しいボリュームウィザード** ダイアログボックスでボリュームを作成するディスクを選択します。
3. 作成するボリュームのサイズを指定します。
4. ドライブ文字を選択します。
5. ボリュームのファイルシステムを選択します。
6. 最後に設定した項目を確認し、**作成**を選択します。

作成したボリュームは**ディスク管理** プログラムで使用できるように設定します。

1. **スタート** ボタンを右クリックして **ディスク管理**を選択します。
2. **ディスクの管理**のディスクリストから、ボリュームを作成したディスクを右クリックして **ドライブ文字とパスの変更**を選択します。
3. 新しいボリュームのドライブ文字とパスを追加します。

これで Windows エクスプローラでディスクが追加されたことを確認できます。ディスク管理に関するより詳細な説明は[新しいディスクの初期化 | Microsoft Docs](https://docs.microsoft.com/ja-jp/windows-server/storage/disk-management/initialize-new-disks)を参照してください。

> [参考] Windows のディスク形式は 2 種類です。
>
> - MBR：以前から使用されているディスク形式で、 2TB 以下のディスクで使用します。
> - GPT： 2TB 以上のディスクで使用する新しいディスク形式です。
>   **サーバー管理者**で設定したディスクは基本的に GPT 形式です。

#### PowerShell を使用してボリュームを追加

Windows で提供する PowerShell でもボリュームを追加できます。 **スタート**をクリックして**Windows PowerShell**をクリックします。

`Get-Disk`コマンドで現在のインスタンスに接続されているディスクリストを出力します。下の結果で`RAW`と表示されているディスクが新たに追加するディスクです。

```
PS C:\Users\Administrator> Get-Disk
Number Friendly Name                            OperationalStatus                    Total Size Partition Style
------ -------------                            -----------------                    ---------- ---------------
0      Red Hat VirtIO SCSI Disk Device          Online                                    50 GB MBR
1      Red Hat VirtIO SCSI Disk Device          Offline                                   10 GB RAW
```

`Initialize-Disk`コマンドでディスクを初期化します。各オプションの説明は次のとおりです。

- Number：初期化するディスクの番号を指定します。
- PartitionStyle：パーティション形式を指定します。この例では**サーバー管理者**での場合同様、パーティション形式を GPT に指定します。

```
PS C:\Users\Administrator> Initialize-Disk -Number 1 -PartitionStyle GPT
Number Friendly Name                            OperationalStatus                    Total Size Partition Style
------ -------------                            -----------------                    ---------- ---------------
1      Red Hat VirtIO SCSI Disk Device          Offline                                   10 GB RAW
```

`New-Partition`コマンドでパーティションを作成します。各オプションの説明は次のとおりです。

- DiskNumber：パーティションを作成するディスク番号を選択します。
- AssignDriveLetter：作成したパーティションにドライブ文字を自動的に割り当てるように設定します。
- UseMaximumSize：パーティションのサイズにディスク可用容量全体を選択します。

```
PS C:\Users\Administrator> New-Partition -DiskNumber 1 -AssignDriveLetter -UseMaximumSize
   Disk Number: 1
PartitionNumber  DriveLetter Offset                                        Size Type
---------------  ----------- ------                                        ---- ----
2                D           34603008                                   9.97 GB Basic
```

`Format-Volume`コマンドでパーティションをフォーマットします。各オプションの説明は次のとおりです。

- FileSystem：使用するファイルシステム形式を指定します。この例では NTFS に指定します。
- Confirm：ユーザー確認のためのプロンプト出力をするか指定します。この例では尋ねないように false に設定します。

```
PS C:\Users\Administrator> Format-Volume -DriveLetter D -FileSystem NTFS -Confirm:$false

DriveLetter       FileSystemLabel  FileSystem       DriveType        HealthStatus        SizeRemaining             Size
-----------       ---------------  ----------       ---------        ------------        -------------             ----
D                                  NTFS             Fixed            Healthy                   9.92 GB          9.97 GB
```

これで、Windows エクスプローラでディスクが追加されたことを確認できます。 PowerShell のより詳細な説明は[PowerShell Module Browser | Microsoft Docs](https://docs.microsoft.com/ja-jp/powershell/module/?view=win10-ps)を参照してください。

> [参考]上の手順を一度に処理するには、下記のスクリプトを参考にしてください。

```
PS C:\Users\Administrator> Get-Disk |
   Where PartitionStyle -eq 'RAW' |
   Initialize-Disk -PartitionStyle GPT -PassThru |
   New-Partition -AssignDriveLetter -UseMaximumSize |
   Format-Volume -FileSystem NTFS -Confirm:$false
```

## ブロックストレージのスナップショット

ブロックストレージのスナップショット機能を利用すると、ユーザーが直接ブロックストレージのデータをコピーするより速くブロックストレージをバックアップできます。ブロックストレージがインスタンスに接続されている状態でもブロックストレージのスナップショットを作成できますが、データの整合性と安定性を保障するためにインスタンスで接続を解除してブロックストレージのスナップショットを作成することを推奨します。安定性向上のため、ブロックストレージをインスタンスから接続解除する前にマウント解除(unmount)を行います。

ブロックストレージのスナップショットは読み取り専用になっているので、インスタンスに直接接続して使用できません。スナップショットを使用するにはスナップショットからブロックストレージを作成し、作成されたブロックストレージをインスタンスに接続します。

> [注意]
> ブロックストレージのスナップショットを持つブロックストレージは削除できません。ブロックストレージを削除するには、そのブロックストレージの全てのスナップショットを削除してください。

### 課金

ブロックストレージは作成した瞬間から課金され、ブロックストレージの作成時に設定された「ブロックストレージのサイズ」に応じて課金されます。スナップショットの場合、元のブロックストレージのサイズに応じて料金が課金されます。
