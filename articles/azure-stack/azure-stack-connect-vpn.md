---
title: Azure Stack VPN을 사용 하 여 Azure에 연결
description: Azure Stack에서 가상 네트워크 VPN을 사용 하 여 Azure에서 가상 네트워크를 연결 하는 방법입니다.
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/24/2018
ms.author: sethm
ms.reviewer: scottnap
ms.openlocfilehash: d215af253471258e487dadcfae0cfd7edafd1c26
ms.sourcegitcommit: 0b7fc82f23f0aa105afb1c5fadb74aecf9a7015b
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2018
ms.locfileid: "51634654"
---
# <a name="connect-azure-stack-to-azure-using-vpn"></a>Azure Stack VPN을 사용 하 여 Azure에 연결

*적용 대상: Azure Stack 통합 시스템*

이 문서에서는 Azure에서 가상 네트워크에 Azure Stack에서 가상 네트워크를 연결 하려면 사이트 간 VPN을 만드는 방법을 보여 줍니다.

## <a name="before-you-begin"></a>시작하기 전에

연결 구성을 완료 하려면 시작 하기 전에 다음 항목이 있는지 확인 합니다.

* Azure Stack 인터넷에 직접 연결 된 시스템 (다중 노드) 배포를 통합 합니다. 외부 공용 IP 주소 범위에 공용 인터넷에서 직접 연결할 수 있어야 합니다.
* 유효한 Azure 구독. Azure 구독이 없는, 하는 경우 만들 수 있습니다는 [무료 Azure 계정에 여기서](https://azure.microsoft.com/free/?b=17.06)합니다.

### <a name="vpn-connection-diagram"></a>VPN 연결 다이어그램

다음 그림은 연결 구성 모양을 완료 되 면:

![사이트 간 VPN 연결 구성](media/azure-stack-connect-vpn/image2.png)

### <a name="network-configuration-example-values"></a>네트워크 구성 예제 값

네트워크 구성 예에서는 테이블에는이 문서의 예제에 사용 되는 값을 보여 줍니다. 이러한 값을 사용 하거나 잘 이해 하기 위해이 문서의 예제를 참조할 수 있습니다.

|   |Azure Stack|Azure|
|---------|---------|---------|
|가상 네트워크 이름     |Azs-VNet|AzureVNet |
|가상 네트워크 주소 공간 |10.1.0.0/16|10.100.0.0/16|
|서브넷 이름     |FrontEnd|FrontEnd|
|서브넷 주소 범위|10.1.0.0/24 |10.100.0.0/24 |
|게이트웨이 서브넷      |10.1.1.0/24|10.100.1.0/24|

## <a name="create-the-network-resources-in-azure"></a>Azure에서 네트워크 리소스 만들기

먼저 Azure에 대 한 네트워크 리소스를 만듭니다. 다음 지침을 사용 하 여 리소스를 만드는 방법을 표시 합니다 [Azure portal](https://portal.azure.com/)합니다.

### <a name="create-the-virtual-network-and-virtual-machine-vm-subnet"></a>가상 네트워크 및 가상 머신 (VM) 서브넷 만들기

1. 에 로그인 합니다 [Azure portal](https://portal.azure.com/) Azure 계정을 사용 합니다.
2. 사용자 포털에서 선택 **+ 리소스 만들기**합니다.
3. 로 이동 **Marketplace**를 선택한 후 **네트워킹**합니다.
4. **가상 네트워크**를 선택합니다.
5. 네트워크 구성 테이블에서 정보를 사용 하 여 Azure에 대 한 값을 식별 **이름을**, **주소 공간**합니다 **서브넷 이름**, 및 **서브넷 주소 범위**합니다.
6. 에 대 한 **리소스 그룹**, 새 리소스 그룹을 만들거나 이미 있는 경우 하나를 선택 **기존 항목 사용**합니다.
7. 선택 된 **위치** VNet입니다.  예제 값을 사용 하는 경우 선택할 **미국 동부** 하거나 다른 위치를 사용 합니다.
8. **대시보드에 고정**을 선택합니다.
9. **만들기**를 선택합니다.

### <a name="create-the-gateway-subnet"></a>게이트웨이 서브넷 만들기

1. 사용자가 만든 가상 네트워크 리소스를 엽니다 (**AzureVNet**) 대시보드에서.
2. 에 **설정을** 섹션에서 **서브넷**합니다.
3. 선택 **게이트웨이 서브넷** 가상 네트워크에 게이트웨이 서브넷을 추가 합니다.
4. 서브넷의 이름은 **GatewaySubnet**이라고 기본으로 설정됩니다.

   >[!IMPORTANT]
   >게이트웨이 서브넷은 특별하며 제대로 작동하려면 특정 이름이 있어야 합니다.

5. 에 **주소 범위** 필드에서 주소 확인 **10.100.1.0/24**합니다.
6. 선택 **확인** 게이트웨이 서브넷을 만들어야 합니다.

### <a name="create-the-virtual-network-gateway"></a>가상 네트워크 게이트웨이 만들기

1. Azure portal에서 선택 **+ 리소스 만들기**합니다.  
2. 로 이동 **Marketplace**를 선택한 후 **네트워킹**합니다.
3. 네트워크 리소스의 목록에서 선택 **가상 네트워크 게이트웨이**합니다.
4. **이름을**, 형식 **Azure-GW**합니다.
5. 가상 네트워크를 선택 하려면 **가상 네트워크**합니다. 선택한 **AzureVnet** 목록에서.
6. **공용 IP 주소**를 선택합니다. 경우는 **공용 IP 주소 선택** 선택 섹션 열립니다 **새로 만들기**합니다.
7. **이름을**, 형식 **Azure-GW-PiP**를 선택한 후 **확인**합니다.
8. 기본적으로 대 한 **VPN 유형**를 **route-based** 을 선택 합니다. 유지 된 **route-based** VPN 유형입니다.
9. **구독** 및 **위치**가 올바른지 확인합니다. 리소스를 대시보드에 고정할 수 있습니다. **만들기**를 선택합니다.

### <a name="create-the-local-network-gateway-resource"></a>로컬 네트워크 게이트웨이 리소스 만들기

1. Azure portal에서 선택 **+ 리소스 만들기**합니다.
2. 로 이동 **Marketplace**를 선택한 후 **네트워킹**합니다.
3. 리소스 목록에서 선택 **로컬 네트워크 게이트웨이**합니다.
4. **이름을**, 형식 **Azs-GW**합니다.
5. **IP 주소**, Azure Stack 가상 네트워크 게이트웨이에 대해 이전에 네트워크 구성 테이블에 나열 된 공용 IP 주소를 입력 합니다.
6. **주소 공간**, Azure Stack에서 입력 합니다 **10.1.0.0/24** 및 **10.1.1.0/24** 대 한 주소 공간 **AzureVNet**합니다.
7. 확인 프로그램 **구독**를 **리소스 그룹**, 및 **위치** 올바른지를 선택한 **만들기**합니다.

## <a name="create-the-connection"></a>연결 만들기

1. 사용자 포털에서 선택 **+ 리소스 만들기**합니다.
2. 로 이동 **Marketplace**를 선택한 후 **네트워킹**합니다.
3. 리소스 목록에서 선택 **연결**합니다.
4. 에 **기본** 설정 섹션에 대 한 합니다 **연결 형식**, 선택 **사이트-사이트 간 (IPSec)** 합니다.
5. 선택 합니다 **구독**를 **리소스 그룹**, 및 **위치**를 선택한 후 **확인**합니다.
6. 에 **설정을** 섹션에서 **가상 네트워크 게이트웨이**를 선택한 후 **Azure-GW**합니다.
7. 선택 **로컬 네트워크 게이트웨이**를 선택한 후 **Azs-GW**합니다.
8. **연결 이름**, 형식 **Azure Azs**합니다.
9. **공유 키 (PSK)**, 형식 **12345**을 선택한 후 **확인**합니다.

   >[!NOTE]
   >공유 키에 대 한 다른 값을 사용 하는 경우 연결의 반대쪽 끝에 생성 되는 공유 키 값과 일치 해야 해야 합니다.

10. 검토 합니다 **요약** 섹션을 선택한 후 **확인**합니다.

## <a name="create-a-virtual-machine"></a>가상 머신 만들기

이제 Azure에서 가상 머신 만들기 및 가상 네트워크의 VM 서브넷에 배치 합니다.

1. Azure portal에서 선택 **+ 리소스 만들기**합니다.
2. 로 이동 **Marketplace**를 선택한 후 **계산**합니다.
3. 가상 머신 이미지 목록에서 선택 합니다 **Windows Server 2016 Datacenter 평가판** 이미지입니다.
4. 에 **기본 사항** 섹션에 대 한 **이름**, 유형 **AzureVM**합니다.
5. 유효한 사용자 이름과 암호를 입력 합니다. 이 계정을 사용 하 여이 만든 후 가상 머신에 로그인 합니다.
6. 제공을 **구독**를 **리소스 그룹**, 및 **위치**를 선택한 후 **확인**합니다.
7. 에 **크기** 섹션을이 인스턴스에 대 한 가상 머신 크기를 선택한 후 **선택**합니다.
8. 에 **설정을** 섹션에서는 기본 설정을 사용할 수 있습니다. 선택 하기 전에 **확인**에 있는지 확인 합니다.

   * 합니다 **AzureVnet** 가상 네트워크가 선택 됩니다.
   * 로 설정 된 서브넷 **10.100.0.0/24**합니다.

   **확인**을 선택합니다.

9. 설정을 검토 합니다 **요약** 섹션을 선택한 후 **확인**합니다.

## <a name="create-the-network-resources-in-azure-stack"></a>Azure Stack에서 네트워크 리소스 만들기

다음으로 Azure Stack에서 네트워크 리소스를 만듭니다.

### <a name="sign-in-as-a-user"></a>사용자로 로그인

서비스 관리자는 테스트 계획, 제품 및 사용자에 게 사용할 수 있는 구독에 사용자로 로그인 수 있습니다. 하는 하나, 아직 없는 경우 [사용자 계정 만들기](azure-stack-add-new-user-aad.md) 로그인 하기 전에 합니다.

### <a name="create-the-virtual-network-and-a-vm-subnet"></a>가상 네트워크 및 VM 서브넷 만들기

1. 사용자 포털에 로그인 하려면 사용자 계정을 사용 합니다.
2. 사용자 포털에서 선택 **+ 리소스 만들기**합니다.

    ![새 가상 네트워크 만들기](media/azure-stack-connect-vpn/image3.png)

3. 로 이동 **Marketplace**를 선택한 후 **네트워킹**합니다.
4. **가상 네트워크**를 선택합니다.
5. 에 대 한 **이름을**를 **주소 공간**를 **서브넷 이름**, 및 **서브넷 주소 범위**, 네트워크 구성 테이블의 값을 사용 합니다.
6. **구독**, 앞서 만든 구독이 표시 됩니다.
7. 에 대 한 **리소스 그룹**, 리소스 그룹 만들기 또는 이미 있는 경우 하나를 선택할 수 있습니다 **기존 항목 사용**합니다.
8. 기본 위치를 확인합니다.
9. **대시보드에 고정**을 선택합니다.
10. **만들기**를 선택합니다.

### <a name="create-the-gateway-subnet"></a>게이트웨이 서브넷 만들기

1. 대시보드에서 만든 Azs VNet 가상 네트워크 리소스를 엽니다.
2. 에 **설정을** 섹션에서 **서브넷**합니다.
3. 가상 네트워크에 게이트웨이 서브넷을 추가 하려면 **게이트웨이 서브넷**합니다.

    ![게이트웨이 서브넷 추가](media/azure-stack-connect-vpn/image4.png)

4. 기본적으로 서브넷 이름을 설정할지 **GatewaySubnet**합니다. 제대로 작동 하려면 게이트웨이 서브넷을 사용 해야 합니다 **GatewaySubnet** 이름입니다.
5. **주소 범위**, 주소 인지 확인 **10.1.1.0/24**합니다.
6. 선택 **확인** 게이트웨이 서브넷을 만들어야 합니다.

### <a name="create-the-virtual-network-gateway"></a>가상 네트워크 게이트웨이 만들기

1. Azure Stack 포털에서 선택 **+ 리소스 만들기**합니다.
2. 로 이동 **Marketplace**를 선택한 후 **네트워킹**합니다.
3. 네트워크 리소스의 목록에서 선택 **가상 네트워크 게이트웨이**합니다.
4. **이름을**, 형식 **Azs-GW**합니다.
5. 선택 된 **가상 네트워크** 가상 네트워크를 선택 하는 항목입니다. 선택 **Azs VNet** 목록에서.
6. 선택 된 **공용 IP 주소** 메뉴 항목입니다. 경우는 **공용 IP 주소 선택** 선택 섹션 열립니다 **새로 만들기**합니다.
7. **이름을**, 형식 **Azs-GW-PiP**를 선택한 후 **확인**합니다.
8. 기본적으로 **경로 기반** 에 대해 선택 되었는지 **VPN 유형**합니다. 유지 된 **route-based** VPN 유형입니다.
9. **구독** 및 **위치**가 올바른지 확인합니다. 리소스를 대시보드에 고정할 수 있습니다. **만들기**를 선택합니다.

### <a name="create-the-local-network-gateway"></a>로컬 네트워크 게이트웨이 만들기

개념을 *로컬 네트워크 게이트웨이* Azure Stack의 Azure 배포 보다 약간 다릅니다.

Azure 배포에서 로컬 네트워크 게이트웨이 Azure에서 가상 네트워크 게이트웨이에 연결 하는 온-프레미스 (사용자 위치)에 물리적 장치를 나타냅니다. 그러나 Azure Stack에서 연결의 양쪽 끝이 가상 네트워크 게이트웨이입니다.

보다 일반적인 설명은 로컬 네트워크 게이트웨이 리소스를 연결의 다른 끝에 있는 원격 게이트웨이 항상 표시 되는지입니다.

### <a name="create-the-local-network-gateway-resource"></a>로컬 네트워크 게이트웨이 리소스 만들기

1. Azure Stack 포털에 로그인 합니다.
2. 사용자 포털에서 선택 **+ 리소스 만들기**합니다.
3. 로 이동 **Marketplace**를 선택한 후 **네트워킹**합니다.
4. 리소스 목록에서 선택 **로컬 네트워크 게이트웨이**합니다.
5. **이름을**, 형식 **Azure-GW**합니다.
6. **IP 주소**, Azure에서 가상 네트워크 게이트웨이의 공용 IP 주소를 입력 **Azure-GW-PiP**합니다. 이 주소가 이전 네트워크 구성 테이블에에서 나타납니다.
7. **주소 공간**, 사용자가 만든 Azure VNET의 주소 공간을 입력 **10.100.0.0/24** 하 고 **10.100.1.0/24**합니다.
8. 확인 프로그램 **구독**, **리소스 그룹**, 및 **위치** 값이 올바르고 선택한 **만들기**합니다.

### <a name="create-the-connection"></a>연결 만들기

1. 사용자 포털에서 선택 **+ 리소스 만들기**합니다.
2. 로 이동 **Marketplace**를 선택한 후 **네트워킹**합니다.
3. 리소스 목록에서 선택 **연결**합니다.
4. 에 **기본 사항** 설정 섹션에 대 한 합니다 **연결 형식**를 선택 **사이트-사이트 간 (IPSec)** 합니다.
5. 선택 합니다 **구독**를 **리소스 그룹**, 및 **위치**를 선택한 후 **확인**합니다.
6. 에 **설정을** 섹션에서 **가상 네트워크 게이트웨이**를 선택한 후 **Azs-GW**합니다.
7. 선택 **로컬 네트워크 게이트웨이**를 선택한 후 **Azure-GW**합니다.
8. **연결 이름**, 형식 **Azs Azure**합니다.
9. **공유 키 (PSK)**, 형식 **12345**를 선택한 후 **확인**합니다.
10. 에 **요약** 섹션에서 **확인**합니다.

### <a name="create-a-virtual-machine-vm"></a>가상 머신 (VM) 만들기

VPN 연결을 확인 하려면 두 개의 Vm 만들기: Azure에서 사용 하 고 Azure Stack의 하나입니다. 이러한 Vm을 만든 후에 VPN 터널을 통해 데이터를 받고 보내는 데 사용할 수 있습니다.

1. Azure portal에서 선택 **+ 리소스 만들기**합니다.
2. 로 이동 **Marketplace**를 선택한 후 **계산**합니다.
3. 가상 머신 이미지 목록에서 선택 합니다 **Windows Server 2016 Datacenter 평가판** 이미지입니다.
4. 에 **기본 사항을** 섹션 **이름**, 형식 **Azs VM**합니다.
5. 유효한 사용자 이름과 암호를 입력 합니다. 이 계정을 사용 하 여이 만들어진 후 VM에 로그인 합니다.
6. 제공을 **구독**를 **리소스 그룹**, 및 **위치**를 선택한 후 **확인**합니다.
7. 에 **크기** 섹션을 선택이 인스턴스에 대 한 가상 머신 크기를 선택한 후 **선택**합니다.
8. 에 **설정을** 섹션에서 기본값을 사용 합니다. 있는지 확인 합니다 **Azs VNet** 가상 네트워크가 선택 됩니다. 서브넷으로 설정 되어 있는지 확인 하십시오 **10.1.0.0/24**합니다. 그런 다음 **확인**을 선택합니다.
9. 에 **요약** 섹션에서 설정을 검토 하 고 선택한 * 확인 * * 합니다.

## <a name="test-the-connection"></a>연결 테스트

사이트 간 연결이 설정 되 면 양방향으로 흐르는 데이터를 가져올 수 있는지를 확인 해야 합니다. 연결을 테스트 하는 가장 쉬운 방법은 ping 테스트를 수행 하는 것:

* Azure Stack 및 Azure에서 가상 머신을 ping에서 만든 가상 머신에 로그인 합니다.
* Azure에서 만든 가상 머신에 로그인 하 고 Azure Stack에서 가상 컴퓨터를 ping 합니다.

>[!NOTE]
>사이트 간 연결을 통해 트래픽을 전송 했는지 확인 하려면 VIP가 아닌 원격 서브넷에서 가상 컴퓨터의 직접 IP (DIP) 주소를 ping 합니다.

### <a name="sign-in-to-the-user-vm-in-azure-stack"></a>사용자 Azure Stack에서 VM에 로그인

1. Azure Stack 포털에 로그인 합니다.
2. 왼쪽된 탐색 모음에서 선택 **가상 머신**합니다.
3. Vm 목록에서 찾을 **Azs VM** 는 이전에 만든 하 고 선택 합니다.
4. 가상 컴퓨터에 대 한 섹션에서 선택 **Connect**, Azs VM.rdp 파일을 엽니다.

     ![연결 단추](media/azure-stack-connect-vpn/image17.png)

5. 가상 머신을 만들 때 구성 된 계정으로 로그인 합니다.
6. 관리자 권한 Windows PowerShell 프롬프트를 엽니다.
7. **ipconfig /all**을 입력합니다.
8. 출력에서 찾을 합니다 **IPv4 주소**, 한 다음 나중에 사용할 주소를 저장 합니다. Azure에서 ping 하는 주소입니다. 주소는 예제 환경의 **10.1.0.4**, 하지만 환경에서 다를 수 있습니다. 내에 포함 해야 합니다 **10.1.0.0/24** 이전에 만든 서브넷입니다.
9. 가상 머신을 ping에 응답할 수 있도록 방화벽 규칙을 만들려면 다음 PowerShell 명령을 실행 합니다.

   ```powershell
   New-NetFirewallRule `
    –DisplayName “Allow ICMPv4-In” `
    –Protocol ICMPv4
   ```

### <a name="sign-in-to-the-tenant-vm-in-azure"></a>테 넌 트 Azure에서 VM에 로그인

1. Azure 포털에 로그인합니다.
2. 왼쪽된 탐색 모음에서 선택 **가상 머신**합니다.
3. 가상 컴퓨터의 목록에서 찾을 **Azure VM** 는 이전에 만든 하 고 선택 합니다.
4. 가상 컴퓨터에 대 한 섹션에서 선택 **Connect**합니다.
5. 가상 머신을 만들 때 구성 된 계정으로 로그인 합니다.
6. 관리자 권한 엽니다 **Windows PowerShell** 창입니다.
7. **ipconfig /all**을 입력합니다.
8. 내에 포함 되는 IPv4 주소가 표시 됩니다 **10.100.0.0/24**합니다. 주소는 예제 환경의 **10.100.0.4**, 하지만 주소는 다를 수 있습니다.
9. 가상 머신을 ping에 응답할 수 있도록 방화벽 규칙을 만들려면 다음 PowerShell 명령을 실행 합니다.

   ```powershell
   New-NetFirewallRule `
    –DisplayName “Allow ICMPv4-In” `
    –Protocol ICMPv4
   ```

10. Azure에서 가상 컴퓨터에서 터널을 통해 Azure Stack에서 가상 컴퓨터를 ping 합니다. 이 위해 Azs VM에서 기록해둔 DIP를 ping 합니다. 예제 환경에서 이것이 **10.1.0.4**, 랩에서 기록해둔 주소를 ping 해야 합니다. 다음 화면 캡처 같은 결과가 표시 됩니다.

    ![성공적인 ping](media/azure-stack-connect-vpn/image19b.png)

11. 원격 가상 컴퓨터의 응답을 테스트에 실패를 나타냅니다. 가상 컴퓨터 창을 닫을 수 있습니다.

테스트 하는 보다 엄격한 데이터 전송을 수행 해야 예를 들어 다른 복사 파일을 양방향으로 크기 조정 합니다.

### <a name="viewing-data-transfer-statistics-through-the-gateway-connection"></a>게이트웨이 연결을 통한 데이터 전송 통계 보기

이 정보는에서 사용할 수 있는 사이트 간 연결을 통해 데이터의 양을 전달 알고 싶다면 합니다 **연결** 섹션입니다. 이 테스트 방금 보낸 ping이 VPN 연결을 통해 발생 실제로 확인 하는 다른 방법 이기도 합니다.

1. Azure Stack 사용자 가상 머신에 로그인 하는 동안 사용자 포털에 로그인 할 사용자 계정을 사용 합니다.
2. 로 이동 **모든 리소스**를 선택한 후 합니다 **Azs Azure** 연결 합니다. **연결** 나타납니다.
3. 에 **연결** 섹션에 대 한 통계 **의 데이터를** 및 **데이터** 표시 합니다. 다음 화면 캡처에는 많은 추가 파일 전송에 소요 되 합니다. 일부 0이 아닌 값이 표시 됩니다.

    ![들어오는/나가는 데이터](media/azure-stack-connect-vpn/Connection.png)

## <a name="next-steps"></a>다음 단계

[Azure 및 Azure에 앱 배포 스택](azure-stack-solution-pipeline.md)
