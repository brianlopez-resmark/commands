# RESMARK Validate Output

Deterministic validation of generated code against RESMARK critical directives and anti-patterns.

**CRITICAL**: This is NOT an AI tool. This is a deterministic validation script that MUST be the final step in every code generation workflow.

## Usage

Validate generated code: $ARGUMENTS (file path or directory)

## Validation Process

Execute these checks in order. Fail fast on any violation.

### Stage 1: Anti-Pattern Detection (CRITICAL)

Check for all 12 empirically-discovered anti-patterns:

```bash
# 1. Direct Database Mock (CRITICAL)
grep -r "return \[{.*id.*:.*1.*,.*name.*:.*Mock" $ARGUMENTS
grep -r "return \[{.*id.*:.*'test'" $ARGUMENTS
grep -r "mockData\s*=\s*\[" $ARGUMENTS
# FAIL if found: "CRITICAL: Mock data in service layer detected"

# 2. Wrong Session Pattern (CRITICAL)
grep -r "req\.user\.businessEntities" $ARGUMENTS
grep -r "req\.session\.permissions" $ARGUMENTS
grep -r "req\.currentEntity" $ARGUMENTS
grep -r "req\.user\.entityId" $ARGUMENTS
# FAIL if found: "CRITICAL: Invalid session pattern - MUST use req.session.entityId"

# 3. In-Memory Map Caching (CRITICAL)
grep -r "new Map()" $ARGUMENTS
grep -r "const.*cache.*=.*{}" $ARGUMENTS
grep -r "rateLimitMap\s*=\s*new Map" $ARGUMENTS
# FAIL if found: "CRITICAL: In-memory caching detected - MUST use Redis"

# 4. Hardcoded Test Bypasses (CRITICAL)
grep -r "if.*entityId.*===.*'test'" $ARGUMENTS
grep -r "entityId.*==.*'test'" $ARGUMENTS
grep -r "// TODO.*remove.*production" $ARGUMENTS
# FAIL if found: "CRITICAL: Hardcoded test bypass detected"

# 5. TypeScript Complexity Trap (HIGH)
# Check for overly complex type definitions in early phase files
grep -r "type.*=.*{.*{.*{.*{" $ARGUMENTS
grep -r "interface.*extends.*,.*extends" $ARGUMENTS
# WARN if found: "HIGH: Complex TypeScript detected in discovery phase"

# 6. Missing Visual Validation (HIGH)
# For React components, check for visual test files
if grep -q "export.*function.*component\|export.*const.*=.*React" $ARGUMENTS; then
  component_name=$(basename $ARGUMENTS .tsx)
  if ! test -f "tests/visual/${component_name}.spec.ts"; then
    echo "HIGH: Missing visual regression test for component"
  fi
fi

# 7. Direct Git Contamination (HIGH)
# Check if code is being written directly to main branches
current_branch=$(git branch --show-current)
if [[ "$current_branch" == "main" || "$current_branch" == "develop" ]]; then
  echo "HIGH: Direct writes to protected branch detected"
fi

# 8. Unvalidated Environment Access (HIGH)
grep -r "process\.env\.[A-Z_]*\s*\|\|" $ARGUMENTS
grep -r "process\.env\.[A-Z_]*\s*\?\?" $ARGUMENTS
# WARN if found: "HIGH: Environment variables with fallback values"

# 9. Incomplete Implementation (MEDIUM)
grep -r "// TODO" $ARGUMENTS
grep -r "throw new Error.*Not implemented" $ARGUMENTS
grep -r "// FIXME" $ARGUMENTS
# WARN if found: "MEDIUM: Incomplete implementation markers"

# 10. Synchronous/Asynchronous Mixing (MEDIUM)
# Check for missing await keywords (heuristic)
grep -r "\.then.*async" $ARGUMENTS
grep -r "async.*{.*[^await].*\.(find\|save\|create\|update\|delete)(" $ARGUMENTS
# WARN if found: "MEDIUM: Potential async/sync mixing"

# 11. Missing Error Boundaries (MEDIUM)
# For React components
if grep -q "export.*function.*component\|export.*const.*=.*React" $ARGUMENTS; then
  if ! grep -q "ErrorBoundary\|componentDidCatch\|error.*boundary" $ARGUMENTS; then
    echo "MEDIUM: React component missing error boundary"
  fi
fi

# 12. Security Vulnerabilities (CRITICAL)
grep -r "eval(" $ARGUMENTS
grep -r "innerHTML\s*=" $ARGUMENTS
grep -r "dangerouslySetInnerHTML" $ARGUMENTS  # Except with proper sanitization
# FAIL if found: "CRITICAL: Potential security vulnerability"
```

