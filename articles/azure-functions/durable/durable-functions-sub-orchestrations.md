---
title: 지속성 함수의 하위 오케스트레이션 - Azure
description: Azure Functions의 지속성 함수 확장에서 오케스트레이션을 호출하는 방법을 설명합니다.
services: functions
author: kashimiz
manager: jeconnoc
keywords: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 10/23/2018
ms.author: azfuncdf
ms.openlocfilehash: 32f8872737fdf6dd766ae8df8ef3ed47692e2c9c
ms.sourcegitcommit: c8088371d1786d016f785c437a7b4f9c64e57af0
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/30/2018
ms.locfileid: "52636968"
---
# <a name="sub-orchestrations-in-durable-functions-azure-functions"></a>지속성 함수의 하위 오케스트레이션(Azure Functions)

오케스트레이터 함수는 작업 함수를 호출하는 것 외에도 다른 오케스트레이터 함수를 호출할 수 있습니다. 예를 들어 오케스트레이터 함수 라이브러리에서 더 큰 오케스트레이션을 작성할 수 있습니다. 또는 오케스트레이터 함수의 여러 인스턴스를 병렬로 실행할 수 있습니다.

오케스트레이터 함수는 [CallSubOrchestratorAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_CallSubOrchestratorAsync_) 또는 [CallSubOrchestratorWithRetryAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_CallSubOrchestratorWithRetryAsync_) 메서드를 호출하여 다른 오케스트레이터 함수를 호출할 수 있습니다. [오류 처리 및 보정](durable-functions-error-handling.md#automatic-retry-on-failure) 문서에서는 자동 다시 시도에 대해 자세히 설명하고 있습니다.

하위 오케스트레이터 함수는 호출자의 관점에서 작업 함수처럼 작동합니다. 이러한 함수는 값을 반환하고, 예외를 throw하며, 부모 오케스트레이터 함수에서 기다릴 수 있습니다.

> [!NOTE]
> `CallSubOrchestratorAsync` 및 `CallSubOrchestratorWithRetryAsync` 메서드는 아직 JavaScript에서 사용할 수 없습니다.

## <a name="example"></a>예

다음 예제에서는 프로비전해야 하는 여러 디바이스가 있는 IoT("사물 인터넷") 시나리오를 보여줍니다. 다음과 같이 각 디바이스마다 수행해야 하는 특정 오케스트레이션이 있습니다.

#### <a name="c"></a>C#

```csharp
public static async Task DeviceProvisioningOrchestration(
    [OrchestrationTrigger] DurableOrchestrationContext ctx)
{
    string deviceId = ctx.GetInput<string>();

    // Step 1: Create an installation package in blob storage and return a SAS URL.
    Uri sasUrl = await ctx.CallActivityAsync<Uri>("CreateInstallationPackage", deviceId);

    // Step 2: Notify the device that the installation package is ready.
    await ctx.CallActivityAsync("SendPackageUrlToDevice", Tuple.Create(deviceId, sasUrl));

    // Step 3: Wait for the device to acknowledge that it has downloaded the new package.
    await ctx.WaitForExternalEvent<bool>("DownloadCompletedAck");

    // Step 4: ...
}
```

#### <a name="javascript-functions-v2-only"></a>JavaScript(Functions v2만 해당)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const deviceId = context.df.getInput();

    // Step 1: Create an installation package in blob storage and return a SAS URL.
    const sasUrl = yield context.df.callActivity("CreateInstallationPackage", deviceId);

    // Step 2: Notify the device that the installation package is ready.
    yield context.df.callActivity("SendPackageUrlToDevice", { id: deviceId, url: sasUrl });

    // Step 3: Wait for the device to acknowledge that it has downloaded the new package.
    yield context.df.waitForExternalEvent("DownloadCompletedAck");

    // Step 4: ...
});
```

이 오케스트레이터 함수는 일회용 디바이스 프로비전을 위해 그대로 사용하거나 더 큰 오케스트레이션의 일부로 포함될 수 있습니다. 후자의 경우 부모 오케스트레이터 함수는 `CallSubOrchestratorAsync`(C#) 또는 `callSubOrchestrator`(JS) API를 사용하여 `DeviceProvisioningOrchestration` 인스턴스를 예약할 수 있습니다.

다음은 여러 오케스트레이터 함수를 병렬로 실행하는 방법을 보여 주는 예제입니다.

#### <a name="c"></a>C#

```csharp
[FunctionName("ProvisionNewDevices")]
public static async Task ProvisionNewDevices(
    [OrchestrationTrigger] DurableOrchestrationContext ctx)
{
    string[] deviceIds = await ctx.CallActivityAsync<string[]>("GetNewDeviceIds");

    // Run multiple device provisioning flows in parallel
    var provisioningTasks = new List<Task>();
    foreach (string deviceId in deviceIds)
    {
        Task provisionTask = ctx.CallSubOrchestratorAsync("DeviceProvisioningOrchestration", deviceId);
        provisioningTasks.Add(provisionTask);
    }

    await Task.WhenAll(provisioningTasks);

    // ...
}
```

#### <a name="javascript-functions-v2-only"></a>JavaScript(Functions v2만 해당)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const deviceIds = yield context.df.callActivity("GetNewDeviceIds");

    // Run multiple device provisioning flows in parallel
    const provisioningTasks = [];
    for (const deviceId of deviceIds) {
        const provisionTask = context.df.callSubOrchestrator("DeviceProvisioningOrchestration", deviceId);
        provisioningTasks.push(provisionTask);
    }

    yield context.df.Task.all(provisioningTasks);

    // ...
});
```

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [작업 허브 정의 및 구성](durable-functions-task-hubs.md)
