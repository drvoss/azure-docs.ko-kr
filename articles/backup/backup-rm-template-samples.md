---
title: Azure Backup용 Azure Resource Manager 템플릿
description: Azure Backup PowerShell 샘플
services: backup
author: rayne-wiselman
manager: carmonm
ms.service: backup
ms.topic: sample
ms.date: 04/18/2018
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: bf6bca668ff97b30789a99dab2f1f3d409ab0624
ms.sourcegitcommit: b0f39746412c93a48317f985a8365743e5fe1596
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2018
ms.locfileid: "52867918"
---
# <a name="azure-resource-manager-templates-for-azure-backup"></a>Azure Backup용 Azure Resource Manager 템플릿

다음 표에는 Recovery Services 자격 증명 모음 및 Azure Backup 기능과 함께 사용할 수 있는 Azure Resource Manager 템플릿에 대한 링크가 포함되어 있습니다.

|   |   |
|---|---|
|**Recovery Services 자격 증명 모음** | |
| [Recovery Services 자격 증명 모음 만들기](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-vault-create)| Recovery Services 자격 증명 모음을 만듭니다. 자격 증명 모음은 Azure Backup 및 Azure Site Recovery에 사용할 수 있습니다. |
|**가상 머신 설정**| |
| [Resource Manager VM 백업](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-backup-vms) | 기존 Recovery Services 자격 증명 모음 및 백업 정책을 사용하여 동일한 리소스 그룹에 있는 Resource Manager 가상 머신을 백업합니다.|
| [Recovery Services 자격 증명 모음에 IaaS VM 백업](https://github.com/Azure/azure-quickstart-templates/tree/master/201-recovery-services-backup-classic-resource-manager-vms) | 클래식 및 Resource Manager 가상 머신을 백업하기 위한 템플릿입니다. |
| [IaaS VM용 매주 백업 정책 만들기](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-weekly-backup-policy-create) | 클래식 및 Resource Manager 가상 머신을 백업하는 데 사용되는 Recovery Services 자격 증명 모음 및 매주 백업 정책을 만드는 템플릿입니다.|
| [IaaS VM용 매일 백업 정책 만들기](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-daily-backup-policy-create) | 클래식 및 Resource Manager 가상 머신을 백업하는 데 사용되는 Recovery Services 자격 증명 모음 및 매일 백업 정책을 만드는 템플릿입니다.|
| [백업이 활성화된 Windows Server VM 배포](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-create-vm-and-configure-backup) | 기본 백업 정책이 활성화된 Windows Server VM 및 Recovery Services 자격 증명 모음을 만드는 템플릿입니다.|
|**백업 작업 모니터링** |  |
| [Log Analytics를 사용하여 Azure Backup 모니터링](https://github.com/Azure/azure-quickstart-templates/tree/master/101-backup-oms-monitoring) | Recovery Services 자격 증명 모음에서 사용된 백업 및 복원 작업, 백업 경고 및 클라우드 저장소를 모니터링할 수 있는 Azure Backup용 Log Analytics 모니터링을 배포하는 템플릿입니다.|  
|   |   |

