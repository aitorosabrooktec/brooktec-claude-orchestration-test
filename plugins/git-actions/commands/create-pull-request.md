Orchestrate pull request creation with Redmine integration, security audit inclusion, and standardized formatting.

[Extended thinking: This command orchestrates the pull request creation process, ensuring all changes are committed, the branch is properly named and pushed, and a comprehensive PR is created with proper Redmine task linking, security audit results, and standardized formatting. The process validates prerequisites, generates professional PR content, and guides through either manual or automated PR creation using GitHub CLI or GitHub MCP when available.]

## Workflow Configuration

### Expected Input

**Required**:
- **Feature Description**: Description of the feature for context (provided in $ARGUMENTS)

**Optional**:
- **Redmine TaskId**: Can be provided upfront or requested during workflow (e.g., `#12345` or `12345`)
- **Base Branch**: Target branch for PR (default: `develop`)

**Example Invocations**:
```bash
# Will request taskId during workflow
/create-pull-request Add user authentication feature with JWT tokens

# With taskId provided
/create-pull-request #12345 Add user authentication feature with JWT tokens
```

### Configuration Options
- `redmine_url`: Base URL for Redmine (default: auto-detect from project or use brooktec.com/redmine)
- `default_base_branch`: Base branch for PRs (default: "develop")
- `auto_add_reviewers`: Automatically add team reviewers (default: false, future feature)
- `pr_template`: Use custom PR template (default: standard Brooktec template)

---

## Phase 1: Prerequisites Validation

### Step 1. Validate Git Repository State
- Use Task tool with subagent_type="git-actions::pull-request-manager"
- Prompt: "Validate pull request prerequisites for: $ARGUMENTS.

  **Prerequisites Validation**:
  - Check that all changes are committed (git status should be clean)
  - Verify branch is pushed to remote
  - Confirm branch name follows feature/{taskId}-* format
  - Check if GitHub CLI (gh) is available for automated creation
  - Check if GitHub MCP is available as alternative automation

  **If Prerequisites Fail**:
  - Uncommitted changes: Guide user to commit all changes first
  - Branch not pushed: Instruct how to push branch to remote (git push -u origin branch-name)
  - Branch naming convention: Warn if doesn't follow feature/{taskId}-* format

  **Tool Detection**:
  - Check gh CLI: Run 'gh --version' and 'gh auth status'
  - Check GitHub MCP: Look for GitHub MCP server availability
  - Report available automation options"

- Expected output: Prerequisites validation status, list of any issues to resolve, available automation methods (gh CLI, GitHub MCP, or manual)
- Context: Current git branch, repository status, uncommitted changes, installed tools
- Action: Ensure environment is ready for PR creation. STOP workflow if critical issues exist (uncommitted changes, branch not pushed)

---

## Phase 2: Information Gathering

