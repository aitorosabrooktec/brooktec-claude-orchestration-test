Orchestrate comprehensive dependency health check with Node.js LTS validation, security vulnerability resolution, safe package updates, and automated verification.

[Extended thinking: This command orchestrates a complete dependency health check workflow that ensures project dependencies are secure, up-to-date, and using the correct Node.js LTS version. The workflow executes methodical phase-by-phase checks: Node.js version validation, security vulnerability resolution, safe package updates (minor/patch only), comprehensive report generation, and automated verification (npm install, linting, building, testing). Each change phase creates a separate git commit for easy rollback. The agent will STOP and request user guidance when encountering deprecated packages, breaking changes, dependency conflicts, or any ambiguous situations. This ensures safe, non-breaking updates while maintaining comprehensive documentation of all changes.]

## Workflow Configuration

### Command Flags

**No flags required** - This command runs with sensible defaults:
- Automatically validates Node.js LTS version
- Resolves critical and high-severity vulnerabilities
- Updates to latest minor/patch versions only (no breaking changes)
- Creates 3 separate commits (one per phase)
- Generates comprehensive markdown report
- Runs verification checks (npm install, lint, build, test)

**Project Prerequisites**:
- `package.json` file must exist
- `npm` must be installed
- Must be in project root directory
- Git working directory should be clean (or user accepts new commits)

---

## Phase 1: Node.js LTS Version Validation

### Step 1. Validate and Update Node.js Version
- Use Task tool with subagent_type="dependency-health::dependency-health-checker"
- Prompt: "Execute Phase 1: Node.js LTS Version Validation

  **Phase 1 Tasks**:
  1. **Check Current Node.js Version**:
     - Run `node --version` to get current version
     - Check `.nvmrc` file if it exists
     - Determine if current version is LTS

  2. **Validate Against LTS**:
     - Run `npx npuc --init` to check LTS recommendation
     - Compare current version with recommended LTS
     - Identify if update needed

  3. **Update .nvmrc (if needed)**:
     - Update `.nvmrc` to recommended LTS version
     - Document version change in commit message

  4. **Create Commit**:
     - Stage changes: `.nvmrc`
     - Commit with message: `chore: validate and update Node.js to LTS version`
     - Include version details in commit body

  5. **Verify Changes**:
     - Confirm `.nvmrc` contains correct LTS version
     - Verify commit was created successfully

  **Safety Checks**:
  - If Node.js not installed: STOP and report error
  - If `.nvmrc` doesn't exist: Create it with LTS version
  - If already on LTS: Document 'no change needed' but still create commit for consistency

  **Output Requirements**:
  - Node.js version before/after
  - LTS version recommended
  - Whether update was needed
  - Commit SHA created"

- Expected output: Node.js version validation complete, .nvmrc updated if needed, commit created with format "chore: validate and update Node.js to LTS version", verification status
- Context: Current Node.js version, existing .nvmrc file, package.json project type
- Action: Ensure project uses Node.js LTS version. Create commit even if no changes needed (for consistency across workflow). STOP if Node.js not installed

---

## Phase 2: Security Vulnerability Resolution

