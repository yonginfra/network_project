# 출처와 작성 기준

## 프로젝트 원본

- 팀 발표 자료: `미니프로젝트_송용_남기석.pptx` (43장)
- Topology 1: `Config/Topology 1/컨피그/`의 장비별 텍스트와 구성도
- Topology 2: `Router-Config.txt`, `SW-Config.txt`, `tunnel ip.txt`와 구성도
- 기간: 사용자가 올린 이력서의 2026.06 - 2026.07 표기

2026-11-12에 파일을 읽고 정리했습니다. 이번 패키지는 문서·원본 설정의 공개용 사본이며, 실제 장비를 새로 실행한 검증 결과를 포함하지 않습니다. 파이널 프로젝트는 문서의 목차와 기술 설명 방식만 참고했고 AWS 내용이나 실행 결과는 넣지 않았습니다.

## 문서와 원본 설정 연결

| 설명 | 설정 파일 |
| --- | --- |
| Core1 VTP·Po·STP·Track·HSRP | [HQ-CORE-1](../configs/topology1/HQ%20SW/HQ-CORE-1.txt) |
| Core2 역할 | [HQ-CORE-2](../configs/topology1/HQ%20SW/HQ-CORE-2.txt) |
| 단말 포트·관리 SVI | [HQ-ACCESS-1](../configs/topology1/HQ%20SW/HQ-ACCESS-1.txt) |
| DMVPN 허브·재분배·NAT | [HQ-EDGE-1](../configs/topology1/HQ%20ROUTER/HQ-EDGE-1.txt), [HQ-EDGE-2](../configs/topology1/HQ%20ROUTER/HQ-EDGE-2.txt) |
| DMVPN 지사 | [BRANCH1](../configs/topology1/BRANCH/BRANCH1.txt), [BRANCH3](../configs/topology1/BRANCH/BRANCH3.txt), [BRANCH4](../configs/topology1/BRANCH/BRANCH4.txt) |
| GRE/IPsec·AD·Route Map | [Topology 2 Router](../configs/topology2/Router-Config.txt) |
| Topology 2 Core·Access | [Topology 2 SW](../configs/topology2/SW-Config.txt) |

`source-manifest.json`에는 원본 상대 경로와 SHA-256, 공개용 사본 경로를 기록했습니다. 비밀값 자체는 목록에 남기지 않았습니다. IOS 바이너리·개인 접속정보·원본 PPT는 이 패키지에 포함하지 않았습니다.

## 기술 설명 참고

- [Cisco HSRP Track·Preempt](https://www.cisco.com/c/en/us/support/docs/ip/hot-standby-router-protocol-hsrp/13780-6.html): 우선순위 추적과 역할 선점은 함께 검토합니다.
- [Cisco HSRP FAQ](https://www.cisco.com/c/en/us/support/docs/ip/hot-standby-router-protocol-hsrp/9281-3.html): Track에 따른 Standby 인계의 Preempt 조건을 확인했습니다.
- [Cisco DMVPN 문제 해결](https://www.cisco.com/c/en/us/support/docs/security/dynamic-multipoint-vpn-dmvpn/111976-dmvpn-troubleshoot-00.html): Phase 2·3 다음 홉 설명과 원본 설정을 구분했습니다.
- [Cisco NHRP Shortcut](https://www.cisco.com/c/en/us/td/docs/routers/ios-xe/ip-addressing/ip-addressing/m_nhrp-switch-enhancemts-dmvpn-0.html): Redirect·Shortcut의 역할을 확인했습니다.
- [Cisco VTP v3](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst6500/ios/15-4SY/config_guide/sup6T/15_3_sy_swcg_6T/vtp.html): Primary 지정 명령을 DB 관리 역할로 설명했습니다.

공식 문서의 대상 플랫폼과 실습 IOS는 다를 수 있습니다. 구체적인 명령 지원 여부는 실습 장비에서 확인합니다.
