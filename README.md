# Trello API Test Automation

A REST API test automation project using **Postman, JavaScript, Newman, and GitHub Actions**, with scheduled execution, JUnit XML reporting, and Slack notifications.

This project validates the lifecycle of Trello Boards, Lists, and Cards through automated API requests and assertions. It demonstrates API workflow testing, dynamic test data management, CI integration, test reporting, and execution monitoring.

## Tech Stack

- **Postman** — API requests, test scripts, and environment management
- **JavaScript** — Assertions, dynamic test data, and variable handling
- **Newman** — Command-line execution of Postman collections
- **GitHub Actions** — CI, manual execution, and scheduled testing
- **JUnit XML** — Structured test execution reports
- **Python** — Report parsing, credential checks, and Slack message generation
- **Slack Incoming Webhooks / Block Kit** — Test result notifications
- **Git / GitHub** — Version control and repository management

## Test Coverage

The Postman collection contains **18 API requests** covering the following workflows:

| Resource | Test Scenarios |
|---|---|
| Board | Create, retrieve, update, verify update, delete, verify deletion |
| List | Create, retrieve, update, verify update, archive, verify archived state |
| Card | Create, retrieve, update, verify update, delete, verify deletion |

Assertions verify:

- HTTP status codes and response data
- Resource IDs and names
- Board–List and List–Card relationships
- Updated resource values
- Archived list status (`closed: true`)
- HTTP 404 responses for deleted resources
- Response times below 2,000 ms

Pre-request scripts generate dynamic resource names and clear previously stored resource IDs before creation requests.

Collection variables pass generated IDs between requests, allowing the full workflow to run sequentially.

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

The local environment file `postman/trello.local.postman_environment.json` is created during setup and excluded from Git.

JUnit XML reports are generated in the GitHub Actions runner's temporary directory and uploaded as workflow artifacts rather than committed to the repository.

## Getting Started

### 1. Prerequisites

- Node.js and npm
- Newman
- Trello API Key and Token
- A Trello account with access to a test workspace

Install Newman:

```bash
npm install -g newman
```

### 2. Clone the Repository

```bash
git clone https://github.com/siyeon98/trello-api-automation.git
cd trello-api-automation
```

The repository uses `main` as its default branch.

### 3. Configure the Environment

Create a local copy of the environment template:

```bash
cp postman/trello.postman_environment.json postman/trello.local.postman_environment.json
```

Edit the local environment file and configure these variables:

| Variable | Value |
|---|---|
| `base_url` | `https://api.trello.com` |
| `key` | Your Trello API Key |
| `token` | Your Trello API Token |

Never commit the local environment file or expose API credentials.

### 4. Run Tests Locally

```bash
newman run postman/trello.postman_collection.json -e postman/trello.local.postman_environment.json
```

The collection should be executed in its defined order because subsequent requests depend on resources created by earlier requests.

**Note:** The collection creates, modifies, and deletes real Trello resources. Use a dedicated test workspace rather than production data.

## Test Execution Results

A verified local Newman run completed successfully:

| Metric | Result |
|---|---|
| Iterations | 1 |
| Requests | 18 executed, 0 failed |
| Test scripts | 36 executed, 0 failed |
| Pre-request scripts | 27 executed, 0 failed |
| Assertions | **77 passed, 0 failed** |
| Total duration | 6.5 seconds |
| Average response time | 342 ms |

These results represent a verified local run. Actual execution times and response times may vary.

GitHub Actions executions have also completed successfully, including scheduled runs and Slack notification delivery.

## GitHub Actions Automation

GitHub Actions executes the Newman collection on a GitHub-hosted Ubuntu runner.

### Execution Triggers

| Trigger | Configuration |
|---|---|
| Push | Automatically runs when changes are pushed to `main` |
| Manual | Runs through Actions → Trello API Tests → Run workflow |
| Scheduled | Configured to run daily at 23:17 Asia/Seoul |

The workflow:

