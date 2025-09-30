---
model: claude-opus-4-1
---

# MDD Component Migration Cycle

Execute complete Migration Driven Development cycle for: $ARGUMENTS

[Extended thinking: This workflow implements the empirically-validated MDD methodology, incorporating all discovered patterns and avoiding all known anti-patterns from the RESMARK knowledge graph.]

## Pre-Flight Checks

Before starting, query the knowledge graph for:
- All CRITICAL directives that APPLY_TO component-migration
- All patterns that IMPLEMENT component-migration
- All anti-patterns with severity CRITICAL or HIGH

## Phase 1: Discovery & Analysis

### 1. Legacy Component Analysis
Use Task tool with subagent_type="legacy-modernizer"

Prompt: "Analyze legacy component: $ARGUMENTS
Focus on:
- Session management patterns (MUST use req.session.entityId)
- Database queries (identify all MongoDB operations)
- Business logic and validation rules
- UI states and user interactions
- Permission and multi-tenant requirements

Reference: resmark-nodejs/ directory (read-only)
Output: Complete requirements document with no assumptions"

### 2. Visual Baseline Capture
Use Task tool with subagent_type="ui-visual-validator"

Prompt: "Capture comprehensive visual baselines for: $ARGUMENTS
Requirements:
- All component states (default, loading, error, empty)
- Mobile and desktop viewports
- Different data scenarios
- Interaction states (hover, focus, active)

Store in: app/frontend/tests/visual/baselines/angular/
Output: Screenshot inventory with selectors"

### 3. Pattern Identification
Use Task tool with subagent_type="code-reviewer"

Prompt: "Identify applicable patterns and risks for migrating: $ARGUMENTS
Check knowledge graph for:
- Validated patterns that apply to this component type
- Anti-patterns commonly found in similar migrations
- Performance benchmarks from previous migrations
- Security considerations

Reference: .claude/knowledge-graph/
Output: Pattern applicability matrix"

## Phase 2: Implementation

### 4. Sandbox Environment Setup
Use Task tool with subagent_type="devops-troubleshooter"

Prompt: "Create ephemeral sandbox for: $ARGUMENTS
Requirements:
- Isolated Docker container
- Temporary git branch (will be destroyed on failure)
- Copy only required files (optimize for <1 second setup)
- MongoDB and Redis connections configured

Implementation: ai-workflow/sandbox-manager.js
Output: Sandbox ID and connection details"

### 5. Component Generation with Constraints
Use Task tool with subagent_type="frontend-developer"

Prompt: "Generate React component for: $ARGUMENTS

CRITICAL CONSTRAINTS (from knowledge graph):
- NEVER use mock data in services (directive-no-mock-data)
- ALWAYS use SessionService for auth (pattern-session-bridge)
- Must include error boundaries (antipattern-missing-error-boundaries)
- Use req.session.entityId ONLY (directive-session-pattern)
- No in-memory caching - Redis only (directive-no-memory-cache)
- Include feature flag wrapper (directive-feature-flags)

Required files:
- Component.tsx (main component)
- Component.test.tsx (>80% coverage)
- Component.stories.tsx (Storybook)
- useComponentData.ts (data hook if needed)

Use validation feedback loop pattern - iterate up to 5 times based on validation results."

### 6. Validation Gauntlet Execution
Use Task tool with subagent_type="code-reviewer"

Prompt: "Execute 7-stage validation for: $ARGUMENTS

Stage 1: Pre-generation intent verification
Stage 2: Anti-pattern detection (check all from knowledge graph)
Stage 3: Static analysis (TypeScript, ESLint)
Stage 4: Unit testing (minimum 80% coverage)
Stage 5: Integration testing (real MongoDB/Redis)
Stage 6: Visual regression (<5% difference threshold)
Stage 7: Security scan (no exposed secrets, XSS prevention)

If any stage fails, generate specific feedback for next iteration.
Maximum 5 iterations before human intervention required.

Output: Validation report with pass/fail per stage"

## Phase 3: Verification

### 7. Visual Comparison Analysis
Use Task tool with subagent_type="ui-visual-validator"

Prompt: "Perform detailed visual comparison for: $ARGUMENTS
- Compare React component against Angular baseline
- Generate diff images highlighting changes
- Calculate similarity percentage
- Flag any differences >5%
- Check responsive behavior

Output: Visual regression report with screenshots"

### 8. Integration Testing
Use Task tool with subagent_type="test-automator"

Prompt: "Execute comprehensive integration tests for: $ARGUMENTS
Requirements:
- Test with production MongoDB connection
- Verify session bridge functionality
- Check all API endpoints used
- Validate multi-tenant isolation
- Test error scenarios
- Verify feature flag toggling

Output: Integration test results with performance metrics"

## Phase 4: Promotion or Rollback

### 9. Success Path - Create PR
If validation passed, use Task tool with subagent_type="deployment-engineer"

Prompt: "Create pull request for validated component: $ARGUMENTS
Include in PR:
- Validation gauntlet results (all 7 stages)
- Visual regression comparisons
- Performance benchmarks vs Angular
- Patterns discovered/confirmed
- Any new anti-patterns identified
- Feature flag configuration

Cleanup sandbox environment after PR creation."

### 10. Failure Path - Cleanup
If validation failed after 5 iterations:

Use Task tool with subagent_type="devops-troubleshooter"

Prompt: "Clean up failed migration attempt for: $ARGUMENTS
- Destroy sandbox environment completely
- Remove temporary branches
- Document failure reasons
- Identify new anti-patterns
- Suggest approach modifications

Output: Failure analysis report for knowledge graph update"

## Phase 5: Knowledge Evolution

### 11. Pattern Discovery & Documentation
Use Task tool with subagent_type="docs-architect"

Prompt: "Document patterns discovered during migration of: $ARGUMENTS
- New patterns that led to success
- Anti-patterns that caused failures
- Performance optimizations found
- Unexpected dependencies discovered

Update: .claude/knowledge-graph/entities/
Format: YAML entities with relations"

### 12. Knowledge Graph Update
Use Task tool with subagent_type="context-manager"

Prompt: "Update knowledge graph with learnings from: $ARGUMENTS
- Add new Pattern entities if unique
- Update AntiPattern severity if needed
- Create new relations discovered
- Update performance benchmarks
- Record session context for future reference

Ensure all agents are notified of updates."

## Coordination Notes

- Sandbox isolation is CRITICAL - no direct repository writes
- Validation gauntlet is mandatory - no skipping stages
- Visual regression against Angular is required
- Session patterns must exactly match legacy
- Feature flags enable instant rollback
- Every discovery updates the knowledge graph

Component to migrate: $ARGUMENTS