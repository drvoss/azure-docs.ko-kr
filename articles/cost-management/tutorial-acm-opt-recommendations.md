---
title: 자습서 - 최적화 권장 사항으로 Azure 비용 절감 | Microsoft Docs
description: 이 자습서는 최적화 권장 사항을 따를 경우 Azure 비용을 절감할 수 있도록 도와줍니다.
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 09/21/2018
ms.topic: conceptual
ms.service: cost-management
manager: dougeby
ms.custom: ''
ms.openlocfilehash: 4d9e47d6da45eaba19cbe089de3fdf053c36046a
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/24/2018
ms.locfileid: "47030680"
---
# <a name="tutorial-optimize-costs-from-recommendations"></a>자습서: 권장 사항에서 비용 최적화

Azure Cost Management는 Azure Advisor와 함께 실행되어 비용 최적화 권장 사항을 제공합니다. Azure Advisor는 유휴 리소스와 사용률이 낮은 리소스를 식별하여 최적화하고 효율성을 개선하는 데 도움이 됩니다. 이 자습서에서는 사용률이 낮은 Azure 리소스를 식별한 다음, 비용을 절감하기 위한 조치를 취하는 예를 안내합니다.

이 자습서에서는 다음 방법에 대해 알아봅니다.

> [!div class="checklist"]
> * 비용 최적화 권장 사항을 검토하여 잠재적인 사용 비효율성 확인인
> * 가상 머신을 비용 효율적인 옵션으로 크기를 조정하는 권장 사항 이행
> * 가상 머신의 크기가 성공적으로 조정되었는지 확인하기 위해 작업 확인

