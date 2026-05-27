# Adopt Synchronous-First Concurrency Model with Async Compatibility: Core Components Provide

These rules are ALWAYS ACTIVE for all components implementing concurrency patterns in the codebase, including chat model interfaces, chain execution logic, memory systems, client utilities, and core framework abstractions exposed to end users.

### Rules

- **R-SYNC-001** MUST: All core components MUST provide synchronous interfaces as the primary API surface.
- **R-SYNC-002** MUST: Async method names MUST follow consistent naming convention with 'a' prefix (e.g., ainvoke, arun, aexecute).
- **R-SYNC-003** MUST: Implement shared business logic in private methods called by both sync and async public interfaces to reduce duplication.
- **R-SYNC-004** MUST: Document concurrency behavior in docstrings using standard format: '@async_variant: ainvoke()' or '@sync_variant: invoke()'.
- **R-SYNC-005** SHOULD: For I/O operations, prefer async-native implementations with sync wrappers using asyncio.run_in_executor() or similar patterns.
- **R-SYNC-006** SHOULD: Implement shared test suites that run against both sync and async variants to verify functional equivalence.
- **R-SYNC-007** MAY: Request exceptions for legacy components with established sync-only APIs (EXC-001) or performance-critical paths (EXC-002) with documented rationale.

### Verify

```bash
# Count async function definitions in codebase
grep -r 'async def' --include='*.py' | wc -l

# List sync methods in core components
grep -r 'def.*invoke\|def.*run\|def.*execute' --include='*.py' libs/langchain/langchain_classic/chains/ libs/partners/ | grep -v 'async def' | head -20

# Analyze sync vs async method counts in a component
python -c "import ast; import sys; tree=ast.parse(open(sys.argv[1]).read()); methods=[n.name for n in ast.walk(tree) if isinstance(n, ast.FunctionDef)]; async_methods=[n.name for n in ast.walk(tree) if isinstance(n, ast.AsyncFunctionDef)]; print(f'Sync: {len(methods)}, Async: {len(async_methods)}')" libs/langchain/langchain_classic/base_memory.py
```

**Accept when:**
- Core components (chains, models, memory) expose both synchronous and asynchronous methods for I/O operations
- Async method names follow consistent naming convention (e.g., 'a' prefix: ainvoke, arun, aexecute)
- Test suites verify functional equivalence between sync and async variants
- Documentation clearly indicates concurrency model for each public API method
- Shared business logic is extracted to private methods to minimize duplication
- Docstrings document both sync and async variants with standard format

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated code review checks, CI pipeline tests, and architecture reviews are mandatory for all new public methods in core components. Violations must be flagged in PR comments, and exceptions require explicit approval from the architecture review board with documented rationale.
</enforcement>