# 본사 내부망과 이중화

## 기술을 사용한 이유

| 선택 | 이번 구성에서의 역할 | 다른 방식과 비교 |
| --- | --- | --- |
| VLAN | Data·Voice·Server·Guest·Mgmt 브로드캐스트 영역 분리 | 하나의 VLAN보다 용도별 관리가 쉬움. 접근 제한은 별도 정책 필요 |
| VTP v3 | VLAN DB 변경 주체와 전파 관리 | 자동 게이트웨이 장애 전환 기능은 아님 |
| LACP | 코어 간 여러 링크를 Po1로 묶음 | 표준 협상 방식. PAgP는 Cisco 방식, static on은 협상 없음 |
| Rapid PVST+ | VLAN별 Root 설정과 빠른 수렴 | VLAN이 적은 실습에 직관적. VLAN이 많으면 MST 인스턴스 통합 검토 |
| HSRP | VLAN별 가상 게이트웨이 유지 | Cisco 환경에서 역할 제어. 다중 제조사 환경은 VRRP 검토 |
| VLAN별 Active 분산 | 두 코어에 VLAN별 주 역할 배분 | GLBP처럼 같은 가상 IP 안의 단말을 여러 전달자로 분산한 구성은 아님 |

위 비교는 현재 구성에 대한 설계 설명입니다. 당시 모든 대안을 직접 성능 비교한 결과는 아닙니다.

## VTP와 LACP — Topology 1 Core1 발췌

```ios
vtp domain HQ
vtp version 3
vtp mode server
interface range gi0/2 - 3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
interface port-channel 1
 switchport trunk allowed vlan 10,20,30,40,99
 switchport mode trunk
```

VTP 인증값은 공개용 설정에서 대체했습니다. 원본의 `vtp primary force`는 특권 EXEC 모드에서 Primary 역할을 지정하는 명령입니다. 운영자가 의도한 승격이며, Core1 장애 시 Core2가 자동 Primary가 되는 것으로 설명하지 않습니다.

LACP active는 협상을 먼저 시작합니다. 각 멤버 포트의 속도·Trunk·VLAN 조건과 상대 설정도 맞아야 합니다. 두 링크를 묶어도 하나의 흐름이 항상 두 링크의 합산 속도를 사용하는 것은 아닙니다.

## Rapid PVST+ — VLAN별 Root

```ios
! Core1
spanning-tree mode rapid-pvst
spanning-tree vlan 1,10,20,99 root primary
spanning-tree vlan 30,40 root secondary
! Core2
spanning-tree mode rapid-pvst
spanning-tree vlan 30,40 root primary
spanning-tree vlan 1,10,20,99 root secondary
```

Root를 명시해 의도하지 않은 액세스 스위치가 Root가 되는 상황을 피했습니다. STP Root와 HSRP Active 위치를 맞추는 것은 경로 정렬의 의도입니다. 액세스 포트 cost를 별도로 지정한 경우 실제 전달 경로는 show spanning-tree와 Traceroute로 함께 확인합니다.

## HSRP와 업링크 추적

```ios
! Topology 1 Core1, VLAN10 발췌
track 1 interface gi0/0 line-protocol
track 2 interface gi0/1 line-protocol
track 10 list boolean or
 object 1
 object 2
interface vlan 10
 ip address 10.10.10.2 255.255.255.0
 standby 10 ip 10.10.10.1
 standby 10 priority 110
 standby 10 timers 1 3
 standby 10 preempt
 standby 10 preempt delay minimum 60
 standby 10 track 10 decrement 20
```

| 명령 | 프로젝트에서의 의미 |
| --- | --- |
| track 1/2 interface | 각 엣지 방향 업링크의 line protocol 추적 |
| list boolean or | 하나라도 Up이면 묶음 Up. 둘 다 Down일 때 Down |
| standby 10 ip | PC가 사용하는 VLAN10 가상 게이트웨이 |
| priority 110 | 상대 100보다 높은 주 역할 우선순위 |
| decrement 20 | 추적 Down 시 110 → 90으로 낮춤 |
| timers 1 3 | Hello 1초 / Hold 3초 설정. 실제 복구시간 측정값은 아님 |
| preempt | 더 높은 우선순위로 Active 역할을 가져올 수 있게 함 |
| preempt delay minimum 60 | 최소 Preempt 지연 설정. 즉시 복귀를 가정하지 않음 |

중요: 우선순위가 낮아졌다는 사실만으로 상대가 반드시 즉시 Active가 되지는 않습니다. Topology 1 보관본의 일부 백업 그룹에는 preempt가 없습니다. 장비 전원 장애 시험과 업링크만 내리는 시험은 구분하며, 후자는 상대 Preempt 설정을 확인한 뒤 검증합니다. Topology 2에는 백업 그룹의 preempt 설정이 기록돼 있습니다.

## 단말 포트 보호

```ios
! Access1 원본: VLAN 번호가 빠져 있어 아래 줄은 원본 결함
switchport access vlan
spanning-tree bpduguard enable
spanning-tree portfast
```

VLAN10 단말 포트라면 `switchport access vlan 10`처럼 번호를 명시해야 합니다. 이는 재현 시 보완할 예시이며 이번에 적용·성공한 명령으로 기록하지 않습니다. PortFast는 단말 포트의 빠른 전달 전환, BPDU Guard는 예상하지 못한 BPDU 수신 시 포트 보호 역할입니다.

## 확인 명령

```ios
show vtp status
show vlan brief
show interfaces trunk
show etherchannel summary
show spanning-tree vlan 10
show standby brief
show track 10
```

VLAN 목록, Trunk 허용 목록, Po 멤버 상태, VLAN별 Root, HSRP Active와 유효 우선순위를 각각 따로 확인합니다.
