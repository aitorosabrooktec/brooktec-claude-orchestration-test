Create a pull request for the completed feature:

[Extended thinking: This command orchestrates the pull request creation process, ensuring all changes are committed, the branch is properly named and pushed, and a comprehensive PR is created with proper Redmine task linking, security audit results, and standardized formatting. The process validates prerequisites, generates professional PR content, and guides through either manual or automated PR creation.]

## Pull Request Creation Workflow

### 1. Validate Prerequisites
- Use Task tool with subagent_type="git-actions::pull-request-manager"
- Prompt: "Validate pull request prerequisites for: $ARGUMENTS.
  
  Prerequisites Validation:
  - Check that all changes are committed (git status should be clean)
  - Verify branch is pushed to remote
  - Confirm branch name follows feature/{taskId}-* format
  - Check if GitHub CLI (gh) is available for automated creation
  
  If Prerequisites Fail:
  - Guide user to commit any uncommitted changes
  - Instruct how to push branch to remote
  - Warn if branch naming doesn't follow convention"
- Expected output: Prerequisites validation status, list of any issues to resolve
- Context: Current git branch, repository status, uncommitted changes
- Action: Ensure environment is ready for PR creation. STOP if critical issues exist.

### 2. Gather PR Information
- Use Task tool with subagent_type="git-actions::pull-request-manager"
- Prompt: "Gather information for pull request: $ARGUMENTS.
  
  Required Information:
  - Redmine taskId: If not provided, ask user: 'Please provide the Redmine taskId for this feature (e.g., #12345 or 12345)'
  - Feature summary: Brief description of what was implemented
  - Security audit results: Summary from Phase 3 security audit
  - Changes overview: High-level list of changes made
  - Testing approach: What testing was performed
  - Breaking changes: Any breaking changes (if applicable)
  - Screenshots/demos: Links or references (if applicable)
  
  Context to Include:
  - Branch name (contains taskId)
  - Feature requirements from Phase 1
  - Implementation details from Phase 2
  - Security findings from Phase 3"
- Expected output: Complete PR information package, validated taskId
- Context: Feature implementation, security audit results, branch name
- Action: Compile all necessary information for comprehensive PR description.

### 3. Generate PR Content
- Use Task tool with subagent_type="git-actions::pull-request-manager"
- Prompt: "Generate pull request content for: $ARGUMENTS.
  
  Generate the following using the gathered information:
  
  1. Commit Message Template (if needed):
     Format: [#taskId] Brief description of changes
     Example: [#12345] Add user authentication with JWT
  
  2. PR Title:
     Format: [#taskId] Feature description
     Example: [#12345] Add user authentication with JWT and protected routes
  
  3. Comprehensive PR Description (Markdown):
     Include all sections from the template:
     - Related Task (with Redmine link)
     - Description (feature summary)
     - Changes Made (bullet list)
     - Security (audit summary with status)
     - Testing (approach and cases)
     - Screenshots/Demo (if applicable)
     - Breaking Changes (if any)
     - Additional Notes
     - Checklist (pre-filled based on project standards)
  
  Ensure:
  - Professional, clear language
  - Proper markdown formatting
  - All security findings summarized
  - Redmine link is correct
  - Checklist reflects actual completion status"
- Expected output: Formatted commit message, PR title, complete PR description in markdown
- Context: Gathered PR information, taskId, security audit results
- Action: Generate professional, comprehensive PR content ready for submission.

### 4. Create Pull Request
- Use Task tool with subagent_type="git-actions::pull-request-manager"
- Prompt: "Create pull request using generated content: $ARGUMENTS.
  
  Creation Method Detection:
  - Check if GitHub CLI (gh) is installed and authenticated
  - Check if GitHub MCP is available
  - Determine: Automated or Manual process
  
  AUTOMATED CREATION (if available):
  - Use GitHub CLI command:
    gh pr create \\
      --title '[#taskId] Feature description' \\
      --body 'Generated PR description' \\
      --base develop \\
      --head feature/taskId-description
  - Or use GitHub MCP equivalent
  - Confirm PR was created successfully
  - Provide PR URL
  
  MANUAL CREATION (current default):
  - Provide step-by-step instructions:
    1. Commit all changes (with commit message template)
    2. Push branch to remote
    3. Navigate to GitHub repository
    4. Create new Pull Request
    5. Use generated title and description
    6. Submit PR
  - Display formatted commit message
  - Display PR title
  - Display PR description (ready to copy-paste)
  - Provide GitHub repository URL
  
  Post-Creation Steps:
  - Instruct user to add reviewers
  - Suggest notifying team
  - Recommend updating Redmine task status"
- Expected output: PR creation confirmation (URL if automated) or complete manual instructions with all generated content
- Context: Generated PR content, repository information, available tools
- Action: Create PR using best available method and confirm completion.

## Configuration Options
- `redmine_url`: Base URL for Redmine (default: auto-detect from project or use brooktec.com/redmine)
- `default_base_branch`: Base branch for PRs (default: "develop")
- `auto_add_reviewers`: Automatically add team reviewers (default: false, future feature)
- `pr_template`: Use custom PR template (default: standard Brooktec template)

## Success Criteria
- All changes are committed and pushed
- Branch naming convention validated
- Redmine taskId obtained and validated
- Comprehensive PR description generated including:
  - Feature summary
  - Changes list
  - Security audit results
  - Testing information
  - Complete checklist
- PR created successfully (automated) or clear manual instructions provided
- Redmine task linked for traceability
- Professional, consistent formatting

## Coordination Notes
- This command should be run after all development and security phases complete
- Requires Redmine taskId from user
- Validates git repository state before proceeding
- Generates standardized content following company PR template
- Provides fallback manual instructions if automation unavailable
- Includes security audit results as mandatory PR section
- Maintains traceability with Redmine task tracker
- Future: Will integrate with GitHub CLI/MCP for full automation
- Future: Will automatically update Redmine task status

## Example Usage
```
Create pull request for: Add user authentication feature with JWT tokens

This will:
1. Validate prerequisites (commits, push, branch name)
2. Request Redmine taskId if not provided
3. Gather feature and security information
4. Generate PR title, description, and commit message
5. Create PR (automated if available, or provide manual steps)
```

## Error Handling
- **Uncommitted changes**: Stop and instruct user to commit all changes first
- **Branch not pushed**: Stop and instruct user to push branch to remote
- **No taskId provided**: Request taskId from user before proceeding
- **Invalid branch name**: Warn but continue (already validated in setup phase)
- **GitHub authentication failed**: Fall back to manual instructions
- **Network issues**: Provide manual instructions as fallback

Feature to create PR for: $ARGUMENTS

