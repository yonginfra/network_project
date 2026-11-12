# 본사 내부망과 이중화

기준 파일: [SW-Config.txt](../configs/topology2/SW-Config.txt)

## VLAN과 링크 구성

업무별 접속망은 VLAN 10·20·30·40으로 나눴습니다. VLAN 99는 관리용이며, VTP v3와 Trunk 설정으로 VLAN 전달 범위를 관리했습니다. 코어 간 두 링크는 LACP Port-channel 12로 묶었습니다.

```ios
interface port-channel 12
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99
interface range gi0/2-3
 channel-protocol lacp
 channel-group 12 mode active
```

## Rapid PVST+와 역할 분담

| VLAN | STP Root 우선 장비 | HSRP 우선 장비 |
| --- | --- | --- |
| 10·20 | Core1 | Core1 |
| 30·40 | Core2 | Core2 |

액세스 스위치의 VLAN별 STP Cost도 조정했습니다. 실제 전달 경로는 설정 의도만으로 판단하지 않고 `show spanning-tree vlan 번호`로 확인합니다.

## HSRP와 업링크 추적

Core1의 VLAN 10·20은 우선순위 150, Core2의 대응 그룹은 100입니다. VLAN 30·40은 반대로 설정했습니다. 주 역할 장비의 해당 그룹은 Track Down 시 우선순위를 100 낮추도록 했습니다.

```ios
track 1 interface gi0/0 line-protocol
track 2 interface gi0/1 line-protocol
track 10 list boolean or
 object 1
 object 2
interface vlan 10
 ip address 10.10.10.252 255.255.255.0
 standby 10 ip 10.10.10.254
 standby 10 priority 150
 standby 10 track 10 decrement 100
```

Track 목록은 OR 조건입니다. 두 업링크 중 하나가 살아 있으면 목록은 Up이며, 둘 다 내려갔을 때 Down이 되는 조건으로 읽어야 합니다. 장비 전원 장애와 업링크 추적에 따른 전환은 다른 시험입니다.

원본에는 주 역할 그룹의 `preempt delay mini 60`과 백업 그룹의 `preempt`가 기록돼 있습니다. 복구 후 실제 선점 시점과 지원 명령 문법은 사용하는 IOS에서 확인합니다.

## 점검 명령

```ios
show vlan brief
show interfaces trunk
show etherchannel summary
show spanning-tree vlan 10
show spanning-tree vlan 30
show standby brief
show track 10
```

코어-액세스 Trunk의 VLAN 99 허용 범위 등 설정 간 차이는 [보완 사항](07-findings.md)을 참고합니다.
