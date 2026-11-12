# 출처와 작성 기준

## 보관한 자료

- 본인이 담당한 GNS3 원본 토폴로지
- [라우터 설정](../configs/topology2/Router-Config.txt)
- [스위치 설정](../configs/topology2/SW-Config.txt)
- [터널 주소 메모](../configs/topology2/tunnel%20ip.txt)
- 기존 발표 자료 중 해당 GRE over IPsec 구성과 터널 전환 시험 부분

2026-11-12에 담당 범위를 기준으로 문서를 다시 정리했습니다. 원본 구성도는 보존하고 별도의 논리 구성도를 추가했습니다. 설명에 사용한 설정 발췌는 위 파일에 연결되며, 장비를 새로 실행한 결과는 포함하지 않습니다.

`source-manifest.json`에는 남긴 설정 3개의 원본 상대 경로와 해시를 기록했습니다. 인증값은 대체했고, IOS 바이너리·접속 정보·원본 발표 파일은 업로드하지 않았습니다.

## 참고 문서

- [Cisco HSRP Track·Preempt](https://www.cisco.com/c/en/us/support/docs/ip/hot-standby-router-protocol-hsrp/13780-6.html)
- [Cisco HSRP FAQ](https://www.cisco.com/c/en/us/support/docs/ip/hot-standby-router-protocol-hsrp/9281-3.html)
- [Cisco VTP v3](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst6500/ios/15-4SY/config_guide/sup6T/15_3_sy_swcg_6T/vtp.html)

공식 문서와 실습 IOS의 플랫폼·버전은 다를 수 있습니다. 명령 지원과 실제 동작은 실습 장비에서 확인합니다.

## 아이콘

논리 구성도의 장비 아이콘은 Lucide 아이콘을 사용했습니다. [ISC 라이선스](../assets/icons/LICENSE-lucide.txt)를 함께 제공합니다.
