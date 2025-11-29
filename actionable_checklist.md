

# Milestones

**Milestone M0: Project Setup & Infrastructure**
- **Start State:** No repository, no development environment, no API accounts
- **End State:** Complete development environment, repository initialized, CI/CD configured, API accounts created, team aligned

## Problem Statement

Customers ordering pizza delivery lack visibility into how delivery conditions (route, weather, timing) affect final pizza temperature, and cannot understand the impact of insulated delivery bags on maintaining pizza heat from oven to door. Without this information, customers may receive colder pizza than expected, leading to dissatisfaction and complaints about delivery quality.

## Objectives

- Set up complete development environment and project infrastructure
- Initialize monorepo structure with all necessary configuration
- Configure CI/CD pipeline for automated testing and deployment
- Obtain and secure all required external API accounts and keys
- Ensure team alignment on requirements and technical approach

## Expected Outcome

A fully configured development environment where team members can:
- Clone repository and run project locally
- Execute automated tests via CI/CD pipeline
- Access required external services (maps, weather) via secured API keys
- Build and deploy application to staging environment
- Begin development work on thermal model (M1) and API integrations (M2)

# Instructions for Agent
*   ### 0. Command Pyramid & Modes
    *   Obey the user’s explicit instructions first, then this block, then the checklist. Do not hide behind the checklist to ignore a direct user correction.
    *   Ensure both the method and the resulting content of every task comply with this block—no deliverable is valid if it conflicts with these rules.
    *   Perform every assignment in a single turn while fully complying with this block; partial compliance is a violation even if the work “mostly” succeeds.
    *   Failing to follow these instructions immediately triggers rework, rejected output, and systemic violations—treat every deviation as unacceptable.
    *   The Instructions for Agent block is an absolute firewall. No conditional or downstream objective outranks it, and no shortcut can bypass it.
    *   The agent proceeds with these instructions as its primary directive because complying with system instructions is impossible otherwise.
    *   Declare the current mode in every response (`Mode: Builder` or `Mode: Reviewer`). Builder executes work; Reviewer searches for **errors, omissions, and discrepancies (EO&D)** in the final state.
*   ### 1. Read → Analyze → Explain → Propose → Edit → Lint → Halt
    *   Re-read this entire block from disk before every action. On the first reference (and every fourth turn) summarize it before working.
    *   Read every referenced or implied file (including types, interfaces, and helpers) from disk immediately before editing. After editing, re-read to confirm the exact change.
    *   Follow the explicit cycle: READ the step + files → ANALYZE gaps → EXPLAIN the delta → PROPOSE the exact edit → EDIT a single file → LINT that file → HALT.
    *   Analyze dependencies; if more than one file is required, stop, explain the discovery, propose the necessary checklist insertion (`Discovery / Impact / Proposed checklist insert`), and wait instead of editing.
    *   Discoveries include merely thinking about multi-file work—report them immediately without ruminating on work-arounds.
    *   Explain & Propose: restate the plan in bullets and explicitly commit, “I will implement exactly this plan now,” noting the checklist step it fulfills.
    *   Edit exactly one file per turn following the plan. Never touch files you were not explicitly instructed to modify.
    *   Lint that file using internal tools and fix all issues.
    *   Halt after linting one file and wait for explicit user/test output before touching another file.
*   ### 2. TDD & Dependency Ordering
    *   One-file TDD cycle: RED test (desired green behavior) → implementation → GREEN test → lint. Documents/types/interfaces are exempt from tests but still follow Read→Halt.
    *   Do not edit executable code without first authoring the RED test that proves the intended green-state behavior; only pure docs/types/interfaces are exempt.
    *   Maintain bottom-up dependency order for both editing and testing: construct types/interfaces/helpers before consumers, then write consumer tests only after producers exist.
    *   Do not advance to another file until the current file’s proof (tests or documented exemption) is complete and acknowledged.
    *   The agent never runs tests directly; rely on provided outputs or internal reasoning while keeping the application in a provable state.
    *   The agent does not run the user’s terminal commands or tests; use only internal tooling and rely on provided outputs.
