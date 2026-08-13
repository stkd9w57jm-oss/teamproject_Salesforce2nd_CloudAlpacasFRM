# 프로젝트 개요

## 문제

가상의 프로야구단 **Cloud Alpacas**의 마케팅 담당자 김매니저에게는 네 가지 문제가 있다.

1. **팬 정보가 흩어져 있다.** 티켓 예매 시스템, 굿즈샵 POS, SNS가 따로 논다. 한 팬이 어디서 왔고 무엇을 했는지 한 화면에서 볼 수 없다.
2. **관심의 시작을 놓친다.** 팬은 보통 SNS에서 선수 영상을 보다가 구단을 알게 되는데, 그 신호는 어디에도 기록되지 않는다. 티켓을 사야 비로소 고객이 된다.
3. **개인화된 안내를 못 한다.** 누가 어떤 선수를 좋아하는지 모르니 전체 발송밖에 못 한다.
4. **VIP를 늦게 발견한다.** 자주 오고 많이 쓰는 팬을 엑셀 정리 후에야 알게 된다. 그때는 이미 늦다.

## 접근

Salesforce의 **Customer 360** 개념을 팬에게 적용한다. 팬 한 명을 Person Account로 두고, 그 사람에게 일어난 모든 사건을 하나의 타임라인으로 모은다.

```
SNS 반응 → 가입 → 티켓 구매 → 첫 직관 → 굿즈 구매 → 재방문 → VIP 후보 감지 → 멤버십 가입
```

각 단계는 Object로 표현되고, 단계 사이의 전환은 Flow가 자동으로 처리한다.

## 데모 시나리오

주인공은 **이루키**, 27세 직장인이다. SNS에서 문태양 선수 영상을 보고 구단을 알게 된 신규 팬이 충성 팬이 되기까지를 8개 Scene으로 보여준다.

| Scene | 팬 앱 | Salesforce |
|---|---|---|
| 1. SNS | 문태양 영상 시청 | — |
| 2. 회원가입 | 가입 | Person Account 생성, Welcome 알림 |
| 3. 첫 티켓 구매 | 예매 | Order + OrderItem |
| 4. 첫 직관 | 입장 | Admission 생성 → **New Fan → Active Fan 자동 전환** |
| 5. 첫 굿즈 구매 | 유니폼 구매 | 최애 선수 기반 추천 자동 생성 |
| 6. 재방문 | 여러 번 관람 | Activity Pattern 누적 |
| 7. **VIP 후보 감지** | — | 조건 충족 → **Slack 알림 발송** ★ |
| 8. 충성 팬 | 멤버십 가입 | 전체 여정이 하나의 타임라인에 |

**Scene 7이 이 데모의 핵심이다.** 김매니저가 찾아내는 게 아니라, 조건이 충족되는 순간 시스템이 먼저 Slack으로 알려준다. Pain Point 4번에 대한 직접적인 답이다.

## 데이터 모델

**표준 Object를 최대한 그대로 쓴다**는 원칙을 세웠다. 커스텀으로 만들면 나중에 "왜 이 Object가 필요한지" 설명하기 어려워지고, Price Book이나 Order 같은 표준 기능과도 끊긴다.

**표준 Object**

| Object | 표현하는 것 |
|---|---|
| Person Account | Fan |
| Contact (RecordType = Player) | 선수 |
| Product2 / PricebookEntry | 티켓 · 시즌권 · 멤버십 · 굿즈와 가격 |
| Order / OrderItem | 티켓 구매 · 굿즈 구매 · 멤버십 가입 |
| Campaign / CampaignMember | 마케팅 캠페인 |
| Case | 팬 문의 |

**Custom Object**

| Object | 왜 필요한가 |
|---|---|
| `Season__c` | 시즌별 경기 수 집계 기준. 관람률 계산의 분모 |
| `Game__c` | 경기. 표준 Object가 없다 |
| `Admission__c` | 게이트 통과 1건. "몇 번 왔나"와 "언제 왔나"를 구분해야 한다 |
| `Attendance_Record__c` | 누적 관람 이력. Roll-Up으로 자동 집계 |
| `Engagement_Signal__c` | **구매 이전의 관심 신호.** Pain Point 2번의 답 |
| `Fan_Activity_Pattern__c` | 시즌별 활동 패턴. VIP 후보 감지의 근거 |
| `Fan_Segment_History__c` | 세그먼트 변화 이력. "언제 바뀌었나"가 자동화의 근거 |
| `Recommendations__c` | Next Best Action 결과물 |
| `Benefits__c` | 발급된 쿠폰·할인·선예매권 |
| `Notification_Log__c` | 팬에게 보낸 개인화 안내 이력 |

## 팬을 보는 세 개의 축

한 팬을 세 가지 다른 관점으로 본다. 서로 섞지 않는 게 설계의 핵심이었다.

| 축 | 필드 | 의미 |
|---|---|---|
| **Life Cycle** | `Current_Segment__c` | 지금 활동 주기의 어디에 있는가 (New / Active / At-Risk / Dormant / Churned) |
| **Engagement** | `Engagement_Level__c` | 행동 기반 관여도 (가입 팬 → 핵심 팬) |
| **Fan Value** | `Fan_Value_Tier__c` | 가치 등급 (일반 / 우수 / VIP) |

예를 들어 **VIP인데 At-Risk인 팬**이 있을 수 있다. 많이 쓰지만 최근에 안 오는 사람이다. 축을 하나로 합쳤다면 이 상태를 표현할 수 없다.

## 팀 구성

| 역할 | 담당 |
|---|---|
| PM / Solution Architect | Business Goal, Domain Model, 화면 UX |
| **Business Analyst / Demo Experience Lead** | **Customer Journey, 데모 시나리오, Sample Data** ← 내 역할 |
| Salesforce Builder | Object / Field / Flow 구축 |
| Experience Lead / QA | 화면·권한 구성, QA, 배포 검증 |
| Developer Lead | Fan App 개발, LWC / Apex, Slack 연동 |
