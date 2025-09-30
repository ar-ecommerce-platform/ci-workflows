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

| Workflow               | Purpose                                                                                                    |
|------------------------|------------------------------------------------------------------------------------------------------------|
| **CI Template**        | Template workflow that all services will use.                                                              |
| **Setup**              | Builds the service artifact, pushes a test (`:ci`) Docker image, and verifies it starts with a smoke test. |
| **Code Quality**       | Runs Spotless and Checkstyle to enforce code style & conventions.                                          |
| **Security Scan**      | Runs OWASP Dependency Check and Snyk container scan.                                                       |
| **Unit Tests**         | Runs unit tests and uploads coverage reports as artifacts.                                                 |
| **Integration Tests**  | Spins up services + runs integration tests.                                                                |
| **Quality Scan**       | SonarCloud analysis for bugs, smells, coverage.                                                            |
| **Check Dependencies** | Verifies all required jobs passed/skipped before continuing.                                               |
| **Docker Release**     | Auto-bumps version, tags commit, retags image for release.                                                 |
| **Notify Slack**       | Sends CI summary to Slack.                                                                                 |

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
  setup:
    uses: ar-ecommerce-platform/ci-workflows/.github/workflows/ci-template.yml@main
    with:
      service_name: config-server
      run_integration_tests: false # Skip integration tests if not needed
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}           # Default GitHub token (tags, releases)
      REPORTS_TRIGGER_TOKEN: ${{ secrets.REPORTS_TRIGGER_TOKEN }} # Triggers reporting workflow
      OWASP_API_KEY: ${{ secrets.OWASP_API_KEY }}         # OWASP Dependency Check API
      OSSINDEX_USERNAME: ${{ secrets.OSSINDEX_USERNAME }} # OSS Index auth (user)
      OSSINDEX_TOKEN: ${{ secrets.OSSINDEX_TOKEN }}       # OSS Index auth (token)
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}             # SonarCloud analysis
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}               # Snyk container scanning
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }} # Slack notifications
```

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
   - Uploads coverage reports and artifacts.
   - Slack notifications provide CI/CD status updates.
   - Logs and reports give traceability for all workflow steps.
6. **Scalability**
    - Works for multiple microservices in the platform.
    - New services can plug in easily with the same template.
    - Optional steps allow flexible workflows.


