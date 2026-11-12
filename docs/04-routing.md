# 라우팅 연동과 경로 선택

기준 파일: [Router-Config.txt](../configs/topology2/Router-Config.txt), [SW-Config.txt](../configs/topology2/SW-Config.txt)

## 본사 OSPF와 지사 EIGRP

본사 코어·엣지는 OSPF Area 0, 지사 LAN과 GRE 터널은 EIGRP 100을 사용합니다. 본사 엣지에서 양방향 재분배해 서로 다른 라우팅 영역의 경로를 연결했습니다.

| 구간 | 설정 |
| --- | --- |
| Core1 / Core2 | OSPF router-id 11.11.11.11 / 22.22.22.22 |
| Edge1 / Edge2 | OSPF router-id 1.1.1.1 / 2.2.2.2 |
| 본사-지사 | EIGRP 100, 터널 인터페이스에서 이웃 형성 |
| 지사 | `maximum-paths 1` 기록 |

## 재분배와 태그 필터

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

EIGRP에서 OSPF로 전달한 경로에는 tag 100, 반대 방향에는 tag 200을 부여했습니다. 반대쪽에서 돌아오는 태그를 차단해 재유입을 제한하려는 구성입니다. 원본 주석과 실제 `set tag`가 다르게 읽히는 곳은 명령 자체를 기준으로 확인합니다.

`distance eigrp 90 109`는 내부·외부 EIGRP의 관리 거리를 지정한 값입니다. 이 값 하나만으로 모든 경로가 최적이거나 루프가 없다고 판단하지 않고, 실제 다음 홉과 태그를 함께 확인합니다.

## 주·예비 터널 경로

Edge1의 Tunnel11·21은 `delay 1000`, Edge2의 Tunnel12·22는 `delay 5000`으로 기록돼 있습니다. 경로 우선순위를 다르게 주려는 설정이며 실제 선택은 EIGRP 이웃·메트릭과 라우팅 테이블에서 확인합니다.

## 점검 명령

```ios
show ip ospf neighbor
show ip eigrp neighbors
show ip protocols
show ip route
show ip route eigrp
show route-map
traceroute 목적지주소
```
