# Enforce Input Validation for External Data Sources in Security-Critical Components: Validation Logic Centralized

These rules are ALWAYS ACTIVE for all chat model integrations, embedding model interfaces, prompt selector and template processing components, callback handlers, configuration parsers, and test utilities that handle external data sources, API integrations, and user-provided inputs in security-critical contexts.

### Rules

- **R-VAL-001** SHOULD: Validation logic SHOULD be centralized in reusable validation functions or decorators to ensure consistency across the codebase.

### Verify

```bash
# Count type checking and validation patterns in integration code
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
Claude Code MUST NOT skip or defer verification. Validation at integration boundaries is mandatory for security-critical components. Static analysis, code review, and integration tests must confirm compliance before merge.
</enforcement>