### Step 1. Audit and Resolve Security Vulnerabilities
- Use Task tool with subagent_type="dependency-health::dependency-health-checker"
- Prompt: "Execute Phase 2: Security Vulnerability Resolution

  **Phase 2 Tasks**:
  1. **Run Security Audit**:
     - Execute `npm audit --json` to get vulnerability report
     - Parse JSON output for structured data
     - Categorize vulnerabilities by severity: critical, high, moderate, low
     - Count total vulnerabilities by severity

  2. **Identify Deprecated Packages**:
     - Check for deprecation warnings in npm output
     - List all deprecated packages with alternatives
     - Mark deprecated packages with ❌ for report

  3. **Resolve Critical and High Vulnerabilities**:
     - Run `npm audit fix` to automatically fix vulnerabilities
     - Check which vulnerabilities were fixed
     - Identify any remaining unfixed vulnerabilities
     - For unfixed vulnerabilities:
       - Check if update available without breaking changes
       - If breaking changes required: STOP and ask user for guidance
       - Document blocked vulnerabilities with ⚠️

  4. **Avoid Breaking Changes**:
     - Before applying any fix, check package version jump
     - If major version change detected: STOP and request user approval
     - Only apply fixes that update to minor/patch versions

  5. **Handle Deprecated Packages**:
     - For each deprecated package:
       - Identify recommended alternative
       - STOP and ask user: 'Package X is deprecated. Recommended alternative: Y. Migrate now? (yes/no)'
       - If yes: Guide migration process
       - If no: Document in report with ❌ and continue

  6. **Create Commit**:
     - Stage changes: `package.json`, `package-lock.json`
     - Commit with message: `chore: resolve security vulnerabilities in dependencies`
     - Include vulnerability count and CVE numbers in commit body

  7. **Verify Changes**:
     - Re-run `npm audit` to confirm fixes applied
     - Check remaining vulnerabilities
     - Verify package-lock.json updated
     - Confirm commit created successfully

  **Safety Checks**:
  - STOP if breaking changes required for vulnerability fix
  - STOP if deprecated package needs migration
  - Never apply major version updates without approval
  - Document all blocked vulnerabilities

  **Output Requirements**:
  - Total vulnerabilities found (by severity)
  - Vulnerabilities resolved (count and CVE IDs)
  - Deprecated packages identified with alternatives
  - Blocked vulnerabilities with reasons
  - Commit SHA created"

- Expected output: Security audit complete, critical/high vulnerabilities resolved, deprecated packages identified, remaining vulnerabilities documented with ⚠️, commit created with format "chore: resolve security vulnerabilities in dependencies"
- Context: package.json dependencies, npm audit output, CVE database, commit history
- Action: Resolve all fixable security vulnerabilities without breaking changes. STOP and request user guidance for deprecated packages or breaking change requirements. Document all blocked items

---

## Phase 3: Package Updates (Minor/Patch Versions)

### Step 1. Update Packages to Latest Minor/Patch Versions
- Use Task tool with subagent_type="dependency-health::dependency-health-checker"
- Prompt: "Execute Phase 3: Package Updates (Minor/Patch Versions)

  **Phase 3 Tasks**:
  1. **Check for Available Updates**:
     - Run `npm outdated --json` to get update report
     - Parse JSON for structured data
     - List all packages with available updates
     - Categorize by update type: patch, minor, major

  2. **Filter Safe Updates**:
     - Identify packages with minor/patch updates only
     - Exclude packages requiring major version updates
     - Check for packages with breaking changes:
       - Read CHANGELOG if available
       - Check package repository for breaking change notes
       - Check npm package page for migration guides

  3. **Apply Minor/Patch Updates**:
     - For each safe update:
       - Run `npm update <package>` to update to latest minor/patch
       - Verify update applied successfully
       - Check for any dependency conflicts
     - If conflicts detected: STOP and request user guidance

  4. **Handle Major Updates**:
     - For packages requiring major updates:
       - Document with ⚠️ (blocked by breaking changes)
       - Note current version and available version
       - Link to migration guide if available
       - STOP if user wants to proceed with major update
       - Otherwise: Document in report for manual action later

  5. **Verify Package Integrity**:
     - Run `npm ls` to check dependency tree
     - Verify no broken dependencies
     - Check for peer dependency warnings
     - If issues found: Attempt to resolve or STOP for guidance

  6. **Create Commit**:
     - Stage changes: `package.json`, `package-lock.json`
     - Commit with message: `chore: update dependencies to latest minor/patch versions`
     - Include package update count in commit body
     - List updated packages with version changes

  7. **Verify Changes**:
     - Re-run `npm outdated` to confirm updates applied
     - Check remaining outdated packages (major updates only)
     - Verify package-lock.json updated correctly
     - Confirm commit created successfully

  **Safety Checks**:
  - Never apply major version updates automatically
  - STOP if breaking changes detected
  - STOP if dependency conflicts arise
  - Document all blocked updates

  **Output Requirements**:
  - Total packages updated (count)
  - List of updated packages with version changes
  - Blocked packages (major updates) with reasons
  - Dependency conflicts (if any)
  - Commit SHA created"

