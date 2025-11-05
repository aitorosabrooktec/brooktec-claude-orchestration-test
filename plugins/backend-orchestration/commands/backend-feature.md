Orchestrate backend feature development with API testing, quality review, security audit, and user approval before PR creation.

[Extended thinking: This workflow coordinates specialized agents to deliver a secure, production-ready backend feature with proper setup validation, clear requirements, code quality assurance, comprehensive API testing, security compliance, and user approval before PR creation. The workflow ensures the development environment is properly configured, validates requirement clarity, intelligently detects the backend technology (Express or NestJS), and proceeds with framework-aware implementation using Node.js best practices. After implementation, the workflow generates and executes API integration tests to verify endpoint functionality, then performs code quality review to check best practices, patterns, and maintainability, followed by comprehensive security auditing. The workflow then STOPS to request explicit user approval. The user reviews the implementation, test results, code quality findings, and security findings before deciding to either proceed to PR creation, finish without PR, or request additional changes. This approval checkpoint ensures quality control and gives the user final say before submitting the feature for review. The node-developer agent automatically detects existing tooling (Express/NestJS, ORM/ODM, validation library, authentication approach) and adapts accordingly, while informing users about alternative approaches when beneficial.]

## Workflow Configuration

### Command Flags
Parse command line arguments to determine workflow behavior:

**--skip-pr Flag** (optional):
- If present: Skip Phase 6 (PR Creation) entirely
- Workflow ends after user approval in Phase 5
- Usage: `/backend-feature --skip-pr Implement user authentication API`
- Use case: When user wants to create PR manually or not at all

**--skip-tests Flag** (optional):
- If present: Skip Phase 3 (API Testing) entirely
- Workflow proceeds directly from implementation to code quality review
- Usage: `/backend-feature --skip-tests Implement data export utility`
- Use case: When feature doesn't have testable API endpoints (utilities, internal services)

**Flag Parsing**:
- Check if `--skip-pr` flag is present in $ARGUMENTS
- Check if `--skip-tests` flag is present in $ARGUMENTS
- Store flag states in workflow context for later phases
- Remove flags from feature description before passing to agents
- Example: `--skip-pr --skip-tests Implement helper functions` → feature description = "Implement helper functions", skipPR = true, skipTests = true

---

## Phase 1: Setup & Requirements Validation

### 1. Project Setup Verification
- Use Task tool with subagent_type="shared-agents::project-setup"
- Prompt: "Verify and setup the development environment for backend development.

  Claude Project Initialization (FIRST):
  - Check if CLAUDE.md file exists in project root
  - If CLAUDE.md doesn't exist:
    - STOP and inform user to run /init in the chat
    - Wait for user to execute /init and confirm completion
    - Re-check for CLAUDE.md after user confirms
    - Continue with remaining setup once verified

  Git Branch Validation (SECOND):
  - Check current git branch
  - STOP IMMEDIATELY if on develop or main branches (user must create feature branch: feature/{taskId}-brief-description)
  - STOP IMMEDIATELY if branch doesn't follow feature/{taskId}-* format
  - Check for uncommitted changes
  - STOP IMMEDIATELY if there are uncommitted changes (user must commit or stash first)

  Environment Setup:
  - Check if node_modules are installed
  - Validate .nvmrc configuration
  - Verify .npmrc if needed
  - Validate environment variables (.env files from Shadows if templates exist)
  - Check if development server is already running
  - Start the local development server if not running (npm run dev, npm run start:dev, or equivalent)"

- Expected output: Claude initialization status (CLAUDE.md exists), git branch validation status, environment status report, confirmation of dependencies installed, Node version validation, environment variables status, dev server running status (existing or newly started)
- Context: Project root directory, git repository, and package.json configuration
- Action: This step must complete successfully before proceeding. STOPS and waits for user action if:
  - CLAUDE.md missing (user must run /init and type 'continue')
  - On develop/main branches (user must create feature branch)
  - Branch doesn't follow feature/{taskId}-* format
  - Uncommitted changes exist (user must commit or stash)
  - .env file is missing when template exists (user must get from Shadows)
  - Any other critical environment issues
  After user resolves issue, workflow continues automatically

