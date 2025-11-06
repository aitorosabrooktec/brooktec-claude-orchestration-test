# Simplified Workflow Diagrams

High-level overview diagrams showing only the main phases and possible outcomes for each orchestrator.

**Each diagram includes**:
- Main workflow phases
- Decision points and conditional branches
- Possible outcomes (Success, STOP, Skip)
- **Agent invocations**: Which specialized agents are used in each phase

## Available Diagrams

### 1. [Frontend Feature](./frontend-feature-simplified.md)
**Command**: `/frontend-feature [--skip-pr]`

**Flow**: Setup → Development → Code Quality → Security → **Approval** → PR Creation

**Outcomes**:
- ✅ Feature complete (with or without PR)
- ❌ STOP: Environment issues, ambiguous framework
- 🛑 User approval: Request changes / Approve without PR / Approve with PR

---

### 2. [Backend Feature](./backend-feature-simplified.md)
**Command**: `/backend-feature [--skip-pr] [--skip-tests]`

**Flow**: Setup → Development → API Testing → Quality & Security → **Approval** → PR Creation

**Outcomes**:
- ✅ API complete, tested (with or without PR)
- ❌ STOP: Environment issues, ambiguous framework, test failures
- ⏭️ SKIP: API testing (--skip-tests)
- 🛑 User approval: Request changes / Approve without PR / Approve with PR

---

### 3. [Pull Request Review](./review-pull-request-simplified.md)
**Command**: `/review-pull-request <PR_NUMBER>`

**Flow**: PR Info → Tech Detection → Code Quality → Security → Lint & Build → Summary

**Outcomes**:
- 🔴 **BLOCKED**: Critical security or build failures
- 🟠 **REQUEST CHANGES**: High priority issues
- 🟡 **COMMENT**: Medium/low suggestions
- 🟢 **APPROVE**: No significant issues

---

### 4. [Create Pull Request](./create-pull-request-simplified.md)
**Command**: `/create-pull-request`

**Flow**: Prerequisites → Information → Content Generation → PR Creation

**Outcomes**:
- ✅ **Automated**: PR created with gh CLI or GitHub MCP
- ✅ **Manual**: Step-by-step instructions provided
- ❌ STOP: Uncommitted changes, missing taskId

---

### 5. [Dependency Health Check](./check-dependencies-simplified.md)
**Command**: `/check-dependencies`

**Flow**: Node.js LTS → Security → Updates → Report → Verification

**Outcomes**:
- ✅ **PASSED**: All checks successful, 3 commits created
- ⚠️ **PASSED WITH WARNINGS**: Non-critical issues
- ❌ **FAILED**: Critical failures (npm install, build)
- 🔴 **CRITICAL**: Rollback option offered
- 📄 Report: dependency-health-report.md

---

### 6. [Backend Test Generation](./create-backend-tests-simplified.md)
**Command**: `/create-backend-tests --target <path> [flags]`

**Flow**: Setup → Analysis → Generation → Execution → Coverage → Quality → **Approval** → PR

**Outcomes**:
- ✅ Tests passing with coverage met (with or without PR)
- ❌ STOP: No target, test failures, low coverage
- 🛑 User approval: Request changes / Approve without PR / Approve with PR

---

## Key Symbols

### Decision Points
- 🛑 **Approval Checkpoint**: Workflow stops for user decision
- ❓ **Conditional**: Depends on flags or conditions

### Outcomes
- ✅ **Success**: Workflow completed successfully
- ❌ **STOP**: User action required before proceeding
- 🔴 **CRITICAL**: Critical failure requiring immediate attention
- ⚠️ **Warning**: Non-critical issue, workflow continues

### Status Indicators (PR Review)
- 🔴 **BLOCKED**: Must fix before merge
- 🟠 **REQUEST CHANGES**: Should fix before merge
- 🟡 **COMMENT**: Consider fixing
- 🟢 **APPROVE**: Ready to merge

### Flags
- ⏭️ **Skip**: Phase skipped due to flag

## Quick Comparison

| Workflow | Phases | Approval Checkpoint | Conditional PR | Retry Logic |
|----------|--------|---------------------|----------------|-------------|
| Frontend Feature | 4 | Yes (after Phase 3) | --skip-pr | No |
| Backend Feature | 6 | Yes (after Phase 4) | --skip-pr, --skip-tests | Yes (tests) |
| PR Review | 6 | No | N/A | No |
| Create PR | 4 | No | N/A | Yes (automation fallback) |
| Dependency Health | 5 | No | N/A | No |
| Backend Tests | 8 | Yes (after Phase 6) | --skip-pr | Yes (tests, coverage) |

## Common Patterns

### All Development Workflows Include:
1. **Setup & Requirements** validation
2. **Technology Detection** for framework-specific implementation
3. **Quality & Security Review** before completion
4. **Approval Checkpoint** where user reviews and decides
5. **Conditional PR Creation** based on user choice and --skip-pr flag

### User Approval Options:
When workflows reach an approval checkpoint, user has 3 choices:
1. **Request Changes** → Return to development phase for modifications
2. **Approve without PR** → End workflow, feature/tests complete
3. **Approve and Create PR** → Continue to PR creation (if --skip-pr not set)

### STOP Conditions:
All workflows stop for:
- Missing prerequisites (environment, dependencies)
- Unclear requirements or ambiguous technology
- Failed critical operations (tests, builds, security)
- User approval or decision needed

## Agent Architecture

Workflows use specialized agents from different plugin libraries:

### Shared Agents (Reusable)
- **`shared-agents::project-setup`**: Environment validation, git checks
- **`shared-agents::requirements-reviewer`**: Requirements clarity validation
- **`shared-agents::technology-detector`**: Framework and tool detection
- **`shared-agents::code-reviewer`**: Code quality review and best practices

### Development Agents (Domain-Specific)
- **`frontend-mobile-development::angular-developer`**: Angular implementation
- **`frontend-mobile-development::react-developer`**: React implementation
- **`frontend-mobile-development::mobile-developer`**: React Native, Flutter, Expo, Ionic
- **`backend-development::node-developer`**: Express.js and NestJS implementation

### Specialized Agents
- **`security-compliance::security-auditor`**: OWASP Top 10 security audit
- **`test-orchestration::test-generator`**: Backend test generation
- **`git-actions::pull-request-manager`**: PR creation and management
- **`dependency-health::dependency-health-checker`**: Dependency health analysis

## Usage Examples

```bash
# Frontend development
/frontend-feature Implement login with OAuth

# Backend development with all features
/backend-feature Implement user authentication API

# Backend without tests (utilities)
/backend-feature --skip-tests Implement data export utils

# Review a pull request
/review-pull-request 42

# Generate backend tests
/create-backend-tests --target src/services/user.service.ts

# Check dependency health
/check-dependencies
```

## Detailed Diagrams

For detailed step-by-step diagrams with all decision points, error handling, and agent invocations, see the [full diagrams](../) in the parent directory.

---

**Note**: These simplified diagrams focus on the high-level flow and main decision points. For implementation details, agent specifications, and comprehensive error handling, refer to the detailed diagrams and CLAUDE.md documentation.