- Expected output: Package updates complete, minor/patch versions updated, major updates documented with ⚠️, no breaking changes applied, commit created with format "chore: update dependencies to latest minor/patch versions"
- Context: npm outdated output, package changelogs, package-lock.json, commit history
- Action: Update all packages to latest minor/patch versions safely. Block major updates and document for manual review. STOP if breaking changes or conflicts detected

---

## Phase 4: Comprehensive Report Generation

### Step 1. Generate Detailed Markdown Report
- Use Task tool with subagent_type="dependency-health::dependency-health-checker"
- Prompt: "Execute Phase 4: Comprehensive Report Generation

  **Phase 4 Tasks**:
  1. **Compile Report Data**:
     - Gather results from Phase 1 (Node.js version)
     - Gather results from Phase 2 (vulnerabilities)
     - Gather results from Phase 3 (package updates)
     - Prepare summary statistics

  2. **Create Report Structure** (Markdown format):
     ```markdown
     # Dependency Health Check Report

     **Generated**: [timestamp]
     **Project**: [project name from package.json]
     **Working Directory**: [current directory]

     ---

     ## Summary

     | Phase | Status | Changes |
     |-------|--------|---------|
     | Node.js LTS Validation | ✅/❌ | [version change or 'no change'] |
     | Security Vulnerabilities | ✅/⚠️ | [X vulnerabilities resolved] |
     | Package Updates | ✅/⚠️ | [X packages updated] |
     | Verification | ✅/❌ | [npm install, lint, build, test results] |

     ---

     ## Phase 1: Node.js Version Validation

     | Item | Before | After | Status |
     |------|--------|-------|--------|
     | Node.js Version | [version] | [version] | ✅/❌ |
     | .nvmrc | [version] | [version] | ✅/❌ |
     | LTS Status | [yes/no] | [yes/no] | ✅ |

     **Commit**: [commit SHA and message]

     ---

     ## Phase 2: Security Vulnerabilities

     ### Vulnerabilities Resolved

     | CVE | Package | Severity | Version Before | Version After | Status |
     |-----|---------|----------|----------------|---------------|--------|
     | [CVE-ID] | [package] | [critical/high] | [version] | [version] | ✅ |

     ### Deprecated Packages

     | Package | Current Version | Alternative | Status |
     |---------|----------------|-------------|--------|
     | [package] | [version] | [alternative] | ❌ |

     ### Vulnerabilities Blocked

     | CVE | Package | Severity | Reason | Status |
     |-----|---------|----------|--------|--------|
     | [CVE-ID] | [package] | [severity] | [breaking changes] | ⚠️ |

     **Commit**: [commit SHA and message]

     ---

     ## Phase 3: Package Updates

     ### Packages Updated

     | Package | Version Before | Version After | Update Type | Status |
     |---------|----------------|---------------|-------------|--------|
     | [package] | [version] | [version] | patch/minor | ✅ |

     ### Updates Blocked (Breaking Changes)

     | Package | Current | Available | Reason | Migration Guide | Status |
     |---------|---------|-----------|--------|-----------------|--------|
     | [package] | [version] | [version] | [breaking changes] | [link] | ⚠️ |

     **Commit**: [commit SHA and message]

     ---

     ## Phase 4: Verification

     | Check | Command | Result | Status |
     |-------|---------|--------|--------|
     | Dependencies Install | npm install | [success/failure] | ✅/❌ |
     | Linting | npm run lint | [passed/failed] | ✅/❌ |
     | Build | npm run build | [passed/failed] | ✅/❌ |
     | Tests | npm test | [passed/failed] | ✅/❌ |

     **Notes**: [any issues encountered during verification]

     ---

     ## Recommendations

     ### Immediate Actions Required
     - [Action 1: e.g., Migrate from deprecated package X to Y]
     - [Action 2: e.g., Manually update package Z with breaking changes]

     ### Future Maintenance
     - [Recommendation 1: e.g., Schedule weekly dependency checks]
     - [Recommendation 2: e.g., Monitor package X for security updates]

     ### Blocked Items Summary
     - [X deprecated packages need migration]
     - [Y vulnerabilities blocked by breaking changes]
     - [Z major updates available but blocked]

     ---

     ## Git Commits Created

     1. [commit SHA] - chore: validate and update Node.js to LTS version
     2. [commit SHA] - chore: resolve security vulnerabilities in dependencies
     3. [commit SHA] - chore: update dependencies to latest minor/patch versions

     ---

     **Report End**
     ```

  3. **Generate Report File**:
     - Write report to `dependency-health-report.md` in project root
     - Ensure proper markdown formatting
     - Include all tables with alignment
     - Add status indicators: ✅ (success), ❌ (deprecated/failed), ⚠️ (blocked/warning)

  4. **Display Report Summary**:
     - Show condensed summary to user
     - Highlight immediate actions required
     - List blocked items needing manual intervention

  **Output Requirements**:
  - Complete markdown report written to file
  - Report includes all phases with detailed tables
  - Status indicators clearly show success/blocked/failed
  - Recommendations section populated with actionable items
  - Git commit SHAs documented
  - CVE IDs included for vulnerabilities"

