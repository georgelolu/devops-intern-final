cat > README.md <<'EOF'
# DevOps Intern Final Assessment

![DevOps CI](https://github.com/georgelolu/devops-intern-final/actions/workflows/ci.yml/badge.svg)

## Student

**Name:** George Omololu Akinbi  
**Date:** August 14, 2026

---

## Project Description

This project demonstrates a small but realistic DevOps workflow using open-source tools and practices.

The project covers:

- Git and GitHub
- Linux and shell scripting
- Python
- Docker
- GitHub Actions CI/CD
- HashiCorp Nomad
- Grafana Loki
- Grafana Promtail
- Log collection and querying
- DevOps documentation

Each stage produces an output that is used by the next stage.

---

# Project Architecture

```text
                    GitHub Repository
                           |
                           v
                    GitHub Actions
                           |
                    python hello.py
                           |
                           v
                     Docker Image
                           |
                           v
                    HashiCorp Nomad
                           |
                           v
                 hello-devops Container
                           |
                           | Docker Logs
                           v
                       Promtail
                           |
                           v
                         Loki
                           |
                           v
                    Log Query / Search
# CI/CD Quality-Control Pipeline with GitHub Actions and SonarQube

## Overview

This section documents the complete implementation and deployment of the CI/CD quality-control pipeline for the **`devops-intern-final`** project.

The objective was to extend the existing DevOps project with an automated Continuous Integration and Code Quality workflow using:

* GitHub
* GitHub Actions
* Self-hosted GitHub Actions Runner
* Linux/WSL
* Python
* SonarQube
* SonarQube Quality Gate
* GitHub Secrets
* YAML-based CI/CD automation

The final implementation automatically validates the Python application, performs SonarQube static code analysis, and verifies that the project satisfies the configured SonarQube Quality Gate.

The repository is available at:

`https://github.com/georgelolu/devops-intern-final`

The repository already contains the main application, Docker configuration, Nomad configuration, monitoring components, scripts, and SonarQube configuration. The CI/CD quality-control workflow was added as an additional automation layer.

---

# 1. Project Objective

The original project was designed to demonstrate a complete DevOps workflow covering:

1. Git and GitHub
2. Linux scripting
3. Docker containerization
4. GitHub Actions CI/CD
5. HashiCorp Nomad deployment
6. Grafana Loki monitoring
7. Technical documentation

The repository contains files and directories including:

```text
devops-intern-final/
├── .github/
│   └── workflows/
├── monitoring/
├── nomad/
├── scripts/
├── .gitignore
├── Dockerfile
├── README.md
├── hello.py
└── sonar-project.properties
```

The CI/CD enhancement added automated code validation and SonarQube quality analysis to the existing project.

---

# 2. Target CI/CD Architecture

The final architecture was designed around a self-hosted GitHub Actions runner because SonarQube was running locally on the same WSL/Linux environment.

```text
                         Developer
                             │
                             │ git push
                             ▼
                    ┌─────────────────┐
                    │ GitHub Repository│
                    │ devops-intern-   │
                    │ final            │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ GitHub Actions   │
                    │ CI Workflow      │
                    └────────┬────────┘
                             │
                             ▼
                 ┌─────────────────────────┐
                 │ Self-Hosted Runner      │
                 │ Ubuntu/WSL              │
                 │                         │
                 │ GitHub Runner 2.336.0   │
                 └───────────┬─────────────┘
                             │
             ┌───────────────┼────────────────┐
             │               │                │
             ▼               ▼                ▼
       Checkout Code    Python Validation   SonarQube
             │               │                │
             │               │                ▼
             │               │         Static Code Analysis
             │               │                │
             │               │                ▼
             │               │          Quality Gate
             │               │                │
             └───────────────┴────────────────┤
                                              ▼
                                     PASS / FAIL
```

The important architectural decision was to run the workflow on the **self-hosted runner** because the SonarQube server was accessible at:

```text
http://localhost:9000
```

`localhost` refers to the machine executing the workflow. Therefore, a GitHub-hosted runner would not be able to reach a SonarQube server running locally on the WSL machine.

GitHub supports routing workflows to self-hosted runners using `runs-on: self-hosted` and runner labels.

---

# 3. Prerequisites

Before implementing the CI/CD pipeline, the following components were required.

## 3.1 GitHub Repository

The project repository:

```text
https://github.com/georgelolu/devops-intern-final
```

The repository contains the application and DevOps configuration required for the project.

## 3.2 Linux/WSL Environment

The self-hosted runner was installed inside WSL.

The runner machine used:

```text
Architecture: x86_64
CPU: Intel(R) Core(TM) i5-3320M CPU @ 2.60GHz
```

## 3.3 Python

System Python was available at:

```text
/usr/bin/python3
```

The installed version was:

```text
Python 3.14.4
```

## 3.4 GitHub Actions Runner

The self-hosted runner used:

```text
Runner version: 2.336.0
```

## 3.5 SonarQube

SonarQube was installed and made available locally at:

```text
http://localhost:9000
```

## 3.6 Git

Git was configured so changes to the workflow could be committed and pushed to GitHub.

---

# 4. Verify the Existing Repository

The first step was to clone or access the project locally.

```bash
git clone https://github.com/georgelolu/devops-intern-final.git
cd devops-intern-final
```

Verify the repository:

```bash
git status
```

Check the project files:

```bash
ls -la
```

The repository contained:

```text
.github/
monitoring/
nomad/
scripts/
Dockerfile
README.md
hello.py
sonar-project.properties
```

---

# 5. Verify the Application

The application entry point was:

```text
hello.py
```

Before introducing CI/CD, the application was tested locally:

```bash
python3 hello.py
```

The purpose of this test was to ensure that the application could execute successfully before automating the validation process.

---

# 6. Install and Configure the Self-Hosted GitHub Actions Runner

A self-hosted runner was selected because the SonarQube server was running locally on the same WSL machine.

GitHub's self-hosted runner model allows workflows to execute on infrastructure controlled by the project owner rather than GitHub-hosted infrastructure. The runner must be online and able to communicate with GitHub Actions.

The runner was installed under:

```text
~/actions-runner
```

The directory contained:

```text
actions-runner-linux-x64-2.336.0.tar.gz
bin/
externals/
config.sh
run.sh
svc.sh
```

After configuration, additional runner files appeared:

```text
.credentials
.credentials_rsaparams
.runner
_work/
_diag/
```

These files confirmed that the runner had been successfully registered.

---

# 7. Start the Self-Hosted Runner

The runner was started with:

```bash
cd ~/actions-runner
./run.sh
```

Successful startup produced:

```text
Connected to GitHub

Current runner version: '2.336.0'
Listening for Jobs
```

When a GitHub Actions workflow was triggered, the runner reported:

```text
Running job: test
```

This confirmed that GitHub was successfully dispatching workflow jobs to the local machine.

The runner must remain online to receive jobs. GitHub documents that a self-hosted runner must be running and connected to GitHub Actions to accept assignments.

---

# 8. Initial CI Workflow

The initial CI workflow was created under:

```text
.github/workflows/ci.yml
```

The initial implementation used:

```yaml
- name: Set up Python
  uses: actions/setup-python@v5
  with:
    python-version: "3.13"
```

The workflow then executed:

```yaml
- name: Run Hello DevOps
  run: python hello.py
```

The purpose was to:

1. Check out the repository.
2. Install/configure Python.
3. Execute the Python application.
4. Run SonarQube analysis.

---

# 9. Initial Pipeline Failure

The first execution failed.

The GitHub Actions runner reported:

```text
./python: CPU ISA level is lower than required
```

followed by:

```text
The process '/usr/bin/bash' failed with exit code 127
```

The job therefore completed with:

```text
Job test completed with result: Failed
```

The runner itself was not disconnected.

It successfully showed:

```text
Connected to GitHub
Listening for Jobs
Running job: test
```

Therefore, the failure was inside the workflow execution rather than the GitHub runner registration.

---

# 10. Diagnose the Python/CPU Compatibility Problem

The runner machine was inspected.

## Check Python

```bash
which python3
```

Result:

```text
/usr/bin/python3
```

Check version:

```bash
python3 --version
```

Result:

```text
Python 3.14.4
```

Check architecture:

```bash
uname -m
```

Result:

```text
x86_64
```

Check CPU:

```bash
lscpu | grep -E 'Architecture|Model name|Flags'
```

The machine was running:

```text
Intel(R) Core(TM) i5-3320M CPU @ 2.60GHz
```

The CPU supports AVX and several other instruction sets but does not support newer instruction sets such as AVX2.

---

# 11. Identify the Incompatible Python Runtime

The runner workspace was inspected:

```bash
find _work -type f -name python -o -name python3 2>/dev/null | head -30
```

This revealed:

```text
_work/_tool/Python/3.13.15/x64/bin/python3
```

This was the critical discovery.

The system Python:

```text
/usr/bin/python3
```

worked correctly.

However, `actions/setup-python` had installed a separate Python runtime under the runner's tool cache.

The GitHub `setup-python` action supports self-hosted runners and uses a tool cache/download mechanism to provide requested Python distributions.

On this particular older CPU, the downloaded Python executable produced the CPU ISA error.

---

# 12. Solution: Use the System Python

Instead of forcing the self-hosted runner to download Python 3.13, the workflow was changed to use the already-installed system Python.

The following step was removed:

```yaml
- name: Set up Python
  uses: actions/setup-python@v5
  with:
    python-version: "3.13"
```

It was replaced with:

```yaml
- name: Check Python
  run: |
    which python3
    python3 --version
```

The application execution was changed from:

```yaml
run: python hello.py
```

to:

```yaml
run: python3 hello.py
```

This ensured that the workflow used:

```text
/usr/bin/python3
```

instead of the incompatible downloaded Python runtime.

---

# 13. Validate the Python Fix

The workflow was committed with:

```bash
git add .github/workflows/ci.yml
git commit -m "fix: use system Python on self-hosted runner"
git push origin main
```

GitHub Actions automatically detected the new commit and dispatched another job to the self-hosted runner.

The runner successfully processed the job.

The commit:

```text
fix: use system Python on self-hosted runner
```

resolved the CPU ISA failure.

The pipeline was now able to execute the Python application successfully.

---

# 14. Initial SonarQube Workflow Problem

The project originally had two workflow files:

```text
.github/workflows/ci.yml
.github/workflows/sonarqube.yml
```

The second workflow used:

```yaml
runs-on: ubuntu-latest
```

while attempting to connect to:

```text
http://localhost:9000
```

This was not appropriate for the current architecture.

A GitHub-hosted runner is a separate machine. Therefore:

```text
localhost:9000
```

on that runner would point to the temporary GitHub-hosted runner itself, not the WSL machine running SonarQube.

---

# 15. Consolidate the SonarQube Workflow

To eliminate duplicate workflows and ensure SonarQube was accessed from the correct machine, the separate:

```text
.github/workflows/sonarqube.yml
```

workflow was removed.

The pipeline was consolidated into:

```text
.github/workflows/ci.yml
```

The final workflow became the single source of truth for CI and SonarQube analysis.

---

# 16. Final CI/CD Workflow

The final workflow used:

```yaml
name: DevOps CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:
  test:
    name: Test and SonarQube Analysis
    runs-on: self-hosted

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Check Python
        run: |
          which python3
          python3 --version

      - name: Compile Python files
        run: |
          python3 -m compileall .

      - name: Run Hello DevOps
        run: |
          python3 hello.py

      - name: Check SonarQube availability
        run: |
          curl --fail --silent --show-error http://localhost:9000/api/system/status
          echo "SonarQube is reachable"

      - name: SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@v7
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: http://localhost:9000
```

The workflow now runs on the self-hosted runner and therefore has access to the local SonarQube server.

GitHub recommends using runner labels when more specific targeting is required; `self-hosted` is the standard label automatically applied to a normal self-hosted runner.

---

# 17. Python Validation Stage

The workflow validates the Python runtime:

```yaml
- name: Check Python
  run: |
    which python3
    python3 --version
```

Expected output:

```text
/usr/bin/python3
Python 3.14.4
```

This provides visibility into which interpreter the self-hosted runner is using.

---

# 18. Python Compilation Test

The workflow performs a syntax/compilation check:

```yaml
- name: Compile Python files
  run: |
    python3 -m compileall .
```

This helps identify Python syntax or compilation problems before SonarQube analysis.

A failure at this stage stops the workflow.

---

# 19. Application Test

The application is executed using:

```yaml
- name: Run Hello DevOps
  run: |
    python3 hello.py
```

This verifies that the project's Python application can actually execute on the runner.

If the command returns a non-zero exit code, GitHub Actions marks the job as failed.

---

# 20. SonarQube Availability Check

Before starting static analysis, the workflow checks whether SonarQube is reachable:

```yaml
- name: Check SonarQube availability
  run: |
    curl --fail --silent --show-error http://localhost:9000/api/system/status
    echo "SonarQube is reachable"
```

This provides an early and clear failure point if SonarQube is unavailable.

A healthy SonarQube instance should return a successful response from:

```text
http://localhost:9000/api/system/status
```

---

# 21. Configure SonarQube Authentication

A SonarQube authentication token was created and stored in GitHub repository secrets.

The GitHub repository was configured with:

```text
SONAR_TOKEN
```

The token was deliberately not stored directly inside the workflow.

Instead, the workflow uses:

```yaml
SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

This prevents the authentication token from being hard-coded into the repository.

SonarSource's GitHub Actions documentation identifies `SONAR_TOKEN` as the required authentication secret and `SONAR_HOST_URL` as the server URL required for a self-hosted SonarQube Server deployment.

---

# 22. Configure SonarQube Server URL

Because SonarQube is running locally on the self-hosted runner, the workflow uses:

```yaml
SONAR_HOST_URL: http://localhost:9000
```

This works because:

```text
GitHub Actions
      ↓
Self-hosted WSL runner
      ↓
localhost:9000
      ↓
Local SonarQube Server
```

The architecture would be different if SonarQube were hosted remotely.

---

# 23. SonarQube Scan

The official SonarQube scan action is executed using:

```yaml
- name: SonarQube Scan
  uses: SonarSource/sonarqube-scan-action@v7
```

The action performs static analysis against the repository.

SonarQube analysis can identify issues such as:

* Bugs
* Vulnerabilities
* Code smells
* Security issues
* Maintainability issues
* Other code-quality problems

The official SonarSource action supports SonarQube Server and SonarQube Cloud analysis through GitHub Actions.

---

# 24. SonarQube Project Configuration

The repository also contains:

```text
sonar-project.properties
```

This file provides project-specific SonarQube analysis configuration.

The configuration should remain in the repository so that analysis is reproducible and version controlled.

Typical configuration includes properties such as:

```properties
sonar.projectKey=<project-key>
sonar.projectName=<project-name>
sonar.sources=.
```

The exact values should match the project configured in SonarQube.

---

# 25. Quality Gate

After SonarQube analyzes the repository, the results are evaluated against the configured SonarQube Quality Gate.

The Quality Gate determines whether the analyzed code meets the configured quality and security standards.

The final pipeline successfully produced:

```text
Quality Gate Passed
```

This confirms that the project met the configured SonarQube quality criteria at the time of analysis.

SonarSource also provides a dedicated Quality Gate action when a workflow needs to explicitly wait for and enforce the gate result after analysis.

---

# 26. Final Pipeline Execution

The final pipeline follows this sequence:

```text
1. Developer pushes code
             │
             ▼
2. GitHub detects push
             │
             ▼
3. GitHub Actions starts workflow
             │
             ▼
4. Job routed to self-hosted runner
             │
             ▼
5. Checkout repository
             │
             ▼
6. Verify system Python
             │
             ▼
7. Compile Python files
             │
             ▼
8. Execute hello.py
             │
             ▼
9. Check SonarQube availability
             │
             ▼
10. Run SonarQube analysis
             │
             ▼
11. Evaluate Quality Gate
             │
             ▼
       ┌─────┴─────┐
       │           │
     PASS         FAIL
       │           │
       ▼           ▼
   CI passes    CI fails
```

---

# 27. Git Commands Used During Implementation

The main Git commands used during implementation were:

### Check repository status

```bash
git status
```

### Stage workflow changes

```bash
git add .github/workflows/ci.yml
```

### Remove duplicate workflow

```bash
git rm .github/workflows/sonarqube.yml
```

### Commit changes

```bash
git commit -m "fix: use system Python on self-hosted runner"
```

and later:

```bash
git commit -m "ci: consolidate SonarQube workflow"
```

### Push to GitHub

```bash
git push origin main
```

Each push to `main` triggered the GitHub Actions workflow.

---

# 28. Troubleshooting Performed

## Problem 1: GitHub Actions Job Failed

### Error

```text
./python: CPU ISA level is lower than required
```

### Secondary error

```text
The process '/usr/bin/bash' failed with exit code 127
```

### Diagnosis

The self-hosted runner itself was healthy.

The problem was the Python distribution downloaded by `actions/setup-python`.

The downloaded runtime was located under:

```text
_work/_tool/Python/3.13.15/x64/
```

The local machine's CPU was an older Intel i5-3320M.

### Solution

Use the system Python:

```text
/usr/bin/python3
```

instead of the downloaded Python 3.13 runtime.

---

# 29. Troubleshooting Problem 2: Duplicate SonarQube Workflows

Two workflows were initially present:

```text
ci.yml
sonarqube.yml
```

Both could trigger on pushes to `main`.

The separate SonarQube workflow used:

```yaml
runs-on: ubuntu-latest
```

while trying to connect to:

```text
http://localhost:9000
```

### Solution

Remove the duplicate SonarQube workflow:

```bash
git rm .github/workflows/sonarqube.yml
```

and integrate SonarQube into:

```text
.github/workflows/ci.yml
```

using:

```yaml
runs-on: self-hosted
```

---

# 30. Final Repository Structure

After the cleanup, the relevant repository structure became:

```text
devops-intern-final/
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── monitoring/
│
├── nomad/
│
├── scripts/
│
├── .gitignore
├── Dockerfile
├── README.md
├── hello.py
└── sonar-project.properties
```

The duplicate:

```text
.github/workflows/sonarqube.yml
```

was removed.

---

# 31. Verification Checklist

The following checklist was used to confirm successful implementation:

* [x] GitHub repository accessible
* [x] Self-hosted runner installed
* [x] Self-hosted runner registered
* [x] Runner connected to GitHub
* [x] Runner listening for jobs
* [x] GitHub Actions successfully dispatched jobs
* [x] Python environment identified
* [x] CPU compatibility issue identified
* [x] Incompatible downloaded Python removed from execution path
* [x] System Python configured
* [x] Python application executed successfully
* [x] SonarQube server running
* [x] SonarQube reachable from runner
* [x] `SONAR_TOKEN` configured as GitHub Secret
* [x] SonarQube scan executed
* [x] Duplicate SonarQube workflow removed
* [x] CI workflow consolidated
* [x] SonarQube analysis completed
* [x] SonarQube Quality Gate passed

---

# 32. Final Result

The completed implementation transformed the project from a manually validated DevOps application into a project with an automated CI/CD quality-control process.

The final workflow provides:

```text
Source Control
      +
Automated Testing
      +
Python Validation
      +
Static Code Analysis
      +
SonarQube Quality Gate
      =
Automated CI Quality Control
```

The final successful result was:

```text
GitHub Actions
      ↓
Self-Hosted Runner
      ↓
Python Validation
      ↓
Application Execution
      ↓
SonarQube Analysis
      ↓
Quality Gate
      ↓
✅ PASSED
```

This demonstrates practical experience with:

* Continuous Integration
* GitHub Actions
* Self-hosted runners
* Linux/WSL administration
* Python automation
* YAML workflow configuration
* Static code analysis
* SonarQube
* Quality Gates
* GitHub Secrets
* CI/CD troubleshooting
* DevOps automation
* Code quality enforcement

---

# 33. Lessons Learned

### 1. Runner connectivity and workflow success are different

A runner can be successfully connected to GitHub while the workflow itself fails.

The following:

```text
Connected to GitHub
Listening for Jobs
```

only confirms runner connectivity.

The actual job logs must be inspected to determine whether the workflow succeeds.

### 2. Self-hosted runners inherit the limitations of their hardware

Although GitHub Actions supports x64 self-hosted runners, individual tools and prebuilt binaries may have CPU instruction-set requirements that exceed the capabilities of older hardware.

Therefore, CPU compatibility should be considered when selecting tools and runtimes.

### 3. Localhost is relative to the machine executing the job

When using:

```text
http://localhost:9000
```

the service must be available on the same machine where the workflow is executing.

This was the reason the SonarQube workflow was moved from a GitHub-hosted runner to the self-hosted runner.

### 4. Avoid duplicate workflows

Having multiple workflows triggered by the same event can result in duplicate builds, duplicate analysis, confusion in troubleshooting, and unnecessary resource consumption.

Consolidating the workflow made the pipeline easier to understand and maintain.

### 5. Secrets should never be hard-coded

The SonarQube token was stored in GitHub Secrets:

```text
SONAR_TOKEN
```

and referenced securely using:

```yaml
${{ secrets.SONAR_TOKEN }}
```

---

# 34. Recommended Future Improvements

The current pipeline provides a solid CI quality-control foundation. Future improvements can include:

### Security scanning

Add automated scanning for:

* Secrets
* Credentials
* API keys
* PII
* Vulnerable dependencies
* Container vulnerabilities

### PII protection

Add a CI stage that scans application logs and test output for unmasked PII and fails the build when unsafe patterns are detected.

Example:

```text
Code
 ↓
Tests
 ↓
PII Scan
 ↓
SonarQube
 ↓
Quality Gate
 ↓
Build/Deploy
```

### Container scanning

Add tools such as:

```text
Trivy
```

to scan the Docker image for vulnerabilities.

### Deployment automation

After the Quality Gate passes, automatically deploy the validated application using the existing Docker/Nomad infrastructure.

### Branch protection

Configure GitHub branch protection so that pull requests cannot be merged unless the required CI checks pass.

### Pull-request quality control

The workflow can be extended to analyze pull requests before they are merged into `main`.

---

# 35. Final DevOps Pipeline

The complete target architecture can therefore be represented as:

```text
                       ┌───────────────┐
                       │   Developer   │
                       └───────┬───────┘
                               │
                            git push
                               │
                               ▼
                       ┌───────────────┐
                       │    GitHub     │
                       │  Repository   │
                       └───────┬───────┘
                               │
                               ▼
                       ┌───────────────┐
                       │GitHub Actions │
                       └───────┬───────┘
                               │
                               ▼
                  ┌─────────────────────────┐
                  │ Self-Hosted WSL Runner  │
                  └────────────┬────────────┘
                               │
                 ┌─────────────┼──────────────┐
                 │             │              │
                 ▼             ▼              ▼
            Checkout       Python          Security/
                            Tests           Quality
                              │              │
                              │              ▼
                              │         SonarQube
                              │              │
                              │              ▼
                              │       Quality Gate
                              │              │
                              └───────┬──────┘
                                      │
                                ┌─────┴─────┐
                                │           │
                              PASS        FAIL
                                │           │
                                ▼           ▼
                           Continue      Stop CI
                                │
                                ▼
                         Docker / Nomad
                           Deployment
                                │
                                ▼
                           Monitoring
                         Grafana / Loki
```

## Conclusion

The `devops-intern-final` project now includes a functioning CI/CD quality-control pipeline that integrates GitHub Actions, a self-hosted WSL runner, system Python validation, SonarQube static analysis, GitHub Secrets, and an automated SonarQube Quality Gate.

The implementation successfully resolved a real-world CPU compatibility issue, eliminated duplicate workflows, securely configured SonarQube authentication, and achieved a final:

**✅ SonarQube Quality Gate: PASSED**

This provides a strong practical demonstration of CI/CD automation, infrastructure troubleshooting, code-quality enforcement, and DevOps engineering practices.

