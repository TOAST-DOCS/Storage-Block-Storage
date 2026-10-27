<!-- machine_translated: true -->

<!-- pre-align:aligned sig=2d1ea01d20b8 -->

<a id="storage-block-storage-api-v3-guide"></a>

## Storage > Block Storage > API v3ガイド { #storage-block-storage-api-v3-guide }

Block StorageはAPI呼び出し時の認証/認可にIaaSトークンを使用します。IaaSトークンは、NHN CloudのOpenStackベースのインフラサービス(IaaS)で使用する認証トークンです。IaaSトークンの発行および使用については、[IaaSトークン](/nhncloud/ja/public-api/iaas-token)を参照してください。

ブロックストレージAPIは`volumev3`タイプのエンドポイントを使用します。正確なエンドポイントは、トークン発行レスポンスの`serviceCatalog`を参照してください。

| タイプ | リージョン | エンドポイント |
|---|---|---|
| volumev3 | 韓国(パンギョ)リージョン<br>韓国(ピョンチョン)リージョン<br>韓国(光州)リージョン<br>日本リージョン | https://kr1-api-block-storage-infrastructure.nhncloudservice.com<br>https://kr2-api-block-storage-infrastructure.nhncloudservice.com<br>https://kr3-api-block-storage-infrastructure.nhncloudservice.com<br>https://jp1-api-block-storage-infrastructure.nhncloudservice.com |

APIレスポンスには、このガイドに記載されていないフィールドが含まれる場合があります。これらのフィールドはNHN Cloud内部で使用されており、事前告知なく変更される可能性があるため、使用しないでください。

<a id="block-storage-type"></a>

## ブロックストレージタイプ { #block-storage-type }

<a id="list-block-storage-types"></a>
### ブロックストレージタイプ一覧の表示 { #list-block-storage-types }
```
GET /v3/{tenantId}/types
X-Auth-Token: {tokenId}
```

<a id="request"></a>
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| tokenId | Header | String | O | トークンID |

<a id="response"></a>
#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| volume_types | Body | Array | ブロックストレージタイプオブジェクトの一覧 |
| volume_types.id | Body | UUID | ブロックストレージタイプID |
| volume_types.name | Body | String | ブロックストレージタイプ名 |
| volume_types.os-volume-type-access:is_public | Body | Boolean | ブロックストレージタイプが公開かどうか |
| volume_types.description | Body | String | ブロックストレージタイプの説明 |
| volume_types.extra_specs | Body | Object | ブロックストレージタイプに関連する追加仕様情報オブジェクト |

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

<a id="block-storage"></a>

## ブロックストレージ { #block-storage }

<a id="block-storage-status"></a>

### ブロックストレージ状態 { #block-storage-status }
ブロックストレージはさまざまな状態を持ち、状態に応じて実行できる操作が決まっています。使用可能な状態の一覧は次のとおりです。

| 状態名 | 説明                         |
|--|----------------------------|
| `creating` | 作成中の状態                   |
| `available` | ブロックストレージが作成され、接続の準備ができている状態      |
| `attaching`| ブロックストレージがインスタンスに接続中の状態         |
| `detaching`| ブロックストレージが接続解除中の状態            |
| `in-use`| ブロックストレージがインスタンスに接続された状態           |
| `reserved`| 終了したインスタンスのルートブロックストレージの状態         |
| `maintenance`| ブロックストレージが別のホスト機器に移行中の状態    |
| `deleting`| ブロックストレージを削除中の状態               |
| `awaiting-transfer`| ブロックストレージが転送を待機している状態            |
| `error`| ブロックストレージの作成時にエラーが発生した状態         |
| `error_deleting`| ブロックストレージの削除時にエラーが発生した状態         |
| `backing-up`| ブロックストレージがバックアップ中の状態               |
| `restoring-backup`| ブロックストレージがバックアップから復元中の状態         |
| `error_backing-up`| バックアップ中にエラーが発生した状態            |
| `error_restoring`| 復元中にエラーが発生した状態            |
| `error_extending`| ブロックストレージの拡張中にエラーが発生した状態         |
| `downloading`| ブロックストレージの作成時に指定したイメージをダウンロードしている状態 |
| `uploading`| イメージの作成時にブロックストレージのイメージをアップロードしている状態 |
| `retyping`| ブロックストレージタイプを変更している状態             |
| `extending`| ブロックストレージを拡張している状態                |

