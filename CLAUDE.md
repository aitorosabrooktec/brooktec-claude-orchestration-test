# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Brooktec Claude Code Workflows** marketplace - a collection of specialized plugins and agents that orchestrate comprehensive frontend development workflows. The project provides workflow automation for:

- Project setup and environment validation
- Requirements review and clarification
- Framework-aware development (Angular, React, React Native, Flutter)
- Security auditing (OWASP Top 10)
- Pull request management with Redmine integration
- PR code review with technology detection
- Dependency health checks

## Repository Structure

```
.
├── plugins/                          # All plugin definitions
│   ├── shared-agents/               # Reusable agents library
│   │   └── agents/                  # project-setup, technology-detector, code-reviewer
│   ├── frontend-orchestration/      # Main workflow orchestrator
│   │   ├── agents/                  # requirements-reviewer (workflow-specific)
│   │   └── commands/                # frontend-feature command
│   ├── frontend-mobile-development/ # Development agents library
│   │   └── agents/                  # frontend-developer, angular-developer, mobile-developer
│   ├── security-compliance/         # Security agents library
│   │   └── agents/                  # security-auditor
│   ├── git-actions/                 # PR management workflow
│   │   ├── agents/                  # pull-request-manager (workflow-specific)
│   │   └── commands/                # create-pull-request
│   ├── pr-review/                   # PR review workflow
│   │   └── commands/                # review-pull-request (no workflow-specific agents)
│   └── dependency-health/           # Dependency management workflow
│       ├── agents/                  # dependency-health-checker (workflow-specific)
│       └── commands/                # check-dependencies
└── README.md                         # User-facing documentation
```

## Architecture Principles

### Plugin-Based Architecture (Hybrid Approach)

The marketplace uses a **hybrid architecture** that separates reusable components from workflow-specific implementations:

**Agent Library Plugins** (agents/ only):
- Provide reusable agents that can be invoked by multiple workflows
- Examples: `shared-agents`, `frontend-mobile-development`, `security-compliance`
- Agents in these plugins are meant to be shared across workflows
- These are "library" plugins that other plugins depend on

**Workflow Plugins** (commands/ + workflow-specific agents/):
- Provide user-facing commands that orchestrate complete workflows
- May include workflow-specific agents as internal implementation details
- Examples: `frontend-orchestration`, `git-actions`, `pr-review`, `dependency-health`
- These are "application" plugins that users interact with directly

### Agent and Command System

- **Agents**: Specialized autonomous task handlers (markdown files in `agents/` directories)
- **Commands**: Workflow orchestrators that coordinate multiple agents (markdown files in `commands/` directories)

Agents are defined in markdown files and invoked using:
```
Tool: Task
subagent_type: "plugin-name::agent-name"
```

Commands are slash commands invoked with:
```
Tool: SlashCommand
command: "/command-name"
```

### Workflow Orchestration Pattern

Commands follow a phase-based execution model:
1. **Sequential phases** - Each phase must complete before the next begins
2. **Approval checkpoints** - Workflows stop for user approval at critical points
3. **Error handling** - Workflows STOP and request user action when encountering blockers
4. **Agent coordination** - Commands invoke multiple specialized agents in sequence

### Dependency Graph

```
AGENT LIBRARY PLUGINS (Reusable):
├── shared-agents
│   ├── project-setup (used by: frontend-orchestration, potentially others)
│   ├── technology-detector (used by: pr-review, potentially others)
│   └── code-reviewer (used by: pr-review, can be used by: frontend-orchestration, quality-gates)
├── frontend-mobile-development
│   ├── frontend-developer (used by: frontend-orchestration)
│   ├── angular-developer (used by: frontend-orchestration)
│   └── mobile-developer (used by: frontend-orchestration)
└── security-compliance
    └── security-auditor (used by: frontend-orchestration, pr-review)

WORKFLOW PLUGINS (User-Facing):
├── frontend-orchestration
│   ├─> Uses: shared-agents::project-setup
│   ├─> Uses: frontend-orchestration::requirements-reviewer (internal)
│   ├─> Uses: frontend-mobile-development::{frontend,angular,mobile}-developer
│   ├─> Uses: security-compliance::security-auditor
│   ├─> Uses: git-actions::create-pull-request command
│   └─> Potential: shared-agents::code-reviewer (for pre-PR quality gate)
├── pr-review
│   ├─> Uses: shared-agents::technology-detector
│   ├─> Uses: shared-agents::code-reviewer
│   └─> Uses: security-compliance::security-auditor
├── git-actions
│   └─> Uses: git-actions::pull-request-manager (internal)
└── dependency-health
    └─> Uses: dependency-health::dependency-health-checker (internal)
```

