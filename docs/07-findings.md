# 원본 확인 사항과 개선 과제

이번 문서 정리에서는 원본 텍스트와 발표 내용을 읽었습니다. 아래는 장비에서 새로 재현한 장애 보고서가 아니라, 재현 전에 확인해야 하는 차이와 보완 사항입니다.

| 확인 사항 | 자료에서 본 내용 | 보완·재확인 방향 |
| --- | --- | --- |
| Access1 VLAN 누락 | Topology 1 `switchport access vlan` 뒤 번호 없음 | 담당 VLAN 확인 후 번호 명시 |
| 실행 모드 섞임 | 일부 파일은 end/wr 이후 interface 명령 등 작업 메모 형식 | 실제 running-config가 아닌 작업 순서 메모로 읽기 |
| 설명과 주소 차이 | Edge1 Gi1/0 주석은 Core1 연결, 실제 /30 주소는 엣지 간 링크와 일치 | 실 주소·결선·인터페이스 상태로 대조 |
| HSRP Preempt | Topology 1 일부 백업 그룹에 preempt 미기재 | 업링크 추적 전환을 별도 시험. Topology 2와 혼합 금지 |
| DMVPN Phase 설명 | Tunnel0 허브 경유를 Phase 2 일반 특징처럼 기재 | Phase 2도 직접 터널 가능. 해당 설정과 관찰로 한정 |
| Phase 3 next-hop-self | Tunnel1 허브에서 no next-hop-self 기재 | 일반 권장 동작과 비교하고 NHRP Shortcut 경로 확인 |
| 재분배 태그 표 | PPT에는 OSPF tag1, Topology 1 설정은 방향별 일부 상이 | 두 엣지의 Route Map과 양방향 필터 교차 확인 |
| Tunnel21/22 주소 | PPT 표와 Topology 2 Router-Config 값 차이 | 재현 시 실행 장비 값을 기준으로 주소표 확정 |
| NAT pool mask | Topology 2 WAN /30에 pool netmask /24 기재 | 장비의 NAT pool 유효성과 필요성 검토. 임의 수정은 미실행 |
| 지사 인터넷 경로 | Topology 1 일부 지사는 203.0.113.0/24 경로만 보관 | ISP 역할 웹서버 주소·기본 경로를 실제로 대조 |
| 주소 계획 | 172.168·172.10·172.100·172.200 내부 대역 사용 | 실제 환경용 주소 계획은 RFC1918 기준 재작성 |
| VTP 표현 | Secondary를 자동 백업 전환처럼 설명 | DB 수정 권한과 관리자가 수행하는 Primary 승격으로 설명 |

## 보존한 것과 바꾼 것

- 보존: 원본 IP, 인터페이스, 프로토콜 값, 불완전한 명령, 중복 명령과 주석.
- 변경: 개행 정규화, 빈 줄 축약, 비밀번호·PSK·NHRP 인증 문자열 대체.
- 별도 설명: 일반 기술 설명에서 틀린 표현과 재현 전 확인점.
- 미수행: 장비 실행, config 적용, 자동 복구 시험, 실제 성능 측정.

따라서 configs는 검토용 보관본입니다. 파일을 통째로 붙여넣어 완성된 실습 환경이 생긴다고 가정하지 않습니다.

## 다음에 보완할 내용

1. 현재 주소표와 장비 결선을 고정하고 전체 running-config 재수집.
2. Track·Preempt 및 DMVPN Phase 시나리오를 분리해 재시험.
3. 원시 출력·시간·Ping 손실 기록을 같은 실행 차수로 보관.
4. 현행 보안 기준에 맞는 관리 접속·인증·암호 정책 설계.
