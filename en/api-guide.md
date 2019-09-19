## Storage > Block Storage > API Guide 

API is currently available only in the Korea region.

## Prerequisites 

To use block storage API, Appkey and token are required: get them prepared through [API Endpoint URL](/Compute/Instance/en/api-guide/#api-endpoint-url) and [Token API](/Compute/Instance/en/api-guide/#api). Include Appkey to API Endpoint URL and token to the Request Header.

For instance, retrieving block storage must be requested to the following URL:  

	GET https://api-compute.cloud.toast.com/compute/v1.0/appkeys/{appkey}/volumes?id={volumeId}


## Block Storage API

Block storage can be created, deleted, and retrieved. Attaching/detaching block storage are available via [Additional Instance Functions API](/Compute/Instance/en/api-guide/#_8). 

### Status of Block Storage 

Block storage has following status values: 

| Status | Description |
| --- | --- |
| creating | Creating |
| available | Can be attached to an instance |
| attaching | Attaching to an instance |
| detaching | Detaching from an instance |
| in-use | Attached to an instance and now in use |
| deleting | Deleting |
| error | Error occurred while creating |
| error_deleting | Error occurred while deleting |
| backing-up | Backup is underway |
| restoring-backup | Restoring backup |
| error_backing-up | Error occurred while backup is underway |
| error_restoring | Error occurred while restoring a backup |
| downloading | Downloading an image |
| uploading | Uploading an image |

### Retrieve Block Storage

Retrieve information of block storage. 

#### Method, URL
```
GET /v1.0/appkeys/{appkey}/volumes?id={volumeId}
X-Auth-Token: {tokenId}
```

|  Name | In | Type | Optional | Description |
|--|--|--|--|--|
| tokenId | Header | String | - | Token ID |
| volumeId | Query | String | O | Block storage ID to retrieve: if unavailable, retrieve information of all block storages. |

#### Request Body
This API does not require the request body. 

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
| Device Name | Body | String  | Refers to device name at an instance, if attached to an instance e.g) "/dev/vdb" |
| Attached Instance ID | Body | String | ID of attached instance, if there is one |
| Attachment ID | Body | String | Attachment ID, if attached to an instance |
| Availability Zone Name | Body | String | Name of availability zone where block storage is located |
| Created At | Body | String | Time when block storage is created, in the format of yyyy-mm-ddTHH:MM:ssZ.  e.g) 2017-05-16T02:17:50.166563 |
| Description | Body | String | Description of block storage |
| Block Storage ID | Body | String | Block storage ID |
| Metadata Key / Value | Body | Boolean | Metadata written for block storage |
| Block Storage Name | Body | String | Name of block storage |
| Size | Body | Integer | Size of block storage (GB) |
| Status | Body | String | Status of block storage |
| Volume Type | Body | String | Type of block storage; one of "General HDD" or "General SSD" |

### Create Block Storage 
Create new block storage. 

#### Method, URL
```
POST /v1.0/appkeys/{appkey}/volumes
X-Auth-Token: {tokenId}
Content-Type: application/json;charset=UTF-8
```

|  Name | In | Type | Optional | Description |
|--|--|--|--|--|
| tokenId | Header | String | - | Token ID |

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
| Description | Body | String | O | Description of block storage |
| Availability Zone Name | Body | String | - | Name of availability zone to create block storage |
| Size | Body | Integer | - | Size of block storage (GB): enter by 10, between 10 and 1000. |
| Volume Type | Body | String | O | Type of block storage; one of "General HDD" or "General SSD" |
| Metadata Key / Metadata Value | Body | String | O | Metadata information wanted for block storage |
| Block Storage Name | Body | String | - | Name of block storage |

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
        "status": "{Status}"
    }
}
```

|  Name | In | Type | Description |
|--|--|--|--|
| Availability Zone Name | Body | String | Name of availability zone where block storage is located |
| Created At | Body | String | Time when block storage is created: in the format of yyyy-mm-ddTHH:MM:ssZ. e.g) 2017-05-16T02:17:50.166563 |
| Description | Body | String | Description of block storage |
| Block Storage ID | Body | String | ID of block storage |
| Metadata Key / Value | Body | Boolean | Metadata information written for block storage |
| Block Storage Name | Body | String | Name of block storage |
| Size | Body | Integer | Size of block storage (GB) |
| Status | Body | String | Status of block storage |

### Delete Block Storage 
Delete block storage: the status, however, must be either "available", "in-use", "error", or "error-restoring". 

#### Method, URL
```
DELETE /v1.0/appkeys/{appkey}/volumes?id={volumeId}
X-Auth-Token: {tokenId}
```
|  Name | In | Type | Optional | Description |
|--|--|--|--|--|
| tokenId | Header | String | - | Token ID |
| volumeId | Query | String | - | Block storage ID to delete |

#### Request Body
This API does not require the request body. 

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