### Why Detailed Prompts in Commands?

Command files contain **detailed prompts** when invoking agents. This is **intentional**, not duplication:

**Commands = Workflow Specification**
- Define the complete workflow logic and orchestration
- Provide context-specific instructions for how agents should be used
- Self-documenting: You can read a command file and understand the entire workflow
- Specify expected outputs and success criteria for each step

**Agents = Implementation Specification**
- Define the agent's capabilities, behavior traits, and knowledge base
- Contain the "how" - implementation details and approach
- Reusable across different workflows with different contexts
- Focus on the agent's expertise domain

**Example**: `project-setup` agent knows HOW to validate environments. The `frontend-feature` command tells it WHAT to validate in the context of frontend feature development (including Shadows integration, ESLint requirements, etc.).

This separation allows:
1. **Workflows to be self-documenting** - all logic visible in command file
2. **Agents to be reusable** - same agent, different contexts
3. **Flexibility** - same agent can be invoked differently by different workflows
4. **Maintainability** - workflow changes don't require agent changes

## Key Workflows

### 1. Frontend Feature Development (`/frontend-feature`)

**Location**: `plugins/frontend-orchestration/commands/frontend-feature.md`

Four-phase workflow with mandatory approval checkpoint:

- **Phase 1**: Setup & Requirements Validation
  - Verifies CLAUDE.md exists (stops if missing, directs user to run `/init`)
  - Validates git branch format: `feature/{taskId}-*` (stops on develop/main)
  - Checks for uncommitted changes (stops if any exist)
  - Validates environment (.env from Shadows, node_modules, dev server)
  - Reviews requirements for completeness

- **Phase 2**: Development (with technology detection)
  - Detects project technology (Angular, React, or Mobile)
  - If ambiguous, STOPS and asks user to specify
  - Routes to appropriate agent:
    - `angular-developer` for Angular projects
    - `frontend-developer` for React projects
    - `mobile-developer` for React Native/Flutter/Expo/Ionic/Native
  - Detects existing tooling (state management, styling, auth) and adapts
  - Implements feature following framework-specific patterns

- **Phase 3**: Security Audit & Approval Checkpoint
  - Performs comprehensive OWASP Top 10 security audit
  - STOPS after audit and presents summary to user
  - User must explicitly approve (yes/approve/continue) to proceed
  - User can request changes (returns to Phase 2)
  - **Does NOT proceed to Phase 4 without user approval**

- **Phase 4**: Pull Request Creation (only after approval)
  - Requests Redmine taskId from user
  - Validates all changes are committed
  - Generates PR content with security audit results
  - Provides manual PR creation instructions with formatted content

### 2. Pull Request Review (`/review-pull-request`)

**Location**: `plugins/pr-review/commands/review-pull-request.md`

Six-phase comprehensive review workflow:

- **Phase 1**: PR Information Gathering (git/gh commands)
- **Phase 2**: Technology Detection (`technology-detector` agent)
- **Phase 3**: Code Quality Review (`pr-reviewer` agent)
- **Phase 4**: Security Audit (`security-auditor` agent)
- **Phase 5**: Lint & Build Verification (bash commands)
- **Phase 6**: Review Summary & Publishing (consolidation and reporting)

Final recommendation: BLOCKED | REQUEST_CHANGES | COMMENT | APPROVE

### 3. Dependency Health Check (`/check-dependencies`)

**Location**: `plugins/dependency-health/commands/check-dependencies.md`

Four-phase dependency management workflow:

- **Phase 1**: Node.js LTS Version Validation → Commit
- **Phase 2**: Security Vulnerability Resolution → Commit
- **Phase 3**: Package Updates (Minor/Patch) → Commit
- **Phase 4**: Report Generation

Creates 3 separate commits for each update phase.

## Framework Detection Logic

### Technology Detection Priority

The system detects technology by examining `package.json` and project structure:

**Web Frameworks** (check package.json):
- `@angular/core` → Angular web project → Use `angular-developer`
- `react` (without `react-native`) → React project → Use `frontend-developer`

