---
author: dlepow
ms.service: batch
ms.topic: include
ms.date: 11/09/2018
ms.author: danlep
ms.openlocfilehash: 7d90ccf55b10eff4299692b481a25ba4cdf41b8a
ms.sourcegitcommit: 8d88a025090e5087b9d0ab390b1207977ef4ff7c
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/21/2018
ms.locfileid: "52279781"
---
## <a name="get-account-credentials"></a>계정 자격 증명 가져오기

이 예제에서는 배치 계정과 저장소 계정에 대한 자격 증명을 제공해야 합니다. 필요한 자격 증명을 가져오는 간단한 방법은 Azure Portal에 있습니다. (Azure API 또는 명령줄 도구를 사용하여 이러한 자격 증명을 가져올 수도 있습니다.)

1. **모든 서비스** > **배치 계정**을 차례로 클릭한 다음, 배치 계정의 이름을 클릭합니다.

2. Batch 자격 증명을 보려면 **키**를 클릭합니다. **배치 계정**, **URL** 및 **기본 액세스 키**의 값을 텍스트 편집기에 복사합니다.

3. 저장소 계정 이름 및 키를 보려면 **저장소 계정**을 클릭합니다. **저장소 계정 이름** 및 **Key1**의 값을 텍스트 편집기에 복사합니다.