## <a name="prerequisites"></a>필수 조건
권장 사항은 모든 [EA(기업계약) 고객](https://azure.microsoft.com/pricing/enterprise-agreement/)이 사용할 수 있습니다. 비용 데이터를 보려면 다음 범위 중 하나 이상에 대해 최소한 읽기 권한이 있어야 합니다.

- 구독
- 리소스 그룹

14일 이상 활동이 있는 활성 가상 머신이 있어야 합니다.

## <a name="sign-in-to-azure"></a>Azure에 로그인
[https://portal.azure.com](https://portal.azure.com/)에서 Azure Portal에 로그인합니다.

## <a name="view-cost-optimization-recommendations"></a>비용 최적화 권장 사항 보기

Azure Portal의 서비스 목록에서 **Cost Management + 청구**를 클릭합니다. 그런 다음, **Cost Management**의 목록에서 **Advisor 권장 사항**을 선택합니다. Advisor 비용 권장 사항이 표시됩니다.

![Advisor 권장 사항](./media/tutorial-acm-opt-recommendations/advisor-recommendations.png)

권장 사항 목록은 추가 비용을 절감할 수 있도록 구매 권장 사항을 표시하거나 사용 비효율성을 식별합니다. 총 **연간 절약 가능 금액**은 권장 사항 규칙을 충족하는 모든 VM을 종료 또는 할당 해제할 경우 절약할 수 있는 총액을 보여 줍니다. 종료하고 싶지 않다면 덜 비싼 VM SKU로 크기 조정을 고려해야 합니다.

**연간 절약 가능 금액**과 함께, **영향** 범주는 가능한 많이 절약 가능한 권장 사항을 식별하기 위한 것입니다. 영향이 높은 권장 사항은 [종량제 비용에 따라 비용을 절감하기 위해 예약 가상 머신 인스턴스 구매](../advisor/advisor-cost-recommendations.md#buy-reserved-virtual-machine-instances-to-save-money-over-pay-as-you-go-costs) 및 [사용률이 낮은 인스턴스의 크기를 조정하여 가상 머신 소비 최적화](../advisor/advisor-cost-recommendations.md#optimize-virtual-machine-spend-by-resizing-or-shutting-down-underutilized-instances)입니다. 영향이 중간인 권장 사항은 [프로비저닝되지 않은 ExpressRoute 회로를 제거하여 비용 절감](../advisor/advisor-cost-recommendations.md#reduce-costs-by-eliminating-unprovisioned-expressroute-circuits) 및 [유휴 가상 네트워크 게이트웨이를 삭제 또는 재구성하여 비용 절감](../advisor/advisor-cost-recommendations.md#reduce-costs-by-deleting-or-reconfiguring-idle-virtual-network-gateways)입니다.

## <a name="act-on-a-recommendation"></a>권장 사항 이행

Azure Advisor는 14일 동안 가상 머신 사용량을 모니터링한 다음, 사용률이 낮은 가상 머신을 식별합니다. 4일 이상 CPU 사용률이 5% 이하이고 네트워크 사용량이 7MB 이하인 가상 머신은 사용률이 낮은 가상 머신으로 간주됩니다.

CPU 사용률 5% 이하 설정은 기본값이지만, 설정을 조정할 수 있습니다. 설정 조정 방법에 대한 자세한 내용은 [사용량이 낮은 가상 머신 권장 사항](../advisor/advisor-get-started.md#configure-the-average-cpu-utilization-rule-for-the-low-usage-virtual-machine-recommendation)에 대한 [평균 CPU 사용률 규칙 구성](../advisor/advisor-get-started.md#configure-the-average-cpu-utilization-rule-for-the-low-usage-virtual-machine-recommendation) 문서를 참조하세요.

일부 시나리오에서는 기본적으로 사용률이 낮을 수 있으나 가상 머신의 크기를 덜 비싼 크기로 변경하여 비용을 절감할 수도 있습니다. 크기 조정 작업을 선택할 경우 실제 절감액이 달라질 수 있습니다. 가상 머신의 크기 조정 예를 살펴보겠습니다.

권장 사항 목록에서 **사용률이 낮은 가상 머신을 적절하게 크기 조정하거나 종료**를 클릭합니다. 가상 머신 후보 목록에서 크기 조정할 가상 머신을 선택한 다음, 해당 가상 머신을 클릭합니다. 사용률 메트릭을 확인할 수 있도록 가상 머신의 세부 정보가 표시됩니다. **연간 절약 가능 금액** 값은 VM을 종료 또는 제거할 경우 절약할 수 있는 금액입니다. VM 크기 조정으로 비용이 절감될 수는 있지만, 연간 절약 가능한 전체 금액이 절감되지는 않습니다.

![권장 사항 세부 정보](./media/tutorial-acm-opt-recommendations/recommendation-details.png)

VM 세부 정보에서 가상 머신의 사용률을 확인하여 적합한 크기 조정 후보인지 확인하세요.

![VM 세부 정보](./media/tutorial-acm-opt-recommendations/vm-details.png)

현재 가상 머신의 크기를 기록합니다. 해당 가상 머신을 크기 조정해야 하는지 확인한 후 VM 세부 정보를 닫으면 가상 머신 목록이 나타납니다.

종료 또는 크기 조정할 후보 목록에서 **가상 머신 크기 조정**을 선택합니다.
![가상 머신 크기 조정](./media/tutorial-acm-opt-recommendations/resize-vm.png)

다음으로, 사용할 수 있는 크기 조정 옵션 목록이 표시됩니다. 사용자 시나리오에 가장 적합한 성능과 비용 효율성을 제공할 옵션을 선택합니다. 다음 예제에서 선택한 옵션은 **DS14\_V2**에서 **DS13\_V2**로 크기를 조정합니다. 권장 사항을 따르면 매월 551.30달러 또는 연간 6,615.60달러가 절감됩니다.

![크기 선택](./media/tutorial-acm-opt-recommendations/choose-size.png)

적합한 크기를 선택한 후 **선택**을 클릭하여 크기 조정 작업을 시작합니다.

크기 조정을 수행하려면 현재 실행 중인 가상 머신을 다시 시작해야 합니다. 가상 머신이 프로덕션 환경에 있는 경우에는 업무 시간 이후에 크기 조정 작업을 실행하는 것이 좋습니다. 다시 시작을 예약하면 일시적인 사용 불가로 인한 중단을 줄일 수 있습니다.

## <a name="verify-the-action"></a>작업 확인

VM 크기 조정이 성공적으로 완료되면 Azure 알림이 표시됩니다.

![크기 조정 완료 알림](./media/tutorial-acm-opt-recommendations/resized-notification.png)

## <a name="next-steps"></a>다음 단계

이 자습서에서는 다음 방법에 대해 알아보았습니다.

> [!div class="checklist"]
> * 비용 최적화 권장 사항을 검토하여 잠재적인 사용 비효율성 확인인
> * 가상 머신을 비용 효율적인 옵션으로 크기를 조정하는 권장 사항 이행
> * 가상 머신의 크기가 성공적으로 조정되었는지 확인하기 위해 작업 확인

Cost Management 모범 사례 문서를 아직 읽지 않았다면 이 모범 사례는 비용 관리를 지원하기 위해 고려할 높은 수준의 지침과 원칙을 제공합니다.

> [!div class="nextstepaction"]
> [Cost Management 모범 사례](cost-mgt-best-practices.md)