### 2. Requirements Review & Clarification
- Use Task tool with subagent_type="shared-agents::requirements-reviewer"
- Prompt: "Review the following backend feature requirements for completeness and clarity: $ARGUMENTS.

  Focus on backend-specific requirements:
  - API endpoint specifications (methods, paths, request/response formats)
  - Business logic requirements
  - Data validation rules
  - Authentication/authorization requirements
  - Database schema changes or queries needed
  - External API integrations
  - Performance requirements (response time, throughput)
  - Error handling expectations
  - Logging requirements

  Identify any missing information, ambiguous specifications, or unclear requirements. If critical information is missing, stop and request clarification from the user."

- Expected output: Requirements validation report, list of clarifying questions (if any), approved requirements document with backend-specific details
- Context: User's feature request and project context
- Action: If clarification is needed, halt the workflow and wait for user input. Only proceed when all requirements are clear and complete

---

## Phase 2: Development

### 3. Technology Detection & Framework Selection

**CRITICAL: Detect backend technology BEFORE implementing the feature**

- Use Task tool with subagent_type="shared-agents::technology-detector"
- Prompt: "Detect Node.js backend technology stack for feature implementation: $ARGUMENTS

  Analysis Required:
  - Analyze package.json and project structure
  - Identify backend framework (Express or NestJS)
  - Detect framework version and architecture patterns
  - Identify ORM/ODM in use (TypeORM, Prisma, Mongoose, Sequelize)
  - Detect validation library (Joi, class-validator, Zod, yup)
  - Identify authentication/authorization approach (JWT, Passport.js, custom)
  - Detect database type (PostgreSQL, MongoDB, MySQL, etc.)
  - Identify API documentation approach (Swagger/OpenAPI)
  - Determine testing framework (Jest, Mocha, Supertest)

  Detection Priority:
  - **Backend Frameworks**: Check for express (Express.js), @nestjs/core (NestJS)
  - **ORMs/ODMs**: Check for typeorm, @prisma/client, mongoose, sequelize
  - **Validation**: Check for joi, class-validator, zod, yup

  Ambiguity Handling:
  - If both Express and NestJS found: Report ambiguity, request user clarification
  - If multiple ORMs found: Report which is primary based on usage, inform user
  - If no clear framework: Report ambiguity, request user clarification

  Output Required:
  - Primary framework identified: Express or NestJS
  - Framework version and architecture (TypeScript/JavaScript)
  - ORM/ODM detected with database type
  - Validation library detected
  - Authentication approach detected
  - Testing framework detected
  - API documentation approach detected

  **STOP and ask user** if framework is ambiguous or cannot be determined"

- Expected output: Technology detection report with primary backend framework, ORM/ODM, validation approach, auth approach, testing setup
- Context: package.json dependencies, project directory structure, configuration files (tsconfig.json, ormconfig.json, etc.)
- Action: Use detected technology to select appropriate development approach. If technology is ambiguous, STOP and request user clarification before proceeding

### 4. Implement Backend Feature

- Use Task tool with subagent_type="backend-development::node-developer"
- Prompt: "Implement the backend feature using detected framework: $ARGUMENTS

  **Technology Context**:
  - Framework: [Express/NestJS from Phase 2.3]
  - ORM/ODM: [detected ORM from Phase 2.3]
  - Validation: [detected validation library from Phase 2.3]
  - Authentication: [detected auth approach from Phase 2.3]
  - Database: [detected database type from Phase 2.3]

  **Implementation Requirements**:
  1. **API Endpoints**:
     - Design RESTful endpoints with proper HTTP methods
     - Implement request/response handling
     - Follow REST conventions for resource naming
     - Use proper HTTP status codes

  2. **Request Validation**:
     - Validate all request inputs (body, query params, path params)
     - Use detected validation library
     - Provide clear validation error messages
     - Sanitize input to prevent injection attacks

  3. **Business Logic**:
     - Implement feature requirements in service layer
     - Apply proper error handling
     - Use transactions where appropriate
     - Follow single responsibility principle

  4. **Database Layer**:
     - Create/update database entities using detected ORM
     - Implement efficient queries with proper indexing
     - Use migrations for schema changes
     - Apply proper relationships (one-to-many, many-to-many)

  5. **Authentication/Authorization**:
     - Implement route protection if required
     - Use detected auth approach
     - Apply role-based or permission-based access control
     - Validate JWT tokens or session state

  6. **Error Handling**:
     - Implement comprehensive error handling
     - Use proper error classes and hierarchies
     - Return standardized error responses
     - Log errors with appropriate context

  7. **Security**:
     - Implement input validation and sanitization
     - Use parameterized queries to prevent SQL injection
     - Apply rate limiting if public endpoint
     - Set security headers (CORS, CSP, etc.)
     - Never expose sensitive data in responses

  8. **Logging**:
     - Add structured logging for debugging
     - Log important operations and errors
     - Never log sensitive data (passwords, tokens)
     - Use appropriate log levels

  9. **Code Quality**:
     - Use TypeScript with strict mode
     - Follow framework conventions and patterns
     - Write clean, maintainable code
     - Document complex logic with comments
     - Export types and interfaces

  10. **API Documentation**:
      - Add OpenAPI/Swagger annotations if project uses it
      - Document request/response schemas
      - Include example requests and responses

  **Framework-Specific Patterns**:
  - **Express**: Use router modules, middleware chains, service layer separation
  - **NestJS**: Use controllers, services, modules, guards, pipes, interceptors, DTOs

  Adapt to detected tooling and inform user about alternative approaches when beneficial"

