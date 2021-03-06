---
title: Conversation Learner 작업의 "예상 엔터티" 속성을 사용하는 방법 - Microsoft Cognitive Services | Microsoft Docs
titleSuffix: Azure
description: Conversation Learner 모델의 "예상 엔터티" 속성을 사용하는 방법을 알아봅니다.
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: 9e685d2281330457e83abde0a8ac26e086da7479
ms.sourcegitcommit: da3459aca32dcdbf6a63ae9186d2ad2ca2295893
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/07/2018
ms.locfileid: "51229648"
---
# <a name="how-to-use-the-expected-entity-property-of-actions"></a>작업의 "예상 엔터티" 속성을 사용하는 방법

이 자습서에서는 작업의 "예상 엔터티" 필드를 설명합니다.

## <a name="video"></a>비디오

[![자습서 4 미리 보기](https://aka.ms/cl-tutorial-04-preview)](https://aka.ms/blis-tutorial-04)

## <a name="requirements"></a>요구 사항
이 자습서를 수행하려면 일반 자습서 봇이 실행 중이어야 합니다.

    npm run tutorial-general

## <a name="details"></a>세부 정보
작업의 "예상 엔터티" 필드를 사용하여 작업에 대한 사용자의 응답이 엔터티의 상태라고 예상하는 시스템과 통신합니다.

구체적으로 작업의 "예상 엔터티" 필드를 $entity로 설정한 경우 다음 사용자 발언에서 시스템은 다음과 같습니다.

1. 먼저 다른 경우와 같이 기계 학습 기반 엔터티 추출 모델을 사용하여 엔터티를 찾으려고 시도합니다.
2. 1단계에 엔터티가 없는 경우 추론적으로 전체 사용자 발언을 $entity에 할당합니다.
3. 다른 경우와 같이 `EntityDetectionCallback`을 호출하고 작업 선택 영역을 계속 진행합니다.

## <a name="steps"></a>단계

### <a name="create-the-model"></a>모델 만들기

1. Web UI에서 새 모델을 클릭합니다.
2. 이름에 ExpectedEntities를 입력합니다. 그런 다음, 만들기를 클릭합니다.

### <a name="create-an-entity"></a>엔터티 만들기

1. 엔터티, 새 엔터티를 차례로 클릭합니다.
2. 엔터티 이름에 이름을 입력합니다.
3. 만들기 클릭 

> [!NOTE]
> 엔터티 형식은 '사용자 지정'입니다. 이 값은 엔터티를 학습할 수 있음을 의미합니다.  동작을 조정할 수 없는 미리 빌드된 엔터티도 있습니다.  이러한 엔터티는 [사전 빌드 엔터티 자습서](./7-built-in-entities.md)에서 설명합니다.

![](../media/tutorial4_entities.PNG)

### <a name="create-two-actions"></a>두 가지 작업 만들기

1. 작업, 새 작업을 차례로 클릭합니다.
2. 응답에서 '이름은 무엇인가요?'를 입력합니다.
3. 예상 엔터티에서 $name을 입력합니다. 저장을 클릭합니다.
    - 이 값은 질문을 할 경우 사용자 응답에서 엔터티가 검색되지 않음을 의미합니다. 봇은 사용자의 응답 전체가 이 엔터티라고 가정해야 합니다.
    - 엔터티가 실격 엔터티로 자동으로 추가됩니다. 
2. 작업, 새 작업을 차례로 클릭하여 두 번째 작업을 만듭니다.
3. 응답에 '안녕하세요 $name'을 입력합니다.
    - 엔터티가 필수 엔터티로 자동으로 추가됩니다.
4. 저장을 클릭합니다.

이제 두 가지 작업이 있습니다.

![](../media/tutorial4_actions.PNG)

### <a name="train-the-bot"></a>봇 학습

1. 학습 대화 상자, 새 학습 대화 상자를 차례로 클릭합니다.
2. '안녕하세요'를 입력합니다.
3. 작업에 점수 지정을 클릭하고 '이름이 무엇인가요?'를 선택합니다.
    - 엔터티 $name을 정의해야 하고 $name이 봇의 메모리에 없기 때문에 응답 '안녕하세요 $name'를 선택할 수 없습니다.
2. 'david'를 입력합니다. 
    - 이름을 엔터티로 강조 표시합니다. 응답을 엔터티로 선택하기 위해 위에 설정한 추론 때문입니다.
5. 작업에 점수 지정을 클릭합니다.
    - 이제 이름 값이 봇의 메모리에 있습니다.
    - 이제 '안녕하세요 $name'을 응답으로 사용할 수 있습니다. 
6. '안녕하세요 $name'을 선택합니다.
7. 학습 완료를 클릭합니다.

"예상 엔터티" 추론이 트리거되지 않도록 기계 학습 엔터티 추출 모델이 이름을 식별하는 두 가지 예제는 다음과 같습니다.

1. 새 학습 대화를 클릭합니다.
2. '내 이름은 david입니다.'를 입력합니다.
    - 모델은 전에 이 단어를 확인했기 때문에 david를 이름 엔터티로 식별합니다.
2. 작업에 점수 지정을 클릭합니다.
3. '안녕하세요 $name'을 선택합니다.
4. '내 이름은 susan입니다.'를 입력합니다.
    - 모델은 이 패턴을 이미 확인했으므로 susan을 이름으로 식별합니다.
2. 작업에 점수 지정을 클릭합니다.
2. '안녕하세요 susan'을 선택합니다.
3. 학습 완료를 클릭합니다.

다음 예제에서 "예상 엔터티" 추론이 트리거되지만 올바르지 않습니다. 그런 후 수정 방법이 제공됩니다.

1. 'jose라고 합니다.'를 입력합니다.
    - 모델은 이름을 엔터티로 인식하지 못합니다.
2. jose를 클릭하고 이름을 선택합니다.
3. 작업에 점수 지정을 클릭합니다.
4. 안녕하세요 $name을 선택합니다.
5. 학습 완료를 클릭합니다.
1. 새 학습 대화를 클릭합니다.
2. '안녕하세요'를 입력합니다.
3. '이름이 무엇인가요'에 대한 응답으로 'frank라고 합니다.'를 입력합니다.
    - 전체 구문이 강조 표시됩니다. 통계 모델이 이름을 찾지 못했기 때문입니다. 따라서 추론이 발생하고 전체 응답을 이름 엔터티로 선택합니다.
2. 이를 해결하려면 강조 표시된 구문을 클릭한 다음, 빨간색 휴지통 아이콘을 클릭합니다. 
3. frank를 선택하도록 클릭한 다음, 이름을 클릭합니다.
2. 작업에 점수 지정을 클릭합니다.
3. '안녕하세요 $name'을 선택합니다.
4. 학습 완료를 클릭합니다.

![](../media/tutorial4_dialogs.PNG)

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [무효화할 수 엔터티](./5-negatable-entities.md)
