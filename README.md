# Trello API Test Automation

REST API test automation project using **Postman, JavaScript, Newman, and GitHub Actions**.

This project validates the lifecycle of Trello Boards, Lists, and Cards through a collection of automated API requests and assertions.

## Tech Stack

- **Postman** — API requests, test scripts, and environment management
- **JavaScript** — Assertions, dynamic test data, and variable handling
- **Newman** — Command-line execution of Postman collections
- **GitHub Actions** — Automated CI execution
- **Git / GitHub** — Version control

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
│   ├── trello.postman_environment.json
│   └── trello.local.postman_environment.json  # Local only, Git-ignored
├── .gitignore
└── README.md
```

The local Postman environment file contains credentials and is excluded from Git. It is not included in the repository.

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

The final local Newman validation completed successfully:

| Metric | Result |
|---|---|
| Iterations | 1 |
| Requests | 18 executed, 0 failed |
| Test scripts | 36 executed, 0 failed |
| Pre-request scripts | 27 executed, 0 failed |
| Assertions | **77 passed, 0 failed** |
| Total duration | 6.5 seconds |
| Average response time | 342 ms |

These results represent a verified local run. Actual execution times may vary.

## Continuous Integration

GitHub Actions automatically runs the Newman test collection when changes are pushed to the **`main` branch**.

The workflow also supports manual execution through **Actions → Trello API Tests → Run workflow**.

The CI workflow:

1. Checks out the repository.
2. Sets up Node.js.
3. Installs Newman.
4. Loads API credentials from GitHub Actions Secrets.
5. Generates a temporary Postman environment file.
6. Runs the automated API collection.

The workflow uses two repository secrets:

- `TRELLO_API_KEY`
- `TRELLO_API_TOKEN`

**CI status:** A GitHub Actions workflow run has been verified as successful.

[View GitHub Actions runs](https://github.com/siyeon98/trello-api-automation/actions)

## Security

- API credentials are not stored in the committed Postman collection.
- The local environment file is excluded using `.gitignore`.
- CI credentials are supplied through GitHub Actions Secrets.
- Generated test reports are excluded from version control.

## Current Scope

This project demonstrates API workflow testing, response validation, test-data handling, command-line automation, and GitHub Actions integration.

The current implementation uses an ordered Postman collection rather than independently isolated API tests. Additional test isolation and broader negative-case coverage may be considered in future iterations.