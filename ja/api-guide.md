## Storage > Block Storage > APIガイド

## 事前準備

ブロックストレージAPIを使用するには、アプリケーションキー(Appkey)とトークンが必要です。 [API Endpoint URL](/Compute/Instance/ja/api-guide/#api-endpoint-url)と[トークンAPI](/Compute/Instance/ja/api-guide/#api)を利用してアプリケーションキーとトークンを準備します。アプリケーションキーはAPI Endpoint URLに、トークンはRequest Headerに含めて使用します。

例えば、ブロックストレージの情報の照会は次のURLでリクエストする必要があります。

	GET https://api-compute.cloud.toast.com/compute/v1.0/appkeys/{appkey}/volumes?id={volumeId}


## ブロックストレージAPI

ブロックストレージの作成、削除、照会機能を提供します。ブロックストレージをインスタンスに接続、解除する機能は[インスタンス追加機能API](/Compute/Instance/ja/api-guide/#_8)で提供されます。

### ブロックストレージの状態

ブロックストレージは次の状態値を持ちます。

| Status | Description |
| --- | --- |
| creating | 作成中 |
| available | インスタンスに接続可能な状態 |
| attaching | インスタンスに接続中 |
| detaching | インスタンスから接続解除中 |
| in-use | インスタンスに接続され、使用中の状態 |
| deleting | 削除中 |
| error | 作成中にエラー発生 |
| error_deleting | 削除中にエラー発生 |
| backing-up | バックアップ進行中 |
| restoring-backup | バックアップ復旧中 |
| error_backing-up | バックアップ進行中にエラー発生 |
| error_restoring | バックアップ復旧中にエラー発生 |
| downloading | イメージダウンロード中 |
| uploading | イメージにアップロード中 |

### ブロックストレージの情報の照会

ブロックストレージの情報を照会します。

#### Method、 URL
```
GET /v1.0/appkeys/{appkey}/volumes?id={volumeId}
X-Auth-Token: {tokenId}
```

|  Name | In | Type | Optional | Description |
|--|--|--|--|--|
| tokenId | Header | String | - | トークンID |
| volumeId | Query | String | O | 照会するブロックストレージID。なければ全てのブロックストレージの情報を照会します。 |

#### Request Body
このAPIはRequest Bodyが必要ありません。

#### Response Body
```json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "SUCCESS"
    },
    "volumes": [
        {
            "attachments": [
                {
                    "device": "{Device Name}",
                    "instanceId": "{Attached Instance ID}",
                    "attachmentId": "{Attachment ID}"
                }
            ],
            "availabilityZone": "{Availability Zone Name}",
            "createdAt": "{Created At}",
            "description": "{Description}",
            "id": "{Block Storage ID}",
            "metadata": {
                "{Metadata Key}": "{Metadata Value}"
            },
            "name": "{Block Storage Name}",
            "size": "{Size}",
            "status": "{Status}",
            "volumeType": "{Volume Type}"
        }
    ]
}
```

|  Name | In | Type | Description |
|--|--|--|--|
| Device Name | Body | String  | インスタンスに接続されている場合、インスタンスでのデバイス名。 ex) "/dev/vdb" |
| Attached Instance ID | Body | String | インスタンスに接続されている場合、接続されたインスタンスのID |
| Attachment ID | Body | String | インスタンスに接続されている場合、接続ID |
| Availability Zone Name | Body | String | ブロックストレージが位置するアベイラビリティーゾーンの名前 |
| Created At | Body | String | ブロックストレージ作成時間。 yyyy-mm-ddTHH:MM:ssZの形式。例) 2017-05-16T02:17:50.166563 |
| Description | Body | String | ブロックストレージの説明 |
| Block Storage ID | Body | String | ブロックストレージのID |
| Metadata Key / Value | Body | Boolean | ブロックストレージに記載されたメタデータ |
| Block Storage Name | Body | String | ブロックストレージの名前 |
| Size | Body | Integer | ブロックストレージのサイズ(GB) |
| Status | Body | String | ブロックストレージの状態 |
| Volume Type | Body | String | ブロックストレージの種類。 "General HDD"または"General SSD"のどちらか |

### ブロックストレージの作成
新しいブロックストレージを作成します。

#### Method、 URL
```
POST /v1.0/appkeys/{appkey}/volumes
X-Auth-Token: {tokenId}
Content-Type: application/json;charset=UTF-8
```

|  Name | In | Type | Optional | Description |
|--|--|--|--|--|
| tokenId | Header | String | - | トークンID |

#### Request Body
```json
{
    "volume": {
        "description": "{Description}",
        "availabilityZone":"{Availability Zone Name}",
        "size":"{Size}",
        "name":"{Block Storage Name}",
        "volumeType":"{Volume Type}",
        "metadata": {
        	"{Metadata Key}" : "{Metadata Value}"
        }
    }
}
```

| Name | In | Type | Optional | Description |
| --- | --- | --- | --- | --- |
| Description | Body | String | O | ブロックストレージの説明 |
| Availability Zone Name | Body | String | - | ブロックストレージを作成するアベイラビリティーゾーンの名前 |
| Size | Body | Integer | - | ブロックストレージのサイズ(GB)。 10～1000の範囲、 10単位で入力 |
| Volume Type | Body | String | O | 作成するブロックストレージの種類。 "General HDD"または"General SSD"のどちらか。 |
| Metadata Key / Metadata Value | Body | String | O | ブロックストレージに記入しようとするメタデータ情報 |
| Block Storage Name | Body | String | - | ブロックストレージの名前 |

#### Response Body
```json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "SUCCESS"
    },
    "volume": {
        "attachments": [],
        "availabilityZone": "{Availability Zone Name}",
        "createdAt": "{Created At}",
        "description": "{Description}",
        "id": "{Block Storage ID}",
        "metadata": {
            "{Metadata Key}": "{Metadata Value}"
        },
        "name": "{Block Storage Name}",
        "size": "{Size}",
        "status": "{Status}",
        "volumeType": "{Volume Type}"
    }
}
```

|  Name | In | Type | Description |
|--|--|--|--|
| Availability Zone Name | Body | String | ブロックストレージが位置するアベイラビリティーゾーンの名前 |
| Created At | Body | String | ブロックストレージの作成時間。 yyyy-mm-ddTHH:MM:ssZの形式。例) 2017-05-16T02:17:50.166563 |
| Description | Body | String | ブロックストレージの説明 |
| Block Storage ID | Body | String | ブロックストレージのID |
| Metadata Key / Value | Body | Boolean | ブロックストレージに記載されたメタデータ |
| Block Storage Name | Body | String | ブロックストレージの名前 |
| Size | Body | Integer | ブロックストレージのサイズ(GB) |
| Status | Body | String | ブロックストレージの状態 |
| Volume Type | Body | String | ブロックストレージの種類。 "General HDD"または"General SSD"のどちらか。 |

### ブロックストレージの削除
ブロックストレージを削除します。 Statusが"available" "in-use" "error" "error_restoring"のブロックストレージのみ削除できます。

#### Method、 URL
```
DELETE /v1.0/appkeys/{appkey}/volumes?id={volumeId}
X-Auth-Token: {tokenId}
```
|  Name | In | Type | Optional | Description |
|--|--|--|--|--|
| tokenId | Header | String | - | トークンID |
| volumeId | Query | String | - | 削除するブロックストレージのID |

#### Request Body
このAPIはRequest Bodyが必要ありません。

#### Response Body
```json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "SUCCESS"
    }
}
```