*   ### 3. Checklist Discipline
    *   Do not edit the checklist (or its statuses) without explicit instruction; when instructed, change only the specified portion using legal-style numbering.
    *   Execute exactly what the active checklist step instructs with no deviation or “creative interpretation.”
    *   Each numbered checklist step equals one file’s entire TDD cycle (deps → types → tests → implementation → proof). Preserve existing detail while adding new requirements.
    *   Document every edit within the checklist. If required edits are missing from the plan, explain the discovery, propose the new step, and halt instead of improvising.
    *   Never update the status of any work step (checkboxes or badges) without explicit instruction.
    *   Following a block of related checklist steps that complete a working implementation, include a commit with a proposed commit message. 
*   ### 4. Builder vs Reviewer Modes
    *   **Builder:** follow the Read→…→Halt loop precisely. If a deviation, blocker, or new requirement is discovered—or the current step simply cannot be completed as written—explain the problem, propose the required checklist change, and halt immediately.
    *   **Reviewer:** treat prior reasoning as untrusted. Re-read relevant files/tests from scratch and produce a numbered EO&D list referencing files/sections. Ignore checklist status or RED/GREEN history unless it causes a real defect. If no EO&D are found, state “No EO&D detected; residual risks: …”
*   ### 5. Strict Typing & Object Construction
    *   Use explicit types everywhere. No `any`, `as`, `as const`, inline ad-hoc types, or casts—except for Supabase clients and intentionally malformed objects in error-handling tests (use dedicated helpers and keep typing strict elsewhere). Every object and variable must be typed. 
    *   Always construct full objects that satisfy existing interfaces/tuples from the relevant type file. Compose complex objects from smaller typed components; never rely on defaults, fallbacks, or backfilling to “heal” missing data.
    *   Use type guards to prove and narrow types for the compiler when required.
    *   Never import entire libraries with *, never alias imports, never add "type" to type imports. 
    *   A ternary is not a type guard, a ternary is a default value. Default values are prohibited. 
*   ### 6. Plan Fidelity & Shortcut Ban
    *   Once a solution is described, implement exactly that solution and the user’s instruction. Expedient shortcuts are forbidden without explicit approval.
    *   If you realize you deviated, stop, report it, and wait for direction. Repeating corrected violations triggers halt-and-wait immediately.
    *   If your solution to a challenge is "rewrite the entire file", you have made an error. Stop, do not rewrite the file. Explain the problem to the user and await instruction. 
    *   Do not ruminate on how to work around the "only write to one file per turn". If you are even thinking about the need to work around that limit, you have made a discovery. Stop immediately, report the discovery to the user, and await instruction. 
    *   Refactors must preserve all existing functionality unless the user explicitly authorizes removals; log and identifier fidelity is mandatory.
*   ### 7. Dependency Injection & Architecture
    *   Use explicit dependency injection everywhere—pass every dependency with no hidden defaults or optional fallbacks.
    *   Build adapters/interfaces for every function and work bottom-up so dependencies compile before consumers. Preserve existing functionality, identifiers, and logging unless explicitly told otherwise.
    *   When a file exceeds 600 lines, stop and propose a logical refactoring to decompose the file into smaller parts providing clear SOC and DRY. 
*   ### 8. Testing Standards
    *   Tests assert the desired passing state (no RED/GREEN labels) and new tests are added to the end of the file. Each test covers exactly one behavior.
    *   Use real application functions/mocks, strict typing, and Deno std asserts. Tests must call out which production type/helper each mock mirrors so partial objects are not invented.
    *   Integration tests must exercise real code paths; unit tests stay isolated and mock dependencies explicitly. Never change assertions to match broken code—fix the code instead.
    *   Tests use the same types, objects, structures, and helpers as the real code, never create new fixtures only for tests - a test that relies on imaginary types or fixtures is invalid. 
    *   Prove the functional gap, the implemented fix, and regressions through tests before moving on; never assume success without proof.
*   ### 9. Logging, Defaults, and Error Handling
    *   Do not add or remove logging, defaults, fallbacks, or silent healing unless the user explicitly instructs you to do so.
    *   Adding console logs solely for troubleshooting is exempt from TDD and checklist obligations, but the exemption applies only to the logging statements themselves.
    *   Believe failing tests, linter flags, and user-reported errors literally; fix the stated condition before chasing deeper causes.
    *   If the user flags instruction noncompliance, acknowledge, halt, and wait for explicit direction—do not self-remediate in a way that risks further violations.
