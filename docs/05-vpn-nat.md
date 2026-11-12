# GRE over IPsec과 인터넷 통신

기준 파일: [Router-Config.txt](../configs/topology2/Router-Config.txt)

## 연결 방식

지사마다 본사 두 엣지에 point-to-point GRE 터널을 구성하고 IPsec 프로파일을 적용했습니다. 지사 1은 Tunnel11·12, 지사 2는 Tunnel21·22를 사용합니다.

```ios
! Edge1의 지사 2 연결 발췌
interface tunnel 21
 ip address 192.168.20.2 255.255.255.252
 tunnel source gi 0/0
 tunnel destination 203.0.113.13
 delay 1000
 tunnel protection ipsec profile IPSEC_BR
```

원본에는 IKE 사전 공유 키 인증, IPsec transform-set과 profile이 함께 기록돼 있습니다. 공개본의 PSK는 `REPLACE_SECRET_n`으로 대체했으며 실행용 키가 아닙니다. 암호 알고리즘·인증 정책은 과거 실습 값을 보존한 것으로, 신규 운영 환경의 권장 설정으로 제시하지 않습니다.

## 외부망과 터널을 나눠 확인

터널의 목적지 WAN 주소까지 먼저 도달할 수 있어야 합니다. 이후 터널의 Up 상태, EIGRP 이웃, IPsec SA와 암복호화 카운터, 실제 LAN 간 통신을 차례로 확인합니다.

```ios
show ip interface brief
show interface tunnel 21
show crypto isakmp sa
show crypto ipsec sa
show ip eigrp neighbors
show ip route
```

## NAT/PAT

본사 엣지는 내부 인터페이스를 `ip nat inside`, WAN을 `ip nat outside`로 지정했습니다. `NAT_ACL`에서 내부·지사 대역 간 통신을 변환 대상에서 제외하고, 인터넷 방향 통신은 overload로 처리하려는 설정입니다.

```ios
ip nat pool EDGE1_NAT 203.0.113.5 203.0.113.5 netmask 255.255.255.0
ip nat inside source list NAT_ACL pool EDGE1_NAT overload
```

위 두 줄은 원본 발췌이며 전체 ACL을 대체하지 않습니다. WAN 인터페이스의 /30과 NAT pool의 /24가 다르게 적힌 점, 지사 표준 NAT ACL의 매칭 범위는 [추가 확인 항목](07-findings.md)에 남겼습니다. NAT ACL의 deny는 해당 NAT 규칙의 변환 대상에서 제외한다는 의미이며, 방화벽 차단과 구분합니다.

```ios
show ip nat translations
show ip nat statistics
show access-lists
```
