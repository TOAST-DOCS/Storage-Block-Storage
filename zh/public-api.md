## Storage > Block Storage > API v2 Guide

To use the API, API endpoint and token are required. Refer to [API usage preparations](/Compute/Compute/en/identity-api/) to prepare the information required to use the API.

Block Storage API uses the `volumev2` type endpoint. Refer to the `serviceCatalog` in the token issuance response for the valid endpoint.

| Type | Region | Endpoint |
|---|---|---|
| volumev2 | Korea (Pangyo)<br>Korea (Pyeongchon)<br>Japan | https://kr1-api-block-storage.infrastructure.cloud.toast.com<br>https://kr2-api-block-storage.infrastructure.cloud.toast.com<br>https://jp1-api-block-storage.infrastructure.cloud.toast.com |

In each API response, you may find fields that are not specified within this guide. Those fields are for NHN Cloud internal usage, so refrain from using them because they may be changed without prior notice.

## Block Storage Type
### List Block Storage Types 
```
GET /v2/{tenantId}/types
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |

#### Response

| Name | Type | Property | Description |
|---|---|---|---|
| volume_types | Body | Array | List of block storage type objects |
| volume_types.id | Body | UUID | ID of block storage type |
| volume_types.name | Body | String | Name of block storage type |
| volume_types.os-volume-type-access:is_public | Body | Boolean | Publicize block storage type or not |
| volume_types.description | Body | String | Description of block storage type |
| volume_types.extra_specs | Body | Object | Information object of extra specifications related with block storage type |

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

## Block Storage
### Block Storage Status
Block storage is available in many statuses with operations defined for each status. See the following list of available statuses:  

| Status Name | Description                         |
|--|----------------------------|
| `creating` | Creating a volume                    |
| `available` | Block storage is created and ready for attachment |
| `attaching`| Attaching block storage to an instance |
| `detaching` | Detaching block storage |
| `in-use`| block storage is attached to an instance |
| `maintenance`| block storage is migrating to another host equipment |
| `deleting`| Deleting block storage |
| `awaiting-transfer`| block storage is waiting for transfer |
| `error`| Error occurred while creating block storage |
| `error_deleting`| Error occurred while deleting block storage |
| `backing-up`| Backing up block storage |
| `restoring-backup`| Restoring block storage from backup |
| `error_backing-up`| Error occurred while backing up |
| `error_restoring`| Error occurred while restoring |
| `error_extending`| Error occurred while extending block storage |
| `downloading`| Downloading specified image while creating block storage |
| `uploading`| Uploading block storage image while creating an image |
| `retyping`| Changing block storage type |
| `extending`| Extending block storage |

### List Block Storage 
Return the list of block storage included to a current tenant. 

```
GET /v2/{tenantId}/volumes
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description                                                                                               |
|---|---|---|---|--------------------------------------------------------------------------------------------------|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |
| sort | Query | String | - | Name of block storage field for sorting<br>Described in the`< key >[: < direction > ]` format<br>e.g.) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | block storage count to return <br>Default is 1000 |
| offset | Query | Integer | - | Start point of the list to return<br>Return from the offset block storage out of the entire list |
| marker | Query | UUID | - | ID of the previous block storage of block storage to return <br>Return as much as the `limit` after block storage specified as the `marker` according to the sorting order |

#### Response

| Name | Type | Property | Description |
|---|---|---|---|
| volumes | Body | Array | List of block storage objects |
| volumes.id | Body | UUID | Block storage ID |
| volumes.links | Body | Object | Reference objects for block storage resource links |
| volumes.name | Body | String | Block storage name |
| volumes_links  | Body | Object | Information object (path indicating the next list) for pagination <br>Return when`limit` and `offset` are added |

<details><summary>Example</summary>
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

### List Block Storage Details 
Return the list of block storage included to a current tenant. 

