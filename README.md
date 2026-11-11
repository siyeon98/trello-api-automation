# Trello API Test Automation

A REST API test automation project using **Postman, JavaScript, Newman, and GitHub Actions**, with scheduled execution, JUnit XML reporting, and Slack notifications.

The project validates Trello Board, List, and Card workflows through automated API requests and assertions designed with reference to the official Trello REST API documentation.

**API & Documentation**

- **API Base URL:** [https://api.trello.com](https://api.trello.com)
- **Official API Documentation:** [https://developer.atlassian.com/cloud/trello/rest/](https://developer.atlassian.com/cloud/trello/rest/)

## Tech Stack

| Category | Technologies |
| --- | --- |
| API Testing | Postman, JavaScript |
| Test Execution | Newman, Node.js |
| CI/CD | GitHub Actions |
| Reporting | JUnit XML |
| Workflow Scripting | Python |
| Notifications | Slack Incoming Webhooks, Block Kit |
| Version Control | Git, GitHub |

## Test Coverage

The Postman collection contains **18 API requests** covering the lifecycle of Trello Boards, Lists, and Cards.

| Resource | Requests | Test Scenarios |
| --- | ---: | --- |
| Board | 6 | Create, retrieve, update, verify update, delete, verify deletion |
| List | 6 | Create, retrieve, update, verify update, archive, verify archived state |
| Card | 6 | Create, retrieve, update, verify update, delete, verify deletion |
| **Total** | **18** | **CRUD workflows and resource state validation** |

Assertions verify HTTP status codes, response values, resource IDs, Board–List and List–Card relationships, updated values, archived list status, deleted-resource responses (HTTP 404), and response times below 2,000 ms.

### Verified Test Results

A verified local Newman execution completed with:

| Metric | Result |
| --- | --- |
| Iterations | 1 |
| API Requests | **18 executed, 0 failed** |
| Test Scripts | 36 executed, 0 failed |
| Pre-request Scripts | 27 executed, 0 failed |
| Assertions | **77 passed, 0 failed** |
| Total Duration | 6.5 seconds |
| Average Response Time | 342 ms |

These results represent a verified local execution. Actual execution times may vary.

## Test Design & Implementation

- **API specification-based testing:** Designed requests and assertions with reference to the official Trello REST API documentation.
- **Dynamic test data:** Pre-request scripts generate resource names and reset stored IDs before creation requests.
- **Variable management:** Collection variables pass resource IDs between dependent requests.
- **Sequential execution:** Requests execute in their defined order to validate complete resource lifecycles.

## Project Structure

```text
trello-api-automation/
├── .github/
│   └── workflows/
│       └── api-tests.yml
├── postman/
│   ├── trello.postman_collection.json
│   └── trello.postman_environment.json
├── .gitignore
└── README.md
```

The local environment file `postman/trello.local.postman_environment.json` is excluded from version control.

## Getting Started

### Prerequisites

- Node.js and npm
- Trello account with access to a dedicated test workspace
- Trello API Key and Token

### Installation

Clone the repository:

```bash
git clone https://github.com/siyeon98/trello-api-automation.git
cd trello-api-automation
```

Install Newman:

```bash
npm install -g newman
```

### Environment Configuration

Create a local environment file from the provided template:

```bash
cp postman/trello.postman_environment.json postman/trello.local.postman_environment.json
```

Configure the following variables:

| Variable | Value |
| --- | --- |
| `base_url` | `https://api.trello.com` |
| `key` | Your Trello API Key |
| `token` | Your Trello API Token |

Do not commit the local environment file or expose API credentials.

**Important:** The collection creates, modifies, and deletes real Trello resources. Use a dedicated test workspace rather than production data.

## Running Tests

Run the complete Postman collection:

```bash
newman run postman/trello.postman_collection.json -e postman/trello.local.postman_environment.json
```

The collection must execute in its defined order because subsequent requests depend on resources created by earlier requests.

## Continuous Integration

GitHub Actions executes the Newman collection on a GitHub-hosted Ubuntu runner.

### Execution Triggers

| Trigger | Configuration |
| --- | --- |
| Push | Automatically runs on pushes to `main` |
| Manual | GitHub Actions → Trello API Tests → Run workflow |
| Scheduled | Configured daily at 23:17 Asia/Seoul |

### Workflow Steps

1. Check out the repository and set up Node.js 22.
2. Install Newman and generate a temporary environment file using GitHub Actions Secrets.
3. Execute the Postman collection and export JUnit XML results.
4. Check the report for configured API credential values before artifact upload.
5. Upload the report and send an execution summary to Slack.

[View GitHub Actions runs](https://github.com/siyeon98/trello-api-automation/actions)

### Test Reporting

| Setting | Value |
| --- | --- |
| Report Format | JUnit XML |
| Report File | `newman-results.xml` |
| Artifact Name | `newman-test-results` |
| Retention Period | 7 days |

The report contains assertion results, failure details, and execution duration. It can be downloaded from individual GitHub Actions runs.

### Slack Test Notifications

GitHub Actions sends structured notifications using **Slack Incoming Webhooks and Block Kit**.

Notifications include the job result, execution trigger, API request count, assertion pass/fail/skip counts, execution duration, up to 10 failed assertions, and a link to the GitHub Actions run.

Successful notification delivery has been verified, including notifications from two scheduled workflow executions.

When JUnit XML results are unavailable, the notification provides the workflow status and execution log link.

## Known Limitations

- **Test isolation:** The collection relies on ordered execution and shared resource IDs rather than independently isolated tests.
- **Response-time threshold:** The 2,000 ms assertion is project-defined, not a Trello service-level agreement. Network latency may affect results.
- **Scheduled execution:** Successful scheduled runs were observed, but consistent execution at the restored daily schedule has not been verified.
- **Test coverage:** Additional negative and boundary scenarios could improve coverage.
- **Resource cleanup:** Cleanup reliability could be improved for interrupted or partially failed executions.

## Troubleshooting

### Scheduled Workflow Timing

**Issue:** During validation with a temporary 15-minute schedule, GitHub Actions did not execute consistently at the configured intervals.

**Verification:** Two successful scheduled runs were observed approximately four hours apart, both with Slack notifications.

**Current Status:** The workflow was restored to a daily 23:17 Asia/Seoul schedule. Exact execution timing is not guaranteed.

## Security

- Trello API credentials are excluded from the committed collection, and local environment files are ignored by Git.
- CI credentials and the Slack Incoming Webhook URL are stored in GitHub Actions Secrets.
- Temporary CI environment files are generated on the runner.
- The workflow checks JUnit XML artifacts for configured API Key and Token values before uploading.
- Slack notifications contain test summaries rather than raw API request and response bodies.
- Generated reports are excluded from version control.

The report credential check is a targeted safeguard, not a comprehensive secret scanner.