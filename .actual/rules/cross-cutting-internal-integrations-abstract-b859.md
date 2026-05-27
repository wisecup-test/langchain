# Standardize External Client Abstraction for Internal API Integration: Internal Integrations Abstract

These rules are ALWAYS ACTIVE for all internal API integrations that interact with external service providers, including partner library integrations (Ollama, OpenAI, Nomic, etc.), chat models, LLMs, embedding implementations, and client utility modules.

### Rules

- **R-INTABS-001** MUST: Internal API integrations MUST abstract external client dependencies through compatibility layers or wrapper utilities.

### Verify

```bash
# Check for direct client imports outside designated compatibility modules
grep -r "import.*client" libs/partners/*/langchain_*/[!tests]*.py | grep -v "_compat\|_utils\|client_utils" || echo 'No direct client imports found'

# Count dedicated compatibility/utility modules
find libs/partners -name '_compat.py' -o -name '_utils.py' -o -name 'client_utils.py' | wc -l

# Count dependency injection patterns in __init__ methods
grep -r "def __init__" libs/partners/*/langchain_*/*.py | grep -c "client.*=" || echo '0'
```

**Accept when:**
- All partner library modules use compatibility or utility layers for external client interactions
- Unit tests can execute without network access by mocking external clients through abstraction interfaces
- At least 80% of partner libraries have dedicated _compat.py, _utils.py, or client_utils.py modules

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review checklist MUST require abstraction layer usage for new partner integrations. CI pipeline MUST check for direct external client imports outside designated compatibility modules. Pull requests introducing direct external client usage without abstraction MUST be flagged in code review.
</enforcement>