### Step 1. Collect PR Information
- Use Task tool with subagent_type="git-actions::pull-request-manager"
- Prompt: "Gather information for pull request: $ARGUMENTS.

  **Required Information**:
  1. **Redmine TaskId**:
     - Check if provided in $ARGUMENTS (format: #12345 or 12345)
     - If not provided, STOP and ask user: 'Please provide the Redmine taskId for this feature (e.g., #12345 or 12345)'
     - Validate format (should be numeric, with or without #)

  2. **Feature Summary**:
     - Extract from $ARGUMENTS or feature description
     - Brief, clear description of what was implemented

  3. **Changes Overview**:
     - High-level list of changes made
     - Key components added/modified
     - New dependencies or configuration changes

  4. **Security Audit Results**:
     - Summary from Phase 3 security audit (if available from previous workflow)
     - Security status: PASSED / WARNINGS / CRITICAL ISSUES
     - Key security findings
     - If no security audit available: Note as 'Security audit not performed'

  5. **Testing Approach**:
     - What testing was performed (unit tests, integration tests, E2E)
     - Test coverage if available
     - Manual testing scenarios

  6. **Breaking Changes** (if applicable):
     - List any breaking changes
     - Migration steps required
     - Affected endpoints or functionality

  7. **Screenshots/Demo** (if applicable):
     - Links or references to visual changes
     - Demo videos or GIFs

  **Context to Include**:
  - Branch name (contains taskId)
  - Git commit history for PR
  - Feature requirements from previous workflow phases (if available)
  - Implementation details from development phase (if available)"

- Expected output: Complete PR information package including validated taskId, feature summary, changes list, security audit summary, testing info, and any additional relevant information
- Context: Feature implementation from previous workflow, security audit results (if available), branch name, git history
- Action: Compile all necessary information for comprehensive PR description. If taskId not provided, STOP and request it from user before proceeding

---

## Phase 3: PR Content Generation

### Step 1. Generate Professional PR Content
- Use Task tool with subagent_type="git-actions::pull-request-manager"
- Prompt: "Generate pull request content for: $ARGUMENTS using the gathered information.

  **Generate the Following**:

  1. **Commit Message Template** (if final commit needed):
     - Format: `[#taskId] Brief description of changes`
     - Example: `[#12345] Add user authentication with JWT`
     - Keep under 72 characters if possible

  2. **PR Title**:
     - Format: `[#taskId] Feature description`
     - Example: `[#12345] Add user authentication with JWT and protected routes`
     - Clear, descriptive, follows taskId convention
     - Max 100 characters

  3. **Comprehensive PR Description** (Markdown format):

     ```markdown
     ## Related Task
     **Redmine**: [#taskId](https://redmine.brooktec.com/issues/taskId)

     ## Description
     [Feature summary - what was implemented and why]

     ## Changes Made
     - [Key change 1]
     - [Key change 2]
     - [Key change 3]

     ## Security
     **Status**: [PASSED / WARNINGS / CRITICAL ISSUES / NOT AUDITED]

     [Security audit summary:
     - Key findings
     - Vulnerabilities addressed
     - Security measures implemented]

     ## Testing
     **Approach**:
     - [Unit tests: X tests added/updated]
     - [Integration tests: Y scenarios covered]
     - [Manual testing: Key scenarios tested]

     **Coverage**: [X% if available]

     ## Screenshots/Demo
     [Links or embedded images if applicable, otherwise 'N/A']

     ## Breaking Changes
     [List breaking changes if any, otherwise 'None']

     ## Additional Notes
     [Any other relevant information]

     ## Checklist
     - [ ] Code follows project style guidelines
     - [ ] Self-review completed
     - [ ] Documentation updated
     - [ ] Tests added/updated
     - [ ] All tests passing
     - [ ] Security audit completed
     - [ ] No console errors/warnings
     - [ ] Accessibility checked (WCAG 2.1 AA)
     - [ ] Responsive design verified
     - [ ] Redmine task linked
     ```

  **Ensure**:
  - Professional, clear language
  - Proper markdown formatting
  - All security findings summarized (or note if not audited)
  - Redmine link is correct format
  - Checklist reflects actual completion status
  - Changes list is specific and actionable"

- Expected output: Three formatted outputs: (1) Commit message template, (2) PR title, (3) Complete PR description in markdown format, all ready to use
- Context: Gathered PR information from Phase 2, taskId, security audit results, branch information
- Action: Generate professional, comprehensive PR content following Brooktec PR template standards

---

## Phase 4: PR Creation

### Step 1. Create Pull Request Using Best Available Method
- Use Task tool with subagent_type="git-actions::pull-request-manager"
- Prompt: "Create pull request using generated content: $ARGUMENTS.

  **Creation Method Detection** (from Phase 1):
  - Check if GitHub CLI (gh) is installed and authenticated
  - Check if GitHub MCP is available
  - Determine: Automated (preferred) or Manual process

  **METHOD 1: AUTOMATED CREATION** (if gh CLI available):

  1. Prepare PR creation command:
     ```bash
     gh pr create \\
       --title '[#taskId] Feature description' \\
       --body '<PR_DESCRIPTION_FROM_PHASE_3>' \\
       --base develop \\
       --head <CURRENT_BRANCH>
     ```

  2. Execute command using Bash tool

  3. Capture output and extract PR URL

  4. Confirm PR was created successfully

  5. Display to user:
     - ✅ Pull Request Created Successfully!
     - PR URL: <URL>
     - Next Steps:
       * Add reviewers if needed
       * Update Redmine task status to 'In Review'
       * Notify team in Slack/communication channel

  **METHOD 2: AUTOMATED WITH MCP** (if GitHub MCP available):

  1. Use GitHub MCP to create PR programmatically

  2. Capture PR URL from MCP response

  3. Display same success message as Method 1

  **METHOD 3: MANUAL CREATION** (default fallback):

  1. Display step-by-step instructions:

     ```
     📋 Pull Request Creation Instructions
     =====================================

     Step 1: Commit Final Changes (if needed)
     Run: git add .
     Run: git commit -m '[#taskId] Brief description'

     Using this commit message:
     ----------------------------------------
     <COMMIT_MESSAGE_FROM_PHASE_3>
     ----------------------------------------

     Step 2: Push Branch to Remote
     Run: git push -u origin <BRANCH_NAME>

     Step 3: Create Pull Request on GitHub
     1. Navigate to: <GITHUB_REPO_URL>
     2. Click 'Pull requests' tab
     3. Click 'New pull request'
     4. Select base: develop
     5. Select compare: <BRANCH_NAME>
     6. Click 'Create pull request'

     Step 4: Fill PR Details
     Use the following content:

     PR TITLE:
     ----------------------------------------
     <PR_TITLE_FROM_PHASE_3>
     ----------------------------------------

     PR DESCRIPTION:
     ----------------------------------------
     <PR_DESCRIPTION_FROM_PHASE_3>
     ----------------------------------------

     Step 5: Submit PR
     1. Review the PR details
     2. Click 'Create pull request'
     3. Add reviewers (team members)
     4. Add labels if applicable

     Step 6: Post-PR Actions
     - Update Redmine task status to 'In Review'
     - Link PR URL in Redmine task
     - Notify team about PR
     ```

  2. Provide all generated content formatted for easy copy-paste

  3. Provide GitHub repository URL

  **Post-Creation Steps** (all methods):
  - Remind user to add reviewers
  - Suggest notifying team
  - Recommend updating Redmine task status
  - Provide Redmine task URL for easy navigation"

- Expected output: PR creation confirmation with URL (if automated) OR complete manual instructions with all generated content formatted for easy copy-paste
- Context: Generated PR content from Phase 3, repository information from Phase 1, available automation tools detected in Phase 1
- Action: Create PR using best available method. If automated, execute and confirm. If manual, provide complete step-by-step instructions with all content ready to use

---

## Success Criteria

### Overall Success
- [ ] All git prerequisites validated (changes committed, branch pushed)
- [ ] Redmine taskId obtained and validated
- [ ] Comprehensive PR description generated with all sections
- [ ] PR created successfully (automated) OR clear manual instructions provided with all content
- [ ] User knows next steps (reviewers, Redmine update, team notification)

### Phase 1 Criteria
- [ ] Git status is clean (all changes committed)
- [ ] Branch is pushed to remote
- [ ] Branch name follows feature/{taskId}-* convention (warning if not)
- [ ] Available automation methods detected (gh CLI, GitHub MCP, or manual)

### Phase 2 Criteria
- [ ] Redmine taskId obtained (from $ARGUMENTS or user prompt)
- [ ] TaskId format validated (numeric with optional #)
- [ ] Feature summary extracted or compiled
- [ ] Security audit results gathered (if available from previous workflow)
- [ ] Testing information collected
- [ ] Breaking changes identified (if any)

### Phase 3 Criteria
- [ ] Commit message template generated (follows [#taskId] format)
- [ ] PR title generated (clear, under 100 chars, includes taskId)
- [ ] PR description generated with all sections:
  - [ ] Related Task with Redmine link
  - [ ] Description with feature summary
  - [ ] Changes Made list
  - [ ] Security section with status and summary
  - [ ] Testing section with approach and coverage
  - [ ] Screenshots/Demo (or N/A)
  - [ ] Breaking Changes (or None)
  - [ ] Checklist with completion status
- [ ] All content uses proper markdown formatting
- [ ] Content is professional and clear

### Phase 4 Criteria
- [ ] PR created using best available method (automated preferred)
- [ ] PR URL provided (if automated) or manual instructions complete
- [ ] All generated content included in manual instructions (if manual)
- [ ] Post-creation steps communicated clearly
- [ ] User knows how to add reviewers and update Redmine

---

## Example Usage

### Example 1: Standard Workflow (Manual)
```bash
/create-pull-request Add user authentication feature with JWT tokens
```
**Flow**:
1. Validates prerequisites: All changes committed, branch pushed
2. Requests Redmine taskId: User provides `#12345`
3. Gathers information: Feature summary, security audit results, testing info
4. Generates PR content:
   - Title: `[#12345] Add user authentication with JWT and protected routes`
   - Description with security summary, testing details, checklist
5. Provides manual instructions: Complete step-by-step guide with all content formatted

### Example 2: With TaskId Provided
```bash
/create-pull-request #12345 Add user authentication with JWT
```
**Flow**:
1. Validates prerequisites
2. Extracts taskId from arguments: `#12345`
3. Gathers remaining information
4. Generates PR content
5. Creates PR (if gh CLI available) or provides manual instructions

### Example 3: Automated with gh CLI
```bash
/create-pull-request Implement password reset functionality
```
**Flow**:
1. Validates prerequisites: Detects gh CLI is available
2. Requests taskId: User provides `#12346`
3. Gathers information
4. Generates PR content
5. Executes `gh pr create` command automatically
6. Displays success message with PR URL: `https://github.com/org/repo/pull/42`

---

## Error Handling

### Phase 1 Errors

**Error**: Uncommitted Changes
- **Cause**: Files modified but not committed
- **Resolution**: STOP workflow. Display: "Error: You have uncommitted changes. Please commit all changes before creating a PR. Run: git add . && git commit -m 'Your message'"
- **Example**: User modified files but forgot to commit

**Error**: Branch Not Pushed
- **Cause**: Branch only exists locally
- **Resolution**: STOP workflow. Display: "Error: Branch not pushed to remote. Please push your branch first. Run: git push -u origin <branch-name>"
- **Example**: New feature branch not yet pushed to GitHub

**Error**: Invalid Branch Name
- **Cause**: Branch doesn't follow feature/{taskId}-* convention
- **Resolution**: WARN but continue. Display: "Warning: Branch name doesn't follow convention (feature/{taskId}-description). Consider renaming for consistency. Continue? (yes/no)"
- **Example**: Branch named `auth-feature` instead of `feature/12345-auth-feature`

### Phase 2 Errors

**Error**: No TaskId Provided
- **Cause**: User didn't include taskId in command
- **Resolution**: STOP and request taskId. Display: "Please provide the Redmine taskId for this feature (e.g., #12345 or 12345):"
- **Example**: `/create-pull-request Add feature` without taskId

**Error**: Invalid TaskId Format
- **Cause**: TaskId is not numeric
- **Resolution**: STOP and request valid taskId. Display: "Invalid taskId format. Please provide a numeric Redmine task ID (e.g., #12345 or 12345)"
- **Example**: User provides `ABC123` instead of numeric ID

### Phase 4 Errors

**Error**: GitHub Authentication Failed
- **Cause**: gh CLI not authenticated
- **Resolution**: Fall back to manual instructions. Display: "GitHub authentication failed. Falling back to manual PR creation. Follow the instructions below:"
- **Example**: gh CLI installed but `gh auth login` not run

**Error**: Network Issues During Automated Creation
- **Cause**: Cannot reach GitHub API
- **Resolution**: Fall back to manual instructions. Display: "Network error occurred. Cannot create PR automatically. Follow manual instructions below:"
- **Example**: Internet connection issues or GitHub API down

**Error**: PR Already Exists
- **Cause**: PR for this branch already exists
- **Resolution**: Display existing PR URL. "Pull request already exists for this branch: <PR_URL>"
- **Example**: User tries to create duplicate PR

---

## Coordination Notes

### Workflow Position
- This command should be run AFTER all development, testing, code quality review, and security audit phases complete
- Typically invoked at end of frontend-feature, backend-feature, or similar orchestration workflows
- Can also be run standalone if feature development completed manually

### Prerequisites
- All code changes must be committed
- Branch must be pushed to remote
- Redmine taskId required for traceability
- Preferably security audit completed (will note if not)

### Integration Points
- **frontend-orchestration**: Called at end of Phase 4 (conditional, if user approves)
- **backend-orchestration**: Will be called at end (future)
- **test-orchestration**: Called at end of Phase 8 (conditional, if user approves)
- **Redmine**: Creates link to Redmine task in PR description
- **GitHub**: Integrates with gh CLI or GitHub MCP when available

### Future Enhancements
- Automatic Redmine task status update to 'In Review' after PR creation
- Auto-add team reviewers based on CODEOWNERS or team configuration
- Auto-add labels based on file changes (e.g., frontend, backend, database)
- Integration with Slack/communication tools for team notifications
- Support for draft PR creation

---

## When to Use This Command

**Use when:**
- Feature development is complete and ready for review
- All tests are passing
- Security audit completed (or documented as not needed)
- Code quality review passed
- Ready to request team review

**Don't use when:**
- Feature is incomplete or work-in-progress (use draft PR instead)
- Tests are failing
- Critical security issues unresolved
- Major refactoring planned before review

**Standalone Usage:**
If not using orchestration workflows, can run standalone:
```bash
/create-pull-request [optional #taskId] Feature description
```

---

## Implementation Notes

### Timing
- **Quick execution**: 30-60 seconds (if automated)
- **Manual process**: 2-3 minutes (following instructions)

### PR Template Standards
- Follows Brooktec PR template
- Mandatory sections: Related Task, Description, Changes, Security, Testing
- Optional sections: Screenshots, Breaking Changes, Additional Notes
- Checklist adapted from project standards

### Traceability
- Redmine task linked in PR description
- TaskId in PR title for easy reference
- Commit messages follow [#taskId] convention
- Future: Bidirectional linking (Redmine → GitHub)

---

**End of Create Pull Request Command**