- Expected output: Complete backend feature implementation with API endpoints, business logic, database layer, validation, authentication, error handling, security measures, logging, and documentation
- Context: Detected technology stack from Phase 2.3, project structure, existing code patterns, requirements from Phase 1.2
- Action: Implement production-ready backend feature following detected framework patterns and best practices. Ensure all code is type-safe, secure, and well-documented

---

## Phase 3: API Testing & Validation

**⚠️ This phase only executes if --skip-tests flag NOT present**

### 5. Generate API Integration Tests

- Use Task tool with subagent_type="test-orchestration::test-generator"
- Prompt: "Generate comprehensive API integration tests for implemented endpoints: $ARGUMENTS

  **Testing Requirements**:
  1. **Test Framework Setup**:
     - Use detected testing framework (Jest, Mocha)
     - Use Supertest for HTTP assertions
     - Set up test database or mocking strategy
     - Configure test environment variables

  2. **Test Scenarios**:
     - **Happy Paths**: Test successful operations
       - Valid requests with expected responses
       - Correct HTTP status codes (200, 201, etc.)
       - Proper response payload structure
     - **Edge Cases**: Test boundary conditions
       - Empty payloads, minimum/maximum values
       - Special characters in inputs
       - Large data sets
     - **Error Cases**: Test failure scenarios
       - Invalid request data (validation errors)
       - Missing required fields
       - Unauthorized access attempts
       - Not found resources (404)
       - Conflict scenarios (409)
       - Server errors (500)

  3. **Authentication Testing** (if applicable):
     - Test with valid authentication tokens
     - Test with invalid/expired tokens
     - Test without authentication
     - Test role-based access control

  4. **Database Testing**:
     - Verify data persistence after operations
     - Test database constraints and validations
     - Test transaction rollback on errors
     - Clean up test data after each test

  5. **Test Structure**:
     - Use describe/it blocks for organization
     - Follow AAA pattern (Arrange, Act, Assert)
     - Create test data fixtures
     - Implement setup and teardown hooks

  6. **Assertions**:
     - Verify HTTP status codes
     - Verify response payload structure and content
     - Verify response headers (Content-Type, etc.)
     - Verify database state changes
     - Use specific assertions (not just truthy/falsy)

  **Test Coverage Goals**:
  - Cover all API endpoints implemented
  - Cover all HTTP methods (GET, POST, PUT, PATCH, DELETE)
  - Cover all success and error scenarios
  - Achieve >80% code coverage for new code

  Output complete test files with all imports, setup, test cases, and teardown"

- Expected output: Complete API integration test files covering happy paths, edge cases, and error scenarios for all implemented endpoints
- Context: Implemented feature from Phase 2.4, detected testing framework, API endpoint specifications
- Action: Generate comprehensive, runnable API tests following detected testing framework patterns

### 6. Execute API Tests

- Use Bash tool to run API tests
- Command: `npm test` or `npm run test:integration` (based on package.json scripts)
- Capture test output (passed/failed/skipped counts)
- Parse test results for failures
- If tests fail:
  - Analyze failure reasons
  - Identify if failure is in test code or implementation
  - If test issue: Fix generated tests and re-run
  - If implementation issue: Fix implementation and re-run tests
  - Maximum retry attempts: 3
  - If still failing after retries: STOP and report to user
- Expected output: All API tests passing (0 failures), test execution summary with counts
- Context: Generated tests from Phase 3.5, running dev server, test database configuration
- Action: Verify all API endpoints work correctly by executing tests. All tests must pass before proceeding to code quality review