- Expected output: Comprehensive markdown report generated at `dependency-health-report.md`, all phases documented with tables, status indicators (✅/❌/⚠️), recommendations provided, git commits listed
- Context: Phase 1-3 results, commit SHAs, package.json changes, vulnerability data
- Action: Create detailed documentation of all changes, blocked items, and recommendations for manual actions

---

## Phase 5: Automated Verification

### Step 1. Verify Changes with Automated Checks
- Use Task tool with subagent_type="dependency-health::dependency-health-checker"
- Prompt: "Execute Phase 5: Automated Verification

  **Phase 5 Tasks**:
  1. **Verify Dependencies Install**:
     - Remove `node_modules` directory
     - Remove `package-lock.json`
     - Run `npm install` from clean state
     - Verify successful installation
     - Check for any warnings or errors
     - If errors: STOP and report issue

  2. **Run Linting**:
     - Check if `npm run lint` script exists in package.json
     - If exists: Run `npm run lint`
     - Capture output and check exit code
     - If lint errors: Document but continue (may be pre-existing)
     - If lint script missing: Skip and note in report

  3. **Run Build**:
     - Check if `npm run build` script exists in package.json
     - If exists: Run `npm run build`
     - Capture output and check exit code
     - If build fails: STOP and report issue (critical failure)
     - If build script missing: Skip and note in report

  4. **Run Tests**:
     - Check if `npm test` script exists in package.json
     - If exists: Run `npm test`
     - Capture output and check exit code
     - If tests fail: Document failures and check if pre-existing
     - Compare with previous test results if available
     - If new failures introduced: STOP and report issue

  5. **Update Report**:
     - Add Phase 5 verification results to report
     - Update verification table with results
     - Document any failures or warnings
     - Add notes about skipped checks

  6. **Final Validation**:
     - Verify all 3 commits exist in git log
     - Verify report file created
     - Check git status (should be clean except for report)
     - Confirm no uncommitted changes beyond report

  **Safety Checks**:
  - STOP if npm install fails (critical)
  - STOP if build fails (critical)
  - Document but continue if lint/tests fail (may be pre-existing)
  - STOP if new test failures introduced by updates

  **Output Requirements**:
  - npm install result (success/failure)
  - Lint result (passed/failed/skipped)
  - Build result (passed/failed/skipped)
  - Test result (passed/failed/skipped with counts)
  - Updated report with verification section
  - Final status: PASSED / PASSED WITH WARNINGS / FAILED"

