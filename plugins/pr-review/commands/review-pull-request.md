# Review Pull Request Command

This command orchestrates a comprehensive multi-phase review of a GitHub pull request, including technology detection, code quality analysis, security auditing, and build verification. The review process adapts to the detected technology stack and provides actionable, structured feedback.

## Command Purpose

Automate and standardize pull request reviews by coordinating multiple specialized agents to ensure:
- Code quality and maintainability
- Technology-specific best practices
- Security compliance (OWASP Top 10)
- Build and lint verification
- Consistent review standards

## Expected Input

The user should provide:
1. **PR Number** (required): The pull request number to review (e.g., `42`, `#123`)
2. **Review Focus** (optional): Specific areas to emphasize (`security`, `performance`, `quality`, `all`)
3. **Repository** (optional): Repository path if not in current directory

**Example invocations:**
- `/review-pull-request 42`
- `/review-pull-request #123 --focus=security`
- `/review-pull-request 89 --focus=all`

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Phase 1: PR Information Gathering                          │
│  - Fetch PR details, changed files, commits                 │
│  - Validate PR exists and is accessible                     │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 2: Technology Detection                              │
│  - Analyze project structure and dependencies               │
│  - Identify frameworks, languages, patterns                 │
│  - Generate technology context for review                   │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 3: Code Quality Review                               │
│  - Analyze code changes with tech-specific lens             │
│  - Check best practices, patterns, maintainability          │
│  - Generate structured feedback with examples               │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 4: Security Audit                                    │
│  - OWASP Top 10 vulnerability scan                          │
│  - Dependency security check                                │
│  - Authentication/authorization review                      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 5: Lint & Build Verification                         │
│  - Run linting on changed files                             │
│  - Execute build process                                    │
│  - Report any errors or warnings                            │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 6: Review Summary & Publishing                       │
│  - Consolidate all findings                                 │
│  - Generate comprehensive report                            │
│  - Post comments to PR (optional)                           │
│  - Provide final recommendation                             │
└─────────────────────────────────────────────────────────────┘
```

## Phase Definitions

---

## Phase 1: PR Information Gathering

### Objective
Fetch and validate pull request details, including changed files, commits, and metadata.

### Process
Use bash commands and git operations to gather PR information.

### Tool Usage
```
Tool: Bash
Purpose: Execute git and gh commands to fetch PR details
```

### Actions

1. **Validate PR Number**
   - Check that PR number is provided
   - Ensure it's a valid integer

2. **Fetch PR Metadata**
   ```bash
   # Using gh CLI if available
   gh pr view <PR_NUMBER> --json number,title,body,author,state,isDraft,labels,headRefName,baseRefName

   # Fallback: git commands
   git fetch origin pull/<PR_NUMBER>/head:pr-<PR_NUMBER>
   git checkout pr-<PR_NUMBER>
   ```

3. **Get Changed Files**
   ```bash
   gh pr diff <PR_NUMBER> --name-only
   # Or: git diff --name-only origin/<base_branch>...HEAD
   ```

4. **Get File Changes with Context**
   ```bash
   gh pr diff <PR_NUMBER>
   # Or: git diff origin/<base_branch>...HEAD
   ```

5. **Collect Commit Information**
   ```bash
   gh pr view <PR_NUMBER> --json commits
   # Or: git log origin/<base_branch>..HEAD --oneline
   ```

### Expected Output
- PR metadata (title, description, author, status)
- List of changed files with line counts
- Full diff of all changes
- Commit history for the PR

### Success Criteria
- PR exists and is accessible
- Changed files list is not empty
- Diff is retrieved successfully

### Error Handling
- **PR Not Found**: Report error, verify PR number and repository
- **No gh CLI**: Fallback to git commands
- **Permission Denied**: Check authentication and repository access
- **Empty Changes**: PR has no file changes, abort review

### Context for Next Phase
Pass to Phase 2:
- List of changed files
- Project root directory
- Technology hints from file extensions

---

## Phase 2: Technology Detection

### Objective
Analyze the project to identify technologies, frameworks, and architectural patterns for targeted review.

### Process
Invoke the technology-detector agent to analyze project structure and dependencies.

### Tool Usage
```
Tool: Task
Subagent: technology-detector
Purpose: Identify tech stack for targeted review
```

### Prompt for Agent
```
Analyze this project to detect the technology stack and architectural patterns.

**Context**: Reviewing PR #<PR_NUMBER> with the following changed files:
<list of changed files>

**Task**: Perform a STANDARD analysis (2-3 minutes) to identify:
1. Primary languages and frameworks
2. Frontend stack (if applicable)
3. Backend stack (if applicable)
4. Build tools and package managers
5. Testing frameworks
6. Key dependencies
7. Architectural patterns

