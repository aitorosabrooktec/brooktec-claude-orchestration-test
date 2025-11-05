Orchestrate comprehensive backend test generation:

[Extended thinking: This workflow coordinates specialized agents to generate, execute, and validate comprehensive backend tests. The workflow analyzes target code, identifies test scenarios, generates unit/integration/E2E tests, executes them to verify they pass, analyzes coverage, reviews test quality, and presents results for user approval. The user can then optionally create a PR with the generated tests. This workflow ensures high-quality, maintainable test suites that improve code confidence and catch bugs early.]

## Workflow Configuration

### Command Flags
Parse command line arguments:

**--skip-pr Flag** (optional):
- If present: Skip Phase 7 (PR Creation) entirely
- Usage: `/create-backend-tests --skip-pr --target src/services/user.service.ts`

**--target Flag** (required):
- Specifies file(s) or directory to generate tests for
- Usage: `--target src/services/user.service.ts` or `--target src/services/`
- Can be single file or directory

**--type Flag** (optional):
- Specifies test type preference: unit, integration, e2e, or all
- Default: unit
- Usage: `--type integration` or `--type all`

**--coverage-threshold Flag** (optional):
- Minimum coverage percentage required
- Default: 80
- Usage: `--coverage-threshold 90`

**Flag Parsing**:
- Extract --skip-pr, --target, --type, --coverage-threshold from $ARGUMENTS
- Validate --target is provided (required)
- Set defaults for optional flags

## Phase 1: Setup & Requirements Validation

### 1. Requirements Clarification & Validation
- Use Task tool with subagent_type="shared-agents::requirements-reviewer"
- Prompt: "Review and validate test generation requirements: $ARGUMENTS

  Requirements to validate:
  - Target files/directories specified (--target flag)
  - Test type specified or default (unit, integration, e2e, all)
  - Coverage threshold specified or default (80%)
  - Clear understanding of what needs to be tested
  - Verify target files/directories exist in project
  - Identify any missing or ambiguous information

  If requirements are clear: Provide summary and proceed
  If requirements are unclear: STOP and request clarification from user

  Required information:
  - What files/directories to generate tests for (--target)
  - What type of tests to generate (optional: --type)
  - What coverage threshold to achieve (optional: --coverage-threshold)
  - Whether to skip PR creation (optional: --skip-pr)"
- Expected output: Requirements validation report, either approved or requesting clarification
- Action: If clarification needed, STOP workflow and wait for user response
- CRITICAL: Must have clear, unambiguous requirements before proceeding to implementation

### 2. Project Setup Verification
- Use Task tool with subagent_type="shared-agents::project-setup"
- Prompt: "Verify development environment for test generation.

  Environment Validation:
  - Check CLAUDE.md exists
  - Validate git branch (feature/{taskId}-*)
  - Check node_modules/dependencies installed
  - Verify test framework is configured (Jest, pytest, JUnit)
  - Check test scripts exist in package.json/setup.py/pom.xml
  - Validate linter is configured"
- Expected output: Environment validation status, test framework detected
- Action: Environment must be ready before test generation

### 3. Test Target Identification
- Parse --target flag to identify code to test
- Verify target files/directories exist
- Identify code language/framework using technology-detector
- List all files to generate tests for
- Check if tests already exist for targets
- If tests exist: Analyze existing coverage and identify gaps
- If no tests: Plan full test suite generation

## Phase 2: Code Analysis & Test Planning

### 4. Technology Detection
- Use Task tool with subagent_type="shared-agents::technology-detector"
- Prompt: "Detect technology stack for test generation: $ARGUMENTS

  Identify:
  - Backend framework (Express, NestJS, Django, FastAPI, Spring Boot)
  - Test framework (Jest, Vitest, pytest, JUnit, etc.)
  - Mocking libraries available
  - Database/ORM in use
  - Existing test patterns in codebase"
- Expected output: Technology stack, test framework, testing patterns
- Context: Target files, package.json/requirements.txt/pom.xml
- Action: Use detected technology for test generation

### 5. Code Analysis & Scenario Identification
- Use Task tool with subagent_type="test-orchestration::test-generator"
- Prompt: "Analyze target code and identify test scenarios: $ARGUMENTS

  Target: $TARGET_FILES
  Test Type: $TEST_TYPE

  Analysis Tasks:
  - Read and analyze each target file
  - Identify functions/methods/classes to test
  - Identify dependencies (DBs, APIs, services)
  - Determine appropriate test type for each unit
  - Identify test scenarios:
    - Happy paths (successful operations)
    - Edge cases (boundary conditions, special values)
    - Error handling (exceptions, failures, invalid inputs)
    - Integration points (if integration tests)
  - Plan mocking strategy for external dependencies
  - Estimate test count and complexity

  Output:
  - List of testable units
  - Test scenarios for each unit (happy, edge, error)
  - Mocking strategy
  - Estimated test suite size"
