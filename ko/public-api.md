## API 버전

### 버전 목록 보기

TOAST 기본 인프라 서비스 Volume API에서 지원하는 버전 목록을 확인할 수 있습니다.

```
GET /
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

#### 응답
<details><summary>펼쳐 보기</summary>
<p>

```json
{
  "versions": [
    {
      "status": "SUPPORTED",
      "updated": "2014-06-28T12:20:21Z",
      "id": "v1.0",
      "links": [
        {
          "href": "http://10.162.50.101:8776/v1/",
          "rel": "self"
        }
      ]
    },
    {
      "status": "CURRENT",
      "updated": "2012-11-21T11:33:21Z",
      "id": "v2.0",
      "links": [
        {
          "href": "http://10.162.50.101:8776/v2/",
          "rel": "self"
        }
      ]
    }
  ]
}
```

</p>
</details>

## 볼륨 타입
### 볼륨 타입 목록 보기
```
GET /v2/{projectid}/types
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URI | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| volume_types | Body | Array | 볼륨 타입 객체 목록 |
| volume_types.id | Body | UUID | 볼륨 타입 ID |
| volume_types.name | Body | String | 볼륨 타입 이름 |
| volume_types.extra_specs | Body | Object | 볼륨 타입 관련 추가 사양 정보 객체 |

#### 예제
<details><summary>펼쳐 보기</summary>
<p>

```json
{
  "volume_types": [
    {
      "os-volume-type-access:is_public": true,
      "extra_specs": {
        "volume_backend_name": "hdd_general"
      },
      "id": "3d4736f6-6424-48f9-9c15-93b658ebff35",
      "name": "General HDD",
      "description": null
    }
  ]
}
```

</p>
</details>

## 볼륨
### 볼륨 상태
볼륨은 다양한 상태를 가지며, 상태에 따라 취할 수 있는 동작이 정해져 있습니다. 가능한 상태 목록은 다음과 같습니다.

| 상태 명 | 설명 |
|--|--|
| `creating` | 생성 중인 상태 |
| `available` | 볼륨이 생성되어 연결할 준비가 된 상태 |
| `attaching`| 볼륨이 인스턴스에 연결 중인 상태 |
| `detaching`| 볼륨이 연결 해제 중인 상태 |
| `in-use`| 볼륨이 인스턴스에 연결된 상태 |
| `maintenance`| 볼륨이 다른 호스트 장비로 이전되는 상태 |
| `deleting`| 볼륨이 삭제 중인 상태 |
| `awaiting-transfer`| 볼륨이 전송을 기다리는 상태 |
| `error`| 볼륨 생성시 오류가 발생한 상태 |
| `error_deleting`| 볼륨 삭제시 오류가 발생한 상태 |
| `backing-up`| 볼륨이 백업 중인 상태 |
| `restoring-backup`| 볼륨이 백업본으로 부터 복구되는 상태 |
| `error_backing-up`| 백업 중 오류가 발생한 상태 |
| `error_restoring`| 복구 중 오류가 발생한 상태 |
| `error_extending`| 볼륨 확장 중 오류가 발생한 상태 |
| `downloading`| 볼륨 생성시 지정한 이미지를 다운받는 상태 |
| `uploading`| 이미지 생성시 볼륨의 이미지를 업로드하는 상태 |
| `retyping`| 볼륨 타입을 변경하는 중인 상태 |
| `extending`| 볼륨을 확장하는 상태 |

### 볼륨 목록 보기
현재 테넌트에 속한 볼륨 목록을 반환합니다.

