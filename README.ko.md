# Trello API Test Automation

[English](README.md) · [한국어](README.ko.md)

Postman·JavaScript·Newman으로 Trello의 보드·리스트·카드 API 시나리오를 자동화한 프로젝트입니다. GitHub Actions에서 테스트 실행, 리포트 보관, Slack 알림을 구성했습니다.

[Trello REST API documentation](https://developer.atlassian.com/cloud/trello/rest/) · [API base URL](https://api.trello.com)

## 기술 스택

| 구분 | 기술 |
| --- | --- |
| Test Automation | Postman, JavaScript |
| Runtime | Node.js, Newman |
| CI | GitHub Actions |
| Reporting | JUnit XML |
| Workflow Scripting | Python |
| Notifications | Slack Incoming Webhooks, Block Kit |
| Version Control | Git, GitHub |

## 테스트 범위

| 리소스 | 요청 수 | 검증 내용 |
| --- | --- | --- |
| Board | 6 | 생성·조회·수정·수정 확인·삭제·삭제 확인 |
| List | 6 | 생성·조회·수정·수정 확인·보관·보관 상태 확인 |
| Card | 6 | 생성·조회·수정·수정 확인·삭제·삭제 확인 |
| **Total** | **18** | 리소스 수명주기 및 상태 검증 |

상태 코드, 응답값, 리소스 ID, 보드–리스트·리스트–카드 관계, 수정값, 보관 상태, 삭제 후 HTTP 404, 응답 시간 2,000ms 미만을 검증합니다.

### 확인한 테스트 결과

[2026년 11월 17일 GitHub Actions 수동 실행](https://github.com/siyeon98/trello-api-automation/actions/runs/44047663724)에서 확인한 결과입니다.

| 항목 | 결과 |
| --- | --- |
| 반복 실행 | 1 |
| API 요청 | **18 executed, 0 failed** |
| 검증 항목 | **77 passed, 0 failed** |
| 테스트 스크립트 | 36 executed, 0 failed |
| 사전 요청 스크립트 | 27 executed, 0 failed |
| 테스트 실행 시간 | 5.2 seconds |
| 평균 응답 시간 | 269 ms |

위 수치는 해당 실행의 결과입니다. 테스트 실행 시간에는 의존성 설치·리포트 업로드 등 다른 워크플로 단계가 포함되지 않습니다. 실행마다 결과와 시간은 달라질 수 있습니다.

## 테스트 설계 및 구현

- 공식 API 문서를 참고한 요청 및 assertion 설계
- 사전 요청 스크립트로 리소스 이름 생성 및 생성 요청 전 저장 ID 초기화
- 컬렉션 변수로 요청 간 리소스 ID 전달
- 순차 실행으로 리소스 수명주기와 최종 상태 검증

## 프로젝트 구성

| 경로 | 용도 |
| --- | --- |
| `.github/workflows/api-tests.yml` | GitHub Actions |
| `postman/trello.postman_collection.json` | API requests and assertions |
| `postman/trello.postman_environment.json` | Environment template |
| `README.md` | English documentation |
| `README.ko.md` | 한국어 문서 |

## 시작하기

### 사전 준비

- Node.js 및 npm
- Trello 계정, 테스트 전용 워크스페이스, API Key 및 Token

### 설치

```bash
git clone https://github.com/siyeon98/trello-api-automation.git
cd trello-api-automation
npm install -g newman
```

### 환경변수 설정

환경변수 템플릿을 복사하고 API 인증 정보를 입력합니다.

```bash
cp postman/trello.postman_environment.json postman/trello.local.postman_environment.json
```

| Variable | Value |
| --- | --- |
| `base_url` | `https://api.trello.com` |
| `key` | Trello API Key |
| `token` | Trello API Token |

컬렉션은 실제 리소스를 생성·수정·삭제합니다. 테스트 전용 워크스페이스를 사용합니다.

로컬 환경 파일은 Git에서 제외됩니다. 인증 정보는 버전 관리에 포함하지 않습니다.

## 테스트 실행

```bash
newman run postman/trello.postman_collection.json -e postman/trello.local.postman_environment.json
```

뒤 요청이 앞 요청에서 만든 리소스에 의존하므로 컬렉션의 정의된 순서대로 실행합니다.

## 지속적 통합

GitHub Actions의 GitHub-hosted Ubuntu runner에서 Newman 컬렉션을 실행합니다.

| Setting | Value |
| --- | --- |
| Workflow | Trello API Tests |
| Job | API Tests |
| Node.js | `22` |

### 실행 조건

| 조건 | 설정 |
| --- | --- |
| Push | `main` 브랜치 push |
| Manual | GitHub Actions → Trello API Tests → Run workflow |
| Scheduled | 기본 브랜치에서 매일 03:23 Asia/Seoul로 설정 |

예약 실행은 설정 시각보다 늦게 시작될 수 있습니다.

[2026년 11월 17일 예약 실행](https://github.com/siyeon98/trello-api-automation/actions/runs/43998951532)이 성공했습니다.

### 워크플로 단계

1. 저장소를 체크아웃하고 Node.js를 설정합니다.
2. Newman을 설치하고 GitHub Actions Secrets로 임시 환경 파일을 생성합니다.
3. 컬렉션을 실행하고 JUnit XML 결과를 생성합니다.
4. 리포트에 설정된 API 인증 정보 값이 포함됐는지 검사합니다.
5. 리포트를 업로드하고 실행 요약을 Slack으로 전송합니다.

### 테스트 리포트

| 설정 | 값 |
| --- | --- |
| Report Format | JUnit XML |
| Artifact Name | `newman-test-results` |
| Retention Period | 30 days |
| Upload Condition | 리포트 인증 정보 검사 통과 시 |

생성·업로드된 리포트는 해당 GitHub Actions 실행 페이지의 **Artifacts**에서 내려받을 수 있습니다.

### Slack 테스트 알림

Slack Incoming Webhooks와 Block Kit을 사용합니다. 알림에는 job 결과, 실행 조건, 실행 링크가 포함됩니다.

JUnit XML 결과가 있으면 요청 수, 검증 통과·실패·스킵 건수, 리포트에 있는 실행 시간, 실패 검증 항목 최대 10건을 포함합니다. 알림 단계는 `always()` 조건을 사용합니다.

리포트 데이터를 사용할 수 없으면 job 결과와 실행 링크를 제공합니다. Slack 전송 실패는 GitHub Actions에 경고로 기록하며 job 실패로 처리하지 않습니다. 테스트 실패는 job 실패로 처리합니다.

예약 실행을 포함한 성공 실행의 Slack 전달을 확인했습니다.

## 알려진 제약 사항

- **테스트 격리** — 요청 순서와 공유 리소스 ID에 의존합니다.
- **응답 시간** — 2,000ms는 프로젝트에서 정한 임계값이며 Trello SLA로 확인한 값이 아닙니다. 네트워크 지연의 영향을 받습니다.
- **검증 범위** — 비정상 입력과 경계값 시나리오를 추가할 여지가 있습니다.
- **데이터 정리** — 중단·부분 실패 시 테스트 리소스가 남아 확인과 정리가 필요할 수 있습니다.

## 인증 정보 관리

- 로컬 환경 파일은 Git에서 제외합니다.
- CI 인증 정보와 Slack webhook은 GitHub Actions Secrets에 저장합니다.
- `TRELLO_API_KEY`, `TRELLO_API_TOKEN`, `SLACK_WEBHOOK_URL`을 사용합니다.
- CI 환경 파일은 runner 임시 디렉터리에 생성합니다.
- JUnit XML 업로드 전에 설정된 API Key·Token 값의 포함 여부를 검사합니다. 검사 범위는 해당 값으로 제한됩니다.
- Slack에는 요약을 전송하고 생성 리포트는 버전 관리에서 제외합니다.
