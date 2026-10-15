# Docker Image Security Gate: Automated CVE and Hardening Checks with Trivy and GitHub Actions

A DevSecOps project that builds a Docker image on every push, audits it for
security problems, and fails the pipeline if critical vulnerabilities are found.
The goal is to catch insecure images before they get deployed, rather than after.

## What it does

- Builds a Docker image from source on every push
- Scans it for CRITICAL CVEs using Trivy and fails the pipeline if any are found
- Runs a Bash audit script that checks hardening basics:
  - Does the container run as a non-root user?
  - Which ports does the image expose?
  - How old is the image?
- Compares `nginx:1.21` against `nginx:latest` to show how image age affects security posture

## Why these checks

| Check | Why it matters |
|-------|----------------|
| CVE scan (Trivy) | Known vulnerabilities in the base image and packages are the most common way an image becomes a risk |
| Non-root user | A container running as root gives an attacker more power if it is compromised |
| Exposed ports | Every exposed port is attack surface, so only the ones the app needs should be open |
| Image age | Old images tend to carry unpatched vulnerabilities |

## Pipeline

```
push code -> build image -> scan for CVEs -> fail if CRITICAL found
```

The workflow is defined in `.github/workflows/`. A push that introduces a
CRITICAL vulnerability turns the check red, so the problem is visible before
the image goes anywhere.

## Tools used

- **Docker** to build and inspect images
- **Trivy** (Aqua Security) for vulnerability scanning
- **GitHub Actions** for CI
- **Bash** for the audit script

## Usage

Run the audit manually on any local image:

```
bash audit.sh <image-name>
```

## Example output

```
[ Checking User ]
✓ PASS - Container runs as non-root user: appuser
[ Running CVE Scan with Trivy ]
Total: 17 (HIGH: 14, CRITICAL: 3)
```

## Comparing two images

To compare the security posture of an older and a newer image, run the same
audit against both and compare the results:

```
bash audit.sh nginx:1.21
bash audit.sh nginx:latest
```

The CVE counts, user check and exposed ports for each image show how much the
choice of base image version changes the risk before any application code is added.

## What I learned

- I understood what happens behind the scenes when vulnerable infrastructure is
  about to be pushed into production, and how companies catch it automatically
  before it ships.
- It made me question how risk depends on context. A vulnerability is not equally
  dangerous everywhere: whether the vulnerable component can actually be reached
  by an attacker matters as much as the severity score itself.

## Limitations

- The pipeline only fails on CRITICAL. Adding HIGH would be stricter but noisier.
- Trivy results could be uploaded as SARIF so they appear in GitHub's Security tab.
- A Dockerfile linter could catch bad practices before the build.
- Trivy can also scan for misconfigurations and secrets, which this project doesn't use.
- The audit script checks one image at a time and has no summary report.
