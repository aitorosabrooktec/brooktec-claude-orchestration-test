# Brooktec Claude Code Workflows

Comprehensive workflow orchestration plugins and agents for frontend development with Claude AI.

## Overview

This marketplace provides a complete frontend development workflow that includes:
- Project setup and validation
- Requirements review
- Framework-aware development (Angular, React)
- Security auditing
- Pull request management with Redmine integration

## Plugins

### 1. shared-agents (Reusable Agents Library)
**Library of reusable agents for all workflows**

- **Agents**:
  - `project-setup` - Environment validation (Claude init, git branch, dependencies, .env)
  - `technology-detector` - Technology stack detection (React, Angular, Mobile, Backend)
  - `code-reviewer` - Code quality review with framework-specific checks
  - `requirements-reviewer` - Requirements validation and clarification

### 2. frontend-orchestration (Workflow Orchestrator)
**Main orchestration command for frontend feature development**

- **Commands**: `frontend-feature` - Complete 4-phase workflow for frontend features

**⚠️ IMPORTANT**: This plugin orchestrates the workflow but **requires other plugins to be installed** to function properly.

**Required Dependencies**:
- `shared-agents` (provides project-setup, requirements-reviewer, technology-detector, code-reviewer)
- `frontend-mobile-development` (provides react-developer agent)
- `security-compliance` (provides security-auditor agent)
- `git-actions` (provides PR creation workflow)

### 3. frontend-mobile-development (Development Agents)
**Expert agents for frontend and mobile development**

- **Agents**:
  - `react-developer` - React expert with modern patterns and hooks
  - `angular-developer` - Angular expert with version-aware patterns and RxJS
  - `mobile-developer` - React Native and Flutter expert

### 4. security-compliance (Security Agents)
**Security auditing and compliance validation**

- **Agents**:
  - `security-auditor` - Comprehensive security assessment with OWASP Top 10

### 5. git-actions (Git & PR Management)
**Pull request creation and management with Redmine integration**

- **Commands**: `create-pull-request` - Orchestrates PR creation workflow
- **Agents**:
  - `pull-request-manager` - Handles PR tasks (validation, content generation, submission)

### 6. pr-review (Pull Request Review)
**Comprehensive pull request review with automated quality and security analysis**

- **Commands**: `review-pull-request` - Complete PR review workflow with technology detection, code quality analysis, security auditing, and build verification

**Required Dependencies**:
- `shared-agents` (provides technology-detector, code-reviewer)
- `security-compliance` (provides security-auditor agent)

### 7. test-orchestration (Test Generation) - NEW in v0.0.5
**Automated backend test generation with execution, coverage analysis, and quality review**

- **Commands**: `create-backend-tests` - 8-phase test generation workflow
- **Agents**:
  - `test-generator` - Generates comprehensive unit, integration, and E2E tests

**Flags**:
- `--target` (required): File or directory to generate tests for
- `--type` (optional): Test type (unit, integration, e2e, all) - default: unit
- `--coverage-threshold` (optional): Minimum coverage percentage - default: 80
- `--skip-pr` (optional): Skip PR creation

**Supported Frameworks**:
- Node.js: Jest, Vitest
- Python: pytest
- Java: JUnit

**Required Dependencies**:
- `shared-agents` (provides requirements-reviewer, project-setup, technology-detector, code-reviewer)

### 8. dependency-health (Dependency Management)
**Automated dependency health checks and updates**

- **Commands**: `check-dependencies` - 4-phase dependency management workflow
- Validates Node version, resolves vulnerabilities, updates packages, generates reports

## Installation

### Option 1: Install Complete Workflow (Recommended)

To use the full `frontend-feature` workflow, PR review, and test generation capabilities, install **ALL** plugins:

