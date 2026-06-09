# Enforce Input Validation for External Data Sources in Security-Critical Components: User Provided Inputs

These rules are ALWAYS ACTIVE for all chat model integrations, embedding model interfaces, prompt selector and template processing components, callback handlers, configuration parsers, and test utilities that handle external data sources, API integrations, and user-provided inputs in security-critical contexts.

### Rules

- **R-INPUT-001** MUST: User-provided inputs including prompts, configuration parameters, and callback handlers MUST be sanitized and validated against expected formats before processing.

### Verify

```bash
# Count type checking and validation usage in integration points
grep -r "isinstance\|type(" libs/partners/*/langchain_*/chat_models.py libs/partners/*/langchain_*/embeddings.py | wc -l

# Count validation and schema validation library usage
grep -r "validate\|ValidationError\|pydantic" libs/partners/*/langchain_*/*.py | wc -l

# Count validation-related test cases
python -m pytest tests/ -k "test_invalid_input or test_validation" --collect-only | grep "<Function" | wc -l
```

**Accept when:**
- All integration points with external providers include explicit type checking or schema validation before processing responses
- At least 80% of integration components use declarative validation (Pydantic, dataclasses with validators, or similar)
- Test suites include negative test cases for invalid inputs in all security-critical integration points

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis scanning, code review checklists, security-focused integration tests, and quarterly security audits are mandatory. CI pipeline MUST fail if unvalidated external data usage is detected in security-critical paths.
</enforcement>