---

## Phase 4: Quality & Security Review

### 7. Code Quality Review

- Use Task tool with subagent_type="shared-agents::code-reviewer"
- Prompt: "Review the implemented backend feature for code quality and best practices: $ARGUMENTS

  **Review Focus**:
  1. **Code Structure & Organization**:
     - Proper separation of concerns (controllers, services, repositories)
     - Modular code with clear responsibilities
     - Appropriate use of design patterns
     - Code reusability and DRY principle

  2. **TypeScript Usage**:
     - Proper type annotations (no `any` types)
     - Interface definitions for DTOs and entities
     - Type safety throughout the codebase
     - Proper use of generics where appropriate

  3. **Error Handling**:
     - Comprehensive error catching and handling
     - Proper use of try-catch blocks
     - Custom error classes where appropriate
     - Meaningful error messages

  4. **Database Queries**:
     - Efficient query patterns
     - Proper use of ORM features
     - No N+1 query problems
     - Appropriate use of transactions
     - Query optimization and indexing

  5. **API Design**:
     - RESTful conventions followed
     - Consistent endpoint naming
     - Proper use of HTTP methods and status codes
     - Request/response payload consistency

  6. **Security Practices**:
     - Input validation and sanitization
     - Parameterized queries (no SQL injection)
     - No sensitive data in logs or responses
     - Proper authentication/authorization checks

  7. **Code Maintainability**:
     - Clear variable and function names
     - Appropriate comments for complex logic
     - Consistent code formatting
     - No commented-out code or TODOs
     - Proper documentation

  8. **Performance Considerations**:
     - Efficient algorithms and data structures
     - No unnecessary database calls
     - Appropriate caching strategies
     - No memory leaks (proper cleanup)

  9. **Testing**:
     - Test coverage for new code
     - Tests are meaningful and not trivial
     - Test code quality (readable, maintainable)

  **Framework-Specific Reviews**:
  - **Express**: Router organization, middleware usage, error middleware
  - **NestJS**: Module organization, dependency injection, decorator usage

  Do NOT review frontend code, only backend implementation"

- Expected output: Comprehensive code quality report with findings categorized by severity (critical, high, medium, low), specific code locations, and recommendations for improvement
- Context: Implemented feature code from Phase 2.4, project coding standards, detected framework patterns
- Action: Identify code quality issues and provide actionable recommendations. User will review findings in approval checkpoint

### 8. Security Audit

- Use Task tool with subagent_type="security-compliance::security-auditor"
- Prompt: "Perform comprehensive security audit on backend implementation: $ARGUMENTS

  **Security Audit Focus**:
  1. **Input Validation & Sanitization**:
     - All user inputs validated at API boundaries
     - Proper data type checking and constraints
     - Sanitization to prevent injection attacks
     - File upload validation (if applicable)

  2. **Injection Prevention**:
     - SQL injection prevention (parameterized queries)
     - NoSQL injection prevention
     - Command injection prevention
     - LDAP injection prevention

  3. **Authentication & Authorization**:
     - Proper authentication implementation
     - Secure token/session management
     - Password hashing (bcrypt, argon2)
     - Token expiration and refresh
     - Authorization checks on protected routes
     - Role/permission validation

  4. **Data Protection**:
     - Sensitive data encryption at rest
     - Sensitive data encryption in transit (HTTPS)
     - No sensitive data in logs or error messages
     - No secrets in code or version control
     - Proper environment variable usage

  5. **API Security**:
     - Rate limiting implementation
     - CORS configuration (no `origin: '*'` in production)
     - Security headers (Helmet.js)
     - CSRF protection (if applicable)
     - API key/token validation

  6. **Error Handling Security**:
     - No stack traces in production responses
     - Generic error messages to clients
     - Detailed errors only in logs
     - No information leakage

  7. **Dependencies**:
     - No known vulnerabilities in dependencies
     - Dependencies up to date
     - No deprecated packages

  8. **OWASP Top 10 Coverage**:
     - Broken access control prevention
     - Cryptographic failures prevention
     - Injection prevention
     - Insecure design mitigation
     - Security misconfiguration prevention
     - Vulnerable components check
     - Authentication failures prevention
     - Data integrity failures prevention
     - Logging and monitoring adequacy
     - Server-side request forgery prevention

  **Output Format**:
  - Security status: PASSED / WARNINGS / CRITICAL ISSUES
  - List of findings with severity (Critical, High, Medium, Low)
  - Specific vulnerable code locations
  - Remediation recommendations
  - OWASP mapping for each finding"

