# RESMARK Commands Integration Strategy

## Decision: Hybrid Composition-Based Approach

After consulting with Gemini, we're adopting a **composition over inheritance** strategy to avoid maintenance hell while ensuring RESMARK patterns are enforced.

## Architecture

```
.claude/commands/
├── resmark-core/              # RESMARK-specific commands (existing 5)
│   ├── anti-pattern-detector.md
│   ├── explain-knowledge.md
│   ├── mdd-cycle.md
│   ├── mdd-orchestrator.md
│   └── resmark-architect.md
│
├── resmark-enhanced/          # New RESMARK-specific commands
│   ├── resmark-validate-output.md    # Critical validation layer
│   ├── resmark-component-migrate.md  # Composite workflows
│   ├── resmark-api-modernize.md
│   └── resmark-test-suite.md
│
├── tools/                     # Original generic tools (unchanged)
├── workflows/                 # Original workflows (unchanged)
├── agents/                    # Original agents (unchanged)
│
└── resmark-engine/           # The orchestration layer
    ├── RESMARK_CONSTRAINTS.md       # All critical directives
    ├── resmark-wrapper.md            # Dynamic injection system
    └── validation-hooks.md           # Anti-pattern checks

```

## Implementation Plan

### Phase 1: Build the RESMARK Engine (Priority 1)

1. **Create `resmark-validate-output.md`**
   - Non-AI validation command
   - Checks for all 12 anti-patterns
   - Enforces critical directives
   - Triggers visual regression tests
   - MUST be final step in every workflow

2. **Create `RESMARK_CONSTRAINTS.md`**
   - Consolidated list of all critical directives
   - Anti-pattern definitions with examples
   - Performance targets
   - Session management patterns

3. **Create `resmark-wrapper.md`**
   - Dynamically injects RESMARK constraints into any command
   - Routes requests to appropriate tools
   - Enforces validation after code generation

### Phase 2: Create RESMARK-Enhanced Commands

Only create RESMARK-specific versions when absolutely necessary:

1. **`resmark-component-migrate.md`**
   - Orchestrates: mdd-cycle + code-migrate + visual validation
   - Hard-coded Angular→React patterns
   - Mandatory visual regression

2. **`resmark-api-modernize.md`**
   - Wraps: api-scaffold with SessionService patterns
   - Enforces Redis caching
   - Prevents in-memory Maps

3. **`resmark-test-suite.md`**
   - Extends: test-generator with visual requirements
   - 80% coverage minimum
   - Session pattern testing

### Phase 3: Update Core RESMARK Commands

Modify existing RESMARK commands to use the wrapper:

```yaml
mdd-cycle.md:
  discovery:
    - run: resmark-wrapper tools/code-explain
    - run: resmark-wrapper tools/dependency-analyzer
  implementation:
    - run: resmark-wrapper tools/api-scaffold
    - run: resmark-validate-output  # MANDATORY
```

## Key Principles

### 1. Composition Over Inheritance
- Don't fork entire repositories
- Wrap and extend instead of modifying
- Keep generic tools generic

### 2. Validation as Law
- Every code generation MUST pass `resmark-validate-output`
- Validation is non-negotiable, non-AI, deterministic
- Failures block progression completely

### 3. Dynamic Injection
- RESMARK constraints injected at runtime
- No modification of original tool files
- Easy to update constraints centrally

### 4. Selective Override
- Only create RESMARK versions when generic fails
- Document why override was necessary
- Maintain minimal custom surface area

## Benefits

1. **Low Maintenance**: Original tools update via simple git pull
2. **Clear Separation**: RESMARK logic isolated in engine
3. **Guaranteed Compliance**: Validation layer ensures patterns
4. **Sustainable**: Scales without exponential complexity
5. **Contribution-Ready**: Can share improvements upstream

## Anti-Patterns to Avoid

1. **Don't Fork Everything**: Massive maintenance overhead
2. **Don't Trust AI Alone**: Always validate with deterministic checks
3. **Don't Modify Originals**: Keep them pristine for updates
4. **Don't Skip Validation**: Even for "simple" changes

## Success Metrics

- Zero anti-patterns in generated code
- 100% validation pass rate before human review
- <5 custom tool overrides needed
- Single-command updates for constraint changes

## Next Steps

1. Build `resmark-validate-output.md` immediately
2. Create `RESMARK_CONSTRAINTS.md` with all directives
3. Test with single component migration
4. Iterate based on results

---

*This strategy balances control with maintainability, ensuring RESMARK patterns are enforced without creating an unmaintainable fork.*