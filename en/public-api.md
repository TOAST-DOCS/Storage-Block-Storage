## Storage > Block Storage > API v2 Guide

To enable APIs, API endpoint and token are required. Prepare information required to enable an API, in reference of  [Preparing for APIs](/Compute/Compute/ko/identity-api/)

Use `volumev2`-type endpoint for Block Storage API. For more details, see `serviceCatalog` from response of token issuance. 

| Type | Region | Endpoint |
|---|---|---|
| volumev2 | Korea (Pangyo) <br>Japan | https://kr1-api-block-storage-infrastructure.nhncloudservice.com<br>https://jp1-api-block-storage-infrastructure.nhncloudservice.com |

In API response, you may find fields that are not specified in the guide. Refrain from using them because such fields are only for the NHN Cloud internal usage and might be changed without previous notice. 

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

| Name | Type | Attribute | Description |
|---|---|---|---|
| volume_types | Body | Array | List of volume type objects |
| volume_types.id | Body | UUID | ID of volume type |
| volume_types.name | Body | String | Name of volume type |
| volume_types.os-volume-type-access:is_public | Body | Boolean | Publicize volume type or not |
| volume_types.description | Body | String | Description of volume type |
| volume_types.extra_specs | Body | Object | Information object of extra specifications related with volume type |

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
Volumes are available in many statuses with operations defined for each status. See the following list of available statuses:  

| Status Name | Description |
|--|--|
| `creating` | Creating a volume |
| `available` | Volume is created and ready for attachment |
| `attaching`| Attaching volume to an instance |
| `detaching` | Detaching volume |
| `in-use`| Volume is attached to an instance |
| `maintenance`| Volume is migrating to another host equipment |
| `deleting`| Deleting a volume |
| `awaiting-transfer`| Volume is waiting for transfer |
| `error`| Error occurred while creating a volume |
| `error_deleting`| Error occurred while deleting a volume |
| `backing-up`| Backing up volume |
| `restoring-backup`| Restoring volume from backup |
| `error_backing-up`| Error occurred while backing up |
| `error_restoring`| Error occurred while restoring |
| `error_extending`| Error occurred while extending volume |
| `downloading`| Downloading specified image while creating a volume |
| `uploading`| Uploading volume image while creating an image |
| `retyping`| Changing volume type |
| `extending`| Extending volume |

### List Volumes 
Return the list of volumes included to a current tenant. 

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
| sort | Query | String | - | Name of volume field for sorting<br>Described in the`< key >[: < direction > ]` format<br>e.g.) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | Volume count to return <br>Default is 1000 |
| offset | Query | Integer | - | Start point of the list to return<br>Return from the offset volume out of the entire list |
| marker | Query | UUID | - | ID of the previous volume of a volume to return <br>Return as much as the `limit` after volume specified as the `marker` according to the sorting order |

#### Response	

| Name | Type | Attributes | Description |
|---|---|---|---|
| volumes | Body | Array | List of volume objects |
| volumes.id | Body | UUID | Volume ID |
| volumes.links | Body | Object | Reference objects for volume resource links |
| volumes.name | Body | String | Volume name |
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

### List Volume Details 
Return the list of volumes included to a current tenant. 

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
| sort | Query | String | - | Name of volume field for sorting<br>Described in the`< key >[: < direction > ]` format<br>e.g.) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | Volume count to return<br>Default is 1000 |
| offset | Query | Integer | - | Start point of the list to return <br/>Return from the offset volume out of the entire list |
| marker | Query | UUID | - | ID of the previous volume of a volume to return <br/>Return as much as `limit` after volume specified as the `marker` according to the sorting order |

#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| volumes | Body | Array | List of detail volume information objects |
| volumes.attachments | Body | Object | Information object for volume attachment |
| volumes.attachments.server_id | Body | UUID | ID of instance with volume attached |
| volumes.attachments.attachment_id | Body | UUID | ID of volume attachment |
| volumes.attachments.volume_id | Body | UUID | Volume ID |
| volumes.attachments.device | Body | String | Name of device within instance |
| volumes.attachments.id | Body | String | Volume ID |
| volumes.links | Body | Object | Reference object of volume resource link |
| volumes.availability_zone | Body | String | Volume availability area |
| volumes.encrypted | Body | Boolean | Volume encrypted or not |
| volumes.os-volume-replication:extended_status | Body | String | Volume extension status |
| volumes.volume_type | Body | String | Name of volume type |
| volumes.snapshot_id | Body | UUID | Snapshot ID specified while creating a volume |
| volumes.id | Body | UUID | Volume ID |
| volumes.size | Body | Integer | Volume size(GB) |
| volumes.user_id | Body | String | ID of volume owner |
| volumes.os-vol-tenant-attr:tenant_id | Body | String | Tenant ID |
| volumes.metadata | Body | Object | Volume metadata object |
| volumes.status | Body | Enum | Volume status |
| volumes.description | Body | String | Volume description |
| volumes.multiattach | Body | Boolean | Multiple attachment or not<br>With `true`, multiple attachment to many instances are available |
| volumes.source_volid | Body | UUID | Volume ID specified when creating a volume |
| volumes.consistencygroup_id | Body | UUID | ID of volume group |
| volumes.name | Body | String | Name of volume |
| volumes.bootable | Body | Boolean | Volume bootable or not |
| volumes.created_at | Body | Datetime | Time of volume creation <br>In the format of `YYYY-MM-DDThh:mm:ss.SSSSSS` |
| volumes.os-volume-replication:driver_data | Body | String | Volume replication data |
| volumes.replication_status | Body | String | Volume replication status |
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

