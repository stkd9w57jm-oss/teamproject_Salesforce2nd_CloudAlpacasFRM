# 작업 기록 — 2026-08-24

## 작업 주제

FRM / PRM App 간 Account · Contact 혼재 문제 해결 및 Record Type 정비

---

## 배경 및 문제

Cloud Alpacas Org에서 FRM(Fan Relationship Management)과 PRM(Partnership Relationship Management)이 하나의 Org에서 운영되면서, Account 탭과 Contact 탭을 열면 서로 다른 업무 맥락의 레코드가 섞여 표시되는 문제가 있었다.

| App | Account 의미 | Contact 의미 |
|---|---|---|
| FRM | Fan (PersonAccount) | 야구 선수 (Player) |
| PRM | 파트너십 체결 기업 | 파트너사 담당자 |

기존 Contact Record Type 중 `Partner`는 Salesforce 기본 PRM 개념인 "distributor / reseller의 담당자"로 정의되어 있어, Cloud Alpacas의 실제 업무(스포츠 구단 파트너사 담당자)와 맞지 않았다.

Account Record Type도 `Partner`라는 명칭이 채널 판매 파트너 의미를 내포하고 있어 스폰서십·제휴 기업을 담당하는 PRM 업무와 어울리지 않았다.

---

## 해결 방향

### 1. Record Type 명칭 정비

| Object | 기존 Label | 변경 Label | 비고 |
|---|---|---|---|
| Account | `Partner` | `Partnership Account` | API Name(`SDO_Account_Partner`) 유지 — 기존 레코드 연결 보존 |
| Contact | `Partner` | `Partnership Contact` | API Name(`Partner`) 유지 |
| Contact | `Contact` (범용) | 유지 (비활성화 안 함) | — |

### 2. Contact Record Type Description 수정

- 기존: `"distributors or resellers"`
- 변경: `"파트너십 계약을 맺은 회사의 담당자"`

### 3. Account ListView 신규 생성

- 이름: `Partnership Accounts`
- 필터: `RecordType.DeveloperName = SDO_Account_Partner`
- 컬럼: 이름 / Relationship Type(`SDO_Partner_Type__c`) / Partnership Status(`SDO_Partnership_Status__c`) / 전화 / 업종 / 담당자

---

## 변경된 파일

| 파일 경로 | 변경 내용 |
|---|---|
| `force-app/main/default/objects/Account/recordTypes/SDO_Account_Partner.recordType-meta.xml` | label `Partner` → `Partnership Account` |
| `force-app/main/default/objects/Contact/recordTypes/Partner.recordType-meta.xml` | 신규 생성 — label `Partnership Contact`, description 업데이트 |
| `force-app/main/default/objects/Account/listViews/PartnershipAccounts.listView-meta.xml` | 신규 생성 — Partnership Account 전용 List View |

---

## 트러블슈팅

### ListView 컬럼 API 포맷 오류 (배포 3회 반복)

**문제:** `listView-meta.xml`의 `<columns>` 값 포맷이 Salesforce 표준과 맞지 않아 배포 실패.

| 잘못된 값 | 올바른 값 | 비고 |
|---|---|---|
| `NAME` | `ACCOUNT.NAME` | Account 표준 Name 컬럼 |
| `PHONE` | `ACCOUNT.PHONE1` | Account 전화 컬럼 |
| `WEBSITE` | `ACCOUNT.SITE` | Account 웹사이트/업종 컬럼 |
| `OWNER` | `CORE.USERS.ALIAS` | 담당자 컬럼 |
| `RecordType.DeveloperName` (filter field) | `ACCOUNT.RECORDTYPE` | RecordType 필터 필드 |

**해결:** 기존 Org에 존재하는 다른 Account List View 메타데이터를 retrieve하여 실제 포맷 확인 후 수정.

---

## 배포 결과

| 컴포넌트 | 배포 상태 |
|---|---|
| `Account.SDO_Account_Partner` RecordType | 성공 |
| `Contact.Partner` RecordType | 성공 |
| `Account.PartnershipAccounts` ListView | 성공 |

---

## 변경하지 않은 항목

| 항목 | 이유 |
|---|---|
| Contact `Player` Record Type | FRM 업무와 맞으므로 그대로 유지 |
| Contact 범용 `Contact` Record Type | 비활성화 시 기존 레코드 영향 우려 — 유지 |
| Account `Business_Account` Record Type | 이미 존재하며 별도 용도로 사용 중 |
| Flow / Automation | Record Type Label 변경은 Flow 참조에 영향 없음 (API Name 유지) |

---

## 권고 후속 작업

1. **FRM App / PRM App 탭별 기본 List View 설정** — 각 App에서 Account·Contact 탭의 기본 뷰를 Record Type 필터 뷰로 고정하면 혼재 현상이 완전히 해소됨.
2. **Contact용 Partnership Contact List View 생성** — Account와 동일하게 Contact 탭에서 `Partnership Contact` Record Type만 필터링하는 뷰 추가 권고.
3. **Player Contact용 List View 생성** — FRM App에서 선수 Contact만 표시하는 뷰 추가 권고.

---

## 관련 작업 기록

- [2026-08-21 — Partner CRM 개선 (Relationship Type, Partnership Status, FlexiPage 탭 재편)](./2026-08-21-partner-crm-improvement.md)
