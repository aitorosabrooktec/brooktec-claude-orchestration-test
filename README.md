# Brooktec Claude Code Workflows

Comprehensive workflow orchestration plugins and agents for frontend development with Claude AI.

## Overview

This marketplace provides a complete frontend development workflow that includes:
- Project setup and validation
- Requirements review
- Framework-aware development (React/Next.js)
- Security auditing
- Pull request management with Redmine integration

## Plugins

### 1. frontend-orchestration (Workflow Orchestrator)
**Main orchestration command for frontend feature development**

- **Commands**: `frontend-feature` - Complete 4-phase workflow for frontend features
- **Agents**: 
  - `project-setup` - Environment validation (Claude init, git branch, dependencies, .env)
  - `requirements-reviewer` - Requirements validation and clarification

**⚠️ IMPORTANT**: This plugin orchestrates the workflow but **requires other plugins to be installed** to function properly.

**Required Dependencies**:
- `frontend-mobile-development` (provides frontend-developer agent)
- `security-compliance` (provides security-auditor agent)
- `git-actions` (provides PR creation workflow)

### 2. frontend-mobile-development (Development Agents)
**Expert agents for frontend and mobile development**

- **Agents**:
  - `frontend-developer` - React/Next.js expert with framework detection and adaptation
  - `mobile-developer` - React Native and Flutter expert

### 3. security-compliance (Security Agents)
**Security auditing and compliance validation**

- **Agents**:
  - `security-auditor` - Comprehensive security assessment with OWASP Top 10

### 4. git-actions (Git & PR Management)
**Pull request creation and management with Redmine integration**

- **Commands**: `create-pull-request` - Orchestrates PR creation workflow
- **Agents**:
  - `pull-request-manager` - Handles PR tasks (validation, content generation, submission)

## Installation

### Option 1: Install Complete Workflow (Recommended)

To use the full `frontend-feature` workflow, install **ALL** plugins:

```bash
# Install all plugins for complete workflow
@frontend-orchestration
@frontend-mobile-development
@security-compliance
@git-actions
```

### Option 2: Install Individual Plugins

If you only need specific agents without the full workflow:

```bash
# Just frontend development agents
@frontend-mobile-development

# Just security auditing
@security-compliance

# Just PR management
@git-actions
```

## Usage

### Complete Frontend Feature Workflow

Once all plugins are installed, use the main orchestration command:

```
/frontend-feature Implement user authentication with JWT tokens
```

This will execute a 4-phase workflow:

#### Phase 1: Setup & Requirements Validation
1. **Project Setup** (`project-setup` agent):
   - Validates Claude project initialization (CLAUDE.md)
   - Checks git branch (must be `feature/{taskId}-*`)
   - Validates no uncommitted changes
   - Checks environment variables (.env from Shadows if needed)
   - Verifies dependencies and starts dev server
   - **⚠️ WARNING**: Checks for ESLint configuration

2. **Requirements Review** (`requirements-reviewer` agent):
   - Validates requirements completeness
   - Identifies missing information
   - Requests clarification if needed

#### Phase 2: Development
3. **Feature Implementation** (`frontend-developer` agent from frontend-mobile-development plugin):
   - Detects framework (Next.js version and router, or standard React)
   - Detects and adapts to existing tooling (state management, styling, auth)
   - Implements feature with proper TypeScript types
   - Ensures accessibility (WCAG 2.1 AA)
   - Includes tests

#### Phase 3: Security Audit
4. **Security Assessment** (`security-auditor` agent from security-compliance plugin):
   - Performs OWASP Top 10 frontend checks
   - Reviews authentication and authorization
   - Checks for XSS and injection vulnerabilities
   - Validates security headers
   - Scans dependencies
   - Generates security report

#### Phase 4: Pull Request Creation
5. **Create PR** (`create-pull-request` command from git-actions plugin):
   - Validates all changes committed
   - Requests Redmine taskId
   - Generates PR title: `[#taskId] Description`
   - Creates comprehensive PR description with security results
   - Provides manual instructions (or automated if GitHub CLI available)

## Frontend Developer Agent Features

The `frontend-developer` agent provides intelligent framework and tooling detection:

### Framework Detection
- **React Detection**: Checks for React in package.json
- **Next.js Detection**: Checks for Next.js and determines version
- **Router Detection**: Identifies App Router (`app/`) vs Pages Router (`pages/`)

### Tooling Adaptation
- **State Management**: 
  - Primary: Zustand
  - Adapts to: Redux Toolkit, Jotai, Valtio, or any existing library
  - Informs about alternatives if beneficial

- **Styling**: 
  - Primary: SCSS
  - Adapts to: PostCSS, Tailwind CSS, styled-components, emotion, or any existing approach
  - Informs about alternatives if beneficial

- **Authentication**:
  - Primary: Custom API-based (JWT, session tokens)
  - Adapts to: Auth0, Clerk, NextAuth.js, or any existing auth
  - Informs about alternatives if beneficial

## Project Setup Requirements

The `project-setup` agent validates:

1. ✅ Claude project initialized (CLAUDE.md exists)
2. ✅ Git branch: `feature/{taskId}-brief-description` format
3. ✅ No uncommitted changes
4. ✅ Node version matches .nvmrc
5. ✅ Dependencies installed
6. ✅ Environment variables (.env from Shadows/Bitwarden if template exists)
7. ✅ Dev server status
8. ⚠️ ESLint configuration (warns if missing, links to https://github.com/brooktec/knowledge-base/)

## Troubleshooting

### "frontend-developer agent not found"

**Problem**: Installed `frontend-orchestration` but the `frontend-developer` agent is not available.

**Solution**: The `frontend-orchestration` plugin requires other plugins. Install all plugins:
```
@frontend-mobile-development
@security-compliance
@git-actions
```

### "Missing .env file"

**Problem**: Project has `.env.example` but no `.env` file.

**Solution**: 
1. Check Shadows (Bitwarden) for environment variables
2. If not in Shadows, contact lead developer
3. Create `.env` file before proceeding

### "Wrong branch"

**Problem**: Working on `develop` or `main` branch.

**Solution**: Create a feature branch:
```bash
git checkout -b feature/12345-your-feature-description
```

### "Uncommitted changes"

**Problem**: Uncommitted changes in working directory.

**Solution**: Commit or stash changes before starting:
```bash
# Commit
git add .
git commit -m "Your message"

# Or stash
git stash
```

## Configuration

### ESLint
Brooktec enforces ESLint for code quality. If not configured, the setup will warn you and direct you to:
https://github.com/brooktec/knowledge-base/

### Environment Variables
Projects using environment variable templates (`.env.example`, `.env.sample`) must have actual `.env` files. Retrieve from Shadows (Bitwarden) or lead developer.

### Redmine Integration
Pull requests require a Redmine taskId for traceability. Format: `#12345` or `12345`

## Version History

### v0.0.2 (Current)
- Added Claude project initialization checking
- Added ESLint configuration validation with warning
- Enhanced framework detection (Next.js version and router)
- Added tooling adaptation (state management, styling, auth)
- Improved git branch validation
- Added comprehensive security auditing
- Created dedicated PR management workflow
- Fixed plugin source paths

### v0.0.1 (Initial)
- Initial plugin structure
- Basic frontend orchestration workflow
- Core agents implementation

## Support

For issues, questions, or configuration help:
- Check the knowledge base: https://github.com/brooktec/knowledge-base/
- Contact the Brooktec development team
- Review agent documentation in each plugin's markdown files

## License

Internal use - Brooktec Development Team

