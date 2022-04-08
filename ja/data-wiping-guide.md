
## Storage > Block Storage > データ完全削除ガイド

インスタンスに接続されたブロックストレージおよびNASのデータを完全に削除する方法を紹介します。
KISAデータ完全削除基準でるDoD 5220.22-Mパターンを適用できるscrub(Linux), Disk Wipe(Windows)を利用してデータを完全に完全に削除できます。

## Linuxインスタンス

1. scrubパッケージをインストールします。
    * Debian, Ubuntuインスタンス

            $ sudo apt-get install scrub

    * CentOSインスタンス

            $ sudo yum install scrub


2. データを完全に削除するストレージデバイスを確認します。

        $ df -h
 
3. データを完全に削除します。
    * (ブロックストレージの場合) dodパターン適用

            $ sudo scrub -p dod /dev/vdb1
 
    * (NASの場合)マウントされたディレクトリにdodパターン適用

            $ sudo scrub -p dod -X /home/ubuntu/nas/scrup


## Windowsインスタンス

1. [https://www.diskwipe.org](https://www.diskwipe.org)からDisk Wipeをダウンロードします。
2. 管理者権限でDisk Wipeを実行してデータを完全に削除するドライブを選択した後、左上の**Wipe Disk**ボタンをクリックします。
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_01.png)
3. **File System** 項目で**NTFS**を選択します。
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_02.png)
4. **Erasing Pattern**項目で**US Department of Defense DoD 5220.22-M(E) (3 passes - slow)**を選択します。
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_03.png)
5. `ERASE ALL`を入力した後、**Finish**(完了)ボタンをクリックします。
![image.png](https://static.toastoven.net/prod_infrastructure/block_storage/data_disposal_04.png)
6. 作業を完了した後、ドライブのデータを確認します。
