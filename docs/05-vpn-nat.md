# VPN과 인터넷 통신

## DMVPN을 사용한 이유

여러 지사를 허브에 등록하고 지사 간 연결 방식도 확인하기 위해 mGRE·NHRP·IPsec을 조합했습니다. mGRE는 하나의 터널 인터페이스에서 여러 상대를 다루고, NHRP는 터널 주소와 실제 전송망 주소의 매핑을 관리합니다. IPsec은 터널 트래픽을 보호합니다.

## Topology 1 Branch3 Tunnel1 발췌

```ios
interface Tunnel1
 ip address 172.16.10.33 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 tunnel key 200
 ip nhrp network-id 200
 ip nhrp holdtime 600
 ip nhrp authentication REPLACE_NHRP_SECRET
 ip nhrp map 172.16.10.1 203.0.113.6
 ip nhrp map 172.16.10.2 203.0.113.10
 ip nhrp map multicast 203.0.113.6
 ip nhrp map multicast 203.0.113.10
 ip nhrp nhs 172.16.10.1
 ip nhrp nhs 172.16.10.2
 ip nhrp shortcut
 tunnel protection ipsec profile DMVPN-PROFILE shared
```

인증값은 설명용 대체값입니다. 허브 .1/.2를 NHS로 지정하고, 허브의 오버레이 주소와 WAN 주소를 매핑합니다. MTU·MSS는 GRE·IPsec 부가 헤더에 따른 패킷 크기 문제를 고려한 설정입니다. 실제 적정값은 캡슐화 방식·전송망 MTU에 맞춰 확인합니다.

## Phase 설명과 원본 설정을 구분

| 구분 | 발표/보관본 | 문서에서의 해석 |
| --- | --- | --- |
| Tunnel0 | Phase 2로 명명, 허브 next-hop-self 유지 | 당시 자료의 명칭. 일반적인 Phase 2의 다음 홉 유지 조건과 차이 있음 |
| Tunnel1 | 허브 redirect, 스포크 shortcut, 허브 no next-hop-self | Redirect/Shortcut 존재는 확인. 전형적인 Phase 3 권장 구성과 차이 있음 |

Phase 2도 스포크 간 직접 터널을 지원합니다. 'Phase 2는 항상 허브 경유'라고 설명하지 않습니다. Phase 3는 허브의 Redirect와 스포크의 Shortcut으로 짧은 전달 경로를 학습합니다. 원본의 허브 next-hop-self 해제는 원본 그대로 보존했지만 Phase 3 필수 설정으로 추천하지 않습니다. 정확한 비교는 NHRP 매핑·라우팅 다음 홉·실제 전달 경로를 함께 재시험해야 합니다.

## GRE over IPsec — Topology 2

```ios
! Edge1
interface tunnel 21
 ip address 192.168.20.2 255.255.255.252
 tunnel source gi0/0
 tunnel destination 203.0.113.13
 delay 1000
 tunnel protection ipsec profile IPSEC_BR
! Edge2
interface tunnel 22
 ip address 192.168.20.6 255.255.255.252
 tunnel source gi0/0
 tunnel destination 203.0.113.13
 delay 5000
 tunnel protection ipsec profile IPSEC_BR
```

GRE는 논리 연결, IPsec은 암호화·무결성 보호 역할입니다. Delay는 EIGRP Metric에 영향을 주는 설정값이지 측정한 실제 네트워크 지연시간이 아닙니다. 낮은 Delay 경로를 선호하도록 의도했으며, 최종 선택은 전체 Metric과 경로 학습 상태에 따라 확인합니다.

Topology 1은 AES256·SHA1·DH5, Topology 2는 AES128·SHA1·DH2의 IKEv1 설정을 보관하고 있습니다. 과거 IOS 실습 기록이며 현재 운영망 암호 정책 권장값으로 제시하지 않습니다. 운영 환경에는 장비 지원과 최신 정책에 맞는 IKE·암호 조합을 별도 설계합니다.

## NAT/PAT — Topology 1 본사

```ios
ip access-list extended NAT-ACL-HQ
 deny ip 10.10.0.0 0.0.255.255 10.10.0.0 0.0.255.255
 deny ip 10.10.0.0 0.0.255.255 172.10.20.0 0.0.0.255
 deny ip 10.10.0.0 0.0.255.255 172.16.0.0 0.0.255.255
 ! 원본에는 다른 지사 대역 제외 항목도 존재함
 permit ip 10.10.0.0 0.0.255.255 any
ip nat inside source list NAT-ACL-HQ interface GigabitEthernet0/0 overload
```

위 블록은 일부 발췌로 전체 NAT ACL을 대체하지 않습니다. 내부·지사·오버레이 통신은 변환 대상에서 제외하고 인터넷 방향은 PAT 처리하려는 구성입니다. NAT를 방화벽이나 암호화 대책으로 설명하지 않습니다.

## 확인 명령

```ios
show interfaces tunnel 1
show dmvpn
show ip nhrp
show crypto isakmp sa
show crypto ipsec sa
show ip nat translations
show ip nat statistics
```

Tunnel Up과 IPsec 실제 암복호화 성공은 같은 판정이 아닙니다. 양방향 트래픽을 보내고 encaps/decaps 카운터를 함께 봅니다. 지사 컨피그는 허브 주소로 가는 경로와 일반 인터넷 기본 경로를 별도로 확인합니다.
