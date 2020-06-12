## Storage > Block Storage > Public APIガイド

## ボリュームタイプ
### ボリュームタイプリスト表示
```
GET /v2/{projectid}/types
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| projectId | URL | String | O | テナントID |
| tokenId | Header | String | O | トークンID |

#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| volume_types | Body | Array | ボリュームタイプオブジェクトリスト |
| volume_types.id | Body | UUID | ボリュームタイプID |
| volume_types.name | Body | String | ボリュームタイプ名 |
| volume_types.os-volume-type-access:is_public | Body | Boolean | ボリュームタイプ公開表示有無 |
| volume_types.description | Body | String | ボリュームタイプの説明 |
| volume_types.extra_specs | Body | Object | ボリュームタイプ関連追加仕様情報オブジェクト |

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

## ボリューム
### ボリューム状態
ボリュームはさまざまな状態があり、状態によって行える動作が決められています。可能な状態リストは次のとおりです。

| 状態名 | 説明 |
|--|--|
| `creating` | 作成中の状態 |
| `available` | ボリュームが作成され、接続する準備ができた状態 |
| `attaching`| ボリュームがインスタンスに接続中の状態 |
| `detaching`| ボリュームが接続解除中の状態 |
| `in-use`| ボリュームがインスタンスに接続された状態 |
| `maintenance`| ボリュームが他のホスト機器に移行される状態 |
| `deleting`| ボリュームが削除中の状態 |
| `awaiting-transfer`| ボリュームが転送待機中の状態 |
| `error`| ボリューム作成時にエラーが発生した状態 |
| `error_deleting`| ボリューム削除時にエラーが発生した状態 |
| `backing-up`| ボリュームがバックアップ中の状態 |
| `restoring-backup`| ボリュームがバックアップから復旧中の状態 |
| `error_backing-up`| バックアップ中にエラーが発生した状態 |
| `error_restoring`| 復旧中にエラーが発生した状態 |
| `error_extending`| ボリューム拡張中にエラーが発生した状態 |
| `downloading`| ボリューム作成時、指定したイメージをダウンロードしている状態 |
| `uploading`| イメージ作成時、ボリュームのイメージをアップロードしている状態 |
| `retyping`| ボリュームタイプを変更中の状態 |
| `extending`| ボリュームを拡張している状態 |

### ボリュームリスト表示
現在テナントに属しているボリュームリストを返します。

```
GET /v2/{projectId}/volumes
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| projectId | URL | String | O | テナントID |
| tokenId | Header | String | O | トークンID |
| sort | Query | String | - | ソートの基準になるボリュームフィールド名<br>`< key >[: < direction > ]`形式で記述<br>例) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | 返すボリュームの個数<br>基本値は1000に設定 |
| offset | Query | Integer | - | 返されるリストの開始点<br>全体リスト中、offset番目のボリュームから返す |
| marker | Query | UUID | - | 返すボリュームの直前のボリュームID<br>ソート順序に応じて`marker`に指定されたボリューム以降から`limit`分を返す |

