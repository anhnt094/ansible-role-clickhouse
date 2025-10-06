# Publishing Guide

This guide explains how to publish this role to GitHub and Ansible Galaxy.

## Prerequisites

1. GitHub account: https://github.com/anhnt094
2. Ansible Galaxy account: https://galaxy.ansible.com/
3. Link your GitHub account to Ansible Galaxy

## Step 1: Push to GitHub

### Initial Setup

```bash
# Initialize git (if not already done)
cd /Users/anhnt/Workspace/ansible-role-clickhouse
git init
git add .
git commit -m "feat: initial release v1.0.0"

# Add remote
git remote add origin https://github.com/anhnt094/ansible-role-clickhouse.git

# Push to GitHub
git branch -M main
git push -u origin main
```

### Create Release

```bash
# Create and push tag
git tag -a v1.0.0 -m "Release v1.0.0 - Initial stable release"
git push origin v1.0.0
```

Then on GitHub:

1. Go to https://github.com/anhnt094/ansible-role-clickhouse/releases
2. Click "Draft a new release"
3. Select tag `v1.0.0`
4. Title: "v1.0.0 - Initial Release"
5. Description: Copy from CHANGELOG.md
6. Click "Publish release"

## Step 2: Publish to Ansible Galaxy

### Option A: Automatic Import (Recommended)

1. Go to https://galaxy.ansible.com/
2. Sign in with GitHub
3. Go to "My Content" → "Import Role"
4. Select `anhnt094/ansible-role-clickhouse`
5. Click "OK" to import

Galaxy will automatically sync from GitHub on every new tag/release.

### Option B: Manual Import

```bash
# Login to Galaxy
ansible-galaxy login

# Import role
ansible-galaxy import anhnt094 ansible-role-clickhouse --branch main
```

## Step 3: Verify Publication

### Check GitHub

```bash
# Visit repository
open https://github.com/anhnt094/ansible-role-clickhouse

# Verify:
# - README displays correctly
# - CI badge shows green
# - Release is published
# - Files are organized properly
```

### Check Ansible Galaxy

```bash
# Visit Galaxy page
open https://galaxy.ansible.com/anhnt094/clickhouse

# Verify:
# - Role information is correct
# - Documentation displays properly
# - Download/install commands work
```

### Test Installation

```bash
# Test install from Galaxy
ansible-galaxy install anhnt094.clickhouse

# Test install from GitHub
ansible-galaxy install git+https://github.com/anhnt094/ansible-role-clickhouse.git

# Verify installation
ansible-galaxy list | grep clickhouse
```

## Step 4: Update for Future Releases

### Making Changes

```bash
# Create feature branch
git checkout -b feature/new-feature

# Make changes
# ... edit files ...

# Commit
git add .
git commit -m "feat: add new feature"

# Push
git push origin feature/new-feature

# Create Pull Request on GitHub
# After review and merge to main...
```

### Publishing Update

```bash
# Checkout main and pull latest
git checkout main
git pull origin main

# Update CHANGELOG.md
# Update version in meta/main.yml if needed

# Commit changes
git add .
git commit -m "chore: prepare v1.1.0 release"

# Create and push tag
git tag -a v1.1.0 -m "Release v1.1.0"
git push origin main
git push origin v1.1.0

# Create GitHub release
# Galaxy will auto-import from new tag
```

## Repository Settings

### Branch Protection (Recommended)

On GitHub → Settings → Branches → Add rule:

- Branch name pattern: `main`
- ✅ Require pull request reviews before merging
- ✅ Require status checks to pass before merging
- ✅ Require branches to be up to date before merging

### Topics/Tags

Add topics to GitHub repo for discoverability:

- `ansible`
- `ansible-role`
- `clickhouse`
- `database`
- `olap`
- `analytics`
- `ubuntu`
- `cluster`
- `keeper`

## Promotion

### Share on Community

- Reddit: r/ansible, r/selfhosted
- Twitter/X: #ansible #clickhouse
- Ansible Discourse: https://forum.ansible.com/
- ClickHouse Community: https://github.com/ClickHouse/ClickHouse/discussions

### Documentation

- Add link to Galaxy page in README
- Add examples and use cases
- Keep documentation up to date
- Respond to issues and PRs promptly

## Maintenance

### Regular Tasks

- Monitor CI/CD pipeline
- Review and merge PRs
- Address issues
- Keep dependencies updated
- Test with new ClickHouse versions
- Update documentation

### Version Strategy

Follow Semantic Versioning:

- `MAJOR.MINOR.PATCH`
- MAJOR: Breaking changes
- MINOR: New features (backward compatible)
- PATCH: Bug fixes

## Support

- Issues: https://github.com/anhnt094/ansible-role-clickhouse/issues
- Discussions: https://github.com/anhnt094/ansible-role-clickhouse/discussions
- Pull Requests: https://github.com/anhnt094/ansible-role-clickhouse/pulls

Good luck! 🚀
