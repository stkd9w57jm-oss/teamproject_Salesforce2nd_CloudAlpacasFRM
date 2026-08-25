# Cloud Alpacas FRM — 프로 야구단 Fan 360 CRM 구축

> Salesforce AI CRM 2nd Track 팀 프로젝트 · 2026
> 가상의 프로야구단 **Cloud Alpacas**를 위한 **팬 관계 관리(FRM, Fan Relationship Management)** 시스템을 Salesforce 위에 구축한다.

---

## 무엇을 만드는가

구단 마케팅 담당자(김매니저)는 팬 정보가 티켓 예매 시스템, 굿즈샵, SNS에 흩어져 있어 **"이 팬이 누구인지"를 한 화면에서 볼 수 없다.** VIP가 될 가능성이 높은 팬도 엑셀을 정리하고 나서야 뒤늦게 발견한다.

이 프로젝트는 그 문제를 **Customer 360**으로 푼다. 한 명의 팬이 SNS에서 구단을 처음 알게 된 순간부터 티켓 구매 → 첫 직관 → 굿즈 구매 → 재방문 → 멤버십 가입까지, **흩어진 접점을 하나의 타임라인으로 연결하고 자동화가 그 위에서 동작하게 만든다.**

핵심 시연 장면은 이것이다. 한 팬의 재방문 횟수와 누적 지출이 임계값을 넘는 순간, Flow가 자동으로 VIP 후보를 감지해 **김매니저의 Slack으로 알림을 보낸다.** 사람이 찾아내는 게 아니라 시스템이 먼저 알려준다.

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

팀은 5명이고, Object/Flow 구축은 Salesforce Builder가, 화면·권한·QA는 Experience Lead가, Fan App 개발은 Developer Lead가 맡는다. 나는 **"무엇을 왜 만드는가"와 "그게 데모에서 어떻게 보이는가"** 사이를 잇는 역할이다.

---

## 사용한 도구

| 도구 | 용도 |
|---|---|
| **Salesforce** | Person Account, Order/OrderItem, Custom Object 11종, Flow, Lightning App |
| **Salesforce Data Loader** | CSV 대량 적재 (Insert / Update / Upsert / Export) |
| **Developer Console** | SOQL로 Org 상태·Id·스키마 확인 |
| **Object Manager (Setup)** | 필드 정의·픽리스트 값·종속 관계 확인 |
| **Google Sheets** | 팀 공용 메타데이터 정의서 (Object·Field·픽리스트 값) |
| **Python** | 참조 무결성이 맞는 더미 데이터 CSV 생성 스크립트 |
| **GitHub** | 문서·기록 관리 |
| **Claude (Cowork)** | 데이터 생성 스크립트 작성, 오류 원인 분석, 문서 초안 |

---

## 진행 기록

작업한 날짜별로 무엇을 했고 어디서 막혔는지 남긴다.

| 날짜 | 내용 |
|---|---|
| [2026-08-25](worklog/2026-08-25.md) | Partner Account Contacts 탭 미노출 트러블슈팅 — 원인은 Page Layout Related Lists 누락 |
| [2026-08-20](worklog/2026-08-20.md) | Lead Convert 실검증 + PRM 더미 데이터 105건 + Partner 화면 계층 구축 |
| [2026-08-19](worklog/2026-08-19.md) | Phase 2 제휴사 Account 생성 + 레코드타입 권한 장애 + Org 실측 |
| [2026-08-14](worklog/2026-08-14.md) | 1차 데모 실측 검토(치명적 결함 2건) + B2B 확장 방향 결정 |
| [2026-08-13](worklog/2026-08-13.md) | 데모용 더미 데이터 설계 → Org 적재 (Object 15종 · 약 270건) |

---

## 문서

- [프로젝트 개요](docs/project-overview.md) — 문제 정의, 페르소나, 데모 시나리오
- [데이터 모델 메모](docs/data-model-notes.md) — 실제로 적재하며 알게 된 제약과 함정

---

## 현재 상태

> Phase 1(B2C Fan 360)은 1차 데모까지 마쳤고, 현재는 **Phase 2 — 팬 데이터를 제휴·스폰서십 영업에 쓰는 B2B 확장**을 진행 중이다.

- 데이터 모델 설계 완료 (표준 Object 8종 + Custom Object 11종)
- 데모 시나리오 8개 Scene 확정 — Phase 2 반영해 재구성 중
- B2C 더미 데이터 적재 완료 — Fan 5,024건. 다만 **가입일이 하루에 몰려 상대 날짜 기반 세그먼트가 전부 0명**, 날짜 역산 재배치가 남아 있음
- B2B 파이프라인 구축 중 — 담당 체인 혜준(Lead) → **아론(Account/Contact)** → 은영(Opportunity) → 승우(Product/Quote/Campaign)
- Partner Account 107건 · Opportunity 6건(Closed Won 1건 포함, 합계 ₩9.5억) 적재, Lead Convert 동작 검증 완료
- Partner 전용 화면 계층(Page Layout 2 · Compact Layout 1 · Lightning Page 할당 8) 구축 완료 — 이후 Related Lists 누락 등 세부 결함 발견·수정 진행 중 (08-25)
- 정리 중 — 제휴사 담당자 Contact 107건이 Player 레코드타입으로 적재됨, Partner Type `Sponsor` 값이 레코드타입에 미할당
- Flow / 화면 구현은 다른 팀원 진행 중
