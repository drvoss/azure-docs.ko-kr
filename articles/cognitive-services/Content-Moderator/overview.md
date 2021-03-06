---
title: Azure Content Moderator란?
titlesuffix: Azure Cognitive Services
description: Content Moderator를 사용하여 사용자 생성 콘텐츠에서 부적절한 자료를 추적하고, 플래깅하고, 평가하고, 필터링하는 방법을 알아봅니다.
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: overview
ms.date: 10/22/2018
ms.author: sajagtap
ms.openlocfilehash: 076948e7434802af7f0ad47f279335009817d40e
ms.sourcegitcommit: 6e09760197a91be564ad60ffd3d6f48a241e083b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/29/2018
ms.locfileid: "50209593"
---
# <a name="what-is-azure-content-moderator"></a>Azure Content Moderator란?

Azure Content Moderator API는 자료에서 불쾌감을 주거나, 위험하거나, 원치 않는 텍스트, 이미지 및 비디오 콘텐츠를 확인하는 인식 서비스입니다. 이러한 자료가 발견되면 서비스가 콘텐츠에 적절한 레이블(플래그)을 적용합니다. 그러면 앱이 규정에 부합하고 원하는 사용자 환경을 유지하기 위해 플래그가 지정된 콘텐츠를 처리할 수 있습니다. 다양한 콘텐츠 플래그의 의미를 자세히 알아보려면 [Content Moderator API](#content-moderator-apis) 섹션을 참조하세요.

## <a name="where-it-is-used"></a>사용 위치

다음은 소프트웨어 개발자나 팀이 Content Moderator를 사용하는 몇 가지 시나리오입니다.

- 제품 카탈로그 및 기타 사용자 생성 콘텐츠를 조정하는 온라인 마켓플레이스
- 사용자 생성 게임 아티팩트 및 대화방을 조정하는 게임 회사
- 사용자가 추가한 이미지, 텍스트 및 비디오를 조정하는 소셜 메시징 플랫폼
- 중앙에서 콘텐츠를 조정하도록 구현하는 엔터프라이즈 미디어 회사
- 학생 및 교육자에게 부적절한 콘텐츠를 필터링하는 K-12 교육 솔루션 공급자

## <a name="what-it-includes"></a>포함되는 항목

Content Moderator 서비스는 REST 호출 및 .NET SDK 둘 다를 통해 사용 가능한 몇 가지 웹 서비스 API로 구성됩니다. 또한 검토자 역할을 하는 사람이 서비스를 보조하고 조정 기능을 개선하거나 세부 조정하는 데 사용되는 사용자 검토 도구도 포함되어 있습니다.

![Moderation API, Review API 및 사용자 검토 도구를 보여주는 Content Moderator의 블록 다이어그램](images/content-moderator-block-diagram.png)

### <a name="content-moderator-apis"></a>Content Moderator API

Content Moderator 서비스에는 다음 시나리오를 위한 API가 포함됩니다.

| 조치 | 설명 |
| ------ | ----------- |
|[**텍스트 조정**](text-moderation-api.md)| 텍스트에서 불쾌감을 주는 콘텐츠, 외설적이거나 선정적인 콘텐츠, 불경한 언어 및 PII(개인 식별 정보)를 검사합니다.|
|[**사용자 지정 용어 목록**](try-terms-list-api.md)| 기본 제공 용어 외에 사용자 지정 용어 목록을 기준으로 검사합니다. 사용자 지정 콘텐츠 정책에 따라 사용자 지정 목록을 사용하여 콘텐츠를 차단하거나 허용합니다.|  
|[**이미지 조정**](image-moderation-api.md)| 이미지에서 성인 또는 외설 콘텐츠를 검사하고, OCR(광학 문자 인식) 기능을 사용하여 이미지에서 텍스트를 감지하고, 얼굴을 감지합니다.|
|[**사용자 지정 이미지 목록**](try-image-list-api.md)| 사용자 지정 이미지 목록을 기준으로 이미지를 검사합니다. 사용자 지정 이미지 목록을 사용하여 다시 분류하지 않으려는 일반적인 반복 콘텐츠의 인스턴스를 필터링합니다.|
|[**비디오 조정**](video-moderation-api.md)| 비디오에서 성인 또는 외설 콘텐츠를 검사하고 언급된 콘텐츠의 시간 마커를 반환합니다.|
|[**검토**](try-review-api-job.md)| [작업](try-review-api-job.md), [검토](try-review-api-review.md) 및 [워크플로](try-review-api-workflow.md) 작업을 사용하여 사용자 검토 도구 내에서 인간 참여형 워크플로를 만들고 자동화합니다. Workflow API는 아직 .NET SDK를 통해 사용할 수 없습니다.|

### <a name="human-review-tool"></a>사용자 검토 도구

Content Moderator 서비스에는 웹 기반 [사용자 검토 도구](Review-Tool-User-Guide/human-in-the-loop.md)도 포함됩니다. 

![Content Moderator 사용자 검토 도구 홈 페이지](images/homepage.PNG)

Review API를 사용하면 팀이 지정된 필터에 따라 텍스트, 이미지 및 비디오 콘텐츠를 검토하도록 설정할 수 있습니다. 그런 다음, 조정자 역할을 맡은 사람이 최종 조정 결정을 내릴 수 있습니다. 사용자 입력은 서비스를 학습시키지 않지만 서비스 및 사용자 검토 팀의 협업을 통해 개발자는 효율성과 정확성 사이에 적절히 균형을 이룰 수 있습니다.

## <a name="next-steps"></a>다음 단계

[빠른 시작](quick-start.md)을 따라 Content Moderator 사용을 시작합니다.
