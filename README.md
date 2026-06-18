# chatgpt-outsource

Use ChatGPT as a bounded external advisor for safe, self-contained tasks such as rewriting, translation, summarization, research, and debugging advice, so coding or work agents can save tokens for local reasoning, execution, and verification while keeping local validation as the final authority.

## Files

- `SKILL.md`: the skill definition and operating rules
- `agents/openai.yaml`: the OpenAI agent interface metadata

## Scope

- Good fit for low-context, non-sensitive tasks that can be expressed in a compact prompt
- Useful when you want coding or work agents to preserve context and token budget for repository-specific reasoning and execution
- Designed for advisory use, not direct local file editing or final factual authority
- Emphasizes prompt sanitization, response validation, and preference for local evidence on conflict

## Sequence Diagram

```mermaid
sequenceDiagram
    actor User
    participant Agent as "Coding or Work Agent"
    participant Skill as "chatgpt-outsource"
    participant ChatGPT as "ChatGPT via Chrome Extension"
    participant Local as "Local Evidence or Validation"

    User->>Agent: Request help on a self-contained task
    Agent->>Skill: Check task fit for outsourcing
    Skill-->>Agent: Approve only if low-context and non-sensitive
    Agent->>Skill: Build compact sanitized prompt
    Skill->>ChatGPT: Send advisory prompt
    ChatGPT-->>Skill: Return rewrite, summary, research, or advice
    Skill-->>Agent: Deliver advisory response
    Agent->>Local: Verify against code, tests, runtime, or sources
    Local-->>Agent: Confirm or contradict response
    alt Verified locally
        Agent-->>User: Use validated result
    else Not verified or conflicts with local evidence
        Agent-->>User: Reject or revise using local evidence
    end
```

## Notes

- This repo was initialized from the installed local skill at `~/.codex/skills/chatgpt-outsource`.
- The current content is suitable for local iteration before any public GitHub cleanup.
