---
title: Azure의 IBM Db2 pureScale
description: 이 문서에서는 Azure에서 IBM Db2 pureScale 환경을 실행하기 위한 아키텍처를 보여 줍니다.
services: virtual-machines-linux
documentationcenter: ''
author: njray
manager: edprice
editor: edprice
tags: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 11/09/2018
ms.author: njray
ms.openlocfilehash: 5a7ae96a3730df2d0dae04dde6d7609ce69cdee5
ms.sourcegitcommit: 8314421d78cd83b2e7d86f128bde94857134d8e1
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/19/2018
ms.locfileid: "51978216"
---
# <a name="ibm-db2-purescale-on-azure"></a>Azure의 IBM Db2 pureScale

IBM Db2 pureScale 환경은 Linux 운영 체제에서 Azure용 데이터베이스 클러스터 솔루션에 고가용성 및 확장성을 제공합니다. 이 문서에서는 Azure에서 Db2 pureScale를 실행하기 위한 아키텍처를 보여 줍니다.

## <a name="overview"></a>개요

엔터프라이즈는 오랫동안 기존의 RDBMS(관계형 데이터베이스 관리 시스템) 플랫폼을 사용하여 OLTP(온라인 트랜잭션 처리) 요구를 충족했습니다. 요즘에는 많은 기업들이 용량을 확장하고, 비용을 절감하고, 안정적인 운영 비용 구조를 유지 관리하기 위한 방법으로 메인프레임 기반 데이터베이스 환경을 Azure로 마이그레이션하고 있습니다.

종종 마이그레이션은 기존 플랫폼을 현대화하는 첫 번째 단계로 사용됩니다. 예를 들어, 한 엔터프라이즈는 최근에 z/OS에서 실행되는 IBM Db2 환경을 Azure의 IBM Db2 pureScale로 다시 호스트했습니다. 원본 환경과 동일하지 않더라도, Linux의 IBM Db2 pureScale은 메인프레임의 병렬 Sysplex 구성에서 실행되는 z/OS용 IBM Db2와 비슷한 고가용성 및 확장성 기능을 제공합니다.

