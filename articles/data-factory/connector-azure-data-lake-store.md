---
title: Data Factory를 사용하여 Azure Data Lake Storage Gen1 간에 데이터 복사 | Microsoft Docs
description: 지원되는 원본 데이터 저장소에서 Azure Data Lake Store로 또는 Azure Data Lake Store에서 지원되는 싱크 데이터 저장소로 Data Factory를 사용하여 데이터를 복사하는 방법에 대해 알아봅니다.
services: data-factory
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: conceptual
ms.date: 11/05/2018
ms.author: jingwang
ms.openlocfilehash: 3b1abe60fc81ae0316e2d0552a1750129171ff5f
ms.sourcegitcommit: 96527c150e33a1d630836e72561a5f7d529521b7
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/09/2018
ms.locfileid: "51345456"
---
# <a name="copy-data-to-or-from-azure-data-lake-storage-gen1-by-using-azure-data-factory"></a>Azure Data Factory를 사용하여 Azure Data Lake Storage Gen1 간에 데이터 복사
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [버전 1](v1/data-factory-azure-datalake-connector.md)
> * [현재 버전](connector-azure-data-lake-store.md)

이 문서에서는 Azure Data Factory의 복사 작업을 사용하여 Azure Data Lake Storage Gen1(이전의 Azure Data Lake Store) 간에 데이터를 복사하는 방법을 간략하게 설명합니다. 이 문서는 복사 작업에 대한 일반적인 개요를 제공하는 [복사 작업 개요](copy-activity-overview.md) 문서를 기반으로 합니다.

## <a name="supported-capabilities"></a>지원되는 기능

