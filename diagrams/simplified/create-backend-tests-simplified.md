# Backend Test Generation Workflow - Simplified

## Command: `/create-backend-tests --target <path> [--type <type>] [--coverage-threshold <num>] [--skip-pr]`

```mermaid
flowchart TD
    Start([/create-backend-tests]) --> CheckTarget{--target provided?}
    CheckTarget -->|❌ No| Stop1[STOP: --target required]
    CheckTarget -->|✅ Yes| Phase1["Phase 1: Setup & Requirements<br/><i>project-setup, requirements-reviewer</i>"]

    Phase1 --> Valid{Valid?}
    Valid -->|❌ Issues| Stop2[STOP: Fix environment]
    Valid -->|✅ OK| Phase2["Phase 2: Code Analysis & Planning<br/><i>technology-detector</i>"]

    Phase2 --> Framework{Framework?}
    Framework -->|❌ Ambiguous| Stop3[STOP: Specify framework]
    Framework -->|✅ Detected| Analyze["Analyze code + Plan scenarios<br/><i>test-generator</i>"]

    Analyze --> Phase3["Phase 3: Test Generation<br/><i>test-generator</i>"]

    Phase3 --> Phase4["Phase 4: Test Execution<br/><i>Jest/pytest/JUnit</i>"]

    Phase4 --> TestsPass{Tests Pass?}
    TestsPass -->|❌ Failed| Retry{Retry < 3?}
    Retry -->|Yes| Fix[Fix tests/code]
    Fix --> Phase4
    Retry -->|No| Stop4[STOP: Manual intervention]
    TestsPass -->|✅ Pass| Phase5["Phase 5: Coverage Analysis<br/><i>coverage tools</i>"]

    Phase5 --> Coverage{Coverage >= Threshold?}
    Coverage -->|❌ Low| Attempt{Attempt < 2?}
    Attempt -->|Yes| GenMore["Generate more tests<br/><i>test-generator</i>"]
    GenMore --> Phase5
    Attempt -->|No| Ask[Ask user: Acceptable?]
    Ask -->|❌ No| Stop5[STOP: More tests needed]
    Ask -->|✅ Yes| Phase6
    Coverage -->|✅ OK| Phase6["Phase 6: Test Quality Review<br/><i>code-reviewer</i>"]

    Phase6 --> Phase7[Phase 7: Approval Checkpoint]

    Phase7 --> Approval[🛑 APPROVAL CHECKPOINT]

    Approval --> Choice{User Choice?}
    Choice -->|🔄 Changes| Phase3
    Choice -->|✅ No PR| End1([End: Tests Generated])
    Choice -->|✅ Create PR| CheckFlag{--skip-pr?}

    CheckFlag -->|Yes| End1
    CheckFlag -->|No| Phase8["Phase 8: Create PR<br/><i>pull-request-manager</i>"]

    Phase8 --> End2([End: PR Created])

    classDef stopStyle fill:#ff6b6b,stroke:#c92a2a,color:#fff
    classDef successStyle fill:#51cf66,stroke:#2f9e44,color:#fff
    classDef phaseStyle fill:#4dabf7,stroke:#1971c2,color:#fff
    classDef decisionStyle fill:#ffd43b,stroke:#f59f00,color:#000
    classDef checkpointStyle fill:#ff6b6b,stroke:#c92a2a,color:#fff,stroke-width:4px

    class Stop1,Stop2,Stop3,Stop4,Stop5 stopStyle
    class End1,End2 successStyle
    class Phase1,Phase2,Phase3,Phase4,Phase5,Phase6,Phase7,Phase8,Analyze,Fix,GenMore phaseStyle
    class CheckTarget,Valid,Framework,TestsPass,Retry,Coverage,Attempt,Ask,Choice,CheckFlag decisionStyle
    class Approval checkpointStyle
```

## Phases

1. **Setup & Requirements** → Validate environment, identify targets, check framework
   - **Agents**: `shared-agents::project-setup`, `shared-agents::requirements-reviewer`

2. **Code Analysis & Planning** → Detect framework, analyze code, plan test scenarios
   - **Agents**: `shared-agents::technology-detector`, `test-orchestration::test-generator`

3. **Test Generation** → Generate tests with proper mocking and AAA pattern
   - **Agents**: `test-orchestration::test-generator`

4. **Test Execution** → Run tests, retry up to 3 times on failure
   - **Tools**: Test framework (Jest, pytest, JUnit) via bash commands

5. **Coverage Analysis** → Check coverage, generate more tests if below threshold
   - **Tools**: Coverage tools (istanbul, coverage.py, JaCoCo) via bash commands

6. **Test Quality Review** → Review test code quality and maintainability
   - **Agents**: `shared-agents::code-reviewer`

7. **Approval Checkpoint** → User reviews and decides next action
   - **Process**: User decision point (no agent)

8. **PR Creation** (conditional) → Create pull request with test summary
   - **Agents**: `git-actions::pull-request-manager` (via `/create-pull-request` command)

## Possible Outcomes

### ❌ STOP Conditions
- **Phase 1**: --target not provided, environment issues
- **Phase 2**: Framework ambiguous
- **Phase 4**: Tests fail after 3 retries
- **Phase 5**: Coverage low and user declines

### 🛑 Approval Checkpoint (After Phase 6)
User decides:
1. **Request Changes** → Return to Phase 3 with feedback
2. **Approve without PR** → End workflow
3. **Approve and Create PR** → Continue to Phase 8 (if --skip-pr not set)

### ✅ Success Outcomes
- **End without PR**: Tests generated, passing, coverage met, quality reviewed
- **End with PR**: Tests generated, passing, coverage met, quality reviewed, PR created

## Flags

- `--target` (required): File or directory to test
- `--type` (optional): Test type (unit/integration/e2e/all) - default: unit
- `--coverage-threshold` (optional): Minimum coverage % - default: 80
- `--skip-pr` (optional): Skip PR creation

## Test Types

- **unit**: Unit tests only (fastest, most common) - DEFAULT
- **integration**: Integration tests (DB, APIs)
- **e2e**: End-to-end tests (full workflows)
- **all**: All test types (comprehensive)

## Supported Frameworks

- **Node.js**: Jest, Vitest (Express, NestJS)
- **Python**: pytest (Django, FastAPI)
- **Java**: JUnit (Spring Boot)

## Example Usage

```bash
# Basic usage
/create-backend-tests --target src/services/user.service.ts

# With custom coverage threshold
/create-backend-tests --target src/services/ --coverage-threshold 90

# Integration tests without PR
/create-backend-tests --skip-pr --target src/api/ --type integration

# All test types
/create-backend-tests --target src/services/payment.service.ts --type all
```

## Success Criteria

- All tests passing
- Coverage meets/exceeds threshold (default: 80%)
- Tests follow AAA pattern
- Proper mocking of dependencies
- Independent tests (no shared state)
- Framework-specific best practices