1. Checks out the repository.
2. Sets up Node.js 22.
3. Installs Newman.
4. Loads Trello credentials from GitHub Actions Secrets.
5. Generates a temporary Postman environment file.
6. Executes the API test collection.
7. Exports JUnit XML test results.
8. Checks the report for configured API credential values.
9. Uploads the report as an artifact if the credential check passes.
10. Sends a Slack notification with the execution result.

[View GitHub Actions runs](https://github.com/siyeon98/trello-api-automation/actions)

### Scheduled Test Execution

The workflow is configured to run daily at **23:17 Asia/Seoul** using GitHub Actions scheduled triggers.

Scheduled execution was verified through **two successful workflow runs**, both of which delivered Slack notifications.

During validation with a temporary 15-minute schedule, execution did not consistently occur at the expected intervals. Two successful scheduled runs were observed approximately four hours apart.

GitHub Actions scheduled workflows may be delayed or skipped, so exact execution timing is not guaranteed.

The daily schedule was restored after testing. Successful execution at the restored daily schedule has not yet been verified separately.

## Test Reporting

Newman exports test execution results in **JUnit XML** format.

The generated report is named:

`newman-results.xml`

The workflow uploads the report to GitHub Actions as an artifact named:

`newman-test-results`

Artifacts are configured with a **7-day retention period**.

The JUnit XML report contains structured test information, including:

- API request groups
- Assertion names and results
- Failed assertions and failure details
- Execution duration

Reports can be downloaded from individual GitHub Actions runs.

The workflow reuses the JUnit XML report to generate Slack notification summaries. An additional Newman JSON report is not required.

## Slack Test Notifications

The project integrates GitHub Actions with Slack through **Incoming Webhooks** and **Block Kit**.

Notifications are configured for Push, manual, and scheduled workflow executions.

The notification includes:

- Job result: passed or failed
- Execution trigger
- Number of executed API requests
- Assertion pass, fail, and skip counts
- Test execution duration
- Failed assertion names, up to 10 entries
- A button linking to the corresponding GitHub Actions run

The message uses a structured Block Kit layout rather than plain-text output.

Successful test execution and Slack Block Kit notification delivery have been verified. Slack notifications were also received from two scheduled executions.

The notification reports the GitHub Actions job status and reads test statistics from the JUnit XML file when available. If the report cannot be read, the notification provides the execution status and a link to the workflow logs.

The notification is sent after the preceding steps finish or fail, provided the notification step itself can execute and Slack is reachable. Workflow cancellation or integration failures may prevent delivery.

## Security

The project implements the following safeguards:

- Trello API credentials are not stored in the committed Postman collection.
- The local environment file is excluded through `.gitignore`.
- CI credentials are supplied through GitHub Actions Secrets.
- The Slack Incoming Webhook URL is stored as a GitHub Actions Secret.
- Temporary CI environment files are generated in the runner rather than committed to Git.
- Before uploading a JUnit XML artifact, the workflow checks for the configured Trello API Key and Token values.
- Slack messages contain test summaries rather than raw API request and response bodies.
- Generated reports are excluded from version control.

The report credential check is a targeted safeguard, not a comprehensive secret scanner.

## Current Scope and Limitations

This project demonstrates:

- REST API workflow automation
- Response validation and assertion design
- Dynamic test data and variable management
- Command-line test execution with Newman
- GitHub Actions CI and scheduled execution
- JUnit XML reporting and artifact retention
- Slack-based execution monitoring
- Basic credential handling in a public repository

The implementation uses an **ordered Postman collection** rather than independently isolated API tests.

The 2,000 ms response-time assertion is a project-defined test threshold, not a Trello service-level agreement. Network or service latency may cause intermittent failures.

Scheduled execution has been demonstrated, but precise timing and consistent execution frequency have not been established.

Potential future improvements include:

- Improving test isolation and cleanup reliability
- Expanding negative and boundary test coverage
- Investigating recurring response-time failures
- Monitoring missing scheduled executions

This repository is a QA automation portfolio project focused on practical API testing, CI integration, and test result visibility.