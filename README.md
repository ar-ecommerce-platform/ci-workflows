# 🛠️ CI/CD Workflows

This repository centralizes **shared CI/CD workflows** and **composite actions** for all services in the platform.

---

## 📂 Project Structure

```text
ci-workflows/
├── .github/
│   ├── actions/        # Composite actions (e.g., wait-for-health)
│   └── workflows/      # All shared CI/CD workflow YAMLs
├── .gitignore
└── README.md
```

---

## 📋 Available Workflows

| Workflow              | Purpose                                                                                  |
|-----------------------|------------------------------------------------------------------------------------------|
| **CI Template**       | Template workflow that all services will call                                            |
| **Setup**             | Builds the service artifact, pushes a test (`:ci`) Docker image, and verifies it starts with a smoke test. |
| **Code Quality**      | Runs Spotless and Checkstyle to enforce code style & conventions.                        |
| **Security Scan**     | Runs OWASP Dependency Check and Snyk container scan.                                     |
| **Unit Tests**        | Runs unit tests and uploads coverage reports as artifacts.                               |
| **Integration Tests** | Spins up services + runs integration tests.                                              |
| **Quality Scan**      | SonarCloud analysis for bugs, smells, coverage.                                          |
| **Publish**           | Auto-bumps version, tags commit, retags image for release.                               |
| **Notify Slack**      | Sends CI summary to Slack.                                                               |

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
      OWASP_API_KEY: ${{ secrets.OWASP_API_KEY }} # Used for OWASP Dependency Check
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }} # Required for SonarCloud analysis
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # Default token for creating tags/releases
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
    - Optional steps (integration tests, Code Quality) allow flexible workflows.