**Output Required**: Provide a structured Technology Detection Report including:
- Project type (Frontend/Backend/Full-Stack/Mobile)
- Primary technologies with confidence levels
- Recommendations for review focus areas

This information will be used to apply technology-specific review criteria.
```

### Expected Output
Technology Detection Report containing:
- Project type classification
- Tech stack with confidence levels
- Framework versions
- Key dependencies
- Review focus recommendations

### Success Criteria
- At least one primary language detected with HIGH confidence
- Project type identified (Frontend/Backend/Full-Stack/Mobile/Library)
- Review recommendations provided

### Error Handling
- **Cannot Detect Technology**: Proceed with generic review
- **Multiple Conflicting Technologies**: Report all, review with multiple lenses
- **Missing Config Files**: Use source code analysis only

### Context for Next Phase
Pass to Phase 3:
- Full technology report
- Framework-specific review rules to apply
- Common pitfalls for detected stack

---

## Phase 3: Code Quality Review

### Objective
Perform comprehensive code quality analysis with technology-specific best practices.

### Process
Invoke the code-reviewer agent with technology context and changed files.

### Tool Usage
```
Tool: Task
Subagent: shared-agents::code-reviewer
Purpose: Analyze code quality, maintainability, and best practices
```

### Prompt for Agent
```
Review the code changes in PR #<PR_NUMBER> for quality, maintainability, and best practices.

**PR Context**:
- Title: <PR_TITLE>
- Author: <PR_AUTHOR>
- Description: <PR_DESCRIPTION>
- Changed Files: <count> files

**Technology Context** (from detection phase):
<technology detection report>

**Changed Files**:
<list of files with +/- line counts>

**Your Task**:
1. Read all changed files to understand the full context
2. Apply technology-specific review criteria based on detected stack
3. Analyze for:
   - Code quality and readability
   - Best practices adherence
   - Performance considerations
   - Maintainability and complexity
   - Testing coverage
   - Documentation completeness
4. Provide structured feedback with:
   - Severity levels (CRITICAL/HIGH/MEDIUM/LOW/NITPICK)
   - File:line references for every issue
   - Current code vs suggested fix examples
   - Rationale and references

**Focus Areas** (based on detected tech):
<tech-specific focus areas from Phase 2>

**Output Required**:
- Detailed findings for each issue
- Code quality review summary
- Positive highlights
- Preliminary recommendation (APPROVE/REQUEST_CHANGES/COMMENT)

Do NOT include security-specific findings (handled in next phase).
Focus on code quality, design, and maintainability.
```

### Expected Output
Structured code review with:
- Categorized findings (Critical/High/Medium/Low/Nitpick)
- Specific file:line references
- Code examples for issues and fixes
- Review summary with recommendation

### Success Criteria
- All changed files analyzed
- At least one finding or explicit "no issues found" statement
- Each finding has file:line reference
- Suggestion provided for each issue

### Error Handling
- **Cannot Read File**: Skip and note in report, continue with other files
- **Binary File**: Note as non-reviewable, continue
- **Too Large**: Review partial content, note limitation

### Context for Next Phase
Pass to Phase 4:
- Code quality findings (to cross-reference with security issues)
- List of files needing security focus
- Database/API files for security review priority

---

## Phase 4: Security Audit

### Objective
Perform comprehensive security analysis focusing on OWASP Top 10 and common vulnerabilities.

### Process
Invoke the security-auditor agent (from security-compliance plugin).

### Tool Usage
```
Tool: Task
Subagent: security-auditor
Purpose: Identify security vulnerabilities and risks
```

### Prompt for Agent
```
Perform a security audit on the code changes in PR #<PR_NUMBER>.

**PR Context**:
- Title: <PR_TITLE>
- Changed Files: <count> files

**Technology Context**:
<relevant security considerations from tech detection>

**Changed Files**:
<list of changed files>

**Code Quality Findings** (from Phase 3):
<any findings that might have security implications>

**Your Task**:
Perform a comprehensive security audit focusing on:
1. OWASP Top 10 vulnerabilities
2. Authentication and authorization issues
3. Input validation and sanitization
4. SQL/NoSQL injection vulnerabilities
5. XSS and CSRF vulnerabilities
6. Sensitive data exposure
7. Insecure dependencies
8. API security
9. Cryptography usage
10. Error handling and information disclosure

**Output Required**:
- Security findings with CRITICAL/HIGH/MEDIUM/LOW severity
- Specific vulnerability descriptions
- Exploit scenarios (if applicable)
- Remediation recommendations with code examples
- Security compliance summary
- BLOCK/WARN/PASS recommendation

