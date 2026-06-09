# Enforce Input Validation for External Data Sources in Security-Critical Components: Components Implement Additional

These rules are ALWAYS ACTIVE for all components handling external data sources, API integrations, and user-provided inputs in security-critical contexts, including chat model integrations, embedding model interfaces, prompt selector and template processing components, callback handlers, configuration parsers, and test utilities that simulate external data sources.

### Rules

- **R-VALIDATION-001** MUST: Implement explicit type checking or schema validation before processing responses from external providers (Ollama, OpenAI, and future providers).
- **R-VALIDATION-002** MUST: Validate all user-provided inputs, prompts, configurations, and callbacks at integration boundaries before processing.
- **R-VALIDATION-003** SHOULD: Use Pydantic models or similar schema validation libraries to define expected input structures for each provider integration.
- **R-VALIDATION-004** SHOULD: Implement validation as early as possible in the data flow, ideally in constructor or initialization methods of integration classes.
- **R-VALIDATION-005** SHOULD: Create reusable validation decorators or mixins that can be applied consistently across chat models, embeddings, and other integration types.
- **R-VALIDATION-006** SHOULD: Log validation failures with sufficient context for security monitoring, sanitizing any potentially sensitive data before logging.
- **R-VALIDATION-007** MAY: Components MAY implement additional context-specific validation rules beyond baseline requirements based on their security risk profile.

### Verify

```bash
# Count type checking and validation usage in integration files
grep -r "isinstance\|type(" libs/partners/*/langchain_*/chat_models.py libs/partners/*/langchain_*/embeddings.py | wc -l

# Count validation framework usage
grep -r "validate\|ValidationError\|pydantic" libs/partners/*/langchain_*/*.py | wc -l

# Count validation test cases
python -m pytest tests/ -k "test_invalid_input or test_validation" --collect-only | grep "<Function" | wc -l
```

**Accept when:**
- All integration points with external providers include explicit type checking or schema validation before processing responses
- At least 80% of integration components use declarative validation (Pydantic, dataclasses with validators, or similar)
- Test suites include negative test cases for invalid inputs in all security-critical integration points
- Validation failures are logged with sufficient context for security monitoring
- Reusable validation utilities are shared across integration components

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis scanning for missing validation at integration boundaries is mandatory. Code review must verify validation requirements for all new provider integrations. Security-focused integration tests injecting malformed data must pass in CI pipeline. Violations block merge and trigger security team review.
</enforcement>