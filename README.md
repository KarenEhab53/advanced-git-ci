# Advanced Git & CI/CD Assignment

## Features
- Docker Build & Push using GitHub Actions
- Docker Compose Integration Testing
- GitHub Secrets for secure Docker login
- CI/CD automation

## Workflows
1. build-push.yml
   - Builds Docker image
   - Pushes image to Docker Hub

2. compose-test.yml
   - Starts Docker Compose stack
   - Tests web service
   - Cleans up containers