# GitLab Cheatsheet

GitLab is a complete DevOps platform that provides Git repository management,
CI/CD pipelines, issue tracking, and more. This cheatsheet covers GitLab usage
from basic Git operations to advanced DevOps workflows.

## Overview

### Key Features

- **Git Repository Management** - Source code version control
- **CI/CD Pipelines** - Continuous integration and deployment
- **Issue Tracking** - Project management and bug tracking
- **Merge Requests** - Code review and collaboration
- **Container Registry** - Docker image storage
- **Security Scanning** - SAST, DAST, dependency scanning
- **Monitoring** - Application performance monitoring
- **Package Registry** - Artifact and package management

### GitLab Editions

- **GitLab.com** - SaaS offering
- **GitLab Community Edition (CE)** - Free self-hosted
- **GitLab Enterprise Edition (EE)** - Premium self-hosted

## Git Basics

### Repository Setup

```bash
# Clone repository
git clone https://gitlab.com/username/project.git
git clone git@gitlab.com:username/project.git  # SSH

# Initialize new repository
git init
git remote add origin https://gitlab.com/username/project.git

# Set up SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"
cat ~/.ssh/id_ed25519.pub  # Add to GitLab profile

# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global init.defaultBranch main
```

### Basic Git Operations

```bash
# Check status
git status
git log --oneline
git branch -a

# Stage and commit changes
git add .
git add specific-file.txt
git commit -m "Commit message"
git commit -am "Add and commit"

# Push changes
git push origin main
git push origin feature-branch
git push -u origin new-branch  # Set upstream

# Pull changes
git pull origin main
git fetch origin
git merge origin/main

# Branch management
git checkout -b feature-branch
git checkout main
git branch -d feature-branch
git push origin --delete feature-branch
```

### Advanced Git Operations

```bash
# Rebase
git rebase main
git rebase -i HEAD~3  # Interactive rebase

# Cherry pick
git cherry-pick commit-hash

# Stash changes
git stash
git stash pop
git stash list
git stash apply stash@{0}

# Reset and revert
git reset --hard HEAD~1
git revert commit-hash

# Squash commits
git reset --soft HEAD~3
git commit -m "Squashed commits"

# View differences
git diff
git diff --staged
git diff branch1..branch2
```

## GitLab CLI (glab)

### Installation

```bash
# macOS
brew install glab

# Linux
curl -s https://api.github.com/repos/profclems/glab/releases/latest | \
  grep "browser_download_url.*linux_amd64" | cut -d : -f 2,3 | tr -d \" | \
  wget -qi -
sudo tar -xzf glab_*_linux_amd64.tar.gz -C /usr/local/bin

# Windows
scoop install glab

# Verify installation
glab version
```

### Authentication and Setup

```bash
# Authenticate with GitLab
glab auth login

# Set default GitLab instance
glab config set gitlab_uri https://gitlab.example.com

# Configure default editor
glab config set editor vim

# View configuration
glab config list
```

### Repository Operations

```bash
# Clone repository
glab repo clone username/project

# Create repository
glab repo create my-project
glab repo create --private my-private-project

# Fork repository
glab repo fork username/project

# View repository info
glab repo view
glab repo view username/project

# Archive repository
glab repo archive username/project
```

## Issues

### Issue Management

```bash
# List issues
glab issue list
glab issue list --state=opened
glab issue list --assignee=@me
glab issue list --label="bug,high-priority"

# View issue
glab issue view 123

# Create issue
glab issue create
glab issue create --title="Bug report" --description="Description"
glab issue create --label="bug" --assignee="username"

# Update issue
glab issue update 123 --title="New title"
glab issue update 123 --state=closed
glab issue update 123 --label="bug,fixed"

# Close issue
glab issue close 123

# Reopen issue
glab issue reopen 123

# Link issues
glab issue create --title="Feature" --linked-mr=456
```

### Issue Templates

```markdown
<!-- .gitlab/issue_templates/Bug.md -->
## Bug Report

### Description
A clear description of the bug.

### Steps to Reproduce
1. Go to '...'
2. Click on '....'
3. Scroll down to '....'
4. See error

### Expected Behavior
What you expected to happen.

### Actual Behavior
What actually happened.

### Environment
- OS: [e.g., Ubuntu 20.04]
- Browser: [e.g., Chrome 91]
- Version: [e.g., v1.2.3]

### Additional Context
Any other information about the problem.

/label ~bug
/assign @username
```

## Merge Requests

### Merge Request Operations