*   ### 10. Linting & Proof
    *   After each edit, lint the touched file and resolve every warning/error. Record lint/test evidence in the response (e.g., “Lint: clean via internal tool; Tests: not run per instructions”).
    *   Evaluate if a linter error can be resolved in-file, or out-of-file. Only resolve in-file linter errors, then report the out-of-file errors and await instruction. 
    *   Testing may produce unresolvable linter errors. Do not silence them with @es flags, create an empty target function, or other work-arounds. The linter error is sometimes itself proof of the RED state of the test. 
    *   Completion proof requires a lint-clean file plus GREEN test evidence (or documented exemption for types/docs).
*   ### 11. Reporting & Traceability
    *   Every response must include: mode declaration, confirmation that this block was re-read, plan bullets (Builder) or EO&D findings (Reviewer), checklist step references, and lint/test evidence.
    *   If tests were not run (per instruction), explicitly state why and list residual risks. If no EO&D are found, state that along with remaining risks.
    *   The agent uses only its own tools and never the user’s terminal.
*   ### 12. Output Constraints
    *   Never output large code blocks (entire files or multi-function dumps) in chat unless the user explicitly requests them.
    *   Never print an entire function and tell the user to paste it in; edit the file directly or provide the minimal diff required.

## Checklist-Specific Editing Rules

*   THE AGENT NEVER TOUCHES THE CHECKLIST UNLESS THEY ARE EXPLICITLY INSTRUCTED TO! 
*   When editing checklists, each numbered step (1, 2, 3, etc.) represents editing ONE FILE with a complete TDD cycle.
*   Sub-steps within each numbered step use legal-style numbering (1.a, 1.b, 1.a.i, 1.a.ii, etc.) for the complete TDD cycle for that file.
*   All changes to a single file are described and performed within that file's numbered step.
*   Types files (interfaces, enums) are exempt from RED/GREEN testing requirements.
*   Each file edit includes: RED test → implementation → GREEN test → optional refactor.
*   Steps are ordered by dependency (lowest dependencies first).
*   Preserve all existing detail and work while adding new requirements.
*   Use proper legal-style nesting for sub-steps within each file edit.
*   NEVER create multiple top-level steps for the same file edit operation.
*   Adding console logs is not required to be detailed in checklist work. 

### Example Checklist

*   `[ ]`   1. **Title** Objective
    *   `[ ]`   1.a. [DEPS] A list explaining dependencies of the function, its signature, and its return shape
        *   `[ ]` 1.a.i. eg. `function(something)` in `file.ts` provides this or that
    *   `[ ]`   1.b. [TYPES] A list strictly typing all the objects used in the function
    *   `[ ]`   1.c. [TEST-UNIT] A list explaining the test cases
        *   `[ ]` 1.c.i. Assert `function(something)` in `file.ts` acts a certain way 
    *   `[ ]`   1.d. [SPACE] A list explaining the implementation requirements
        *   `[ ]` 1.d.i. Implement `function(something)` in `file.ts` acts a certain way 
    *   `[ ]`   1.d. [TEST-UNIT] Rerun and expand test proving the function
        *   `[ ]` 1.d.i. Implement `function(something)` in `file.ts` acts a certain way 
    *   `[ ]`   1.d. [TEST-INT] If there is a chain of functions that work together, prove it
        *   `[ ]` 1.d.i. For every cross-function interaction, assert `thisFunction(something)` in `this_file.ts` acts a certain way towards `thatFunction(other)` in `that_file.ts`
    *   `[ ]`   1.d. [CRITERIA] A list explaining the acceptence criteria to consider the work complete and correct. 
    *   `[ ]`   1.e. [COMMIT] A commit that explains the function and its proofs

*   `[ ]`   2. **Title** Objective
    *   `[ ]`   2.a. [DEPS] Low level providers are always build before high level consumers (DI/DIP)
    *   `[ ]`   2.b. [TYPES] DI/DIP and strict typing ensures unit tests can always run 
    *   `[ ]`   2.c. [TEST-UNIT] All functions matching defined external objects and acting as asserted helps ensure integration tests pass

## Legend - You must use this EXACT format. Do not modify it, adapt it, or "improve" it. The bullets, square braces, ticks, nesting, and numbering are ABSOLUTELY MANDATORY and UNALTERABLE. 

