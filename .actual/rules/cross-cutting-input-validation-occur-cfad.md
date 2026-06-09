# Enforce Input Validation for External Data Sources in Security-Critical Components: Input Validation Occur

These rules are ALWAYS ACTIVE for all chat model integrations, embedding model interfaces, prompt selector and template processing components, callback handlers, configuration parsers, and test utilities that handle external data sources, API integrations, and user-provided inputs in security-critical contexts.

### Rules

- **R-VALIDATION-001** MUST: Input validation MUST occur at the earliest possible point in the data flow, preferably at API boundaries and integration points with external providers (Ollama, OpenAI, and future providers).

### Verify

```bash
# Count type checking and validation patterns in integration files
grep -r "isinstance\|type(" libs/partners/*/langchain_*/chat_models.py libs/partners/*/langchain_*/embeddings.py 2>/dev/null | wc -l

# Count validation and schema validation usage
grep -r "validate\|ValidationError\|pydantic" libs/partners/*/langchain_*/*.py 2>/dev/null | wc -l

# Count validation-related test cases
python -m pytest tests/ -k "test_invalid_input or test_validation" --collect-only 2>/dev/null | grep "<Function" | wc -l
```

**Accept when:**
- All integration points with external providers include explicit type checking or schema validation before processing responses
- At least 80% of integration components use declarative validation (Pydantic, dataclasses with validators, or similar)
- Test suites include negative test cases for invalid inputs in all security-critical integration points

<enforcement>
Claude Code MUST NOT skip or defer verification of input validation at integration boundaries. Static analysis, code review, and security-focused integration tests are mandatory before accepting changes to provider integration code.
</enforcement>