```bash
# List merge requests
glab mr list
glab mr list --state=opened
glab mr list --author=@me
glab mr list --reviewer=username

# View merge request
glab mr view 456
glab mr view 456 --web  # Open in browser

# Create merge request
glab mr create
glab mr create --title="Feature: Add new functionality"
glab mr create --source-branch=feature --target-branch=main
glab mr create --draft  # Create draft MR

# Update merge request
glab mr update 456 --title="New title"
glab mr update 456 --description="New description"
glab mr update 456 --label="feature,ready"

# Merge request actions
glab mr approve 456
glab mr unapprove 456
glab mr merge 456
glab mr close 456
glab mr reopen 456

# Checkout merge request
glab mr checkout 456

# View merge request diff
glab mr diff 456

# Add note to merge request
glab mr note 456 --message="Looks good to me!"
```

### Merge Request Templates

```markdown
<!-- .gitlab/merge_request_templates/Default.md -->
## Description
Brief description of the changes.

## Changes
- [ ] Feature 1
- [ ] Feature 2
- [ ] Bug fix

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Screenshots
<!-- Add screenshots if applicable -->

## Related Issues
Closes #123
Related to #456

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] Tests added/updated

/label ~feature
/assign @reviewer
```

## CI/CD Pipelines

### GitLab CI/CD Configuration

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - security
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"

# Global before_script
before_script:
  - echo "Starting CI/CD pipeline"

# Build stage
build:
  stage: build
  image: node:16
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour
  only:
    - main
    - merge_requests

# Test stage
test:unit:
  stage: test
  image: node:16
  script:
    - npm ci
    - npm run test:unit
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

test:integration:
  stage: test
  image: node:16
  services:
    - postgres:13
  variables:
    POSTGRES_DB: test_db
    POSTGRES_USER: test_user
    POSTGRES_PASSWORD: test_pass
  script:
    - npm ci
    - npm run test:integration
  only:
    - main
    - merge_requests

# Security scanning
security:sast:
  stage: security
  template: Security/SAST.gitlab-ci.yml

security:dependency_scanning:
  stage: security
  template: Security/Dependency-Scanning.gitlab-ci.yml

security:container_scanning:
  stage: security
  template: Security/Container-Scanning.gitlab-ci.yml

# Deployment stages
deploy:staging:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - echo "Deploying to staging"
    - curl -X POST "$STAGING_DEPLOY_URL"
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - main

deploy:production:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying to production"
    - curl -X POST "$PRODUCTION_DEPLOY_URL"
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

### Advanced CI/CD Features

```yaml
# Multi-project pipelines
trigger:downstream:
  trigger:
    project: group/downstream-project
    branch: main
    strategy: depend

# Parallel jobs
test:parallel:
  stage: test
  parallel: 4
  script:
    - npm run test -- --shard=$CI_NODE_INDEX/$CI_NODE_TOTAL

# Matrix builds
test:matrix:
  stage: test
  parallel:
    matrix:
      - NODE_VERSION: ["14", "16", "18"]
        OS: ["ubuntu", "alpine"]
  image: node:$NODE_VERSION-$OS
  script:
    - npm test

# Conditional jobs
deploy:review:
  stage: deploy
  script:
    - echo "Deploy review app"
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://$CI_COMMIT_REF_SLUG.review.example.com
    on_stop: stop:review
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

stop:review:
  stage: deploy
  script:
    - echo "Stop review app"
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop
  when: manual
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: manual
```

### Docker Integration

```yaml
# Build and push Docker image
build:docker:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  variables:
    DOCKER_HOST: tcp://docker:2376
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest

# Deploy with Docker Compose
deploy:docker:
  stage: deploy
  image: docker/compose:latest
  script:
    - docker-compose -f docker-compose.prod.yml up -d
  environment:
    name: production
```

## GitLab API

### Authentication

```bash
# Using personal access token
TOKEN="your-personal-access-token"
GITLAB_URL="https://gitlab.com/api/v4"
HEADERS="PRIVATE-TOKEN: $TOKEN"

# Using project access token
PROJECT_TOKEN="your-project-access-token"
HEADERS="PRIVATE-TOKEN: $PROJECT_TOKEN"
```

### Project API

