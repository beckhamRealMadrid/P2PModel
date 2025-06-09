# 🌐 P2PModel – 실시간 P2P 네트워크 & MMO Shard 구조 실험

본 리포지토리는 NAT 환경 하에서도 안정적인 **P2P 통신 구조**와, 대규모 MMO 게임 서버에서 요구되는 **Shard 기반의 수평 확장 구조**를 결합한 하이브리드 아키텍처를 실험 및 검토하기 위한 프로젝트입니다.

이 구조는 다음과 같은 문제 해결을 목표로 합니다:

- 서버 중심 구조의 **트래픽 집중/비용 부담 해소**
- **클라이언트 간 직접 통신**을 통한 지연 시간 단축
- **Shard 단위 격리 및 확장성 확보**를 통한 운영 유연성 강화

---

## 📐 아키텍처 개요

### 🔷 P2P 통신 구조

![P2P Model](./P2PModel.bmp)

- **Mesh 구조** 기반: 클라이언트 간 직접 연결
- 중앙 서버 없이도 실시간 통신 가능
- 트래픽이 한 지점에 몰리지 않도록 분산 처리

> 각 클라이언트는 독립적인 Peer로 동작하며, 복수 대상과 직접 연결을 구성합니다.  
> 이는 단순 중계 모델 대비 **지연 시간**과 **서버 부하**를 효과적으로 줄이는 구조입니다.

---

## 🔄 NAT Traversal: UDP Hole Punching

### 🔹 기본 연결 절차

![UDP Hole Punching](./UDP_Hole_Punching.bmp)

- 클라이언트는 중계 서버로부터 상대방의 외부 IP/포트를 수신
- **양방향으로 UDP 패킷 전송** → NAT에 외부 포트를 강제로 개방

### 🔹 연결 이후 상태

![UDP Hole Punching Success](./UDP_Hole_Punching_1.bmp)

- 성공 시 **직접 연결 전환**, 이후 서버 개입 없음
- 실시간 게임에 적합한 저지연 통신 경로 확보

---

## 🛰 Fallback: UDP Relay 서버

![UDP Relay Server](./UDPRelayServer.bmp)

- NAT 제약(예: Symmetric NAT, 방화벽)으로 P2P 실패 시 **Relay 서버 자동 사용**
- 서버는 UDP 패킷만 포워딩 → 지연 최소화 유지
- **Fallback 경로 제공을 통해 연결 성공률 극대화**

---

## 🗃 MMO Shard 기반 구조

![Shard Server Structure](./Shard.bmp)

- **Agent → Lobby → Game** 구조로 구성된 단위 Shard
- 외부 연결은 Agent Server가 수신 후 내부 Shard로 라우팅
- 각 Shard는 내부망에 위치하여 보안 및 자원 격리 확보
- Shard 단위로 확장, 장애시 독립 격리 운용 가능
- UDP Hole Punching 기반 P2P 통신을 통해 서버 자원 소모 감소

---

## 🧪 테스트 시나리오 정리

| 시나리오 ID | 조건 | NAT 유형 | 예상 동작 | 실제 처리 흐름 |
|-------------|------|----------|------------|----------------|
| TC-01 | A ↔ B | Full Cone ↔ Full Cone | 직접 연결 | Hole Punching 성공 |
| TC-02 | C ↔ D | Restricted ↔ Port Restricted | 직접 연결 | 양방향 시도 후 연결 성립 |
| TC-03 | E ↔ F | Symmetric ↔ Symmetric | 연결 실패 | Relay 서버 경유 |
| TC-04 | G ↔ H | Full Cone ↔ 방화벽 차단 | 연결 실패 | Fallback 전환 |
| TC-05 | I 재접속 | NAT 타임아웃 | 세션 종료 | KeepAlive 미작동 |
| TC-06 | 모바일 ↔ Wi-Fi | 4G ↔ 공유기 | 간헐 연결 | 재시도 및 세션 유지 |
| TC-07 | 랜덤 포트 매핑 | Symmetric NAT | 연결 실패 | Relay fallback 사용 |

---

## ⚙️ 기술 선택 배경

- 기존 MMO 구조는 **서버 비용 및 성능 병목** 이슈 존재
- P2P 우선 → 실패 시 Relay 전환 방식으로 **연결 유연성 확보**
- NAT 환경 대응을 위한 UDP Hole Punching 설계 반영
- Shard 구조와 결합하여 **확장성/격리성/보안성 동시 확보**

---

## 🧠 리뷰 및 적용 포인트

- UDP Hole Punching은 **연결 성공률이 NAT 조건에 따라 달라짐** → 시나리오 기반 시뮬레이션 필수
- Relay 경유 구조는 **지연 관리 및 보안 정책 설계 필요**
- Shard 구조는 **유저 수 급증/장애 분리 요구에 강력히 대응 가능**
- 실서비스 적용 시 **KeepAlive 정책, 세션 추적, 장애 감지 로직**까지 포함 필요

---

## 📌 참고 사항

- 본 프로젝트는 실무 MMO 서버 환경에서의 **네트워크 구조 실험 및 구조 리뷰 목적**으로 작성되었습니다.
- 각 구성 요소는 독립적으로 테스트 가능하며, 실제 연동 구조는 이후 모듈 단위로 분리될 예정입니다.

---

## 📬 Contact

구조 리뷰 또는 기술 논의는 [Issues](https://github.com/beckhamRealMadrid/P2PModel/issues)를 통해 자유롭게 남겨주세요.