- Expected output: Comprehensive security audit report with security status, findings categorized by severity, vulnerable code locations, OWASP mappings, and remediation recommendations
- Context: Implemented feature code from Phase 2.4, API endpoints, authentication/authorization logic, database queries, dependencies
- Action: Identify security vulnerabilities and provide remediation recommendations. Critical and high-severity findings should be addressed before user approval

---

## Phase 5: Approval Checkpoint

**STOP HERE and request user approval before proceeding to Phase 6**

### 9. User Review & Approval

Present comprehensive summary to user:

```
Backend Feature Implementation Complete

📊 **Implementation Summary:**
- Feature: [Feature description]
- Framework: [Express/NestJS]
- API Endpoints: [List of endpoints created/modified]
- Database Changes: [Entities/migrations created]
- Authentication: [Auth approach used]

📝 **Code Quality Review:**
- Status: [PASSED / WARNINGS / ISSUES]
- Critical Issues: [count]
- Warnings: [count]
- Key Findings:
  [Top 3-5 findings from code-reviewer]

🔒 **Security Audit:**
- Status: [PASSED / WARNINGS / CRITICAL ISSUES]
- Critical Vulnerabilities: [count]
- High-Severity Issues: [count]
- Key Findings:
  [Top 3-5 security findings from security-auditor]

✅ **API Testing** (if not skipped):
- Total Tests: [count]
- Passed: [count]
- Failed: [count]
- Coverage: [percentage]

**Your options:**
1. ✅ **Approve and Create PR**: Type 'yes', 'approve', or 'continue' to proceed to PR creation
2. ✅ **Approve without PR**: Type 'approve without pr', 'skip pr', or 'finish' to approve and end workflow
3. 🔄 **Request Changes**: Type 'no' or 'changes needed' to modify implementation
4. ❓ **Need More Info**: Ask questions about implementation, tests, quality, or security findings
```

**User Response Handling**:
- If **APPROVED WITH PR** (yes/approve/continue):
  - Check --skip-pr flag: If present, treat as "Approve without PR"
  - If flag not present: Proceed to Phase 6 (PR Creation)
- If **APPROVED WITHOUT PR** (approve without pr/skip pr/finish):
  - Workflow ends successfully
  - Feature is implemented and validated
  - User can create PR manually later
- If **CHANGES NEEDED** (no/changes needed):
  - Ask user for specific changes
  - Return to appropriate phase based on change type:
    - Implementation changes: Return to Phase 2.4
    - Test changes: Return to Phase 3.5
    - Both: Return to Phase 2.4, then re-run Phase 3
  - Re-run subsequent phases (quality review, security audit)
  - Return to approval checkpoint
- If **QUESTIONS**:
  - Answer questions and ask again for approval

**IMPORTANT**:
- Phase 6 only executes if user explicitly chooses "Approve and Create PR" AND --skip-pr flag NOT present
- Critical security issues should be highlighted and recommended for resolution before PR

---

## Phase 6: Pull Request Creation (Conditional)

**⚠️ This phase only executes if:**
1. User chose "Approve and create PR" option
2. --skip-pr flag was NOT provided
3. User did NOT choose "Approve without PR"

### 10. Create Pull Request

- Use Command tool with command="git-actions::create-pull-request"
- Arguments: "Implemented [feature description from $ARGUMENTS]"
- PR Description should include:
  - Feature summary
  - API endpoints created/modified
  - Database changes (entities, migrations)
  - Authentication/authorization changes
  - Code quality assessment
  - Security audit status
  - Test coverage achieved
  - Link to Redmine task
  - Breaking changes (if any)
  - Deployment notes (if any)
- Expected output: PR created with comprehensive description, PR URL provided
- If skipped: Workflow ends, user can create PR manually
- Context: All previous phase results, commit history, branch information

---

## Success Criteria

### Overall Success
- [ ] Environment validated and ready (Phase 1.1)
- [ ] Requirements clear and approved (Phase 1.2)
- [ ] Technology detected correctly (Phase 2.3)
- [ ] Feature implemented with all requirements (Phase 2.4)
- [ ] API tests generated and passing (Phase 3 - if not skipped)
- [ ] Code quality reviewed (Phase 4.7)
- [ ] Security audit completed (Phase 4.8)
- [ ] User approved implementation (Phase 5)
- [ ] PR created (Phase 6 - if user approved with PR)