- Expected output: Comprehensive test plan with scenarios
- Action: Use test plan for generation in Phase 3

## Phase 3: Test Generation

### 6. Generate Test Suite
- Use Task tool with subagent_type="test-orchestration::test-generator"
- Prompt: "Generate comprehensive test suite: $ARGUMENTS

  Target: $TARGET_FILES
  Test Type: $TEST_TYPE
  Test Plan: $TEST_PLAN_FROM_PHASE_2
  Framework: $DETECTED_FRAMEWORK
  Coverage Goal: $COVERAGE_THRESHOLD%

  Generation Requirements:
  - Generate tests for all identified scenarios
  - Use proper test framework syntax ($DETECTED_FRAMEWORK)
  - Follow AAA pattern (Arrange, Act, Assert)
  - Mock all external dependencies properly
  - Include descriptive test names
  - Add comments for complex test scenarios
  - Use test data builders if needed
  - Ensure test independence (no shared state)
  - Generate both positive and negative test cases
  - Cover all code branches

  Test File Organization:
  - Follow project conventions for test file naming
  - Create test files in proper directory (__tests__/, tests/, test/)
  - Match source file structure

  Output:
  - Complete test files with all imports
  - Mock setup and teardown if needed
  - Test data fixtures if needed
  - Configuration for test framework if needed"
- Expected output: Complete runnable test files
- Action: Write generated tests to appropriate test directories

## Phase 4: Test Execution & Validation

### 7. Execute Generated Tests
- Run test command using Bash tool
- Commands based on detected framework:
  - Node.js: `npm test` or `npm run test:unit`
  - Python: `pytest` or `python -m pytest`
  - Java: `mvn test` or `./gradlew test`
- Capture test output (passed/failed/skipped)
- If tests fail:
  - Analyze failure reasons
  - Identify if failure is in generated test or actual code bug
  - If test issue: Fix generated tests
  - If code bug: Report to user
  - Re-run until tests pass or user intervention needed
- Maximum retry attempts: 3
- Action: All generated tests must pass before proceeding

## Phase 5: Coverage Analysis

### 8. Run Coverage Report
- Execute coverage command:
  - Node.js: `npm run test:coverage` or `npx jest --coverage`
  - Python: `pytest --cov=src --cov-report=term`
  - Java: Use JaCoCo or similar
- Parse coverage output
- Analyze coverage metrics:
  - Line coverage percentage
  - Branch coverage percentage
  - Function coverage percentage
  - Uncovered lines/branches
- Compare against coverage threshold
- If coverage below threshold:
  - Identify uncovered areas
  - Generate additional tests for gaps
  - Re-run coverage analysis
- Action: Meet or exceed coverage threshold

## Phase 6: Test Quality Review

### 9. Review Test Code Quality
- Use Task tool with subagent_type="shared-agents::code-reviewer"
- Prompt: "Review generated test code for quality and best practices: $ARGUMENTS

  Review Focus:
  - Test code structure and organization
  - Proper use of testing framework
  - Mock quality and appropriateness
  - Test independence (no shared state)
  - Assertion quality (meaningful, not too broad)
  - Test naming conventions
  - AAA pattern adherence
  - Test maintainability
  - Absence of flaky patterns
  - Proper error handling in tests
  - Test documentation/comments
  - Performance (fast execution)

  Frameworks-Specific Checks:
  - Jest/Vitest: Proper mock usage, describe/it structure
  - pytest: Fixture usage, parametrize patterns
  - JUnit: Annotation usage, assertion library

  Do NOT review source code, only test code quality"
- Expected output: Test quality report with findings
- Action: Address critical test quality issues

## Phase 7: Approval Checkpoint

**STOP HERE and request user approval before proceeding to Phase 8**

Present to user:
- Test generation summary
- Test execution results (all tests passing)
- Coverage analysis (X% coverage achieved, threshold: Y%)
- Test quality review findings
- Number of tests generated
- Files modified/created

Ask user:
"Test generation, execution, and validation complete.

📊 **Summary:**
- Tests Generated: [X unit, Y integration, Z E2E]
- Test Execution: [All X tests passing]
- Coverage: [X% lines, Y% branches - threshold: Z%]
- Test Quality: [Assessment from code-reviewer]

**Your options:**
1. ✅ **Approve and Create PR**: Type 'yes', 'approve', or 'continue' to proceed to PR creation
2. ✅ **Approve without PR**: Type 'approve without pr', 'skip pr', or 'finish' to approve and end workflow
3. 🔄 **Request Changes**: Type 'no' or 'changes needed' to regenerate tests with modifications
4. ❓ **Need More Info**: Ask questions about generated tests or coverage"

**User Response Handling:**
- If **APPROVED WITH PR** (yes/approve/continue):
  - Check --skip-pr flag: If present, treat as "Approve without PR"
  - If flag not present: Proceed to Phase 8 (PR Creation)
