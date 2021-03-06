---
title: 온-프레미스 Apache Hadoop 클러스터를 Azure HDInsight로 마이그레이션 - 인프라 모범 사례
description: 온-프레미스 Hadoop 클러스터를 Azure HDInsight로 마이그레이션하는 인프라 모범 사례를 알아봅니다.
services: hdinsight
author: hrasheed-msft
ms.reviewer: ashishth
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 10/25/2018
ms.author: hrasheed
ms.openlocfilehash: 614fdae1865f008bdbc2cb8d5e8b96c0addcc112
ms.sourcegitcommit: f0c2758fb8ccfaba76ce0b17833ca019a8a09d46
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/06/2018
ms.locfileid: "51036932"
---
# <a name="migrate-on-premises-apache-hadoop-clusters-to-azure-hdinsight---infrastructure-best-practices"></a>온-프레미스 Apache Hadoop 클러스터를 Azure HDInsight로 마이그레이션 - 인프라 모범 사례

이 문서에서는 Azure HDInsight 클러스터의 인프라를 관리하기 위한 권장 사항을 제공합니다. 온-프레미스 Apache Hadoop 시스템을 Azure HDInsight로 마이그레이션하는 데 도움을 주는 모범 사례를 제공하는 시리즈의 일부입니다.

## <a name="plan-well-for-the-capacity-needed-for-hdinsight-clusters"></a>HDInsight 클러스터에 필요한 용량을 적절하게 계획

HDInsight 클러스터 용량 계획을 만들기 위한 주요 선택 항목은 다음과 같습니다.

- **지역 선택** - Azure 지역은 클러스터가 물리적으로 프로비전되는 위치를 결정합니다. 읽기 및 쓰기 대기 시간을 최소화하려면 클러스터가 데이터와 동일한 지역에 있어야 합니다.
- **저장소 위치 및 크기 선택** - 기본 저장소는 클러스터와 동일한 지역에 있어야 합니다. 48 노드 클러스터의 경우 4~8개의 저장소 계정을 갖는 것이 좋습니다. 총 저장소가 이미 충분할 수 있지만 각 저장소 계정이 계산 노드에 대해 추가 네트워킹 대역폭을 제공합니다. 저장소 계정이 여러 개 있는 경우 각 저장소 계정에 대해 접두사 없는 임의 이름을 사용합니다. 임의 이름을 지정하는 목적은 모든 계정에서 저장소 병목 상태(제한) 또는 일반 모드 오류가 발생할 가능성을 줄이려는 것입니다. 성능 향상을 위해서는 저장소 계정당 하나의 컨테이너만 사용합니다.
- **VM 크기 및 유형 선택(이제 G-시리즈 지원)** - 각 클러스터 유형마다 노드 유형 집합이 있으며 각 노드 유형은 해당 VM 크기 및 유형에 대한 특정 옵션을 제공합니다. VM 크기와 유형은 CPU 처리 성능, RAM 크기 및 네트워크 대기 시간에 따라 결정됩니다. 각 노드 유형에 대한 최적의 VM 크기 및 유형을 결정하는 데 시뮬레이트된 워크로드를 사용할 수 있습니다.
- **작업자 노드 수 선택** - 시뮬레이트된 워크로드를 사용하여 작업자 노드의 초기 수를 결정할 수 있습니다. 최대 부하 요구 사항에 맞게 더 많은 작업자 노드를 추가하여 나중에 클러스터를 확장할 수 있습니다. 추가 작업자 노드가 필요하지 않을 때 클러스터를 나중에 축소할 수 있습니다.

자세한 내용은 문서 [HDInsight 클러스터에 대한 용량 계획](../hdinsight-capacity-planning.md)을 참조하세요.

## <a name="use-the-recommended-virtual-machine-types-for-cluster-nodes"></a>클러스터 노드에 권장되는 가상 머신 유형 사용

