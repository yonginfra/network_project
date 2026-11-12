# 명령어와 점검 순서

실습 IOS의 지원 문법과 실제 인터페이스 이름을 확인한 뒤 사용합니다. 설정 보관본은 여러 장비가 합쳐진 작업 메모이므로 hostname 또는 장비 제목을 기준으로 나눠 읽습니다.

## 1. 장비·포트 확인

```ios
show version
show ip interface brief
show interfaces status
show cdp neighbors
```

## 2. VLAN·링크·게이트웨이

```ios
show vlan brief
show interfaces trunk
show etherchannel summary
show spanning-tree vlan 10
show spanning-tree vlan 30
show standby brief
show track 10
```

## 3. 라우팅과 VPN

```ios
show ip ospf neighbor
show ip eigrp neighbors
show ip route
show route-map
show interface tunnel 21
show interface tunnel 22
show crypto isakmp sa
show crypto ipsec sa
```

## 4. Tunnel21 중단·복귀 시험 예시

먼저 Edge1·Edge2·BRANCH2의 정상 이웃·다음 홉과 연속 Ping을 기록합니다. 다른 설정은 그대로 두고 본사 Edge1의 Tunnel21만 중단합니다.

```ios
configure terminal
interface tunnel 21
 shutdown
end
```

BRANCH2에서 Tunnel22 이웃과 실제 경로·통신을 확인합니다. 시험 후에는 원래 인터페이스를 복구하고 경로 복귀를 기록합니다.

```ios
configure terminal
interface tunnel 21
 no shutdown
end
```

이 명령은 시험 절차 예시이며 이번 문서 수정에서 실행하지 않았습니다.

## 5. 인터넷 통신과 NAT

```ios
show access-lists
show ip nat statistics
show ip nat translations
show ip route 0.0.0.0
```

WAN 도달성, VPN 상태, 라우팅, NAT 변환을 나눠 확인하면 장애 범위를 좁히기 쉽습니다.
