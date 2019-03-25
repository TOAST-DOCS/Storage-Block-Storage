## Storage > Block Storage > API指南

## 事前准备

如果要使用块存储（block storage ）API，则需要App key与令牌（token）。利用[API Endpoint URL](/Compute/Instance/zh/api-guide/#api-endpoint-url)和 [令牌 API](/Compute/Instance/zh/api-guide/#api)来准备App key和令牌。API Endpoint URL中包含App key， Request Body中包含令牌。

例如，想查询块存储信息，应通过以下URL来进行查询。

	GET https://api-compute.cloud.toast.com/compute/v1.0/appkeys/{appkey}/volumes?id={volumeId}


## 块存储API

提供块存储的创建、删除、查询功能。同时，通过 [实例添加功能 API](/Compute/Instance/zh/api-guide/#_8)，提供将块存储设备挂载到实例和从实例上卸载的功能。

### 块存储状态

块存储具备如下状态值。

| Status | Description |
| --- | --- |
| creating | 创建中 | 
| available | 可用的状态 |
| attaching | 挂载中 |
| detaching | 卸载中 | 
| in-use | 使用中 |
| deleting | 删除中 |
| error | 创建过程中出现错误 |
| error_deleting | 删除过程中出现错误 |
| backing-up | 备份中 |
| restoring-backup | 备份恢复中 |
| error_backing-up | 备份过程中出现错误 |
| error_restoring | 备份恢复过程中出现错误 |
| downloading | 镜像下载中 |
| uploading | 镜像上传中 |

### 块存储信息查询

查询块存储信息。

#### Method, URL
```
GET /v1.0/appkeys/{appkey}/volumes?id={volumeId}
X-Auth-Token: {tokenId}
```

|  Name | In | Type | Optional | Description |
|--|--|--|--|--|
| tokenId | Header | String | - | 令牌ID |
| volumeId | Query | String | O | 要查询的块存储ID.如果没有将查询所有块存储信息。|

#### Request Body
该 API无需Request Body。

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
            "status": "{Status}"
        }
    ]
}
```

|  Name | In | Type | Description |
|--|--|--|--|
| Device Name | Body | String  | 使用中的情况，实例上挂载的设备名称. ex) "/dev/vdb" | 
| Attached Instance ID | Body | String | 使用中的情况，已挂载的实例ID | 
| Attachment ID | Body | String | 使用中的情况，连接ID |
| Availability Zone Name | Body | String | 块存储所在可用区的名称 |
| Created At | Body | String | 块存储创建时间. yyyy-mm-ddTHH:MM:ssZ的格式. 例) 2017-05-16T02:17:50.166563 |
| Description | Body | String | 块存储描述 |
| Block Storage ID | Body | String | 块存储ID |
| Metadata Key / Value | Body | Boolean | 块存储中的元数据 |
| Block Storage Name | Body | String | 块存储名称 |
| Size | Body | Integer | 块存储大小(GB) |
| Status | Body | String | 块存储状态 |

### 创建块存储
创建新的块存储。

#### Method, URL
```
POST /v1.0/appkeys/{appkey}/volumes
X-Auth-Token: {tokenId}
Content-Type: application/json;charset=UTF-8
```

|  Name | In | Type | Optional | Description |
|--|--|--|--|--|
| tokenId | Header | String | - | 令牌ID |

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
| Description | Body | String | O | 块存储描述 |
| Availability Zone Name | Body | String | - | 块存储所在可用区的名称 |
| Size | Body | Integer | - | 块存储大小(GB). 10~1000 范围, 以10整数倍为单位的方式输入 |
| Volume Type | Body | String | - | 要创建的块存储的类型，目前无法另行提供类型，因此应以空字符串方式设置。 |
| Metadata Key / Metadata Value | Body | String | O | 要写入块存储的元数据信息 |
| Block Storage Name | Body | String | - | 块存储名称 |

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
| Availability Zone Name | Body | String | 块存储所在可用区的名称 |
| Created At | Body | String | 块存储创建时间. yyyy-mm-ddTHH:MM:ssZ的格式. 例) 2017-05-16T02:17:50.166563 |
| Description | Body | String | 块存储描述 |
| Block Storage ID | Body | String | 块存储ID |
| Metadata Key / Value | Body | Boolean | 块存储中的元数据 |
| Block Storage Name | Body | String | 块存储名称 |
| Size | Body | Integer | 块存储大小(GB) |
| Status | Body | String | 块存储状态 |

### 删除块存储
删除块存储。只有Status为 "available" "in-use" "error" "error_restoring"的块存储才可以删除。

#### Method, URL
```
DELETE /v1.0/appkeys/{appkey}/volumes?id={volumeId}
X-Auth-Token: {tokenId}
```
|  Name | In | Type | Optional | Description |
|--|--|--|--|--|
| tokenId | Header | String | - | 令牌ID |
| volumeId | Query | String | - | 要删除的块存储ID |

#### Request Body
该 API无需 Request Body。

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

