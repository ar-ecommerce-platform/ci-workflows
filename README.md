# 🛠️ CI/CD Workflows

This repository centralizes **shared CI/CD workflows** and **composite actions** for all services in the platform.

---

## 📂 Project Structure

```text
ci-workflows/
├── .github/
│   ├── actions/        # Composite actions (e.g., wait-for-health)
│   └── workflows/      # All shared CI/CD workflow YAMLs
├── gradle/
│   ├── checkstyle/               # Checkstyle config (XML rules)
│   │   └── checkstyle.xml
│   └── build-quality.gradle      # Shared Gradle config (Spotless, Checkstyle, OWASP, Snyk, JaCoCo)
├── .gitignore
└── README.md
```

---

## 📋 Available Workflows

| Workflow               | Purpose                                                                                                             |
|------------------------|---------------------------------------------------------------------------------------------------------------------|
| **CI Template**        | Template workflow that all services will use.                                                                       |
| **Setup**              | Builds the service artifact, pushes a test (`:ci`) Docker image, and verifies it starts with a smoke test.          |
| **Code Quality**       | Runs Spotless and Checkstyle to enforce code style & conventions.                                                   |
| **Security Scan**      | Runs OWASP Dependency Check and Snyk container scan.                                                                |
| **Unit Tests**         | Runs unit tests and uploads coverage reports as artifacts.                                                          |
| **Integration Tests**  | Spins up services + runs integration tests.                                                                         |
| **Quality Scan**       | SonarCloud analysis for bugs, smells, coverage.                                                                     |
| **Check Dependencies** | Verifies all required jobs passed/skipped before continuing.                                                        |
| **Test Reports**       | Collects and uploads all CI reports (unit tests, integration tests, security scans, coverage) to test reports repo. |
| **Docker Release**     | Auto-bumps version, tags commit, retags image for release.                                                          |
| **Notify Slack**       | Sends CI summary to Slack.                                                                                          |

---

## 🧩 Composite Actions

- **wait-for-health**  
  Polls container health status up to 10 times and fails if it never becomes healthy.  
  Useful for databases, config server, API gateways, or other services that need to be ready before tests run.

---

## 🚀 Usage
All services(e.g., auth, user, product) will use `ci-template.yml`. Example:

```yaml
jobs:
  call-ci-template:
    uses: ar-ecommerce-platform/ci-workflows/.github/workflows/ci-template.yml@main
    with:
      run_integration_tests: false # Input to skip integration tests
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}           # Default GitHub token (tags, releases)
      PAT_REPORTS_TOKEN: ${{ secrets.PAT_REPORTS_TOKEN }} # Token for pushing reports
      OWASP_API_KEY: ${{ secrets.OWASP_API_KEY }}         # OWASP Dependency Check API
      OSSINDEX_USERNAME: ${{ secrets.OSSINDEX_USERNAME }} # OSS Index auth (user)
      OSSINDEX_TOKEN: ${{ secrets.OSSINDEX_TOKEN }}       # OSS Index auth (token)
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}             # SonarCloud analysis
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}               # Snyk container scanning
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }} # Slack notifications
```

### 🔑 Secrets

| Secret                | Required        | Used In                          | Condition (Input Flag)         |
|------------------------|-----------------|----------------------------------|--------------------------------|
| `GITHUB_TOKEN`         | ✅ Always       | Tagging commits, pushing releases | Always provided by GitHub      |
| `PAT_REPORTS_TOKEN`| ✅ Always       |    Token for pushing reports    | Always                         |
| `SLACK_WEBHOOK_URL`    | ✅ Always       | Slack notifications               | Always                         |
| `SONAR_TOKEN`          | ⚠️ Conditional | SonarCloud analysis               | `run_quality_scan: true`       |
| `OWASP_API_KEY`        | ⚠️ Conditional | OWASP Dependency Check (NVD feed) | `run_security_scan: true`      |
| `OSSINDEX_USERNAME`    | ⚠️ Conditional | OSS Index vulnerability scan      | `run_security_scan: true`      |
| `OSSINDEX_TOKEN`       | ⚠️ Conditional | OSS Index vulnerability scan      | `run_security_scan: true`      |
| `SNYK_TOKEN`           | ⚠️ Conditional | Snyk container scan               | `run_security_scan: true`      |

### ⚙️ Inputs
These flags let each service control which parts of the CI pipeline should run.  
By default, all are enabled (`true`). You can override them when calling the template.

| Flag                   | Type    | Default | Description                                                    | Requires Secrets                |
|-------------------------|---------|---------|----------------------------------------------------------------|---------------------------------|
| `enable_database`       | boolean | true    | Start a database for integration tests.                        | —                               |
| `run_code_quality`      | boolean | true    | Run code quality checks (Spotless, Checkstyle).                | —                               |
| `run_unit_tests`        | boolean | true    | Run unit tests with coverage reports.                          | —                               |
| `run_integration_tests` | boolean | true    | Run integration tests.                                         | —                               |
| `run_quality_scan`      | boolean | true    | Run SonarCloud analysis (bugs, smells, coverage).              | `SONAR_TOKEN`                   |
| `run_security_scan`     | boolean | true    | Run security scans (OWASP Dependency Check + OSS Index + Snyk).| `OWASP_API_KEY`, `OSSINDEX_USERNAME`, `OSSINDEX_TOKEN`, `SNYK_TOKEN` |
---

## 📌 Notes
1. **Modularity & Reusability**
   - Each workflow (Setup, Code Quality, Unit Tests, etc.) is independent.
   - Services opt in/out easily via ci-template.yml.
   - Composite actions like wait-for-health can be reused anywhere.
2. **Fail Fast Principle**
   - Smoke test immediately after building the Docker image prevents wasting resources.
   - Gradle caching and skipping unnecessary steps make CI efficient.
3. **Security & Quality**
   - OWASP & Snyk scans ensure security compliance.
   - Spotless / Checkstyle enforce coding standards.
   - SonarCloud tracks code quality and coverage.
4. **Versioning & Release Automation**
   - Uses Conventional Commits to auto-bump versions.
   - Publish workflow updates version.txt, tags commits, and tags Docker images.
   - Only triggers on merges to main.
5. **Observability & Reporting**
   - Gives developers quick feedback on CI/CD runs.
   - Uploads coverage reports and artifacts. 
   - Sends Slack messages with workflow status. 
   - Maintains logs for traceability of workflow steps.
6. **Compliance & Test Tracking**
   - Centralizes test results in the test-reports repository.
   - Tracks unit, integration, and security tests across services.
   - Helps with audits and keeps a record for compliance purposes.
7. **Scalability**
   - Works for multiple microservices in the platform.
   - New services can plug in easily with the same template.
   - Optional steps allow flexible workflows.