**Mobile Frameworks** (check package.json and files):
- `react-native` → React Native → Use `mobile-developer`
- `expo` → Expo → Use `mobile-developer`
- `@ionic` → Ionic → Use `mobile-developer`
- `pubspec.yaml` file → Flutter → Use `mobile-developer`
- `ios/` and `android/` directories → Native → Use `mobile-developer`

**Ambiguity Handling**:
- Multiple web frameworks → STOP and ask user
- Multiple mobile frameworks → STOP and ask user
- Both web and mobile → STOP and ask user
- No clear indicators → STOP and ask user

### Framework-Specific Adaptations

**Angular Projects** (`angular-developer`):
- Detects Angular version and architecture (standalone vs NgModule)
- Applies version-specific patterns (signals for 16+, control flow for 17+)
- Uses OnPush change detection by default
- Detects state management (NgRx Store, NgRx SignalStore, Akita, NGXS) or uses signals/RxJS
- HttpClient for API integration
- Angular CDK A11y for accessibility

**React Projects** (`frontend-developer`):
- Detects React version and patterns
- Applies framework-specific patterns based on detection
- Detects state management (Redux Toolkit, Zustand, Jotai, Valtio) or uses Zustand default
- Detects styling (Tailwind, SCSS, styled-components) or uses SCSS default
- Detects auth (Auth0, Clerk) or uses custom API-based default

**Mobile Projects** (`mobile-developer`):
- Detects platform (React Native, Flutter, Expo, Ionic, Native iOS/Android)
- Detects state management (Redux Toolkit, Zustand, Riverpod, Bloc, Provider)
- Detects navigation (React Navigation, Flutter Navigator)
- Follows platform design guidelines (HIG for iOS, Material for Android)
- Handles permissions, offline support, app lifecycle

## Environment Setup Requirements

### Mandatory Validations (project-setup agent)

1. **Claude Initialization** (FIRST check):
   - CLAUDE.md must exist in project root
   - If missing: STOPS, user must run `/init`

2. **Git Branch Validation** (SECOND check):
   - Must be on `feature/{taskId}-*` branch
   - STOPS if on `develop` or `main`
   - STOPS if uncommitted changes exist

3. **Environment Configuration**:
   - Node version matches `.nvmrc`
   - `node_modules` installed
   - `.env` file present if `.env.example` exists (retrieve from Shadows/Bitwarden)

4. **ESLint Configuration** (warns if missing):
   - Links to: https://github.com/brooktec/knowledge-base/

### Branch Naming Convention

All feature branches must follow: `feature/{taskId}-brief-description`

Example: `feature/12345-user-authentication`

### Redmine Integration

All pull requests require a Redmine taskId in the format:
- PR title: `[#12345] Feature description`
- Links to Redmine task for traceability

## Common Development Commands

### Installing Plugins (User instructions)

Complete workflow installation:
```
@frontend-orchestration
@frontend-mobile-development
@security-compliance
@git-actions
@pr-review
@dependency-health
```

### Running Workflows

```bash
# Complete feature development workflow
/frontend-feature Implement user authentication with JWT tokens

# Review a pull request
/review-pull-request 42

# Check dependency health
/dependency-health::check-dependencies
```

### Git Workflow

```bash
# Create feature branch
git checkout -b feature/{taskId}-brief-description

# Development workflow handles commits automatically
# Or manual commit if needed
git add .
git commit -m "[#12345] Description"

# PR creation is handled by /frontend-feature workflow
```

## Security Standards

All code goes through comprehensive security auditing:

- **OWASP Top 10** checks (frontend-specific)
- **Authentication security** validation
- **XSS and injection** vulnerability scanning
- **Security headers** validation
- **Dependency vulnerabilities** scanning
- **Sensitive data exposure** checks
- **Mobile-specific**: Biometric auth, secure storage, certificate pinning, code obfuscation

Security findings are categorized:
- CRITICAL: Must fix before merge
- HIGH: Should fix before merge
- MEDIUM: Consider fixing
- LOW: Nice to have

## Testing Standards

All implementations must include:
- **Angular**: Unit tests with TestBed/Jasmine/Jest
- **React**: Jest + React Testing Library or Vitest
- **Mobile**: Platform-appropriate testing (Jest for RN, widget tests for Flutter)
- **Accessibility**: WCAG 2.1 AA compliance for web, platform accessibility for mobile

