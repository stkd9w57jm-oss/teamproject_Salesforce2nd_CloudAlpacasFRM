# Cloud Alpacas FRM — 프로 야구단 Fan 360 CRM 구축

> Salesforce AI CRM 2nd Track 팀 프로젝트 · 2026
> 가상의 프로야구단 **Cloud Alpacas**를 위한 **팬 관계 관리(FRM, Fan Relationship Management)** 시스템을 Salesforce 위에 구축한다.

---

## 무엇을 만드는가

구단 마케팅 담당자(김매니저)는 팬 정보가 티켓 예매 시스템, 굿즈샵, SNS에 흩어져 있어 **"이 팬이 누구인지"를 한 화면에서 볼 수 없다.** VIP가 될 가능성이 높은 팬도 엑셀을 정리하고 나서야 뒤늦게 발견한다.

이 프로젝트는 그 문제를 **Customer 360**으로 푼다. 한 명의 팬이 SNS에서 구단을 처음 알게 된 순간부터 티켓 구매 → 첫 직관 → 굿즈 구매 → 재방문 → 멤버십 가입까지, **흩어진 접점을 하나의 타임라인으로 연결하고 자동화가 그 위에서 동작하게 만든다.**

핵심 시연 장면은 이것이다. 한 팬의 재방문 횟수와 누적 지출이 임계값을 넘는 순간, Flow가 자동으로 VIP 후보를 감지해 **김매니저의 Slack으로 알림을 보낸다.** 사람이 찾아내는 게 아니라 시스템이 먼저 알려준다.

**Phase 2에서 이 구조는 한 바퀴 더 돈다.** 그렇게 쌓인 팬 데이터를 무기로 제휴·스폰서십 계약까지 연결하는 PRM(Partnership Relationship Management) 영역을 같은 Org 위에 올린다.

---

## 내가 맡은 역할

**Business Analyst / Demo Experience Lead**

Object나 Field를 먼저 설계하는 게 아니라, **비즈니스 스토리를 먼저 만들고 그 스토리에 필요한 데이터를 찾아가는** 순서로 일했다.

| 담당 | 내용 |
|---|---|
| Customer Journey 설계 | 신규 팬이 충성 팬이 되기까지의 Happy Path를 8개 Scene으로 구조화 |
| 데모 시나리오 작성 | Scene별 화면·발표 멘트·필요 데이터 정의 |
| **Sample / Dummy Data 기획·구축** | 시나리오가 실제로 동작하도록 데이터 설계 후 Org 적재 |
| NBA 문구 기획 | 추천(Recommendation)·알림(Notification)에 들어갈 실제 메시지 작성 |
| **Partner Account / Contact 구축** (Phase 2) | 제휴사·담당자 레코드와 Partner 전용 화면 계층(Layout·Record Page) 구성 |
| **파트너십 화면 필드 설계** (Phase 2) | "담당자가 이 화면에서 답해야 할 질문"에서 역산한 필드 설계 · 리스트 뷰 컬럼 재설계 |

팀은 5명이고, Object/Flow 구축은 Salesforce Builder가, 화면·권한·QA는 Experience Lead가, Fan App 개발은 Developer Lead가 맡는다. 나는 **"무엇을 왜 만드는가"와 "그게 데모에서 어떻게 보이는가"** 사이를 잇는 역할이다.

---

## 사용한 도구

| 도구 | 용도 |
|---|---|
| **Salesforce** | Person Account, Order/OrderItem, Custom Object 11종, Flow, Lightning App |
| **Salesforce Data Loader** | CSV 대량 적재 (Insert / Update / Upsert / Export) |
| **Agentforce Vibes IDE (Salesforce DX)** | `force-app` 메타데이터 소스 관리 — RecordType·ListView·FlexiPage를 XML로 작성해 배포 |
| **REST / Tooling API** | Org 상태·스키마·실제 필드 바인딩 검증 (Setup UI가 라벨만 보여줄 때의 유일한 확인 수단) |
| **Developer Console** | SOQL로 Org 상태·Id·스키마 확인 |
| **Object Manager (Setup)** | 필드 정의·픽리스트 값·종속 관계 확인 |
| **Lightning App Builder** | Dynamic Forms 전환, Record Page 구성 |
| **Google Sheets** | 팀 공용 메타데이터 정의서 (Object·Field·픽리스트 값) |
| **Python / Node.js** | 참조 무결성이 맞는 더미 데이터 생성 스크립트 |
| **GitHub** | 문서·기록·메타데이터 관리 |
| **Claude (Cowork)** | 데이터 생성 스크립트 작성, 오류 원인 분석, 문서 초안 |

---

## 진행 기록

작업한 날짜별로 무엇을 했고 어디서 막혔는지 남긴다.

