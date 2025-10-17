Orchestrate frontend feature development:

[Extended thinking: This workflow coordinates specialized agents to deliver a frontend feature with proper setup validation and clear requirements. The workflow ensures the development environment is properly configured before starting development, validates requirement clarity, and then proceeds with implementation using best practices for modern React/Next.js development.]

## Phase 1: Setup & Requirements Validation

### 1. Project Setup Verification
- Use Task tool with subagent_type="frontend-orchestration::project-setup"
- Prompt: "Verify and setup the development environment for the project. Check if node_modules are installed, validate .nvmrc configuration, verify .npmrc if needed, and start the local development server."
- Expected output: Environment status report, confirmation of dependencies installed, Node version validation, dev server running status
- Context: Project root directory and package.json configuration
- Action: This step must complete successfully before proceeding. If any issues are found, they must be resolved first.

### 2. Requirements Review & Clarification
- Use Task tool with subagent_type="frontend-orchestration::requirements-reviewer"
- Prompt: "Review the following feature requirements for completeness and clarity: $ARGUMENTS. Identify any missing information, ambiguous specifications, or unclear requirements. If critical information is missing, stop and request clarification from the user."
- Expected output: Requirements validation report, list of clarifying questions (if any), approved requirements document
- Context: User's feature request and project context
- Action: If clarification is needed, halt the workflow and wait for user input. Only proceed when all requirements are clear and complete.

## Phase 2: Development

### 3. Frontend Feature Implementation
- Use Task tool with subagent_type="frontend-mobile-development::frontend-developer"
- Prompt: "Implement the following frontend feature: $ARGUMENTS. Build React/Next.js components with proper TypeScript types, implement state management, handle API integration with proper error handling and loading states, ensure responsive design, and follow accessibility best practices (WCAG 2.1 AA compliance). Include proper error boundaries and optimize for performance."
- Expected output: React components, TypeScript types, state management implementation, API integration code, responsive styles, accessibility implementations, unit tests for components
- Context: Validated requirements from Phase 1, existing project structure and design system
- Action: Implement the complete feature with production-ready code, following project conventions and best practices.

## Configuration Options
- `stack`: Frontend technology stack (default: "React/Next.js/TypeScript")
- `state_management`: Preferred state management solution (e.g., "Zustand", "Context API", "Redux Toolkit")
- `styling`: Styling approach (e.g., "Tailwind CSS", "CSS Modules", "styled-components")
- `testing`: Testing framework (e.g., "Jest + React Testing Library", "Vitest")

## Success Criteria
- Development environment properly configured and running
- All dependencies installed and Node version correct
- Requirements are clear, complete, and validated
- Frontend implementation follows React/Next.js best practices
- TypeScript types are properly defined (no 'any' types)
- Components are responsive and accessible
- Error handling and loading states are implemented
- Code follows project conventions and style guide
- Basic component tests are included

## Coordination Notes
- Phase 1 must complete successfully before Phase 2 begins
- Any environment issues must be resolved before development starts
- Requirements clarification is mandatory if any ambiguity exists
- Development phase proceeds only with validated, clear requirements
- Maintain consistency with existing project patterns and architecture

Feature to implement: $ARGUMENTS