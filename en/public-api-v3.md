<!-- machine_translated: true -->

<a id="storage-block-storage-api-v3-guide"></a>

## Storage > Block Storage > API v3 Guide { #storage-block-storage-api-v3-guide }

Block Storage uses the IaaS token for authentication/authorization when making API calls. The IaaS token is the authentication token used by NHN Cloud's OpenStack-based infrastructure service (IaaS). For more information on issuing and using IaaS tokens, see [IaaS token](/nhncloud/en/public-api/iaas-token).

The block storage API uses the `volumev3` type endpoint. Refer to the `serviceCatalog` in the token issuance response for the exact endpoint.

| Type | Region | Endpoint |
|---|---|---|
| volumev3 | Korea (Pangyo) Region<br>Korea (Pyeongchon) Region<br>Korea (Gwangju) Region<br>Japan Region | https://kr1-api-block-storage-infrastructure.nhncloudservice.com<br>https://kr2-api-block-storage-infrastructure.nhncloudservice.com<br>https://kr3-api-block-storage-infrastructure.nhncloudservice.com<br>https://jp1-api-block-storage-infrastructure.nhncloudservice.com |

API response may show the fields not specified by the guide. These fields are internally used by NHN Cloud, and not used because they are subject to change without prior notice.

<a id="block-storage-type"></a>

## Block Storage Type { #block-storage-type }

<a id="list-block-storage-types"></a>
### List Block Storage Types { #list-block-storage-types }
```
GET /v3/{tenantId}/types
X-Auth-Token: {tokenId}
```

<a id="request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |

<a id="response"></a>
#### Response

| Name | Type | Property | Description |
|---|---|---|---|
| volume_types | Body | Array | List of block storage type objects |
| volume_types.id | Body | UUID | Block storage type ID |
| volume_types.name | Body | String | Block storage type name |
| volume_types.os-volume-type-access:is_public | Body | Boolean | Whether the block storage type is public |
| volume_types.description | Body | String | Block storage type description |
| volume_types.extra_specs | Body | Object | Object containing additional specification information for the block storage type |

<details><summary>Example</summary>
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

## Block Storage { #block-storage }

<a id="block-storage-status"></a>

### Block Storage Status { #block-storage-status }
Block storage exists in various statuses, and each status defines its own set of permissible operations. See the following list of statuses.

| Status Name | Description |
|--|---|
| `creating` | Being created |
| `available` | Block storage has been created and is ready for attachment |
| `attaching` | Block storage is being attached to an instance |
| `detaching` | Block storage is being detached |
| `in-use` | Block storage is attached to an instance |
| `reserved` | Root block storage of a terminated instance |
| `maintenance` | Block storage is being migrated to another host machine |
| `deleting` | Block storage is being deleted |
| `awaiting-transfer` | Block storage is waiting to be transferred |
| `error` | An error occurred while creating block storage |
| `error_deleting` | An error occurred while deleting block storage |
| `backing-up` | Block storage is being backed up |
| `restoring-backup` | Block storage is being restored from a backup |
| `error_backing-up` | An error occurred during backup |
| `error_restoring` | An error occurred during restoration |
| `error_extending` | An error occurred while extending block storage |
| `downloading` | Downloading the image specified when creating block storage |
| `uploading` | Uploading the block storage image when creating an image |
| `retyping` | Block storage type is being changed |
| `extending` | Block storage is being extended |

<a id="list-block-storage"></a>

### List Block Storage { #list-block-storage }
Returns a list of block storage belonging to the current tenant.

```
GET /v3/{tenantId}/volumes
X-Auth-Token: {tokenId}
```