```
GET /v2/{projectId}/volumes
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URI | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |
| sort | Query | String | - | 정렬 기준이될 볼륨 필드 이름<br>`< key >[: < direction > ]` 형태로 기술<br>예) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | 반환할 볼륨의 갯수<br>기본 값은 1000으로 설정 |
| offset | Query | Integer | - | 반환될 목록의 시작점<br>전체 목록 중 offset 번째 볼륨부터 반환 |
| marker | Query | Integer | - | 반환할 볼륨들의 직전 볼륨 ID<br>정렬 순서에 따라 `marker`로 지정된 이미지 이후부터 `limit` 만큼 반환 |

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| volumes | Body | Array | 볼륨 객체 목록 |
| volumes.id | Body | UUID | 볼륨 ID |
| volumes.links | Body | Object | 볼륨 리소스 링크 레퍼런스 객체 |
| volumes.name | Body | String | 볼륨 이름 |
| volumes_links  | Body | Object | 페이지네이션을 위한 정보 객체<br>`limit`, `offset`를 추가한 경우 반환<br>다음 목록을 가리키는 경로를 포함 |

#### 예제
<details><summary>펼쳐 보기</summary>
<p>

```json
{
  "volumes": [
      {
          "id": "45baf976-c20a-4894-a7c3-c94b7376bf55",
          "links": [
              {
                  "href": "http://localhost:8776/v2/0c2eba2c5af04d3f9e9d0d410b371fde/volumes/45baf976-c20a-4894-a7c3-c94b7376bf55",
                  "rel": "self"
              },
              {
                  "href": "http://localhost:8776/0c2eba2c5af04d3f9e9d0d410b371fde/volumes/45baf976-c20a-4894-a7c3-c94b7376bf55",
                  "rel": "bookmark"
              }
          ],
          "name": "vol-004"
      },
      {
          "id": "5aa119a8-d25b-45a7-8d1b-88e127885635",
          "links": [
              {
                  "href": "http://localhost:8776/v2/0c2eba2c5af04d3f9e9d0d410b371fde/volumes/5aa119a8-d25b-45a7-8d1b-88e127885635",
                  "rel": "self"
              },
              {
                  "href": "http://localhost:8776/0c2eba2c5af04d3f9e9d0d410b371fde/volumes/5aa119a8-d25b-45a7-8d1b-88e127885635",
                  "rel": "bookmark"
              }
          ],
          "name": "vol-003"
      }
  ],
  "volumes_links": [{
      "href": "https://158.69.65.111/volume/v2/4ad9f06ab8654e40befa59a2e7cac86d/volumes/detail?limit=1&marker=3b451d5d-9358-4a7e-a746-c6fd8b0e1462",
      "rel": "next"
  }]
}
```

</p>
</details>

### 볼륨 상세 목록 보기
현재 테넌트에 속한 볼륨 목록을 반환합니다.

```
GET /v2/{projectId}/volumes/detail
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URI | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |
| sort | Query | String | - | 정렬 기준이될 볼륨 필드 이름<br>`< key >[: < direction > ]` 형태로 기술<br>예) `name:asc`, `created_at:desc` |
| limit | Query | Integer | - | 반환할 볼륨의 갯수<br>기본 값은 1000으로 설정 |
| offset | Query | Integer | - | 반환될 목록의 시작점<br>전체 목록 중 offset 번째 볼륨부터 반환 |
| marker | Query | Integer | - | 반환할 볼륨들의 직전 볼륨 ID<br>정렬 순서에 따라 `marker`로 지정된 이미지 이후부터 `limit` 만큼 반환 |

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| volumes | Body | Array | 볼륨 상세 정보 객체 목록 |
| volumes.attachments | Body | Object | 볼륨 연결 정보 객체 |
| volumes.attachments.server_id | Body | UUID | 볼륨이 연결된 인스턴스 ID |
| volumes.attachments.attachment_id | Body | UUID | 볼륨 연결 ID |
| volumes.attachments.volume_id | Body | UUID | 볼륨 ID |
| volumes.attachments.device_id | Body | String | 인스턴스내 장치 이름 |
| volumes.attachments.id | Body | String | 볼륨 ID |
| volumes.links | Body | Object | 볼륨 리소스 링크 레퍼런스 객체 |
| volumes.availability_zone | Body | String | 볼륨 가용성 영역 |
| volumes.encrypted | Body | Boolean | 볼륨 암호화 여부 |
| volumes.os-volume-replication:extended_status | Body | String | 볼륨 확장 상태 |
| volumes.volume_type | Body | String | 볼륨 타입 이름 |
| volumes.snapshot_id | Body | UUID | 볼륨 생성시 지정한 Snapshot ID |
| volumes.id | Body | UUID | 볼륨 ID |
| volumes.size | Body | Integer | 볼륨 크기 |
| volumes.user_id | Body | String | 볼륨 소유주 ID |
| volumes.os-vol-tenant-attr:tenant_id | Body | String | 테넌트 ID |
| volumes.metadata | Body | Object | 볼륨 메타데이터 객체 |
| volumes.status | Body | Enum | 볼륨 상태 |
| volumes.description | Body | String | 볼륨 설명 |
| volumes.multiattach | Body | Boolean | 다중 연결 가능 여부<br>`true`로 설정하면<br>여러 인스턴스에 동시에 연결할 수 있음 |
| volumes.source_volid | Body | UUID | 볼륨 생성시 지정한 Volume ID |
| volumes.consistencygroup_id | Body | UUID | 볼륨 Consistency 그룹 ID |
| volumes.name | Body | UUID | 볼륨 이름 |
| volumes.bootable | Body | Boolean | 볼륨 부팅 가능 여부 |
| volumes.created_at | Body | Datetime | 볼륨 생성 시각<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` |
| volumes.os-volume-replication:driver_data | Body | String | 볼륨 복제 데이터 |
| volumes.replication_status | Body | String | 볼륨 복제 상태 |
| volumes.volumes_links  | Body | Object | 페이지네이션을 위한 정보 객체<br>`limit`, `offset`를 추가한 경우 반환<br>다음 목록을 가리키는 경로를 포함 |

#### 예제
<details><summary>펼쳐 보기</summary>
<p>

```json
{
  "volumes_links": [
    {
      "href": "http://alp-gov-cinder.iaas.tcc1.cloud.toastoven.net:8776/v2/19eeb40d58684543aef29cbb5ebfe8f0/volumes/detail?limit=1&marker=6f96cef5-0b27-4f97-bea4-3ff7bb8d7b8c",
      "rel": "next"
    }
  ],
  "volumes": [
    {
      "attachments": [
        {
          "server_id": "36e4a785-432f-4872-9248-cfe981155c52",
          "attachment_id": "1c760212-dba3-484c-afe5-d9da10d09029",
          "host_name": null,
          "volume_id": "6f96cef5-0b27-4f97-bea4-3ff7bb8d7b8c",
          "device": "/dev/vda",
          "id": "6f96cef5-0b27-4f97-bea4-3ff7bb8d7b8c"
        }
      ],
      "links": [
        {
          "href": "http://alp-gov-cinder.iaas.tcc1.cloud.toastoven.net:8776/v2/19eeb40d58684543aef29cbb5ebfe8f0/volumes/6f96cef5-0b27-4f97-bea4-3ff7bb8d7b8c",
          "rel": "self"
        },
        {
          "href": "http://alp-gov-cinder.iaas.tcc1.cloud.toastoven.net:8776/19eeb40d58684543aef29cbb5ebfe8f0/volumes/6f96cef5-0b27-4f97-bea4-3ff7bb8d7b8c",
          "rel": "bookmark"
        }
      ],
      "availability_zone": "kr-pub-b",
      "encrypted": false,
      "os-volume-replication:extended_status": null,
      "volume_type": "General HDD",
      "snapshot_id": null,
      "id": "6f96cef5-0b27-4f97-bea4-3ff7bb8d7b8c",
      "size": 20,
      "user_id": "1f056df3ce654bc19a12bd522d6a2147",
      "os-vol-tenant-attr:tenant_id": "19eeb40d58684543aef29cbb5ebfe8f0",
      "metadata": {
        "readonly": "False",
        "attached_mode": "rw"
      },
      "status": "in-use",
      "volume_image_metadata": {
        "tc_env": "sysmon",
        "project_domain": "WDI;NORMAL",
        "container_format": "bare",
        "min_ram": "0",
        "disk_format": "qcow2",
        "image_name": "CentOS 6.10 (2020.02.18)",
        "image_id": "cd632922-d08a-4c20-9e0b-3f90350aee1c",
        "os_architecture": "amd64",
        "min_disk": "20",
        "login_username": "centos",
        "os_type": "linux",
        "description": "CentOS 6.10 (2020.02.18)",
        "os_distro": "CentOS",
        "hypervisor_type": "qemu",
        "release_date": "2020.02.18",
        "monitoring_agent": "sysmon",
        "os_version": "6.10",
        "checksum": "63570f149c62be0914235206cef6bcb4",
        "deprecate_date": "",
        "size": "2199846912"
      },
      "description": null,
      "multiattach": false,
      "source_volid": null,
      "consistencygroup_id": null,
      "name": null,
      "bootable": "true",
      "created_at": "2020-03-02T12:24:54.000000",
      "os-volume-replication:driver_data": null,
      "replication_status": "disabled"
    }
  ]
}
```

</p>
</details>

### 볼륨 보기
지정한 볼륨에 대한 상세 정보를 반환합니다.

```
GET /v2/{projectId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URI | String | O | 테넌트 ID |
| volumeId | URI | UUID | O | 볼륨 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| volume | Body | Object | 볼륨 상세 정보 객체 |
| volume.attachments | Body | Object | 볼륨 연결 정보 객체 |
| volume.attachments.server_id | Body | UUID | 볼륨이 연결된 인스턴스 ID |
| volume.attachments.attachment_id | Body | UUID | 볼륨 연결 ID |
| volume.attachments.volume_id | Body | UUID | 볼륨 ID |
| volume.attachments.device_id | Body | String | 인스턴스내 장치 이름 |
| volume.attachments.id | Body | String | 볼륨 ID |
| volume.links | Body | Object | 볼륨 리소스 링크 레퍼런스 객체 |
| volume.availability_zone | Body | String | 볼륨 가용성 영역 |
| volume.encrypted | Body | Boolean | 볼륨 암호화 여부 |
| volume.os-volume-replication:extended_status | Body | String | 볼륨 확장 상태 |
| volume.volume_type | Body | String | 볼륨 타입 이름 |
| volume.snapshot_id | Body | UUID | 볼륨 생성시 지정한 Snapshot ID |
| volume.id | Body | UUID | 볼륨 ID |
| volume.size | Body | Integer | 볼륨 크기 |
| volume.user_id | Body | String | 볼륨 소유주 ID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | 테넌트 ID |
| volume.metadata | Body | Object | 볼륨 메타데이터 객체 |
| volume.status | Body | Enum | 볼륨 상태 |
| volume.description | Body | String | 볼륨 설명 |
| volume.multiattach | Body | Boolean | 다중 연결 가능 여부<br>`true`로 설정하면<br>여러 인스턴스에 동시에 연결할 수 있음 |
| volume.source_volid | Body | UUID | 볼륨 생성시 지정한 Volume ID |
| volume.consistencygroup_id | Body | UUID | 볼륨 Consistency 그룹 ID |
| volume.name | Body | UUID | 볼륨 이름 |
| volume.bootable | Body | Boolean | 볼륨 부팅 가능 여부 |
| volume.created_at | Body | Datetime | 볼륨 생성 시각<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` |
| volume.os-volume-replication:driver_data | Body | String | 볼륨 복제 데이터 |
| volume.replication_status | Body | String | 볼륨 복제 상태 |

#### 예제
<details><summary>펼쳐 보기</summary>
<p>

```json
{
    "volume": {
        "status": "available",
        "attachments": [],
        "links": [
            {
                "href": "http://localhost:8776/v2/0c2eba2c5af04d3f9e9d0d410b371fde/volumes/5aa119a8-d25b-45a7-8d1b-88e127885635",
                "rel": "self"
            },
            {
                "href": "http://localhost:8776/0c2eba2c5af04d3f9e9d0d410b371fde/volumes/5aa119a8-d25b-45a7-8d1b-88e127885635",
                "rel": "bookmark"
            }
        ],
        "availability_zone": "nova",
        "bootable": "false",
        "os-vol-host-attr:host": "ip-10-168-107-25",
        "source_volid": null,
        "snapshot_id": null,
        "id": "5aa119a8-d25b-45a7-8d1b-88e127885635",
        "description": "Super volume.",
        "name": "vol-002",
        "created_at": "2013-02-25T02:40:21.000000",
        "volume_type": "None",
        "os-vol-tenant-attr:tenant_id": "0c2eba2c5af04d3f9e9d0d410b371fde",
        "size": 1,
        "metadata": {
            "contents": "not junk"
        }
    }
}
```

</p>
</details>

### 볼륨 생성하기
새로운 볼륨을 생성합니다. 이미지 또는 스냅숏으로부터 새로운 볼륨을 생성할 수 있습니다.
볼륨은 생성 직후 즉시 사용할 수 없습니다. 볼륨 상태를 조회해서 `available` 상태로 전이하는 것을 확인한 후 사용할 수 있습니다.

```
POST /v2/{projectId}/volumes/detail
X-Auth-Token: {tokenId}
```

#### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URI | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |
| volume | Body | Object | O | 볼륨 생성 요청 객체 |
| volume.size | Body | Integer | O | 볼륨 크기 |
| volume.description | Body | String | - | 볼륨 설명 |
| volume.imageRef | Body | String | - | 원본 이미지 ID |
| volume.multiattach | Body | Boolean | - | 다중 연결 가능 여부<br>`true`로 설정하면<br>여러 인스턴스에 동시에 연결할 수 있음 |
| volume.availability_zone | Body | String | - | 볼륨 가용성 영역 이름 |
| volume.source_volid | Body | UUID | - | 원본 볼륨 ID |
| volume.name | Body | String | - | 볼륨 이름 |
| volume.consistencygroup_id | Body | UUID | - | 볼륨 Consistency 그룹 ID |
| volume.volume_type | Body | String | - | 볼륨 타입 이름 |
| volume.snapshot_id | Body | UUID | - | 원본 스냅숏 ID |
| volume.metadata | Body | Object | - | 볼륨 메타데이터 객체 |

#### 예제
<details><summary>펼쳐 보기</summary>
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

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| volume | Body | Object | 볼륨 상세 정보 객체 |
| volume.attachments | Body | Object | 볼륨 연결 정보 객체 |
| volume.attachments.server_id | Body | UUID | 볼륨이 연결된 인스턴스 ID |
| volume.attachments.attachment_id | Body | UUID | 볼륨 연결 ID |
| volume.attachments.volume_id | Body | UUID | 볼륨 ID |
| volume.attachments.device_id | Body | String | 인스턴스내 장치 이름 |
| volume.attachments.id | Body | String | 볼륨 ID |
| volume.links | Body | Object | 볼륨 리소스 링크 레퍼런스 객체 |
| volume.availability_zone | Body | String | 볼륨 가용성 영역 |
| volume.encrypted | Body | Boolean | 볼륨 암호화 여부 |
| volume.os-volume-replication:extended_status | Body | String | 볼륨 확장 상태 |
| volume.volume_type | Body | String | 볼륨 타입 이름 |
| volume.snapshot_id | Body | UUID | 볼륨 생성시 지정한 Snapshot ID |
| volumes.id | Body | UUID | 볼륨 ID |
| volume.size | Body | Integer | 볼륨 크기 |
| volume.user_id | Body | String | 볼륨 소유주 ID |
| volume.os-vol-tenant-attr:tenant_id | Body | String | 테넌트 ID |
| volume.metadata | Body | Object | 볼륨 메타데이터 객체 |
| volume.status | Body | Enum | 볼륨 상태 |
| volume.description | Body | String | 볼륨 설명 |
| volume.multiattach | Body | Boolean | 여러 인스턴스에 연결 가능 여부 |
| volume.source_volid | Body | UUID | 볼륨 생성시 지정한 Volume ID |
| volume.consistencygroup_id | Body | UUID | 볼륨 Consistency 그룹 ID |
| volume.name | Body | UUID | 볼륨 이름 |
| volume.bootable | Body | Boolean | 볼륨 부팅 가능 여부 |
| volume.created_at | Body | Datetime | 볼륨 생성 시각<br>`YYYY-MM-DDThh:mm:ss.SSSSSS` |
| volume.os-volume-replication:driver_data | Body | String | 볼륨 복제 데이터 |
| volume.replication_status | Body | String | 볼륨 복제 상태 |

#### 예제
<details><summary>펼쳐 보기</summary>
<p>

```json
{
    "volume": {
        "status": "creating",
        "migration_status": null,
        "user_id": "0eea4eabcf184061a3b6db1e0daaf010",
        "attachments": [],
        "links": [
            {
                "href": "http://23.253.248.171:8776/v2/bab7d5c60cd041a0a36f7c4b6e1dd978/volumes/6edbc2f4-1507-44f8-ac0d-eed1d2608d38",
                "rel": "self"
            },
            {
                "href": "http://23.253.248.171:8776/bab7d5c60cd041a0a36f7c4b6e1dd978/volumes/6edbc2f4-1507-44f8-ac0d-eed1d2608d38",
                "rel": "bookmark"
            }
        ],
        "availability_zone": "nova",
        "bootable": "false",
        "encrypted": false,
        "created_at": "2015-11-29T03:01:44.000000",
        "description": null,
        "updated_at": null,
        "volume_type": "lvmdriver-1",
        "name": "test-volume-attachments",
        "replication_status": "disabled",
        "consistencygroup_id": null,
        "source_volid": null,
        "snapshot_id": null,
        "multiattach": false,
        "metadata": {},
        "id": "6edbc2f4-1507-44f8-ac0d-eed1d2608d38",
        "size": 2
    }
}
```

</p>
</details>

### 볼륨 삭제하기
지정한 볼륨을 삭제합니다. 연결되어 있거나 스냅샷이 생성된 볼륨은 삭제할 수 없습니다.

```
GET /v2/{projectId}/volumes/{volumeId}
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URI | String | O | 테넌트 ID |
| volumeId | URI | String | O | 볼륨 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답
이 API는 응답 본문을 반환하지 않습니다.


## 스냅숏
### 스냅숏 상태
스냅숏은 다양한 상태를 가지며, 상태에 따라 취할 수 있는 동작이 정해져 있습니다. 가능한 상태 목록은 다음과 같습니다.

| 상태 명 | 설명 |
|--|--|
| `creating` | 생성 중인 상태 |
| `available` | 스냅숏이 생성되어 사용할 준비가 된 상태 |
| `backing-up`| 스냅숏이 백업 중인 상태 |
| `deleting`| 스냅숏이 삭제 중인 상태 |
| `error`| 생성 중 오류가 발생한 상태 |
| `deleted`| 삭제된 상태 |
| `unmanaging`| 스냅숏에 대한 관리 모드 해제 중인 상태 |
| `restoring`| 스냅숏으로부터 볼륨을 복원 중인 상태 |
| `error_deleting`| 삭제 중 오류가 발생한 상태 |

### 스냅숏 목록 보기
스냅숏 목록을 반환합니다.

```
GET /v2/{projectId}/snapshots
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URI | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| snapshots | Body | Array | 스냅숏 정보 객체 목록 |
| snapshots.status | Body | Enum | 스냅숏 상태 |
| snapshots.description | Body | String | 스냅숏 설명 |
| snapshots.created_at | Body | Datetime | 스냅숏 생성 시간<br>YYYY-MM-DDThh:mm:ss.SSSSSS |
| snapshots.metadata | Body | Object | 스냅숏 메타데이터 객체 |
| snapshots.volume_id | Body | UUID | 스냅숏의 원본 볼륨 ID |
| snapshots.size | Body | Integer | 스냅숏의 원본 볼륨 크기 (GB) |
| snapshots.id | Body | UUID | 스냅숏 ID |
| snapshots.name | Body | String | 스냅숏 이름 |

#### 예제
<details><summary>펼쳐 보기</summary>
<p>

```json
{
  "snapshots": [
    {
      "status": "available",
      "description": "",
      "created_at": "2019-07-30T02:05:13.000000",
      "metadata": {},
      "volume_id": "13461857-3ac4-42f3-b821-3eb9cddf0cbf",
      "size": 20,
      "id": "efb57271-2967-4e0f-bfcd-e14fa7c6acd7",
      "name": "jj-img-hdd-snapshot"
    }
  ]
}
```

</p>
</details>

### 스냅숏 목록 상세 보기
스냅숏 상세 정보 목록을 반환합니다.

```
GET /v2/{projectId}/snapshots/detail
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URI | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| snapshots | Body | Array | 스냅숏 상세 정보 객체 목록 |
| snapshots.status | Body | Enum | 스냅숏 상태 |
| snapshots.description | Body | String | 스냅숏 설명 |
| snapshots.os-extended-snapshot-attributes:progress | Body | String | 스냅숏 생성 진행 상태 |
| snapshots.created_at | Body | Datetime | 스냅숏 생성 시간<br>YYYY-MM-DDThh:mm:ss.SSSSSS |
| snapshots.metadata | Body | Object | 스냅숏 메타데이터 객체 |
| snapshots.volume_id | Body | UUID | 스냅숏의 원본 볼륨 ID |
| snapshots.os-extended-snapshot-attributes:project_id | Body | String | 테넌트 ID |
| snapshots.size | Body | Integer | 스냅숏의 원본 볼륨 크기 (GB) |
| snapshots.id | Body | UUID | 스냅숏 ID |
| snapshots.name | Body | String | 스냅숏 이름 |

#### 예제
<details><summary>펼쳐 보기</summary>
<p>

```json
{
  "snapshots": [
    {
      "status": "available",
      "os-extended-snapshot-attributes:progress": "100%",
      "description": "",
      "created_at": "2019-07-30T02:05:13.000000",
      "metadata": {},
      "volume_id": "13461857-3ac4-42f3-b821-3eb9cddf0cbf",
      "os-extended-snapshot-attributes:project_id": "19eeb40d58684543aef29cbb5ebfe8f0",
      "size": 20,
      "id": "efb57271-2967-4e0f-bfcd-e14fa7c6acd7",
      "name": "jj-img-hdd-snapshot"
    }
  ]
}
```

</p>
</details>

### 스냅숏 보기
지정한 스냅숏에 대한 상세 정보를 반환합니다.

```
GET /v2/{projectId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URI | String | O | 테넌트 ID |
| snapshotId | URI | UUID | O | 스냅숏 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| snapshot | Body | Array | 스냅숏 상세 정보 객체 목록 |
| snapshot.status | Body | Enum | 스냅숏 상태 |
| snapshot.description | Body | String | 스냅숏 설명 |
| snapshot.os-extended-snapshot-attributes:progress | Body | String | 스냅숏 생성 진행 상태 |
| snapshot.created_at | Body | Datetime | 스냅숏 생성 시간<br>YYYY-MM-DDThh:mm:ss.SSSSSS |
| snapshot.metadata | Body | Object | 스냅숏 메타데이터 객체 |
| snapshot.volume_id | Body | UUID | 스냅숏의 원본 볼륨 ID |
| snapshot.os-extended-snapshot-attributes:project_id | Body | String | 테넌트 ID |
| snapshot.size | Body | Integer | 스냅숏의 원본 볼륨 크기 (GB) |
| snapshot.id | Body | UUID | 스냅숏 ID |
| snapshot.name | Body | String | 스냅숏 이름 |

#### 예제
<details><summary>펼쳐 보기</summary>
<p>

```json
{
  "snapshot": {
    "status": "available",
    "os-extended-snapshot-attributes:progress": "100%",
    "description": "",
    "created_at": "2019-07-30T02:05:13.000000",
    "metadata": {},
    "volume_id": "13461857-3ac4-42f3-b821-3eb9cddf0cbf",
    "os-extended-snapshot-attributes:project_id": "19eeb40d58684543aef29cbb5ebfe8f0",
    "size": 20,
    "id": "efb57271-2967-4e0f-bfcd-e14fa7c6acd7",
    "name": "jj-img-hdd-snapshot"
  }
}
```

</p>
</details>

### 스냅숏 생성하기
지정한 볼륨에 대한 스냅숏을 생성합니다.

```
POST /v2/{projectId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URI | String | O | 테넌트 ID |
| tokenId | Header | String | O | 토큰 ID |
| snapshot | Body | Object | O | 스냅숏 생성 요청 객체 |
| snapshot.volume_id | Body | UUID | O | 원본 볼륨 ID |
| snapshot.force | Body | Boolean | - | 강제 스냅숏 생성 여부<br>`true`이면 볼륨이 연결되어도 스냅숏을 생성 |
| snapshot.description | Body | String | - | 스냅숏 설명 |
| snapshot.name | Body | String | - | 스냅숏 이름 |

#### 예제
<details><summary>펼쳐 보기</summary>
<p>

```json
{
    "snapshot": {
        "name": "snap-001",
        "description": "Daily backup",
        "volume_id": "5aa119a8-d25b-45a7-8d1b-88e127885635",
        "force": true
    }
}
```

</p>
</details>

#### 응답

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| snapshot | Body | Array | 스냅숏 상세 정보 객체 목록 |
| snapshot.status | Body | Enum | 스냅숏 상태 |
| snapshot.description | Body | String | 스냅숏 설명 |
| snapshot.created_at | Body | Datetime | 스냅숏 생성 시간<br>YYYY-MM-DDThh:mm:ss.SSSSSS |
| snapshot.metadata | Body | Object | 스냅숏 메타데이터 객체 |
| snapshot.volume_id | Body | UUID | 스냅숏의 원본 볼륨 ID |
| snapshot.size | Body | Integer | 스냅숏의 원본 볼륨 크기 (GB) |
| snapshot.id | Body | UUID | 스냅숏 ID |
| snapshot.name | Body | String | 스냅숏 이름 |

#### 예제
<details><summary>펼쳐 보기</summary>
<p>

```json
{
  "snapshot": {
    "status": "creating",
    "description": null,
    "created_at": "2020-03-02T13:51:33.318695",
    "metadata": {},
    "volume_id": "13461857-3ac4-42f3-b821-3eb9cddf0cbf",
    "size": 20,
    "id": "f2ae2667-b3b7-47ef-aa31-efab536888b9",
    "name": "CCCCC"
  }
}
```

</p>
</details>

### 스냅숏 삭제하기
지정한 스냅숏을 삭제합니다.

```
GET /v2/{projectId}/snapshots/{snapshotId}
X-Auth-Token: {tokenId}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| projectId | URI | String | O | 테넌트 ID |
| snapshotId | URI | String | O | 스냅숏 ID |
| tokenId | Header | String | O | 토큰 ID |

#### 응답
이 API는 응답 본문을 반환하지 않습니다.