각 HDInsight 클러스터 유형에 대한 권장되는 가상 머신 유형은 [클러스터에 대한 기본 노드 구성 및 가상 머신 크기](../hdinsight-component-versioning.md#default-node-configuration-and-virtual-machine-sizes-for-clusters)를 참조하세요.

## <a name="check-the-availability-of-hadoop-components-in-hdinsight"></a>HDInsight에서 Hadoop 구성 요소의 가용성 확인

각 HDInsight 버전은 HDP(Hortonworks Data Platform) 버전의 클라우드 배포판이며 Hadoop 에코 시스템 구성 요소의 집합으로 구성됩니다. 모든 HDInsight 구성 요소 및 해당 현재 버전에 대한 자세한 내용은 [HDInsight 구성 요소 버전 관리](../hdinsight-component-versioning.md)를 참조하세요.

또한 Ambari UI 또는 Ambari REST API를 사용하여 HDInsight에서 Hadoop 구성 요소 및 버전을 확인할 수 있습니다.

온-프레미스 클러스터에서 사용할 수 있었지만 HDInsight 클러스터의 일부가 아닌 응용 프로그램 또는 구성 요소를 HDInsight 클러스터와 동일한 VNet에서 에지 노드 또는 VM에 추가할 수 있습니다. Azure HDInsight에서 사용할 수 없는 타사 Hadoop 응용 프로그램을 HDInsight 클러스터에서 "응용 프로그램" 옵션을 사용하여 설치할 수 있습니다. 사용자 지정 Hadoop 응용 프로그램을 "스크립트 동작"을 사용하여 HDInsight 클러스터에 설치할 수 있습니다. 다음 표는 일반적인 응용 프로그램 및 해당 HDInsight 통합 옵션 중 일부를 나열합니다.

|**응용 프로그램**|**통합**
|---|---|
|기류|IaaS 또는 HDInsight 에지 노드
|Alluxio|IaaS  
|Arcadia|IaaS 
|Atlas|없음(HDP만)
|Datameer|HDInsight 에지 노드
|Datastax(Cassandra)|IaaS(CosmosDB Azure에서 대체)
|DataTorrent|IaaS 
|Drill|IaaS 
|Ignite|IaaS
|Jethro|IaaS 
|Mapador|IaaS 
|Mongo|IaaS(CosmosDB Azure에서 대체)
|NiFi|IaaS 
|Presto|IaaS 또는 HDInsight 에지 노드
|Python 2|PaaS 
|Python 3|PaaS 
|R|PaaS 
|SAS|IaaS 
|Vertica|IaaS(SQLDW Azure에서 대체)
|Tableau|IaaS 
|Waterline|HDInsight 에지 노드
|StreamSets|HDInsight 에지 
|Palantir|IaaS 
|Sailpoint|Iaas 

자세한 내용은 [각 HDInsight 버전에서 제공되는 Apache Hadoop 구성 요소](../hdinsight-component-versioning.md#apache-hadoop-components-available-with-different-hdinsight-versions) 문서 참조

## <a name="customize-hdinsight-clusters-using-script-actions"></a>스크립트 동작을 사용하여 HDInsight 클러스터 사용자 지정

HDInsight는 **스크립트 동작**이라는 클러스터 구성의 메서드를 제공합니다. 스크립트 동작은 HDInsight 클러스터의 노드에서 실행되는 Bash 스크립트이며 추가 구성 요소를 설치하고 구성 설정을 변경하는 데 사용할 수 있습니다.

스크립트 동작은 HDInsight 클러스터에서 액세스할 수 있는 URI에 저장되어야 합니다. 클러스터를 만드는 동안 또는 만든 후 사용할 수 있으며 특정 노드 형식에서만 실행되도록 제한될 수도 있습니다.

스크립트를 지속하거나 한 번 실행할 수 있습니다. 지속형 스크립트는 크기 조정 작업을 통해 클러스터에 추가되는 새 작업자 노드를 사용자 지정하는 데 사용됩니다. 지속형 스크립트는 크기 조정 작업이 발생하면 헤드 노드 등의 다른 노드 유형에도 변경 사항을 적용할 수 있습니다.

HDInsight는 HDInsight 클러스터에서 다음 구성 요소를 설치하는 미리 작성된 스크립트를 제공합니다.

- Azure Storage 계정 추가
- Hue 설치
- Presto 설치
- Solr 설치
- Giraph 설치
- Hive 라이브러리 사전 로드
- Mono 설치 또는 업데이트

> [!Note]
> HDInsight는 사용자 지정 hadoop 구성 요소 또는 스크립트 동작을 사용하여 설치된 구성 요소에 대한 직접적인 지원을 제공하지 않습니다.

스크립트 작업을 Azure Marketplace에 HDInsight 응용 프로그램으로 게시할 수도 있습니다.

자세한 내용은 다음 문서를 참조하세요.

- [HDInsight에 타사 Hadoop 응용 프로그램 설치](../hdinsight-apps-install-applications.md)
- [스크립트 동작을 사용하여 HDInsight 클러스터 사용자 지정](../hdinsight-hadoop-customize-cluster-linux.md)
- [Azure Marketplace에 HDInsight 응용 프로그램 게시](../hdinsight-apps-publish-applications.md)

## <a name="customize-hdinsight-configs-using-bootstrap"></a>부트스트랩을 사용하여 HDInsight 클러스터 구성 사용자 지정

`core-site.xml`, `hive-site.xml` 및 `oozie-env.xml`과 같은 구성 파일에서 구성에 대한 변경 내용은 부트스트랩을 사용하여 만들 수 있습니다. 다음 스크립트는 Powershell을 사용하는 예제입니다.

```powershell
# hive-site.xml configuration
$hiveConfigValues = @{"hive.metastore.client.socket.timeout"="90"}

$config = New—AzureRmHDInsightClusterConfig '
    | Set—AzureRmHDInsightDefaultStorage
    —StorageAccountName "$defaultStorageAccountName.blob. core.windows.net" `
    —StorageAccountKey "defaultStorageAccountKey " `
    | Add—AzureRmHDInsightConfigValues `
        —HiveSite $hiveConfigValues

New—AzureRmHDInsightCluster `
    —ResourceGroupName $existingResourceGroupName `
    —Cluster-Name $clusterName `
    —Location $location `
    —ClusterSizeInNodes $clusterSizeInNodes `
    —ClusterType Hadoop `
    —OSType Linux `
    —Version "3.6" `
    —HttpCredential $httpCredential `
    —Config $config
```

자세한 내용은 문서 [부트스트랩을 사용하여 HDInsight 클러스터 사용자 지정](../hdinsight-hadoop-customize-cluster-bootstrap.md)을 참조하세요.

## <a name="use-edge-nodes-on-hadoop-clusters-in-hdinsight-to-access-the-client-tools"></a>HDInsight의 Hadoop 클러스터에서 에지 노드를 사용하여 클라이언트 도구에 액세스

빈 에지 노드는 Hadoop 서비스가 실행되지 않지만 헤드 노드와 같이 설치되고 구성된 동일한 클라이언트 도구를 사용하는 Linux 가상 머신입니다. 에지 노드는 다음과 같은 용도로 사용될 수 있습니다.

- 클러스터에 액세스
- 클라이언트 응용 프로그램 테스트
- 클라이언트 응용 프로그램 호스트

Azure Portal을 통해 에지 노드를 만들거나 삭제할 수 있으며 클러스터를 만드는 동안 또는 만든 후에 사용할 수 있습니다. 에지 노드를 만든 후 SSH를 사용하여 에지 노드에 연결할 수 있고, HDInsight에서 Hadoop 클러스터에 액세스하는 클라이언트 도구를 실행할 수 있습니다. 에지 노드 ssh 엔드포인트는 `<EdgeNodeName>.<ClusterName>-ssh.azurehdinsight.net:22`입니다.

자세한 내용은 문서 [HDInsight의 Hadoop 클러스터에서 빈 에지 노드 사용](../hdinsight-apps-use-edge-node.md)을 참조하세요.

## <a name="use-the-scale-up-and-scale-down-feature-of-clusters"></a>클러스터의 확장 및 축소 기능 사용

HDInsight는 클러스터의 작업자 노드 수를 증가 및 감소하는 옵션을 제공하여 유연성을 보장합니다. 이 기능을 통해 몇 시간 후에 또는 주말에 클러스터를 축소했다가, 비즈니스 요구가 최대 수준일 때 확장할 수 있습니다.

다음 메서드를 사용하여 클러스터 크기 조정을 자동화할 수 있습니다.

### <a name="powershell-cmdlet"></a>PowerShell cmdlet

```powershell
Set-AzureRmHDInsightClusterSize -ClusterName <Cluster Name> -TargetInstanceCount <NewSize>
```

### <a name="azure-cli"></a>Azure CLI

```powershell
azure hdinsight cluster resize [options] <clusterName> <Target Instance Count>
```

### <a name="azure-portal"></a>Azure portal

실행 중인 HDInsight 클러스터에 노드를 추가할 경우 보류 중이거나 실행 중인 작업에는 영향을 미치지 않습니다. 크기 조정 프로세스가 실행되는 동안 새 작업을 안전하게 제출할 수 있습니다. 어떤 이유로든 크기 조정 작업이 실패할 경우 오류가 정상적으로 처리되며 클러스터는 작동 상태를 유지합니다.

그러나 노드를 제거하여 클러스터를 축소하는 경우 크기 조정 작업이 완료될 때 보류 중이거나 실행 중인 작업은 모두 실패합니다. 이 오류는 이 프로세스 동안 일부 서비스가 다시 시작되기 때문입니다. 이 문제를 해결하기 위해 클러스터를 축소하기 전에 작업이 완료될 때까지 기다린 다음, 수동으로 작업을 종료하거나 크기 조정 작업이 완료된 후에 작업을 다시 제출할 수 있습니다.

클러스터를 최소 1개의 작업자 노드로 축소하는 경우 작업자 노드가 패치를 위해 재부팅될 때 또는 크기 조정 작업 직후에 HDFS가 안전 모드에서 고정될 수 있습니다. HDFS를 안전 모드에서 해제하는 다음 명령을 실행할 수 있습니다.

```bash
hdfs dfsadmin -D 'fs.default.name=hdfs://mycluster/' -safemode leave
```

안전 모드를 해제한 후에는 임시 파일을 수동으로 제거하거나 Hive에서 자동으로 정리할 때까지 기다릴 수 있습니다.

자세한 내용은 문서 [HDInsight 클러스터 크기 조정](../hdinsight-scaling-best-practices.md)을 참조하세요.

## <a name="use-hdinsight-with-azure-virtual-network"></a>Azure Virtual Network에 HDInsight 사용

Azure Virtual Network를 통해 네트워크 트래픽을 필터링 및 라우팅하여 Azure Virtual Machines와 같은 Azure 리소스가 상호 간 통신, 인터넷 통신 및 온-프레미스 네트워크 통신을 안전하게 수행할 수 있습니다.

HDInsight와 함께 Azure Virtual Network를 사용하면 다음 시나리오가 가능합니다.

- 온-프레미스 네트워크에서 HDInsight에 직접 연결합니다.
- Azure Virtual Network에서 데이터 저장소에 HDInsight를 연결합니다.
- 인터넷을 통해 공개적으로 사용할 수 없는 Hadoop 서비스에 직접 액세스합니다. 예를 들어 Kafka API 또는 HBase Java API.

HDInsight를 새 또는 기존 Azure Virtual Network에 추가할 수 있습니다. HDInsight가 기존 Virtual Network에 추가되는 경우 기존 네트워크 보안 그룹 및 사용자 정의 경로는 Azure 데이터 센터에서 [여러 개의 IP 주소](../hdinsight-extend-hadoop-virtual-network.md#hdinsight-ip-1)에 대한 무제한 액세스를 허용하도록 업데이트되어야 합니다. 또한 HDInsight 서비스에서 사용되는 [포트](../hdinsight-extend-hadoop-virtual-network.md#hdinsight-ports)에 대한 트래픽을 차단하지 않도록 해야 합니다.

> [!Note]
> HDInsight는 현재 강제 터널링을 지원하지 않습니다. 강제 터널링은 검사 및 로깅을 위해 디바이스에 아웃바운드 인터넷 트래픽을 적용하는 서브넷 설정입니다. 서브넷에 HDInsight를 설치하기 전에 강제 터널링을 제거하거나 HDInsight에 대해 새 서브넷을 만듭니다. HDInsight는 또한 아웃바운드 네트워크 연결 제한을 지원하지 않습니다.

자세한 내용은 다음 문서를 참조하세요.

- [Azure 가상 네트워크 개요](../../virtual-network/virtual-networks-overview.md)
- [Azure Virtual Network를 사용하여 Azure HDInsight 확장](../hdinsight-extend-hadoop-virtual-network.md)

## <a name="use-azure-virtual-network-service-endpoints-to-securely-connect-to-other-azure-services"></a>Azure Virtual Network 서비스 엔드포인트를 사용하여 다른 Azure 서비스에 안전하게 연결

HDInsight는 Azure Blob Storage, Azure Data Lake Storage Gen2, Cosmos DB 및 SQL 데이터베이스에 안전하게 연결할 수 있도록 하는 [가상 네트워크 서비스 엔드포인트를](../../virtual-network/virtual-network-service-endpoints-overview.md) 지원합니다. Azure HDInsight에 대한 서비스 엔드포인트를 활성화하여 트래픽은 Azure 데이터 센터 내에서 보안된 경로를 통해 흐릅니다. 네트워킹 계층에서 이 향상된 수준의 보안을 사용하여 빅 데이터 저장소 계정을 지정된 VNET(Virtual Network)으로 잠글 수 있으며 여전히 HDInsight 클러스터를 원활하게 사용하여 해당 데이터에 액세스하고 처리할 수 있습니다.

자세한 내용은 다음 문서를 참조하세요.

- [가상 네트워크 서비스 엔드포인트](../../virtual-network/virtual-network-service-endpoints-overview.md)
- [서비스 엔드포인트를 사용하여 HDInsight 보안 강화](https://azure.microsoft.com/blog/enhance-hdinsight-security-with-service-endpoints/.md)

## <a name="connect-hdinsight-to-the-on-premises-network"></a>온-프레미스 네트워크에 HDInsight 연결

Azure Virtual Network와 VPN 게이트웨이를 사용하여 HDInsight를 온-프레미스 네트워크에 연결할 수 있습니다. 연결을 설정하는 데 다음 단계를 사용할 수 있습니다.

- 온-프레미스 네트워크에 연결된 Azure Virtual Network에서 HDInsight를 사용합니다.
- 가상 네트워크와 온-프레미스 네트워크 간에 DNS 이름 확인을 구성합니다.
- 네트워크 보안 그룹 또는 UDR(사용자 정의 경로)을 구성하여 네트워크 트래픽을 제어합니다.

자세한 내용은 문서 [온-프레미스 네트워크에 HDInsight 연결](../connect-on-premises-network.md)을 참조하세요.

## <a name="next-steps"></a>다음 단계

이 시리즈의 다음 문서를 읽어보세요.

- [온-프레미스에서 Azure HDInsight Hadoop 마이그레이션에 대한 저장소 모범 사례](apache-hadoop-on-premises-migration-best-practices-storage.md)