```bash
# List projects
curl -H "$HEADERS" "$GITLAB_URL/projects"

# Get project
curl -H "$HEADERS" "$GITLAB_URL/projects/123"

# Create project
curl -X POST -H "$HEADERS" -H "Content-Type: application/json" \
  "$GITLAB_URL/projects" \
  -d '{
    "name": "My Project",
    "description": "Project description",
    "visibility": "private"
  }'

# Update project
curl -X PUT -H "$HEADERS" -H "Content-Type: application/json" \
  "$GITLAB_URL/projects/123" \
  -d '{
    "description": "Updated description"
  }'

# Delete project
curl -X DELETE -H "$HEADERS" "$GITLAB_URL/projects/123"
```

### Issues API

```bash
# List issues
curl -H "$HEADERS" "$GITLAB_URL/projects/123/issues"

# Get issue
curl -H "$HEADERS" "$GITLAB_URL/projects/123/issues/456"

# Create issue
curl -X POST -H "$HEADERS" -H "Content-Type: application/json" \
  "$GITLAB_URL/projects/123/issues" \
  -d '{
    "title": "Bug report",
    "description": "Issue description",
    "labels": "bug,high-priority"
  }'

# Update issue
curl -X PUT -H "$HEADERS" -H "Content-Type: application/json" \
  "$GITLAB_URL/projects/123/issues/456" \
  -d '{
    "state_event": "close"
  }'
```

### Merge Requests API

```bash
# List merge requests
curl -H "$HEADERS" "$GITLAB_URL/projects/123/merge_requests"

# Get merge request
curl -H "$HEADERS" "$GITLAB_URL/projects/123/merge_requests/789"

# Create merge request
curl -X POST -H "$HEADERS" -H "Content-Type: application/json" \
  "$GITLAB_URL/projects/123/merge_requests" \
  -d '{
    "source_branch": "feature-branch",
    "target_branch": "main",
    "title": "Feature: Add new functionality"
  }'

# Approve merge request
curl -X POST -H "$HEADERS" \
  "$GITLAB_URL/projects/123/merge_requests/789/approve"

# Merge request
curl -X PUT -H "$HEADERS" \
  "$GITLAB_URL/projects/123/merge_requests/789/merge"
```

### Pipelines API

```bash
# List pipelines
curl -H "$HEADERS" "$GITLAB_URL/projects/123/pipelines"

# Get pipeline
curl -H "$HEADERS" "$GITLAB_URL/projects/123/pipelines/456"

# Create pipeline
curl -X POST -H "$HEADERS" -H "Content-Type: application/json" \
  "$GITLAB_URL/projects/123/pipeline" \
  -d '{
    "ref": "main"
  }'

# Cancel pipeline
curl -X POST -H "$HEADERS" \
  "$GITLAB_URL/projects/123/pipelines/456/cancel"

# Retry pipeline
curl -X POST -H "$HEADERS" \
  "$GITLAB_URL/projects/123/pipelines/456/retry"

# Get pipeline jobs
curl -H "$HEADERS" "$GITLAB_URL/projects/123/pipelines/456/jobs"
```

## Container Registry

### Docker Registry Usage

```bash
# Login to GitLab Container Registry
docker login registry.gitlab.com

# Build and tag image
docker build -t registry.gitlab.com/username/project:latest .

# Push image
docker push registry.gitlab.com/username/project:latest

# Pull image
docker pull registry.gitlab.com/username/project:latest

# List registry repositories
curl -H "$HEADERS" "$GITLAB_URL/projects/123/registry/repositories"

# Delete registry repository
curl -X DELETE -H "$HEADERS" \
  "$GITLAB_URL/projects/123/registry/repositories/456"
```

### Multi-stage Docker Build

```dockerfile
# Dockerfile
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:16-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

## Package Registry

### NPM Package Registry

```bash
# Configure npm registry
npm config set @your-scope:registry https://gitlab.com/api/v4/projects/123/packages/npm/
npm config set //gitlab.com/api/v4/projects/123/packages/npm/:_authToken $TOKEN

# Publish package
npm publish

# Install package
npm install @your-scope/package-name
```

### Maven Package Registry

```xml
<!-- pom.xml -->
<repositories>
  <repository>
    <id>gitlab-maven</id>
    <url>https://gitlab.com/api/v4/projects/123/packages/maven</url>
  </repository>
</repositories>

<distributionManagement>
  <repository>
    <id>gitlab-maven</id>
    <url>https://gitlab.com/api/v4/projects/123/packages/maven</url>
  </repository>
</distributionManagement>
```

### Python Package Registry

```bash
# Configure pip
echo "[global]
index-url = https://gitlab.com/api/v4/projects/123/packages/pypi/simple" > ~/.pip/pip.conf

# Upload package with twine
twine upload --repository-url https://gitlab.com/api/v4/projects/123/packages/pypi dist/*