- Expected output: All verification checks complete, npm install successful, build passing, lint/test results documented, report updated with Phase 5 results, final status provided
- Context: Updated package.json and package-lock.json, node_modules state, package.json scripts availability
- Action: Verify all changes work correctly by running automated checks. STOP if critical failures (npm install, build). Document non-critical issues (lint, tests) but continue to allow user review

---

## Success Criteria

### Overall Success
- [ ] Node.js LTS version validated and updated (Phase 1)
- [ ] Security vulnerabilities resolved (no breaking changes) (Phase 2)
- [ ] Packages updated to latest minor/patch versions (Phase 3)
- [ ] Comprehensive report generated with all documentation (Phase 4)
- [ ] Automated verification passed (npm install, build) (Phase 5)
- [ ] 3 separate git commits created (one per change phase)
- [ ] All blocked items documented with status indicators

### Phase 1 Criteria
- [ ] Current Node.js version checked
- [ ] LTS version identified via npx npuc --init
- [ ] .nvmrc updated to LTS version (or verified already LTS)
- [ ] Commit created: "chore: validate and update Node.js to LTS version"
- [ ] Changes verified

### Phase 2 Criteria
- [ ] Security audit completed (npm audit)
- [ ] Critical and high vulnerabilities identified
- [ ] Vulnerabilities resolved without breaking changes
- [ ] Deprecated packages identified with alternatives
- [ ] Blocked vulnerabilities documented with ⚠️
- [ ] Commit created: "chore: resolve security vulnerabilities in dependencies"
- [ ] Audit re-run to verify fixes

### Phase 3 Criteria
- [ ] Package updates checked (npm outdated)
- [ ] Minor/patch updates applied
- [ ] Major updates blocked and documented with ⚠️
- [ ] No breaking changes applied
- [ ] Dependency tree verified (npm ls)
- [ ] Commit created: "chore: update dependencies to latest minor/patch versions"
- [ ] Updates verified

### Phase 4 Criteria
- [ ] Comprehensive report generated (dependency-health-report.md)
- [ ] Report includes all phases with detailed tables
- [ ] Status indicators present (✅/❌/⚠️)
- [ ] Git commit SHAs documented
- [ ] CVE IDs included for vulnerabilities
- [ ] Recommendations section populated
- [ ] Blocked items clearly listed

### Phase 5 Criteria
- [ ] npm install successful (clean state)
- [ ] Build passed (if build script exists)
- [ ] Lint results documented (if lint script exists)
- [ ] Test results documented (if test script exists)
- [ ] Report updated with verification results
- [ ] Final status provided (PASSED/WARNINGS/FAILED)

---

## Coordination Notes

### Workflow Position
- This command can be run standalone at any time
- Typically invoked during routine maintenance or before major features
- Can be scheduled weekly/monthly for proactive maintenance
- Should be run before releases to ensure production-ready dependencies

### Prerequisites
- Must have package.json in project root
- Must have npm installed
- Git working directory should be clean (or user accepts new commits)
- Node.js should be installed (will validate version)

### Commit Strategy
- Creates 3 separate commits (one per change phase)
- Allows easy rollback of individual phases if needed
- Each commit has descriptive message and detailed body
- Commits include version changes, CVE IDs, package counts

### User Interaction Points
- STOP when deprecated package needs migration decision
- STOP when breaking changes required for vulnerability fix
- STOP when major package update requested
- STOP when dependency conflicts detected
- STOP when verification checks fail critically

### Safety Features
- Never applies breaking changes without approval
- Verifies each phase before progressing
- Documents all blocked items with reasons
- Creates comprehensive audit trail via commits and report
- Allows easy rollback via separate commits

### Integration Points
- **frontend-orchestration**: Can be called before Phase 1 (project setup)
- **backend-orchestration**: Can be called before Phase 1 (project setup)
- **Redmine**: Report can be attached to maintenance tasks
- **CI/CD**: Can be integrated into maintenance pipelines