## Critical Workflow Rules

### STOP Conditions

Workflows STOP and request user action when:
1. CLAUDE.md missing (user must run `/init`)
2. On develop/main branch (user must create feature branch)
3. Branch name doesn't match `feature/{taskId}-*` format
4. Uncommitted changes exist (user must commit or stash)
5. Missing `.env` when template exists (user must get from Shadows)
6. Technology is ambiguous (user must specify framework)
7. Phase 3 security audit completes (user must approve to continue)
8. Deprecated packages found (user must decide on migration)
9. Breaking changes detected (user must approve manual update)

### Approval Checkpoints

**Frontend Feature Workflow**:
- After Phase 3 (Security Audit): User must explicitly approve before PR creation
- User responses: "yes/approve/continue" → proceed | "no/changes" → return to Phase 2

**Dependency Health Workflow**:
- Deprecated packages: User must decide on migration strategy
- Breaking changes: User must approve manual update approach

## File Organization

### Plugin Organization Guidelines

**When to Create Agent Library Plugins** (agents/ only):
- Agent is reusable across multiple workflows
- Agent provides domain-specific expertise (development, security, infrastructure)
- Agent can be invoked with different contexts
- Examples: `shared-agents`, `frontend-mobile-development`, `security-compliance`

**When to Create Workflow Plugins** (commands/ + workflow-specific agents/):
- Providing user-facing workflow automation
- Orchestrating multiple agents in a multi-phase process
- Agents in `agents/` folder are internal implementation details only used by this workflow
- Examples: `frontend-orchestration`, `pr-review`, `dependency-health`

**Deciding Agent Placement**:
```
Is this agent used by multiple workflows?
├─ YES → Put in agent library plugin (shared-agents, domain-agents)
└─ NO  → Put in workflow plugin as internal agent
```

### Plugin Markdown Files

All agent and command definitions are markdown files containing:
- Purpose and objective
- Expected inputs/outputs
- Success criteria
- Error handling
- Coordination notes

### Adding New Agents

1. **Determine if agent is reusable or workflow-specific**:
   - Reusable across workflows → Add to agent library plugin (`shared-agents` or domain-specific)
   - Only used by one workflow → Add to that workflow's plugin as internal agent

2. **Create markdown file** in appropriate plugin's `agents/` directory

3. **Define agent specification**:
   - Purpose and capabilities
   - Behavioral traits and knowledge base
   - Response approach and success criteria
   - Error handling and critical rules

4. **Reference agent in commands** using `plugin-name::agent-name`

### Adding New Commands

1. **Create new workflow plugin** (or add to existing):
   - Create `commands/` directory
   - Create `agents/` directory for workflow-specific agents (if needed)

2. **Create command markdown file** in `commands/` directory:
   - Define complete workflow phases
   - Provide detailed prompts for agent invocations (workflow specification)
   - Specify STOP conditions and approval checkpoints
   - Include expected outputs and context for each phase

3. **Document in README.md**:
   - Add to workflow list
   - Explain dependencies on agent library plugins
   - Provide usage examples

## Tooling Preferences

The system adapts to existing project tooling but has preferred defaults:

### State Management
- **Angular**: Signals/RxJS (adapts to NgRx Store, NgRx SignalStore, Akita, NGXS)
- **React**: Zustand (adapts to Redux Toolkit, Jotai, Valtio)
- **Mobile**: Varies by platform (Redux Toolkit, Zustand, Riverpod, Bloc, Provider)

### Styling
- **Angular**: SCSS (adapts to Tailwind, Angular Material)
- **React**: SCSS (adapts to Tailwind, styled-components, emotion, PostCSS)
- **Mobile**: Platform-specific (adapts to styled-components, NativeWind, Flutter themes)

### Authentication
- **All frameworks**: Custom API-based with JWT/session tokens (adapts to existing auth providers)

**Adaptation Rule**: Always use existing tooling for consistency. Inform user about alternatives only if they provide clear, measurable advantages.

## Company-Specific Conventions

### Brooktec Standards

- **ESLint**: Required for all projects (links to knowledge-base if missing)
- **Environment Variables**: Store in Shadows (Bitwarden)
- **Redmine Integration**: All PRs must reference task ID
- **Branch Protection**: No commits to develop/main directly
- **Security First**: OWASP Top 10 compliance mandatory
- **Accessibility**: WCAG 2.1 AA compliance for web applications