```bash
# Install all plugins for complete workflow
@shared-agents
@frontend-orchestration
@frontend-mobile-development
@security-compliance
@git-actions
@pr-review
@test-orchestration
@dependency-health
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

# Just PR review (requires shared-agents and security-compliance)
@shared-agents
@pr-review
@security-compliance

# Just test generation (requires shared-agents)
@shared-agents
@test-orchestration

# Just dependency health checks
@dependency-health
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

2. **Requirements Review** (`requirements-reviewer` agent from shared-agents plugin):
   - Validates requirements completeness
   - Identifies missing information
   - Requests clarification if needed
   - Stops workflow if critical information is missing

#### Phase 2: Development
3. **Feature Implementation** (framework-specific agent from frontend-mobile-development plugin):
   - Detects framework (Angular, React, or Mobile)
   - Routes to appropriate agent (angular-developer, react-developer, or mobile-developer)
   - Detects and adapts to existing tooling (state management, styling, auth)
   - Implements feature with proper TypeScript types
   - Ensures accessibility (WCAG 2.1 AA)
   - Includes tests

#### Phase 2.5: Code Quality Review
3.5. **Code Quality Assessment** (`code-reviewer` agent from shared-agents plugin):
   - Reviews code structure, patterns, and best practices
   - Framework-specific checks (Angular/React/Mobile patterns)
   - TypeScript type safety validation
   - Performance, testing, and accessibility review
   - Provides categorized findings with code examples
   - Identifies improvements before security audit

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

## Pull Request Review Workflow

Once the `pr-review` plugin is installed (along with `security-compliance`), use the review command:

```
/review-pull-request 42
```

This will execute a comprehensive 6-phase review workflow:

#### Phase 1: PR Information Gathering
- Fetches PR details (title, description, author, status)
- Gets list of changed files with line counts
- Retrieves full diff of all changes
- Collects commit history
- Validates PR is accessible

#### Phase 2: Technology Detection
**Uses `technology-detector` agent**:
- Analyzes project structure and configuration files
- Identifies languages, frameworks, and architectural patterns
- Detects:
  - Frontend stack (Angular, React, Vue, etc.)
  - Backend stack (Node.js/Express, NestJS, FastAPI, Django, etc.)
  - State management (Redux, Zustand, Jotai, etc.)
  - Styling approach (Tailwind, SCSS, styled-components, etc.)
  - Testing frameworks (Jest, Vitest, Pytest, etc.)
- Provides technology-specific review focus areas

#### Phase 3: Code Quality Review
**Uses `code-reviewer` agent from shared-agents plugin**:
- Reviews all changed files with technology-specific lens
- Analyzes:
  - Code quality and readability
  - Best practices adherence
  - Performance considerations
  - Maintainability and complexity
  - Testing coverage
  - Documentation completeness
- Provides structured feedback with:
  - Severity levels (CRITICAL/HIGH/MEDIUM/LOW/NITPICK)
  - Specific file:line references
  - Current code vs suggested fix examples
  - Rationale and documentation links

#### Phase 4: Security Audit
**Uses `security-auditor` agent from security-compliance plugin**:
- Performs OWASP Top 10 vulnerability scan
- Checks for:
  - SQL/NoSQL injection vulnerabilities
  - XSS and CSRF vulnerabilities
  - Authentication/authorization issues
  - Sensitive data exposure
  - Input validation problems
- Generates security compliance report

#### Phase 5: Lint & Build Verification
- Runs project linting tools (ESLint, pylint, etc.)
- Executes build process
- Checks for type errors (TypeScript, mypy, etc.)
- Reports all errors and warnings

#### Phase 6: Review Summary & Publishing
- Consolidates all findings from previous phases
- Generates comprehensive review report
- Provides final recommendation:
  - **BLOCKED**: Critical security issues or build failures
  - **REQUEST CHANGES**: High priority issues
  - **COMMENT**: Medium/low priority suggestions
  - **APPROVE**: No significant issues
- Optionally posts comments to PR (if GitHub CLI available)

### Usage Examples

**Basic PR review:**
```
/review-pull-request 42
```

**Security-focused review:**
```
/review-pull-request 123 --focus=security
```

**Quick quality check:**
```
/review-pull-request 89 --focus=quality
```

### Review Output

The review generates:
- **Detailed findings** for each issue with code examples
- **Severity categorization** (Critical → Nitpick)
- **File:line references** for all issues
- **Suggested fixes** with code snippets
- **Positive highlights** of good patterns
- **Actionable next steps** for the PR author

## Backend Test Generation Workflow

Once the `test-orchestration` plugin is installed (along with `shared-agents`), use the test generation command:

```
/create-backend-tests --target src/services/user.service.ts
```

This will execute a comprehensive 8-phase test generation workflow:

#### Phase 1: Setup & Test Target Identification
- Validates development environment
- Verifies test framework is configured
- Identifies target files to generate tests for
- Analyzes existing test coverage

#### Phase 2: Code Analysis & Test Planning
- Detects technology stack (Node.js, Python, Java)
- Analyzes target code structure and dependencies
- Identifies test scenarios (happy paths, edge cases, error handling)
- Plans mocking strategy for external dependencies

#### Phase 3: Test Generation
- Generates comprehensive test suite
- Follows framework conventions (Jest, pytest, JUnit)
- Uses AAA pattern (Arrange, Act, Assert)
- Includes proper mocking for databases, APIs, external services
- Covers all code branches and error paths

#### Phase 4: Test Execution & Validation
- Runs generated tests
- Automatically fixes test failures (up to 3 attempts)
- Ensures all tests pass before proceeding

#### Phase 5: Coverage Analysis
- Generates coverage report
- Analyzes line, branch, and function coverage
- Identifies uncovered areas
- Generates additional tests if below threshold

#### Phase 6: Test Quality Review
- Uses `code-reviewer` agent to assess test quality
- Checks test structure, assertions, and maintainability
- Validates proper use of testing framework
- Ensures tests are independent and deterministic

#### Phase 7: Approval Checkpoint
**Stops and requests user approval before PR creation**

Presents:
- Test generation summary (X tests created)
- Test execution results (all passing)
- Coverage analysis (X% achieved)
- Test quality assessment

User options:
1. **Approve and Create PR**: Proceed to PR creation
2. **Approve without PR**: Finish workflow, create PR manually later
3. **Request Changes**: Regenerate tests with modifications

#### Phase 8: Pull Request Creation (Conditional)
- Only executes if approved with PR and `--skip-pr` flag not present
- Creates PR with comprehensive test description
- Links to Redmine task
- Includes coverage metrics

### Test Generation Usage Examples

**Generate unit tests for single file:**
```
/create-backend-tests --target src/services/user.service.ts
```

**Generate tests for directory with custom coverage threshold:**
```
/create-backend-tests --target src/services/ --coverage-threshold 90
```

**Generate integration tests without creating PR:**
```
/create-backend-tests --skip-pr --target src/api/controllers/ --type integration
```

**Generate all test types (unit, integration, E2E):**
```
/create-backend-tests --target src/services/payment.service.ts --type all
```

### Supported Test Frameworks

- **Node.js**: Jest, Vitest
- **Python**: pytest
- **Java**: JUnit

## Frontend Developer Agent Features

The development agents provide intelligent framework and tooling detection:

### Framework Detection
- **Angular Detection** (`angular-developer`): Checks for @angular/core, detects version and architecture (standalone vs NgModule)
- **React Detection** (`react-developer`): Checks for React in package.json, detects version and patterns
- **Mobile Detection** (`mobile-developer`): Detects React Native, Flutter, Expo, Ionic, or native frameworks

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

### "react-developer agent not found"

**Problem**: Installed `frontend-orchestration` but the `react-developer` agent is not available.

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

### v0.0.6 (Current)
- **AGENT RENAMING**: `frontend-developer` → `react-developer`
  - Renamed agent to clearly reflect React-only focus
  - Removed all remaining Next.js references from agent
  - Updated all 6+ files referencing the agent
  - Updated marketplace.json manifest
- **REQUIREMENTS-REVIEWER MOVED TO SHARED-AGENTS**:
  - Moved from `frontend-orchestration` to `shared-agents` for reusability
  - Generalized agent to support frontend, backend, mobile, and testing workflows
  - Now used by both `frontend-feature` and `create-backend-tests` commands
  - Updated agent to be context-aware (adapts to different workflow types)
  - All orchestration commands now use `shared-agents::requirements-reviewer`
- **DOCUMENTATION UPDATES**:
  - Updated README.md to list `shared-agents` as first plugin
  - Updated all dependency graphs showing agent locations
  - Updated marketplace.json with v0.0.6 and complete plugin list
  - Added v0.0.6 version markers in dependency graphs

### v0.0.5
- **NEW PLUGIN**: `test-orchestration` - Automated backend test generation workflow
  - Created `test-generator` agent for comprehensive test creation
  - Added `create-backend-tests` command with 8-phase workflow
  - Supports unit, integration, and E2E test generation
  - Automatic test execution and validation
  - Coverage analysis with configurable thresholds
  - Test quality review with code-reviewer agent
  - Supports Jest, pytest, JUnit frameworks
  - Flags: `--target` (required), `--type`, `--coverage-threshold`, `--skip-pr`
- **ENHANCED FRONTEND WORKFLOW**: Major improvements to `/frontend-feature`
  - Added `--skip-pr` flag to skip PR creation
  - Enhanced approval checkpoint with 3 options:
    1. Approve and create PR
    2. Approve without PR (finish workflow)
    3. Request changes (return to development)
  - Phase 4 (PR Creation) now conditional based on user choice and --skip-pr flag
  - Phase 2 now uses `shared-agents::technology-detector` for consistency
  - Improved workflow flexibility and user control
- **DOCUMENTATION UPDATES**:
  - Updated CLAUDE.md with test-orchestration workflow
  - Enhanced frontend-feature workflow documentation
  - Updated dependency graphs
  - Added test generation examples and usage patterns

### v0.0.4
- **ARCHITECTURE REORGANIZATION**: Hybrid plugin structure
  - Created `shared-agents` plugin for reusable agents
  - Moved `project-setup` from frontend-orchestration to shared-agents
  - Moved `technology-detector` from pr-review to shared-agents
  - Moved and renamed `pr-reviewer` → `code-reviewer` in shared-agents
  - `pr-review` plugin now has no workflow-specific agents (fully uses shared-agents)
- **ENHANCED CODE REVIEWER**: Generalized for multiple technologies
  - Supports React, Angular, Mobile (React Native/Flutter), Node.js/Express
  - Removed all Next.js-specific references
  - Added Angular-specific review patterns (OnPush, signals, subscriptions)
  - Can review PRs, individual files, git diffs, or pre-commit changes
  - Flexible review contexts (PR review, file review, quality gates)
- **FRONTEND WORKFLOW ENHANCEMENT**: Added Phase 2.5 Code Quality Review
  - Integrated `code-reviewer` agent before security audit
  - Provides code quality feedback early in workflow
  - Reviews structure, patterns, best practices, and maintainability
  - Framework-specific checks based on detected technology
  - Approval checkpoint now reviews both quality (Phase 2.5) and security (Phase 3)
- **REMOVED NEXT.JS REFERENCES**: Simplified to React-only
  - Updated all documentation and workflow descriptions
  - `react-developer` (renamed from frontend-developer) now focuses on React patterns
  - Framework detection simplified (Angular/React/Mobile)
- **DOCUMENTATION UPDATES**:
  - Updated CLAUDE.md with hybrid architecture explanation
  - Added "Why Detailed Prompts in Commands?" section
  - Updated dependency graphs and plugin organization guidelines
  - Clarified agent reusability patterns and placement decisions

### v0.0.3
- **NEW PLUGIN**: `pr-review` - Comprehensive pull request review workflow
  - Added `technology-detector` agent for automatic tech stack detection
  - Added `pr-reviewer` agent for code quality analysis
  - Added `review-pull-request` command with 6-phase review workflow
  - Integrated security auditing with `security-auditor` agent
  - Added lint/build verification phase
  - Generates structured review reports with severity levels
  - Supports technology-specific review criteria
  - Provides actionable feedback with code examples
- Updated README with PR review documentation

### v0.0.2
- Added Claude project initialization checking
- Added ESLint configuration validation with warning
- Enhanced framework detection
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

