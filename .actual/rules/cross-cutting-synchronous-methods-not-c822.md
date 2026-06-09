# Adopt Synchronous-First Concurrency Model with Async Compatibility: Synchronous Methods Not

These rules are ALWAYS ACTIVE for all components implementing concurrency patterns in the codebase, including chat model interfaces, chain execution logic, memory systems, client utilities, and core framework abstractions exposed to end users.

### Rules

- **R-SYNC-001** MUST NOT: Synchronous methods MUST NOT block the event loop when called from async contexts without explicit documentation.

### Verify

```bash
# Count async method definitions
grep -r 'async def' --include='*.py' | wc -l

# Find synchronous invoke/run/execute methods
grep -r 'def.*invoke\|def.*run\|def.*execute' --include='*.py' libs/langchain/langchain_classic/chains/ libs/partners/ | grep -v 'async def' | head -20

# Analyze sync vs async method counts in a file
python -c "import ast; import sys; tree=ast.parse(open(sys.argv[1]).read()); methods=[n.name for n in ast.walk(tree) if isinstance(n, ast.FunctionDef)]; async_methods=[n.name for n in ast.walk(tree) if isinstance(n, ast.AsyncFunctionDef)]; print(f'Sync: {len(methods)}, Async: {len(async_methods)}')" libs/langchain/langchain_classic/base_memory.py
```

**Accept when:**
- Core components (chains, models, memory) expose both synchronous and asynchronous methods for I/O operations
- Async method names follow consistent naming convention (e.g., 'a' prefix: ainvoke, arun, aexecute)
- Test suites verify functional equivalence between sync and async variants
- Documentation clearly indicates concurrency model for each public API method
- Synchronous methods do not call blocking I/O directly when invoked from async contexts without explicit documentation warning of event loop blocking

<enforcement>
Claude Code MUST NOT skip or defer verification. All public methods in scope must be checked for event loop blocking behavior. Violations must be flagged in code review with specific guidance on async wrapper implementation or documentation requirements.
</enforcement>