<a id="list-block-storage-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description                                                                                               |
|---|---|---|---|--------------------------------------------------------------------------------------------------|
| tenantId | URL | String | O | Tenant ID                                                                                           |
| tokenId | Header | String | O | Token ID                                                                                            |
| sort | Query | String | - | Name of the block storage field to use as the sorting criterion<br>Describe it in the form `< key >[: < direction > ]`<br>Example: `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | Number of block storage items to return<br>Default value is set to 1000                                                                 |
| offset | Query | Integer | - | Starting point of the list to return<br>Return block storage starting from offset of the entire list                                               |
| marker | Query | UUID | - | ID of the block storage immediately preceding the block storage to return<br>Returns as much as the `limit` after block storage specified as the `marker` according to the sorting order                 |

<a id="list-block-storage-response"></a>
#### Response

| Name | Type | Property | Description |
|---|---|---|---|
| volumes | Body | Array | List of block storage objects |
| volumes.id | Body | UUID | Block storage ID |
| volumes.links | Body | Object | Block storage resource link reference object |
| volumes.name | Body | String | Block storage name |
| volumes_links  | Body | Object | Information object for pagination (path to next list)<br>Returned when `limit` and `offset` are added |

<details><summary>Example</summary>
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

### List Block Storage Details { #list-block-storage-details }
Returns a list of block storage belonging to the current tenant.

```
GET /v3/{tenantId}/volumes/detail
X-Auth-Token: {tokenId}
```

<a id="list-block-storage-details-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description                                                                                               |
|---|---|---|---|--------------------------------------------------------------------------------------------------|
| tenantId | URL | String | O | Tenant ID                                                                                           |
| tokenId | Header | String | O | Token ID                                                                                            |
| sort | Query | String | - | Field name of block storage to sort by<br>`< key >[: < direction > ]` format<br>Example: `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | Number of block storage to return<br>Default is set to 1000                                                                 |
| offset | Query | Integer | - | Starting point of the list to return<br/>Return block storage starting from offset of the entire list                                              |
| marker | Query | UUID | - | ID of the block storage immediately before the block storage to return<br/>Returns as much as the `limit` after block storage specified as the `marker` according to the sorting order                |