# Install package
pip install --index-url https://gitlab.com/api/v4/projects/123/packages/pypi/simple your-package
```

## Security and Compliance

### Security Scanning

```yaml
# SAST (Static Application Security Testing)
include:
  - template: Security/SAST.gitlab-ci.yml

# Dependency Scanning
include:
  - template: Security/Dependency-Scanning.gitlab-ci.yml

# Container Scanning
include:
  - template: Security/Container-Scanning.gitlab-ci.yml

# DAST (Dynamic Application Security Testing)
include:
  - template: Security/DAST.gitlab-ci.yml

dast:
  variables:
    DAST_WEBSITE: https://example.com

# License Scanning
include:
  - template: Security/License-Scanning.gitlab-ci.yml

# Secret Detection
include:
  - template: Security/Secret-Detection.gitlab-ci.yml
```

### Compliance Pipeline

```yaml
# Compliance framework
include:
  - project: 'compliance/security-policies'
    file: 'security-pipeline.yml'

# Compliance checks
compliance:audit:
  stage: compliance
  script:
    - echo "Running compliance checks"
    - audit-tool --config compliance.json
  artifacts:
    reports:
      junit: compliance-report.xml
  only:
    - main
    - merge_requests
```

## Monitoring and Observability

### Application Performance Monitoring

```yaml
# Enable APM
performance:
  stage: test
  image: sitespeedio/sitespeed.io
  script:
    - sitespeed.io --budget budget.json https://example.com
  artifacts:
    reports:
      performance: performance.json
