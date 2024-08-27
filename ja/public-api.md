## Storage > Block Storage > API v2ガイド

APIを使用するにはAPIエンドポイントとトークンなどが必要です。 [API使用準備](/Compute/Compute/ko/identity-api/)を参照してAPIを使用するのに必要な情報を準備します。

ブロックストレージAPIは`volumev2`タイプエンドポイントを利用します。正確なエンドポイントはトークン発行レスポンスの`serviceCatalog`を参照します。

| タイプ | リージョン | エンドポイント |
|---|---|---|
| volumev2 | 韓国(パンギョ)リージョン<br>韓国(ピョンチョン)リージョン<br>日本リージョン<br>米国(カリフォルニア)リージョン | https://kr1-api-block-storage-infrastructure.nhncloudservice.com<br>https://kr2-api-block-storage-infrastructure.nhncloudservice.com<br>https://jp1-api-block-storage-infrastructure.nhncloudservice.com<br>https://us1-api-block-storage-infrastructure.nhncloudservice.com |

APIレスポンスにガイドに明示されていないフィールドが表示される場合があります。それらのフィールドは、NHN Cloud内部用途で使用され、事前に告知せずに変更する場合があるため使用しないでください。

## ブロックストレージタイプ
### ブロックストレージタイプリスト表示
```
GET /v2/{tenantId}/types
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| tokenId | Header | String | O | トークンID |

#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| volume_types | Body | Array | ブロックストレージタイプオブジェクトリスト |
| volume_types.id | Body | UUID | ブロックストレージタイプID |
| volume_types.name | Body | String | ブロックストレージタイプ名 |
| volume_types.os-volume-type-access:is_public | Body | Boolean | ブロックストレージタイプ公開表示有無 |
| volume_types.description | Body | String | ブロックストレージタイプの説明 |
| volume_types.extra_specs | Body | Object | ブロックストレージタイプ関連追加仕様情報オブジェクト |

<details><summary>例</summary>
<p>

```json
{
  "volume_types": [
    {
      "os-volume-type-access:is_public": true,
      "extra_specs": {
        "volume_backend_name": "ssd_general"
      },
      "id": "4e36aa51-df30-422e-aff1-eba1f3d9612f",
      "name": "General SSD",
      "description": null
    },
    {
      "os-volume-type-access:is_public": true,
      "extra_specs": {
        "volume_backend_name": "hdd_general"
      },
      "id": "6bda35e2-b2b9-497a-8f65-67a73839c856",
      "name": "General HDD",
      "description": null
    }
  ]
}
```

</p>
</details>

---

## ブロックストレージ
### ブロックストレージ状態
ブロックストレージはさまざまな状態があり、状態によって行える動作が決められています。可能な状態リストは次のとおりです。

| 状態名 | 説明                        |
|--|----------------------------|
| `creating` | 作成中の状態                  |
| `available` | ブロックストレージが作成され、接続する準備ができた状態     |
| `attaching`| ブロックストレージがインスタンスに接続中の状態        |
| `detaching`| ブロックストレージが接続解除中の状態           |
| `in-use`| ブロックストレージがインスタンスに接続された状態          |
| `reserved`| 終了したインスタンスのルートブロックストレージ状態        |
| `maintenance`| ブロックストレージが他のホスト機器に移行される状態   |
| `deleting`| ブロックストレージが削除中の状態              |
| `awaiting-transfer`| ブロックストレージが転送待機中の状態           |
| `error`| ブロックストレージ作成時にエラーが発生した状態        |
| `error_deleting`| ブロックストレージ削除時にエラーが発生した状態        |
| `backing-up`| ブロックストレージがバックアップ中の状態              |
| `restoring-backup`| ブロックストレージがバックアップから復旧中の状態        |
| `error_backing-up`| バックアップ中にエラーが発生した状態           |
| `error_restoring`| 復旧中にエラーが発生した状態           |
| `error_extending`| ブロックストレージ拡張中にエラーが発生した状態        |
| `downloading`| ブロックストレージ作成時、指定したイメージをダウンロードしている状態 |
| `uploading`| イメージ作成時、ブロックストレージのイメージをアップロードしている状態 |
| `retyping`| ブロックストレージタイプを変更中の状態            |
| `extending`| ブロックストレージを拡張している状態               |

### ブロックストレージリスト表示
現在テナントに属しているブロックストレージリストを返します。

```
GET /v2/{tenantId}/volumes
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明                                                                                              |
|---|---|---|---|--------------------------------------------------------------------------------------------------|
| tenantId | URL | String | O | テナントID                                                                                           |
| tokenId | Header | String | O | トークンID                                                                                            |
| sort | Query | String | - | ソートの基準になるブロックストレージフィールド名<br>`< key >[: < direction > ]`形式で記述<br>例) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | 返すブロックストレージの個数<br>基本値は1000に設定                                                                 |
| offset | Query | Integer | - | 返されるリストの開始点<br>全体リスト中、offset番目のブロックストレージから返す                                               |
| marker | Query | UUID | - | 返すブロックストレージの直前のブロックストレージID<br>ソート順序に応じて`marker`に指定されたブロックストレージ以降から`limit`分を返す                 |

