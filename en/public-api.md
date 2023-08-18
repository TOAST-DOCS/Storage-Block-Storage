## Storage > Block Storage > API v2 Guide

To use the API, API endpoint and token are required. Refer to [API usage preparations](/Compute/Compute/en/identity-api/) to prepare the information required to use the API.

Block Storage API uses the `volumev2` type endpoint. Refer to the `serviceCatalog` in the token issuance response for the valid endpoint.

| Type | Region | Endpoint |
|---|---|---|
| volumev2 | Korea (Pangyo)<br>Korea (Pyeongchon)<br>Japan | https://kr1-api-block-storage.infrastructure.cloud.toast.com<br>https://kr2-api-block-storage.infrastructure.cloud.toast.com<br>https://jp1-api-block-storage.infrastructure.cloud.toast.com |

In each API response, you may find fields that are not specified within this guide. Those fields are for NHN Cloud internal usage, so refrain from using them because they may be changed without prior notice.

## Volume Type
### List Volume Types
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
| volume_types | Body | Array | List of volume type objects |
| volume_types.id | Body | UUID | Volume type ID |
| volume_types.name | Body | String | Volume type name |
| volume_types.os-volume-type-access:is_public | Body | Boolean | Volume type is public or not |
| volume_types.description | Body | String | Volume type description |
| volume_types.extra_specs | Body | Object | Object for additional specification related to volume type |

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

## Volume
### Volume Status
Volumes exist in various statuses, and each status defines its own set of permissible operations. See the following list of volume statuses.

| Status Name | Description |
|--|--|
| `creating` | Volume being created |
| `available` | Volume created and ready to attach |
| `attaching`| Volume is being attached to the instance |
| `detaching`| Volume is being detached |
| `in-use`| Volume attached to the instance |
| `maintenance`| Volume is being migrated to another host device. |
| `deleting`| Volume is being deleted |
| `awaiting-transfer`| Volume awaiting transfer |
| `error`| An error occurred when creating a volume |
| `error_deleting`| An error occurred when deleting a volume |
| `backing-up`| Volume is being backed up |
| `restoring-backup`| Volume is recovering from backup |
| `error_backing-up`| An error occurred during backup |
| `error_restoring`| An error occurred while recovering |
| `error_extending`| An error occurred while expanding volume |
| `downloading`| Downloading the image specified when creating a volume |
| `uploading`| Uploading the volume image when creating an image |
| `retyping`| Changing the volume type |
| `extending`| Expanding the volume |

### List Volumes
Returns a list of volumes belonging to the current tenant.

```
GET /v2/{tenantId}/volumes
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |
| sort | Query | String | - | Name of the volume field to sort by<br>`< key >[: < direction > ]` <br>Example) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | Number of volumes to return<br>Default set to 1000 |
| offset | Query | Integer | - | Start point of returned list<br>Return volumes starting from offset of the entire list |
| marker | Query | UUID | - | The previous volume ID of the volume to be returned.<br>Returns as much as the `limit` after volume specified as the `marker` according to the sorting order |

#### Response

| Name | Type | Property | Description |
|---|---|---|---|
| volumes | Body | Array | List of volume objects |
| volumes.id | Body | UUID | Volume ID |
| volumes.links | Body | Object | Volume resource link reference object |
| volumes.name | Body | String | Volume name |
| volumes_links  | Body | Object | Information object for pagination (path to next list)<br>Return when `limit` or `offset` is added |

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

### List Volumes with Details
Returns a list of volumes belonging to the current tenant.

```
GET /v2/{tenantId}/volumes/detail
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |
| sort | Query | String | - | Name of the volume field to sort by<br>`< key >[: < direction > ]` <br>Example) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | Number of volumes to return<br>Default set to 1000 |
| offset | Query | Integer | - | Start point of returned list<br/>Return volumes starting from offset of the entire list |
| marker | Query | UUID | - | The previous volume ID of the volume to be returned.<br/>Returns as much as the `limit` after volume specified as the `marker` according to the sorting order |

