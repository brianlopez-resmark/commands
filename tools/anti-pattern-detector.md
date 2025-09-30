# RESMARK Anti-Pattern Detector

Detect known anti-patterns in code based on empirically discovered failures: $ARGUMENTS

[Extended thinking: This tool queries the RESMARK knowledge graph for all known anti-patterns and validates code against them. These patterns were discovered through real migration failures and cause production issues.]

## Detection Process

Use Task tool with subagent_type="code-reviewer"

Prompt: "Perform comprehensive anti-pattern detection on: $ARGUMENTS

## CRITICAL Anti-Patterns to Detect (Severity: CRITICAL)

### 1. Direct Database Mock (antipattern-direct-database-mock)
Search for patterns like:
- `return [{ id: 1, name: 'Mock User' }]` in service methods
- Hardcoded arrays/objects returned from data access layers
- Test data embedded in production code
Impact: Integration failures, false confidence

### 2. Wrong Session Pattern (antipattern-wrong-session-pattern)
Search for AI hallucinations:
- `req.user.businessEntities` (doesn't exist!)
- `req.session.permissions` (wrong location)
- `req.currentEntity` (not in legacy)
- Any pattern except `req.session.entityId`
Impact: Complete auth failure, security violations

### 3. In-Memory Map Caching (antipattern-memory-map-cache)
Search for:
- `new Map()` used for caching
- Plain objects used as caches without TTL
- Rate limiting without Redis
- Any cache without automatic expiration
Impact: Memory leaks, scaling failures

### 4. Hardcoded Test Bypasses (antipattern-hardcoded-test-bypass)
Search for:
- `if (entityId === 'test')`
- Special cases for specific entity IDs
- Hardcoded development bypasses
- Mock data returns based on entity
Impact: Security vulnerabilities

## HIGH Severity Anti-Patterns

### 5. TypeScript Complexity Trap (antipattern-typescript-complexity-trap)
Check for:
- Complex type definitions in discovery phase
- Over-engineered generics before requirements clear
- Type gymnastics blocking progress
Recommendation: Use JavaScript first, TypeScript after validation

### 6. Missing Visual Validation (antipattern-missing-visual-validation)
Check for:
- Components without visual regression tests
- Missing Playwright visual comparisons
- No baseline screenshots referenced
Impact: UI regressions, UX degradation

### 7. Direct Git Contamination (antipattern-direct-git-contamination)
Check for:
- Direct writes to main/develop branches
- Missing sandbox isolation
- Commits of unvalidated code
Impact: Repository pollution

### 8. Unvalidated Environment Access (antipattern-unvalidated-environment-access)
Search for:
- Direct `process.env.X` without defaults
- Missing environment validation
- No error handling for missing vars
Impact: Runtime failures

## MEDIUM Severity Anti-Patterns

### 9. Incomplete Implementation (antipattern-incomplete-implementation)
Search for:
- `// TODO` comments
- `throw new Error('Not implemented')`
- `// FIXME` markers
- Empty function bodies
Impact: False positive tests

### 10. Synchronous/Asynchronous Mixing (antipattern-synchronous-async-mixing)
Check for:
- Missing `await` keywords
- Callbacks mixed with promises
- `.then()` chains with async/await
Impact: Race conditions

### 11. Missing Error Boundaries (antipattern-missing-error-boundaries)
Check React components for:
- No error boundary wrapper
- Missing componentDidCatch/ErrorBoundary
- No fallback UI for errors
Impact: Full app crashes

## Analysis Output Format

For each detected anti-pattern, provide:
1. **Location**: File path and line numbers
2. **Severity**: CRITICAL/HIGH/MEDIUM
3. **Pattern ID**: Reference to knowledge graph entity
4. **Impact**: What will break in production
5. **Fix**: Specific correction needed
6. **Example**: Show the correct pattern

## Validation Rules

Apply these rules from .claude/knowledge-graph/entities/anti-patterns/:
- All patterns in known-anti-patterns.yaml
- Check frequency statistics (e.g., 'AI generates in 60% of attempts')
- Reference empirical evidence from migrations

## Integration with Validation Gauntlet

This detection runs as Stage 2 of the 7-stage validation pipeline.
If any CRITICAL anti-pattern is found, block progression immediately.
HIGH severity patterns require fix before human review.
MEDIUM severity patterns generate warnings.

## Continuous Learning

If new anti-patterns are discovered:
1. Document the pattern with evidence
2. Add to knowledge graph
3. Update this detector
4. Notify all agents

Target code for scanning: $ARGUMENTS"

## Response Format

Return structured results:
```yaml
scan_results:
  critical_violations: []
  high_violations: []
  medium_violations: []
  warnings: []
  patterns_checked: 11
  files_scanned: X
  recommendation: BLOCK|FIX|PROCEED
```

Code/directory to analyze: $ARGUMENTS