# 명령어와 점검 순서

명령어는 Cisco IOS 계열 기준입니다. 장비·이미지에 따라 지원 문법과 인터페이스 이름이 다를 수 있습니다. Topology 1/2를 먼저 선택한 후 적용 대상을 확인합니다. `configs/`는 비밀값을 대체한 원본 작업 메모이며 자동 설치 스크립트가 아닙니다.

## 1. 현재 환경 파악

```ios
show version
show running-config
show ip interface brief
show interfaces description
```

장비명·IOS·연결 포트와 실제 IP를 먼저 확인합니다. 이전 PPT의 이름과 현재 장비가 같다고 가정하지 않습니다.

## 2. L2 기반 확인

```ios
show vtp status
show vlan brief
show interfaces trunk
show etherchannel summary
show spanning-tree vlan 10
```

VTP 도메인·버전, VLAN 존재 여부, Trunk 허용 목록과 실제 활성 VLAN, Po 멤버 참여 상태를 봅니다. Port-channel은 Trunk일 수 있어 show vlan brief의 access 포트 목록만으로 VLAN 소속을 판정하지 않습니다.

## 3. 게이트웨이·라우팅 확인

```ios
show standby brief
show standby vlan 10
show track 10
show ip ospf neighbor
show ip eigrp neighbors
show ip route
show ip protocols
show route-map
```

`show standby vlan 10` 지원 여부는 IOS에서 확인합니다. 필요하면 `show standby` 전체 출력으로 대체합니다. VIP·Active·Standby·현재 우선순위·Preempt와 목적지 다음 홉을 확인합니다.

## 4. VPN·NAT 확인

```ios
show dmvpn
show ip nhrp
show crypto isakmp sa
show crypto ipsec sa
show ip nat translations
show ip nat statistics
```

WAN 도달 → NHRP 등록 → 라우팅 이웃 → SA → 양방향 암복호화 순서로 구분합니다. 허브까지 Ping이 되는 것만으로 지사 간 직접 전달을 증명하지 않습니다.

## 5. Tunnel21 장애 시험 예시 — Topology 2 Edge1

아래는 원본 발표 시나리오를 다시 확인하기 위한 수동 절차입니다. 현재 실행한 결과가 아닙니다. 대상이 격리된 실습 장비인지 확인하고 정상 상태를 저장한 후 수행합니다.

```ios
enable
show ip route
show ip eigrp neighbors
configure terminal
interface tunnel 21
 shutdown
end
show ip eigrp neighbors
show ip route
```

지사에서 본사 목적지로 Ping·Traceroute를 수행해 백업 경로를 관찰합니다. 이웃이 기존부터 있었는지, 새로 형성됐는지 출력으로 구분합니다.

```ios
configure terminal
interface tunnel 21
 no shutdown
end
show ip eigrp neighbors
show ip route
```

복구 후 다시 통신을 확인합니다. `copy running-config startup-config`는 정상 상태와 보완 내용을 확인한 뒤 별도 수행합니다. 장애를 만든 상태를 의도 없이 저장하지 않습니다.

## 6. 문제별 확인 방향

| 증상 | 먼저 볼 것 | 다음으로 볼 것 |
| --- | --- | --- |
| 같은 VLAN도 불통 | 포트 Up, access VLAN, 단말 주소 | Trunk·STP·Po |
| 다른 VLAN만 불통 | SVI와 VIP, 단말 게이트웨이 | ip routing·라우팅·ACL |
| 코어는 살아 있는데 외부 불통 | Track과 업링크 상태 | HSRP Preempt·OSPF 기본 경로 |
| 허브 등록 실패 | WAN 경로·NHS 매핑 | NHRP 인증·Tunnel·IPsec |
| VPN만 불통 | 지사 경로·SA 카운터 | NAT 제외·되돌아오는 경로·MTU |
| 큰 패킷만 문제 | 경로 MTU·단편화 | GRE/IPsec 부가 헤더·MSS |

## 설정 읽기 순서

Topology 1은 HQ-CORE-1 → HQ-CORE-2 → HQ-ACCESS → HQ-EDGE → BRANCH → R-ISP 순서로 읽습니다. Topology 2는 SW-Config와 Router-Config에 여러 장비가 합쳐져 있으므로 hostname 또는 장비 제목 기준으로 분리해 봅니다. 상세 대조는 [원본 출처](10-sources.md)를 참고합니다.