#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| volumes | Body | Array | List of volume details objects |
| volumes.attachments | Body | Object | Volume attachment information object |
| volumes.attachments.server_id | Body | UUID | Instance ID to which the volume is attached |
| volumes.attachments.attachment_id | Body | UUID | Volume attachment ID |
| volumes.attachments.volume_id | Body | UUID | Volume ID |
| volumes.attachments.device | Body | String | Device name in instance |
| volumes.attachments.id | Body | String | Volume ID |
| volumes.links | Body | Object | Volume resource link reference object |
| volumes.availability_zone | Body | String | Volume availability zone |
| volumes.encrypted | Body | Boolean | Whether encrypted or not |
| volumes.os-volume-replication:extended_status | Body | String | Volume expansion status |
| volumes.volume_type | Body | String | Volume type name |
| volumes.snapshot_id | Body | UUID | Snapshot ID specified when creating the volume |
| volumes.id | Body | UUID | Volume ID |
| volumes.size | Body | Integer | Volume size (GB) |
| volumes.user_id | Body | String | Volume Owner ID |
| volumes.os-vol-tenant-attr:tenant_id | Body | String | Tenant ID |
| volumes.metadata | Body | Object | Volume metadata object |
| volumes.status | Body | Enum | Volume Status |
| volumes.description | Body | String | Volume description |
| volumes.multiattach | Body | Boolean | Multiple attachment is possible or not<br>If `true`, you can attach multiple instances simultaneously |
| volumes.source_volid | Body | UUID | Volume ID specified when creating the volume |
| volumes.consistencygroup_id | Body | UUID | Volume group ID |
| volumes.name | Body | String | Volume name |
| volumes.bootable | Body | Boolean | Volume is bootable or not |
| volumes.created_at | Body | Datetime | Volume creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`format |
| volumes.os-volume-replication:driver_data | Body | String | Volume clone data |
| volumes.replication_status | Body | String | Volume replication status |
| volumes.volumes_links  | Body | Object | Information object for pagination (path to next list)<br>Return when `limit` or `offset` is added |

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

### View Volume
Returns details of the specified volume.

```
GET /v2/{tenantId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| volumeId | URL | UUID | O | Volume ID |
| tokenId | Header | String | O | Token ID |

#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| volume | Body | Object | Volume detail object |
| volume.attachments | Body | Object | Volume attachment information object |
| volume.attachments.server_id | Body | UUID | Instance ID to which the volume is attached |
| volume.attachments.attachment_id | Body | UUID | Volume attachment ID |
| volume.attachments.volume_id | Body | UUID | Volume ID |
| volume.attachments.device | Body | String | Device name in instance |
| volume.attachments.id | Body | String | Volume ID |
| volume.links | Body | Object | Volume resource link reference object |
| volume.availability_zone | Body | String | Volume availability zone |
| volume.encrypted | Body | Boolean | Whether encrypted or not |
| volume.os-volume-replication:extended_status | Body | String | Volume expansion status |
| volume.volume_type | Body | String | Volume type name |
| volume.snapshot_id | Body | UUID | Snapshot ID specified when creating the volume |
| volume.id | Body | UUID | Volume ID |
| volume.size | Body | Integer | Volume size (GB) |
| volume.user_id | Body | String | Volume Owner ID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | Tenant ID |
| volume.metadata | Body | Object | Volume metadata object |
| volume.status | Body | Enum | Volume Status |
| volume.description | Body | String | Volume description |
| volume.multiattach | Body | Boolean | Multiple attachment is possible or not<br>If `true`, you can attach multiple instances simultaneously |
| volume.source_volid | Body | UUID | Volume ID specified when creating the volume |
| volume.consistencygroup_id | Body | UUID | Volume consistency group ID |
| volume.name | Body | String | Volume name |
| volume.bootable | Body | Boolean | Volume is bootable or not |
| volume.created_at | Body | Datetime | Volume creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` |
| volume.os-volume-replication:driver_data | Body | String | Volume clone data |
| volume.replication_status | Body | String | Volume replication status |

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

### Create Volume
Creates a new volume from a snapshot or create an empty volume.

A volume cannot be used immediately after creation. Query the volume status and use it after confirming that it is `available`.

```
POST /v2/{tenantId}/volumes
X-Auth-Token: {tokenId}
```

#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |
| volume | Body | Object | O | Volume creation request object |
| volume.size | Body | Integer | O | Volume size (GB) |
| volume.description | Body | String | - | Volume description |
| volume.multiattach | Body | Boolean | - | Multiple attachment is possible or not<br>if set to `true`<br>you can attach multiple instances simultaneously |
| volume.availability_zone | Body | String | - | Volume availability zone name |
| volume.name | Body | String | - | Volume name |
| volume.volume_type | Body | String | - | Volume type name |
| volume.snapshot_id | Body | UUID | - | Original snapshot ID, if omitted, an empty volume is created |
| volume.metadata | Body | Object | - | Volume metadata object |

<details><summary>Example</summary>
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
| volume | Body | Object | Volume detail object |
| volume.attachments | Body | Object | Volume attachment information object |
| volume.links | Body | Object | Volume resource link reference object |
| volume.availability_zone | Body | String | Volume availability zone |
| volume.encrypted | Body | Boolean | Whether encrypted or not |
| volume.os-volume-replication:extended_status | Body | String | Volume expansion status |
| volume.volume_type | Body | String | Volume type name |
| volume.snapshot_id | Body | UUID | Snapshot ID specified when creating the volume |
| volumes.id | Body | UUID | Volume ID |
| volume.size | Body | Integer | Volume size (GB) |
| volume.user_id | Body | String | Volume Owner ID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | Tenant ID |
| volume.metadata | Body | Object | Volume metadata object |
| volume.status | Body | Enum | Volume Status |
| volume.description | Body | String | Volume description |
| volume.multiattach | Body | Boolean | Available to attach to multiple instances |
| volume.name | Body | String | Volume name |
| volume.bootable | Body | Boolean | Volume is bootable or not |
| volume.created_at | Body | Datetime | Volume creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`format |
| volume.os-volume-replication:driver_data | Body | String | Volume clone data |
| volume.replication_status | Body | String | Volume replication status |

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

### Delete Volume

Delete the specified volume. Attached or snapshotted volumes cannot be deleted.

```
DELETE /v2/{tenantId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| volumeId | URL | String | O | Volume ID |
| tokenId | Header | String | O | Token ID |

#### Response
This API does not return a response body.

---

### Create an Image with a Volume
Creates an image from a volume. 

At least 100KB of free space is required for basic initialization after image creation. The initialization operation may fail if the free space is less than this.

```
POST /v2/{tenantId}/volumes/{volumeId}/action
X-Auth-Token: {tokenId}
```

#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| volumeId | URL | UUID | O | Volume ID |
| tokenId | Header | String | O | Token ID |
| os-volume_upload_image | Body | Object | O | Volume image creation request object |
| os-volume_upload_image.image_name | Body | String | O | Image Name |
| os-volume_upload_image.force | Body | Boolean | - | Whether to allow image creation for volumes attached to instances<br>Default value is False |
| os-volume_upload_image.disk_format | Body | String | - | image disk format |
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
| os-volume_upload_image | Body | Object | Volume image creation response object |
| os-volume_upload_image.status | Body | String | Volume Status |
| os-volume_upload_image.image_name | Body | String | Image Name |
| os-volume_upload_image.disk_format | Body | String | image disk format |
| os-volume_upload_image.container_format | Body | String | Image container format |
| os-volume_upload_image.updated_at | Body | Datetime | Image modification time |
| os-volume_upload_image.image_id | Body | UUID | Image ID |
| os-volume_upload_image.display_description | Body | String | Volume description |
| os-volume_upload_image.id | Body | UUID | Volume ID |
| os-volume_upload_image.size | Body | Integer | Volume size (GB) |
| os-volume_upload_image.volume_type | Body | Object | Volume type information object |

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

| Status Name | Description |
|--|--|
| `creating` | Volume being created |
| `available` | Snapshot created and ready to use |
| `backing-up`| Snapshot is being backed up |
| `deleting`| Snapshot is being deleted |
| `error`| An error occurred during creation |
| `deleted`| deleted |
| `unmanaging`| Admin mode for snapshots being turned off |
| `restoring`| Restoring a volume from a snapshot |
| `error_deleting`| An error occurred while deleting |

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

| Name | Type | Property | Description |
|---|---|---|---|
| snapshots | Body | Array | Snapshot info object list |
| snapshots.status | Body | Enum | Snapshot status |
| snapshots.description | Body | String | Snapshot description |
| snapshots.created_at | Body | Datetime | Snapshot creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`format |
| snapshots.metadata | Body | Object | Snapshot metadata object |
| snapshots.volume_id | Body | UUID | Snapshot source volume ID |
| snapshots.size | Body | Integer | Original volume size of snapshot (GB) |
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

| Name | Type | Format | Description |
|---|---|---|---|
| snapshots | Body | Array | Snapshot Details Object List |
| snapshots.status | Body | Enum | Snapshot status |
| snapshots.description | Body | String | Snapshot description |
| snapshots.os-extended-snapshot-attributes:progress | Body | String | Snapshot creation progress |
| snapshots.created_at | Body | Datetime | Snapshot creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS`format |
| snapshots.metadata | Body | Object | Snapshot metadata object |
| snapshots.volume_id | Body | UUID | Snapshot source volume ID |
| snapshots.os-extended-snapshot-attributes:project_id | Body | String | Tenant ID |
| snapshots.size | Body | Integer | Original volume size of snapshot (GB) |
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
| snapshot.volume_id | Body | UUID | Snapshot source volume ID |
| snapshot.os-extended-snapshot-attributes:project_id | Body | String | Tenant ID |
| snapshot.size | Body | Integer | Original volume size of snapshot (GB) |
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

### Create Snapshots
Creates a snapshot of the specified volume.

```
POST /v2/{tenantId}/snapshots
X-Auth-Token: {tokenId}
```

#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |
| snapshot | Body | Object | O | Snapshot creation request object |
| snapshot.volume_id | Body | UUID | O | Original volume id |
| snapshot.force | Body | Boolean | - | Whether to force snapshot<br>If `true`, create a snapshot even if the volume is attached |
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
| snapshot.volume_id | Body | UUID | Snapshot source volume ID |
| snapshot.size | Body | Integer | Original volume size of snapshot (GB) |
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
