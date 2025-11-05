Orchestrate comprehensive pull request review with technology detection, code quality analysis, security auditing, and build verification.

[Extended thinking: This workflow coordinates multiple specialized agents to provide thorough, technology-aware PR reviews. The process adapts to the detected tech stack (React, Angular, Mobile, Node.js backend, Python, etc.) and provides actionable feedback with severity categorization. The review ensures code quality, security compliance (OWASP Top 10), and build verification before providing a final recommendation (APPROVE/REQUEST CHANGES/COMMENT/BLOCKED).]

## Workflow Configuration

### Expected Input

**Required**:
- **PR Number**: The pull request number to review (e.g., `42`, `#123`)

**Optional**:
- **Review Focus**: Specific areas to emphasize (`--focus=security`, `--focus=performance`, `--focus=quality`, `--focus=all`)
- **Repository Path**: Repository if not in current directory

**Example Invocations**:
```bash
/review-pull-request 42
/review-pull-request #123 --focus=security
/review-pull-request 89 --focus=all --repo=/path/to/repo
```

### Parameter Parsing
- Extract PR number from $ARGUMENTS (with or without # prefix)
- Parse --focus flag if present (default: all)
- Parse --repo flag if present (default: current directory)
- Validate PR number is valid integer

---

## Phase 1: PR Information Gathering

### Step 1. Fetch PR Details
- Use Bash tool to execute `gh pr view` commands
- Command: `gh pr view <PR_NUMBER> --json title,author,state,isDraft,additions,deletions,changedFiles,commits,reviews`
- Expected output: JSON with PR metadata
- Context: PR number extracted from $ARGUMENTS
- Action: Parse JSON and store PR details. If PR not found or not accessible, STOP workflow with error message

### Step 2. Get Changed Files List
- Use Bash tool to execute `gh pr diff` command
- Command: `gh pr diff <PR_NUMBER> --name-only`
- Expected output: List of changed file paths
- Context: PR number
- Action: Store list of changed files for analysis. If no files changed, STOP workflow (nothing to review)

### Step 3. Fetch Full Diff
- Use Bash tool to execute `gh pr diff` command
- Command: `gh pr diff <PR_NUMBER>`
- Expected output: Complete diff of all changes
- Context: PR number and changed files list
- Action: Store full diff for code review. Parse diff to identify added/modified/deleted lines

### Step 4. Get Commit History
- Use Bash tool to execute `gh pr view` command
- Command: `gh pr view <PR_NUMBER> --json commits`
- Expected output: List of commits in PR
- Context: PR number
- Action: Store commit history for context. Extract commit messages for review summary

---

## Phase 2: Technology Detection

### Step 1. Analyze Project Technology Stack
- Use Task tool with subagent_type="shared-agents::technology-detector"
- Prompt: "Analyze this project to detect the technology stack and architectural patterns.

  **Context**: Reviewing PR #<PR_NUMBER> with the following changed files:
  <LIST_OF_CHANGED_FILES>

  **Analysis Required**:
  - Primary language and framework (React, Angular, Node.js, Python, Java, etc.)
  - Backend framework if applicable (Express, NestJS, Django, FastAPI, Spring Boot)
  - State management approach (Redux, Zustand, etc.)
  - Styling approach (Tailwind, SCSS, styled-components, etc.)
  - Testing framework (Jest, Vitest, pytest, JUnit, etc.)
  - Database/ORM if detectable
  - Build tools and configuration

  **Output Required**:
  - Primary technology stack summary
  - Framework versions if detectable
  - Key architectural patterns
  - Review focus areas based on technology (e.g., React hooks, Angular lifecycle, Node.js async patterns)
  - Technology-specific risks or concerns"

- Expected output: Technology detection report with framework, version, tooling, and recommended review focus areas
- Context: Project root directory, package.json, changed files list
- Action: Use detected technology to inform code review in Phase 3. If detection fails, continue with generic review

---

## Phase 3: Code Quality Review

### Step 1. Review Code Changes with Technology-Specific Lens
- Use Task tool with subagent_type="shared-agents::code-reviewer"
- Prompt: "Review the code changes in PR #<PR_NUMBER> for quality, maintainability, and best practices.

  **PR Context**:
  - Title: <PR_TITLE>
  - Author: <PR_AUTHOR>
  - Changed Files: <COUNT> files (<ADDITIONS> additions, <DELETIONS> deletions)

  **Technology Context** (from Phase 2):
  <TECHNOLOGY_DETECTION_REPORT>

  **Changed Files to Review**:
  <LIST_OF_CHANGED_FILES_WITH_LINE_COUNTS>

  **Full Diff**:
  <COMPLETE_DIFF_FROM_PHASE_1>

  **Your Task**:
  1. Read and analyze all changed files
  2. Apply technology-specific review criteria based on detected stack
  3. Identify code quality issues categorized by severity:
     - CRITICAL: Major bugs, security issues, broken functionality
     - HIGH: Significant problems affecting maintainability or performance
     - MEDIUM: Code smells, minor bugs, improvement opportunities
     - LOW: Style issues, minor optimizations
     - NITPICK: Formatting, naming conventions (if significant)

  4. For each finding, provide:
     - Severity level
     - File and line number (file:line format)
     - Issue description
     - Current code example (from diff)
     - Suggested fix with code example
     - Rationale and impact

  5. Identify positive patterns and good practices

  **Technology-Specific Checks**:
  <INSERT_RELEVANT_CHECKS_BASED_ON_DETECTED_TECHNOLOGY>

  **Focus Areas** (if --focus flag provided):
  <FOCUS_AREA_FROM_PARAMETERS>

  **Output Format**:
  Structured review with categorized findings, specific file:line references, and code examples"

- Expected output: Detailed code review report with categorized findings (CRITICAL/HIGH/MEDIUM/LOW/NITPICK), file:line references, current code vs suggested fixes, rationale for each finding, positive highlights
- Context: Full diff from Phase 1, technology context from Phase 2, PR metadata
- Action: Parse findings and categorize by severity. Count critical/high/medium/low issues for final recommendation

---

## Phase 4: Security Audit

### Step 1. Perform Security Analysis
- Use Task tool with subagent_type="security-compliance::security-auditor"
- Prompt: "Perform security audit on PR #<PR_NUMBER> changes.

  **PR Context**:
  - Title: <PR_TITLE>
  - Author: <PR_AUTHOR>
  - Technology Stack: <FROM_PHASE_2>

  **Changed Files**:
  <LIST_OF_CHANGED_FILES>

  **Full Diff**:
  <COMPLETE_DIFF_FROM_PHASE_1>

  **Security Assessment Required**:
  1. OWASP Top 10 vulnerability scan:
     - Injection (SQL, NoSQL, Command, XSS)
     - Broken Authentication
     - Sensitive Data Exposure
     - XML External Entities (XXE)
     - Broken Access Control
     - Security Misconfiguration
     - Cross-Site Scripting (XSS)
     - Insecure Deserialization
     - Using Components with Known Vulnerabilities
     - Insufficient Logging & Monitoring

  2. Technology-specific security checks based on detected stack
  3. Authentication and authorization validation
  4. Input validation and sanitization
  5. Dependency security (if package changes detected)
  6. Secrets or credentials exposure
  7. API security (rate limiting, authentication)

  **Output Required**:
  - Security findings categorized by severity (CRITICAL/HIGH/MEDIUM/LOW)
  - Specific file:line references for each issue
  - Vulnerability description and potential impact
  - Remediation steps with code examples
  - OWASP category reference where applicable"

- Expected output: Security audit report with vulnerability findings categorized by severity, OWASP references, remediation steps, and overall security posture assessment
- Context: Code changes from Phase 1, technology stack from Phase 2
- Action: Parse security findings. Flag CRITICAL security issues for BLOCKED recommendation. Count security issues for final report

---

## Phase 5: Lint & Build Verification

### Step 1. Run Linting on Changed Files
- Use Bash tool to execute linting commands based on detected technology
- Commands (technology-dependent):
  - **Node.js/TypeScript**: `npx eslint <CHANGED_FILES> --format=json` or `npm run lint`
  - **Python**: `pylint <CHANGED_FILES> --output-format=json` or `flake8 <CHANGED_FILES>`
  - **Java**: `./gradlew checkstyleMain` or `mvn checkstyle:check`
- Expected output: Linting results with errors and warnings
- Context: Changed files list, detected technology
- Action: Parse linting output. Count errors and warnings. If linter not configured, skip with note in report

### Step 2. Run Build Process
- Use Bash tool to execute build command
- Commands (technology-dependent):
  - **Node.js**: `npm run build` or `yarn build`
  - **Python**: `python setup.py build` or check import errors
  - **Java**: `mvn compile` or `./gradlew build --no-test`
- Expected output: Build success/failure with error messages if any
- Context: Project root, detected build tool
- Action: Capture build result. If build fails, mark as CRITICAL issue for final recommendation. If build not configured, attempt to verify no syntax errors

### Step 3. Check Type Errors (if applicable)
- Use Bash tool to run type checker
- Commands (if applicable):
  - **TypeScript**: `npx tsc --noEmit`
  - **Python**: `mypy <CHANGED_FILES>`
  - **Java**: Covered by build step
- Expected output: Type errors if any
- Context: Changed files, detected language
- Action: Report type errors. Consider type errors as HIGH severity issues

---

## Phase 6: Review Summary & Publishing

### Step 1. Consolidate All Findings
- Aggregate findings from all phases:
  - Code quality issues from Phase 3 (categorized by severity)
  - Security vulnerabilities from Phase 4 (categorized by severity)
  - Lint errors/warnings from Phase 5
  - Build errors from Phase 5
  - Type errors from Phase 5
- Count issues by severity: CRITICAL, HIGH, MEDIUM, LOW, NITPICK
- Identify blocking issues (CRITICAL security vulnerabilities, build failures)
- Extract positive highlights from code review

### Step 2. Generate Final Recommendation
- Decision logic:
  - **BLOCKED**: IF any CRITICAL security issues OR build failures
  - **REQUEST CHANGES**: IF any HIGH severity issues (and no blocking issues)
  - **COMMENT**: IF only MEDIUM/LOW/NITPICK issues
  - **APPROVE**: IF no significant issues found
- Generate recommendation rationale based on issue counts and severity

### Step 3. Format Comprehensive Report
- Structure report with sections:
  1. **PR Summary**: Title, author, changed files count, additions/deletions
  2. **Technology Stack**: Detected frameworks and patterns from Phase 2
  3. **Review Recommendation**: APPROVE/REQUEST CHANGES/COMMENT/BLOCKED with rationale
  4. **Critical Issues**: Blocking issues that must be addressed (if any)
  5. **High Priority Issues**: Important issues to address (if any)
  6. **Code Quality Findings**: Categorized findings from Phase 3
  7. **Security Findings**: Vulnerabilities from Phase 4
  8. **Build & Lint Results**: Errors and warnings from Phase 5
  9. **Positive Highlights**: Good patterns identified
  10. **Next Steps**: Actionable items for PR author

- Format findings with:
  - Severity badge (🔴 CRITICAL, 🟠 HIGH, 🟡 MEDIUM, 🟢 LOW, ⚪ NITPICK)
  - File:line reference (clickable if possible)
  - Issue description
  - Current code vs suggested fix (code blocks)
  - Rationale

### Step 4. Present Report to User
- Display formatted report in console/output
- Save report to file: `.claude/pr-review-<PR_NUMBER>.md`
- Summary statistics:
  - Total issues: X (CRITICAL: X, HIGH: X, MEDIUM: X, LOW: X, NITPICK: X)
  - Files reviewed: X
  - Lines changed: +X / -X
  - Final recommendation: APPROVE/REQUEST CHANGES/COMMENT/BLOCKED

### Step 5. Optionally Post to PR (if gh CLI available)
- Ask user: "Would you like to post this review as a comment on PR #<PR_NUMBER>? (yes/no)"
- If yes and gh CLI available:
  - Use Bash: `gh pr comment <PR_NUMBER> --body "$(cat .claude/pr-review-<PR_NUMBER>.md)"`
  - Confirm comment posted successfully
- If no or gh CLI not available:
  - Inform user: "Review saved to .claude/pr-review-<PR_NUMBER>.md. You can manually post to PR."

---

## Success Criteria

### Overall Success
- [ ] PR information gathered successfully (all details, files, diff)
- [ ] Technology stack correctly detected
- [ ] Code quality review completed with categorized findings
- [ ] Security audit completed with vulnerability assessment
- [ ] Build and lint verification completed (or documented as N/A)
- [ ] Comprehensive report generated with actionable recommendations
- [ ] Final recommendation provided (APPROVE/REQUEST/COMMENT/BLOCKED)

### Phase 1 Criteria
- [ ] PR exists and is accessible
- [ ] Changed files list retrieved (non-empty)
- [ ] Full diff obtained
- [ ] Commit history fetched

### Phase 2 Criteria
- [ ] Technology stack identified (at minimum: language and primary framework)
- [ ] Review focus areas determined based on technology
- [ ] Technology-specific checks prepared for Phase 3

### Phase 3 Criteria
- [ ] All changed files reviewed
- [ ] Findings categorized by severity (CRITICAL/HIGH/MEDIUM/LOW/NITPICK)
- [ ] File:line references provided for all findings
- [ ] Code examples included (current vs suggested)
- [ ] Positive patterns identified

### Phase 4 Criteria
- [ ] OWASP Top 10 checks performed
- [ ] Technology-specific security checks completed
- [ ] Security findings categorized by severity
- [ ] Remediation steps provided for each vulnerability
- [ ] Critical security issues flagged for BLOCKED recommendation

### Phase 5 Criteria
- [ ] Linting attempted (or documented as not configured)
- [ ] Build attempted (or documented as not configured)
- [ ] Type checking attempted if applicable
- [ ] All errors and warnings captured and reported

### Phase 6 Criteria
- [ ] All findings consolidated from all phases
- [ ] Final recommendation calculated correctly
- [ ] Comprehensive report generated and formatted
- [ ] Report saved to file
- [ ] User provided with clear next steps

---

## Example Usage

### Example 1: Standard Review
```bash
/review-pull-request 42
```
**Flow**:
- Fetches PR #42 details, files, and diff
- Detects technology (e.g., React + TypeScript + Express backend)
- Reviews code quality with React/TypeScript best practices
- Performs security audit checking for XSS, injection, auth issues
- Runs ESLint and TypeScript compiler
- Generates report with findings: 2 HIGH, 5 MEDIUM, 3 LOW issues
- Recommendation: REQUEST CHANGES (due to HIGH issues)

### Example 2: Security-Focused Review
```bash
/review-pull-request 123 --focus=security
```
**Flow**:
- Fetches PR #123 details
- Detects technology (e.g., Django + PostgreSQL)
- Performs in-depth security audit (OWASP Top 10, SQL injection, auth)
- Reviews code quality with focus on security patterns
- Runs pylint and build
- Generates report emphasizing security findings
- Recommendation: BLOCKED (CRITICAL SQL injection vulnerability found)

### Example 3: Quick Quality Check
```bash
/review-pull-request 89 --focus=quality
```
**Flow**:
- Fetches PR #89 details
- Detects technology (e.g., Angular + RxJS)
- Performs thorough code quality review (Angular patterns, RxJS usage, performance)
- Lighter security check (basic validation)
- Runs ng lint and ng build
- Generates report focused on code quality and maintainability
- Recommendation: COMMENT (only LOW/NITPICK issues, overall good quality)

---

## Error Handling

### Phase 1 Errors

**Error**: PR Not Found
- **Cause**: Invalid PR number or PR doesn't exist
- **Resolution**: STOP workflow. Display: "Error: Pull request #<NUMBER> not found. Please check the PR number and try again."
- **Example**: `/review-pull-request 99999` when PR doesn't exist

**Error**: No Changed Files
- **Cause**: PR has no file changes (empty PR)
- **Resolution**: STOP workflow. Display: "Pull request #<NUMBER> has no changed files. Nothing to review."

**Error**: gh CLI Not Available
- **Cause**: GitHub CLI not installed or not authenticated
- **Resolution**: STOP workflow. Display: "Error: gh CLI not available. Please install: https://cli.github.com/ and authenticate with: gh auth login"

### Phase 2 Errors

**Error**: Technology Detection Fails
- **Cause**: Unable to determine project technology from files
- **Resolution**: Continue with generic review. Note in report: "Technology detection failed, using generic review criteria."

### Phase 3 Errors

**Error**: Cannot Read Changed File
- **Cause**: File path invalid, file deleted, or permission issue
- **Resolution**: Skip file. Note in report: "Could not read <FILE_PATH>: <ERROR>. Skipped in review."

### Phase 4 Errors

**Error**: Security Agent Unavailable
- **Cause**: security-compliance plugin not installed
- **Resolution**: Perform basic manual security checks. Note in report: "Security agent unavailable, performed manual basic checks only."

### Phase 5 Errors

**Error**: Build Fails
- **Cause**: Code changes introduced build errors
- **Resolution**: Mark as CRITICAL issue. Include build error output in report. Recommendation: BLOCKED

**Error**: Linter Not Configured
- **Cause**: No linter configuration found in project
- **Resolution**: Skip linting. Note in report: "Linting skipped: No linter configuration found."

### Phase 6 Errors

**Error**: Cannot Post Comment to PR
- **Cause**: gh CLI fails or authentication issue
- **Resolution**: Save report to file. Display: "Could not post comment to PR. Review saved to .claude/pr-review-<NUMBER>.md"

---

## Coordination Notes

### Phase Flow
- Phases 1-5 are sequential and cannot be parallelized (each depends on previous)
- Phase 1 must succeed (PR fetching) before proceeding to Phase 2
- Phase 2 technology detection informs Phase 3 code review focus
- Phase 3 and 4 can theoretically run in parallel but kept sequential for clarity
- Phase 5 verification depends on detected technology from Phase 2
- Phase 6 consolidates all previous phases

### Conditional Phase Execution
- If PR not found in Phase 1: STOP entire workflow
- If technology detection fails in Phase 2: Continue with generic review
- If security agent unavailable in Phase 4: Perform basic manual checks
- If build/lint not configured in Phase 5: Skip with note, don't fail workflow

### Retry Logic
- No automatic retries for Phase 1 (user must fix PR number)
- Phase 2 technology detection: 1 retry with more generic search if first attempt fails
- Phase 5 build: No retries (build failure is a finding, not a workflow error)

### Focus Flag Behavior
- `--focus=security`: Emphasize security in Phase 4, lighter code quality in Phase 3
- `--focus=quality`: Emphasize code quality in Phase 3, lighter security in Phase 4
- `--focus=performance`: Add performance-specific checks in Phase 3
- `--focus=all` or no flag: Full comprehensive review (default)

---

## Integration with Existing Plugins

### Uses (Agents from Other Plugins)
- **shared-agents::technology-detector**: Phase 2 technology detection
- **shared-agents::code-reviewer**: Phase 3 code quality analysis
- **security-compliance::security-auditor**: Phase 4 security audit

### Coordinates With
- **git-actions**: For PR management and publishing (potential future integration)
- **frontend-mobile-development agents**: Indirectly via code-reviewer's technology-specific checks

### Outputs For
- **PR Authors**: Clear, actionable feedback on what to fix
- **Team Leads**: Quality metrics and review summary
- **CI/CD**: Can be integrated into automation pipeline

---

## When to Use This Command

**Use when:**
- Reviewing a pull request before merging
- Performing quality gates in CI/CD
- Getting a second opinion on complex changes
- Ensuring security compliance before deployment
- Standardizing review criteria across team

**Don't use when:**
- PR is draft (wait until ready for review)
- Major refactoring without tests (test first)
- Emergency hotfix (post-review documentation instead)

---

## Implementation Notes

### Timing Expectations
- **Quick PR** (< 5 files, < 200 lines): 2-3 minutes
- **Standard PR** (5-20 files, < 1000 lines): 5-10 minutes
- **Large PR** (> 20 files, > 1000 lines): 15-30 minutes

### Review Quality
- Leverages specialized agents for best results
- Technology-aware: adapts to React, Angular, Node.js, Python, etc.
- Consistent standards across all reviews
- Actionable feedback with code examples

### Extensibility
- Easy to add new focus areas (--focus=accessibility, --focus=i18n)
- Can integrate additional verification steps (test coverage, performance benchmarks)
- Pluggable security checks based on project needs

---

**End of Review Pull Request Command**
