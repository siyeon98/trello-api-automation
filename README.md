# Trello API Test Automation

[English](README.md) · [한국어](README.ko.md)

Automated REST API tests for Trello Board, List, and Card workflows using Postman, JavaScript, and Newman. GitHub Actions provides automated execution, reports, and Slack notifications.

[Trello REST API documentation](https://developer.atlassian.com/cloud/trello/rest/) · [API base URL](https://api.trello.com)

## Tech Stack

| Category | Technologies |
| --- | --- |
| Test Automation | Postman, JavaScript |
| Runtime | Node.js, Newman |
| CI | GitHub Actions |
| Reporting | JUnit XML |
| Workflow Scripting | Python |
| Notifications | Slack Incoming Webhooks, Block Kit |
| Version Control | Git, GitHub |

## Test Coverage

| Resource | Requests | Coverage |
| --- | --- | --- |
| Board | 6 | Create, retrieve, update, verify update, delete, verify deletion |
| List | 6 | Create, retrieve, update, verify update, archive, verify archived state |
| Card | 6 | Create, retrieve, update, verify update, delete, verify deletion |
| **Total** | **18** | Resource lifecycle and state validation |

Assertions cover status codes, response values, resource IDs, Board–List and List–Card relationships, updated values, archived state, HTTP 404 after deletion, and response times below 2,000 ms.

### Verified Test Results

Results from the [November 17, 2026 manual GitHub Actions run](https://github.com/siyeon98/trello-api-automation/actions/runs/44047663724).

| Metric | Result |
| --- | --- |
| Iterations | 1 |
| API Requests | **18 executed, 0 failed** |
| Assertions | **77 passed, 0 failed** |
| Test Scripts | 36 executed, 0 failed |
| Pre-request Scripts | 27 executed, 0 failed |
| Test Execution Duration | 5.2 seconds |
| Average Response Time | 269 ms |

These figures describe this execution. Test duration excludes other workflow steps such as dependency installation and artifact upload. Results and timings may vary between runs.

## Test Design & Implementation

- Requests and assertions designed with reference to the official API documentation.
- Pre-request scripts generate resource names and reset stored IDs before creation requests.
- Collection variables pass resource IDs between dependent requests.
- Ordered execution verifies resource lifecycles and final states.

## Project Structure

| Path | Purpose |
| --- | --- |
| `.github/workflows/api-tests.yml` | GitHub Actions |
| `postman/trello.postman_collection.json` | API requests and assertions |
| `postman/trello.postman_environment.json` | Environment template |
| `README.md` | English documentation |
| `README.ko.md` | 한국어 문서 |

## Getting Started

### Prerequisites

- Node.js and npm
- Trello account, a dedicated test workspace, API Key, and Token

### Installation

```bash
git clone https://github.com/siyeon98/trello-api-automation.git
cd trello-api-automation
npm install -g newman
```

### Environment Configuration

Copy the environment template and enter your API credentials.

```bash
cp postman/trello.postman_environment.json postman/trello.local.postman_environment.json
```

| Variable | Value |
| --- | --- |
| `base_url` | `https://api.trello.com` |
| `key` | Trello API Key |
| `token` | Trello API Token |

The collection creates, modifies, and deletes real resources. Use a dedicated test workspace.

The local environment file is ignored by Git. Keep credentials out of version control.

## Running Tests

```bash
newman run postman/trello.postman_collection.json -e postman/trello.local.postman_environment.json
```

Run the collection in its defined order because later requests depend on resources created by earlier requests.

## Continuous Integration

GitHub Actions executes the Newman collection on a GitHub-hosted Ubuntu runner.

| Setting | Value |
| --- | --- |
| Workflow | Trello API Tests |
| Job | API Tests |
| Node.js | `22` |

### Execution Triggers

| Trigger | Configuration |
| --- | --- |
| Push | Pushes to `main` |
| Manual | GitHub Actions → Trello API Tests → Run workflow |
| Scheduled | Configured daily at 03:23 Asia/Seoul on the default branch |

Scheduled runs may start after the configured time.

A [scheduled run on November 17, 2026](https://github.com/siyeon98/trello-api-automation/actions/runs/43998951532) completed successfully.

### Workflow Steps

1. Check out the repository and set up Node.js.
2. Install Newman and generate a temporary environment file from GitHub Actions Secrets.
3. Run the collection and export JUnit XML results.
4. Check the report for configured API credential values before artifact upload.
5. Upload the report and send the execution summary to Slack.

### Test Reporting

| Setting | Value |
| --- | --- |
| Report Format | JUnit XML |
| Artifact Name | `newman-test-results` |
| Retention Period | 30 days |
| Upload Condition | Upload attempted when the report credential check succeeds |

Download generated and uploaded reports from the **Artifacts** section of the corresponding GitHub Actions run.

### Slack Test Notifications

Notifications use Slack Incoming Webhooks and Block Kit. They include the job result, execution trigger, and a link to the run.

When JUnit XML results are available, notifications include request counts, assertion pass/fail/skip counts, duration when reported, and up to 10 failed assertions. The notification step uses `always()`.

If report data is unavailable, the notification includes the job result and run link. Slack delivery failures are recorded as warnings in GitHub Actions and do not fail the job. Test failures still fail the job.

Successful Slack delivery has been verified, including scheduled executions.

## Known Limitations

- **Test isolation** — requests rely on ordered execution and shared resource IDs.
- **Response times** — the 2,000 ms threshold is project-defined. Trello has not been established to provide this threshold as an SLA. Network latency affects results.
- **Coverage** — additional negative and boundary scenarios remain possible extensions.
- **Cleanup** — interrupted or partially failed runs can leave test resources requiring inspection and cleanup.

## Security

- Local environment files are ignored by Git.
- CI credentials and the Slack webhook are stored in GitHub Actions Secrets.
- CI uses `TRELLO_API_KEY`, `TRELLO_API_TOKEN`, and `SLACK_WEBHOOK_URL`.
- The CI environment file is generated in the runner temporary directory.
- The JUnit XML artifact is checked for configured API Key and Token values before upload. This check detects those specific values and has limited coverage.
- Slack messages contain summaries. Generated reports are excluded from version control.
