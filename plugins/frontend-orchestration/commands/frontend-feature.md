Orchestrate frontend feature development:

[Extended thinking: This workflow coordinates specialized agents to deliver a secure, production-ready frontend feature with proper setup validation, clear requirements, security compliance, and user approval before PR creation. The workflow ensures the development environment is properly configured (including environment variables from Shadows), validates requirement clarity, intelligently detects the project technology (Angular vs React/Next.js), and proceeds with framework-aware implementation using best practices for modern frontend development. If the technology is ambiguous or unclear, it STOPS and asks the user to specify which framework to use. The workflow performs comprehensive security auditing, and then STOPS to request explicit user approval. The user reviews the implementation and security findings before deciding to either proceed to PR creation or request additional changes. This approval checkpoint ensures quality control and gives the user final say before submitting the feature for review. The developer agents (angular-developer or frontend-developer) automatically detect existing tooling (framework version, state management, styling, authentication) and adapt accordingly, while informing users about alternative approaches when beneficial.]

## Phase 1: Setup & Requirements Validation

### 1. Project Setup Verification
- Use Task tool with subagent_type="frontend-orchestration::project-setup"
- Prompt: "Verify and setup the development environment for the project. 
  
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
  - Start the local development server if not running"
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
- Use Task tool with subagent_type="frontend-orchestration::requirements-reviewer"
- Prompt: "Review the following feature requirements for completeness and clarity: $ARGUMENTS. Identify any missing information, ambiguous specifications, or unclear requirements. If critical information is missing, stop and request clarification from the user."
- Expected output: Requirements validation report, list of clarifying questions (if any), approved requirements document
- Context: User's feature request and project context
- Action: If clarification is needed, halt the workflow and wait for user input. Only proceed when all requirements are clear and complete.

## Phase 2: Development

### 3. Technology Detection & Agent Selection

**CRITICAL: Detect project technology BEFORE implementing the feature**

1. **Check package.json for frontend framework**:
   - Look for @angular/core → Angular project
   - Look for react and/or next → React/Next.js project
   - If BOTH found: STOP and ask user which technology to use for this feature
   - If NEITHER found: STOP and ask user which technology this project uses

2. **If unclear or ambiguous**:
   - Multiple frameworks detected
   - No clear framework indicators
   - Package.json missing or incomplete

   **→ STOP IMMEDIATELY and ask user:**
   "I detected [describe what was found]. Which technology should I use for this feature?
   - Angular
   - React/Next.js
   - Other (please specify)"

   **Wait for user response before proceeding.**

3. **Once technology is confirmed, proceed with appropriate agent:**

### 3a. Angular Feature Implementation (if Angular detected)
- Use Task tool with subagent_type="frontend-mobile-development::angular-developer"
- Prompt: "Implement the following Angular feature: $ARGUMENTS.

  Detection phase:
  - Check package.json for Angular version (@angular/core)
  - Detect standalone components vs NgModule architecture
  - Check for signals usage (Angular 16+)
  - Check for control flow syntax (Angular 17+)
  - Detect state management: check for existing library (NgRx Store, NgRx SignalStore, Akita, NGXS) - continue if found
  - Detect styling: check for existing approach (Tailwind, Angular Material, SCSS) - continue if found
  - Detect authentication: check for existing auth implementation - continue if found
  - For each detected tool that differs from preferred stack: inform user about preferred tool benefits ONLY if they provide clear advantages, then continue with existing approach

  Implementation phase:
  - Build Angular components with proper TypeScript types and OnPush change detection
  - Use standalone components (Angular 17+) or NgModule based on project architecture
  - Implement state management with detected approach (use existing library if found, otherwise signals/RxJS)
  - Style components using detected approach (use existing if found, otherwise SCSS)
  - Implement authentication using detected approach (use existing if found, otherwise custom API-based with guards and interceptors)
  - Handle API integration with HttpClient, proper error handling, and loading states
  - Ensure responsive design and follow accessibility best practices (WCAG 2.1 AA compliance, Angular CDK A11y)
  - Manage subscriptions properly (async pipe or takeUntil pattern)
  - Optimize performance with OnPush, trackBy, lazy loading

  Communication:
  - If using tools different from preferred stack, inform user once about alternative benefits (only if advantageous)
  - Provide specific, measurable advantages
  - Always continue with detected approach for consistency"
