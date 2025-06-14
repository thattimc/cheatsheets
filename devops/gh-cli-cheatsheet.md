# GitHub CLI (gh) Cheatsheet

## Installation & Authentication

### Install gh
```bash
# macOS
brew install gh

# Ubuntu/Debian
sudo apt install gh

# Windows
winget install --id GitHub.cli
```

### Authentication
```bash
# Login with web browser
gh auth login

# Login with token
gh auth login --with-token < token.txt

# Check authentication status
gh auth status

# Logout
gh auth logout
```

## Repository Management

### Create Repository
```bash
# Create new repo (interactive)
gh repo create

# Create public repo
gh repo create my-repo --public

# Create private repo with description
gh repo create my-repo --private --description "My awesome project"

# Create repo from current directory
gh repo create my-repo --source . --push

# Create repo and clone it
gh repo create my-repo --clone

# Create from template
gh repo create my-app --template owner/template-repo
```

### Repository Operations
```bash
# Clone repository
gh repo clone owner/repo
gh repo clone owner/repo target-directory

# Fork repository
gh repo fork owner/repo
gh repo fork owner/repo --clone

# View repository
gh repo view
gh repo view owner/repo
gh repo view owner/repo --web

# List repositories
gh repo list
gh repo list owner
gh repo list --limit 50

# Delete repository
gh repo delete owner/repo
```

## Issues

### Create & Manage Issues
```bash
# Create issue (interactive)
gh issue create

# Create issue with title and body
gh issue create --title "Bug report" --body "Description here"

# Create issue from template
gh issue create --template bug_report.md

# Assign issue
gh issue create --assignee username

# Add labels
gh issue create --label bug,priority-high
```

### View & List Issues
```bash
# List issues
gh issue list
gh issue list --state open
gh issue list --state closed
gh issue list --assignee username
gh issue list --label bug

# View specific issue
gh issue view 123
gh issue view 123 --web

# Show issue comments
gh issue view 123 --comments
```

### Issue Actions
```bash
# Close issue
gh issue close 123
gh issue close 123 --comment "Fixed in PR #456"

# Reopen issue
gh issue reopen 123

# Edit issue
gh issue edit 123
gh issue edit 123 --title "New title"
gh issue edit 123 --body "New description"

# Add comment
gh issue comment 123 --body "This is a comment"

# Pin/unpin issue
gh issue pin 123
gh issue unpin 123
```

## Pull Requests

### Create Pull Requests
```bash
# Create PR (interactive)
gh pr create

# Create PR with details
gh pr create --title "Feature: Add new functionality" --body "Description"

# Create draft PR
gh pr create --draft

# Create PR to specific branch
gh pr create --base main --head feature-branch

# Create PR with reviewers
gh pr create --reviewer username1,username2

# Create PR with assignees
gh pr create --assignee username
```

### View & List Pull Requests
```bash
# List PRs
gh pr list
gh pr list --state open
gh pr list --state merged
gh pr list --author username

# View specific PR
gh pr view 123
gh pr view 123 --web

# Show PR diff
gh pr diff 123

# Show PR checks
gh pr checks 123
```

### Pull Request Actions
```bash
# Checkout PR branch
gh pr checkout 123

# Merge PR
gh pr merge 123
gh pr merge 123 --merge    # Create merge commit
gh pr merge 123 --squash   # Squash and merge
gh pr merge 123 --rebase   # Rebase and merge

# Close PR
gh pr close 123

# Reopen PR
gh pr reopen 123

# Mark as ready for review
gh pr ready 123

# Convert to draft
gh pr convert-to-draft 123

# Add comment
gh pr comment 123 --body "LGTM!"

# Request review
gh pr edit 123 --add-reviewer username
```

### Pull Request Reviews
```bash
# Review PR
gh pr review 123
gh pr review 123 --approve
gh pr review 123 --request-changes
gh pr review 123 --comment --body "Looks good!"
```

## Releases

### Create & Manage Releases
```bash
# Create release (interactive)
gh release create

# Create release with tag
gh release create v1.0.0

# Create release with title and notes
gh release create v1.0.0 --title "Version 1.0.0" --notes "Release notes"

# Create pre-release
gh release create v1.0.0-beta --prerelease

# Create draft release
gh release create v1.0.0 --draft

# Upload assets
gh release create v1.0.0 ./dist/*
```