### Stage 2: Required Pattern Enforcement

Ensure critical patterns ARE present:

```bash
# Check for proper session handling
if grep -q "router\.\|app\.\|api\." $ARGUMENTS; then
  if ! grep -q "req\.session\.entityId" $ARGUMENTS; then
    echo "CRITICAL: API endpoint missing req.session.entityId check"
    exit 1
  fi
fi

# Check for Redis caching if any caching is implemented
if grep -q "cache\|Cache" $ARGUMENTS; then
  if ! grep -q "redis\|Redis" $ARGUMENTS; then
    echo "CRITICAL: Caching implementation must use Redis"
    exit 1
  fi
fi

# Check for feature flags in new components
if grep -q "export.*function.*component\|export.*const.*=.*React" $ARGUMENTS; then
  if ! grep -q "featureFlags\|feature-flag" $ARGUMENTS; then
    echo "WARNING: Component should be wrapped in feature flag"
  fi
fi

# Check for SessionService usage in services
if [[ $ARGUMENTS == *"service"* ]]; then
  if ! grep -q "SessionService\|sessionService" $ARGUMENTS; then
    echo "HIGH: Service should use SessionService for auth"
  fi
fi
```

### Stage 3: Performance Validation

```bash
# Check for performance anti-patterns
grep -r "SELECT \* FROM" $ARGUMENTS  # Should specify columns
grep -r "forEach.*await" $ARGUMENTS  # Should use Promise.all
grep -r "JSON\.parse.*JSON\.stringify" $ARGUMENTS  # Deep clone anti-pattern
```

### Stage 4: Test Coverage Check

```bash
# For any new code file, check for corresponding test
for file in $(find $ARGUMENTS -name "*.ts" -o -name "*.tsx" | grep -v ".test."); do
  test_file="${file%.*}.test.${file##*.}"
  if [ ! -f "$test_file" ]; then
    echo "WARNING: Missing test file for $file"
  fi
done
```

### Stage 5: Visual Regression Trigger

For React components, automatically trigger visual regression:

```bash
if grep -q "export.*function.*component\|export.*const.*=.*React" $ARGUMENTS; then
  component_file=$ARGUMENTS
  component_name=$(basename $component_file .tsx)

  # Check if baseline exists
  baseline="tests/visual/baselines/angular/${component_name}.png"
  if [ -f "$baseline" ]; then
    echo "Triggering visual regression test for $component_name"
    npm run test:visual -- --component=$component_name
  else
    echo "WARNING: No Angular baseline for visual regression"
  fi
fi
```

## Output Format

Return structured validation report:

```yaml
validation_result:
  status: PASS | FAIL | WARNING

  critical_violations:
    - type: "direct-database-mock"
      location: "services/user.service.ts:45"
      message: "Mock data in service layer"

  high_severity:
    - type: "missing-visual-test"
      component: "SearchInput"
      message: "No visual regression test"

  medium_severity:
    - type: "incomplete-implementation"
      location: "api/routes.ts:23"
      message: "TODO marker found"

  warnings:
    - type: "performance"
      location: "services/data.ts:67"
      message: "SELECT * should specify columns"

  required_patterns:
    session_handling: ✓
    redis_caching: ✓
    feature_flags: ✗

  metrics:
    files_scanned: 12
    anti_patterns_checked: 12
    patterns_validated: 4

  recommendation:
    BLOCK: "Critical violations must be fixed"
    | FIX: "High severity issues should be addressed"
    | PROCEED: "Code meets RESMARK standards"
```

## Integration Requirements

This command MUST be called:
1. After every code generation
2. Before any PR creation
3. As part of CI/CD pipeline
4. Before human review

## Non-Negotiable Rules

1. **ANY critical violation = BLOCK progression**
2. **3+ high severity = BLOCK progression**
3. **Visual regression required for ALL UI components**
4. **No bypassing or skipping validation**
5. **Results must be attached to PRs**

## Continuous Improvement

When new anti-patterns are discovered:
1. Add detection rule to this validator
2. Update knowledge graph
3. Notify all RESMARK commands
4. Re-validate recent code

Target to validate: $ARGUMENTS