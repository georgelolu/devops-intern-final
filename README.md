# DevOps Intern Final Assessment

![DevOps CI](https://github.com/georgelolu/devops-intern-final/actions/workflows/ci.yml/badge.svg)

## Student Information

**Name:** George Omololu Akinbi  
**Date:** August 13, 2026

## Project Description

This project demonstrates a complete DevOps workflow using open-source tools and practices.

The project covers:

1. Git and GitHub
2. Linux scripting
3. Docker containerization
4. GitHub Actions CI/CD
5. Nomad job deployment
6. Grafana Loki monitoring
7. Technical documentation and evidence

Each stage produces an output that is used in the following stage.

## Technologies

- Linux
- Git
- GitHub
- Python
- Bash
- Docker
- GitHub Actions
- HashiCorp Nomad
- Grafana Loki

## Step 3 - Docker

The `hello.py` application was containerized using Docker.

### Build the image

```bash
docker build -t devops-intern-final:1.0 .