### View & List Releases
```bash
# List releases
gh release list
gh release list --limit 10

# View specific release
gh release view v1.0.0
gh release view latest

# Download release assets
gh release download v1.0.0
gh release download v1.0.0 --pattern "*.tar.gz"
```

## Workflows & Actions

### GitHub Actions
```bash
# List workflows
gh workflow list

# View workflow runs
gh run list
gh run list --workflow=ci.yml

# View specific run
gh run view 123456789

# Watch run in real-time
gh run watch 123456789

# Rerun workflow
gh run rerun 123456789

# Cancel run
gh run cancel 123456789

# Download run artifacts
gh run download 123456789
```

### Trigger Workflows
```bash
# Trigger workflow dispatch
gh workflow run workflow.yml

# Trigger with inputs
gh workflow run workflow.yml --field key=value
```

## Gists

### Create & Manage Gists
```bash
# Create gist from file
gh gist create file.txt

# Create private gist
gh gist create file.txt --secret

# Create gist with description
gh gist create file.txt --desc "My code snippet"

# Create gist from stdin
echo "Hello World" | gh gist create --filename hello.txt
```

### View & Edit Gists
```bash
# List gists
gh gist list
gh gist list --public
gh gist list --secret

# View gist
gh gist view abc123

# Edit gist
gh gist edit abc123

# Clone gist
gh gist clone abc123

# Delete gist
gh gist delete abc123
```

## Search

### Search GitHub
```bash
# Search repositories
gh search repos "machine learning" --language python
gh search repos --owner microsoft

# Search issues
gh search issues "bug" --repo owner/repo
gh search issues --author username

# Search pull requests
gh search prs "feature" --state open

# Search code
gh search code "function main" --language go
```

## Aliases

### Create & Manage Aliases
```bash
# List aliases
gh alias list

# Create alias
gh alias set pv 'pr view'
gh alias set co 'pr checkout'

# Delete alias
gh alias delete pv
```

## Configuration

### Settings
```bash
# List configuration
gh config list

# Set configuration
gh config set editor vim
gh config set git_protocol ssh
gh config set prompt enabled

# Get configuration value
gh config get editor
```

### Common Config Options
```bash
# Set default editor
gh config set editor code

# Set Git protocol
gh config set git_protocol https
# or
gh config set git_protocol ssh

# Disable interactive prompts
gh config set prompt disabled
```

## Extensions

### Manage Extensions
```bash
# List extensions
gh extension list

# Install extension
gh extension install owner/gh-extension-name

# Upgrade extensions
gh extension upgrade --all
gh extension upgrade extension-name

# Remove extension
gh extension remove extension-name
```

## Useful Flags & Options

### Global Flags
```bash
--help          # Show help
--version       # Show version
--repo owner/repo  # Specify repository
--json          # Output in JSON format
--jq EXPRESSION # Filter JSON output
--template STRING  # Format output using Go templates
```

### Common Patterns
```bash
# Use JSON output with jq
gh pr list --json number,title,author | jq '.[] | select(.author.login == "username")'

# Use templates
gh issue list --template '{{range .}}{{.number}}: {{.title}}{{"\n"}}{{end}}'

# Work with specific repo
gh issue list --repo owner/repo
```

## Tips & Tricks

### Productivity Tips
1. **Use aliases** for frequently used commands
2. **Set up SSH** for faster authentication
3. **Use templates** for consistent issue/PR creation
4. **Combine with jq** for powerful JSON processing
5. **Use --web flag** to open items in browser

### Environment Variables
```bash
# Set default repository
export GH_REPO="owner/repo"

# Set token (alternative to gh auth login)
export GH_TOKEN="your_token_here"

# Set editor
export GH_EDITOR="code"
```

### Bash/Zsh Completion
```bash
# Add to ~/.bashrc or ~/.zshrc
eval "$(gh completion -s bash)"  # for bash
eval "$(gh completion -s zsh)"   # for zsh
```