*   `[ ]` 1. Unstarted work step. Each work step will be uniquely named for easy reference. We begin with 1.
    *   `[ ]` 1.a. Work steps will be nested as shown. Substeps use characters, as is typical with legal documents.
        *   `[ ]` 1. a. i. Nesting can be as deep as logically required, using roman numerals, according to standard legal document numbering processes.
*   `[✅]` Represents a completed step or nested set.
*   `[🚧]` Represents an incomplete or partially completed step or nested set.
*   `[⏸️]` Represents a paused step where a discovery has been made that requires backtracking or further clarification.
*   `[❓]` Represents an uncertainty that must be resolved before continuing.
*   `[🚫]` Represents a blocked, halted, or stopped step or has an unresolved problem or prior dependency to resolve before continuing.

## Component Types and Labels

*   `[DB]` Database Schema Change (Migration)
*   `[RLS]` Row-Level Security Policy
*   `[BE]` Backend Logic (Edge Function / RLS / Helpers / Seed Data)
*   `[API]` API Client Library (`@paynless/api` - includes interface definition in `interface.ts`, implementation in `adapter.ts`, and mocks in `mocks.ts`)
*   `[STORE]` State Management (`@paynless/store` - includes interface definition, actions, reducers/slices, selectors, and mocks)
*   `[UI]` Frontend Component (e.g., in `apps/web`, following component structure rules)
*   `[CLI]` Command Line Interface component/feature
*   `[IDE]` IDE Plugin component/feature
*   `[TEST-UNIT]` Unit Test Implementation/Update
*   `[TEST-INT]` Integration Test Implementation/Update (API-Backend, Store-Component, RLS)
*   `[TEST-E2E]` End-to-End Test Implementation/Update
*   `[DOCS]` Documentation Update (READMEs, API docs, user guides)
*   `[REFACTOR]` Code Refactoring Step
*   `[PROMPT]` System Prompt Engineering/Management
*   `[CONFIG]` Configuration changes (e.g., environment variables, service configurations)
*   `[COMMIT]` Checkpoint for Git Commit (aligns with "feat:", "test:", "fix:", "docs:", "refactor:" conventions)
*   `[DEPLOY]` Checkpoint for Deployment consideration after a major phase or feature set is complete and tested.

# Work Breakdown Structure

## Milestone M0: Project Setup & Infrastructure

*   `[ ]`   1. **[CONFIG] Repository & Monorepo Structure Setup** - Initialize Git repository and configure monorepo structure
    *   `[ ]`   1.a. [DEPS] No dependencies - foundational setup task
    *   `[ ]`   1.b. [CONFIG] Initialize Git repository with proper .gitignore configuration
        *   `[ ]` 1.b.i. Create `.gitignore` file with Node.js, TypeScript, and monorepo patterns
        *   `[ ]` 1.b.ii. Ensure API keys and environment variables are excluded
    *   `[ ]`   1.c. [CONFIG] Set up Turborepo monorepo structure
        *   `[ ]` 1.c.i. Create root `package.json` with workspace configuration
        *   `[ ]` 1.c.ii. Create `turbo.json` configuration file
        *   `[ ]` 1.c.iii. Create directory structure: `apps/`, `packages/`, `tools/`, `docs/`
    *   `[ ]`   1.d. [CONFIG] Initialize workspace package.json files
        *   `[ ]` 1.d.i. Create `apps/frontend/package.json` with Next.js dependencies
        *   `[ ]` 1.d.ii. Create `apps/backend/package.json` with Netlify Functions dependencies
        *   `[ ]` 1.d.iii. Create `packages/types/package.json` for shared types
        *   `[ ]` 1.d.iv. Create `packages/utils/package.json` for shared utilities
        *   `[ ]` 1.d.v. Create `packages/thermal-model/package.json` for thermal model package
        *   `[ ]` 1.d.vi. Create `packages/map-adapter/package.json` for map adapter package
        *   `[ ]` 1.d.vii. Create `packages/weather-adapter/package.json` for weather adapter package
    *   `[ ]`   1.e. [DOCS] Create initial README files
        *   `[ ]` 1.e.i. Create root `README.md` with project overview and setup instructions
        *   `[ ]` 1.e.ii. Create package-specific README files where applicable
    *   `[ ]`   1.f. [CRITERIA] Repository initialized, monorepo structure created, all package.json files exist, README files created, .gitignore properly configured
    *   `[ ]`   1.g. [COMMIT] feat: initialize repository with monorepo structure