```

### Error Tracking

```javascript
// Sentry integration
import * as Sentry from '@sentry/browser';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  release: process.env.CI_COMMIT_SHA
});
```

## GitLab Pages

### Static Site Deployment

```yaml
# Deploy to GitLab Pages
pages:
  stage: deploy
  image: node:16
  script:
    - npm ci
    - npm run build
    - mkdir public
    - cp -r dist/* public/
  artifacts:
    paths:
      - public
  only:
    - main

# Custom domain setup
# 1. Add CNAME record: your-domain.com -> username.gitlab.io
# 2. Add domain in project settings
# 3. Obtain SSL certificate
```

### Documentation Site

```yaml
# MkDocs documentation
pages:mkdocs:
  stage: deploy
  image: python:3.9
  before_script:
    - pip install mkdocs mkdocs-material
  script:
    - mkdocs build
    - mv site public
  artifacts:
    paths:
      - public
  only:
    - main
```

## GitLab Runners

### Shared Runners

```yaml
# Use GitLab.com shared runners
job:shared:
  image: ubuntu:20.04
  script:
    - echo "Running on shared runner"
  tags:
    - docker
```

### Self-hosted Runners

```bash
# Install GitLab Runner
curl -L \
  "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | \
  sudo bash
sudo apt-get install gitlab-runner

# Register runner
sudo gitlab-runner register \
  --url https://gitlab.com/ \
  --registration-token $REGISTRATION_TOKEN \
  --description "My Runner" \
  --tag-list "docker,linux" \
  --executor docker \
  --docker-image ubuntu:20.04

# Start runner
sudo gitlab-runner start

# Check runner status
sudo gitlab-runner status

# Update runner
sudo gitlab-runner stop
sudo apt-get update && sudo apt-get install gitlab-runner
sudo gitlab-runner start
```

### Runner Configuration

```toml
# /etc/gitlab-runner/config.toml
concurrent = 4
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "docker-runner"
  url = "https://gitlab.com/"
  token = "TOKEN"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "ubuntu:20.04"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
    shm_size = 0
```

## Project Management

### Project Templates

```bash
# Create project from template
glab repo create my-project --template=group/template-project

# Use GitLab built-in templates
# - Ruby on Rails
# - Spring Boot
# - Django
# - Express.js
# - Android
# - iOS
```

### Group Management

```bash
# List groups
glab group list

# Create group
glab group create my-group

# View group
glab group view my-group

# List group projects
glab group list-projects my-group
```

### Milestones and Epics

```bash
# List milestones
glab milestone list

# Create milestone
glab milestone create "v1.0.0" --description="First release"

# List epics (GitLab Premium)
glab epic list

# Create epic
glab epic create --title="User Authentication" --description="Epic description"
```

## Advanced Features

### Multi-project Pipelines

```yaml
# Trigger downstream pipeline
trigger:downstream:
  stage: deploy
  trigger:
    project: group/downstream-project
    branch: main
    strategy: depend
  variables:
    UPSTREAM_PROJECT: $CI_PROJECT_NAME
    UPSTREAM_COMMIT: $CI_COMMIT_SHA
```

### Parent-child Pipelines

```yaml
# Generate dynamic child pipeline
generate:config:
  stage: build
  script:
    - python generate-pipeline.py > child-pipeline.yml
  artifacts:
    paths:
      - child-pipeline.yml

trigger:child:
  stage: deploy
  trigger:
    include:
      - artifact: child-pipeline.yml
        job: generate:config
    strategy: depend
```

### Auto DevOps

```yaml
# Enable Auto DevOps
include:
  - template: Auto-DevOps.gitlab-ci.yml

variables:
  AUTO_DEVOPS_DOMAIN: example.com
  POSTGRES_ENABLED: "true"
  POSTGRES_VERSION: "13"
```

## Troubleshooting

### Common Git Issues

```bash
# Fix merge conflicts
git status
git add .
git commit -m "Resolve merge conflicts"

# Reset to remote state
git fetch origin
git reset --hard origin/main

# Undo last commit
git reset --soft HEAD~1

# Change last commit message
git commit --amend -m "New commit message"

# Push force (be careful!)
git push --force-with-lease origin feature-branch
```

### Pipeline Debugging

```yaml
# Debug pipeline
debug:pipeline:
  stage: test
  script:
    - echo "Debug information"
    - env | sort
    - ls -la
    - pwd
  when: manual
```

### Runner Issues

```bash
# Check runner logs
sudo gitlab-runner --debug run

# Clear runner cache
sudo gitlab-runner exec docker --cache-dir /cache cleanup

# Verify runner connection
sudo gitlab-runner verify

# Re-register runner
sudo gitlab-runner unregister --name "runner-name"
sudo gitlab-runner register
```

## Useful Commands Reference

| Command | Description |
|---------|-------------|
| `glab auth login` | Authenticate with GitLab |
| `glab repo clone <repo>` | Clone repository |
| `glab issue create` | Create new issue |
| `glab mr create` | Create merge request |
| `glab mr merge <id>` | Merge request |
| `glab pipeline list` | List pipelines |
| `git push origin <branch>` | Push changes |
| `git pull origin <branch>` | Pull changes |
| `git checkout -b <branch>` | Create new branch |
| `git merge <branch>` | Merge branch |

## Environment Variables

### GitLab CI/CD Variables

```bash
# Predefined variables
CI_COMMIT_SHA           # Commit SHA
CI_COMMIT_REF_NAME      # Branch or tag name
CI_PROJECT_NAME         # Project name
CI_PROJECT_ID           # Project ID
CI_PIPELINE_ID          # Pipeline ID
CI_JOB_ID              # Job ID
CI_REGISTRY            # GitLab Container Registry URL
CI_REGISTRY_IMAGE      # Full image path
GITLAB_USER_EMAIL      # User email

# Custom variables (set in project settings)
API_KEY                # Custom API key
DATABASE_URL          # Database connection string
DEPLOY_TOKEN          # Deployment token
```

### CLI Configuration

```bash
# GitLab CLI configuration
export GITLAB_TOKEN="your-personal-access-token"
export GITLAB_URI="https://gitlab.example.com"
export GLAB_CONFIG_DIR="$HOME/.config/glab"

# Git configuration
export GIT_AUTHOR_NAME="Your Name"
export GIT_AUTHOR_EMAIL="your.email@example.com"
export GIT_COMMITTER_NAME="Your Name"
export GIT_COMMITTER_EMAIL="your.email@example.com"
```

## Best Practices

### Repository Management

1. **Branch Protection** - Protect main/master branch
2. **Merge Request Reviews** - Require code reviews
3. **Commit Messages** - Use conventional commit format
4. **Issue Templates** - Standardize issue reporting
5. **Documentation** - Maintain comprehensive README

### CI/CD Pipeline Design

1. **Fast Feedback** - Optimize pipeline speed
2. **Fail Fast** - Put quick tests early
3. **Parallel Execution** - Run independent jobs in parallel
4. **Artifact Management** - Cache dependencies
5. **Security First** - Include security scans

### Security

1. **Access Tokens** - Use project/group tokens with minimal scope
2. **Secret Management** - Use CI/CD variables for secrets
3. **Dependency Updates** - Regularly update dependencies
4. **Security Scanning** - Enable all security templates
5. **Access Control** - Implement proper role-based access

---

*For more detailed information, visit the [official GitLab documentation](https://docs.gitlab.com/)*