<a id="list-block-storage"></a>

### ブロックストレージ一覧の表示 { #list-block-storage }
現在のテナントに属するブロックストレージの一覧を返します。

```
GET /v3/{tenantId}/volumes
X-Auth-Token: {tokenId}
```

<a id="list-block-storage-request"></a>
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明                                                                                               |
|---|---|---|---|--------------------------------------------------------------------------------------------------|
| tenantId | URL | String | O | テナントID                                                                                           |
| tokenId | Header | String | O | トークンID                                                                                            |
| sort | Query | String | - | ソートの基準となるブロックストレージのフィールド名<br>`< key >[: < direction > ]`の形式で指定<br>例) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | 返すブロックストレージの数<br>デフォルト値は1000                                                                 |
| offset | Query | Integer | - | 返す一覧の開始点<br>全体の一覧のうち、offset番目のブロックストレージから返します                                               |
| marker | Query | UUID | - | 返すブロックストレージの直前のブロックストレージID<br>ソート順に従い、`marker`で指定されたブロックストレージ以降から`limit`件を返します                 |

<a id="list-block-storage-response"></a>
#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| volumes | Body | Array | ブロックストレージオブジェクトの一覧 |
| volumes.id | Body | UUID | ブロックストレージID |
| volumes.links | Body | Object | ブロックストレージのリソースリンクリファレンスオブジェクト |
| volumes.name | Body | String | ブロックストレージ名 |
| volumes_links  | Body | Object | ページネーションのための情報オブジェクト(次の一覧を指すパス)<br>`limit`または`offset`を追加した場合に返されます |

<details><summary>例</summary>
<p>

