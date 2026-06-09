# Adopt Synchronous-First Concurrency Model with Async Compatibility: Synchronous Asynchronous Implementations

These rules are ALWAYS ACTIVE for all components implementing concurrency patterns in the codebase, including chat model interfaces, chain execution logic, memory systems, client utilities, and core framework abstractions exposed to end users.

### Rules

- **R-SYNC-001** MUST: Synchronous and asynchronous implementations MUST maintain functional parity and consistent behavior.
- **R-SYNC-002** MUST: Use consistent naming conventions where synchronous methods use standard names and async variants prefix with 'a' (e.g., invoke() and ainvoke()).
- **R-SYNC-003** MUST: Implement shared business logic in private methods called by both sync and async public interfaces to reduce duplication.
- **R-SYNC-004** SHOULD: For I/O operations, prefer async-native implementations with sync wrappers using asyncio.run_in_executor() or similar patterns.
- **R-SYNC-005** SHOULD: Document concurrency behavior in docstrings using standard format: '@async_variant: ainvoke()' or '@sync_variant: invoke()'.
- **R-SYNC-006** MUST: Test suites MUST verify functional equivalence between sync and async variants.
- **R-SYNC-007** MUST: Maintain sync/async test coverage parity within 5% variance.

### Verify

```bash
# Count async function definitions
grep -r 'async def' --include='*.py' | wc -l

# List sync methods in core components
grep -r 'def.*invoke\|def.*run\|def.*execute' --include='*.py' libs/langchain/langchain_classic/chains/ libs/partners/ | grep -v 'async def' | head -20

# Analyze sync vs async method counts in a file
python -c "import ast; import sys; tree=ast.parse(open(sys.argv[1]).read()); methods=[n.name for n in ast.walk(tree) if isinstance(n, ast.FunctionDef)]; async_methods=[n.name for n in ast.walk(tree) if isinstance(n, ast.AsyncFunctionDef)]; print(f'Sync: {len(methods)}, Async: {len(async_methods)}')" libs/langchain/langchain_classic/base_memory.py
```

**Accept when:**
- Core components (chains, models, memory) expose both synchronous and asynchronous methods for I/O operations
- Async method names follow consistent naming convention (e.g., 'a' prefix: ainvoke, arun, aexecute)
- Test suites verify functional equivalence between sync and async variants
- Documentation clearly indicates concurrency model for each public API method
- Blocking calls do not appear in async contexts
- Sync/async test coverage divergence does not exceed 5%

<enforcement>
Verification by automated code review checks for new public methods without corresponding async/sync variants. CI pipeline runs test suites against both sync and async implementations. Architecture review validates concurrency model compliance. Static analysis detects blocking calls in async contexts. PR comments flag missing async variants for I/O-bound operations. CI fails if sync/async test coverage diverges by more than 5%. Architecture review board evaluates exception requests. Quarterly audits identify and remediate inconsistencies. Claude Code MUST NOT skip or defer verification.
</enforcement>