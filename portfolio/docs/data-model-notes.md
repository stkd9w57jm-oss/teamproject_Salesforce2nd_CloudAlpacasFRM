# 데이터 적재 메모

설계 문서만 보고는 알 수 없었던 것들. 실제로 Org에 넣어보면서 하나씩 부딪힌 기록이다.
**설계 문서는 의도이고, 진짜 기준은 Org다.**

---

## 도구 선택

| | Data Import Wizard | Data Loader |
|---|---|---|
| 위치 | Setup 안 (설치 불필요) | 별도 설치 |
| 지원 Object | Account, Contact, Lead, Person Account, 커스텀 오브젝트 | **전부** |
| 최대 건수 | 5만 | 500만 |

Order, OrderItem, Product2, PricebookEntry, Campaign, Case는 **Import Wizard가 지원하지 않는다.** 파일마다 도구를 바꾸면 헷갈리니 처음부터 Data Loader로 통일하는 게 낫다.

---

## 관계를 연결하는 두 가지 방법

CSV에는 Salesforce Id가 없다. 부모 레코드를 어떻게 가리킬 것인가.

### External Id 방식

각 Object에 `External_Id__c` (Text, Unique, External ID) 필드를 만들고, CSV의 임시 번호(`FAN-01`)를 그 값으로 쓴다. Upsert 시 Data Loader가 알아서 찾아 연결한다.

**제약이 두 가지 있다.**

- **Upsert에서만 동작한다.** Insert 마법사에는 관계 지정 화면(Step 2b)이 없다. Insert로 넣으면 오류 없이 관계만 조용히 빈다
- **PricebookEntry는 커스텀 필드를 못 만든다** → 이 방식 자체가 불가능

Upsert 마법사 **Step 2b "Choose your related objects"** 에서 Lookup마다 드롭다운을 `External_Id__c`로 바꿔야 다음 매핑 화면에 `Fan__r:External_Id__c` 항목이 나타난다. 기본값이 `Id`라서 그냥 넘기면 매핑 목록에 아예 안 보인다.

### 실제 Id를 직접 채우는 방식

부모를 넣고 → Export나 SOQL로 Id를 뽑고 → 자식 CSV에 채운다.

손이 더 가지만 **Insert로도 되고, 어떤 Object에서든 통한다.** External Id 필드가 일부 Object에만 만들어져 있는 상황에서는 이쪽이 확실했다.

---

## 적재 순서와 제약

```
Season → Contact(선수) → Product2 → PricebookEntry → Game
→ Person Account → Order(Draft) → OrderItem → Order를 Activated로
→ Attendance Record → Admission
→ Engagement Signal / Activity Pattern / Segment History
→ Recommendations → Benefits
→ Campaign → CampaignMember / Notification Log
→ Case
```

**순서를 지켜야 하는 이유**

| 제약 | 내용 |
|---|---|
| PricebookEntry → OrderItem | **가격표가 없으면 OrderItem이 아예 안 들어간다** |
| Order는 Draft로 | **Draft 상태에서만 상품을 추가할 수 있다.** OrderItem을 넣은 뒤 Activated로 일괄 Update |
| 상품 없는 Order | **Activated로 바꿀 수 없다.** OrderItem이 실패하면 그 Order도 활성화 불가 |
| Attendance Record → Admission | Master-Detail 부모가 먼저 있어야 한다 |
| OrderItem의 상품 | **`Product2Id`를 직접 넣을 수 없다.** `PricebookEntryId`를 넣으면 상품이 따라온다 |

---

## 자주 만난 오류와 원인

### `Record Type ID: this ID value isn't valid for the user`

Id는 맞는데 **프로파일에 할당되지 않았다.** Record Type을 새로 만들면 자동으로 붙지 않는다.

> Setup → Profiles → 해당 프로파일 → Object Settings → 대상 Object → Edit → Record Type을 Assigned로

Record Type은 **이름으로 넣을 수 없다.** CSV에 `Player`가 아니라 18자리 Id가 들어가야 한다. Object Manager에서 Record Type을 열면 URL 끝에 있다.

### `Field name provided, Id is not an External ID or indexed field for OOO`

그 Object에 **External Id 필드가 없거나**, Step 2b에서 Lookup Field를 `Id`로 두고 넘어간 경우.

### `CostBook ID: id value of incorrect type`

`Pricebook2Id` 칸에 Id가 아니라 "Standard Price Book"이라는 **글자**를 넣었을 때.

### `Required fields are missing: [Name]`

PricebookEntry에서 `Product2Id`가 비어 있을 때. Salesforce가 상품을 못 찾아서 나는 오류인데 문구만 봐서는 원인을 알 수 없다.

### Data Loader Object 목록에 `Campaign`이 없다

Campaign 생성에는 User 레코드의 **`Marketing User`** 체크가 별도로 필요하다. Data Loader는 "내가 만들 수 있는 Object"만 보여주므로 통째로 빠진다. CampaignMember도 같은 권한이 필요하다.

### `Show all Salesforce objects` 체크박스

`PricebookEntry` 같은 Object는 기본 목록에 안 나온다. Step 2 왼쪽 아래 체크박스를 켜야 보인다.

---

## Person Account 관련

- Object는 **Contact이 아니라 `Account`** 를 고른다
- `LastName`, `PersonEmail`, `PersonBirthdate` 같은 Person 필드를 쓴다
- **CampaignMember는 Person Account를 직접 못 가리킨다.** `Account.PersonContactId`(숨은 Contact)를 `ContactId`에 넣어야 한다
- Person Account 활성화는 **한 번 켜면 되돌릴 수 없다**

---

## 문서와 실제 Org가 달랐던 것

| 항목 | 정의서 | 실제 Org |
|---|---|---|
| Benefit Object | `Benefit__c` | `Benefits__c` |
| Recommendation Object | `Recommendation__c` | `Recommendations__c` |
| `Position__c` | 투수/포수/내야수/외야수 | 1루수·2루수·좌익수·우익수 등 세분화 |
| `Refund_Reason__c` | `[Ticket] 경기취소` | `경기취소` (Order Type 종속 픽리스트) |
| `Section__c` | 내야 지정석 | 값 세트에 없음 (제한 픽리스트) |
| `Purchase_Channel__c` | 온라인 / 구장 굿즈샵 | `Online` / `Stadium` |

**제한 픽리스트**("값 세트에 정의된 값으로 제한"이 켜진 경우)는 정의되지 않은 값을 거부한다. 데이터를 만들기 전에 **Object Manager에서 픽리스트 값을 먼저 확인**하는 게 맞다.

---

## Org 설정도 먼저 봐야 한다

스키마만 보고 Org 설정을 안 봤다가 Order 계열을 통째로 다시 만들게 됐다.

- **통화** — Org 기본 통화가 USD면 10,000원이 $10,000으로 들어간다. Setup → Company Information → Currency Locale
- **데이터 저장 한도** — Developer Edition은 5MB(≈2,500레코드). Person Account는 1건이 2건 분량을 차지한다
- **공용 Org** — 다른 팀원이 넣은 데이터가 이미 있을 수 있다. 우리 Org에는 상품 151건, 주문 14건이 먼저 들어와 있었다. 접두사(`PRD-`)로 구분하고 날짜로 걸러냈다

---

## 되돌리기

Data Loader **Delete**에 `success...csv`를 그대로 넣으면 방금 넣은 것만 지워진다. 단 **넣은 역순으로** 지워야 한다. 자식이 남아 있으면 부모가 안 지워진다.