- Expected output: Angular components (with version-appropriate patterns), TypeScript types with strict mode, state management implementation (using detected library or signals/RxJS default), styles using detected approach (or SCSS default), API integration with HttpClient and detected auth approach (or custom guards/interceptors), responsive styles, accessibility implementations with Angular CDK, unit tests with TestBed/Jasmine/Jest, optional note about alternative tools if advantageous
- Context: Validated requirements from Phase 1, existing project structure, Angular version and architecture (standalone vs NgModule), state management library, styling approach
- Action: Implement the complete Angular feature with production-ready code, following Angular best practices and project conventions.

### 3b. React/Next.js Feature Implementation (if React/Next.js detected)
- Use Task tool with subagent_type="frontend-mobile-development::frontend-developer"
- Prompt: "Implement the following frontend feature: $ARGUMENTS.

  Detection phase:
  - Check package.json to detect if Next.js is being used and its version
  - Check for app/ or pages/ directory structure
  - Detect state management: check for any existing library (continue if found), otherwise use Zustand
  - Detect styling: check for any existing approach (continue if found), otherwise use SCSS
  - Detect authentication: check for existing auth implementation (continue if found)
  - For each detected tool that differs from preferred stack: inform user about preferred tool benefits ONLY if they provide clear advantages, then continue with existing approach

  Implementation phase:
  - Build React components with proper TypeScript types using patterns appropriate for the detected framework and version
  - Implement state management with detected approach (use existing library if found, otherwise Zustand)
  - Style components using detected approach (use existing approach if found, otherwise SCSS)
  - Implement authentication using detected approach (use existing if found, otherwise custom API-based with JWT)
  - Handle API integration with proper error handling and loading states
  - Ensure responsive design and follow accessibility best practices (WCAG 2.1 AA compliance)
  - Include proper error boundaries and optimize for performance

  Communication:
  - If using tools different from preferred stack, inform user once about alternative benefits (only if advantageous)
  - Provide specific, measurable advantages (e.g., performance metrics, feature capabilities)
  - Always continue with detected approach for consistency"
- Expected output: React components (with Next.js-specific patterns if detected), TypeScript types, state management implementation (using detected library or Zustand default), styles using detected approach (or SCSS default), API integration code with detected auth approach (or custom API-based default), responsive styles, accessibility implementations, unit tests for components, optional note about alternative tools if they provide advantages
- Context: Validated requirements from Phase 1, existing project structure and design system, framework version and routing approach, state management library, styling approach
- Action: Implement the complete feature with production-ready code, following project conventions and framework-specific best practices.

## Phase 3: Security Audit & Compliance

### 4. Security Audit & Vulnerability Assessment
- Use Task tool with subagent_type="security-compliance::security-auditor"
- Prompt: "Perform comprehensive security audit of the implemented frontend feature: $ARGUMENTS.
  
  Frontend Security Assessment:
  - Review authentication implementation for security vulnerabilities (token storage, session management, CSRF protection)
  - Check for XSS vulnerabilities in component rendering and user input handling
  - Validate input sanitization and output encoding
  - Review API integration for secure communication (HTTPS, headers, CORS configuration)
  - Check for sensitive data exposure in client-side code or console logs
  - Verify proper error handling without information leakage
  - Assess security headers implementation (CSP, X-Frame-Options, HSTS)
  - Review dependencies for known vulnerabilities
  
  OWASP Top 10 Frontend Checks:
  - Broken Access Control: Verify proper authorization checks before sensitive operations
  - Cryptographic Failures: Check for secure data transmission and storage
  - Injection: Validate protection against XSS, SQL injection in API calls
  - Insecure Design: Review security architecture and patterns
  - Security Misconfiguration: Check for exposed credentials, debug mode, default configs
  - Vulnerable Components: Scan dependencies for security vulnerabilities
  - Authentication Failures: Review authentication implementation and session management
  - Data Integrity Failures: Check for secure data validation and integrity checks
  - Logging Failures: Verify security events are logged without sensitive data exposure
  - Server-Side Request Forgery: Check for SSRF vulnerabilities in API calls
  
  Compliance & Best Practices:
  - Accessibility compliance (WCAG 2.1 AA)
  - Data privacy considerations (localStorage vs httpOnly cookies)
  - TypeScript type safety (no 'any' types that could hide security issues)
  - Environment variable security (no secrets in client-side code)
  - Rate limiting considerations for API calls
  
  Output Requirements:
  - Security findings report with severity levels (Critical, High, Medium, Low)
  - Specific remediation recommendations for each finding
  - Code examples for security fixes where applicable
  - Compliance checklist status
  - Security score and risk assessment"
