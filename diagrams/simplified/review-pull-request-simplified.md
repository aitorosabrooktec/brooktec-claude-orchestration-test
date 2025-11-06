# Pull Request Review Workflow - Simplified

## Command: `/review-pull-request <PR_NUMBER>`

```mermaid
flowchart TD
    Start([/review-pull-request]) --> Phase1["Phase 1: PR Information Gathering<br/><i>git, gh commands</i>"]

    Phase1 --> PRFound{PR Found?}
    PRFound -->|❌ No| Stop1[STOP: PR not accessible]
    PRFound -->|✅ Yes| Phase2["Phase 2: Technology Detection<br/><i>technology-detector</i>"]

    Phase2 --> Phase3["Phase 3: Code Quality Review<br/><i>code-reviewer</i>"]

    Phase3 --> Phase4["Phase 4: Security Audit<br/><i>security-auditor</i>"]

    Phase4 --> Phase5["Phase 5: Lint & Build Verification<br/><i>bash commands</i>"]

    Phase5 --> Phase6["Phase 6: Review Summary<br/><i>orchestrator</i>"]

    Phase6 --> Determine{Determine Recommendation}

    Determine -->|Critical Issues| Blocked[🔴 BLOCKED]
    Determine -->|High Priority| Changes[🟠 REQUEST CHANGES]
    Determine -->|Medium/Low| Comment[🟡 COMMENT]
    Determine -->|No Issues| Approve[🟢 APPROVE]

    Blocked --> End([End: Review Complete])
    Changes --> End
    Comment --> End
    Approve --> End

    classDef stopStyle fill:#ff6b6b,stroke:#c92a2a,color:#fff
    classDef successStyle fill:#51cf66,stroke:#2f9e44,color:#fff
    classDef phaseStyle fill:#4dabf7,stroke:#1971c2,color:#fff
    classDef decisionStyle fill:#ffd43b,stroke:#f59f00,color:#000
    classDef blockedStyle fill:#ff6b6b,stroke:#c92a2a,color:#fff,stroke-width:3px
    classDef changesStyle fill:#ffa94d,stroke:#fd7e14,color:#000,stroke-width:3px
    classDef commentStyle fill:#74c0fc,stroke:#339af0,color:#000,stroke-width:3px
    classDef approveStyle fill:#51cf66,stroke:#2f9e44,color:#fff,stroke-width:3px

    class Stop1 stopStyle
    class End successStyle
    class Phase1,Phase2,Phase3,Phase4,Phase5,Phase6 phaseStyle
    class PRFound,Determine decisionStyle
    class Blocked blockedStyle
    class Changes changesStyle
    class Comment commentStyle
    class Approve approveStyle
```

## Phases

1. **PR Information Gathering** → Fetch PR details, changes, commits
   - **Tools**: `git`, `gh` commands (no agent)

2. **Technology Detection** → Identify languages, frameworks, tools
   - **Agents**: `shared-agents::technology-detector`

3. **Code Quality Review** → Analyze code with tech-specific lens
   - **Agents**: `shared-agents::code-reviewer` (invoked as pr-reviewer)

4. **Security Audit** → OWASP Top 10 vulnerability scan
   - **Agents**: `security-compliance::security-auditor`

5. **Lint & Build Verification** → Run linting, build, type checks
   - **Tools**: Bash commands (no agent)

6. **Review Summary** → Consolidate findings and determine recommendation
   - **Process**: Orchestrator consolidates all findings (no agent)

## Possible Outcomes

### ❌ STOP Conditions
- **Phase 1**: PR not found or inaccessible

### Review Recommendations

#### 🔴 BLOCKED
- **Triggers**: Critical security vulnerabilities OR build failures
- **Action**: Do not merge until resolved

#### 🟠 REQUEST CHANGES
- **Triggers**: High priority code quality issues OR security warnings
- **Action**: Address issues before merge

#### 🟡 COMMENT
- **Triggers**: Medium/low priority suggestions OR performance optimizations
- **Action**: Consider addressing, but can merge

#### 🟢 APPROVE
- **Triggers**: No significant issues found
- **Action**: Ready to merge

## Review Report Includes

- **Summary**: Issue counts by severity (Critical, High, Medium, Low, Nitpick)
- **Technology Stack**: Detected frameworks and tools
- **Code Quality Findings**: Categorized by severity with file:line references
- **Security Findings**: OWASP vulnerabilities with remediation
- **Build Results**: Lint errors, build failures, type errors
- **Positive Highlights**: Good patterns found
- **Next Steps**: Actionable recommendations
