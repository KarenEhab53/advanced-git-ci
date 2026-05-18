# Advanced Git & CI/CD Challenge

## 1. Workflow Architecture & Protection

### Recommended Branching Strategy: Trunk-Based Development

For a fast-moving startup, Trunk-Based Development is the best choice because it allows developers to:
- Ship features faster
- Merge small changes frequently
- Reduce merge conflicts
- Improve collaboration between team members
- Maintain continuous integration and continuous delivery (CI/CD)

In this strategy, developers create short-lived feature branches and merge them into the `main` branch through Pull Requests after testing and review.

---

### GitHub Branch Protection Rules

To protect the `main` branch and prevent broken code from being merged, the following rules should be enabled:

1. Require pull request reviews before merging
2. Require status checks to pass before merging
3. Restrict direct pushes to the `main` branch
4. Require branches to be up to date before merging

These rules improve code quality and repository stability.

---

# 2. History Cleanup (Interactive Rebase)

### Difference Between `pick` and `squash`

#### `pick`
Keeps the commit exactly as it is.

Example:
```bash
pick abc123 add login feature
```

#### `squash`
Combines the current commit with the previous commit.

Example:
```bash
squash def456 fix typo
```

This helps developers clean up messy commit history before creating a Pull Request.

---

### Interactive Rebase Command

To clean up the last 5 commits:

```bash
git rebase -i HEAD~5
```

Example rebase editor:

```text
pick a111111 wip1
s b222222 wip2
s c333333 wip3
s d444444 typo
s e555555 done
```

After saving and closing the editor, Git combines all commits into one clean commit.

---

# 3. The Emergency Surgery (Cherry-Picking)

### What is `git cherry-pick`?

`git cherry-pick` copies one specific commit from one branch and applies it to another branch without merging the entire branch.

This is perfect for emergency production fixes because we can move only the bug fix while avoiding unfinished or broken features from the `develop` branch.

---

### Cherry-Pick Commands

Critical fix commit hash:

```text
8f3a9b2
```

Commands:

```bash
# Switch to the main branch
git checkout main

# Pull latest changes
git pull origin main

# Apply the critical fix commit
git cherry-pick 8f3a9b2

# Push the fix to production
git push origin main
```

---

# 4. Integration Testing with Docker Compose (GitHub Actions)

### Why Use Integration Testing in CI?

Running integration tests in a CI pipeline instead of locally ensures:
- Consistent environments for all developers
- Automatic testing before merging code
- Early detection of deployment or networking issues
- Reliable verification that services communicate correctly
- Reduced "works on my machine" problems

Docker Compose helps simulate the full application environment during testing.

---

## GitHub Actions Workflow

File path:

```text
.github/workflows/compose-test.yml
```

Workflow configuration:

```yaml
name: Docker Compose Integration Test

on:
  pull_request:
    branches:
      - main

jobs:
  test-compose-stack:
    runs-on: ubuntu-latest

    steps:
      # Checkout repository
      - name: Checkout Code
        uses: actions/checkout@v4

      # Install Docker Compose
      - name: Install Docker Compose
        run: |
          sudo apt-get update
          sudo apt-get install docker-compose -y

      # Start Docker Compose stack
      - name: Start Docker Compose Stack
        run: docker-compose up -d

      # Wait for services to initialize
      - name: Wait for services
        run: sleep 15

      # Test web service
      - name: Test Web Service
        run: |
          curl -f http://localhost:8080 || exit 1

      # Cleanup containers and display logs
      - name: Tear down stack
        if: always()
        run: |
          docker-compose logs
          docker-compose down
```

---

# Additional CI/CD Workflow (Docker Build & Push)

File path:

```text
.github/workflows/build-push.yml
```

Workflow:

```yaml
name: build-push
run-name: build and push docker image

on:
  workflow_dispatch:
    inputs:
      image-version:
        description: "Docker image version"
        required: true
        type: string

jobs:
  build-push:
    runs-on: ubuntu-latest

    steps:
      - name: Cloning repository
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v4
        with:
          username: karenehab53
          password: ${{ secrets.DOCKER_CREDS }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and push
        uses: docker/build-push-action@v7
        with:
          context: .
          push: true
          tags: karenehab53/advanced-git-ci:${{ inputs.image-version }}
```

---

# Docker Compose File

File:

```text
docker-compose.yml
```

```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

---

# Dockerfile

File:

```text
Dockerfile
```

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

---

# Conclusion

This assignment demonstrates:
- Advanced Git workflows
- Interactive Rebase
- Cherry-Picking
- Branch Protection Rules
- Docker Build & Push automation
- Docker Compose integration testing
- GitHub Actions CI/CD pipelines
- Secure Docker authentication using GitHub Secrets