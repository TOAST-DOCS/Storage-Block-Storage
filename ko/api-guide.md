## Storage > Block Storage > API 가이드

블록 스토리지 API를 사용하려면 토큰 발급과 같은 사전 준비가 필요합니다. [API 사용 준비](/Infrastructure%20Common/ko/api-guide/)를 참조합니다.

## 블록 스토리지 API

블록 스토리지 생성/삭제 및 조회 기능을 제공합니다. 블록 스토리지를 인스턴스에 연결/해제하는 기능은 [인스턴스 API](/Compute/Instance/ko/api-guide/)를 통해 제공됩니다.

### 블록 스토리지 상태

블록 스토리지는 다음과 같은 상태 값을 갖습니다.

| Status | Description |
| --- | --- |
| creating | 생성 중 |
| available | 인스턴스에 연결 가능한 상태 |
| attaching | 인스턴스에 연결 중 |
| detaching | 인스턴스에서 연결 해제 중 |
| in-use | 인스턴스에 연결되어 사용 중인 상태 |
| deleting | 삭제 중 |
| error | 생성 중 에러 발생 |
| error_deleting | 삭제 중 에러 발생 |
| backing-up | 백업 진행 중 |
| restoring-backup | 백업 복구 중 |
| error_backing-up | 백업 진행 중 에러 발생 |
| error_restoring | 백업 복구 중 에러 발생 |
| downloading | Image 다운로드 중 |
| uploading | Image로 업로드 중 |

### 블록 스토리지 조회

블록 스토리지의 정보를 조회합니다.

#### Method, URL
```
GET /v1.0/appkeys/{appkey}/volumes?id={volumeId}
X-Auth-Token: {tokenId}
```

|  Name | In | Type | Optional | Description |
|--|--|--|--|--|
| tokenId | Header | String | - | 토큰 ID |
| volumeId | Query | String | O | 조회할 블록 스토리지 ID. 기재하지 않을 경우 모든 블록 스토리지의 정보를 조회합니다. |

#### Request Body
이 API는 request body를 필요로 하지 않습니다.

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
            "createdAt": "Created At",
            "description": "{Description}",
            "id": "{Block Storage ID}",
            "metadata": {
                "Metadata Key": "{Metadata Value}"
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
| Device Name | Body | String  | 인스턴스에 연결되어 있는 경우, 인스턴스에서의 장치명. ex) "/dev/vdb" |
| Attached Instance ID | Body | String | 인스턴스에 연결되어 있는 경우, 연결된 인스턴스의 ID |
| Attachment ID | Body | String | 인스턴스에 연결되어 있는 경우, 연결 ID |
| Availability Zone Name | Body | String | 블록 스토리지가 위치한 가용성 영역 이름 |
| Created At | Body | String | 블록 스토리지 생성 시간. yyyy-mm-ddTHH:MM:ssZ의 형태. 예) 2017-05-16T02:17:50.166563 |
| Description | Body | String | 블록 스토리지 설명 |
| Block Storage ID | Body | String | 블록 스토리지 ID |
| Metadata Key / Value | Body | Boolean | 블록 스토리지에 기재된 메타 데이터 |
| Block Storage Name | Body | String | 블록 스토리지 이름 |
| Size | Body | Integer | 블록 스토리지 크기 (GB) |
| Status | Body | String | 블록 스토리지 상태 |

### 블록 스토리지 생성
새로운 블록 스토리지를 생성합니다.

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
```
{
    "volume": {
        "description": "{Description}",
        "availabilityZone":"{Availability Zone Name}",
        "snapshotId":"{Snapshot ID}",
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
| Description | Body | String | O | 블록 스토리지 설명 |
| Availability Zone Name | Body | String | - | 블록 스토리지를 생성할 가용성 영역 이름 |
| Snaptshot ID | Body | String | O | 스냅샷으로부터 블록 스토리지를 생성하고자 할 경우 사용하는 스냅샷 ID |
| Size | Body | Integer | - | 블록 스토리지 크기. GB |
| Volume Type | Body | String | - | 생성할 블록 스토리지의 종류, 현재는 별도로 타입이 제공되지 않으므로 빈 문자열로 설정.  |
| Metadata Key / Metadata Value | Body | String | O | 블록 스토리지에 기입하고자 하는 메타데이터 정보 |
| Block Storage Name | Body | String | - | 블록 스토리지 이름 |

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
        "createdAt": "Created At",
        "description": "{Description}",
        "id": "{Block Storage ID}",
        "metadata": {
            "Metadata Key": "{Metadata Value}"
        },
        "name": "{Block Storage Name}",
        "size": "{Size}",
        "status": "{Status}"
    }
}
```

|  Name | In | Type | Description |
|--|--|--|--|
| Availability Zone Name | Body | String | 블록 스토리지가 위치한 가용성 영역 이름 |
| Created At | Body | String | 블록 스토리지 생성 시간. yyyy-mm-ddTHH:MM:ssZ의 형태. 예) 2017-05-16T02:17:50.166563 |
| Description | Body | String | 블록 스토리지 설명 |
| Block Storage ID | Body | String | 블록 스토리지 ID |
| Metadata Key / Value | Body | Boolean | 블록 스토리지에 기재된 메타 데이터 |
| Block Storage Name | Body | String | 블록 스토리지 이름 |
| Size | Body | Integer | 블록 스토리지 크기 (GB) |
| Status | Body | String | 블록 스토리지 상태 |

### 블록 스토리지 삭제
블록 스토리지를 삭제합니다. Status가 "available" "in-use" "error" "error_restoring" 인 블록 스토리지만 삭제할 수 있습니다.

#### Method, URL
```
DELETE /v1.0/appkeys/{appkey}/volumes?id={volumeId}
X-Auth-Token: {tokenId}
```
|  Name | In | Type | Optional | Description |
|--|--|--|--|--|
| tokenId | Header | String | - | 토큰 ID |
| volumeId | Query | String | - | 삭제할 블록 스토리지 ID |

#### Request Body
이 API는 request body를 필요로 하지 않습니다.

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
