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

| Workflow              | Purpose                                                             |
|-----------------------|---------------------------------------------------------------------|
| ci-template.yml       | Template workflow that all services will call                       |
| **Setup**             | Builds the service artifact and pushes a test (`:ci`) Docker image. |
| **Code Quality**      | Runs Spotless and Checkstyle to enforce code style & conventions.   |
| **Security Scan**     | Runs OWASP Dependency Check and Snyk container scan.                |
| **Unit Tests**        | Runs unit tests and uploads coverage reports as artifacts.          |
| **Integration Tests** | Spins up services + runs integration tests.                         |
| **Quality Scan**      | SonarCloud analysis for bugs, smells, coverage.                     |
| **Publish**           | Auto-bumps version, tags commit, retags image for release.          |
| **Notify Slack**      | Sends CI summary to Slack.                                          |

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
- **Modular Workflows** – Each workflow is independent; services can opt in to the ones they need 
(e.g., run security scans but skip integration tests).
- **Versioning & Publish**
    - Uses **Conventional Commits** for auto-bumping:
        - `feat:` → minor
        - `fix:` → patch
        - `BREAKING CHANGE:` → major
    - Updates `version.txt`, tags the commit, and Docker image.
    - **Triggers only on merges to `main`.**
- **Fail Fast** – Workflows stop early if setup, linting, or tests fail to avoid wasting resources.
- **Caching** – Gradle dependencies are cached to speed up builds.
- **Reproducible Builds** – Docker images are built the same way every time for consistency.
- **Security Scans** – OWASP Dependency Check + Snyk run on every build to catch vulnerabilities.
- **Notifications** – Slack workflow provides instant feedback to developers after CI runs (success/failure).
- **Observability** – Logs, reports, and Slack notifications provide full traceability.


