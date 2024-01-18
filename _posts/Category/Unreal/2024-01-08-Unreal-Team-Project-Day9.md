---
layout: single
title: "Unreal Team Project - Day 9"
categories: Unreal
classes: wide
---

# Day 9 : Hit Launch Replication

## 2024.01.08 Monday Scrum Log

### Daily Review

- MH → Shop Component 추가, Shop Component에서 모든 Skill Data관리

- IS → 클라이언트 색깔 변경 성공, Replication 실패

- SW → GameMode 및 GameState 작업

- HJ → 버프를 관리하는 Buff Component 추가

- 작성자 → Fireball Replication 성공


### Works to do

- MH → Shop Component Drag & Drop

- IS → 캐릭터 색깔 바꾸기 Replication 착수

- SW → GameMode가 클라이언트 접속 전에 작동안하게 적용

- HJ → 버프 리플리케이션 및 아이콘

- 작성자 → 피격 시 Launch 및 Replicate 되게


## Daily 작업 과정

**Character Launch**

Character Launch하면서 깨달은 것 → Launch값이 모두 0이어도 Launch 판정이 적용되기에 Launch값의 합이 0이 아닐때만 적용해야 한다.

*첫 트라이*

어차피 Fireball은 Replicate 되고 캐릭터도 Replicate되니, Launch를 적용하는 것은 Fireball이니 Launch 호출은 클라이언트에서 하면 결국 모든 클라이언트에서 Launch되지 않을까?

→ Launch Replicate 되지 않음. Fireball을 시전한 클라이언트에서는 Launch가 되지만, 다른 클라이언트에서는 Launch되지 않고 움직이면 원래 자리로 돌아옴.

Launch는 피격 시 Server 및 NetMulticast를 통해 Replicate해야 원활히 작동.

추가적으로 FHitdata만 보냈던 기존 Hit함수에서 Attacker를 보내어 Launch 방향도 조절.

```cpp
//PushCharacter.cpp의 Hit 및 Launch Server and NetMulticast
void APushCharacter::Hit(AActor* InAttacker, const FHitData& InHitData)
{
    FVector direction = GetActorLocation() - InAttacker->GetActorLocation();
    direction = direction.GetSafeNormal();

    FVector launch = FVector(InHitData.xLaunchPower * direction.X, InHitData.xLaunchPower * direction.Y, InHitData.zLaunchPower);

    CLog::Log(InAttacker->GetActorLocation());
    CLog::Log(launch);

    if(ResourceComponent != nullptr)
    {
		ResourceComponent->AdjustHP_Server(-InHitData.Damage);
    }

    if(launch.X + launch.Y + launch.Z > 0.0f)
    {
        LaunchServer(launch);
    }

    if(InHitData.Effect != nullptr)
    {
        UParticleSystem* particle = Cast<UParticleSystem>(InHitData.Effect);
        UNiagaraSystem* niagara = Cast<UNiagaraSystem>(InHitData.Effect);

        if(particle != nullptr)
        {
            UGameplayStatics::SpawnEmitterAtLocation(GetWorld(), particle, InHitData.Location, FRotator::ZeroRotator, InHitData.EffectScale);
        }

        if(niagara != nullptr)
        {
            UNiagaraFunctionLibrary::SpawnSystemAtLocation(GetWorld(), niagara, InHitData.Location, FRotator::ZeroRotator, InHitData.EffectScale);
        }
    }

    if(InHitData.Sound != nullptr)
    {
        UGameplayStatics::SpawnSoundAtLocation(GetWorld(), InHitData.Sound, InHitData.Location);
    }
}

//Server에게 NetMulticast를 요청하는 부분
void APushCharacter::LaunchServer_Implementation(FVector InLaunch)
{
    LaunchNMC_Implementation(InLaunch);
}

//NetMulticast를 통해 동일한 캐릭터를 모든 클라이언트에서 Launch 하는 부분
void APushCharacter::LaunchNMC_Implementation(FVector InLaunch)
{
    LaunchCharacter(InLaunch, false, false);
}
```
