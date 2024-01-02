---
layout: single
title: "Unreal Engine - RPC"
categories: Unreal
classes: wide
---

## Unreal Engine - RPC (Remote Procedure Call)

Remote Procedure Call은 로컬 호출을 통해 외부 다른 머신에서 원격 호출되는 함수를 의미합니다.

멀티 플레이어 환경에서 서버와 클라이언트 사이에 리플리케이션 해야하는 요소가 존재할 때 필요합니다.


**RPC 적용**

RPC는 UFUNTION 매크로 안에 다음 3가지의 키워드 중 하나를 적용하여 실행할 수 있습니다: Server, Client, NetMulticast

서버에서 호출하며 클라이언트에서 실행하는 함수는   
> UFUNCTION(Client)

클라이언트에서 호출하며 서버에서 실행되는 함수는   
> UFUNCTION(Server)

NetMultiCast는 클라이언트에서 호출된 경우 로컬에서만 실행하지만, 서버에서 호출된 경우 모든 클라이언트에 적용되게 합니다.   
> UFUNCTION(NetMulticast)

이 때, 언리얼 4.26 기준으로 NetMulticast가 적용되는 함수는 뒤에 Reliable 또는 Unreliable을 적용하여 신뢰성 보장에 대한 내용을 적어주어야 합니다.


**조건 및 주의사항**

RPC의 작동을 위한 조건은 다음과 같습니다.

1. Actor에서 호출되어야 합니다.

2. Replicated된 Actor여야 합니다.

3. Client 문구로 서버에서 호출되어 클라이언트에서 실행되는 경우는 해당 Actor를 소유한 클라이언트에서만 실행됩니다.

4. Server 문구로 클라이언트에서 호출되고 서버에서 실행되는 경우는 클라이언트가 RPC를 호출하는 해당 Actor를 소유해야합니다.

서버와 클라이언트에서 호출되는 RPC에 따른 결과는 다음과 같습니다.

![](/assets/images/Unreal/RPC.PNG)
*출처:<https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/Networking/Actors/RPCs/>*