- Expected output: Comprehensive security audit report, vulnerability assessment with severity ratings, specific remediation steps, compliance status, security best practices recommendations
- Context: Implemented feature from Phase 2, authentication patterns used, API integration approach, detected framework and libraries
- Action: Identify security vulnerabilities and provide actionable remediation steps. If critical vulnerabilities are found, flag them for immediate attention before feature deployment.

### ⚠️ MANDATORY APPROVAL CHECKPOINT

**STOP HERE and request user approval before proceeding to Phase 4**

Present to user:
- Summary of implemented feature
- Security audit results (critical/high/medium/low findings)
- Overall quality assessment
- Any remaining concerns or recommendations

Ask user:
"The feature implementation and security audit are complete. 

📊 **Summary:**
- Feature: [Brief description]
- Security Status: [Status with findings count]
- Code Quality: [Assessment]

Please review the implementation and security findings.

✅ **Approve and Create PR**: Type 'yes', 'approve', or 'continue' to proceed to PR creation
🔄 **Request Changes**: Type 'no', 'changes needed', or describe what needs to be fixed to return to development phase
❓ **Need More Info**: Ask questions about the implementation or security findings"

**User Response Handling:**
- If APPROVED (yes/approve/continue): Proceed to Phase 4 (PR Creation)
- If CHANGES NEEDED (no/changes needed/specific requests): Return to Phase 2 (Development) with user's feedback
- If QUESTIONS: Answer questions and ask again for approval

**IMPORTANT**: Do NOT proceed to Phase 4 without explicit user approval.

## Phase 4: Pull Request Creation

**⚠️ This phase only executes after explicit user approval from Phase 3 checkpoint**

