# Create Pull Request Workflow - Simplified

## Command: `/create-pull-request` or via other workflows

```mermaid
flowchart TD
    Start([create-pull-request]) --> Phase1["Phase 1: Prerequisites Validation<br/><i>pull-request-manager</i>"]

    Phase1 --> Valid{Valid?}
    Valid -->|❌ Issues| Stop1[STOP: Commit/push changes]
    Valid -->|✅ OK| Phase2["Phase 2: Information Gathering<br/><i>pull-request-manager</i>"]

    Phase2 --> TaskID{TaskId Provided?}
    TaskID -->|❌ No| Stop2[STOP: Provide Redmine taskId]
    TaskID -->|✅ Yes| Phase3["Phase 3: PR Content Generation<br/><i>pull-request-manager</i>"]

    Phase3 --> Phase4["Phase 4: PR Creation<br/><i>pull-request-manager</i>"]

    Phase4 --> Method{Automation Available?}

    Method -->|gh CLI| Auto1["Automated with gh<br/><i>gh pr create</i>"]
    Method -->|GitHub MCP| Auto2["Automated with MCP<br/><i>GitHub MCP</i>"]
    Method -->|None| Manual["Manual Instructions<br/><i>step-by-step guide</i>"]

    Auto1 --> Success1{Success?}
    Success1 -->|❌ Failed| Manual
    Success1 -->|✅ OK| End1([End: PR Created Automatically])

    Auto2 --> Success2{Success?}
    Success2 -->|❌ Failed| Manual
    Success2 -->|✅ OK| End1

    Manual --> End2([End: Manual Instructions Provided])

    classDef stopStyle fill:#ff6b6b,stroke:#c92a2a,color:#fff
    classDef successStyle fill:#51cf66,stroke:#2f9e44,color:#fff
    classDef phaseStyle fill:#4dabf7,stroke:#1971c2,color:#fff
    classDef decisionStyle fill:#ffd43b,stroke:#f59f00,color:#000
    classDef manualStyle fill:#868e96,stroke:#495057,color:#fff

    class Stop1,Stop2 stopStyle
    class End1,End2 successStyle
    class Phase1,Phase2,Phase3,Phase4,Auto1,Auto2 phaseStyle
    class Valid,TaskID,Method,Success1,Success2 decisionStyle
    class Manual manualStyle
```

## Phases

1. **Prerequisites Validation** → Check git status, branch pushed, branch name
   - **Agents**: `git-actions::pull-request-manager`

2. **Information Gathering** → Collect taskId, feature summary, changes, security results
   - **Agents**: `git-actions::pull-request-manager`

3. **PR Content Generation** → Generate PR title, description, commit message
   - **Agents**: `git-actions::pull-request-manager`

4. **PR Creation** → Automated (gh CLI/MCP) or manual instructions
   - **Agents**: `git-actions::pull-request-manager`
   - **Methods**: GitHub CLI (`gh`), GitHub MCP, or manual instructions

## Possible Outcomes

### ❌ STOP Conditions
- **Phase 1**: Uncommitted changes, branch not pushed
- **Phase 2**: TaskId not provided or invalid format

### ✅ Success Outcomes

#### Automated (Preferred)
- **gh CLI**: PR created automatically with gh pr create
- **GitHub MCP**: PR created programmatically via MCP
- **Fallback**: Falls back to manual if automation fails

#### Manual (Fallback)
- Complete step-by-step instructions
- All PR content formatted for copy-paste
- GitHub repository URL provided
- Reminder about post-PR actions

## PR Description Includes

- **Redmine Task**: Link to task
- **Description**: Feature summary
- **Changes Made**: Key changes list
- **Security**: Audit status and summary
- **Testing**: Approach and coverage
- **Screenshots/Demo**: If applicable
- **Breaking Changes**: If any
- **Checklist**: Completion status

## Post-Creation Actions

- Add reviewers
- Update Redmine task status to 'In Review'
- Notify team