### Knowledge Base

Reference for configuration standards:
https://github.com/brooktec/knowledge-base/

## Version Management

Current plugin versions (as of v0.0.4):
- `shared-agents`: Reusable agents library (NEW in v0.0.4)
  - project-setup (moved from frontend-orchestration)
  - technology-detector (moved from pr-review)
  - code-reviewer (moved and renamed from pr-review::pr-reviewer)
- `frontend-orchestration`: Core workflow orchestrator (Enhanced in v0.0.4 with Phase 2.5)
- `frontend-mobile-development`: Development agents (Angular, React, Mobile)
- `security-compliance`: Security auditing
- `git-actions`: PR management
- `pr-review`: PR review workflow (Simplified in v0.0.4, uses only shared-agents)
- `dependency-health`: Dependency management

## Notes for Claude Code Instances

### When Working on This Repository

1. **This is a plugin marketplace**, not a user project
2. **Plugin organization**:
   - Agent library plugins (agents/ only): `shared-agents`, `frontend-mobile-development`, `security-compliance`
   - Workflow plugins (commands/ + workflow-specific agents/): `frontend-orchestration`, `pr-review`, `git-actions`, `dependency-health`
3. **Editing agents/commands**: Modify markdown files in `plugins/*/agents/` or `plugins/*/commands/`
4. **Moving agents**:
   - If making an agent reusable: Move to `shared-agents/` or domain-specific library plugin
   - Update all command references: `old-plugin::agent` → `new-plugin::agent`
5. **Testing changes**: Plugins are loaded by Claude Code from these markdown files
6. **Documentation**: Keep README.md in sync with plugin capabilities
7. **Versioning**: Update version history in README.md when making changes

### When This Repository is Installed in a User Project

1. **You are orchestrating workflows**, not modifying this repository
2. **Invoke agents** using Task tool with `subagent_type: "plugin-name::agent-name"`
3. **Follow workflow phases** exactly as defined in command markdown files
4. **STOP at checkpoints** and request user approval/input as specified
5. **Adapt to project technology** detected in user's codebase
6. **Never skip validations** defined in project-setup agent

### Common Mistakes to Avoid

- ❌ Don't proceed past approval checkpoints without explicit user approval
- ❌ Don't skip CLAUDE.md validation (must exist before any work)
- ❌ Don't commit to develop/main branches
- ❌ Don't apply breaking changes without user approval
- ❌ Don't guess technology when ambiguous - always ask user
- ❌ Don't skip security audits
- ❌ Don't create PRs without Redmine taskId

### Best Practices

- ✅ Always verify CLAUDE.md exists first
- ✅ Validate git branch before starting work
- ✅ Detect project technology before implementing
- ✅ Ask user when ambiguity exists
- ✅ Stop at approval checkpoints and wait for explicit approval
- ✅ Adapt to existing project tooling
- ✅ Create separate commits for each logical phase
- ✅ Include security audit results in PR descriptions
- ✅ Provide actionable feedback with code examples

### Understanding the Architecture

**Agent Library vs Workflow Plugin**:
- If a plugin has ONLY `agents/` directory → It's a library, agents are meant to be reused
- If a plugin has `commands/` directory → It's a workflow plugin
- Workflow plugins may have `agents/` for internal helpers (not meant for external use)

**Command Prompts vs Agent Files**:
- Detailed prompts in commands are INTENTIONAL, not duplication
- Commands = "WHAT to do in this workflow context" (specification)
- Agents = "HOW to do it" (implementation)
- Same agent can be used differently by different workflows

**Agent Reusability Examples**:
- `shared-agents::project-setup` - Environment validation (used by: frontend-orchestration, can be used by any workflow)
- `shared-agents::technology-detector` - Tech stack detection (used by: pr-review, can be used by any workflow)
- `shared-agents::code-reviewer` - Code quality review (used by: pr-review, potential uses: pre-commit gates, ad-hoc reviews, pre-PR quality checks)
- `frontend-orchestration::requirements-reviewer` - Internal to frontend-feature workflow only
- When in doubt, check if agent is in a library plugin (`shared-agents`, `frontend-mobile-development`, `security-compliance`) or workflow plugin
