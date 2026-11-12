# 본사-지사 통합 네트워크 구축 및 이중화

본사 내부망과 지사망을 연결하고, 코어·엣지·VPN 경로 장애 시 역할과 경로 변화를 확인한 팀 프로젝트입니다.

**진행 흐름:** 토폴로지 설계 → VLAN·LACP·STP → HSRP·Track → OSPF·EIGRP → VPN·NAT → 장애·복귀 확인

기술을 나열하기보다 **사용한 이유 → 실제 설정 → 확인 명령 → 기존 결과와 남은 확인점** 순서로 정리했습니다.

## 전체 구성

![Topology 1](assets/topology-1.png)

| 구분 | 내용 |
| --- | --- |
| 기간·팀 | 2026.06 - 2026.07 / 송용·남기석 |
| 본사 내부망 | VLAN 10·20·30·40·99, VTP v3, LACP, Rapid PVST+, HSRP·Track |
| 라우팅 | OSPF Area 0, EIGRP 100·200, 재분배·태깅·경로 선택 |
| Topology 1 | 지사 4곳, DMVPN 이중 허브 |
| Topology 2 | 지사 2곳, GRE over IPsec 주·백업 경로 |
| 인터넷 통신 | NAT 대상 제외와 PAT |

## 프로젝트 문서

| 문서 | 내용 |
| --- | --- |
| [프로젝트 개요](docs/01-overview.md) | 목표, 진행 흐름, 팀 작업과 개인 역할 |
| [전체 구성](docs/02-architecture.md) | 두 토폴로지, VLAN·주소·통신 흐름 |
| [본사 내부망과 이중화](docs/03-switching-ha.md) | 선택 이유, STP·HSRP·Track·LACP 명령 |
| [라우팅과 재분배](docs/04-routing.md) | OSPF·EIGRP, Tag·Route Map·AD |
| [VPN과 인터넷 통신](docs/05-vpn-nat.md) | DMVPN, GRE/IPsec, NAT 명령과 해석 |
| [장애 전환과 검증](docs/06-validation.md) | 기존 PPT의 시험 기록과 재시험 순서 |
| [원본 확인 사항](docs/07-findings.md) | 누락 명령·설정 차이·설명 보완 |
| [명령어와 점검 순서](docs/08-runbook.md) | 조회 명령, 터널 장애·복귀 절차 |
| [면접 설명](docs/09-interview.md) | 선택 이유와 검증 범위에 대한 구어체 답변 |
| [출처와 작성 기준](docs/10-sources.md) | 원본 설정 링크와 기술 참고 문서 |

## 확인한 내용

- VLAN별 Root와 HSRP Active 분담 구성.
- 기존 발표 기록의 코어 중단·복구, 엣지 우회, 터널 주·백업 전환.
- OSPF·EIGRP 경로 연동 및 태그를 이용한 재유입 제한.
- 인터페이스·라우팅·VPN·NAT를 나눠 확인하는 점검 절차.

## 읽기 전에

2026-11-12에 원본 설정과 발표 자료를 정리했습니다. **이번에 장비를 다시 실행한 것은 아닙니다.** configs는 비밀값을 대체한 보관본으로 일부 미완성 명령과 메모가 남아 있습니다. 두 토폴로지의 주소·명령을 섞어 일괄 적용하지 않습니다.

원본의 DMVPN Phase 설명과 HSRP Track·Preempt 설정에는 재확인할 내용이 있어 별도 문서에 기록했습니다. 모든 장애에서 무중단, 루프 완전 제거, 단일 장애점 전체 제거를 입증한 프로젝트로 표현하지 않습니다.

## 파일 구성

```text
README.md              # 프로젝트 홈
docs/                   # 주제별 설명·명령·검증·면접 자료
assets/                 # 두 토폴로지 구성도
configs/topology1/      # 장비별 원본 설정의 인증값 대체본
configs/topology2/      # Router·SW 통합 설정의 인증값 대체본
source-manifest.json     # 원본 상대 경로와 해시
```

저장소를 만들 경우 이 폴더의 내용물을 루트에 업로드하면 README와 상대 링크를 사용할 수 있습니다. 공개 사용을 위한 임의의 소스 라이선스는 부여하지 않았습니다.
