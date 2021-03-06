---
author: genlin
ms.service: virtual-network
ms.topic: include
ms.date: 11/09/2018
ms.author: genli
ms.openlocfilehash: 5c1caf87f32ddd827b85263ee634d3e15d821124
ms.sourcegitcommit: fa758779501c8a11d98f8cacb15a3cc76e9d38ae
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/20/2018
ms.locfileid: "52271337"
---
가상 네트워크의 IaaS VM(가상 머신) 및 PaaS 역할 인스턴스는 연결된 서브넷에 따라 지정한 범위의 개인 IP 주소를 자동으로 수신합니다. 해당 주소는 VM 및 역할 인스턴스가 서비스 해제될 때까지 유지됩니다. PowerShell, Azure CLI 또는 Azure 포털에서 중지하여 VM 또는 역할 인스턴스를 서비스 해제합니다. 이러한 경우 VM 또는 역할 인스턴스가 다시 시작되면 Azure 인프라에서 사용 가능한 IP 주소를 수신하며, 이전 IP 주소와 다를 수도 있습니다. 게스트 운영 체제에서 VM 또는 역할 인스턴스를 종료하는 경우 보유한 IP 주소가 유지됩니다.  

VM이 DNS를 실행하거나 도메인 컨트롤러가 되는 경우와 같이 VM 또는 역할 인스턴스에 고정 IP 주소를 지정하려는 경우가 있습니다. 이렇게 하려면 고정 개인 IP 주소를 설정합니다.

