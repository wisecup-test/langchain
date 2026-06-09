# Adopt Synchronous-First Concurrency Model with Async Compatibility: Components Implement Sync

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all components implementing concurrency patterns in the codebase.

## Context

- The codebase serves diverse use cases including synchronous scripts, web servers, and async event loops, requiring flexible concurrency support
- Python's dual concurrency model (sync and async) creates architectural tension between simplicity and performance optimization
- Multiple components (chat models, chains, memory systems, and test utilities) exhibit consistent patterns in handling both synchronous and asynchronous execution
- The pattern appears across 4 files with 91.83% confidence, indicating a deliberate architectural choice rather than isolated implementation details
- Framework components must support both blocking and non-blocking I/O patterns to accommodate different deployment environments and user preferences

## Problem Statement

How should the codebase handle concurrency models to balance developer ergonomics, performance requirements, and compatibility across synchronous and asynchronous execution contexts without forcing users into a single paradigm or creating maintenance burden through code duplication?

## Decision

1. MAY: Components MAY implement sync-only interfaces for CPU-bound operations or simple utilities where async provides no benefit

## Policy Block

- MAY Components MAY implement sync-only interfaces for CPU-bound operations or simple utilities where async provides no benefit

In scope:
- Chat model interfaces and LLM integrations
- Chain execution and routing logic
- Memory systems and state management
- Client utilities and API wrappers
- Core framework abstractions exposed to end users

Out of scope:
- Internal utility functions with no I/O operations
- Pure data transformation and validation logic
- Configuration and schema definitions
- Test fixtures and mocks (unless testing concurrency behavior)

Exceptions:
- EXC-001: Legacy components with established sync-only APIs where adding async would break backward compatibility
- EXC-002: Performance-critical paths where dual implementation creates unacceptable overhead

## Rationale

- The pattern detected across 4 files (chat_models.py, router/base.py, base_memory.py, test_client_utils.py) demonstrates consistent adoption of dual concurrency support in critical framework components
- Synchronous-first design reduces cognitive load for simple use cases and scripts while async variants enable high-performance applications
- Python's ecosystem increasingly expects async support for I/O-bound libraries, making this pattern essential for framework competitiveness
- Maintaining functional parity between sync and async prevents fragmentation and ensures users can switch paradigms without rewriting business logic

## Consequences

Positive:
- Developers can choose the concurrency model that fits their use case without framework lock-in
- Simple scripts and notebooks benefit from straightforward synchronous APIs without async complexity
- High-throughput applications can leverage async/await for efficient I/O multiplexing
- Framework remains compatible with both traditional WSGI and modern ASGI deployment environments

Negative:
- Maintaining two implementations increases code surface area and testing requirements
- Risk of behavioral divergence between sync and async paths if not carefully managed
- Developers must understand both paradigms to contribute to core components
- Documentation burden increases to explain when to use each variant

## Alternatives

- Async-only architecture with sync wrappers using asyncio.run() (rejected)
  Rejected because: asyncio.run() creates new event loops which conflicts with existing loops in async contexts, causing runtime errors and poor ergonomics for simple scripts
  When valid: Greenfield projects with no legacy sync code and async-native deployment targets
- Sync-only architecture with no async support (rejected)
  Rejected because: Fails to support high-concurrency use cases and modern async frameworks like FastAPI, limiting framework adoption in performance-critical applications
  When valid: Simple libraries with no I/O operations or legacy codebases with no async requirements
- Separate sync and async packages with no shared code (rejected)
  Rejected because: Creates massive code duplication, doubles maintenance burden, and fragments the user community into incompatible ecosystems
  When valid: Extremely large frameworks where complete separation enables independent evolution

## Risks

- Behavioral divergence between sync and async implementations leading to subtle bugs
  Mitigation: Implement shared test suites that run against both variants, use property-based testing to verify equivalence, establish code review checklist for dual implementations
  Owner: Engineering team
- Performance overhead from maintaining dual code paths and wrapper layers
  Mitigation: Profile critical paths, optimize hot loops, consider async-first with efficient sync wrappers using thread pools for I/O operations
  Owner: Performance engineering team
- Developer confusion about when to use sync vs async variants
  Mitigation: Provide clear decision matrix in documentation, create examples for common scenarios, establish naming conventions (e.g., amethod for async variants)
  Owner: Documentation team

## Implementation Notes

- Use consistent naming: synchronous methods use standard names, async variants prefix with 'a' (e.g., invoke() and ainvoke())
- Implement shared business logic in private methods called by both sync and async public interfaces to reduce duplication
- For I/O operations, prefer async-native implementations with sync wrappers using asyncio.run_in_executor() or similar patterns
- Document concurrency behavior in docstrings using standard format: '@async_variant: ainvoke()' or '@sync_variant: invoke()'

## Continuation Context


Verify commands:
- grep -r 'async def' --include='*.py' | wc -l
- grep -r 'def.*invoke\|def.*run\|def.*execute' --include='*.py' libs/langchain/langchain_classic/chains/ libs/partners/ | grep -v 'async def' | head -20
- python -c "import ast; import sys; tree=ast.parse(open(sys.argv[1]).read()); methods=[n.name for n in ast.walk(tree) if isinstance(n, ast.FunctionDef)]; async_methods=[n.name for n in ast.walk(tree) if isinstance(n, ast.AsyncFunctionDef)]; print(f'Sync: {len(methods)}, Async: {len(async_methods)}')" libs/langchain/langchain_classic/base_memory.py

Accept when:
- Core components (chains, models, memory) expose both synchronous and asynchronous methods for I/O operations
- Async method names follow consistent naming convention (e.g., 'a' prefix: ainvoke, arun, aexecute)
- Test suites verify functional equivalence between sync and async variants
- Documentation clearly indicates concurrency model for each public API method

## Enforcement

- Verified by: Automated code review checks for new public methods without corresponding async/sync variants
- Verified by: CI pipeline runs test suites against both sync and async implementations
- Verified by: Architecture review for new components validates concurrency model compliance
- Verified by: Static analysis tools detect blocking calls in async contexts
- Violation handling: PR comments flag missing async variants for I/O-bound operations
- Violation handling: CI fails if sync/async test coverage diverges by more than 5%
- Violation handling: Architecture review board evaluates exception requests for sync-only or async-only components
- Violation handling: Quarterly audits identify and remediate concurrency model inconsistencies
- Exception process: Submit exception request with rationale (performance, legacy compatibility, or scope justification)
- Exception process: Provide benchmarking data for performance exceptions or migration plan for legacy exceptions
- Exception process: Architecture review board evaluates within 1 sprint
- Exception process: Approved exceptions documented in component README and tracked in architecture decision log