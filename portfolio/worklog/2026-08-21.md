# 2026-08-21 작업 기록 — Cloud Alpacas B2B CRM 화면 개선

## 작업 배경

Cloud Alpacas Salesforce Org의 Account "Partner" Record Type 화면이
Salesforce PRM의 "판매 채널 파트너/리셀러" 관점으로 구성되어 있어,
구단의 실제 B2B 업무(스폰서십, 브랜드 제휴, 공동 마케팅)와 맞지 않았음.
이를 Cloud Alpacas의 Business Context에 맞게 개선.

---

## 사전 영향도 분석

변경 전 현재 Org 상태를 SOQL로 확인한 결과:

- Account RecordType: `SDO_Account_Partner` (Label: Partner) 포함 총 7개
- `Business_Account` RecordType이 이미 별도로 존재 → Option A(Partner 유지 개선) 채택
- Partner 관련 필드 중복 존재 확인:
  - `SDO_Partner_Type__c` / `SDO_Partnership_Status__c` (SDO 계열)
  - `Partnership_Status__c` / `Partner_Certification__c` 등 (별도 생성 필드)
  - `SDO_Partnership_Status__c` 를 주 필드로 사용하기로 결정
- Active Flow 4개 확인 — Partner 관련 필드를 직접 참조하는 Flow 없음 → 변경 안전
- Lightning Record Page: `SDO_Sales_Account_Partner_Account` (Account - Partner Account)
- Opportunity Stage: Discovery / Qualification / Proposal/Quote / Negotiation / Closed Won / Closed Lost (Contracting 없음)

---

## 변경 내용

### 1. `SDO_Partner_Type__c` — Relationship Type으로 개편

**파일:** `force-app/main/default/objects/Account/fields/SDO_Partner_Type__c.field-meta.xml`

| 항목 | 변경 전 | 변경 후 |
|---|---|---|
| Label | Partner Type | Relationship Type |
| description | PRM field used in reporting | Cloud Alpacas와 기업 간의 관계 유형 정의 |
| inlineHelpText | (없음) | 관계 유형 선택 안내 추가 |
| restricted | (미설정) | true |

**Picklist 값 변경:**

| 값 | 상태 |
|---|---|
| Sponsor | 활성 (신규) |
| Strategic Partner | 활성 (신규) |
| Media Partner | 활성 (신규) |
| Marketing Partner | 활성 (신규) |
| Event Partner | 활성 (신규) |
| Technology Partner | 활성 (신규) |
| Other | 활성 (신규) |
| Partner, Reseller, Distributor, Alliance Partner, Service Provider, System Integrator, Service & Support Specialist | 비활성 (데이터 보존) |

---

### 2. `SDO_Partnership_Status__c` — 업무 맥락에 맞게 재정의

**파일:** `force-app/main/default/objects/Account/fields/SDO_Partnership_Status__c.field-meta.xml`

| 항목 | 변경 전 | 변경 후 |
|---|---|---|
| description | Account - Partner page layout에 표시 | 현재 관계 상태 관리 목적 명시 |
| inlineHelpText | (없음) | 상태 선택 안내 추가 |
| restricted | (미설정) | true |

**Picklist 값 변경:**

| 값 | 상태 |
|---|---|
| Prospect | 활성 (신규) |
| Negotiating | 활성 (신규) |
| Active | 활성 (유지) |
| Renewal Due | 활성 (신규) |
| Inactive | 활성 (유지) |
| Pending | 비활성 (데이터 보존) |
| Suspended | 비활성 (데이터 보존) |

---

### 3. `OpportunityStage` — Contracting Stage 추가

**파일:** `force-app/main/default/standardValueSets/OpportunityStage.standardValueSet-meta.xml`

Negotiation → Closed Won 사이에 `Contracting` Stage 추가.
스폰서십/제휴 계약 체결 단계를 명시적으로 관리하기 위함.

| Stage | forecastCategory | probability |
|---|---|---|
| Contracting (신규) | Forecast | 95% |

**최종 Stage 순서:**
Discovery → Qualification → Proposal/Quote → Negotiation → **Contracting** → Closed Won / Closed Lost

---

### 4. FlexiPage `Account - Partner Account` — 탭 구성 재편

**파일:** `force-app/main/default/flexipages/SDO_Sales_Account_Partner_Account.flexipage-meta.xml`

| 탭 | 변경 전 | 변경 후 |
|---|---|---|| Details | Details | 유지 |
| Contacts | Contacts | 유지 |
| 3번 | Channel Sales (ChannelProgramMembers, ChannelAccountPlans, AccountsTo/From) | **Opportunities** (Opportunities 관련 리스트) |
| 4번 | Service (Cases) | Service (유지 — 데이터 접근 보존) |
| 5번 | Channel Marketing (PartnerFundAllocations/Requests/Claims) | **Partnership Activities** (Contacts + ActivityHistories) |

---

### 5. `SDO_Account_Partner` RecordType — Picklist 값 할당 추가

**파일:** `force-app/main/default/objects/Account/recordTypes/SDO_Account_Partner.recordType-meta.xml`

**트러블슈팅:** 배포 후 Relationship Type에서 Sponsor 선택 시 저장 불가 현상 발생.
원인: RecordType 파일에 `SDO_Partner_Type__c`에 대한 `<picklistValues>` 블록이 없었음.
Salesforce restricted picklist는 RecordType에 명시적으로 허용된 값만 저장 가능.

- `SDO_Partner_Type__c` picklist 블록 신규 추가 (7개 활성 값 전체 할당)
- `SDO_Partnership_Status__c` picklist 블록 업데이트 (Active/Inactive → 5개 값으로 확장)

---

## 배포 결과

| 컴포넌트 | 타입 | 결과 |
|---|---|---|
| Account.SDO_Partner_Type__c | CustomField | 성공 |
| Account.SDO_Partnership_Status__c | CustomField | 성공 |
| OpportunityStage | StandardValueSet | 성공 |
| SDO_Sales_Account_Partner_Account | FlexiPage | 성공 |
| Account.SDO_Account_Partner | RecordType | 성공 |

---

## 변경하지 않은 항목 및 이유

| 항목 | 이유 |
|---|---|
| Account RecordType Label ("Partner") | 기존 데이터 및 다른 RecordType(Business_Account)과의 충돌 방지. Label 변경보다 필드로 관계 유형 구분하는 방향 채택 |
| `Partnership_Status__c` (별도 필드) | `SDO_Partnership_Status__c`를 주 필드로 사용하기로 결정. 별도 필드는 Page Layout에서 미노출 권장 |
| Active Flow 4개 | Partner 필드 미참조 확인 후 변경 불필요 판단 |
| Service 탭 컴포넌트(Cases) | Standard 기능 제거보다 탭 유지하여 데이터 접근 보존 |

---

## 최종 구현된 Business Flow

```
Business Account (SDO_Account_Partner RecordType)
  └─ Relationship Type: Sponsor / Strategic Partner / Media Partner 등
  └─ Partnership Status: Prospect → Negotiating → Active → Renewal Due / Inactive
       └─ Opportunity
            └─ Discovery → Qualification → Proposal/Quote
                  → Negotiation → Contracting → Closed Won
                                              → Closed Lost
```