*   `[ ]`   2. **[CONFIG] TypeScript Configuration** - Configure TypeScript with strict mode across all packages
    *   `[ ]`   2.a. [DEPS] Depends on repository structure (Step 1)
    *   `[ ]`   2.b. [CONFIG] Create root TypeScript configuration
        *   `[ ]` 2.b.i. Create root `tsconfig.json` with strict mode enabled
        *   `[ ]` 2.b.ii. Configure base compiler options (target, module, lib)
        *   `[ ]` 2.b.iii. Set up path aliases for workspace packages
    *   `[ ]`   2.c. [CONFIG] Create package-specific tsconfig.json files
        *   `[ ]` 2.c.i. Create `apps/frontend/tsconfig.json` extending root config
        *   `[ ]` 2.c.ii. Create `apps/backend/tsconfig.json` extending root config
        *   `[ ]` 2.c.iii. Create `packages/*/tsconfig.json` files extending root config
    *   `[ ]`   2.d. [CRITERIA] All TypeScript config files created, strict mode enabled, path aliases configured, packages can reference each other
    *   `[ ]`   2.e. [COMMIT] feat: configure TypeScript with strict mode across monorepo

*   `[ ]`   3. **[CONFIG] Development Environment Configuration Files** - Set up code quality tools and editor configuration
    *   `[ ]`   3.a. [DEPS] Depends on repository structure (Step 1)
    *   `[ ]`   3.b. [CONFIG] Configure ESLint
        *   `[ ]` 3.b.i. Create root `.eslintrc.js` or `.eslintrc.json` with Next.js, TypeScript rules
        *   `[ ]` 3.b.ii. Install ESLint dependencies in root package.json
        *   `[ ]` 3.b.iii. Configure ESLint for monorepo workspace structure
    *   `[ ]`   3.c. [CONFIG] Configure Prettier
        *   `[ ]` 3.c.i. Create `.prettierrc` configuration file
        *   `[ ]` 3.c.ii. Create `.prettierignore` file
        *   `[ ]` 3.c.iii. Install Prettier dependencies
    *   `[ ]`   3.d. [CONFIG] Configure EditorConfig
        *   `[ ]` 3.d.i. Create `.editorconfig` file with consistent editor settings
    *   `[ ]`   3.e. [CONFIG] Set up Husky and lint-staged
        *   `[ ]` 3.e.i. Install Husky for git hooks
        *   `[ ]` 3.e.ii. Configure pre-commit hook for linting
        *   `[ ]` 3.e.iii. Configure lint-staged to run on staged files only
        *   `[ ]` 3.e.iv. Create `.husky/pre-commit` hook file
    *   `[ ]`   3.f. [CRITERIA] ESLint, Prettier, EditorConfig, Husky, and lint-staged all configured and working
    *   `[ ]`   3.g. [COMMIT] feat: configure code quality tools (ESLint, Prettier, Husky)

*   `[ ]`   4. **[CONFIG] CI/CD Pipeline Configuration** - Set up GitHub Actions workflows for automated testing and deployment
    *   `[ ]`   4.a. [DEPS] Depends on repository structure (Step 1)
    *   `[ ]`   4.b. [CONFIG] Create GitHub Actions workflow for tests
        *   `[ ]` 4.b.i. Create `.github/workflows/test.yml` file
        *   `[ ]` 4.b.ii. Configure workflow to run on pull requests
        *   `[ ]` 4.b.iii. Set up Node.js environment
        *   `[ ]` 4.b.iv. Configure pnpm installation
        *   `[ ]` 4.b.v. Add steps to run linting and type checking
        *   `[ ]` 4.b.vi. Add steps to run unit tests
    *   `[ ]`   4.c. [CONFIG] Create GitHub Actions workflow for build and deployment
        *   `[ ]` 4.c.i. Create `.github/workflows/deploy.yml` file
        *   `[ ]` 4.c.ii. Configure workflow to run on merge to main branch
        *   `[ ]` 4.c.iii. Add steps to build frontend and backend
        *   `[ ]` 4.c.iv. Configure Netlify deployment integration
        *   `[ ]` 4.c.v. Set up environment variable secrets
    *   `[ ]`   4.d. [CONFIG] Configure GitHub repository secrets
        *   `[ ]` 4.d.i. Document required secrets for GitHub Actions
        *   `[ ]` 4.d.ii. Note: Actual secret values added via GitHub UI (not in code)
    *   `[ ]`   4.e. [CRITERIA] CI/CD workflows created, test workflow runs on PRs, deploy workflow configured, secrets documented
    *   `[ ]`   4.f. [COMMIT] feat: configure CI/CD pipeline with GitHub Actions