- If **APPROVED WITHOUT PR** (approve without pr/skip pr/finish):
  - Workflow ends successfully
  - Tests are generated and passing
  - User can create PR manually later
- If **CHANGES NEEDED** (no/changes needed):
  - Ask user for specific changes
  - Return to Phase 3 with feedback
- If **QUESTIONS**:
  - Answer questions and ask again for approval

**IMPORTANT**:
- Phase 8 only executes if user explicitly chooses "Approve and Create PR" AND --skip-pr flag NOT present

## Phase 8: Pull Request Creation (Conditional)

**⚠️ This phase only executes if:**
1. User chose "Approve and create PR" option
2. --skip-pr flag was NOT provided
3. User did NOT choose "Approve without PR"

### 10. Create Pull Request
- Use Command tool with command="git-actions::create-pull-request"
- Arguments: "Generated comprehensive tests for [target]"
- PR Description should include:
  - Tests generated (count by type)
  - Coverage achieved
  - Files tested
  - Link to Redmine task
  - Test quality assessment
- Expected output: PR created with comprehensive description
- If skipped: Workflow ends, user can create PR manually

## Success Criteria

### Phase 1 Criteria
- Environment validated and ready
- Test framework detected and configured
- Target files identified and exist
- Existing tests analyzed (if any)

### Phase 2 Criteria
- Technology stack detected
- Code analyzed successfully
- Test scenarios identified (happy, edge, error)
- Mocking strategy planned
- Test plan created

### Phase 3 Criteria
- Test files generated for all targets
- Tests follow framework conventions
- Proper mocking implemented
- AAA pattern used
- Test names are descriptive
- Tests are independent

### Phase 4 Criteria
- All generated tests executed
- All tests passing (0 failures)
- No flaky tests detected
- Test execution time reasonable

### Phase 5 Criteria
- Coverage report generated
- Coverage meets or exceeds threshold
- All critical code paths covered
- Branch coverage adequate

### Phase 6 Criteria
- Test quality reviewed
- No critical quality issues
- Tests are maintainable
- Proper testing patterns used

### Phase 7 Criteria (Approval Checkpoint)
- All previous phases completed successfully
- User reviewed summary
- User made explicit approval choice:
  - Approve and create PR
  - Approve without PR
  - Request changes

### Phase 8 Criteria (Conditional)
- Only if approved with PR and --skip-pr not set
- Redmine taskId provided
- PR created with comprehensive description
- Tests committed and pushed

## Coordination Notes

### Phase Flow
- Parse flags at workflow start (--skip-pr, --target, --type, --coverage-threshold)
- Phase 1-6 are sequential and mandatory
- Phase 7 is approval checkpoint (STOP and wait for user)
- Phase 8 is conditional based on user choice and --skip-pr flag
- If Phase 4 tests fail: Retry up to 3 times, then stop for user intervention
- If Phase 5 coverage below threshold: Generate additional tests automatically
- If Phase 8 skipped: Workflow ends successfully, user creates PR manually

### Target Specification
- --target can be single file: `--target src/services/user.service.ts`
- --target can be directory: `--target src/services/`
- --target can be multiple: `--target src/services/user.service.ts src/services/payment.service.ts`
- If directory: Test all files in directory recursively

### Test Type Selection
- `--type unit`: Generate unit tests only (fastest, most common)
- `--type integration`: Generate integration tests (DB, APIs)
- `--type e2e`: Generate E2E tests (full workflows)
- `--type all`: Generate all test types (comprehensive but time-consuming)

### Coverage Goals
- Default threshold: 80%
- Can be overridden with `--coverage-threshold X`
- If coverage below threshold: Automatically generate additional tests
- Maximum attempts to reach threshold: 2

## Example Usage

```bash
# Generate unit tests for single file
/create-backend-tests --target src/services/user.service.ts

# Generate tests for directory with custom threshold
/create-backend-tests --target src/services/ --coverage-threshold 90

# Generate integration tests without creating PR
/create-backend-tests --skip-pr --target src/api/controllers/ --type integration

# Generate all test types
/create-backend-tests --target src/services/payment.service.ts --type all
```

## Error Handling

### Missing --target Flag
- Error: "Target file or directory must be specified using --target flag"
- Example: `--target src/services/user.service.ts`

### Target Not Found
- Error: "Target file/directory not found: [path]"
- Suggest: Check file path and ensure it exists

### Tests Fail After Generation
- Attempt to fix automatically (up to 3 times)
- If unable to fix: Present error to user with details
- User can choose to proceed anyway or request changes

### Coverage Below Threshold
- Automatically generate additional tests (up to 2 attempts)
- If still below: Report to user, ask if acceptable
- User can approve with lower coverage or request additional tests

### Test Framework Not Detected
- Error: "Unable to detect test framework"
- Suggest: Ensure test dependencies are installed
- List common test frameworks for detected language

This workflow generates production-ready, comprehensive test suites that improve code quality and confidence.
