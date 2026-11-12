# OSPF·EIGRP와 경로 재분배

## 서로 다른 라우팅 도메인 연결

본사는 OSPF Area 0, Topology 1 지사는 EIGRP 100·200으로 구성했습니다. Topology 2는 지사 EIGRP 100과 본사 OSPF를 연결합니다. 이는 서로 다른 프로토콜을 운영하는 망의 연동 실습이며, 모든 신규 구축에서 두 프로토콜이 필요한 것은 아닙니다.

## OSPF — Topology 1 Core1

```ios
router ospf 1
 router-id 1.1.1.1
 auto-cost reference-bandwidth 10000
 passive-interface default
 no passive-interface gi0/0
 no passive-interface gi0/1
 no passive-interface vlan99
 network 10.10.10.0 0.0.0.255 area 0
 network 172.168.10.4 0.0.0.3 area 0
interface gi0/0
 ip ospf network point-to-point
 ip ospf hello-interval 5
 ip ospf dead-interval 15
```

- router-id는 OSPF 장비 식별값입니다.
- reference-bandwidth는 링크 Cost 산정 기준입니다. 참여 장비 간 기준을 맞춥니다.
- passive-interface는 해당 인터페이스의 이웃 형성을 제한합니다. 네트워크 광고 자체를 모두 중단한다는 의미가 아닙니다.
- network 명령은 일치하는 로컬 인터페이스에서 OSPF를 활성화합니다. 임의의 원격 경로를 만들어 광고하는 명령이 아닙니다.
- point-to-point 네트워크 유형과 Hello/Dead 값은 양쪽 연결 설정과 함께 확인합니다.

## EIGRP — Branch3 발췌

```ios
router eigrp 100
 no auto-summary
 passive-interface default
 no passive-interface Tunnel1
 network 172.16.10.0 0.0.0.255
 network 172.10.30.0 0.0.0.255
```

AS100에서 Tunnel1로 이웃을 형성하고 지사 LAN 경로를 전달하려는 설정입니다. Branch4는 AS200입니다. 실제 라우팅 테이블에서는 내부 D 경로와 재분배된 D EX 경로를 구분합니다.

## 재분배·태깅 — Topology 2 Edge 발췌

```ios
route-map EIGRP-TO-OSPF deny 10
 match tag 200
route-map EIGRP-TO-OSPF permit 20
 set tag 100
route-map OSPF-TO-EIGRP deny 10
 match tag 100
route-map OSPF-TO-EIGRP permit 20
 set tag 200
router ospf 1
 redistribute eigrp 100 subnets route-map EIGRP-TO-OSPF
router eigrp 100
 redistribute ospf 1 metric 10000 100 255 1 1500 route-map OSPF-TO-EIGRP
 distance eigrp 90 109
```

| 설정 | 의미 |
| --- | --- |
| match tag 200 / deny | OSPF에서 왔던 경로가 EIGRP를 거쳐 돌아오는 재유입 제한 |
| set tag 100 | EIGRP에서 OSPF로 보낸 경로에 출처 표식 부여 |
| 반대 방향 tag 100 차단 | EIGRP 출신 경로가 OSPF를 거쳐 되돌아오는 것 제한 |
| metric의 5개 값 | Bandwidth, Delay, Reliability, Load, MTU |
| distance eigrp 90 109 | 내부 EIGRP AD 90, 외부 EIGRP AD 109로 설정 |

AD는 동일 목적지 프리픽스를 서로 다른 출처에서 배울 때의 선택에 영향을 줍니다. 이 설정은 OSPF AD110보다 외부 EIGRP를 선호하려는 의도입니다. 모든 경로의 최적성이나 루프 방지를 보장하는 설정은 아닙니다. 루프 방지는 태그·필터·토폴로지를 함께 검토합니다.

Topology 1의 태그값과 Route Map은 Topology 2와 다릅니다. 발표 표에 있는 OSPF tag1 부여·차단이 Topology 1 보관본의 모든 방향에 구현돼 있지는 않으므로, 표와 원본을 섞어 완성 설정이라고 설명하지 않습니다.

## 확인 명령

```ios
show ip ospf neighbor
show ip eigrp neighbors
show ip protocols
show ip route
show route-map
show ip eigrp topology
```

목적지 프리픽스, 다음 홉, 출구 인터페이스, AD/Metric, 외부 경로 태그를 확인합니다. 재분배 설정이 존재하는 것과 실제 필요한 경로가 학습되는 것은 별도로 확인합니다.