<a id="list-block-storage-details-response"></a>
#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| volumes | Body | Array | List of block storage details objects |
| volumes.attachments | Body | Object | Block storage attachment information object |
| volumes.attachments.server_id | Body | UUID | ID of the instance to which block storage is attached |
| volumes.attachments.attachment_id | Body | UUID | Block storage attachment ID |
| volumes.attachments.volume_id | Body | UUID | Block storage ID |
| volumes.attachments.device | Body | String | Device name within the instance |
| volumes.attachments.id | Body | String | Block storage ID |
| volumes.links | Body | Object | Block storage resource link reference object |
| volumes.availability_zone | Body | String | Block storage availability area |
| volumes.encrypted | Body | Boolean | Whether block storage is encrypted |
| volumes.os-volume-replication:extended_status | Body | String | Block storage extended status |
| volumes.volume_type | Body | String | Block storage type name |
| volumes.snapshot_id | Body | UUID | Snapshot ID specified when creating block storage |
| volumes.id | Body | UUID | Block storage ID |
| volumes.size | Body | Integer | Block storage size (GB) |
| volumes.user_id | Body | String | Block storage owner ID |
| volumes.os-vol-tenant-attr:tenant_id | Body | String | Tenant ID |
| volumes.metadata | Body | Object | Block storage metadata object |
| volumes.status | Body | Enum | Block storage status |
| volumes.description | Body | String | Block storage description |
| volumes.multiattach | Body | Boolean | Whether Multiple attachment is possible<br>If `true`, can be attached to multiple instances simultaneously |
| volumes.source_volid | Body | UUID | Block storage ID specified when creating block storage |
| volumes.consistencygroup_id | Body | UUID | Block storage group ID |
| volumes.name | Body | String | Block storage name |
| volumes.bootable | Body | String | Whether block storage is bootable |
| volumes.created_at | Body | Datetime | Block storage creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` format |
| volumes.os-volume-replication:driver_data | Body | String | Block storage replication data |
| volumes.replication_status | Body | String | Block storage replication status |
| volumes.volumes_links  | Body | Object | Information object for pagination (path to next list)<br>Returned when `limit` and `offset` are added |
| volumes.nhn_encryption            | Body | Object | Block storage encryption information |
| volumes.nhn_encryption.skm_key_version | Body | Integer | Version of the symmetric key of Secure Key Manager to be used to create encrypted block storage |
| volumes.nhn_encryption.skm_key_id | Body | String | Symmetric key ID of Secure Key Manager to be used to create encrypted block storage |

<details><summary>Example</summary>
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

### Get Block Storage { #get-block-storage }
Returns the details of the specified block storage.

```
GET /v3/{tenantId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

<a id="get-block-storage-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| volumeId | URL | UUID | O | Block storage ID |
| tokenId | Header | String | O | Token ID |

<a id="get-block-storage-response"></a>
#### Response

| Name | Type | Format | Description                                           |
|---|---|---|----------------------------------------------|
| volume | Body | Object | Block storage details object                                  |
| volume.attachments | Body | Object | Block storage attachment information object                                  |
| volume.attachments.server_id | Body | UUID | ID of the instance that the block storage is attached to                              |
| volume.attachments.attachment_id | Body | UUID | Block storage attachment ID                                     |
| volume.attachments.volume_id | Body | UUID | Block storage ID                                        |
| volume.attachments.device | Body | String | Device name within the instance                                 |
| volume.attachments.id | Body | String | Block storage ID                                        |
| volume.links | Body | Object | Block storage resource link reference object                              |
| volume.availability_zone | Body | String | Block storage availability area                                    |
| volume.encrypted | Body | Boolean | Whether the block storage is encrypted                                    |
| volume.os-volume-replication:extended_status | Body | String | Block storage extended status                                     |
| volume.volume_type | Body | String | Block storage type name                                     |
| volume.snapshot_id | Body | UUID | Snapshot ID specified when creating block storage                           |
| volume.id | Body | UUID | Block storage ID                                        |
| volume.size | Body | Integer | Block storage size (GB)                                    |
| volume.user_id | Body | String | Block storage owner ID                                    |
| volume.os-vol-tenant-attr:tenant_id | Body | String | Tenant ID                                       |
| volume.metadata | Body | Object | Block storage metadata object                                  |
| volume.status | Body | Enum | Block storage status                                        |
| volume.description | Body | String | Block storage description                                        |
| volume.multiattach | Body | Boolean | Whether Multiple attachment is available<br>If `true`, you can attach multiple instances simultaneously |
| volume.source_volid | Body | UUID | Block storage ID specified when creating block storage                            |
| volume.consistencygroup_id | Body | UUID | Block storage consistency group ID                          |
| volume.name | Body | String | Block storage name                                        |
| volume.bootable | Body | String | Whether the block storage is bootable                                  |
| volume.created_at | Body | Datetime | Block storage created time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`     |
| volume.os-volume-replication:driver_data | Body | String | Block storage replication data                                    |
| volume.replication_status | Body | String | Block storage replication status                                     |
| volume.nhn_encryption            | Body | Object | Block storage encryption information |
| volume.nhn_encryption.skm_key_version | Body | Integer | Symmetric key version of Secure Key Manager to be used to create encrypted block storage |
| volume.nhn_encryption.skm_key_id | Body | String | Symmetric key ID of Secure Key Manager to be used to create encrypted block storage |

<details><summary>Example</summary>
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

### Create Block Storage { #create-block-storage }
Creates new block storage from a snapshot or creates empty block storage.

Block storage cannot be used immediately after creation. Query the block storage status and use it after confirming that it is `available`.

```
POST /v3/{tenantId}/volumes
X-Auth-Token: {tokenId}
```

<a id="create-block-storage-request"></a>
#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |
| volume | Body | Object | O | Block storage creation request object |
| volume.size | Body | Integer | O | Block storage size (GB) |
| volume.description | Body | String | - | Block storage description |
| volume.availability_zone | Body | String | - | Block storage availability area name |
| volume.name | Body | String | - | Block storage name |
| volume.volume_type | Body | String | - | Block storage type name |
| volume.snapshot_id | Body | UUID | - | Original snapshot ID, if omitted, empty block storage is created |
| volume.metadata | Body | Object | - | Block storage metadata object |
| volume.nhn_encryption            | Body | Object | - | Block storage encryption information |
| volume.nhn_encryption.skm_appkey | Body | String | - | Appkey of the Secure Key Manager service |
| volume.nhn_encryption.skm_key_id | Body | String | - | Symmetric key ID of Secure Key Manager to be used to create encrypted block storage |

<details><summary>Example</summary>
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
#### Response

| Name | Type | Property | Description |
|---|---|---|---|
| volume | Body | Object | Block storage details object |
| volume.attachments | Body | Object | Block storage attachment information object |
| volume.links | Body | Object | Block storage resource link reference object |
| volume.availability_zone | Body | String | Block storage availability area |
| volume.encrypted | Body | Boolean | Whether block storage is encrypted |
| volume.os-volume-replication:extended_status | Body | String | Block storage extended status |
| volume.volume_type | Body | String | Block storage type name |
| volume.snapshot_id | Body | UUID | Snapshot ID specified when creating block storage |
| volumes.id | Body | UUID | Block storage ID |
| volume.size | Body | Integer | Block storage size (GB) |
| volume.user_id | Body | String | Block storage owner ID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | Tenant ID |
| volume.metadata | Body | Object | Block storage metadata object |
| volume.status | Body | Enum | Block storage status |
| volume.description | Body | String | Block storage description |
| volume.multiattach | Body | Boolean | Whether it can be attached to multiple instances |
| volume.name | Body | String | Block storage name |
| volume.bootable | Body | String | Whether block storage is bootable |
| volume.created_at | Body | Datetime | Block storage creation time<br>In the format `YYYY-MM-DDThh:mm:ss.SSSSSS` |
| volume.os-volume-replication:driver_data | Body | String | Block storage replication data |
| volume.replication_status | Body | String | Block storage replication status |
| volume.nhn_encryption            | Body | Object | Block storage encryption information |
| volume.nhn_encryption.skm_key_version | Body | Integer | Symmetric key version of Secure Key Manager to be used to create encrypted block storage |
| volume.nhn_encryption.skm_key_id | Body | String | Symmetric key ID of Secure Key Manager to be used to create encrypted block storage |

<details><summary>Example</summary>
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

### Delete Block Storage { #delete-block-storage }

Deletes the specified block storage. Block storage that is attached or has snapshots cannot be deleted.

```
DELETE /v3/{tenantId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

<a id="delete-block-storage-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| volumeId | URL | String | O | Block storage ID |
| tokenId | Header | String | O | Token ID |

<a id="delete-block-storage-response"></a>
#### Response
This API does not return a response body.

---

<a id="create-image-with-block-storage"></a>

### Create Image with Block Storage { #create-image-with-block-storage }
Creates an image from block storage.

At least 100KB of free space is required for basic initialization after image creation. The initialization operation may fail if the free space is less than this.

> [Caution]
> The size of the generated image might be larger than the actual usage of the root block storage.

```
POST /v3/{tenantId}/volumes/{volumeId}/action
X-Auth-Token: {tokenId}
```

<a id="create-image-with-block-storage-request"></a>
#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| volumeId | URL | UUID | O | Block storage ID |
| tokenId | Header | String | O | Token ID |
| os-volume_upload_image | Body | Object | O | Object for block storage image creation request |
| os-volume_upload_image.image_name | Body | String | O | Image name |
| os-volume_upload_image.force | Body | Boolean | - | Whether to allow image creation for block storage attached to an instance<br>Default is false |
| os-volume_upload_image.disk_format | Body | String | - | Image disk format |
| os-volume_upload_image.container_format | Body | String | - | Image container format |
| os-volume_upload_image.visibility | Body | String | - | Image visibility<br>One of `private` or `shared` |
| os-volume_upload_image.protected | Body | Boolean | - | Whether the image is protected</br>Cannot be modified or deleted when protected=true |

<details><summary>Example</summary>
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
#### Response

| Name | Type | Property | Description |
|---|---|---|---|
| os-volume_upload_image | Body | Object | Object for block storage image creation response |
| os-volume_upload_image.status | Body | String | Block storage status |
| os-volume_upload_image.image_name | Body | String | Image name |
| os-volume_upload_image.disk_format | Body | String | Image disk format |
| os-volume_upload_image.container_format | Body | String | Image container format |
| os-volume_upload_image.updated_at | Body | Datetime | Time when the image was last updated |
| os-volume_upload_image.image_id | Body | UUID | Image ID |
| os-volume_upload_image.display_description | Body | String | Block storage description |
| os-volume_upload_image.id | Body | UUID | Block storage ID |
| os-volume_upload_image.size | Body | Integer | Block storage size (GB) |
| os-volume_upload_image.volume_type | Body | Object | Object containing block storage type information |

<details><summary>Example</summary>
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

## Snapshot { #snapshot }

<a id="snapshot-status"></a>
### Snapshot Status { #snapshot-status }
Snapshots exist in various statuses, and each status defines its own set of permissible operations. See the following list of volume statuses.

| Status Name | Description |
|--|-------------------------|
| `creating` | Being created |
| `available` | Snapshot created and ready to use |
| `backing-up` | Snapshot being backed up |
| `deleting` | Snapshot being deleted |
| `error` | An error occurred during creation |
| `deleted` | Deleted |
| `unmanaging` | Admin mode for snapshots being turned off |
| `restoring` | Restoring block storage from the snapshot |
| `error_deleting` | An error occurred during deletion |

<a id="list-snapshots"></a>
### List Snapshots { #list-snapshots }
Retrieves a list of snapshots.

```
GET /v3/{tenantId}/snapshots
X-Auth-Token: {tokenId}
```

<a id="list-snapshots-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |

<a id="list-snapshots-response"></a>
#### Response

| Name | Type | Property | Description |
|---|---|---|---|
| snapshots | Body | Array | List of snapshot information objects |
| snapshots.status | Body | Enum | Snapshot status |
| snapshots.description | Body | String | Snapshot description |
| snapshots.created_at | Body | Datetime | Snapshot creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` format |
| snapshots.metadata | Body | Object | Snapshot metadata object |
| snapshots.volume_id | Body | UUID | Original block storage ID of the snapshot |
| snapshots.size | Body | Integer | Original block storage size of the snapshot (GB) |
| snapshots.id | Body | UUID | Snapshot ID |
| snapshots.name | Body | String | Snapshot name |

<details><summary>Example</summary>
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
### List Snapshots with Details { #list-snapshots-with-details }
Retrieves a list of snapshot details.

```
GET /v3/{tenantId}/snapshots/detail
X-Auth-Token: {tokenId}
```

<a id="list-snapshots-with-details-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |

<a id="list-snapshots-with-details-response"></a>
#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| snapshots | Body | Array | List of snapshot detail objects |
| snapshots.status | Body | Enum | Snapshot status |
| snapshots.description | Body | String | Snapshot description |
| snapshots.os-extended-snapshot-attributes:progress | Body | String | Snapshot creation progress |
| snapshots.created_at | Body | Datetime | Snapshot creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` format |
| snapshots.metadata | Body | Object | Snapshot metadata object |
| snapshots.volume_id | Body | UUID | Original block storage ID of the snapshot |
| snapshots.os-extended-snapshot-attributes:project_id | Body | String | Tenant ID |
| snapshots.size | Body | Integer | Original block storage size of the snapshot (GB) |
| snapshots.id | Body | UUID | Snapshot ID |
| snapshots.name | Body | String | Snapshot name |

<details><summary>Example</summary>
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
### Get Snapshot { #view-snapshot }
Retrieves the details of the specified snapshot.

```
GET /v3/{tenantId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

<a id="view-snapshot-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| snapshotId | URL | UUID | O | Snapshot ID |
| tokenId | Header | String | O | Token ID |

<a id="view-snapshot-response"></a>
#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| snapshot | Body | Object | Snapshot detail object |
| snapshot.status | Body | Enum | Snapshot status |
| snapshot.description | Body | String | Snapshot description |
| snapshot.os-extended-snapshot-attributes:progress | Body | String | Snapshot creation progress |
| snapshot.created_at | Body | Datetime | Snapshot creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` format |
| snapshot.metadata | Body | Object | Snapshot metadata object |
| snapshot.volume_id | Body | UUID | Original block storage ID of the snapshot |
| snapshot.os-extended-snapshot-attributes:project_id | Body | String | Tenant ID |
| snapshot.size | Body | Integer | Original block storage size of the snapshot (GB) |
| snapshot.id | Body | UUID | Snapshot ID |
| snapshot.name | Body | String | Snapshot name |

<details><summary>Example</summary>
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
### Create Snapshot { #create-snapshot }
Creates a snapshot of the specified block storage.

```
POST /v3/{tenantId}/snapshots
X-Auth-Token: {tokenId}
```

<a id="create-snapshot-request"></a>
#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |
| snapshot | Body | Object | O | Snapshot creation request object |
| snapshot.volume_id | Body | UUID | O | Original block storage ID |
| snapshot.force | Body | Boolean | - | Whether to force snapshot creation<br>If `true`, creates a snapshot even when block storage is attached |
| snapshot.description | Body | String | - | Snapshot description |
| snapshot.name | Body | String | - | Snapshot name |

<details><summary>Example</summary>
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
#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| snapshot | Body | Object | Snapshot detail object |
| snapshot.status | Body | Enum | Snapshot status |
| snapshot.description | Body | String | Snapshot description |
| snapshot.created_at | Body | Datetime | Snapshot creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` format |
| snapshot.metadata | Body | Object | Snapshot metadata object |
| snapshot.volume_id | Body | UUID | Original block storage ID of the snapshot |
| snapshot.size | Body | Integer | Original block storage size of the snapshot (GB) |
| snapshot.id | Body | UUID | Snapshot ID |
| snapshot.name | Body | String | Snapshot name |

<details><summary>Example</summary>
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
### Delete Snapshot { #delete-snapshots }
Deletes the specified snapshot.

```
DELETE /v3/{tenantId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

<a id="delete-snapshots-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| snapshotId | URL | String | O | Snapshot ID |
| tokenId | Header | String | O | Token ID |

<a id="delete-snapshots-response"></a>
#### Response
This API does not return a response body.