### Get Volume 
Return details of specified volume.  

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
| volume | Body | Object | Detail information object of volume |
| volume.attachments | Body | Object | Information object for volume attachment |
| volume.attachments.server_id | Body | UUID | ID of instance attached to volume |
| volume.attachments.attachment_id | Body | UUID | ID of volume attachment |
| volume.attachments.volume_id | Body | UUID | Volume ID |
| volume.attachments.device | Body | String | Name of equipment within instance |
| volume.attachments.id | Body | String | Volume ID |
| volume.links | Body | Object | Reference object for volume resource link |
| volume.availability_zone | Body | String | Volume availability area |
| volume.encrypted | Body | Boolean | Volume encrypted or not |
| volume.os-volume-replication:extended_status | Body | String | Volume extended status |
| volume.volume_type | Body | String | Volume type name |
| volume.snapshot_id | Body | UUID | Snapshot ID specified when creating volume |
| volume.id | Body | UUID | Volume ID |
| volume.size | Body | Integer | Volume size (GB) |
| volume.user_id | Body | String | Volume owner ID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | Tenant ID |
| volume.metadata | Body | Object | Volume metadata object |
| volume.status | Body | Enum | Volume status |
| volume.description | Body | String | Volume description |
| volume.multiattach | Body | Boolean | Multiple attachment is available or not<br>With `true`, simultaneous attachment to many instances is enabled |
| volume.source_volid | Body | UUID | Volume ID specified when creating volume |
| volume.consistencygroup_id | Body | UUID | Volume consistency volume ID |
| volume.name | Body | String | Volume name |
| volume.bootable | Body | Boolean | Volume bootable |
| volume.created_at | Body | Datetime | Volume creation time<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` |
| volume.os-volume-replication:driver_data | Body | String | Volume replication data |
| volume.replication_status | Body | String | Volume replication status |

<details><summary>Example</summary>
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

### Create Volume
Create a new or empty volume from snapshot. 

Volumes are not immediately available after created. Query volume status and check if it is `available`  first.  

```
POST /v2/{tenantId}/volumes
X-Auth-Token: {tokenId}
```

#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |
| volume | Body | Object | O | Object requesting of creating volume |
| volume.size | Body | Integer | O | Volume size (GB) |
| volume.description | Body | String | - | Volume description |
| volume.availability_zone | Body | String | - | Name of volume availability area |
| volume.name | Body | String | - | Volume name |
| volume.volume_type | Body | String | - | Volume type name |
| volume.snapshot_id | Body | UUID | - | Original snapshot ID: if left blank, empty volume is created |
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

| Name | Type | Attributes | Description |
|---|---|---|---|
| volume | Body | Object | Information object for volume details |
| volume.attachments | Body | Object | Information object for volume attachment |
| volume.links | Body | Object | Reference object for volume resource links |
| volume.availability_zone | Body | String | Volume visibility area |
| volume.encrypted | Body | Boolean | Volume encrypted or not |
| volume.os-volume-replication:extended_status | Body | String | Volume extended status |
| volume.volume_type | Body | String | Volume type name |
| volume.snapshot_id | Body | UUID | Snapshot ID specified for creating volume |
| volumes.id | Body | UUID | Volume ID |
| volume.size | Body | Integer | Volume size (GB) |
| volume.user_id | Body | String | Volume owner ID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | Tenant ID |
| volume.metadata | Body | Object | Volume metadata object |
| volume.status | Body | Enum | Volume status |
| volume.description | Body | String | Volume description |
| volume.multiattach | Body | Boolean | Attachable to many instances |
| volume.consistencygroup_id | Body | UUID | Volume consistency group ID |
| volume.name | Body | String | Volume name |
| volume.bootable | Body | Boolean | Volume bootable |
| volume.created_at | Body | Datetime | Volume creation time<br>In the`YYYY-MM-DDThh:mm:ss.SSSSSS` format |
| volume.os-volume-replication:driver_data | Body | String | Volume replication data |
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

### Delete Volume 

Delete specified volume. Volumes that are attached or with snapshots created cannot be deleted. 

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

### Create Image with Volume
Create image from volume. 

At least 100KB space is required for basic initialization after image is created. If you have less space than that, initialization may fail. 

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
| os-volume_upload_image | Body | Object | O | Object requesting of creating volume image |
| os-volume_upload_image.image_name | Body | String | O | Image name |
| os-volume_upload_image.force | Body | Boolean | - | Image creation allowed or not for volumes attached to instance <br>Default is false |
| os-volume_upload_image.disk_format | Body | String | - | Image disk format |
| os-volume_upload_image.container_format | Body | String | - | Image container format |
| os-volume_upload_image.visibility | Body | String | - | Image visibility<br>Either`private`, or `shared` |
| os-volume_upload_image.protected | Body | Boolean | - | Image protected or not</br>Unable to modify or delete, if protected=true |

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

| Name | Type | Format | Description |
|---|---|---|---|
| os-volume_upload_image | Body | Object | Response object creating volume image |
| os-volume_upload_image.status | Body | String | Volume status |
| os-volume_upload_image.image_name | Body | String | Image name |
| os-volume_upload_image.disk_format | Body | String | Image disk format |
| os-volume_upload_image.container_format | Body | String | Image container format |
| os-volume_upload_image.updated_at | Body | Datetime | Image modification time |
| os-volume_upload_image.image_id | Body | UUID | Image ID |
| os-volume_upload_image.display_description | Body | String | Volume description |
| os-volume_upload_image.id | Body | UUID | Volume ID |
| os-volume_upload_image.size | Body | Integer | Volume size (GB) |
| os-volume_upload_image.volume_type | Body | Object | Information object for volume type |

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
### Snapshot Status
Snapshots exist in various statues, and each status defines operations.  See the list of available statuses like below:

| Status Name | Description |
|--|--|
| `creating` | Creating a snapshot |
| `available` | Snapshot has been created and is available |
| `backing-up      | Backing up a snapshot                               |
| `deleting`| Deleting a snapshot |
| `error`| Error has occurred while creating a snapshot |
| `deleted`| Snapshot has been deleted |
| `unmanaging`| Snapshot has been released from the management mode |
| `restoring`| Restoring volume from snapshot |
| `error_deleting`| Error has occurred while deleting a snapshot |