*   `[ ]`   5. **[CONFIG] Environment Variables Configuration** - Set up environment variable templates and documentation
    *   `[ ]`   5.a. [DEPS] No code dependencies, configuration only
    *   `[ ]`   5.b. [CONFIG] Create environment variable template files
        *   `[ ]` 5.b.i. Create `.env.example` file with all required variables
        *   `[ ]` 5.b.ii. Create `.env.local.example` for local development
        *   `[ ]` 5.b.iii. Document each environment variable's purpose
    *   `[ ]`   5.c. [DOCS] Document environment setup process
        *   `[ ]` 5.c.i. Add environment variable setup instructions to README
        *   `[ ]` 5.c.ii. Document where to obtain API keys
        *   `[ ]` 5.c.iii. Document Netlify and Supabase environment variable configuration
    *   `[ ]`   5.d. [CRITERIA] Environment variable templates created, documentation complete, setup instructions clear
    *   `[ ]`   5.e. [COMMIT] docs: add environment variable configuration templates and documentation

*   `[ ]`   6. **[DOCS] Development Setup Documentation** - Create comprehensive setup and development documentation
    *   `[ ]`   6.a. [DEPS] Depends on repository structure and configuration files
    *   `[ ]`   6.b. [DOCS] Create development setup guide
        *   `[ ]` 6.b.i. Document Node.js version requirements
        *   `[ ]` 6.b.ii. Document pnpm installation and usage
        *   `[ ]` 6.b.iii. Document local development setup steps
        *   `[ ]` 6.b.iv. Document how to run tests
        *   `[ ]` 6.b.v. Document how to build and run locally
    *   `[ ]`   6.c. [DOCS] Create contribution guidelines
        *   `[ ]` 6.c.i. Document coding standards
        *   `[ ]` 6.c.ii. Document commit message conventions
        *   `[ ]` 6.c.iii. Document pull request process
    *   `[ ]`   6.d. [CRITERIA] Setup documentation complete, contribution guidelines clear, developers can successfully set up environment
    *   `[ ]`   6.e. [COMMIT] docs: add development setup and contribution guidelines

*   `[ ]`   7. **[CONFIG] Package Manager Configuration** - Configure pnpm workspace and package dependencies
    *   `[ ]`   7.a. [DEPS] Depends on repository structure (Step 1)
    *   `[ ]`   7.b. [CONFIG] Configure pnpm workspace
        *   `[ ]` 7.b.i. Create `pnpm-workspace.yaml` file
        *   `[ ]` 7.b.ii. Configure workspace packages paths
    *   `[ ]`   7.c. [CONFIG] Set up root package.json scripts
        *   `[ ]` 7.c.i. Add scripts for building all packages
        *   `[ ]` 7.c.ii. Add scripts for running tests across workspace
        *   `[ ]` 7.c.iii. Add scripts for linting across workspace
        *   `[ ]` 7.c.iv. Add scripts for type checking
    *   `[ ]`   7.d. [CRITERIA] pnpm workspace configured, root scripts functional, workspace packages recognized
    *   `[ ]`   7.e. [COMMIT] feat: configure pnpm workspace and root build scripts

*   `[ ]`   8. **[CONFIG] Netlify Configuration** - Configure Netlify for frontend hosting and serverless functions
    *   `[ ]`   8.a. [DEPS] Depends on repository structure (Step 1)
    *   `[ ]`   8.b. [CONFIG] Create Netlify configuration file
        *   `[ ]` 8.b.i. Create `netlify.toml` configuration file
        *   `[ ]` 8.b.ii. Configure build settings for Next.js frontend
        *   `[ ]` 8.b.iii. Configure serverless functions directory
        *   `[ ]` 8.b.iv. Configure redirect rules
        *   `[ ]` 8.b.v. Configure environment variables structure
    *   `[ ]`   8.c. [DOCS] Document Netlify setup process
        *   `[ ]` 8.c.i. Document how to connect repository to Netlify
        *   `[ ]` 8.c.ii. Document environment variable configuration in Netlify UI
    *   `[ ]`   8.d. [CRITERIA] netlify.toml created, configuration complete, setup documented
    *   `[ ]`   8.e. [COMMIT] feat: configure Netlify for deployment