### Phase 1 Criteria
- [ ] CLAUDE.md exists (project initialized)
- [ ] Git branch validated (feature/{taskId}-* format, not on develop/main)
- [ ] No uncommitted changes
- [ ] Dependencies installed (node_modules)
- [ ] Environment variables configured
- [ ] Dev server running
- [ ] Requirements clear and complete
- [ ] No ambiguous specifications

### Phase 2 Criteria
- [ ] Backend framework detected (Express or NestJS)
- [ ] ORM/ODM identified
- [ ] Validation library identified
- [ ] Authentication approach identified
- [ ] Feature implemented following framework patterns
- [ ] API endpoints created with proper HTTP methods
- [ ] Request validation implemented
- [ ] Business logic in service layer
- [ ] Database entities created/updated
- [ ] Authentication/authorization applied
- [ ] Error handling comprehensive
- [ ] Security measures implemented
- [ ] Logging added
- [ ] Code is type-safe (TypeScript strict mode)

### Phase 3 Criteria (if not skipped)
- [ ] API integration tests generated
- [ ] Tests cover happy paths, edge cases, error scenarios
- [ ] Tests cover all implemented endpoints
- [ ] All tests passing (0 failures)
- [ ] Test coverage >80% for new code
- [ ] Tests are maintainable and readable

### Phase 4 Criteria
- [ ] Code quality review completed
- [ ] Code follows framework conventions
- [ ] TypeScript types are proper (no `any`)
- [ ] Error handling is comprehensive
- [ ] Database queries are efficient
- [ ] Code is maintainable
- [ ] Security audit completed
- [ ] No critical security vulnerabilities
- [ ] Input validation implemented
- [ ] No injection vulnerabilities
- [ ] Authentication/authorization secure
- [ ] OWASP Top 10 considerations addressed

### Phase 5 Criteria
- [ ] Implementation summary provided to user
- [ ] Code quality findings shared
- [ ] Security findings shared
- [ ] Test results shared (if applicable)
- [ ] User made explicit approval choice

### Phase 6 Criteria (if applicable)
- [ ] Only executes if approved with PR and --skip-pr not set
- [ ] Redmine taskId provided
- [ ] PR created with comprehensive description
- [ ] Feature committed and pushed

---

## Coordination Notes

### Workflow Position
- This command can be run standalone for backend feature development
- Typically invoked when starting new backend feature work
- Can be called from higher-level orchestration workflows
- Should be run after planning and design phases complete

### Prerequisites
- CLAUDE.md must exist (run /init if missing)
- Must be on feature branch (feature/{taskId}-*)
- Git working directory should be clean
- Dependencies installed (node_modules)
- Environment variables configured
- Node.js LTS version recommended

### Flag Interactions
- --skip-pr and --skip-tests can be combined
- Flags are removed from feature description before passing to agents
- Flag state is tracked throughout workflow execution

### User Interaction Points
- STOP when CLAUDE.md missing (user must run /init)
- STOP when on wrong git branch (user must create feature branch)
- STOP when uncommitted changes (user must commit or stash)
- STOP when requirements unclear (user must clarify)
- STOP when technology ambiguous (user must specify framework)
- STOP when tests fail repeatedly (user intervention needed)
- STOP at approval checkpoint (user must approve/reject/request changes)

### Safety Features
- Environment validation prevents broken development setup
- Requirements validation prevents ambiguous implementation
- Technology detection prevents wrong framework usage
- API testing validates endpoint functionality
- Code quality review catches maintainability issues
- Security audit identifies vulnerabilities
- User approval checkpoint ensures quality control
- Separate git commits allow easy rollback

### Integration Points
- **shared-agents**: Uses project-setup, requirements-reviewer, technology-detector, code-reviewer
- **backend-development**: Uses node-developer agent
- **test-orchestration**: Uses test-generator agent
- **security-compliance**: Uses security-auditor agent
- **git-actions**: Uses create-pull-request command
- **Redmine**: PR links to Redmine task
- **GitHub**: PR created with comprehensive description

---

## Example Usage

### Example 1: Standard Workflow (With Tests and PR)
```bash
/backend-feature Implement user authentication API with JWT
```

