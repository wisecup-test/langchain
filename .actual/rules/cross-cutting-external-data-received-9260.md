# Enforce Input Validation for External Data Sources in Security-Critical Components: External Data Received

These rules are ALWAYS ACTIVE for all components handling external data sources, API integrations, and user-provided inputs in security-critical contexts, including chat model integrations, embedding model interfaces, prompt selectors, callback handlers, configuration parsers, and test utilities that simulate external data sources.

### Rules

- **R-EXT-001** MUST: All external data received from AI model providers (Ollama, OpenAI, etc.) MUST be validated for type correctness and schema compliance before processing.
- **R-EXT-002** MUST: Implement validation as early as possible in the data flow, ideally in the constructor or initialization methods of integration classes.
- **R-EXT-003** SHOULD: Use Pydantic models or similar schema validation libraries to define expected input structures for each provider integration.
- **R-EXT-004** SHOULD: Create reusable validation decorators or mixins that can be applied consistently across chat models, embeddings, and other integration types.
- **R-EXT-005** SHOULD: Log validation failures with sufficient context for security monitoring, but sanitize any potentially sensitive data before logging.
- **R-EXT-006** SHOULD: Include validation test cases in integration test suites, covering both valid inputs and common attack patterns (oversized inputs, type mismatches, injection attempts).

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
Claude Code MUST NOT skip or defer verification of these rules. Validation at external data integration boundaries is a mandatory security control.
</enforcement>