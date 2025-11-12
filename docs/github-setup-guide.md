# GitHub Repository Setup Guide

## Step 1: Create GitHub Repository

### Option A: Using GitHub Web Interface
1. Go to [GitHub.com](https://github.com)
2. Click the **+** icon in the top right corner
3. Select **New repository**
4. Fill in the details:
   - **Repository name**: `Azure_Cloud_Migration`
   - **Description**: `Azure cloud migration project with documentation and tools`
   - **Visibility**: Choose Public or Private
   - **DO NOT** initialize with README, .gitignore, or license (we already have these)
5. Click **Create repository**

### Option B: Using GitHub CLI
```bash
# Install GitHub CLI if not already installed
brew install gh

# Authenticate
gh auth login

# Create repository
gh repo create Azure_Cloud_Migration --public --description "Azure cloud migration project"
```

## Step 2: Connect Local Repository to GitHub

After creating the GitHub repository, you'll see a page with setup instructions. Use these commands:

```bash
# Navigate to your project directory
cd /Users/calvinlee/ai_workspace_local/Azure_Cloud_Migration

# Add GitHub as remote origin (replace YOUR_USERNAME with your GitHub username)
git remote add origin https://github.com/YOUR_USERNAME/Azure_Cloud_Migration.git

# Verify remote was added
git remote -v

# Push your code to GitHub
git branch -M main
git push -u origin main
```

## Step 3: Verify Connection

After pushing, visit your GitHub repository URL:
```
https://github.com/YOUR_USERNAME/Azure_Cloud_Migration
```

You should see all your files including:
- README.md
- docs/ folder with migration guides
- templates/ folder
- .gitignore

## Step 4: Configure Git User Information (if needed)

If you haven't configured Git yet:

```bash
# Set your name
git config --global user.name "Your Name"

# Set your email
git config --global user.email "your.email@example.com"

# Verify configuration
git config --list
```

## Step 5: Set Up Branch Protection (Optional but Recommended)

For production environments:

1. Go to repository **Settings**
2. Navigate to **Branches**
3. Click **Add rule** under "Branch protection rules"
4. Set branch name pattern: `main`
5. Enable:
   - ✅ Require pull request reviews before merging
   - ✅ Require status checks to pass before merging
   - ✅ Require conversation resolution before merging
6. Click **Create**

## Working with GitHub

### Daily Workflow

```bash
# Before making changes, pull latest updates
git pull origin main

# Make your changes to files...

# Check status
git status

# Add changed files
git add .

# Commit with descriptive message
git commit -m "Add database migration assessment template"

# Push to GitHub
git push origin main
```

### Creating Feature Branches

For major changes:

```bash
# Create and switch to new branch
git checkout -b feature/add-kubernetes-migration

# Make your changes...

# Commit changes
git add .
git commit -m "Add Kubernetes migration guide"

# Push branch to GitHub
git push origin feature/add-kubernetes-migration

# Create Pull Request on GitHub
# Go to your repository and click "Compare & pull request"
```

### Syncing with Remote

```bash
# Fetch all changes from remote
git fetch origin

# View all branches (local and remote)
git branch -a

# Merge remote changes into current branch
git merge origin/main

# Or use pull (fetch + merge)
git pull origin main
```

## Collaboration Guidelines

### Commit Message Best Practices

Use conventional commit format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

**Examples**:
```bash
git commit -m "feat(discovery): Add VMware discovery automation script"
git commit -m "docs(assessment): Update SQL assessment guidelines"
git commit -m "fix(migration): Correct network configuration template"
```

### Pull Request Template

Create `.github/pull_request_template.md`:

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Documentation update
- [ ] New feature/tool
- [ ] Bug fix
- [ ] Migration script

## Migration Phase
- [ ] Discovery
- [ ] Assessment
- [ ] Planning
- [ ] Execution
- [ ] Optimization

## Checklist
- [ ] Documentation updated
- [ ] Scripts tested
- [ ] Examples provided
- [ ] No sensitive data included
```

## Troubleshooting

### Authentication Issues

**Using HTTPS with Personal Access Token**:
```bash
# If prompted for password, use Personal Access Token instead
# Create token at: https://github.com/settings/tokens
# Use token as password when pushing
```

**Switch to SSH**:
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Add to ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copy public key
cat ~/.ssh/id_ed25519.pub

# Add to GitHub: Settings > SSH and GPG keys > New SSH key

# Change remote URL to SSH
git remote set-url origin git@github.com:YOUR_USERNAME/Azure_Cloud_Migration.git
```

### Push Rejected

```bash
# If push is rejected due to remote changes
git pull --rebase origin main
git push origin main
```

### Undo Last Commit (not pushed)

```bash
# Keep changes in working directory
git reset --soft HEAD~1

# Discard changes
git reset --hard HEAD~1
```

## GitHub Repository Structure

Your repository should look like this:

```
Azure_Cloud_Migration/
├── .git/
├── .gitignore
├── README.md
├── docs/
│   ├── 01-discovery-phase.md
│   ├── 02-assessment-phase.md
│   ├── 03-migration-planning.md
│   ├── 04-migration-execution.md
│   ├── 05-optimization-phase.md
│   └── azure-mcp-guide.md
├── templates/
│   └── server-discovery-template.md
├── assessments/          (to be added)
├── migration-plans/      (to be added)
├── scripts/              (to be added)
└── tools/                (to be added)
```

## Next Steps After GitHub Setup

1. ✅ Repository created and connected
2. Set up GitHub Actions for automation (optional)
3. Enable GitHub Issues for tracking
4. Set up GitHub Projects for project management
5. Configure webhooks for notifications
6. Add collaborators if working in a team

## Using GitHub Features for Migration Project

### GitHub Issues
Track migration tasks:
- Create issues for each migration wave
- Label issues: `discovery`, `assessment`, `migration`, `optimization`
- Assign team members
- Track progress with milestones

### GitHub Projects
Create a project board:
- Columns: To Do, In Progress, Testing, Done
- Add issues as cards
- Track migration progress visually

### GitHub Wiki
Document additional information:
- Meeting notes
- Decision logs
- Architecture diagrams
- Runbooks

## Security Considerations

**Never commit**:
- ❌ Passwords or API keys
- ❌ Connection strings with credentials
- ❌ SSL certificates
- ❌ Personal access tokens
- ❌ Sensitive customer data

**Use .gitignore** to exclude:
```
*.credentials
*.key
*.pem
secrets/
.env
*-confidential.*
```

**Use Git secrets detection**:
```bash
# Install git-secrets
brew install git-secrets

# Set up hooks
git secrets --install
git secrets --register-aws
```

## Completed! 🎉

Your local repository is now connected to GitHub. You can:
- View code at: `https://github.com/YOUR_USERNAME/Azure_Cloud_Migration`
- Clone on other machines: `git clone https://github.com/YOUR_USERNAME/Azure_Cloud_Migration.git`
- Collaborate with team members
- Track issues and project progress
