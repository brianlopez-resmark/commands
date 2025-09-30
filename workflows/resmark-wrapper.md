# RESMARK Wrapper - Dynamic Constraint Injection

Orchestrator that wraps ANY command with RESMARK constraints and validation.

## Purpose

Dynamically inject RESMARK constraints into generic commands at runtime, ensuring all generated code adheres to empirically-validated patterns without modifying original tool files.

## Usage

Wrap any command with RESMARK constraints: $ARGUMENTS

## Orchestration Process

### Step 1: Load RESMARK Constraints

```yaml
constraints:
  source: RESMARK_CONSTRAINTS.md
  injection_point: system_prompt
  validation: resmark-validate-output
```

### Step 2: Parse Command Request

Determine which tool/command is being requested:

```javascript
// Parse the command structure
const parseCommand = (request) => {
  // Examples:
  // "resmark-wrapper tools/code-migrate SearchComponent"
  // "resmark-wrapper tools/api-scaffold user-endpoint"

  const [wrapper, toolPath, ...args] = request.split(' ');
  return {
    tool: toolPath,
    arguments: args.join(' '),
    isCodeGeneration: isCodeGenerationTool(toolPath)
  };
};
```

### Step 3: Inject Constraints into Tool

Modify the tool's execution context:

```yaml
enhanced_prompt: |
  # CRITICAL: RESMARK CONSTRAINTS ACTIVE

  You MUST adhere to ALL RESMARK Constraints. These are non-negotiable.

  ## Critical Directives (Automatic BLOCK on violation):
  1. NO mock data in service layers - Use real database connections
  2. ONLY use req.session.entityId for session management
  3. NO in-memory Maps for caching - Use Redis exclusively
  4. NO hardcoded test bypasses - Use feature flags
  5. Visual regression tests REQUIRED for all UI components

  ## Session Management (MANDATORY):
  - Backend: Use SessionService.reconstructSession()
  - Frontend: Use useSession() hook
  - NEVER use req.user.* patterns (they don't exist)

  ## Original Tool Instructions:
  ${original_tool_content}

  ## Post-Generation Validation:
  After generating code, you MUST:
  1. Self-check against RESMARK constraints
  2. Flag any potential violations
  3. Note that resmark-validate-output will run automatically
```

### Step 4: Execute Enhanced Tool

Run the tool with injected constraints:

```yaml
execution:
  - Load original tool/command
  - Prepend RESMARK constraints
  - Execute with enhanced context
  - Capture output
```

### Step 5: Automatic Validation

For code-generation tools, automatically validate output:

```bash
# If code was generated, validate it
if [ "$IS_CODE_GENERATION" = true ]; then
  echo "Running RESMARK validation..."

  # Execute validation
  validation_result=$(resmark-validate-output $generated_files)

  # Check for critical violations
  if echo "$validation_result" | grep -q "CRITICAL"; then
    echo "❌ CRITICAL VIOLATION DETECTED"
    echo "$validation_result"
    echo "Code generation BLOCKED. Fix violations and retry."
    exit 1
  fi

  # Check for high severity issues
  high_count=$(echo "$validation_result" | grep -c "HIGH")
  if [ "$high_count" -ge 3 ]; then
    echo "⚠️ Multiple HIGH severity issues detected"
    echo "$validation_result"
    echo "Fix issues before proceeding."
    exit 1
  fi
fi
```

### Step 6: Return Enhanced Results

Provide results with validation report:

```yaml
output:
  generated_code: ${tool_output}
  validation_status: ${validation_result}
  recommendations:
    - Fix any violations before proceeding
    - Run visual regression tests if UI component
    - Ensure feature flags are configured
```

## Tool Classification

Identify which tools generate code (require validation):

```yaml
code_generation_tools:
  - tools/code-migrate
  - tools/api-scaffold
  - tools/test-generator
  - tools/component-scaffold
  - tools/database-migrate
  - workflows/feature-development
  - workflows/full-stack-feature

analysis_tools:  # No validation needed
  - tools/code-explain
  - tools/dependency-analyzer
  - tools/performance-audit
  - tools/security-audit
```

## Integration Examples

### Example 1: Wrapping API Scaffold

```bash
# Original command
resmark-wrapper tools/api-scaffold create user-management endpoint

# What happens:
1. Loads tools/api-scaffold.md
2. Injects RESMARK session patterns
3. Ensures SessionService usage
4. Validates no mock data
5. Returns validated code
```

### Example 2: Wrapping Component Migration

```bash
# Original command
resmark-wrapper tools/code-migrate Angular SearchComponent to React

# What happens:
1. Loads tools/code-migrate.md
2. Injects req.session.entityId pattern
3. Requires visual regression test
4. Validates against anti-patterns
5. Ensures feature flag wrapper
```

### Example 3: Wrapping Test Generation

```bash
# Original command
resmark-wrapper tools/test-generator for UserService

# What happens:
1. Loads tools/test-generator.md
2. Requires 80% coverage minimum
3. Adds session mocking patterns
4. Includes visual tests for UI
5. Validates test quality
```

## Constraint Override (Exceptional Cases Only)

In rare cases where constraints must be bypassed:

```yaml
override:
  command: resmark-wrapper --override=pattern-name tool/command
  requires:
    - Explicit justification
    - Document in PR
    - Architecture review
    - Update knowledge graph if pattern invalid
```

## Monitoring & Learning

Track wrapper usage for continuous improvement:

```yaml
metrics:
  - Tool usage frequency
  - Validation failure rates
  - Most common violations
  - Pattern effectiveness

feedback_loop:
  - New anti-patterns discovered → Update RESMARK_CONSTRAINTS.md
  - Tool improvements needed → Enhance wrapper logic
  - Success patterns → Document in knowledge graph
```

## Error Handling

Handle various failure modes:

```yaml
errors:
  tool_not_found:
    message: "Tool ${tool} not found in commands directory"
    action: List available tools

  validation_failure:
    message: "Generated code violates RESMARK constraints"
    action: Block progression, show violations

  constraint_conflict:
    message: "Tool requirements conflict with RESMARK patterns"
    action: Escalate to resmark-architect for resolution
```

## Benefits

1. **Zero modification** of original tools
2. **Centralized constraint management**
3. **Automatic validation** for all code
4. **Easy updates** via RESMARK_CONSTRAINTS.md
5. **Learning loop** for pattern discovery

## Integration with MDD Workflow

The wrapper is automatically used by:
- `mdd-cycle.md` for all tool invocations
- `mdd-orchestrator.md` for agent coordination
- `resmark-architect.md` for validation checks

## Command to wrap: $ARGUMENTS