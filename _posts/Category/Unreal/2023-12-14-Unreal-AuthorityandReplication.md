---
layout: single
title: "Unreal Engine - Authority and Replication"
categories: Unreal
classes: wide
---

### Unreal Engine - Authority and Replication

Authority와 Replication은 언리얼에서 멀티 플레이어 게임을 제작할 때 서버와 클라이언트를 구별하고 서버가 클라이언트를 동기화 하는 기능을 의미합니다.


####Authority

언리얼에는 Role과 RemoteRole이라는 프로퍼티가 있습니다. Role은 현재 엔진 인스턴스를 실행한 기기의 Authority를 확인하는 프로퍼티이며, RemoteRole은 이 기기와 연결된 다른 기기의 Authority를 확인하는 프로퍼티입니다.

언리얼에는 총 3가지의 Role이 존재합니다: ROLE_Authority, ROLE_SimulatedProxy, ROLE_AutonomousProxy.

ROLE_Authority는 서버만이 보유할 수 있으며, 클라이언트는 ROLE_SimulatedProxy 또는 ROLE_AutonomousProxy를 보유할 수 있습니다.

어떠한 프로퍼티나 액터를 리플리케이트 하려 할 때, 이러한 Role을 검사하여 진행합니다. 클라이언트는 서버에게 리플리케이트 할 수 없으며, 오직 서버만이 리플리케이트를 통해 다양한 요소들을 동기화 할 수 있습니다.


####Client Role

서버가 리플리케이션을 실행할 때, 모든 업데이트마다 액터를 리플리케이트 한다면 많은 리소스를 사용하게 되기 때문에 AActor::NetUpdateFrequency의 빈도로 업데이트 하게 됩니다.

그렇기에 클라이언트는 그 업데이트 사이 사이 액터의 상태를 시뮬레이션을 통해 부드럽게 업데이트 하며, 이 방법을 정하는 것이 ROLE_SimulatedProxy와 ROLE_AutonomousProxy입니다.

**ROLE_SimulatedProxy**는 일반적인 시뮬레이션 방법으로 마지막으로 업데이트 된 속도에 따라 움직임을 반영하는 방법입니다.

**ROLE_AutonomousProxy**는 플레이어 컨트롤러로 인해 빙의된 액터에만 사용되며, 일반적으로 속도만이 아닌 플레이어의 입력에 따라 시뮬레이션 하는 방법을 사용합니다.


#### Property Replication

언리얼은 액터만이 아닌 프로퍼티에 대한 리플리케이션을 제공합니다.

프로퍼티 리플리케이션은 다음과 같은 과정을 통해 프로퍼티가 리플리케이트되게 할 수 있습니다.

1. 프로퍼티의 메타 정보로 Replicated를 설정합니다.

    ```cpp
    class ENGINE_API AActor : public UObject
    {
        UPROPERTY( replicated )
        AActor * Owner;
    };
    ```

2. 액터 클래스에서 GetLifeTimeReplicatedProps함수를 오버라이드하여 구현하여야 합니다.

    ```cpp
    void AActor::GetLifetimeReplicatedProps( TArray< FLifetimeProperty > & OutLifetimeProps ) const
    {
        DOREPLIFETIME( AActor, Owner );
    }
    ```

3. 마지막으로 액터의 생성자에서 bReplicates 변수를 true로 설정합니다.

    ```cpp
    AActor::AActor( const class FPostConstructInitializeProperties & PCIP ) : Super( PCIP )
    {
        bReplicates = true;
    }
    ```
