# Dependency Health Check Command

This command performs a comprehensive dependency health check on your project, ensuring your dependencies are secure, up-to-date, and using the correct Node.js LTS version.

## What This Command Does

This command orchestrates a complete dependency health check workflow that:

1. **Validates Node.js Version**: Ensures you're using an LTS version of Node.js
2. **Resolves Security Vulnerabilities**: Identifies and fixes critical and high-severity vulnerabilities
3. **Updates Packages Safely**: Updates packages to minor/patch versions while checking for breaking changes
4. **Generates Comprehensive Report**: Creates a detailed report with all changes and recommendations

## Workflow Overview

The command invokes the `dependency-health-checker` agent which will:

- Execute `npx npuc --init` to validate Node.js version
- Update `.nvmrc` if needed to use LTS version
- Run `npm audit` to identify security vulnerabilities
- Check for deprecated packages and suggest alternatives
- Fix vulnerabilities (avoiding breaking changes)
- Update packages to latest minor/patch versions
- Check for breaking changes before applying updates
- Create **3 separate commits** (one for each phase):
  1. Node.js LTS validation
  2. Security vulnerability fixes
  3. Package updates
- Generate a detailed markdown report with status tables

## How It Works

The agent follows a **methodical, phase-by-phase approach**:

- **Phase 1**: Node Version Validation → Commit
- **Phase 2**: Security Vulnerability Resolution → Commit
- **Phase 3**: Package Updates (Minor/Patch) → Commit
- **Phase 4**: Report Generation

**Important**: The agent will **STOP and ask for your input** if it encounters:
- Deprecated packages that need migration
- Packages with breaking changes
- Dependency conflicts
- Any ambiguous situations

## Prerequisites

Before running this command, ensure:
- You have a `package.json` file in your project
- You have `npm` installed
- You're in the root directory of your project
- Your git working directory is clean (or you're okay with new commits)

## Expected Output

After completion, you'll receive:

1. **3 Git Commits**:
   - `chore: validate and update Node.js to LTS version`
   - `chore: resolve security vulnerabilities in dependencies`
   - `chore: update dependencies to latest minor/patch versions`

2. **Comprehensive Report** with tables showing:
   - Node.js version changes
   - Security vulnerabilities resolved (with severity levels)
   - Package updates applied
   - Deprecated packages marked with ❌
   - Packages blocked by breaking changes marked with ⚠️
   - Recommendations for manual actions

3. **Status Indicators**:
   - ✅ Successfully completed
   - ❌ Deprecated package (needs migration)
   - ⚠️ Blocked (breaking changes detected)

## Safety Features

- **No Breaking Changes**: Never applies updates with breaking changes
- **Verification Checks**: Re-checks each phase after completion
- **User Approval**: Stops for guidance when needed
- **Separate Commits**: Easy to revert individual phases if needed
- **Detailed Notes**: All blockers and issues documented

## Usage

Simply run this command from your project root:

```
/dependency-health::check-dependencies
```

Or invoke programmatically:

```
Use Command tool with command="dependency-health::check-dependencies"
```

## Example Scenarios

### Scenario 1: All Updates Successful
```
✅ Node.js updated to 20.x LTS
✅ 5 critical vulnerabilities resolved
✅ 12 packages updated to latest minor versions
✅ No deprecated packages found
```

### Scenario 2: Manual Action Required
```
✅ Node.js updated to 20.x LTS
✅ 3 critical vulnerabilities resolved
❌ Package 'request' is deprecated → Use 'axios' instead
⚠️ Package 'webpack' has breaking changes in v6 → Manual update required
```

## When to Run This Command

Run this command regularly to maintain dependency health:

- **Weekly/Monthly**: As part of routine maintenance
- **Before Major Features**: Ensure clean dependency state
- **After Security Alerts**: Respond to vulnerability notifications
- **Project Onboarding**: When joining a new project
- **Before Releases**: Ensure production-ready dependencies

## Command Execution

Now executing the dependency health check workflow...

---

## Orchestration Instructions

Use the Task tool to invoke the dependency-health-checker agent:

**Task Tool Parameters**:
- `subagent_type`: `"dependency-health::dependency-health-checker"`
- `prompt`: Provide detailed context about the project and any specific concerns

**Prompt Template**:

```
Perform a comprehensive dependency health check on this project.

Project Context:
- Working directory: [current directory]
- Package manager: npm
- Project type: [describe if known]

Execute all 4 phases of the dependency health check:

1. Node.js LTS Version Validation
   - Run npx npuc --init
   - Update .nvmrc if needed
   - Verify and commit changes

2. Security Vulnerability Resolution
   - Audit all dependencies
   - Identify deprecated packages
   - Fix critical and high vulnerabilities
   - Commit changes

3. Package Updates (Minor/Patch)
   - Check for available updates
   - Verify no breaking changes
   - Update safely
   - Commit changes

4. Report Generation
   - Create comprehensive markdown report
   - Include all tables and status indicators
   - Provide recommendations

Remember:
- STOP and ask for guidance if you encounter any ambiguity
- Create separate commits for phases 1, 2, and 3
- Verify each phase before progressing
- Never apply breaking changes without approval
- Mark deprecated packages with ❌
- Mark blocked packages with ⚠️

Project files to consider:
- package.json
- package-lock.json
- .nvmrc
- node_modules/ (check existence)

Provide a comprehensive report at the end with all changes documented.
```

---

## Agent Invocation

Use Task tool with:
- subagent_type: "dependency-health::dependency-health-checker"
- description: "Execute comprehensive dependency health check"
- prompt: [Use the template above with current project context]

The agent will handle the entire workflow autonomously, stopping only when user input is required.