### 5. Create Pull Request
- Use Command tool with command="git-actions::create-pull-request"
- Arguments: Feature description and context from $ARGUMENTS
- The command will orchestrate the PR creation process through multiple steps:
  
  Required Information:
  - Redmine taskId: If not provided, ask user: 'Please provide the Redmine taskId for this feature (e.g., #12345)'
  
  
  Step 1 - Validate Prerequisites:
  - Verify all changes are committed (git status clean)
  - Ensure branch is pushed to remote
  - Validate branch name follows feature/{taskId}-* format
  - Check for GitHub CLI availability
  
  Step 2 - Gather Information:
  - Request Redmine taskId if not provided
  - Collect feature summary
  - Gather security audit results from Phase 3
  - Compile changes overview
  - Note testing approach
  
  Step 3 - Generate Content:
  - Create commit message template: [#taskId] Description
  - Generate PR title: [#taskId] Feature description
  - Build comprehensive PR description with:
    - Feature summary and purpose
    - Changes made
    - Security audit results
    - Testing performed
    - Complete checklist
  - Include Redmine task link
  
  Step 4 - Create PR:
  - Automated (if GitHub CLI/MCP available): Create PR directly
  - Manual (current): Provide step-by-step instructions with generated content"
- Expected output: PR created successfully (with URL if automated) or complete manual instructions with commit message template, PR title, and comprehensive description ready to use
- Context: Completed implementation from Phase 2, security audit results from Phase 3, Redmine taskId, branch name, git repository status
- Action: Orchestrate complete PR creation workflow through pull-request-manager agent, ensuring standardized format and comprehensive information.

## Configuration Options
- `stack`: Frontend technology stack (automatically detected from package.json)
- `state_management`: Automatically detected (Primary: Zustand, Adapts to: Redux Toolkit if found)
- `styling`: Automatically detected (Primary: SCSS, Adapts to: PostCSS if found)
- `authentication`: Custom API-based authentication (JWT, session tokens)
- `testing`: Testing framework (e.g., "Jest + React Testing Library", "Vitest")
- `security_level`: Security audit depth (default: "comprehensive", options: "comprehensive" for all checks, "essential" for critical only)

## Success Criteria

### Phase 1 Criteria
- Claude project initialized (CLAUDE.md exists in project root)
- Git branch validated (on feature/{taskId}-* branch, not develop/main)
- No uncommitted changes in working directory
- Development environment properly configured and running
- All dependencies installed and Node version correct
- Environment variables configured (from Shadows if needed)
- Development server running (existing or newly started)
- Requirements are clear, complete, and validated

### Phase 2 Criteria
- **Technology correctly identified**: Angular or React/Next.js detected from package.json
- **Ambiguity resolved**: If unclear, user was asked to specify technology
- **Appropriate agent used**: angular-developer for Angular, frontend-developer for React/Next.js

**For Angular projects:**
- Angular version and architecture correctly detected (standalone vs NgModule)
- Version-appropriate patterns applied (signals for 16+, control flow for 17+)
- OnPush change detection strategy used by default
- Proper subscription management implemented (async pipe or takeUntil)
- State management correctly detected and implemented (using existing or signals/RxJS default)
- Styling approach correctly detected and used (using existing or SCSS default)
- Authentication with guards and interceptors (using existing or custom API default)
- HttpClient used for API integration
- Angular CDK A11y patterns for accessibility (WCAG 2.1 AA)

**For React/Next.js projects:**
- Framework and version correctly detected (Next.js version and router type, or standard React)
- Next.js patterns match the installed version and router structure (App Router vs Pages Router)
- State management correctly detected and implemented (using existing or Zustand default)
- Styling approach correctly detected and used (using existing or SCSS default)
- Authentication implemented using detected approach (using existing or custom API default)

**Common criteria (both technologies):**
- TypeScript types are properly defined (no 'any' types, strict mode)
- Components are responsive and accessible (WCAG 2.1 AA)
- Error handling and loading states are implemented
- Code follows project conventions and style guide
- Frontend implementation follows framework-appropriate best practices
- Basic component/unit tests are included

### Phase 3 Criteria
- Comprehensive security audit completed
- No critical or high severity vulnerabilities remaining
- Authentication security validated (token storage, session management, CSRF protection)
- XSS and injection vulnerabilities addressed
- Security headers properly configured
- Dependencies scanned for vulnerabilities
- OWASP Top 10 checks passed
- Accessibility compliance verified (WCAG 2.1 AA)
- No sensitive data exposed in client-side code
- Security best practices implemented
- Remediation plan for any medium/low findings documented

### Approval Checkpoint Criteria
- Phase 3 security audit completed successfully
- Summary presented to user with all findings
- User explicitly approves to continue (yes/approve/continue)
- OR user requests changes with specific feedback (returns to Phase 2)

### Phase 4 Criteria (Only after approval)
- User approval received from Phase 3 checkpoint
- Redmine taskId provided by user
- All changes committed with descriptive message including taskId
- Pull Request created (manually for now) with:
  - Proper title format: [taskId] Brief description
  - Complete description including feature summary and security audit results
  - Link to Redmine task
  - Changes properly documented

## Coordination Notes

### Phase Flow
- Phase 1 must complete successfully before Phase 2 begins
- Phase 2 must complete successfully before Phase 3 begins
- Phase 3 must complete successfully and receive **USER APPROVAL** before Phase 4 begins
- **APPROVAL CHECKPOINT**: After Phase 3, workflow STOPS and requests user approval
- Any environment issues must be resolved before development starts
- Requirements clarification is mandatory if any ambiguity exists
- Development phase proceeds only with validated, clear requirements
- Security audit is mandatory before feature is considered production-ready
- **User can request changes**: If not approved, workflow returns to Phase 2 with feedback
- Pull Request creation requires Redmine taskId from user
- Pull Request creation only proceeds with explicit user approval

### Phase 1 (Setup & Requirements)
- Project-setup agent checks Claude initialization FIRST (CLAUDE.md must exist)
- If CLAUDE.md missing: Stops and instructs user to run /init, waits for user confirmation
- After user runs /init and confirms, re-checks and continues
- Checks git branch SECOND (must be feature/{taskId}-*)
- STOPS if on develop or main branches (directs user to create feature branch)
- STOPS if branch doesn't follow naming convention
- STOPS if there are uncommitted changes (directs user to commit or stash)
- Verifies environment, dependencies, .env files, and dev server
- STOPS if .env template exists but actual .env is missing (directs to Shadows)
- Checks for already running dev server before starting a new one
- Requirements-reviewer agent validates requirement completeness and clarity
- STOPS if critical information is missing and requests user clarification

### Phase 2 (Development)
- **Technology detection is mandatory FIRST**: Check package.json for @angular/core (Angular) or react/next (React/Next.js)
- **Ambiguity handling**: If both frameworks found, STOP and ask user which to use
- **Missing indicators**: If no clear framework found, STOP and ask user which technology to use
- **Angular projects**: Use angular-developer agent with Angular-specific patterns
  - Detects Angular version and architecture (standalone vs NgModule)
  - Applies version-appropriate patterns (signals for 16+, control flow for 17+)
  - Uses OnPush change detection, proper subscription management
- **React/Next.js projects**: Use frontend-developer agent with React-specific patterns
  - Detects Next.js version and router type (App Router vs Pages Router)
  - Applies framework-specific patterns based on detection
- Always detect and respect existing tooling (state management, styling, authentication)
- Continue with detected approaches for consistency, even if different from preferred stack
- Inform user about alternative tool benefits only when they provide clear, specific advantages
- Provide precise, measurable information about alternative approaches
- Maintain consistency with existing project patterns and architecture

### Phase 3 (Security Audit & Approval Checkpoint)
- Security-auditor agent performs comprehensive frontend security assessment
- Covers OWASP Top 10 vulnerabilities specific to frontend
- Reviews authentication implementation, XSS protection, data exposure
- Validates security headers, dependency vulnerabilities, and compliance
- Flags critical vulnerabilities for immediate remediation
- Provides specific, actionable remediation steps with code examples
- Documents security score and risk assessment
- **STOPS at approval checkpoint after audit completes**
- Presents summary to user with security findings
- Requests explicit approval to proceed to Phase 4
- If user requests changes: Returns to Phase 2 with feedback
- If user approves: Proceeds to Phase 4

### Phase 4 (Pull Request Creation)
- Uses git-actions::create-pull-request command to orchestrate PR workflow
- Command coordinates pull-request-manager agent through 4 steps:
  1. Prerequisites validation (commits, push, branch name, GitHub CLI)
  2. Information gathering (taskId, summary, security results, changes)
  3. Content generation (commit message, PR title, description)
  4. PR creation (automated or manual with instructions)
- Requests Redmine taskId from user if not already provided
- Verifies all changes are committed and branch is pushed
- Generates standardized PR title: [#taskId] Description
- Generates comprehensive PR description with:
  - Feature summary and changes
  - Security audit results from Phase 3
  - Testing information
  - Complete checklist
- Links to Redmine task for full traceability
- Provides step-by-step manual instructions (current implementation)
- Future: Will use GitHub CLI/MCP for fully automated PR creation

Feature to implement: $ARGUMENTS