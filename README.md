# Cloud Alpacas FRM/PRM — Salesforce DX Project

Cloud Alpacas 팀의 Salesforce 개발 소스를 관리하는 DX 프로젝트입니다.  
FRM(Fan Relationship Management)과 PRM(Partnership Relationship Management) 두 앱을 하나의 Org에서 운영합니다.

---

## 개발 환경 세팅 (새 컴퓨터에서 시작할 때)

### 1. 필수 도구 설치

| 도구 | 설명 |
|---|---|
| [Salesforce CLI](https://developer.salesforce.com/tools/salesforcecli) | `sf` 명령어 |
| [VS Code](https://code.visualstudio.com/) + [Salesforce Extension Pack](https://marketplace.visualstudio.com/items?itemName=salesforce.salesforcedx-vscode) | 개발 IDE |
| [Node.js](https://nodejs.org/) (v18 이상) | Jest 테스트 실행용 |
| Git | 버전 관리 |

### 2. 저장소 클론 및 의존성 설치

```bash
git clone https://github.com/stkd9w57jm-oss/teamproject_Salesforce2nd_CloudAlpacasFRM.git
cd teamproject_Salesforce2nd_CloudAlpacasFRM
npm install
```

### 3. Org 인증

```bash
# 개발 Org 로그인
sf org login web --alias CloudAlpacas
```

### 4. 메타데이터 배포

```bash
# 전체 배포
sf project deploy start --source-dir force-app --target-org CloudAlpacas

# 특정 컴포넌트만 배포
sf project deploy start --metadata CustomObject:Account --target-org CloudAlpacas
```

### 5. 메타데이터 가져오기 (Org → 로컬)

```bash
sf project retrieve start --source-dir force-app --target-org CloudAlpacas
```

---

## 프로젝트 구조

```
dx-project/
├── force-app/main/default/
│   ├── applications/       # FRM / PRM Lightning App 정의
│   ├── objects/            # Custom Object, Field, RecordType, ListView
│   ├── flexipages/         # Lightning Record Page
│   ├── layouts/            # Page Layout
│   ├── lwc/                # Lightning Web Components
│   ├── classes/            # Apex 클래스
│   ├── permissionsets/     # Permission Set
│   ├── tabs/               # Custom Tab
│   └── ...
├── config/                 # Scratch Org 정의 파일
├── scripts/                # 자동화 스크립트
├── docs/
│   └── work-log/           # 날짜별 작업 기록
├── sfdx-project.json
└── package.json
```

---

## 앱 구성

| App | 대상 | 주요 Object |
|---|---|---|
| **FRM** (Fan Relationship Management) | 팬 관리 | Account (PersonAccount), Contact (Player) |
| **PRM** (Partnership Relationship Management) | 파트너사 관리 | Account (Partnership Account), Contact (Partnership Contact) |

---

## 주요 Record Type

| Object | Record Type Label | API Name | 용도 |
|---|---|---|---|
| Account | Partnership Account | `SDO_Account_Partner` | 파트너십 체결 기업 |
| Contact | Partnership Contact | `Partner` | 파트너사 담당자 |
| Contact | Player | `Player` | 야구 선수 (FRM) |

---

## 작업 기록

`docs/work-log/` 폴더에 날짜별 작업 내용이 기록되어 있습니다.

| 날짜 | 내용 |
|---|---|
| [2026-08-24](./docs/work-log/2026-08-24-frm-prm-account-contact-separation.md) | FRM/PRM Account·Contact RecordType 정비, Partnership Accounts ListView 생성 |
| [2026-08-21](./docs/work-log/2026-08-21-partner-crm-improvement.md) | Partner CRM 개선 — Relationship Type, Partnership Status 필드, FlexiPage 탭 재편 |

---

## 자주 쓰는 명령어

```bash
# Org 상태 확인
sf org list

# 현재 연결된 Org 열기
sf org open --target-org CloudAlpacas

# Apex 테스트 실행
sf apex run test --target-org CloudAlpacas --result-format human

# SOQL 쿼리
sf data query --query "SELECT Id, Name, RecordType.Name FROM Account LIMIT 10" --target-org CloudAlpacas
```
