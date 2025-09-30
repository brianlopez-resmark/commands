# RESMARK Constraints - Bedrock Principles

This document contains the **non-negotiable, empirically-validated constraints** that MUST be enforced in ALL RESMARK code generation. These are not suggestions - they are laws derived from real migration attempts and documented failures.

## Critical Directives (Severity: CRITICAL)

Violation of ANY critical directive = immediate BLOCK. No exceptions.

### 1. NO Mock Data in Services (directive-no-mock-data)
```javascript
// ❌ NEVER DO THIS - AI consistently generates this pattern
async getUsers() {
  return [{ id: 1, name: 'Mock User' }];  // CRITICAL VIOLATION
}

// ✅ ALWAYS DO THIS
async getUsers() {
  return await this.userRepository.findByEntity(this.entityId);
}
```
**Source**: Knowledge graph anti-patterns, documented in migration attempts
**Impact**: Integration failures, false test passes, production crashes

### 2. Session Pattern Sacred (directive-session-pattern)
```javascript
// ❌ AI HALLUCINATIONS - These patterns DO NOT EXIST
req.user.businessEntities      // DOES NOT EXIST
req.session.permissions        // WRONG LOCATION
req.currentEntity              // NOT IN LEGACY
req.user.entityId              // INCORRECT PATH

// ✅ ONLY VALID PATTERN - From legacy analysis
req.session.entityId           // Current business entity
req.session.userAccessMap      // Permissions mapping
req.session.businessEntityList // Available entities
```
**Source**: Legacy codebase analysis in `/controllers/secure/index.js`
**Impact**: Complete authentication failure, security violations

### 3. NO In-Memory Caching (directive-no-memory-cache)
```javascript
// ❌ MEMORY LEAK IN PRODUCTION
const rateLimitMap = new Map();
const cache = {};

// ✅ PRODUCTION-READY
const rateLimitKey = `rate_limit_${entityId}_${userId}`;
await redis.setex(rateLimitKey, ttl, value);
```
**Source**: Documented anti-pattern from migration failures
**Impact**: Memory leaks, server crashes, scaling failures

### 4. NO Hardcoded Bypasses (directive-no-hardcoded-bypass)
```javascript
// ❌ SECURITY VULNERABILITY
if (entityId === 'test') {
  return mockData;  // NEVER DO THIS
}

// ✅ USE FEATURE FLAGS
if (featureFlags['test_mode']) {
  return testData;
}
```
**Source**: Security audit findings
**Impact**: Security holes, production data exposure

### 5. Legacy is Immutable (directive-legacy-immutable)
- `resmark-nodejs/` directory is READ-ONLY
- NEVER modify legacy code
- Use as reference only
**Source**: CLAUDE.md master directive
**Impact**: Breaking source of truth, losing business logic

### 6. Visual Regression Required (directive-visual-regression)
- Every UI component MUST have visual tests
- Existing test suite lacks visual coverage
- Capture Angular baseline BEFORE migration
**Source**: CLAUDE.md September 2025 discovery - "17,800 existing tests provide 0% visual regression coverage"
**Impact**: UI regressions, broken user experience

### 7. Ephemeral Sandboxes Only (directive-ephemeral-sandbox)
- AI NEVER writes directly to main branches
- All generation in isolated Docker environments
- Sandbox destroyed on failure
**Source**: WORKSPACE_SETUP_TASKS.md architectural decision
**Impact**: Repository contamination, broken builds

### 8. Feature Flags Required (directive-feature-flags)
- Every AI-generated feature wrapped in flag
- Enables instant rollback without deployment
```javascript
// ✅ REQUIRED PATTERN
if (featureFlags['ai_search_enabled']) {
  return <AISearchComponent />
} else {
  return <LegacySearchBridge />
}
```
**Source**: CLAUDE.md critical directives
**Impact**: No rollback capability, production outages

### 9. Validation Gauntlet Mandatory (directive-validation-gauntlet)
All code MUST pass 7 stages:
1. Pre-generation intent
2. Anti-pattern detection
3. Static analysis
4. Unit testing (80% minimum)
5. Integration testing
6. Visual regression
7. Security scan
**Source**: WORKSPACE_SETUP_TASKS.md validation gates
**Impact**: Unvalidated code in production

## High-Priority Patterns (Severity: HIGH)

### Session Bridge Pattern (pattern-session-bridge)
```javascript
// Backend: SessionService handles complexity
const session = await SessionService.reconstructSession(email, entityId);
req.session.entityId = session.entityId;

// Frontend: Simple consumption via hook
const { entityId, permissions, hasPermission } = useSession();
```
**Source**: Documented in knowledge graph patterns

### Database Connection Pattern (pattern-database-connection)
```javascript
// Always use connection pool
const result = await this.db
  .collection('users')
  .find({ entityId: req.session.entityId })
  .toArray();
```
**Source**: Migration best practices

### Error Boundary Pattern (pattern-error-boundary)
```javascript
// Every React component tree needs protection
<ErrorBoundary fallback={<ErrorFallback />}>
  <Component />
</ErrorBoundary>
```
**Source**: Anti-pattern detection - missing error boundaries

## Performance Targets

These are requirements from CLAUDE.md:

- **Session Query**: <50ms target
- **Component Load**: <200ms target
- **API Response**: <100ms target
- **Visual Diff**: <5% threshold
- **Test Coverage**: >80% minimum

## Anti-Pattern Detection Rules

### Critical Anti-Patterns (Instant BLOCK)
1. `return [{` in service files → Mock data
2. `req.user.` anywhere → Wrong session pattern
3. `new Map()` for caching → Memory leak
4. `=== 'test'` → Hardcoded bypass

### High Anti-Patterns (Fix Required)
5. Complex TypeScript in discovery phase
6. Missing visual tests for components
7. Direct writes to main/develop branches
8. `process.env.X || 'default'` → Unsafe defaults

### Medium Anti-Patterns (Warnings)
9. `// TODO` or `// FIXME` markers
10. Mixed async/sync patterns
11. Missing error boundaries
12. `SELECT * FROM` queries

## Injection Instructions

When this document is injected into ANY command or agent:

1. **Prepend to system prompt**: "You MUST adhere to ALL RESMARK Constraints. These are non-negotiable."
2. **Add validation step**: "Before returning code, verify against RESMARK Constraints"
3. **Include in instructions**: "Run resmark-validate-output on all generated code"

## Evidence Base

These constraints are derived from:
- **Knowledge Graph**: `.claude/knowledge-graph/entities/` containing documented patterns and anti-patterns
- **Project Documentation**: CLAUDE.md master directives and discovered failures
- **Legacy Analysis**: Actual session patterns from `resmark-nodejs/controllers/secure/index.js`
- **Meeting Notes**: `2025-08-13_ai_validation_pivot.md` defining R&D model
- **Empirical Discovery**: Anti-patterns documented through actual migration attempts

Key documented findings:
- AI consistently generates mock data in service layers (documented in anti-patterns)
- AI frequently hallucinates incorrect session patterns (documented in directives)
- Visual regression gap discovered in September 2025 analysis
- MDD methodology proven through iterative discovery

## Enforcement

1. **Every code generation** must validate against these constraints
2. **No exceptions** for "simple" or "quick" tasks
3. **Violations block PR** creation
4. **Results attached to PRs** for audit trail

## Updates

When new patterns are discovered:
1. Add to this document with evidence
2. Update `resmark-validate-output.md`
3. Notify all commands via resmark-wrapper
4. Re-validate recent code

---

**Remember**: These constraints are NOT optional. They are the difference between success and failure in the RESMARK modernization project.