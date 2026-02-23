# Common Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Vague goals | Loop never completes | Add checkable criteria |
| Missing verification | Can't detect success | Add explicit test commands |
| Giant steps | Overwhelms single iteration | Break into atomic actions |
| No self-correction | Gets stuck on errors | Add fix-and-retry protocol |
| Implicit completion | Unclear when done | Use `<promise>` tags |

## When NOT Suitable for Ralph

Redirect user if task requires:
- Subjective human judgment
- Interactive user decisions mid-task
- Unclear success criteria that can't be defined
- Production debugging with real data
- Tasks needing external approvals

## Alternatives

- Manual implementation with superpowers:test-driven-development
- Interactive session with superpowers:brainstorming first