#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| volumes | Body | Array | ボリュームオブジェクトリスト |
| volumes.id | Body | UUID | ボリュームID |
| volumes.links | Body | Object | ボリュームリソースリンクレファレンスオブジェクト |
| volumes.name | Body | String | ボリューム名 |
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
          "href": "https://kr1-api-block-storage.infrastructure.cloud.toast.com/v2/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/90712f4f-2faa-4e4f-8eb1-9313a8595570",
          "rel": "self"
        },
        {
          "href": "https://kr1-api-block-storage.infrastructure.cloud.toast.com/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/90712f4f-2faa-4e4f-8eb1-9313a8595570",
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

### ボリューム詳細リスト表示
現在テナントに属しているボリュームリストを返します。

```
GET /v2/{projectId}/volumes/detail
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| projectId | URL | String | O | テナントID |
| tokenId | Header | String | O | トークンID |
| sort | Query | String | - | ソートの基準になるボリュームフィールド名<br>`< key >[: < direction > ]`形式で記述<br>例) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | 返すボリュームの個数<br>基本値は1000に設定 |
| offset | Query | Integer | - | 返されるリストの開始点<br>全体リスト中、offset番目のボリュームから返す |
| marker | Query | UUID | - | 返すボリュームの直前のボリュームID<br>ソート順序に応じて`marker`に指定されたボリューム以降から`limit`分を返す |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| volumes | Body | Array | ボリューム詳細情報オブジェクトリスト |
| volumes.attachments | Body | Object | ボリューム接続情報オブジェクト |
| volumes.attachments.server_id | Body | UUID | ボリュームが接続されたインスタンスID |
| volumes.attachments.attachment_id | Body | UUID | ボリューム接続ID |
| volumes.attachments.volume_id | Body | UUID | ボリュームID |
| volumes.attachments.device | Body | String | インスタンス内の機器名 |
| volumes.attachments.id | Body | String | ボリュームID |
| volumes.links | Body | Object | ボリュームリソースリンクレファレンスオブジェクト |
| volumes.availability_zone | Body | String | ボリュームアベイラビリティゾーン |
| volumes.encrypted | Body | Boolean | ボリュームの暗号化有無 |
| volumes.os-volume-replication:extended_status | Body | String | ボリューム拡張状態 |
| volumes.volume_type | Body | String | ボリュームタイプ名 |
| volumes.snapshot_id | Body | UUID | ボリューム作成時に指定したSnapshot ID |
| volumes.id | Body | UUID | ボリュームID |
| volumes.size | Body | Integer | ボリュームサイズ(GB)|
| volumes.user_id | Body | String | ボリュームのオーナーID |
| volumes.os-vol-tenant-attr:tenant_id | Body | String | テナントID |
| volumes.metadata | Body | Object | ボリュームメタデータオブジェクト |
| volumes.status | Body | Enum | ボリューム状態 |
| volumes.description | Body | String | ボリュームの説明 |
| volumes.multiattach | Body | Boolean | 多重接続可否<br>`true`の場合、複数のインスタンスに同時に接続できる |
| volumes.source_volid | Body | UUID | ボリューム作成時に指定したVolume ID |
| volumes.consistencygroup_id | Body | UUID | ボリュームConsistencyグループID |
| volumes.name | Body | String | ボリューム名 |
| volumes.bootable | Body | Boolean | ボリューム起動可否 |
| volumes.created_at | Body | Datetime | ボリューム作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| volumes.os-volume-replication:driver_data | Body | String | ボリューム複製データ |
| volumes.replication_status | Body | String | ボリューム複製状態 |
| volumes.volumes_links  | Body | Object | ページネーション用の情報オブジェクト(次のリストを指すパス)<br>`limit`、`offset`を追加した場合に返す |

<details><summary>例</summary>
<p>

```json
{
  "volumes": [
    {
      "attachments": [],
      "links": [
        {
          "href": "https://kr1-api-block-storage.infrastructure.cloud.toast.com/v2/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
          "rel": "self"
        },
        {
          "href": "https://kr1-api-block-storage.infrastructure.cloud.toast.com/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
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

### ボリューム表示
指定したボリュームの詳細情報を返します。

```
GET /v2/{projectId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| projectId | URL | String | O | テナントID |
| volumeId | URL | UUID | O | ボリュームID |
| tokenId | Header | String | O | トークンID |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| volume | Body | Object | ボリューム詳細情報オブジェクト |
| volume.attachments | Body | Object | ボリューム接続情報オブジェクト |
| volume.attachments.server_id | Body | UUID | ボリュームが接続されたインスタンスID |
| volume.attachments.attachment_id | Body | UUID | ボリューム接続ID |
| volume.attachments.volume_id | Body | UUID | ボリュームID |
| volume.attachments.device | Body | String | インスタンス内の機器名 |
| volume.attachments.id | Body | String | ボリュームID |
| volume.links | Body | Object | ボリュームリソースリンクリファレンスオブジェクト |
| volume.availability_zone | Body | String | ボリュームアベイラビリティゾーン |
| volume.encrypted | Body | Boolean | ボリュームの暗号化有無 |
| volume.os-volume-replication:extended_status | Body | String | ボリューム拡張状態 |
| volume.volume_type | Body | String | ボリュームタイプ名 |
| volume.snapshot_id | Body | UUID | ボリューム作成時に指定したSnapshot ID |
| volume.id | Body | UUID | ボリュームID |
| volume.size | Body | Integer | ボリュームサイズ(GB) |
| volume.user_id | Body | String | ボリュームのオーナーID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | テナントID |
| volume.metadata | Body | Object | ボリュームメタデータオブジェクト |
| volume.status | Body | Enum | ボリュームの状態 |
| volume.description | Body | String | ボリュームの説明 |
| volume.multiattach | Body | Boolean | 多重接続可否<br>`true`の場合、複数のインスタンスに同時に接続できる |
| volume.source_volid | Body | UUID | ボリューム作成時に指定したVolume ID |
| volume.consistencygroup_id | Body | UUID | ボリュームConsistencyグループID |
| volume.name | Body | String | ボリューム名 |
| volume.bootable | Body | Boolean | ボリューム起動可否 |
| volume.created_at | Body | Datetime | ボリューム作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` |
| volume.os-volume-replication:driver_data | Body | String | ボリューム複製データ |
| volume.replication_status | Body | String | ボリューム複製状態 |

<details><summary>例</summary>
<p>

```json
{
  "volume": {
    "attachments": [],
    "links": [
      {
        "href": "https://kr1-api-block-storage.infrastructure.cloud.toast.com/v2/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
        "rel": "self"
      },
      {
        "href": "https://kr1-api-block-storage.infrastructure.cloud.toast.com/6cdebe3eb0094910bc41f1d42ebe4cb7/volumes/17975e9d-1533-40db-bd02-2072cd2ccb7f",
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

### ボリュームを作成する
スナップショットから新しいボリュームを作成したり、空のボリュームを作成します。

ボリュームは、作成直後は使用できません。ボリューム状態を照会して`available`状態に変わったことを確認してから使用します。

```
POST /v2/{projectId}/volumes
X-Auth-Token: {tokenId}
```

#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| projectId | URL | String | O | テナントID |
| tokenId | Header | String | O | トークンID |
| volume | Body | Object | O | ボリューム作成リクエストオブジェクト |
| volume.size | Body | Integer | O | ボリュームサイズ(GB) |
| volume.description | Body | String | - | ボリュームの説明 |
| volume.multiattach | Body | Boolean | - | 多重接続可否<br>`true`に設定すると<br>複数のインスタンスに同時に接続できる |
| volume.availability_zone | Body | String | - | ボリュームアベイラビリティゾーン名 |
| volume.name | Body | String | - | ボリューム名 |
| volume.volume_type | Body | String | - | ボリュームタイプ名 |
| volume.snapshot_id | Body | UUID | - | 原本スナップショットID。省略すると空のボリュームが作成される。 |
| volume.metadata | Body | Object | - | ボリュームメタデータオブジェクト |

<details><summary>例</summary>
<p>

```json
{
    "volume": {
        "size": 10,
        "availability_zone": null,
        "source_volid": null,
        "description": null,
        "multiattach": false,
        "snapshot_id": null,
        "name": null,
        "imageRef": null,
        "volume_type": null,
        "metadata": {},
        "consistencygroup_id": null
    }
}
```

</p>
</details>

#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| volume | Body | Object | ボリューム詳細情報オブジェクト |
| volume.attachments | Body | Object | ボリューム接続情報オブジェクト |
| volume.links | Body | Object | ボリュームリソースリンクリファレンスオブジェクト |
| volume.availability_zone | Body | String | ボリュームアベイラビリティゾーン |
| volume.encrypted | Body | Boolean | ボリュームの暗号化有無 |
| volume.os-volume-replication:extended_status | Body | String | ボリューム拡張状態 |
| volume.volume_type | Body | String | ボリュームタイプ名 |
| volume.snapshot_id | Body | UUID | ボリューム作成時に指定したSnapshot ID |
| volumes.id | Body | UUID | ボリュームID |
| volume.size | Body | Integer | ボリュームサイズ(GB) |
| volume.user_id | Body | String | ボリュームのオーナーID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | テナントID |
| volume.metadata | Body | Object | ボリュームメタデータオブジェクト |
| volume.status | Body | Enum | ボリュームの状態 |
| volume.description | Body | String | ボリュームの説明 |
| volume.multiattach | Body | Boolean | 複数のインスタンスへの接続可否 |
| volume.consistencygroup_id | Body | UUID | ボリュームConsistencyグループID |
| volume.name | Body | String | ボリューム名 |
| volume.bootable | Body | Boolean | ボリューム起動可否 |
| volume.created_at | Body | Datetime | ボリューム作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| volume.os-volume-replication:driver_data | Body | String | ボリューム複製データ |
| volume.replication_status | Body | String | ボリューム複製状態 |

<details><summary>例</summary>
<p>

```json
{
  "volume": {
    "status": "creating",
    "user_id": "94acd5b4d2bf47dda734e34a113f96ff",
    "attachments": [],
    "links": [{
      "href": "https://kr1-api-block-storage.infrastructure.cloud.toast.com/v2/c0e5e63026e449e6b7e94c779021d150/volumes/87882cf4-ca05-4ef2-b598-b93b2caf041e",
      "rel": "self"
    }, {
      "href": "https://kr1-api-block-storage.infrastructure.cloud.toast.com/c0e5e63026e449e6b7e94c779021d150/volumes/87882cf4-ca05-4ef2-b598-b93b2caf041e",
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

### ボリュームを削除する

指定したボリュームを削除します。接続されていたり、スナップショットが作成されたボリュームは削除できません。

```
DELETE /v2/{projectId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| projectId | URL | String | O | テナントID |
| volumeId | URL | String | O | ボリュームID |
| tokenId | Header | String | O | トークンID |

#### レスポンス
このAPIはレスポンス本文を返しません。

---

## スナップショット
### スナップショット状態
スナップショットはさまざまな状態があり、状態によって行える動作が決められています。可能な状態リストは次のとおりです。

| 状態名 | 説明 |
|--|--|
| `creating` | 作成中の状態 |
| `available` | スナップショットが作成され、使用する準備ができた状態 |
| `backing-up`| スナップショットがバックアップ中の状態 |
| `deleting`| スナップショットが削除中の状態 |
| `error`| 作成中にエラーが発生した状態 |
| `deleted`| 削除された状態 |
| `unmanaging`| スナップショットの管理モードが解除中の状態 |
| `restoring`| スナップショットからボリュームを復元中の状態 |
| `error_deleting`| 削除中にエラーが発生した状態 |

### スナップショットのリスト表示
スナップショットのリストを返します。

```
GET /v2/{projectId}/snapshots
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| projectId | URL | String | O | テナントID |
| tokenId | Header | String | O | トークンID |

#### レスポンス

| 名前 | 種類 | プロパティ | 説明 |
|---|---|---|---|
| snapshots | Body | Array | スナップショット情報オブジェクトリスト |
| snapshots.status | Body | Enum | スナップショットの状態 |
| snapshots.description | Body | String | スナップショットの説明 |
| snapshots.created_at | Body | Datetime | スナップショットの作成日時<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`の形式 |
| snapshots.metadata | Body | Object | スナップショットメタデータオブジェクト |
| snapshots.volume_id | Body | UUID | スナップショットの原本ボリュームID |
| snapshots.size | Body | Integer | スナップショットの原本ボリュームサイズ(GB) |
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
GET /v2/{projectId}/snapshots/detail
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| projectId | URL | String | O | テナントID |
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
| snapshots.volume_id | Body | UUID | スナップショットの原本ボリュームID |
| snapshots.os-extended-snapshot-attributes:project_id | Body | String | テナントID |
| snapshots.size | Body | Integer | スナップショットの原本ボリュームサイズ(GB) |
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
GET /v2/{projectId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| projectId | URL | String | O | テナントID |
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
| snapshot.volume_id | Body | UUID | スナップショットの原本ボリュームID |
| snapshot.os-extended-snapshot-attributes:project_id | Body | String | テナントID |
| snapshot.size | Body | Integer | スナップショットの原本ボリュームサイズ(GB) |
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
指定したボリュームのスナップショットを作成します。

```
POST /v2/{projectId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| projectId | URL | String | O | テナントID |
| tokenId | Header | String | O | トークンID |
| snapshot | Body | Object | O | スナップショット作成リクエストオブジェクト |
| snapshot.volume_id | Body | UUID | O | 原本ボリュームID |
| snapshot.force | Body | Boolean | - | 強制的にスナップショットを作成するかどうか<br>`true`の場合、ボリュームが接続されていてもスナップショットを作成 |
| snapshot.description | Body | String | - | スナップショットの説明 |
| snapshot.name | Body | String | - | スナップショットの名前 |

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
| snapshot.volume_id | Body | UUID | スナップショットの原本ボリュームID |
| snapshot.size | Body | Integer | スナップショットの原本ボリュームサイズ(GB) |
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
DELETE /v2/{projectId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| projectId | URL | String | O | テナントID |
| snapshotId | URL | String | O | スナップショットID |
| tokenId | Header | String | O | トークンID |

#### レスポンス
このAPIはレスポンス本文を返しません。