*   `[ ]`   9. **[DOCS] API Accounts Setup Documentation** - Document external service account creation and API key management
    *   `[ ]`   9.a. [DEPS] No code dependencies, documentation only
    *   `[ ]`   9.b. [DOCS] Document Google Maps Platform setup
        *   `[ ]` 9.b.i. Document account creation process
        *   `[ ]` 9.b.ii. Document API key creation and restrictions
        *   `[ ]` 9.b.iii. Document required APIs (Maps, Directions, Geocoding, Places)
        *   `[ ]` 9.b.iv. Document billing and quota management
    *   `[ ]`   9.c. [DOCS] Document OpenWeatherMap setup
        *   `[ ]` 9.c.i. Document account creation process
        *   `[ ]` 9.c.ii. Document API key creation
        *   `[ ]` 9.c.iii. Document required API endpoints
        *   `[ ]` 9.c.iv. Document rate limits and free tier usage
    *   `[ ]`   9.d. [DOCS] Document Supabase setup (optional)
        *   `[ ]` 9.d.i. Document account creation
        *   `[ ]` 9.d.ii. Document project setup
        *   `[ ]` 9.d.iii. Document database connection string
    *   `[ ]`   9.e. [DOCS] Document Sentry setup
        *   `[ ]` 9.e.i. Document account creation
        *   `[ ]` 9.e.ii. Document project setup
        *   `[ ]` 9.e.iii. Document DSN configuration
    *   `[ ]`   9.f. [CRITERIA] All API account setup processes documented, team knows how to obtain keys
    *   `[ ]`   9.g. [COMMIT] docs: add API account setup documentation

*   `[ ]`   10. **[CONFIG] Test Framework Configuration** - Configure Vitest for unit testing across packages
    *   `[ ]`   10.a. [DEPS] Depends on repository structure and TypeScript configuration
    *   `[ ]`   10.b. [CONFIG] Set up Vitest in root
        *   `[ ]` 10.b.i. Install Vitest and related dependencies in root package.json
        *   `[ ]` 10.b.ii. Create `vitest.config.ts` configuration file
        *   `[ ]` 10.b.iii. Configure test file patterns
        *   `[ ]` 10.b.iv. Configure test environment and globals
        *   `[ ]` 10.b.v. Configure coverage settings
    *   `[ ]`   10.c. [CONFIG] Create test utilities and helpers
        *   `[ ]` 10.c.i. Create test helper files for common testing patterns
        *   `[ ]` 10.c.ii. Document test utilities usage
    *   `[ ]`   10.d. [CRITERIA] Vitest configured, test commands functional, coverage configured
    *   `[ ]`   10.e. [COMMIT] feat: configure Vitest test framework

*   `[ ]`   11. **[CONFIG] Playwright E2E Test Configuration** - Configure Playwright for end-to-end testing
    *   `[ ]`   11.a. [DEPS] Depends on repository structure and frontend setup (future)
    *   `[ ]`   11.b. [CONFIG] Install and configure Playwright
        *   `[ ]` 11.b.i. Install Playwright dependencies
        *   `[ ]` 11.b.ii. Create `playwright.config.ts` configuration file
        *   `[ ]` 11.b.iii. Configure browser targets
        *   `[ ]` 11.b.iv. Configure test directory structure
        *   `[ ]` 11.b.v. Set up test fixtures
    *   `[ ]`   11.c. [DOCS] Document E2E testing approach
        *   `[ ]` 11.c.i. Document how to write E2E tests
        *   `[ ]` 11.c.ii. Document test data management
    *   `[ ]`   11.d. [CRITERIA] Playwright configured, E2E test structure ready, documentation complete
    *   `[ ]`   11.e. [COMMIT] feat: configure Playwright for E2E testing

*   `[ ]`   12. **[COMMIT] Milestone M0 Complete** - Final checkpoint for milestone completion
    *   `[ ]`   12.a. [CRITERIA] All M0 tasks complete, repository accessible, development environment functional, CI/CD working, documentation complete
    *   `[ ]`   12.b. [COMMIT] feat: complete milestone M0 - project setup and infrastructure