```
GET /v2/{tenantId}/volumes/detail
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|------------------------------------------------------------------------------------------------|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |
| sort | Query | String | - | Name of block storage field for sorting<br>Described in the`< key >[: < direction > ]` format<br>e.g.) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | block storage count to return<br>Default is 1000 |
| offset | Query | Integer | - | Start point of the list to return <br/>Return from the offset block storage out of the entire list |
| marker | Query | UUID | - | ID of the previous block storage of block storage to return <br/>Return as much as `limit` after block storage specified as the `marker` according to the sorting order |

#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| volumes | Body | Array | List of detail block storage information objects |
| volumes.attachments | Body | Object | Information object for block storage attachment |
| volumes.attachments.server_id | Body | UUID | ID of instance with block storage attached |
| volumes.attachments.attachment_id | Body | UUID | ID of block storage attachment |
| volumes.attachments.volume_id | Body | UUID | Block storage ID |
| volumes.attachments.device | Body | String | Name of device within instance |
| volumes.attachments.id | Body | String | Block storage ID |
| volumes.links | Body | Object | Reference object of block storage resource link |
| volumes.availability_zone | Body | String | Block storage availability area |
| volumes.encrypted | Body | Boolean | Block storage encrypted or not |
| volumes.os-volume-replication:extended_status | Body | String | Block storage extension status |
| volumes.volume_type | Body | String | Name of block storage type |
| volumes.snapshot_id | Body | UUID | Snapshot ID specified while creating block storage |
| volumes.id | Body | UUID | Block storage ID |
| volumes.size | Body | Integer | Block storage size(GB) |
| volumes.user_id | Body | String | ID of block storage owner |
| volumes.os-vol-tenant-attr:tenant_id | Body | String | Tenant ID |
| volumes.metadata | Body | Object | Block storage metadata object |
| volumes.status | Body | Enum | Block storage status |
| volumes.description | Body | String | Block storage description |
| volumes.multiattach | Body | Boolean | Multiple attachment or not<br>With `true`, multiple attachment to many instances are available |
| volumes.source_volid | Body | UUID | Block storage ID specified when creating block storage |
| volumes.consistencygroup_id | Body | UUID | ID of block storage group |
| volumes.name | Body | String | Name of block storage |
| volumes.bootable | Body | Boolean | block storage bootable or not |
| volumes.created_at | Body | Datetime | Time of block storage creation <br>In the format of `YYYY-MM-DDThh:mm:ss.SSSSSS` |
| volumes.os-volume-replication:driver_data | Body | String | block storage replication data |
| volumes.replication_status | Body | String | block storage replication status |
| volumes.volumes_links  | Body | Object | Information object (pointing to the next list) for pagination <br>Return when`limit` and `offset` are added |



<details><summary>Example</summary>
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

### Get Block Storage 
Return details of specified block storage.  
```
GET /v2/{tenantId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| volumeId | URL | UUID | O | Block storage ID |
| tokenId | Header | String | O | Token ID |

#### Response

| Name | Type | Format | Description                                           |
|---|---|---|----------------------------------------------|
| volume | Body | Object | Detail information object of block storage |
| volume.attachments | Body | Object | Information object for block storage attachment |
| volume.attachments.server_id | Body | UUID | ID of instance attached to block storage |
| volume.attachments.attachment_id | Body | UUID | ID of block storage attachment |
| volume.attachments.volume_id | Body | UUID | block storage ID |
| volume.attachments.device | Body | String | Name of equipment within instance |
| volume.attachments.id | Body | String | block storage ID |
| volume.links | Body | Object | Reference object for block storage resource link |
| volume.availability_zone | Body | String | block storage availability area |
| volume.encrypted | Body | Boolean | block storage encrypted or not |
| volume.os-volume-replication:extended_status | Body | String | block storage extended status |
| volume.volume_type | Body | String | block storage type name |
| volume.snapshot_id | Body | UUID | Snapshot ID specified when creating block storage |
| volume.id | Body | UUID | block storage ID |
| volume.size | Body | Integer | block storage size (GB) |
| volume.user_id | Body | String | block storage owner ID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | Tenant ID |
| volume.metadata | Body | Object | block storage metadata object |
| volume.status | Body | Enum | block storage status |
| volume.description | Body | String | block storage description |
| volume.multiattach | Body | Boolean | Multiple attachment is available or not<br>With `true`, simultaneous attachment to many instances is enabled |
| volume.source_volid | Body | UUID | block storage ID specified when creating block storage |
| volume.consistencygroup_id | Body | UUID | block storage consistency block storage ID |
| volume.name | Body | String | block storage name |
| volume.bootable | Body | Boolean | block storage bootable |
| volume.created_at | Body | Datetime | block storage creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` |
| volume.os-volume-replication:driver_data | Body | String | block storage replication data |
| volume.replication_status | Body | String | block storage replication status |

<details><summary>Example</summary>
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

### Create Block Storage
Create a new or empty block storage from snapshot. 

Block storage is not immediately available after created. Query block storage status and check if it is `available`  first.  

```
POST /v2/{tenantId}/volumes
X-Auth-Token: {tokenId}
```

#### Request

| Name | Type | Format | Required | Description                        |
|---|---|---|---|---------------------------|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |
| volume | Body | Object | O | Object requesting of creating block storage |
| volume.size | Body | Integer | O | block storage size (GB) |
| volume.description | Body | String | - | block storage description |
| volume.availability_zone | Body | String | - | Name of voblock storagelume availability area |
| volume.name | Body | String | - | block storage name |
| volume.volume_type | Body | String | - | block storage type name |
| volume.snapshot_id | Body | UUID | - | Original snapshot ID: if left blank, empty block storage is created |
| volume.metadata | Body | Object | - | block storage metadata object |

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

#### Response

| Name | Type | Property | Description |
|---|---|---|---|
| volume | Body | Object | Information object for block storage details |
| volume.attachments | Body | Object | Information object for block storage attachment |
| volume.links | Body | Object | Reference object for block storage resource links |
| volume.availability_zone | Body | String | block storage visibility area |
| volume.encrypted | Body | Boolean | block storage encrypted or not |
| volume.os-volume-replication:extended_status | Body | String | block storage extended status |
| volume.volume_type | Body | String | block storage type name |
| volume.snapshot_id | Body | UUID | Snapshot ID specified for creating block storage |
| volumes.id | Body | UUID | block storage ID |
| volume.size | Body | Integer | block storage size (GB) |
| volume.user_id | Body | String | block storage owner ID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | Tenant ID |
| volume.metadata | Body | Object | block storage metadata object |
| volume.status | Body | Enum | block storage status |
| volume.description | Body | String | block storage description |
| volume.multiattach | Body | Boolean | Attachable to many instances |
| volume.consistencygroup_id | Body | UUID | block storage consistency group ID |
| volume.name | Body | String | block storage name |
| volume.bootable | Body | Boolean | block storage bootable |
| volume.created_at | Body | Datetime | block storage creation time<br>In the`YYYY-MM-DDThh:mm:ss.SSSSSS` format |
| volume.os-volume-replication:driver_data | Body | String | block storage replication data |
| volume.replication_status | Body | String | block storage replication status |

<details><summary>Example</summary>
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

### Delete Block Storage 

Delete specified block storage. Block storage that is attached or with snapshots created cannot be deleted. 

```
DELETE /v2/{tenantId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| volumeId | URL | String | O | Block storage ID |
| tokenId | Header | String | O | Token ID |

#### Response
This API does not return a response body.

---

### Create Image with Block Storage
Create image from block storage. 

At least 100KB of free space is required for basic initialization after image creation. The initialization operation may fail if the free space is less than this.

```
POST /v2/{tenantId}/volumes/{volumeId}/action
X-Auth-Token: {tokenId}
```

#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| volumeId | URL | UUID | O | Block storage ID |
| tokenId | Header | String | O | Token ID |
| os-volume_upload_image | Body | Object | O | Object requesting of creating block storage image |
| os-volume_upload_image.image_name | Body | String | O | Image name |
| os-volume_upload_image.force | Body | Boolean | - | Image creation allowed or not for block storage attached to instance <br>Default is false |
| os-volume_upload_image.disk_format | Body | String | - | Image disk format |
| os-volume_upload_image.container_format | Body | String | - | Image container format |
| os-volume_upload_image.visibility | Body | String | - | Image visibility<br>One of `private` or `shared` |
| os-volume_upload_image.protected | Body | Boolean | - | Whether to protect image</br>Cannot be modified or deleted when protected=true |

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

#### Response

| Name | Type | Property | Description |
|---|---|---|---|
| os-volume_upload_image | Body | Object | Response object creating block storage image |
| os-volume_upload_image.status | Body | String | block storage status |
| os-volume_upload_image.image_name | Body | String | Image name |
| os-volume_upload_image.disk_format | Body | String | Image disk format |
| os-volume_upload_image.container_format | Body | String | Image container format |
| os-volume_upload_image.updated_at | Body | Datetime | Image modification time |
| os-volume_upload_image.image_id | Body | UUID | Image ID |
| os-volume_upload_image.display_description | Body | String | block storage description |
| os-volume_upload_image.id | Body | UUID | block storage ID |
| os-volume_upload_image.size | Body | Integer | block storage size (GB) |
| os-volume_upload_image.volume_type | Body | Object | Information object for block storage type |

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

## Snapshot
### Snapshot status
Snapshots exist in various statuses, and each status defines its own set of permissible operations. See the following list of volume statuses.

| Status Name | Description                     |
|--|-------------------------|
| `creating` | Creating a snapshot |
| `available` | Snapshot has been created and is available |
| `backing-up      | Backing up a snapshot                               |
| `deleting`| Deleting a snapshot |
| `error`| Error has occurred while creating a snapshot |
| `deleted`| Snapshot has been deleted |
| `unmanaging`| Snapshot has been released from the management mode |
| `restoring`| Restoring block storage from snapshot |
| `error_deleting`| Error has occurred while deleting a snapshot |

### List Snapshots
Returns the list of snapshots.

```
GET /v2/{tenantId}/snapshots
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |

#### Response

| Name                 | Type | Format   | Description                                                  |
| -------------------- | ---- | -------- | ------------------------------------------------------------ |
| snapshot             | Body | Array    | Information object of snapshot details                       |
| snapshot.status      | Body | Enum     | Snapshot status                                              |
| snapshot.description | Body | String   | Snapshot description                                         |
| snapshot.created_at  | Body | Datetime | Snapshot creation time <br>In the`YYYY-MM-DDThh:mm:ss.SSSSSS` format |
| snapshot.metadata    | Body | Object   | Snapshot metadata object                                     |
| snapshot.volume_id   | Body | UUID     | Original block storage ID of snapshot                               |
| snapshot.size        | Body | Integer  | Original block storage size of snapshot (GB)                        |
| snapshot.id          | Body | UUID     | Snapshot ID                                                  |
| snapshot.name        | Body | String   | Snapshot name                                                |

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

### List Snapshots with Details
Returns a list of snapshot details.

```
GET /v2/{tenantId}/snapshots/detail
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |

#### Response

| Name                                                | Type | Format   | Description                                                  |
| --------------------------------------------------- | ---- | -------- | ------------------------------------------------------------ |
| snapshot                                            | Body | Array    | Information object of snapshot details                       |
| snapshot.status                                     | Body | Enum     | Snapshot status                                              |
| snapshot.description                                | Body | String   | Snapshot description                                         |
| snapshot.os-extended-snapshot-attributes:progress   | Body | String   | Progress of snapshot creation                                |
| snapshot.created_at                                 | Body | Datetime | Snapshot creation time In the`YYYY-MM-DDThh:mm:ss.SSSSSS` format |
| snapshot.metadata                                   | Body | Object   | Snapshot metadata object                                     |
| snapshot.volume_id                                  | Body | UUID     | Original block storage ID of snapshot                               |
| snapshot.os-extended-snapshot-attributes:project_id | Body | String   | Tenant ID                                                    |
| snapshot.size                                       | Body | Integer  | Original block storage size of snapshot (GB)                        |
| snapshot.id                                         | Body | UUID     | Snapshot ID                                                  |
| snapshot.name                                       | Body | String   | Snapshot name                                                |

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

### View Snapshot
Returns details of the specified snapshot.

```
GET /v2/{tenantId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| snapshotId | URL | UUID | O | Snapshot ID |
| tokenId | Header | String | O | Token ID |

#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| snapshot | Body | Object | Snapshot Details Object |
| snapshot.status | Body | Enum | Snapshot status |
| snapshot.description | Body | String | Snapshot description |
| snapshot.os-extended-snapshot-attributes:progress | Body | String | Snapshot creation progress |
| snapshot.created_at | Body | Datetime | Snapshot creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`format |
| snapshot.metadata | Body | Object | Snapshot metadata object |
| snapshot.volume_id | Body | UUID | Original block storage ID of snapshot |
| snapshot.os-extended-snapshot-attributes:project_id | Body | String | Tenant ID |
| snapshot.size | Body | Integer | Original block storage size of snapshot (GB) |
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

### Create Snapshot
Create snapshot for specified block storage. 

```
POST /v2/{tenantId}/snapshots
X-Auth-Token: {tokenId}
```

#### Request

| Name | Type | Format | Required | Description                                       |
|---|---|---|---|-------------------------------------------|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |
| snapshot | Body | Object | O | Object requesting of creating snapshot |
| snapshot.volume_id | Body | UUID | O | Original block storage ID |
| snapshot.force | Body | Boolean | - | Forced to create snapshot or not<br>With`true`, snapshot is created even if block storage is attached |
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

#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| snapshot | Body | Object | Snapshot Details Object |
| snapshot.status | Body | Enum | Snapshot status |
| snapshot.description | Body | String | Snapshot description |
| snapshot.created_at | Body | Datetime | Snapshot creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`format |
| snapshot.metadata | Body | Object | Snapshot metadata object |
| snapshot.volume_id | Body | UUID | Original block storage ID of snapshot |
| snapshot.size | Body | Integer | Original block storage size of snapshot (GB) |
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

### Delete Snapshots
Deletes a specified snapshot.

```
DELETE /v2/{tenantId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| snapshotId | URL | String | O | Snapshot ID |
| tokenId | Header | String | O | Token ID |

#### Response
This API does not return a response body.