---

## Example Usage

### Example 1: Standard Workflow (All Updates Successful)
```bash
/check-dependencies
```

**Flow**:
1. Validates Node.js version: v18.x → v20.x LTS (updated .nvmrc)
2. Resolves 5 critical vulnerabilities (no deprecated packages)
3. Updates 12 packages to latest minor versions
4. Generates comprehensive report with all changes
5. Verifies: npm install ✅, build ✅, lint ✅, tests ✅

**Output**:
```
✅ Phase 1 Complete: Node.js updated to 20.x LTS
✅ Phase 2 Complete: 5 critical vulnerabilities resolved
✅ Phase 3 Complete: 12 packages updated
✅ Phase 4 Complete: Report generated at dependency-health-report.md
✅ Phase 5 Complete: All verification checks passed

3 commits created:
- [abc123] chore: validate and update Node.js to LTS version
- [def456] chore: resolve security vulnerabilities in dependencies
- [ghi789] chore: update dependencies to latest minor/patch versions
```

### Example 2: Manual Action Required (Deprecated Package)
```bash
/check-dependencies
```

**Flow**:
1. Validates Node.js version: Already on LTS (no change)
2. Resolves 3 vulnerabilities, identifies deprecated package 'request'
3. STOPS and asks user: "Package 'request' is deprecated. Recommended: 'axios'. Migrate now?"
4. User responds: "No, later"
5. Continues: Updates 8 packages, documents 'request' with ❌
6. Generates report with recommendation to migrate
7. Verifies: All checks pass

**Output**:
```
✅ Phase 1 Complete: Node.js already on LTS (no change)
⚠️ Phase 2 Partial: 3 vulnerabilities resolved, 1 deprecated package found
   ❌ 'request' deprecated → Use 'axios' (marked for manual migration)
✅ Phase 3 Complete: 8 packages updated
⚠️ Recommendations: 1 deprecated package needs migration

See dependency-health-report.md for details
```

### Example 3: Breaking Changes Blocked
```bash
/check-dependencies
```

**Flow**:
1. Validates Node.js version: v20.x LTS (already up to date)
2. Resolves 2 critical vulnerabilities
3. Identifies webpack v5 → v6 update available (breaking changes)
4. Blocks major update, documents with ⚠️
5. Updates other 10 packages successfully
6. Generates report with migration guide link for webpack
7. Verifies: All checks pass

**Output**:
```
✅ Phase 1 Complete: Node.js already on LTS
✅ Phase 2 Complete: 2 critical vulnerabilities resolved
⚠️ Phase 3 Partial: 10 packages updated, 1 blocked by breaking changes
   ⚠️ webpack v5→v6 blocked (migration guide: https://...)

See dependency-health-report.md for manual update instructions
```

---

## Error Handling

### Phase 1 Errors

**Error**: Node.js Not Installed
- **Cause**: Node.js binary not found
- **Resolution**: STOP workflow. Display: "Error: Node.js is not installed. Please install Node.js LTS from https://nodejs.org/"
- **Example**: User tries to run on machine without Node.js

**Error**: npx Command Failed
- **Cause**: npuc tool execution failed
- **Resolution**: STOP workflow. Display: "Error: Failed to validate LTS version with npx npuc. Check npm installation and network connectivity."
- **Example**: Network issues prevent npx from downloading npuc

### Phase 2 Errors

**Error**: npm audit Fails
- **Cause**: Network issues or npm registry unavailable
- **Resolution**: STOP workflow. Display: "Error: npm audit failed. Check network connectivity and npm registry status."
- **Example**: npm registry is down or behind firewall

**Error**: Vulnerability Requires Breaking Change
- **Cause**: Only fix available requires major version update
- **Resolution**: STOP and ask user. Display: "Critical vulnerability in package X requires major update (breaking changes). Options: 1) Apply and manually fix breaks, 2) Document and defer"
- **Example**: express v4 vulnerability only fixed in v5

