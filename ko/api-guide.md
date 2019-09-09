## Storage > Block Storage > API 가이드

API는 현재 한국 리전에서만 사용할 수 있습니다.

## 사전 준비

블록 스토리지 API를 사용하려면 앱키(Appkey)와 토큰이 필요합니다. [API Endpoint URL](/Compute/Instance/ko/api-guide/#api-endpoint-url)과 [토큰 API](/Compute/Instance/ko/api-guide/#api)를 이용하여 앱키와 토큰을 준비합니다. 앱키는 API Endpoint URL에, 토큰은 Request Header에 포함하여 사용합니다.

예를 들어, 블록 스토리지 정보 조회는 다음 URL로 요청해야 합니다.

	GET https://api-compute.cloud.toast.com/compute/v1.0/appkeys/{appkey}/volumes?id={volumeId}


## 블록 스토리지 API

블록 스토리지 생성, 삭제, 조회 기능을 제공합니다. 블록 스토리지를 인스턴스에 연결하고 해제하는 기능은 [인스턴스 추가 기능 API](/Compute/Instance/ko/api-guide/#_8)로 제공됩니다.

### 블록 스토리지 상태

블록 스토리지는 다음과 같은 상탯값을 갖습니다.

| Status | Description |
| --- | --- |
| creating | 생성 중 |
| available | 인스턴스에 연결 가능한 상태 |
| attaching | 인스턴스에 연결 중 |
| detaching | 인스턴스에서 연결 해제 중 |
| in-use | 인스턴스에 연결되어 사용 중인 상태 |
| deleting | 삭제 중 |
| error | 생성 중 오류 발생 |
| error_deleting | 삭제 중 오류 발생 |
| backing-up | 백업 진행 중 |
| restoring-backup | 백업 복구 중 |
| error_backing-up | 백업 진행 중 오류 발생 |
| error_restoring | 백업 복구 중 오류 발생 |
| downloading | 이미지 다운로드 중 |
| uploading | 이미지로 업로드 중 |

### 블록 스토리지 정보 조회

블록 스토리지의 정보를 조회합니다.

#### Method, URL
```
GET /v1.0/appkeys/{appkey}/volumes?id={volumeId}
X-Auth-Token: {tokenId}
```

|  Name | In | Type | Optional | Description |
|--|--|--|--|--|
| tokenId | Header | String | - | 토큰 ID |
| volumeId | Query | String | O | 조회할 블록 스토리지 ID. 없으면 모든 블록 스토리지의 정보를 조회합니다. |

#### Request Body
이 API는 Request Body가 필요 없습니다.

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
| Device Name | Body | String  | 인스턴스에 연결되어 있는 경우, 인스턴스에서의 장치명. ex) "/dev/vdb" |
| Attached Instance ID | Body | String | 인스턴스에 연결되어 있는 경우, 연결된 인스턴스의 ID |
| Attachment ID | Body | String | 인스턴스에 연결되어 있는 경우, 연결 ID |
| Availability Zone Name | Body | String | 블록 스토리지가 위치한 가용성 영역 이름 |
| Created At | Body | String | 블록 스토리지 생성 시간. yyyy-mm-ddTHH:MM:ssZ의 형태. 예) 2017-05-16T02:17:50.166563 |
| Description | Body | String | 블록 스토리지 설명 |
| Block Storage ID | Body | String | 블록 스토리지 ID |
| Metadata Key / Value | Body | Boolean | 블록 스토리지에 기재된 메타 데이터 |
| Block Storage Name | Body | String | 블록 스토리지 이름 |
| Size | Body | Integer | 블록 스토리지 크기(GB) |
| Status | Body | String | 블록 스토리지 상태 |
| Volume Type | Body | String | 블록 스토리지 종류. "General HDD" 또는 "General SSD" 중 하나. |

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
| tokenId | Header | String | - | 토큰 ID |

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
| Description | Body | String | O | 블록 스토리지 설명 |
| Availability Zone Name | Body | String | - | 블록 스토리지를 생성할 가용성 영역 이름 |
| Size | Body | Integer | - | 블록 스토리지 크기(GB). 10~1000 범위, 10단위로 입력 |
| Volume Type | Body | String | O | 생성할 블록 스토리지 종류. "General HDD" 또는 "General SSD" 중 하나. |
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
| Availability Zone Name | Body | String | 블록 스토리지가 위치한 가용성 영역 이름 |
| Created At | Body | String | 블록 스토리지 생성 시간. yyyy-mm-ddTHH:MM:ssZ의 형태. 예) 2017-05-16T02:17:50.166563 |
| Description | Body | String | 블록 스토리지 설명 |
| Block Storage ID | Body | String | 블록 스토리지 ID |
| Metadata Key / Value | Body | Boolean | 블록 스토리지에 기재된 메타 데이터 |
| Block Storage Name | Body | String | 블록 스토리지 이름 |
| Size | Body | Integer | 블록 스토리지 크기 (GB) |
| Status | Body | String | 블록 스토리지 상태 |
| Volume Type | Body | String | 블록 스토리지 종류. "General HDD" 또는 "General SSD" 중 하나. |

### 블록 스토리지 삭제
블록 스토리지를 삭제합니다. Status가 "available" "in-use" "error" "error_restoring"인 블록 스토리지만 삭제할 수 있습니다.

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
이 API는 Request Body가 필요 없습니다.

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