```json
{
  "volumes": [
    {
      "id": "90712f4f-2faa-4e4f-8eb1-9313a8595570",
      "links": [
        {
          "href": "https://kr1-api-block-storage-infrastructure.nhncloudservice.com/v3/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/90712f4f-2faa-4e4f-8eb1-9313a8595570",
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

<a id="list-block-storage-details"></a>

### ブロックストレージ詳細リストの表示 { #list-block-storage-details }
現在のテナントに属するブロックストレージのリストを返します。

```
GET /v3/{tenantId}/volumes/detail
X-Auth-Token: {tokenId}
```

<a id="list-block-storage-details-request"></a>
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明                                                                                               |
|---|---|---|---|--------------------------------------------------------------------------------------------------|
| tenantId | URL | String | O | テナントID                                                                                           |
| tokenId | Header | String | O | トークンID                                                                                            |
| sort | Query | String | - | ソート基準となるブロックストレージのフィールド名<br>`< key >[: < direction > ]`の形式で記述<br>例) `name:asc`、`created_at:desc` |
| limit | Query | Integer | - | 返すブロックストレージの数<br>デフォルト値は1000                                                                 |
| offset | Query | Integer | - | 返すリストの開始点<br/>全体リストのうちオフセット(offset)番目のブロックストレージから返します                                              |
| marker | Query | UUID | - | 返すブロックストレージの直前のブロックストレージID<br/>ソート順に従い、`marker`で指定されたブロックストレージ以降から`limit`分を返します                |

<a id="list-block-storage-details-response"></a>
#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| volumes | Body | Array | ブロックストレージ詳細情報オブジェクトリスト |
| volumes.attachments | Body | Object | ブロックストレージ接続情報オブジェクト |
| volumes.attachments.server_id | Body | UUID | ブロックストレージが接続されているインスタンスID |
| volumes.attachments.attachment_id | Body | UUID | ブロックストレージ接続ID |
| volumes.attachments.volume_id | Body | UUID | ブロックストレージID |
| volumes.attachments.device | Body | String | インスタンス内のデバイス名 |
| volumes.attachments.id | Body | String | ブロックストレージID |
| volumes.links | Body | Object | ブロックストレージリソースリンクリファレンスオブジェクト |
| volumes.availability_zone | Body | String | ブロックストレージのアベイラビリティゾーン |
| volumes.encrypted | Body | Boolean | ブロックストレージの暗号化かどうか |
| volumes.os-volume-replication:extended_status | Body | String | ブロックストレージの拡張状態 |
| volumes.volume_type | Body | String | ブロックストレージのタイプ名 |
| volumes.snapshot_id | Body | UUID | ブロックストレージの作成時に指定したスナップショットID |
| volumes.id | Body | UUID | ブロックストレージID |
| volumes.size | Body | Integer | ブロックストレージのサイズ(GB) |
| volumes.user_id | Body | String | ブロックストレージのオーナーID |
| volumes.os-vol-tenant-attr:tenant_id | Body | String | テナントID |
| volumes.metadata | Body | Object | ブロックストレージのメタデータオブジェクト |
| volumes.status | Body | Enum | ブロックストレージの状態 |
| volumes.description | Body | String | ブロックストレージの説明 |
| volumes.multiattach | Body | Boolean | 多重接続可否<br>`true`の場合、複数のインスタンスに同時に接続できます |
| volumes.source_volid | Body | UUID | ブロックストレージの作成時に指定したブロックストレージID |
| volumes.consistencygroup_id | Body | UUID | ブロックストレージグループID |
| volumes.name | Body | String | ブロックストレージの名前 |
| volumes.bootable | Body | String | ブロックストレージの起動可否 |
| volumes.created_at | Body | Datetime | ブロックストレージの作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| volumes.os-volume-replication:driver_data | Body | String | ブロックストレージの複製データ |
| volumes.replication_status | Body | String | ブロックストレージの複製ステータス |
| volumes.volumes_links  | Body | Object | ページネーションのための情報オブジェクト(次のリストへのパス)<br>`limit`、`offset`を追加した場合に返されます |
| volumes.nhn_encryption            | Body | Object | ブロックストレージの暗号化情報 |
| volumes.nhn_encryption.skm_key_version | Body | Integer | 暗号化ブロックストレージの作成に使用するSecure Key Managerの対称キーバージョン |
| volumes.nhn_encryption.skm_key_id | Body | String | 暗号化ブロックストレージの作成に使用するSecure Key Managerの対称キーID |

<details><summary>例</summary>
<p>

```json
{
  "volumes": [
    {
      "attachments": [],
      "links": [
        {
          "href": "https://kr1-api-block-storage-infrastructure.nhncloudservice.com/v3/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
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

<a id="get-block-storage"></a>

### ブロックストレージの表示 { #get-block-storage }
指定したブロックストレージの詳細情報を返します。

```
GET /v3/{tenantId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

<a id="get-block-storage-request"></a>
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| volumeId | URL | UUID | O | ブロックストレージID |
| tokenId | Header | String | O | トークンID |

<a id="get-block-storage-response"></a>
#### レスポンス

| 名前 | 種類 | 形式 | 説明                                           |
|---|---|---|----------------------------------------------|
| volume | Body | Object | ブロックストレージ詳細情報オブジェクト                                  |
| volume.attachments | Body | Object | ブロックストレージ接続情報オブジェクト                                  |
| volume.attachments.server_id | Body | UUID | ブロックストレージが接続されているインスタンスID                              |
| volume.attachments.attachment_id | Body | UUID | ブロックストレージ接続ID                                     |
| volume.attachments.volume_id | Body | UUID | ブロックストレージID                                        |
| volume.attachments.device | Body | String | インスタンス内のデバイス名                                 |
| volume.attachments.id | Body | String | ブロックストレージID                                        |
| volume.links | Body | Object | ブロックストレージリソースリンクリファレンスオブジェクト                              |
| volume.availability_zone | Body | String | ブロックストレージアベイラビリティゾーン                                    |
| volume.encrypted | Body | Boolean | ブロックストレージ暗号化可否                                    |
| volume.os-volume-replication:extended_status | Body | String | ブロックストレージ拡張状態                                     |
| volume.volume_type | Body | String | ブロックストレージタイプ名                                     |
| volume.snapshot_id | Body | UUID | ブロックストレージ作成時に指定したスナップショットID                           |
| volume.id | Body | UUID | ブロックストレージID                                        |
| volume.size | Body | Integer | ブロックストレージサイズ(GB)                                    |
| volume.user_id | Body | String | ブロックストレージオーナーID                                    |
| volume.os-vol-tenant-attr:tenant_id | Body | String | テナントID                                       |
| volume.metadata | Body | Object | ブロックストレージメタデータオブジェクト                                  |
| volume.status | Body | Enum | ブロックストレージ状態                                        |
| volume.description | Body | String | ブロックストレージ説明                                        |
| volume.multiattach | Body | Boolean | 多重接続可否<br>`true`の場合、複数のインスタンスに同時に接続できます |
| volume.source_volid | Body | UUID | ブロックストレージ作成時に指定したブロックストレージID                            |
| volume.consistencygroup_id | Body | UUID | ブロックストレージコンシステンシー(一貫性)グループID                          |
| volume.name | Body | String | ブロックストレージ名前                                        |
| volume.bootable | Body | String | ブロックストレージ起動可否                                  |
| volume.created_at | Body | Datetime | ブロックストレージ作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`     |
| volume.os-volume-replication:driver_data | Body | String | ブロックストレージ複製データ                                    |
| volume.replication_status | Body | String | ブロックストレージ複製ステータス                                     |
| volume.nhn_encryption            | Body | Object | ブロックストレージ暗号化情報 |
| volume.nhn_encryption.skm_key_version | Body | Integer | 暗号化ブロックストレージ作成に使用するSecure Key Managerの対称キーバージョン |
| volume.nhn_encryption.skm_key_id | Body | String | 暗号化ブロックストレージ作成に使用するSecure Key Managerの対称キーID |

<details><summary>例</summary>
<p>

```json
{
  "volume": {
    "attachments": [],
    "links": [
      {
        "href": "https://kr1-api-block-storage-infrastructure.nhncloudservice.com/v3/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
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

<a id="create-block-storage"></a>

### ブロックストレージの作成 { #create-block-storage }
スナップショットから新しいブロックストレージを作成するか、空のブロックストレージを作成します。

ブロックストレージは作成直後すぐに使用できません。ブロックストレージの状態を照会し、`available`状態であることを確認してから使用します。

```
POST /v3/{tenantId}/volumes
X-Auth-Token: {tokenId}
```

<a id="create-block-storage-request"></a>
#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明                        |
|---|---|---|---|---------------------------|
| tenantId | URL | String | O | テナントID                    |
| tokenId | Header | String | O | トークンID                     |
| volume | Body | Object | O | ブロックストレージ作成リクエストオブジェクト               |
| volume.size | Body | Integer | O | ブロックストレージのサイズ(GB)                 |
| volume.description | Body | String | - | ブロックストレージの説明                     |
| volume.availability_zone | Body | String | - | ブロックストレージのアベイラビリティゾーン名              |
| volume.name | Body | String | - | ブロックストレージの名前                     |
| volume.volume_type | Body | String | - | ブロックストレージのタイプ名                  |
| volume.snapshot_id | Body | UUID | - | 元のスナップショットID。省略すると空のブロックストレージが作成されます |
| volume.metadata | Body | Object | - | ブロックストレージのメタデータオブジェクト               |
| volume.nhn_encryption            | Body | Object | - | ブロックストレージの暗号化情報 |
| volume.nhn_encryption.skm_appkey | Body | String | - | Secure Key Managerサービスのアプリキー |
| volume.nhn_encryption.skm_key_id | Body | String | - | 暗号化ブロックストレージの作成に使用するSecure Key Managerの対称キーID |

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

<a id="create-block-storage-response"></a>
#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| volume | Body | Object | ブロックストレージ詳細情報オブジェクト |
| volume.attachments | Body | Object | ブロックストレージの接続情報オブジェクト |
| volume.links | Body | Object | ブロックストレージのリソースリンクリファレンスオブジェクト |
| volume.availability_zone | Body | String | ブロックストレージのアベイラビリティゾーン |
| volume.encrypted | Body | Boolean | ブロックストレージを暗号化するかどうか |
| volume.os-volume-replication:extended_status | Body | String | ブロックストレージの拡張状態 |
| volume.volume_type | Body | String | ブロックストレージのタイプ名 |
| volume.snapshot_id | Body | UUID | ブロックストレージ作成時に指定したスナップショットID |
| volumes.id | Body | UUID | ブロックストレージID |
| volume.size | Body | Integer | ブロックストレージのサイズ(GB) |
| volume.user_id | Body | String | ブロックストレージのオーナーID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | テナントID |
| volume.metadata | Body | Object | ブロックストレージのメタデータオブジェクト |
| volume.status | Body | Enum | ブロックストレージの状態 |
| volume.description | Body | String | ブロックストレージの説明 |
| volume.multiattach | Body | Boolean | 複数のインスタンスに接続できるかどうか |
| volume.name | Body | String | ブロックストレージの名前 |
| volume.bootable | Body | String | ブロックストレージの起動可否 |
| volume.created_at | Body | Datetime | ブロックストレージの作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| volume.os-volume-replication:driver_data | Body | String | ブロックストレージの複製データ |
| volume.replication_status | Body | String | ブロックストレージの複製ステータス |
| volume.nhn_encryption            | Body | Object | ブロックストレージの暗号化情報 |
| volume.nhn_encryption.skm_key_version | Body | Integer | 暗号化ブロックストレージの作成に使用するSecure Key Managerの対称キーバージョン |
| volume.nhn_encryption.skm_key_id | Body | String | 暗号化ブロックストレージの作成に使用するSecure Key Managerの対称キーID |

<details><summary>例</summary>
<p>

```json
{
  "volume": {
    "status": "creating",
    "user_id": "94acd5b4d2bf47dda734e34a113f96ff",
    "attachments": [],
    "links": [{
      "href": "https://kr1-api-block-storage-infrastructure.nhncloudservice.com/v3/c0e5e63026e449e6b7e94c779021d150/volumes/87882cf4-ca05-4ef2-b598-b93b2caf041e",
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

<a id="delete-block-storage"></a>

### ブロックストレージの削除 { #delete-block-storage }

指定したブロックストレージを削除します。接続中またはスナップショットが作成されているブロックストレージは削除できません。

```
DELETE /v3/{tenantId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

<a id="delete-block-storage-request"></a>
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| volumeId | URL | String | O | ブロックストレージID |
| tokenId | Header | String | O | トークンID |

<a id="delete-block-storage-response"></a>
#### レスポンス
このAPIはレスポンス本文を返しません。

---

<a id="create-image-with-block-storage"></a>

### ブロックストレージからイメージを作成する { #create-image-with-block-storage }
ブロックストレージからイメージを作成します。

イメージ作成後、基本的な初期化作業のために最低100KBの空き容量が必要です。残り容量がこれより小さい場合、初期化作業が失敗する可能性があります。

> [注意]
> 作成されたイメージのサイズは、ルートブロックストレージの実際の使用量より大きくなる場合があります。

```
POST /v3/{tenantId}/volumes/{volumeId}/action
X-Auth-Token: {tokenId}
```

<a id="create-image-with-block-storage-request"></a>
#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| volumeId | URL | UUID | O | ブロックストレージID |
| tokenId | Header | String | O | トークンID |
| os-volume_upload_image | Body | Object | O | ブロックストレージイメージ作成リクエストオブジェクト |
| os-volume_upload_image.image_name | Body | String | O | イメージ名 |
| os-volume_upload_image.force | Body | Boolean | - | インスタンスに接続されたブロックストレージの場合にイメージ作成を許可するかどうか<br>デフォルトはfalse |
| os-volume_upload_image.disk_format | Body | String | - | イメージディスクフォーマット |
| os-volume_upload_image.container_format | Body | String | - | イメージコンテナフォーマット |
| os-volume_upload_image.visibility | Body | String | - | イメージの可視性<br>`private`、`shared`のいずれか |
| os-volume_upload_image.protected | Body | Boolean | - | イメージを保護するかどうか</br>protected=trueの場合、変更および削除ができません |

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

<a id="create-image-with-block-storage-response"></a>
#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| os-volume_upload_image | Body | Object | ブロックストレージイメージ作成レスポンスオブジェクト |
| os-volume_upload_image.status | Body | String | ブロックストレージの状態 |
| os-volume_upload_image.image_name | Body | String | イメージ名 |
| os-volume_upload_image.disk_format | Body | String | イメージディスクフォーマット |
| os-volume_upload_image.container_format | Body | String | イメージコンテナフォーマット |
| os-volume_upload_image.updated_at | Body | Datetime | イメージ更新日時 |
| os-volume_upload_image.image_id | Body | UUID | イメージID |
| os-volume_upload_image.display_description | Body | String | ブロックストレージの説明 |
| os-volume_upload_image.id | Body | UUID | ブロックストレージID |
| os-volume_upload_image.size | Body | Integer | ブロックストレージのサイズ(GB) |
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

<a id="snapshot"></a>

## スナップショット { #snapshot }

<a id="snapshot-status"></a>
### スナップショットの状態 { #snapshot-status }
スナップショットはさまざまな状態を持ち、状態に応じて実行できる操作が決まっています。可能な状態の一覧は次のとおりです。

| 状態名 | 説明                      |
|--|-------------------------|
| `creating` | 作成中の状態                |
| `available` | スナップショットが作成され、使用する準備ができた状態  |
| `backing-up`| スナップショットがバックアップ中の状態           |
| `deleting`| スナップショットが削除中の状態           |
| `error`| 作成中にエラーが発生した状態         |
| `deleted`| 削除された状態                  |
| `unmanaging`| スナップショットの管理モードが解除中の状態 |
| `restoring`| スナップショットからブロックストレージを復元中の状態    |
| `error_deleting`| 削除中にエラーが発生した状態         |

<a id="list-snapshots"></a>
### スナップショット一覧の表示 { #list-snapshots }
スナップショットの一覧を返します。

```
GET /v3/{tenantId}/snapshots
X-Auth-Token: {tokenId}
```

<a id="list-snapshots-request"></a>
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| tokenId | Header | String | O | トークンID |

<a id="list-snapshots-response"></a>
#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| snapshots | Body | Array | スナップショット情報オブジェクトの一覧 |
| snapshots.status | Body | Enum | スナップショットの状態 |
| snapshots.description | Body | String | スナップショットの説明 |
| snapshots.created_at | Body | Datetime | スナップショットの作成時刻<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| snapshots.metadata | Body | Object | スナップショットのメタデータオブジェクト |
| snapshots.volume_id | Body | UUID | スナップショットの元のブロックストレージID |
| snapshots.size | Body | Integer | スナップショットの元のブロックストレージサイズ(GB) |
| snapshots.id | Body | UUID | スナップショットID |
| snapshots.name | Body | String | スナップショット名 |

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

<a id="list-snapshots-with-details"></a>
### スナップショット一覧の詳細表示 { #list-snapshots-with-details }
スナップショットの詳細情報の一覧を返します。

```
GET /v3/{tenantId}/snapshots/detail
X-Auth-Token: {tokenId}
```

<a id="list-snapshots-with-details-request"></a>
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| tokenId | Header | String | O | トークンID |

<a id="list-snapshots-with-details-response"></a>
#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| snapshots | Body | Array | スナップショット詳細情報オブジェクトの一覧 |
| snapshots.status | Body | Enum | スナップショットの状態 |
| snapshots.description | Body | String | スナップショットの説明 |
| snapshots.os-extended-snapshot-attributes:progress | Body | String | スナップショット作成の進行状態 |
| snapshots.created_at | Body | Datetime | スナップショットの作成時刻<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| snapshots.metadata | Body | Object | スナップショットのメタデータオブジェクト |
| snapshots.volume_id | Body | UUID | スナップショットの元のブロックストレージID |
| snapshots.os-extended-snapshot-attributes:project_id | Body | String | テナントID |
| snapshots.size | Body | Integer | スナップショットの元のブロックストレージサイズ(GB) |
| snapshots.id | Body | UUID | スナップショットID |
| snapshots.name | Body | String | スナップショット名 |

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

<a id="view-snapshot"></a>
### スナップショットの表示 { #view-snapshot }
指定したスナップショットの詳細情報を返します。

```
GET /v3/{tenantId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

<a id="view-snapshot-request"></a>
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| snapshotId | URL | UUID | O | スナップショットID |
| tokenId | Header | String | O | トークンID |

<a id="view-snapshot-response"></a>
#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| snapshot | Body | Object | スナップショット詳細情報オブジェクト |
| snapshot.status | Body | Enum | スナップショットの状態 |
| snapshot.description | Body | String | スナップショットの説明 |
| snapshot.os-extended-snapshot-attributes:progress | Body | String | スナップショット作成の進行状態 |
| snapshot.created_at | Body | Datetime | スナップショットの作成時刻<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| snapshot.metadata | Body | Object | スナップショットのメタデータオブジェクト |
| snapshot.volume_id | Body | UUID | スナップショットの元のブロックストレージID |
| snapshot.os-extended-snapshot-attributes:project_id | Body | String | テナントID |
| snapshot.size | Body | Integer | スナップショットの元のブロックストレージサイズ(GB) |
| snapshot.id | Body | UUID | スナップショットID |
| snapshot.name | Body | String | スナップショット名 |

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

<a id="create-snapshot"></a>
### スナップショットの作成 { #create-snapshot }
指定したブロックストレージのスナップショットを作成します。

```
POST /v3/{tenantId}/snapshots
X-Auth-Token: {tokenId}
```

<a id="create-snapshot-request"></a>
#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明                                        |
|---|---|---|---|-------------------------------------------|
| tenantId | URL | String | O | テナントID                                    |
| tokenId | Header | String | O | トークンID                                     |
| snapshot | Body | Object | O | スナップショット作成リクエストオブジェクト                              |
| snapshot.volume_id | Body | UUID | O | 元のブロックストレージID                                  |
| snapshot.force | Body | Boolean | - | 強制スナップショット作成かどうか<br>`true`の場合、ブロックストレージが接続されていてもスナップショットを作成します |
| snapshot.description | Body | String | - | スナップショットの説明                                    |
| snapshot.name | Body | String | - | スナップショット名                                    |

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

<a id="create-snapshot-response"></a>
#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| snapshot | Body | Object | スナップショット詳細情報オブジェクト |
| snapshot.status | Body | Enum | スナップショットの状態 |
| snapshot.description | Body | String | スナップショットの説明 |
| snapshot.created_at | Body | Datetime | スナップショットの作成時刻<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| snapshot.metadata | Body | Object | スナップショットのメタデータオブジェクト |
| snapshot.volume_id | Body | UUID | スナップショットの元のブロックストレージID |
| snapshot.size | Body | Integer | スナップショットの元のブロックストレージサイズ(GB) |
| snapshot.id | Body | UUID | スナップショットID |
| snapshot.name | Body | String | スナップショット名 |

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

<a id="delete-snapshots"></a>
### スナップショットの削除 { #delete-snapshots }
指定したスナップショットを削除します。

```
DELETE /v3/{tenantId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

<a id="delete-snapshots-request"></a>
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | URL | String | O | テナントID |
| snapshotId | URL | String | O | スナップショットID |
| tokenId | Header | String | O | トークンID |

<a id="delete-snapshots-response"></a>
#### レスポンス
このAPIはレスポンス本文を返しません。