**Error**: Deprecated Package Migration Required
- **Cause**: Package is deprecated and needs replacement
- **Resolution**: STOP and ask user. Display: "Package X is deprecated. Recommended alternative: Y. Migrate now? (yes/no/later)"
- **Example**: 'request' package is deprecated, recommend 'axios'

### Phase 3 Errors

**Error**: Package Update Causes Dependency Conflict
- **Cause**: Updated package has incompatible peer dependencies
- **Resolution**: STOP and display conflict details. Ask user: "Package X update conflicts with Y. Options: 1) Skip update, 2) Update both, 3) Resolve manually"
- **Example**: React Router update requires React major version bump

**Error**: npm update Fails
- **Cause**: Network issues or corrupted package
- **Resolution**: STOP workflow. Display: "Error: Failed to update package X. Details: [error message]. Check npm logs and package availability."
- **Example**: Package version no longer available on registry

**Error**: Dependency Tree Broken
- **Cause**: Updates created inconsistent dependency state
- **Resolution**: STOP and rollback Phase 3 commit. Display: "Error: Dependency tree is broken after updates. Rolling back Phase 3. Run npm ls to diagnose."
- **Example**: npm ls shows UNMET DEPENDENCY errors

### Phase 5 Errors

**Error**: npm install Fails (Critical)
- **Cause**: Corrupted package-lock.json or incompatible dependencies
- **Resolution**: STOP workflow. Display: "CRITICAL: npm install failed after updates. This indicates a serious issue. Rollback commits? (yes/no)"
- **Example**: Package updates created unresolvable dependency conflicts

**Error**: Build Fails (Critical)
- **Cause**: Updated packages introduced breaking changes in build
- **Resolution**: STOP workflow. Display: "CRITICAL: Build failed after updates. Details: [build errors]. Rollback commits? (yes/no)"
- **Example**: TypeScript package update changed types, breaking compilation

**Error**: Tests Fail (Warning)
- **Cause**: Updated packages changed behavior, breaking tests
- **Resolution**: Continue but warn. Display: "WARNING: X tests failed after updates. Review test failures to determine if issues are from updates or pre-existing. Details in report."
- **Example**: Jest update changed mock behavior, tests need adjustment

---

## When to Use This Command

**Use when:**
- Starting routine maintenance (weekly/monthly schedule)
- Before beginning major feature development
- After security vulnerability notifications
- Onboarding to new project (understand dependency state)
- Before production releases (ensure clean dependencies)
- After long periods without updates (technical debt reduction)

**Don't use when:**
- In middle of active feature development (may disrupt work)
- When git working directory has uncommitted changes (create clean state first)
- During production incidents (focus on resolution first)
- If major refactoring planned soon (updates may conflict)

**Standalone Usage:**
Can run anytime as standalone command:
```bash
/check-dependencies
```

**Programmatic Usage:**
```
Use Command tool with command="dependency-health::check-dependencies"
```

---

## Implementation Notes

### Timing
- **Quick execution**: 5-10 minutes (if all updates straightforward)
- **Extended execution**: 15-30 minutes (if deprecated packages or conflicts)

### Report Format
- Markdown format for easy reading and version control
- Tables for structured data (versions, vulnerabilities, updates)
- Status indicators (✅/❌/⚠️) for visual clarity
- Recommendations section for actionable next steps
- Git commit SHAs for audit trail

### Commit Conventions
- Phase 1: "chore: validate and update Node.js to LTS version"
- Phase 2: "chore: resolve security vulnerabilities in dependencies"
- Phase 3: "chore: update dependencies to latest minor/patch versions"
- Each commit includes detailed body with changes documented
- Follows conventional commit format for automated changelog generation

### Safety Philosophy
- Conservative approach: Only apply safe, non-breaking updates automatically
- User approval required for any ambiguous or breaking changes
- Comprehensive documentation of all blocked items
- Easy rollback via separate commits per phase
- Verification phase ensures changes don't break project

---

**End of Dependency Health Check Command**