**Important**:
- Prioritize findings that could lead to data breach or unauthorized access
- Provide actionable remediation steps
- Reference OWASP or CWE identifiers where applicable
```

### Expected Output
Security audit report with:
- Vulnerabilities categorized by severity
- Exploit scenarios and impact assessment
- Remediation recommendations
- Compliance status
- Security recommendation (BLOCK/WARN/PASS)

### Success Criteria
- All changed files scanned for security issues
- OWASP Top 10 checks completed
- Each vulnerability has remediation guidance
- Overall security recommendation provided

### Error Handling
- **Security Agent Unavailable**: Perform basic security checks manually
- **Cannot Scan File**: Note limitation, continue with other files
- **False Positives**: Mark as potential issues for human review

### Context for Next Phase
Pass to Phase 5:
- Security findings to include in final report
- Critical security blocks that prevent merge
- Files that failed security checks

---

## Phase 5: Lint & Build Verification

### Objective
Verify that code passes linting rules and builds successfully.

### Process
Execute project-specific lint and build commands.

### Tool Usage
```
Tool: Bash
Purpose: Run linting and build commands
```

### Actions

1. **Detect Lint Commands**
   ```bash
   # Check package.json scripts
   if [ -f package.json ]; then
     npm run lint 2>&1
   fi

   # Python projects
   if [ -f setup.py ] || [ -f pyproject.toml ]; then
     pylint <changed_files> 2>&1
     flake8 <changed_files> 2>&1
   fi

   # Other languages
   # Java: mvn checkstyle:check
   # Go: golangci-lint run
   ```

2. **Run Build**
   ```bash
   # Node.js projects
   if [ -f package.json ]; then
     npm run build 2>&1
   fi

   # Python projects
   if [ -f setup.py ]; then
     python setup.py build 2>&1
   fi

   # Java projects
   if [ -f pom.xml ]; then
     mvn compile 2>&1
   fi
   ```

3. **Check for Errors**
   - Parse output for errors vs warnings
   - Count issues by severity
   - Extract file:line references

4. **Run Type Checking** (if applicable)
   ```bash
   # TypeScript
   npx tsc --noEmit 2>&1

   # Python
   mypy <changed_files> 2>&1
   ```

### Expected Output
- Lint results (errors and warnings count)
- Build status (success/failure)
- Type checking results (if applicable)
- List of lint/build errors with file:line references

### Success Criteria
- Lint command executed (or noted as unavailable)
- Build command executed (or noted as unavailable)
- Results parsed and categorized

### Error Handling
- **No Lint Script**: Note in report, not a blocker
- **Build Fails**: Mark as CRITICAL issue, must fix
- **Lint Errors**: Include in report, assess severity
- **Command Not Found**: Try alternative commands, note limitation

### Context for Next Phase
Pass to Phase 6:
- Lint error count and details
- Build status and errors
- Type checking results

---

## Phase 6: Review Summary & Publishing

### Objective
Consolidate all findings into a comprehensive report and optionally publish to PR.

### Process
Aggregate results from all phases and generate structured summary.

### Actions

1. **Consolidate Findings**
   ```
   - Merge code quality findings (Phase 3)
   - Merge security findings (Phase 4)
   - Add lint/build findings (Phase 5)
   - Remove duplicates
   - Sort by severity and file
   ```

2. **Generate Summary Report**
   ```markdown
   # PR #<NUMBER> Comprehensive Review Report

   ## PR Information
   - **Title**: <title>
   - **Author**: <author>
   - **Branch**: <head> → <base>
   - **Files Changed**: X files (+Y lines, -Z lines)
   - **Commits**: N commits

   ## Technology Stack
   <summary from Phase 2>

   ## Review Summary

   ### Overall Recommendation: [APPROVE / REQUEST CHANGES / COMMENT / BLOCKED]

   ### Findings Overview
   - ⛔ Critical Issues: X (must fix)
   - ⚠️  High Priority: Y (should fix)
   - ℹ️  Medium Priority: Z (consider fixing)
   - 💡 Low Priority: W (nice to have)
   - 🎨 Nitpicks: V (optional)

   ### Security Status: [PASS / WARN / BLOCK]
   - Security vulnerabilities: X
   - OWASP Top 10 checks: [status]

   ### Build Status: [PASS / FAIL]
   - Lint errors: X
   - Build errors: Y
   - Type errors: Z

   ## Blocking Issues
   [List of critical issues that must be fixed before merge]

   ## Detailed Findings

   ### Code Quality Issues
   [Findings from Phase 3]

   ### Security Issues
   [Findings from Phase 4]

   ### Lint/Build Issues
   [Findings from Phase 5]

   ## Positive Highlights
   [Good patterns, well-tested code, clear documentation]

   ## Recommendations
   1. [Actionable next steps]
   2. [Follow-up items]
   3. [Learning resources]

   ## Next Steps for Author
   [Clear guidance on what to do next]
   ```

3. **Determine Final Recommendation**
   ```
   Logic:
   - BLOCKED: Critical security issues OR build fails
   - REQUEST CHANGES: High priority code quality issues OR security warnings
   - COMMENT: Medium/Low issues only, code is mergeable
   - APPROVE: No significant issues found
   ```

4. **Ask User for Publishing Preference**
   ```
   Review complete! I've analyzed <N> files and found:
   - X critical issues
   - Y high priority issues
   - Z total findings

   **Final Recommendation**: [APPROVE / REQUEST CHANGES / COMMENT / BLOCKED]

   Would you like me to:
   1. Post this review as comments on the PR (requires gh CLI)
   2. Save the review to a file (review-summary.md)
   3. Display the full report here
   4. Focus on specific findings

   What would you prefer?
   ```

5. **Publish if Requested**
   ```bash
   # Post comments to PR using gh CLI
   gh pr review <PR_NUMBER> --comment --body "<review_summary>"

   # Post individual inline comments
   for each finding with file:line:
     gh pr comment <PR_NUMBER> --body "<finding>" \
       --path <file> --line <line>

   # Or save to file
   echo "<full_report>" > review-pr-<PR_NUMBER>.md
   ```

### Expected Output
- Comprehensive review summary
- Final recommendation (APPROVE/REQUEST_CHANGES/COMMENT/BLOCKED)
- Published comments (if requested)
- Clear next steps for PR author

### Success Criteria
- All findings consolidated
- Recommendation is justified by findings
- Report is clear and actionable
- User knows what to do next

### Error Handling
- **gh CLI Not Available**: Save to file instead
- **Permission Error**: Display report, ask user to post manually
- **API Rate Limit**: Wait and retry, or save locally

### Coordination Notes
- This is the final phase
- Wait for user decision before posting
- Provide multiple output options
- Ensure report is accessible and actionable

---

## Usage Examples

### Example 1: Basic PR Review
```bash
/review-pull-request 42
```
**Flow**:
- Fetches PR #42 details
- Detects React + TypeScript project
- Reviews 8 changed files
- Runs security audit
- Checks lint and build
- Generates summary with 3 high priority issues
- Recommends REQUEST CHANGES

### Example 2: Security-Focused Review
```bash
/review-pull-request 123 --focus=security
```
**Flow**:
- Emphasizes security analysis
- Deep OWASP Top 10 scan
- Dependency vulnerability check
- Lighter code quality review
- Recommends BLOCKED due to SQL injection

### Example 3: Quick Quality Check
```bash
/review-pull-request 89 --focus=quality
```
**Flow**:
- Focuses on code quality and maintainability
- Standard security checks
- Detailed best practices analysis
- Recommends COMMENT with suggestions

## Success Metrics

A successful PR review workflow:
1. ✅ Completes all 6 phases without errors
2. ✅ Identifies critical issues (if any exist)
3. ✅ Provides actionable feedback with code examples
4. ✅ Generates clear recommendation (APPROVE/REQUEST/COMMENT/BLOCKED)
5. ✅ Delivers report in user's preferred format
6. ✅ Author understands what to do next

## Failure Scenarios & Handling

| Scenario | Phase | Handling |
|----------|-------|----------|
| PR not found | Phase 1 | Abort with clear error message |
| No changed files | Phase 1 | Abort, nothing to review |
| Tech detection fails | Phase 2 | Continue with generic review |
| File read error | Phase 3 | Skip file, note in report |
| Security agent unavailable | Phase 4 | Perform basic manual checks |
| Build fails | Phase 5 | Mark as CRITICAL, include in report |
| gh CLI not available | Phase 6 | Save to file instead |

## Integration with Existing Plugins

### Uses:
- **technology-detector** (shared-agents plugin)
- **code-reviewer** (shared-agents plugin)
- **security-auditor** (security-compliance plugin)

### Coordinates With:
- **git-actions**: For PR management and publishing
- **frontend-developer**: For implementation suggestions
- **mobile-developer**: For React Native/Flutter specific reviews

### Output For:
- PR authors: Clear feedback and next steps
- Reviewers: Comprehensive analysis to supplement human review
- CI/CD: Pass/fail signals for automated gates

---

## Implementation Notes

### Prerequisites
- Git repository with remote configured
- Pull request exists on remote
- User has read access to PR
- Optional: gh CLI installed for PR interaction

### Configuration
None required - adapts to detected technology stack

### Timing
- **Quick Review**: 2-4 minutes (small PRs, <100 lines)
- **Standard Review**: 5-10 minutes (medium PRs, 100-500 lines)
- **Deep Review**: 10-20 minutes (large PRs, >500 lines, complex changes)

### Extensibility
- Add new technology-specific review agents
- Customize security rules per project
- Add project-specific lint configurations
- Extend with custom review phases

---

**End of Review Pull Request Command Definition**