#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| volumes | Body | Array | ブロックストレージオブジェクトリスト |
| volumes.id | Body | UUID | ブロックストレージID |
| volumes.links | Body | Object | ブロックストレージリソースリンクレファレンスオブジェクト |
| volumes.name | Body | String | ブロックストレージ名 |
| volumes_links  | Body | Object | ページネーション用の情報オブジェクト(次のリストを指すパス)<br>`limit`、`offset`を追加した場合に返す |

<details><summary>例</summary>
<p>

```json
{
  "volumes": [
    {
      "id": "90712f4f-2faa-4e4f-8eb1-9313a8595570",
      "links": [
        {
          "href": "https://kr1-api-block-storage-infrastructure.nhncloudservice.com/v2/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/90712f4f-2faa-4e4f-8eb1-9313a8595570",
          "rel": "self"
        },
        {
          "href": "https://kr1-api-block-storage-infrastructure.nhncloudservice.com/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/90712f4f-2faa-4e4f-8eb1-9313a8595570",
          "rel": "bookmark"
        }
      ],
      "name": null
    }
  ]
}
```

</p>
</details>

---

### ブロックストレージ詳細リスト表示
現在テナントに属しているブロックストレージリストを返します。

```
GET /v2/{tenantId}/volumes/detail
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明                                                                                              |
|---|---|---|---|--------------------------------------------------------------------------------------------------|
| tenantId | URL | String | O | テナントID                                                                                           |
| tokenId | Header | String | O | トークンID                                                                                            |
| sort | Query | String | - | ソートの基準になるブロックストレージフィールド名<br>`< key >[: < direction > ]`形式で記述<br>例) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | 返すブロックストレージの個数<br>基本値は1000に設定                                                                |
| offset | Query | Integer | - | 返されるリストの開始点<br>全体リスト中、offset番目のブロックストレージから返す                                              |
| marker | Query | UUID | - | 返すブロックストレージの直前のブロックストレージID<br>ソート順序に応じて`marker`に指定されたブロックストレージ以降から`limit`分を返す                |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| volumes | Body | Array | ブロックストレージ詳細情報オブジェクトリスト |
| volumes.attachments | Body | Object | ブロックストレージ接続情報オブジェクト |
| volumes.attachments.server_id | Body | UUID | ブロックストレージが接続されたインスタンスID |
| volumes.attachments.attachment_id | Body | UUID | ブロックストレージ接続ID |
| volumes.attachments.volume_id | Body | UUID | ブロックストレージID |
| volumes.attachments.device | Body | String | インスタンス内の機器名 |
| volumes.attachments.id | Body | String | ブロックストレージID |
| volumes.links | Body | Object | ブロックストレージリソースリンクレファレンスオブジェクト |
| volumes.availability_zone | Body | String | ブロックストレージアベイラビリティゾーン |
| volumes.encrypted | Body | Boolean | ブロックストレージの暗号化有無 |
| volumes.os-volume-replication:extended_status | Body | String | ブロックストレージ拡張状態 |
| volumes.volume_type | Body | String | ブロックストレージタイプ名 |
| volumes.snapshot_id | Body | UUID | ブロックストレージ作成時に指定したSnapshot ID |
| volumes.id | Body | UUID | ブロックストレージID |
| volumes.size | Body | Integer | ブロックストレージサイズ(GB)|
| volumes.user_id | Body | String | ブロックストレージのオーナーID |
| volumes.os-vol-tenant-attr:tenant_id | Body | String | テナントID |
| volumes.metadata | Body | Object | ブロックストレージメタデータオブジェクト |
| volumes.status | Body | Enum | ブロックストレージ状態 |
| volumes.description | Body | String | ブロックストレージの説明 |
| volumes.multiattach | Body | Boolean | 多重接続可否<br>`true`の場合、複数のインスタンスに同時に接続できる |
| volumes.source_volid | Body | UUID | ブロックストレージ作成時に指定したVolume ID |
| volumes.consistencygroup_id | Body | UUID | ブロックストレージConsistencyグループID |
| volumes.name | Body | String | ブロックストレージ名 |
| volumes.bootable | Body | String | ブロックストレージ起動可否 |
| volumes.created_at | Body | Datetime | ブロックストレージ作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| volumes.os-volume-replication:driver_data | Body | String | ブロックストレージ複製データ |
| volumes.replication_status | Body | String | ブロックストレージ複製状態 |
| volumes.volumes_links  | Body | Object | ページネーション用の情報オブジェクト(次のリストを指すパス)<br>`limit`、`offset`を追加した場合に返す |
| volumes.nhn_encryption            | Body | Object | ブロックストレージの暗号化情報 |
| volumes.nhn_encryption.skm_key_version | Body | Integer | 暗号化ブロックストレージの作成に使用するSecure Key Managerの対称鍵バージョン |
| volumes.nhn_encryption.skm_key_id | Body | String | 暗号化ブロックストレージの作成に使用するSecure Key Managerの対称鍵ID |

<details><summary>例</summary>
<p>

```json
{
  "volumes": [
    {
      "attachments": [],
      "links": [
        {
          "href": "https://kr1-api-block-storage-infrastructure.nhncloudservice.com/v2/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
          "rel": "self"
        },
        {
          "href": "https://kr1-api-block-storage-infrastructure.nhncloudservice.com/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
          "rel": "bookmark"
        }
      ],
      "availability_zone": "kr-pub-a",
      "encrypted": false,
      "os-volume-replication:extended_status": null,
      "volume_type": "General HDD",
      "snapshot_id": null,
      "id": "17975e9d-1533-40db-bd02-2072cd2ccb7f",
      "size": 50,
      "user_id": "5e6524d826084188ae549815b3e33380",
      "os-vol-tenant-attr:tenant_id": "6cdebe3eb0094910bc41f1d42ebe4cb7",
      "metadata": {},
      "status": "available",
      "description": null,
      "multiattach": false,
      "source_volid": null,
      "consistencygroup_id": null,
      "name": "volume-f4f47065-300e-480e-8c9c-fb7ec985bffb",
      "bootable": "false",
      "created_at": "2018-12-18T05:43:12.000000",
      "os-volume-replication:driver_data": null,
      "replication_status": "disabled"
    }
  ]
}
```

</p>
</details>

---

### ブロックストレージ表示
指定したブロックストレージの詳細情報を返します。

```
GET /v2/{tenantId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| volumeId | URL | UUID | O | ブロックストレージID |
| tokenId | Header | String | O | トークンID |

#### レスポンス

| 名前 | 種類 | 形式 | 説明                                          |
|---|---|---|----------------------------------------------|
| volume | Body | Object | ブロックストレージ詳細情報オブジェクト                                 |
| volume.attachments | Body | Object | ブロックストレージ接続情報オブジェクト                                 |
| volume.attachments.server_id | Body | UUID | ブロックストレージが接続されたインスタンスID                              |
| volume.attachments.attachment_id | Body | UUID | ブロックストレージ接続ID                                     |
| volume.attachments.volume_id | Body | UUID | ブロックストレージID                                        |
| volume.attachments.device | Body | String | インスタンス内の機器名                                |
| volume.attachments.id | Body | String | ブロックストレージID                                        |
| volume.links | Body | Object | ブロックストレージリソースリンクリファレンスオブジェクト                             |
| volume.availability_zone | Body | String | ブロックストレージアベイラビリティゾーン                                   |
| volume.encrypted | Body | Boolean | ブロックストレージの暗号化有無                                    |
| volume.os-volume-replication:extended_status | Body | String | ブロックストレージ拡張状態                                    |
| volume.volume_type | Body | String | ブロックストレージタイプ名                                    |
| volume.snapshot_id | Body | UUID | ブロックストレージ作成時に指定したスナップショットID                           |
| volume.id | Body | UUID | ブロックストレージID                                        |
| volume.size | Body | Integer | ブロックストレージサイズ(GB)                                    |
| volume.user_id | Body | String | ブロックストレージのオーナーID                                    |
| volume.os-vol-tenant-attr:tenant_id | Body | String | テナントID                                       |
| volume.metadata | Body | Object | ブロックストレージメタデータオブジェクト                                 |
| volume.status | Body | Enum | ブロックストレージの状態                                       |
| volume.description | Body | String | ブロックストレージの説明                                       |
| volume.multiattach | Body | Boolean | 多重接続可否<br>`true`の場合、複数のインスタンスに同時に接続できる |
| volume.source_volid | Body | UUID | ブロックストレージ作成時に指定したブロックストレージID                            |
| volume.consistencygroup_id | Body | UUID | ブロックストレージConsistencyグループID                          |
| volume.name | Body | String | ブロックストレージ名                                       |
| volume.bootable | Body | String | ブロックストレージ起動可否                                 |
| volume.created_at | Body | Datetime | ブロックストレージ作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`     |
| volume.os-volume-replication:driver_data | Body | String | ブロックストレージ複製データ                                   |
| volume.replication_status | Body | String | ブロックストレージ複製状態                                    |
| volume.nhn_encryption            | Body | Object | ブロックストレージの暗号化情報 |
| volume.nhn_encryption.skm_key_version | Body | Integer | 暗号化ブロックストレージの作成に使用するSecure Key Managerの対称鍵バージョン |
| volume.nhn_encryption.skm_key_id | Body | String | 暗号化ブロックストレージの作成に使用するSecure Key Managerの対称鍵ID |


<details><summary>例</summary>
<p>

```json
{
  "volume": {
    "attachments": [],
    "links": [
      {
        "href": "https://kr1-api-block-storage-infrastructure.nhncloudservice.com/v2/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
        "rel": "self"
      },
      {
        "href": "https://kr1-api-block-storage-infrastructure.nhncloudservice.com/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
        "rel": "bookmark"
      }
    ],
    "availability_zone": "kr-pub-a",
    "encrypted": false,
    "os-volume-replication:extended_status": null,
    "volume_type": "General HDD",
    "snapshot_id": null,
    "id": "17975e9d-1533-40db-bd02-2072cd2ccb7f",
    "size": 50,
    "user_id": "5e6524d826084188ae549815b3e33380",
    "os-vol-tenant-attr:tenant_id": "6cdebe3eb0094910bc41f1d42ebe4cb7",
    "metadata": {},
    "status": "available",
    "description": null,
    "multiattach": false,
    "source_volid": null,
    "consistencygroup_id": null,
    "name": "volume-f4f47065-300e-480e-8c9c-fb7ec985bffb",
    "bootable": "false",
    "created_at": "2018-12-18T05:43:12.000000",
    "os-volume-replication:driver_data": null,
    "replication_status": "disabled"
  }
}
```

</p>
</details>

---

### ブロックストレージを作成する
スナップショットから新しいブロックストレージを作成したり、空のブロックストレージを作成します。

ブロックストレージは、作成直後は使用できません。ブロックストレージ状態を照会して`available`状態に変わったことを確認してから使用します。

```
POST /v2/{tenantId}/volumes
X-Auth-Token: {tokenId}
```

#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明                       |
|---|---|---|---|---------------------------|
| tenantId | URL | String | O | テナントID                    |
| tokenId | Header | String | O | トークンID                     |
| volume | Body | Object | O | ブロックストレージ作成リクエストオブジェクト              |
| volume.size | Body | Integer | O | ブロックストレージサイズ(GB)                 |
| volume.description | Body | String | - | ブロックストレージの説明                    |
| volume.availability_zone | Body | String | - | ブロックストレージアベイラビリティゾーン名             |
| volume.name | Body | String | - | ブロックストレージ名                    |
| volume.volume_type | Body | String | - | ブロックストレージタイプ名                 |
| volume.snapshot_id | Body | UUID | - | 原本スナップショットID。省略すると空のブロックストレージが作成される。 |
| volume.metadata | Body | Object | - | ブロックストレージメタデータオブジェクト              |
| volume.nhn_encryption            | Body | Object | - | ブロックストレージ暗号化情報 |
| volume.nhn_encryption.skm_appkey | Body | String | - | Secure Key Manager商品のアプリケーションキー |
| volume.nhn_encryption.skm_key_id | Body | String | - | 暗号化ブロックストレージの作成に使用するSecure Key Managerの対称鍵ID |

<details><summary>例</summary>
<p>

```json
{
    "volume": {
        "size": 10,
        "availability_zone": null,
        "source_volid": null,
        "description": null,
        "snapshot_id": null,
        "name": null,
        "volume_type": null,
        "metadata": {}
    }
}
```

</p>
</details>

#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| volume | Body | Object | ブロックストレージ詳細情報オブジェクト |
| volume.attachments | Body | Object | ブロックストレージ接続情報オブジェクト |
| volume.links | Body | Object | ブロックストレージリソースリンクリファレンスオブジェクト |
| volume.availability_zone | Body | String | ブロックストレージアベイラビリティゾーン |
| volume.encrypted | Body | Boolean | ブロックストレージの暗号化有無 |
| volume.os-volume-replication:extended_status | Body | String | ブロックストレージ拡張状態 |
| volume.volume_type | Body | String | ブロックストレージタイプ名 |
| volume.snapshot_id | Body | UUID | ブロックストレージ作成時に指定したSnapshot ID |
| volumes.id | Body | UUID | ブロックストレージID |
| volume.size | Body | Integer | ブロックストレージサイズ(GB) |
| volume.user_id | Body | String | ブロックストレージのオーナーID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | テナントID |
| volume.metadata | Body | Object | ブロックストレージメタデータオブジェクト |
| volume.status | Body | Enum | ブロックストレージの状態 |
| volume.description | Body | String | ブロックストレージの説明 |
| volume.multiattach | Body | Boolean | 複数のインスタンスへの接続可否 |
| volume.name | Body | String | ブロックストレージ名 |
| volume.bootable | Body | String | ブロックストレージ起動可否 |
| volume.created_at | Body | Datetime | ブロックストレージ作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| volume.os-volume-replication:driver_data | Body | String | ブロックストレージ複製データ |
| volume.replication_status | Body | String | ブロックストレージ複製状態 |
| volume.nhn_encryption            | Body | Object | ブロックストレージの暗号化情報 |
| volume.nhn_encryption.skm_key_version | Body | Integer | 暗号化ブロックストレージの作成に使用するSecure Key Managerの対称鍵バージョン |
| volume.nhn_encryption.skm_key_id | Body | String | 暗号化ブロックストレージの作成に使用するSecure Key Managerの対称鍵ID |

<details><summary>例</summary>
<p>

```json
{
  "volume": {
    "status": "creating",
    "user_id": "94acd5b4d2bf47dda734e34a113f96ff",
    "attachments": [],
    "links": [{
      "href": "https://kr1-api-block-storage-infrastructure.nhncloudservice.com/v2/c0e5e63026e449e6b7e94c779021d150/volumes/87882cf4-ca05-4ef2-b598-b93b2caf041e",
      "rel": "self"
    }, {
      "href": "https://kr1-api-block-storage-infrastructure.nhncloudservice.com/c0e5e63026e449e6b7e94c779021d150/volumes/87882cf4-ca05-4ef2-b598-b93b2caf041e",
      "rel": "bookmark"
    }],
    "availability_zone": "kr-pub-a",
    "bootable": "false",
    "encrypted": false,
    "created_at": "2020-03-03T10:54:33.163206",
    "description": null,
    "volume_type": "General HDD",
    "name": "DATA",
    "replication_status": "disabled",
    "consistencygroup_id": null,
    "source_volid": null,
    "snapshot_id": null,
    "multiattach": false,
    "metadata": {},
    "id": "87882cf4-ca05-4ef2-b598-b93b2caf041e",
    "size": 10
  }
}
```

</p>
</details>

---

### ブロックストレージを削除する

指定したブロックストレージを削除します。接続されていたり、スナップショットが作成されたブロックストレージは削除できません。

```
DELETE /v2/{tenantId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| volumeId | URL | String | O | ブロックストレージID |
| tokenId | Header | String | O | トークンID |

#### レスポンス
このAPIはレスポンス本文を返しません。

---

### ブロックストレージでイメージを作成する
ブロックストレージからイメージを作成します。 

イメージ作成後、基本的な初期化作業のために100KBの空き容量が必要です。空き容量がそれ以下の場合、初期化作業に失敗する場合があります。

> [注意]
> 作成されたイメージのサイズは、ルートブロックストレージの実際の使用量より大きい場合があります。

```
POST /v2/{tenantId}/volumes/{volumeId}/action
X-Auth-Token: {tokenId}
```

#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| volumeId | URL | UUID | O | ブロックストレージID |
| tokenId | Header | String | O | トークンID |
| os-volume_upload_image | Body | Object | O | ブロックストレージイメージ作成リクエストオブジェクト |
| os-volume_upload_image.image_name | Body | String | O | イメージ名 |
| os-volume_upload_image.force | Body | Boolean | - | インスタンスに接続されたブロックストレージの場合、イメージ作成を許可するかどうか<br>デフォルト値はfalse |
| os-volume_upload_image.disk_format | Body | String | - | イメージディスクフォーマット |
| os-volume_upload_image.container_format | Body | String | - | イメージコンテナフォーマット |
| os-volume_upload_image.visibility | Body | String | - | イメージの可視性<br>`private`または`shared` |
| os-volume_upload_image.protected | Body | Boolean | - | イメージ保護</br>protected=trueの場合、修正および削除不可 |

<details><summary>例</summary>
<p>

```json
{
	"os-volume_upload_image":{
        "image_name": "VOLUME IMAGE",
        "force": true,
        "disk_format": "qcow2",
        "container_format": "bare",
        "visibility": "private",
        "protected": false
    }
}
```

</p>
</details>

#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| os-volume_upload_image | Body | Object | ブロックストレージイメージ作成レスポンスオブジェクト |
| os-volume_upload_image.status | Body | String | ブロックストレージの状態 |
| os-volume_upload_image.image_name | Body | String | イメージ名 |
| os-volume_upload_image.disk_format | Body | String | イメージディスクフォーマット |
| os-volume_upload_image.container_format | Body | String | イメージコンテナフォーマット |
| os-volume_upload_image.updated_at | Body | Datetime | イメージ修正時刻 |
| os-volume_upload_image.image_id | Body | UUID | イメージID |
| os-volume_upload_image.display_description | Body | String | ブロックストレージの説明 |
| os-volume_upload_image.id | Body | UUID | ブロックストレージID |
| os-volume_upload_image.size | Body | Integer | ブロックストレージサイズ(GB) |
| os-volume_upload_image.volume_type | Body | Object | ブロックストレージタイプ情報オブジェクト |

<details><summary>例</summary>
<p>

```json
{
    "os-volume_upload_image": {
        "status": "uploading",
        "image_name": "public api test2",
        "disk_format": "qcow2",
        "container_format": "bare",
        "updated_at": "2020-05-18T04:21:15.000000",
        "image_id": "01956bf6-5609-4b43-88ea-1be866114368",
        "id": "d16d64e8-a5c9-47fe-a559-1119778c739c",
        "size": 20,
        "volume_type": {
            "name": "General HDD",
            "qos_specs_id": "ec4ef37d-9273-4e6f-a495-bd43b0f2d0f2",
            "deleted": false,
            "deleted_at": "null",
            "created_at": "2019-10-10T06:34:33.000000",
            "updated_at": "2019-10-10T06:37:52.000000",
            "extra_specs": [
                {
                    "volume_type_id": "964a6c6b-7190-4e27-9311-cce8d6f860f3",
                    "deleted": false,
                    "created_at": "2019-10-10T06:39:35.000000",
                    "updated_at": "null",
                    "deleted_at": "null",
                    "value": "hdd_general",
                    "key": "volume_backend_name",
                    "id": 1
                }
            ],
            "is_public": true,
            "id": "964a6c6b-7190-4e27-9311-cce8d6f860f3",
            "description": "null"
        }
    }
}
```

</p>
</details>

---

## スナップショット
### スナップショット状態
スナップショットはさまざまな状態があり、状態によって行える動作が決められています。可能な状態リストは次のとおりです。

| 状態名 | 説明                     |
|--|-------------------------|
| `creating` | 作成中の状態               |
| `available` | スナップショットが作成され、使用する準備ができた状態 |
| `backing-up`| スナップショットがバックアップ中の状態          |
| `deleting`| スナップショットが削除中の状態          |
| `error`| 作成中にエラーが発生した状態        |
| `deleted`| 削除された状態                 |
| `unmanaging`| スナップショットの管理モードが解除中の状態 |
| `restoring`| スナップショットからブロックストレージを復元中の状態   |
| `error_deleting`| 削除中にエラーが発生した状態        |

### スナップショットのリスト表示
スナップショットのリストを返します。

```
GET /v2/{tenantId}/snapshots
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| tokenId | Header | String | O | トークンID |

#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| snapshots | Body | Array | スナップショット情報オブジェクトリスト |
| snapshots.status | Body | Enum | スナップショットの状態 |
| snapshots.description | Body | String | スナップショットの説明 |
| snapshots.created_at | Body | Datetime | スナップショットの作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| snapshots.metadata | Body | Object | スナップショットメタデータオブジェクト |
| snapshots.volume_id | Body | UUID | スナップショットの原本ブロックストレージID |
| snapshots.size | Body | Integer | スナップショットの原本ブロックストレージサイズ(GB) |
| snapshots.id | Body | UUID | スナップショットID |
| snapshots.name | Body | String | スナップショットの名前 |

<details><summary>例</summary>
<p>

```json
{
  "snapshots": [
    {
      "status": "available",
      "description": "",
      "created_at": "2020-03-03T11:03:40.000000",
      "metadata": {},
      "volume_id": "17975e9d-1533-40db-bd02-2072cd2ccb7f",
      "size": 50,
      "id": "f63af601-43cd-41ab-8905-e1e93995f366",
      "name": "SNAPSHOT"
    }
  ]
}
```

</p>
</details>

---

### スナップショットリスト詳細表示
スナップショット詳細情報リストを返します。

```
GET /v2/{tenantId}/snapshots/detail
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| tokenId | Header | String | O | トークンID |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| snapshots | Body | Array | スナップショット詳細情報オブジェクトリスト |
| snapshots.status | Body | Enum | スナップショットの状態 |
| snapshots.description | Body | String | スナップショットの説明 |
| snapshots.os-extended-snapshot-attributes:progress | Body | String | スナップショット作成進行状態 |
| snapshots.created_at | Body | Datetime | スナップショットの作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| snapshots.metadata | Body | Object | スナップショットメタデータオブジェクト |
| snapshots.volume_id | Body | UUID | スナップショットの原本ブロックストレージID |
| snapshots.os-extended-snapshot-attributes:project_id | Body | String | テナントID |
| snapshots.size | Body | Integer | スナップショットの原本ブロックストレージサイズ(GB) |
| snapshots.id | Body | UUID | スナップショットID |
| snapshots.name | Body | String | スナップショットの名前 |

<details><summary>例</summary>
<p>

```json
{
  "snapshots": [
    {
      "status": "available",
      "os-extended-snapshot-attributes:progress": "100%",
      "description": "",
      "created_at": "2020-03-03T11:03:40.000000",
      "metadata": {},
      "volume_id": "17975e9d-1533-40db-bd02-2072cd2ccb7f",
      "os-extended-snapshot-attributes:project_id": "6cdebe3eb0094910bc41f1d42ebe4cb7",
      "size": 50,
      "id": "f63af601-43cd-41ab-8905-e1e93995f366",
      "name": "SNAPSHOT"
    }
  ]
}
```

</p>
</details>

---

### スナップショット表示
指定したスナップショットの詳細情報を返します。

```
GET /v2/{tenantId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| snapshotId | URL | UUID | O | スナップショットID |
| tokenId | Header | String | O | トークンID |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| snapshot | Body | Object | スナップショット詳細情報オブジェクト |
| snapshot.status | Body | Enum | スナップショットの状態 |
| snapshot.description | Body | String | スナップショットの説明 |
| snapshot.os-extended-snapshot-attributes:progress | Body | String | スナップショットの作成進行状態 |
| snapshot.created_at | Body | Datetime | スナップショットの作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| snapshot.metadata | Body | Object | スナップショットメタデータオブジェクト |
| snapshot.volume_id | Body | UUID | スナップショットの原本ブロックストレージID |
| snapshot.os-extended-snapshot-attributes:project_id | Body | String | テナントID |
| snapshot.size | Body | Integer | スナップショットの原本ブロックストレージサイズ(GB) |
| snapshot.id | Body | UUID | スナップショットID |
| snapshot.name | Body | String | スナップショットの名前 |

<details><summary>例</summary>
<p>

```json
{
  "snapshot": {
    "status": "available",
    "os-extended-snapshot-attributes:progress": "100%",
    "description": "",
    "created_at": "2020-03-03T11:03:40.000000",
    "metadata": {},
    "volume_id": "17975e9d-1533-40db-bd02-2072cd2ccb7f",
    "os-extended-snapshot-attributes:project_id": "6cdebe3eb0094910bc41f1d42ebe4cb7",
    "size": 50,
    "id": "f63af601-43cd-41ab-8905-e1e93995f366",
    "name": "SNAPSHOT"
  }
}
```

</p>
</details>

---

### スナップショットを作成する
指定したブロックストレージのスナップショットを作成します。

```
POST /v2/{tenantId}/snapshots/
X-Auth-Token: {tokenId}
```

#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明                                       |
|---|---|---|---|-------------------------------------------|
| tenantId | URL | String | O | テナントID                                    |
| tokenId | Header | String | O | トークンID                                     |
| snapshot | Body | Object | O | スナップショット作成リクエストオブジェクト                             |
| snapshot.volume_id | Body | UUID | O | 原本ブロックストレージID                                  |
| snapshot.force | Body | Boolean | - | 強制的にスナップショットを作成するかどうか<br>`true`の場合、ブロックストレージが接続されていてもスナップショットを作成 |
| snapshot.description | Body | String | - | スナップショットの説明                                   |
| snapshot.name | Body | String | - | スナップショットの名前                                   |

<details><summary>例</summary>
<p>

```json
{
    "snapshot": {
        "name": "SNAPSHOT-001",
        "description": "Daily backup",
        "volume_id": "5aa119a8-d25b-45a7-8d1b-88e127885635",
        "force": true
    }
}
```

</p>
</details>

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| snapshot | Body | Object | スナップショット詳細情報オブジェクト |
| snapshot.status | Body | Enum | スナップショットの状態 |
| snapshot.description | Body | String | スナップショットの説明 |
| snapshot.created_at | Body | Datetime | スナップショットの作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| snapshot.metadata | Body | Object | スナップショットメタデータオブジェクト |
| snapshot.volume_id | Body | UUID | スナップショットの原本ブロックストレージID |
| snapshot.size | Body | Integer | スナップショットの原本ブロックストレージサイズ(GB) |
| snapshot.id | Body | UUID | スナップショットID |
| snapshot.name | Body | String | スナップショットの名前 |

<details><summary>例</summary>
<p>

```json
{
  "snapshot": {
    "status": "creating",
    "description": null,
    "created_at": "2020-03-02T13:51:33.318695",
    "metadata": {},
    "volume_id": "5aa119a8-d25b-45a7-8d1b-88e127885635",
    "size": 20,
    "id": "f2ae2667-b3b7-47ef-aa31-efab536888b9",
    "name": "SNAPSHOT"
  }
}
```

</p>
</details>

---

### スナップショットを削除する
指定したスナップショットを削除します。

```
DELETE /v2/{tenantId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| snapshotId | URL | String | O | スナップショットID |
| tokenId | Header | String | O | トークンID |

#### レスポンス
このAPIはレスポンス本文を返しません。