이 문서는 이 Azure 마이그레이션에 사용되는 아키텍처를 설명합니다. Red Hat Linux 7.4는 구성을 테스트하는 데 사용되었습니다. 이 버전은 Azure Marketplace에서 사용할 수 있습니다. Linux 배포판을 선택하기 전에 현재 지원되는 버전을 확인해야 합니다. 자세한 내용은 [IBM Db2 pureScale](https://www.ibm.com/support/knowledgecenter/SSEPGG) 및 [GlusterFS](https://docs.gluster.org/en/latest/)에 대한 설명서를 참조하세요.

이 문서는 Db2 구현 계획을 위한 시작점에 불과합니다. 사용자의 비즈니스 요구 사항은 다르지만 동일한 기본 패턴이 적용됩니다. 이러한 아키텍처 패턴을 Azure의 OLAP(온라인 분석 처리) 애플리케이션에서도 사용할 수 있습니다.

이 문서에서 z/OS용 IBM DB2 데이터베이스를 Linux에서 실행되는 IBM Db2 pureScale로 이동하기 위한 가능한 마이그레이션 작업 및 기능의 차이점은 다루지 않습니다. Db2 z/OS에서 Db2 pureScale로 이동할 때의 크기 예측 및 작업 분석도 제공하지 않습니다. 작업 환경에 가장 적합한 Db2 pureScale 아키텍처를 결정하는 데 도움을 주기 위해 전체 크기 예측 연습 과정을 마치고 가설을 설정할 것을 강력히 권장합니다. 원본 시스템에서는 데이터 공유 아키텍처, 결합 기능 구성 및 DDF 사용량 통계를 포함하는 Db2 z/OS Parallel Sysplex를 고려해야 합니다.

> [!NOTE]
> 이 문서에서는 Db2 마이그레이션 방법 중 하나를 설명하지만 다른 방법도 있습니다. 예를 들어, Db2 pureScale를 가상화된 온-프레미스 환경에서 실행할 수도 있습니다. IBM은 다양한 구성의 Microsoft Hyper-V에서 Db2를 지원합니다. 자세한 내용은 IBM 지식 센터에서 [Db2 pureScale 가상화 아키텍처](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.qb.server.doc/doc/r0061462.html)를 참조하세요.

## <a name="architecture"></a>아키텍처

Azure에서 고가용성 및 확장성을 지원하기 위해 Db2 pureScale에 대해 스케일 아웃 공유 데이터 아키텍처를 사용할 수 있습니다. 다음 예제 아키텍처는 고객 마이그레이션에 사용되었습니다.

![](media/db2-purescale-on-azure/pureScaleArchitecture.png "스토리지 및 네트워킹을 보여 주는 Azure Virtual Machines의 Db2 pureScale")


이 아키텍처 다이어그램에서는 Db2 pureScale 클러스터에 필요한 논리 계층을 보여 줍니다. 여기에는 클라이언트한, 관리, 캐싱, 데이터베이스 엔진 및 공유 스토리지에 대한 가상 머신이 포함됩니다. 이 다이어그램에는 데이터베이스 엔진 노드 외에, 클러스터 CF(캐싱 기능)로 알려진 기능에 사용되는 두 개의 노드가 포함됩니다. 데이터베이스 엔진 자체에는 최소 2개의 노드가 사용됩니다. pureScale 클러스터에 속하는 Db2 서버를 멤버라고 합니다. 클러스터는 스케일 아웃 스토리지와 고가용성을 제공하기 위해 iSCSI를 통해 3노드 GlusterFS 공유 스토리지 클러스터에 연결됩니다. Db2 pureScale는 Linux를 실행하는 Azure Virtual Machines에 설치됩니다.

이 방법은 조직이 요구하는 크기 및 규모에 맞게 수정할 수 있는 템플릿으로, 다음을 기준으로 합니다.

-   둘 이상의 데이터베이스 멤버가 2개 이상의 CF 노드에 결합됩니다. 이 노드는 공유 메모리에 대한 GBP(전역 버퍼 풀)과 GLM(전역 잠금 관리자) 서비스를 관리하여 여러 활성 멤버의 공유 액세스 및 잠금 경합을 제어합니다. 하나의 CF 노드가 주 노드로 사용되고 다른 CF 노드가 보조 장애 조치(failover) CF 노드로 사용됩니다. 단일 실패 지점이 있는 환경을 만들지 않으려면 Db2 pureScale 클러스터에 대해 최소 4개의 노드가 필요합니다.

-   Gluster FS 노드 각각에 사용되는 고성능 공유 스토리지(그림 1에서 P30 크기로 표시됨)

-   데이터 멤버 및 공유 스토리지에 대한 고성능 네트워킹

### <a name="compute-considerations"></a>컴퓨팅 고려 사항

이 아키텍처는 Azure Virtual Machines에서 애플리케이션, 스토리지 및 데이터 계층을 실행합니다. [배포 설정 스크립트](http://aka.ms/db2onazure)는 다음을 만듭니다.

-   Db2 pureScale 클러스터. Azure에서 필요한 컴퓨팅 리소스의 형식은 설정에 따라 달라집니다. 일반적으로 다음 두 가지 접근 방법을 사용할 수 있습니다.

    -   여러 중소규모 인스턴스가 공유 스토리지에 액세스하는 다중 노드, HPC(고성능 컴퓨팅) 스타일 네트워크를 사용합니다. 이 HPC 유형의 구성에서 Azure 메모리 최적화 E 시리즈 또는 스토리지 최적화 L 시리즈 [가상 머신](https://docs.microsoft.com/azure/virtual-machines/windows/sizes)은 필요한 컴퓨팅 성능을 제공합니다.

    -   데이터 엔진에 대해 더 적은 수의 대형 가상 머신 인스턴스를 사용합니다. 대형 인스턴스의 경우 가장 큰 메모리 최적화 [M 시리즈](https://azure.microsoft.com/pricing/details/virtual-machines/series/) 가상 머신이 과도한 메모리 내 작업에 적합하지만 Db2를 실행하는 데 사용되는 LPAR(논리적 파티션) 크기에 따라, 전용 인스턴스가 필요할 수도 있습니다.

-   Db2 CF는 E 시리즈 또는 L 시리즈와 같은 메모리 최적화 가상 머신을 사용합니다.

-   GlusterFS 스토리지는 Linux를 실행하는 Standard\_DS4\_v2 가상 머신을 사용합니다.

-   GlusterFS Jumpbox는 Linux를 실행하는 Standard\_DS2\_v2 가상 머신입니다.

-   클라이언트는 Windows를 실행하는 Standard\_DS3\_v2 가상 머신(테스트용)입니다.

-   미러링 모니터 서버는 Linux를 실행하는 Standard\_DS3\_v2 가상 머신(Db2 pureScale에 사용)입니다.

> [!NOTE]
> Db2 pureScale 클러스터에는 최소 두 개의 Db2 인스턴스가 필요합니다. 캐시 인스턴스 및 잠금 관리자 인스턴스도 필요합니다.

### <a name="storage-considerations"></a>저장소 고려 사항

Oracle RAC와 같이, Db2 pureScale는 고성능 블록 I/O를 스케일 아웃 데이터베이스입니다. 요구에 적합한 가장 큰 [Azure Premium Storage](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage)를 사용하는 것이 좋습니다. 예를 들어, 프로덕션 환경에서는 종종 더 큰 스토리지 용량이 필요하지만, 개발 및 테스트 환경에서는 더 작은 스토리지 옵션이 적합할 수 있습니다. 이 예제 아키텍처는 크기 및 가격 대비 IOPS 비율 때문에 [P30](https://azure.microsoft.com/pricing/details/managed-disks/)을 사용합니다. 크기에 관계없이, 최상의 성능을 얻으려면 Premium Storage를 사용합니다.

Db2 pureScale는 모든 클러스터 노드에서 모든 데이터에 액세스할 수 있는 모두 공유 아키텍처를 사용합니다. Premium Storage는 주문형이든 또는 전용 인스턴스이든 관계없이 여러 인스턴스에서 공유되어야 합니다.

대형 Db2 pureScale 클러스터에는 IOPS가 100,000인 200TB(테라바이트) 이상의 Premium 공유 Storage가 필요할 수 있습니다. Db2 pureScale는 Azure에서 사용할 수 있는 iSCSI 블록 인터페이스를 지원합니다. iSCSI 인터페이스에는 GlusterFS, S2D 또는 다른 도구를 사용하여 구현할 수 있는 공유 스토리지 클러스터가 필요합니다. 이 유형의 솔루션은 Azure에서 vSAN(가상 저장 영역 네트워크) 디바이스를 만듭니다. Db2 pureScale는 vSAN을 사용하여 여러 가상 머신 간에 데이터를 공유하는 데 사용되는 클러스터형 파일 시스템을 설치합니다.

이 아키텍처의 경우 특히 클러스터 스토리지에 최적화된 무료, 확장 가능, 오픈 소스 분산 파일 시스템인 GlusterFS 파일 시스템을 사용했습니다.

### <a name="networking-considerations"></a>네트워킹 고려 사항

IBM은 Db2 pureScale 클러스터의 모든 멤버에 InfiniBand 네트워킹을 권장합니다. Db2 pureScale은 또한 사용되는 CF에 대해 RDMA(원격 직접 메모리 액세스)를 사용합니다(사용 가능한 경우).

설치 중에 모든 가상 머신을 포함하는 Azure [리소스 그룹](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)이 만들어집니다. 일반적으로 리소스는 수명 및 관리하는 주체에 따라 그룹화됩니다. 이 아키텍처의 가상 머신은 SR-IOV(단일 루트 I/O 가상화)를 통해 가상 머신에 일관되게 매우 짧은 네트워크 대기 시간을 제공하는 Azure 기능인 [가속 네트워킹](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/)이 필요합니다.

모든 Azure Virtual Machine은 여러 서브넷, 즉 주 서브넷, Gluster FS 프런트 엔드(gfsfe), Gluster FS 백 엔드(bfsbe), Db2 pureScale(db2be) 및 Db2 purescale 프런트 엔드(db2fe)로 분할되는 가상 네트워크에 배포됩니다. 설치 스크립트는 주 서브넷의 가상 머신에 주 [NIC](https://docs.microsoft.com/azure/virtual-machines/linux/multiple-nics)도 만듭니다.

NSG([네트워크 보안 그룹](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg))는 가상 네트워크 내의 네트워크 트래픽을 제한하고 서브넷을 격리하는 데 사용됩니다.

Azure에서 Db2 pureScale는 스토리지에 대한 네트워크 연결로 TCP/IP를 사용해야 합니다.

## <a name="next-steps"></a>다음 단계

-   [Azure에서 이 아키텍처 배포](deploy-ibm-db2-purescale-azure.md)
