# Task 03 — Netfree Policy Loader

**Priority:** P1  
**Estimate:** 1 h  
**Assignee:** sw_developer  
**Depends on:** Task 01  
**Plan refs:** PLAN.md §5 (policy YAML), §8.1 (analyzer uses policy), backlog #4

## Description
Implement `src/services/policy.py` to load `config/netfree_policy.yaml`, validate it, and produce a formatted prompt string for injection into the Gemini request.

## Acceptance Criteria
- `load_policy(path)` returns a structured policy object.
- `format_prompt(policy)` returns a string listing each active category with its description.
- Categories can be toggled via the `severity` field or a future `enabled` flag.
- Raises a clear error if the YAML is missing or malformed.

## Policy YAML
See PLAN.md §5 for the full initial `config/netfree_policy.yaml` content. Copy it verbatim.

**Verdict schema expected from Gemini:**
```json
{"approved": bool, "violated_categories": [...], "reasoning": "...", "confidence": 0.0–1.0}
```

## 3rd-Party Tools
| Package | Purpose |
|---|---|
| `PyYAML` | Parse policy YAML |
| `pydantic` | Validate policy structure |

## Notes
- Policy file path configurable via `POLICY_PATH` env var (default `config/netfree_policy.yaml`).
- Keep policy loading separate from analyzer so it can be unit-tested independently.