**Flow**:
1. Validates environment: CLAUDE.md exists, on feature/12345-user-auth branch
2. Reviews requirements: Clarifies JWT expiration, refresh token strategy
3. Detects technology: Express + TypeORM + Joi + JWT
4. Implements feature: Login, logout, refresh endpoints with validation
5. Generates tests: Happy paths, invalid credentials, token expiration tests
6. Executes tests: All 15 tests passing
7. Code quality review: All checks passed, minor style suggestions
8. Security audit: Passed, password hashing confirmed, token storage secure
9. User approval: User types "yes"
10. Creates PR: PR #42 created with full description

**Output**:
```
✅ Phase 1 Complete: Environment validated
✅ Phase 2 Complete: Authentication API implemented (Express + TypeORM)
✅ Phase 3 Complete: 15 API tests generated and passing
✅ Phase 4 Complete: Code quality PASSED, Security audit PASSED
✅ Phase 5 Complete: User approved
✅ Phase 6 Complete: PR created - https://github.com/org/repo/pull/42
```

### Example 2: Without Tests (Utility Feature)
```bash
/backend-feature --skip-tests Implement data export utility functions
```

**Flow**:
1. Validates environment
2. Reviews requirements: Confirms export formats (CSV, JSON, XML)
3. Detects technology: Express + TypeORM
4. Implements feature: Export utility functions in service layer
5. **Skips Phase 3**: No API endpoints to test
6. Code quality review: Passed with recommendations
7. Security audit: Passed
8. User approval: User types "approve without pr"
9. **Skips Phase 6**: User chose no PR

**Output**:
```
✅ Phases 1-2 Complete: Utility functions implemented
⏭️  Phase 3 Skipped: --skip-tests flag present
✅ Phase 4 Complete: Code quality PASSED, Security audit PASSED
✅ Phase 5 Complete: User approved without PR
Workflow complete. Create PR manually when ready.
```

### Example 3: Security Issues Found
```bash
/backend-feature Implement payment processing API
```

**Flow**:
1-6. Standard workflow through tests
7. Code quality review: Passed
8. Security audit: **CRITICAL ISSUES FOUND**
   - Missing input validation on amount field
   - No rate limiting on payment endpoint
   - Sensitive data in logs
9. **Stops at approval**: User reviews critical security findings
10. User types "changes needed"
11. **Returns to Phase 2.4**: Fixes security issues
12. Re-runs Phases 3-4: Tests pass, security audit now PASSED
13. User approval: User types "yes"
14. Creates PR with security fixes documented

**Output**:
```
⚠️  Phase 4 Security Audit: CRITICAL ISSUES (3 critical, 2 high)
📋 Approval Required: Review security findings
👤 User: "changes needed"
🔄 Returning to Phase 2.4 to fix security issues...
✅ Security issues resolved
✅ Tests re-run: All passing
✅ Security audit re-run: PASSED
✅ User approved
✅ PR created with security fixes documented
```

---

## Error Handling

### Phase 1 Errors

**Error**: CLAUDE.md Missing
- **Cause**: Project not initialized with Claude Code
- **Resolution**: STOP workflow. Display: "Error: CLAUDE.md not found. Please run /init to initialize the project, then type 'continue'."
- **Example**: User tries to run on project without /init

**Error**: Wrong Git Branch
- **Cause**: On develop or main branch
- **Resolution**: STOP workflow. Display: "Error: Cannot develop on develop/main branch. Create feature branch: feature/{taskId}-brief-description"
- **Example**: User forgot to create feature branch

**Error**: Branch Name Convention
- **Cause**: Branch doesn't follow feature/{taskId}-* format
- **Resolution**: STOP workflow. Display: "Error: Branch must follow format: feature/{taskId}-description. Current: {currentBranch}"
- **Example**: Branch named "my-feature" instead of "feature/12345-my-feature"

**Error**: Uncommitted Changes
- **Cause**: Working directory has uncommitted changes
- **Resolution**: STOP workflow. Display: "Error: Uncommitted changes detected. Commit or stash changes first: git add . && git commit -m 'message'"
- **Example**: User modified files without committing

**Error**: Missing Environment Variables
- **Cause**: .env file missing when .env.example exists
- **Resolution**: STOP workflow. Display: "Error: .env file missing. Copy from Shadows or create from .env.example"
- **Example**: Fresh clone without .env configuration

### Phase 2 Errors

**Error**: Ambiguous Framework
- **Cause**: Both Express and NestJS detected or neither detected
- **Resolution**: STOP and ask user. Display: "Multiple/no backend frameworks detected. Which framework should I use? (Express/NestJS)"
- **Example**: Project has both frameworks in dependencies