모든 지원되는 원본 데이터 저장소에서 Azure Data Lake Store로 데이터를 복사하거나 Azure Data Lake Store에서 모든 지원되는 싱크 데이터 저장소로 데이터를 복사할 수 있습니다. 복사 작업의 원본 또는 싱크로 지원되는 데이터 저장소 목록은 [지원되는 데이터 저장소](copy-activity-overview.md#supported-data-stores-and-formats) 표를 참조하세요.

특히 이 Azure Data Lake Store 커넥터는 다음을 지원합니다.

- **서비스 주체** 또는 **Azure 리소스에 대한 관리 ID** 인증을 사용하여 파일을 복사합니다.
- 파일을 있는 그대로 복사하거나 [지원되는 파일 형식 및 압축 코덱](supported-file-formats-and-compression-codecs.md)을 사용하여 파일을 붙여넣거나 생성합니다.

> [!IMPORTANT]
> 자체 호스팅 Integration Runtime을 사용하여 데이터를 복사하는 경우 포트 443에서 `<ADLS account name>.azuredatalakestore.net` 및 `login.microsoftonline.com/<tenant>/oauth2/token`에 대한 아웃바운드 트래픽을 허용하도록 회사 방화벽을 구성합니다. 후자는 액세스 토큰을 가져오기 위해 IR에서 통신해야 하는 Azure STS(보안 토큰 서비스)입니다.

## <a name="get-started"></a>시작하기

> [!TIP]
> Azure Data Lake Store 커넥터를 사용하는 연습을 보려면 [Azure Data Lake Store에 데이터 로드](load-azure-data-lake-store.md)를 참조하세요.

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

다음 섹션에서는 Azure Data Lake Store에 한정된 Data Factory 엔터티를 정의하는 데 사용되는 속성에 대해 자세히 설명합니다.

## <a name="linked-service-properties"></a>연결된 서비스 속성

Azure Data Lake Store 연결된 서비스에 다음 속성이 지원됩니다.

| 자산 | 설명 | 필수 |
|:--- |:--- |:--- |
| 형식 | type 속성은 **AzureDataLakeStore**로 설정해야 합니다. | yes |
| dataLakeStoreUri | Azure Data Lake Store 계정에 대한 정보. 이 정보는 `https://[accountname].azuredatalakestore.net/webhdfs/v1` 또는 `adl://[accountname].azuredatalakestore.net/` 형식 중 하나를 사용합니다. | yes |
| subscriptionId | Data Lake Store 계정이 속하는 Azure 구독 ID. | 싱크에 필요 |
| resourceGroupName | Data Lake Store 계정이 속하는 Azure 리소스 그룹 이름. | 싱크에 필요 |
| connectVia | 데이터 저장소에 연결하는 데 사용할 [Integration Runtime](concepts-integration-runtime.md)입니다. Azure Integration Runtime 또는 자체 호스팅 Integration Runtime을 사용할 수 있습니다(데이터 저장소가 개인 네트워크에 있는 경우). 지정하지 않으면 기본 Azure Integration Runtime을 사용합니다. |아니요 |

다른 인증 형식에 대한 더 많은 속성 및 JSON 샘플은 각각 다음 섹션을 참조하세요.

- [서비스 주체 인증 사용](#using-service-principal-authentication)
- [Azure 리소스 인증용 관리 ID 사용](#managed-identity)

### <a name="using-service-principal-authentication"></a>서비스 주체 인증 사용

서비스 주체 인증을 사용하려면 Azure AD(Azure Active Directory)에서 응용 프로그램 엔터티를 등록한 후 Data Lake Store에서 액세스 권한을 부여합니다. 자세한 단계는 [서비스 간 인증](../data-lake-store/data-lake-store-authenticate-using-active-directory.md)을 참조하세요. 연결된 서비스를 정의하는 데 사용되므로 다음 값을 적어둡니다.

- 응용 프로그램 UI
- 응용 프로그램 키
- 테넌트 ID

>[!IMPORTANT]
> Azure Data Lake Store에서 서비스 주체에게 적절한 권한을 부여해야 합니다.
>- **원본의 경우**, 데이터 탐색기 -> 액세스에서 폴더/하위 폴더의 파일을 나열하고 복사할 수 있도록 **읽기 + 실행** 권한 이상을 부여하거나 단일 파일을 복수할 수 있도록 **읽기** 권한을 부여하고, **이 폴더 및 모든 자식 폴더**에 재귀적으로 추가하도록 선택하고 **액세스 권한 및 기본 권한 항목**으로 추가합니다. 계정 수준 액세스 제어(IAM)가 필요하지 않습니다.
>- **싱크의 경우**, 데이터 탐색기 -> 액세스에서 폴더에 자식 항목을 만들 수 있도록 **쓰기 + 실행** 권한 이상을 부여하고, **이 폴더 및 모든 자식 폴더**에 재귀적으로 추가하도록 선택하고 **액세스 권한 및 기본 권한 항목**으로 추가합니다. Azure IR을 사용하여 복사하는 경우(원본과 싱크가 모두 클라우드에 있음) Data Factory가 Data Lake Store의 지역을 감지할 수 있도록 계정 액세스 제어(IAM)에서 적어도 **읽기 권한자** 역할을 부여합니다. 이 IAM 역할을 방지하려면 Data Lake Store의 위치에서 명시적으로 [Azure IR을 만들고](create-azure-integration-runtime.md#create-azure-ir), 다음 예제와 같이 Data Lake Store 연결된 서비스에서 연결합니다.

>[!NOTE]
>**데이터 복사 도구**를 사용하여 복사 파이프라인을 작성하거나 **ADF UI**를 사용하여 작성 중에 연결 테스트/폴더 탐색을 수행하는 경우 루트부터 폴더를 표시하려면 **"실행" 권한과 함께 루트 수준에서** 서비스 주체의 권한이 부여되어야 합니다. 반면 복사할 데이터에 대한 권한이 부여되는 한 복사 작업 실행은 계속 작동할 수 있습니다. 권한을 제한해야 하는 경우 작성 작업을 건너뛸 수 있습니다.

다음과 같은 속성이 지원됩니다.

| 속성 | 설명 | 필수 |
|:--- |:--- |:--- |
| servicePrincipalId | 응용 프로그램의 클라이언트 ID를 지정합니다. | yes |
| servicePrincipalKey | 응용 프로그램의 키를 지정합니다. 이 필드를 SecureString으로 표시하여 Data Factory에 안전하게 저장하거나 [Azure Key Vault에 저장되는 비밀을 참조](store-credentials-in-key-vault.md)합니다. | yes |
| tenant | 응용 프로그램이 있는 테넌트 정보(도메인 이름 또는 테넌트 ID)를 지정합니다. Azure Portal의 오른쪽 위 모서리에 마우스를 이동하여 검색할 수 있습니다. | yes |

**예제:**

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="managed-identity"></a> Azure 리소스 인증용 관리 ID 사용

특정 데이터 팩터리를 나타내는 [Azure 리소스용 관리 ID](data-factory-service-identity.md)와 데이터 팩터리를 연결할 수 있습니다. 고유한 서비스 주체를 사용하는 것과 유사하게 Data Lake Store 인증에 이 서비스 ID를 직접 사용할 수 있습니다. 이 지정된 팩터리는 Data Lake Store 간에 데이터에 액세스하고 복사할 수 있습니다.

Azure 리소스 인증에 관리 ID를 사용하려면

1. 팩터리와 함께 생성된 "서비스 ID 응용 프로그램 ID"의 값을 복사하여 [데이터 팩터리 서비스 ID를 검색](data-factory-service-identity.md#retrieve-service-identity)합니다.
2. 아래 메모에 따라 서비스 주체에서 수행한 것과 동일한 방식으로 Data Lake Store에 서비스 ID 액세스 권한을 부여합니다.

>[!IMPORTANT]
> Azure Data Lake Store에서 데이터 팩터리 서비스 ID에 적절한 사용 권한을 부여해야 합니다.
>- **원본의 경우**, 데이터 탐색기 -> 액세스에서 폴더/하위 폴더의 파일을 나열하고 복사할 수 있도록 **읽기 + 실행** 권한 이상을 부여하거나 단일 파일을 복수할 수 있도록 **읽기** 권한을 부여하고, **이 폴더 및 모든 자식 폴더**에 재귀적으로 추가하도록 선택하고 **액세스 권한 및 기본 권한 항목**으로 추가합니다. 계정 수준 액세스 제어(IAM)가 필요하지 않습니다.
>- **싱크의 경우**, 데이터 탐색기 -> 액세스에서 폴더에 자식 항목을 만들 수 있도록 **쓰기 + 실행** 권한 이상을 부여하고, **이 폴더 및 모든 자식 폴더**에 재귀적으로 추가하도록 선택하고 **액세스 권한 및 기본 권한 항목**으로 추가합니다. Azure IR을 사용하여 복사하는 경우(원본과 싱크가 모두 클라우드에 있음) Data Factory가 Data Lake Store의 지역을 감지할 수 있도록 계정 액세스 제어(IAM)에서 적어도 **읽기 권한자** 역할을 부여합니다. 이 IAM 역할을 방지하려면 Data Lake Store의 위치에서 명시적으로 [Azure IR을 만들고](create-azure-integration-runtime.md#create-azure-ir), 다음 예제와 같이 Data Lake Store 연결된 서비스에서 연결합니다.

>[!NOTE]
>**데이터 복사 도구**를 사용하여 복사 파이프라인을 작성하거나 **ADF UI**를 사용하여 작성 중에 연결 테스트/폴더 탐색을 수행하는 경우 루트부터 폴더를 표시하려면 **"실행" 권한과 함께 루트 수준에서** 서비스 주체의 권한이 부여되어야 합니다. 반면 복사할 데이터에 대한 권한이 부여되는 한 복사 작업 실행은 계속 작동할 수 있습니다. 권한을 제한해야 하는 경우 작성 작업을 건너뛸 수 있습니다.

Azure Data Factory에서 연결된 서비스의 일반 Data Lake Store 정보 이외의 속성을 지정할 필요가 없습니다.

**예제:**

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>데이터 세트 속성

데이터 세트 정의에 사용할 수 있는 섹션 및 속성의 전체 목록은 데이터 세트 문서를 참조하세요. 이 섹션에서는 Azure Data Lake Store 데이터 세트에서 지원하는 속성의 목록을 제공합니다.

Azure Data Lake Store 간에 데이터를 복사하려면 데이터 세트의 type 속성을 **AzureDataLakeStoreFile**로 설정합니다. 다음과 같은 속성이 지원됩니다.

| 속성 | 설명 | 필수 |
|:--- |:--- |:--- |
| 형식 | 데이터 세트의 type 속성을 **AzureDataLakeStoreFile**로 설정해야 합니다. |yes |
| folderPath | Data Lake Store의 폴더에 대한 경로입니다. 와일드카드 필터는 지원되지 않습니다. 지정하지 않으면 루트를 가리킵니다. 예: rootfolder/subfolder/ |아니요 |
| fileName | 지정된 "folderPath" 아래의 파일에 대한 **이름 또는 와일드 카드 필터**입니다. 이 속성의 값을 지정하지 않으면 데이터 세트는 폴더에 있는 모든 파일을 가리킵니다. <br/><br/>필터에 허용되는 와일드카드는 `*`(문자 0자 이상 일치) 및 `?`(문자 0자 또는 1자 일치)입니다.<br/>- 예 1: `"fileName": "*.csv"`<br/>- 예 2: `"fileName": "???20180427.txt"`<br/>`^`을 사용하여 실제 파일 이름 내에 와일드카드 또는 이 이스케이프 문자가 있는 경우 이스케이프합니다.<br/><br/>fileName이 출력 데이터 세트에 대해 지정되지 않고 **preserveHierarchy**가 활동 싱크에 지정되지 않으면, 복사 활동에서 자동으로 다음 형식의 파일 이름을 생성합니다. "*Data.[activity run id GUID].[GUID if FlattenHierarchy].[format if configured].[compression if configured]*" 예: "Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt.gz" |아니요 |
| format | 파일 기반 저장소(이진 복사) 간에 **파일을 있는 그대로 복사**하려는 경우 입력 및 출력 데이터 세트 정의 둘 다에서 형식 섹션을 건너뜁니다.<br/><br/>특정 형식으로 파일을 생성하거나 구문 분석하려는 경우 **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**과 같은 파일 형식 유형이 지원됩니다. 이 값 중 하나로 서식에서 **type** 속성을 설정합니다. 자세한 내용은 [텍스트 형식](supported-file-formats-and-compression-codecs.md#text-format), [Json 형식](supported-file-formats-and-compression-codecs.md#json-format), [Avro 형식](supported-file-formats-and-compression-codecs.md#avro-format), [Orc 형식](supported-file-formats-and-compression-codecs.md#orc-format) 및 [Parquet 형식](supported-file-formats-and-compression-codecs.md#parquet-format) 섹션을 참조하세요. |아니요(이진 복사 시나리오에만 해당) |
| 압축 | 데이터에 대한 압축 유형 및 수준을 지정합니다. 자세한 내용은 [지원되는 파일 형식 및 압축 코덱](supported-file-formats-and-compression-codecs.md#compression-support)을 참조하세요.<br/>지원되는 형식은 **GZip**, **Deflate**, **BZip2** 및 **ZipDeflate**입니다.<br/>지원되는 수준은 **최적** 및 **가장 빠름**입니다. |아니요 |

>[!TIP]
>폴더 아래에서 모든 파일을 복사하려면 **folderPath**만을 지정합니다.<br>지정된 이름의 단일 파일을 복사하려면 폴더 부분으로 **folderPath** 및 파일 이름으로 **fileName**을 지정합니다.<br>폴더 아래에서 파일의 하위 집합을 복사하려면 폴더 부분으로 **folderPath** 및 와일드카드 필터로 **fileName**을 지정합니다. 

**예제:**

```json
{
    "name": "ADLSDataset",
    "properties": {
        "type": "AzureDataLakeStoreFile",
        "linkedServiceName":{
            "referenceName": "<ADLS linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "datalake/myfolder/",
            "fileName": "myfile.csv.gz",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

## <a name="copy-activity-properties"></a>복사 작업 속성

작업 정의에 사용할 수 있는 섹션 및 속성의 전체 목록은 [파이프라인](concepts-pipelines-activities.md) 문서를 참조하세요. 이 섹션에서는 Azure Data Lake 원본 및 싱크에서 지원하는 속성 목록을 제공합니다.

### <a name="azure-data-lake-store-as-source"></a>Azure Data Lake Store를 원본으로

Azure Data Lake Store에서 데이터를 복사하려면 복사 작업의 원본 형식을 **AzureDataLakeStoreSource**로 설정합니다. 복사 작업 **source** 섹션에서 다음 속성이 지원됩니다.

| 자산 | 설명 | 필수 |
|:--- |:--- |:--- |
| 형식 | 복사 작업 원본의 type 속성을 **AzureDataLakeStoreSource**로 설정해야 합니다. |yes |
| recursive | 하위 폴더에서 또는 지정된 폴더에서만 데이터를 재귀적으로 읽을지 여부를 나타냅니다. recursive가 true로 설정되고 싱크가 파일 기반 저장소인 경우 싱크에서 빈 폴더/하위 폴더가 복사/생성되지 않습니다.<br/>허용되는 값은 **true**(기본값), **false**입니다. | 아니요 |

**예제:**

```json
"activities":[
    {
        "name": "CopyFromADLS",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<ADLS input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "AzureDataLakeStoreSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="azure-data-lake-store-as-sink"></a>Azure Data Lake Store를 싱크로

Azure Data Lake Store에 데이터를 복사하려면 복사 작업의 싱크 형식을 **AzureDataLakeStoreSink**로 설정합니다. **sink** 섹션에서 다음 속성이 지원됩니다.

| 자산 | 설명 | 필수 |
|:--- |:--- |:--- |
| 형식 | 복사 작업 싱크의 type 속성을 **AzureDataLakeStoreSink**로 설정해야 합니다. |yes |
| copyBehavior | 원본이 파일 기반 데이터 저장소의 파일인 경우 복사 동작을 정의합니다.<br/><br/>허용되는 값은 다음과 같습니다.<br/><b>- PreserveHierarchy(기본값)</b>: 대상 폴더에서 파일 계층 구조를 유지합니다. 원본 폴더의 원본 파일 상대 경로는 대상 폴더의 대상 파일 상대 경로와 동일합니다.<br/><b>- FlattenHierarchy</b>: 원본 폴더의 모든 파일은 대상 폴더의 첫 번째 수준에 있게 됩니다. 대상 파일은 자동 생성된 이름을 갖습니다. <br/><b>- MergeFiles</b>: 원본 폴더의 모든 파일을 하나의 파일로 병합합니다. 파일/Blob 이름이 지정된 경우 지정된 이름이 병합된 파일 이름이 됩니다. 그렇지 않으면 자동 생성된 파일 이름이 병합된 파일 이름이 됩니다. | 아니요 |

**예제:**

```json
"activities":[
    {
        "name": "CopyToADLS",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<ADLS output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDataLakeStoreSink",
                "copyBehavior": "PreserveHierarchy"
            }
        }
    }
]
```

### <a name="recursive-and-copybehavior-examples"></a>recursive 및 copyBehavior 예제

이 섹션에서는 다양한 recursive 및 copyBehavior 값 조합에 대한 복사 작업의 결과 동작을 설명합니다.

| recursive | copyBehavior | 원본 폴더 구조 | 결과 대상 |
|:--- |:--- |:--- |:--- |
| true |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | Folder1 대상 폴더가 다음과 같이 원본 폴더와 동일한 구조로 만들어집니다.<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5. |
| true |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | Folder1 대상 폴더가 다음과 같은 구조로 만들어집니다. <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1에 대해 자동으로 생성된 이름<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2에 대해 자동 생성된 이름<br/>&nbsp;&nbsp;&nbsp;&nbsp;File3에 대해 자동 생성된 이름<br/>&nbsp;&nbsp;&nbsp;&nbsp;File4에 대해 자동 생성된 이름<br/>&nbsp;&nbsp;&nbsp;&nbsp;File5에 대해 자동 생성된 이름 |
| true |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | Folder1 대상 폴더가 다음과 같은 구조로 만들어집니다. <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1, File2, File3, File4 및 File5의 파일 내용이 자동 생성된 파일 이름을 사용하는 파일 하나로 병합됩니다. |
| false |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | Folder1 대상 폴더가 다음과 같은 구조로 만들어집니다.<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/>File3, File4, File5를 가진 Subfolder1은 선택되지 않습니다. |
| false |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | Folder1 대상 폴더가 다음과 같은 구조로 만들어집니다.<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1에 대해 자동으로 생성된 이름<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2에 대해 자동 생성된 이름<br/><br/>File3, File4, File5를 가진 Subfolder1은 선택되지 않습니다. |
| false |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | Folder1 대상 폴더가 다음과 같은 구조로 만들어집니다.<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1과 File2의 파일 내용이 자동 생성된 파일 이름을 사용하는 파일 하나로 병합됩니다. File1에 대해 자동으로 생성된 이름<br/><br/>File3, File4, File5를 가진 Subfolder1은 선택되지 않습니다. |

## <a name="next-steps"></a>다음 단계
Azure Data Factory에서 복사 작업의 원본 및 싱크로 지원되는 데이터 저장소 목록은 [지원되는 데이터 저장소](copy-activity-overview.md##supported-data-stores-and-formats)를 참조하세요.