### List Snapshots 
Return the list of snapshots. 

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
| snapshot.volume_id   | Body | UUID     | Original volume ID of snapshot                               |
| snapshot.size        | Body | Integer  | Original volume size of snapshot (GB)                        |
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

### List Snapshot Details 
Return the list of snapshot details. 

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
| snapshot.volume_id                                  | Body | UUID     | Original volume ID of snapshot                               |
| snapshot.os-extended-snapshot-attributes:project_id | Body | String   | Tenant ID                                                    |
| snapshot.size                                       | Body | Integer  | Original volume size of snapshot (GB)                        |
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

### Get Snapshot
Return details of specified snapshot. 

```
GET /v2/{tenantId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### Request
This API does not require a request body. .

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| snapshotId | URL | UUID | O | Snapshot ID |
| tokenId | Header | String | O | Token ID |

#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| snapshot | Body | Object | Information object of snapshot details |
| snapshot.status | Body | Enum | Snapshot status |
| snapshot.description | Body | String | Snapshot description |
| snapshot.os-extended-snapshot-attributes:progress | Body | String | Progress of snapshot creation |
| snapshot.created_at | Body | Datetime | Snapshot creation time<br>In the`YYYY-MM-DDThh:mm:ss.SSSSSS` format |
| snapshot.metadata | Body | Object | Snapshot metadata object |
| snapshot.volume_id | Body | UUID | Original volume ID of snapshot |
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

### Create Snapshot
Create snapshot for specified volume. 

```
POST /v2/{tenantId}/snapshots
X-Auth-Token: {tokenId}
```

#### Request 

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | URL | String | O | Tenant ID |
| tokenId | Header | String | O | Token ID |
| snapshot | Body | Object | O | Object requesting of creating snapshot |
| snapshot.volume_id | Body | UUID | O | Original volume ID |
| snapshot.force | Body | Boolean | - | Forced to create snapshot or not<br>With`true`, snapshot is created even if volume is attached |
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
| snapshot | Body | Object | Information object of snapshot details |
| snapshot.status | Body | Enum | Snapshot status |
| snapshot.description | Body | String | Snapshot description |
| snapshot.created_at | Body | Datetime | Snapshot creation time<br>In the`YYYY-MM-DDThh:mm:ss.SSSSSS` format |
| snapshot.metadata | Body | Object | Snapshot metadata object |
| snapshot.volume_id | Body | UUID | Original volume ID of snapshot |
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

### Delete Snapshot
Delete snapshot as specified. 

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
This API does not require a request body. 