**Error**: Requirements Unclear
- **Cause**: Missing critical information (API spec, data model, validation rules)
- **Resolution**: STOP and request clarification. Display specific questions about missing information
- **Example**: "Implement user API" without specifying endpoints or fields

**Error**: Implementation Failure
- **Cause**: Code generation error or framework-specific issue
- **Resolution**: STOP and report error details. Display: "Error implementing feature: [details]. Review requirements and try again."
- **Example**: Invalid TypeScript syntax or ORM configuration error

### Phase 3 Errors

**Error**: Test Generation Failure
- **Cause**: Cannot generate tests due to unclear endpoint specification
- **Resolution**: STOP and request endpoint details. Display: "Cannot generate tests: [reason]. Provide endpoint specifications."
- **Example**: Custom response format not documented

**Error**: Tests Fail After Generation
- **Cause**: Generated tests fail due to implementation issues
- **Resolution**: Attempt to fix automatically (up to 3 times). If unable: STOP and report failures
- **Example**: Validation error in test data

**Error**: Test Execution Failure
- **Cause**: Test framework error or environment issue
- **Resolution**: STOP and report error. Display: "Test execution failed: [details]. Check test framework configuration."
- **Example**: Database connection error in test environment

### Phase 4 Errors

**Error**: Code Review Tool Failure
- **Cause**: code-reviewer agent error
- **Resolution**: STOP and retry once. If still fails: Continue without code review but warn user
- **Example**: Agent timeout or tool error

**Error**: Security Audit Tool Failure
- **Cause**: security-auditor agent error
- **Resolution**: STOP and retry once. If still fails: STOP workflow and warn user security audit incomplete
- **Example**: Agent timeout or dependency scanning error

### Phase 6 Errors

**Error**: PR Creation Failure
- **Cause**: GitHub authentication failed or branch not pushed
- **Resolution**: STOP and display manual PR creation instructions. Provide all PR content formatted for copy-paste
- **Example**: gh CLI not authenticated or branch only exists locally

**Error**: No Redmine TaskId
- **Cause**: Branch doesn't contain taskId or taskId cannot be parsed
- **Resolution**: STOP and ask user for Redmine taskId. Display: "Cannot determine taskId from branch. Provide Redmine taskId: (e.g., #12345)"
- **Example**: Branch named "feature/auth" without taskId

---

## When to Use This Command

**Use when:**
- Starting new backend feature development
- Implementing new API endpoints
- Adding new business logic or services
- Modifying existing backend functionality
- All requirements are clear or can be clarified
- Ready for comprehensive quality and security review

**Don't use when:**
- Feature requirements are completely undefined (do planning first)
- Making hotfix or emergency changes (use expedited process)
- Just exploring or prototyping (use manual development)
- Frontend-only changes (use frontend-feature instead)
- Making configuration-only changes (manual approach simpler)

**Standalone Usage:**
Run anytime to implement backend feature:
```bash
/backend-feature Implement user registration API with email verification
```

**With Flags:**
```bash
# Skip PR creation
/backend-feature --skip-pr Implement data export functions

# Skip API testing
/backend-feature --skip-tests Implement utility helper functions

# Skip both
/backend-feature --skip-pr --skip-tests Implement internal service functions
```

---

## Implementation Notes

### Timing
- **Quick execution**: 15-30 minutes (simple feature with tests and reviews)
- **Standard execution**: 30-60 minutes (complex feature with multiple endpoints)
- **Extended execution**: 60-120 minutes (complex feature with external integrations)

### Framework Adaptability
- Automatically detects Express or NestJS
- Applies framework-specific patterns and conventions
- Uses framework-appropriate file structure
- Follows framework best practices and idioms

### Testing Philosophy
- API integration tests verify endpoint functionality
- Tests act as living documentation
- Tests should be fast and isolated
- Test database separate from development database
- Tests clean up after themselves

### Security Priority
- Security audit is mandatory (cannot be skipped)
- Critical security issues highlighted in approval checkpoint
- Recommend fixing critical issues before PR
- All input validation required at API boundaries
- OWASP Top 10 coverage ensures comprehensive security

### Code Quality Standards
- TypeScript strict mode required
- No `any` types allowed (use proper types)
- Comprehensive error handling required
- Structured logging required
- Code must be maintainable and documented

---

**End of Backend Feature Orchestration Command**
