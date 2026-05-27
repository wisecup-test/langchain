# Adopt Synchronous-First Concurrency Model with Async Compatibility: Components Use Async

These rules are ALWAYS ACTIVE for all components implementing concurrency patterns in the codebase, including chat model interfaces, chain execution logic, memory systems, client utilities, and core framework abstractions exposed to end users.

### Rules

- **R-ASYNC-001** SHOULD: Components SHOULD use async as the underlying implementation with sync wrappers where performance permits.
- **R-ASYNC-002** MUST: Core components (chains, models, memory) expose both synchronous and asynchronous methods for I/O operations.
- **R-ASYNC-003** MUST: Async method names follow consistent naming convention with 'a' prefix (e.g., ainvoke, arun, aexecute).
- **R-ASYNC-004** MUST: Test suites verify functional equivalence between sync and async variants.
- **R-ASYNC-005** MUST: Documentation clearly indicates concurrency model for each public API method.
- **R-ASYNC-006** SHOULD: Use consistent naming: synchronous methods use standard names, async variants prefix with 'a'.
- **R-ASYNC-007** SHOULD: Implement shared business logic in private methods called by both sync and async public interfaces to reduce duplication.
- **R-ASYNC-008** SHOULD: For I/O operations, prefer async-native implementations with sync wrappers using asyncio.run_in_executor() or similar patterns.
- **R-ASYNC-009** SHOULD: Document concurrency behavior in docstrings using standard format: '@async_variant: ainvoke()' or '@sync_variant: invoke()'.

### Verify

```bash
# Count async function definitions
grep -r 'async def' --include='*.py' | wc -l

# Find sync-only methods in core components
grep -r 'def.*invoke\|def.*run\|def.*execute' --include='*.py' libs/langchain/langchain_classic/chains/ libs/partners/ | grep -v 'async def' | head -20

# Analyze sync vs async method counts in a component
python -c "import ast; import sys; tree=ast.parse(open(sys.argv[1]).read()); methods=[n.name for n in ast.walk(tree) if isinstance(n, ast.FunctionDef)]; async_methods=[n.name for n in ast.walk(tree) if isinstance(n, ast.AsyncFunctionDef)]; print(f'Sync: {len(methods)}, Async: {len(async_methods)}')" libs/langchain/langchain_classic/base_memory.py
```

**Accept when:**
- Core components (chains, models, memory) expose both synchronous and asynchronous methods for I/O operations
- Async method names follow consistent naming convention (e.g., 'a' prefix: ainvoke, arun, aexecute)
- Test suites verify functional equivalence between sync and async variants
- Documentation clearly indicates concurrency model for each public API method
- Shared business logic is implemented in private methods to reduce duplication
- Docstrings document concurrency behavior using standard format

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated code review checks MUST flag new public methods without corresponding async/sync variants. CI pipeline MUST run test suites against both sync and async implementations. Architecture review MUST validate concurrency model compliance for new components. Static analysis tools MUST detect blocking calls in async contexts. Sync/async test coverage divergence MUST not exceed 5%.
</enforcement>