| 날짜 | 내용 |
|---|---|
| [2026-08-28](worklog/2026-08-28.md) | 스폰서십 계약 이력 5년치(Opp 253·Order 220·Asset 99) + Lead 전환 시 DART 공시 자동보강 시스템 구축·배포 |
| [2026-08-27](worklog/2026-08-27.md) | Partnership 계정 화면 정상화(레코드페이지 활성화) + 소유권 이관 300건 + 공유규칙 + 스폰서 등급 자동화 |
| [2026-08-25](worklog/2026-08-25.md) | 파트너십 화면 필드 설계 + 더미 100건 + 사흘 전 심은 ListView 버그 추적 |
| [2026-08-24](worklog/2026-08-24.md) | FRM/PRM Account·Contact 혼재 해소, RecordType 명칭 정비 (Vibes IDE) |
| [2026-08-21](worklog/2026-08-21.md) | Partner CRM 개선 — Relationship Type, Partnership Status, FlexiPage 탭 재편 |
| [2026-08-20](worklog/2026-08-20.md) | Lead Convert 실검증 + PRM 더미 데이터 105건 + Partner 화면 계층 구축 |
| [2026-08-19](worklog/2026-08-19.md) | Phase 2 제휴사 Account 생성 + 레코드타입 권한 장애 + Org 실측 |
| [2026-08-14](worklog/2026-08-14.md) | 1차 데모 실측 검토(치명적 결함 2건) + B2B 확장 방향 결정 |
| [2026-08-13](worklog/2026-08-13.md) | 데모용 더미 데이터 설계 → Org 적재 (Object 15종 · 약 270건) |

> 작업 기록은 **`portfolio/worklog/` 한 곳**에 모은다. 8/24 DX 프로젝트 전환 때 잠시 `docs/work-log/`가 생겼지만 8/25에 이쪽으로 통합했다.

---

## 문서

- [프로젝트 개요](docs/project-overview.md) — 문제 정의, 페르소나, 데모 시나리오
- [데이터 모델 메모](docs/data-model-notes.md) — 실제로 적재하며 알게 된 제약과 함정

---

## 레포 구조

이 저장소는 두 가지를 같이 담고 있다.

| 경로 | 무엇 |
|---|---|
| `portfolio/` | **이 문서가 있는 곳.** 작업 기록(`worklog/`)과 설명 문서(`docs/`) |
| `force-app/` · `config/` · `manifest/` · `scripts/` | Salesforce DX 프로젝트 — Org 메타데이터(RecordType·ListView·Field·FlexiPage)를 XML 소스로 관리 |

2026-08-24부터 Setup 화면 클릭 대신 **Agentforce Vibes IDE에서 메타데이터를 소스로 관리해 배포**하는 방식으로 바꿨다. 무엇이 언제 왜 바뀌었는지가 git 히스토리에 남는다.

---

## 현재 상태

> Phase 1(B2C Fan 360)은 1차 데모까지 마쳤고, 현재는 **Phase 2 — 팬 데이터를 제휴·스폰서십 영업에 쓰는 B2B 확장**을 진행 중이다.

**B2C (FRM)**

- 데이터 모델 설계 완료 (표준 Object 8종 + Custom Object 11종)
- 데모 시나리오 8개 Scene 확정 — Phase 2 반영해 재구성 중
- Fan(Person Account) 5,005건 적재. 다만 **가입일이 하루에 몰려 상대 날짜 기반 세그먼트가 전부 0명**, 날짜 역산 재배치가 남아 있음

**B2B (PRM)**

- 담당 체인 혜준(Lead) → **아론(Account/Contact)** → 은영(Opportunity) → 승우(Product/Quote/Campaign)
- Partner Account 104건 · 08-25 생성 Opportunity 100건 (Closed Won 73 / Contracting 6 / Closed Lost 6 / Proposal 4 / Discovery 4 / Negotiation 4 / Qualification 3)
- Partnership Status 분포 — Active 50 / Pending 21 / Onboarding 20 / Inactive 6 / Suspended 4 / 미지정 3
- Partner 전용 화면 계층(Page Layout 2 · Compact Layout 1 · Lightning Page 할당 8) 구축, Dynamic Forms 전환 완료
- Lead Convert 동작 검증 완료 — Account·Contact·Opportunity 동시 생성 확인

**남은 것**

- Relationship Type 신규 값 5개 추가, Opportunity Stage 3단계 확장(+ `Channel (Partner)` Sales Process 한정)
- Opportunity 100건이 Master 레코드타입으로 생성됨 — 프로필 할당 후 일괄 수정 필요
- `SDO_` 접두사 중복 필드 전수 점검 — 실제 버그(리스트 뷰 오배선)를 만든 이력 있음
- Flow / 화면 구현은 다른 팀원 진행 중
