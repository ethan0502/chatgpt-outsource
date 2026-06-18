---
name: chatgpt-outsource
description: Use the ChatGPT webpage through the available Chrome extension to outsource low-context, high-token, or advisory tasks such as writing, rewriting, translation, summarization, web search, research, prompt drafting, and debugging advice. Use when the task can be expressed in a compact self-contained prompt, does not require full repository context, and the returned result can be independently verified before use. Do not use for sensitive secrets, private credentials, local-file editing, or final authority on code, commands, citations, or recent facts.
---

# chatgpt-outsource

Use ChatGPT as a bounded external advisor, not as a substitute for local inspection or final judgment.

## Workflow

1. Check whether the task is a good fit.
   Use this skill only when the request is self-contained, non-sensitive, and likely to consume many tokens locally or benefit from a second opinion.
2. Minimize and sanitize the prompt.
   Include only the task, the necessary context, the expected output shape, and any constraints. Redact secrets, personal data, private paths, tokens, cookies, and confidential content.
3. Ask for a useful output format.
   Prefer checklists, concise rewrites, structured comparisons, issue lists, or source-linked research summaries that are easy to verify locally.
4. Treat the result as advisory.
   Verify commands before running them, code before applying it, translations before submitting them, and factual claims against primary or local evidence.
5. Prefer local evidence on conflict.
   If the outsourced answer disagrees with the codebase, runtime behavior, tests, or verified sources, trust the local evidence.

## Browser Execution Notes

When using this skill through the Chrome extension:

1. Confirm the browser path works before trusting the setup.
   A successful page load is not enough. Verify that ChatGPT accepts a prompt and returns a visible assistant message.
2. If the extension handshake times out, open a fresh Chrome window on the selected profile and retry once.
   Do not keep looping on the same failed connection attempt.
3. Distinguish transport success from answer success.
   If the prompt appears in the conversation but the assistant container stays empty, the browser path may still be working while ChatGPT itself failed server-side.
4. If ChatGPT returns a generic error or stalls, reload once and retry with a minimal prompt.
   A minimal probe such as `Reply with exactly: OK` is a good smoke test because it isolates page health from prompt complexity.
5. After the minimal probe succeeds, send the real outsourced prompt.
   If the minimal probe fails too, treat the ChatGPT session as unhealthy and stop relying on it for the current task.

## Good Tasks

- Rewrite or condense text without introducing new facts
- Translate self-contained passages
- Summarize long documentation into actionable notes
- Research current external information with links
- Ask for debugging hypotheses or architecture alternatives
- Generate prompt drafts, checklists, or structured outlines

## Bad Tasks

- Edit local files precisely
- Reason over broad repository state that is hard to compress safely
- Send secrets, credentials, cookies, certificates, or personal data
- Delegate final security judgment, code review sign-off, or command execution decisions
- Rely on unverified claims about recent facts, prices, regulations, or current product behavior

## Prompt Construction

Build prompts with this shape when possible:

```text
Task:
[What ChatGPT should do]

Context:
[Only the minimum needed context]

Constraints:
- [Formatting, tone, scope, or factual limits]
- [What not to do]

Output:
[Exact output shape]
```

Keep prompts compact. If the task needs large amounts of local context to be correct, do not outsource it.

## Response Acceptance

Accept the outsourced run only when all of the following are true:

- The prompt was actually sent
- A visible assistant response was returned
- The response content matches the requested output shape
- The content can be checked locally or against primary sources

Do not treat a spinner, empty assistant container, or generic `Something went wrong` page error as a successful run.

## Prompt Templates

### Rewrite

```text
Rewrite the following text.

Goal:
[formal academic English / concise technical style / natural Chinese / etc.]

Requirements:
- Preserve the meaning.
- Do not add facts.
- Keep technical terms accurate.
- Output only the rewritten text.

Text:
[paste text]
```

### Translation

```text
Translate the following text from [source language] to [target language].

Requirements:
- Preserve technical meaning.
- Keep commands, code, filenames, and URLs unchanged.
- Use natural phrasing.
- Flag any ambiguous phrase briefly if needed.

Text:
[paste text]
```

### Debugging Advice

```text
I need debugging advice for this issue.

Goal:
[what should happen]

Problem:
[what happens instead]

Context:
[minimal relevant setup]

What I already tried:
[attempts]

Question:
Give likely causes, diagnostic steps, and possible fixes.

Important:
Do not assume access to my machine. Separate high-confidence fixes from guesses.
```

### Code Review Advice

```text
Review this code or design for likely issues.

Goal:
[intended behavior]

Context:
[minimal relevant context]

Code or design:
[paste code or description]

Please check:
- Correctness
- Edge cases
- Security concerns
- Maintainability
- Simpler alternatives

Output:
Give concrete findings first, then suggestions.
```

### Research Summary

```text
Find current information about [topic].

Requirements:
- Prefer primary or official sources when possible.
- Include links.
- Distinguish verified facts from inference.
- End with a practical recommendation or concise summary.
```

### Documentation Summary

```text
Summarize the following material for implementation work.

Requirements:
- Extract only actionable details.
- Preserve exact function names, commands, and option names.
- Remove repetition.
- Do not invent information.
- Output a checklist.

Material:
[paste material]
```

## Safety Rules

Before sending anything to ChatGPT, remove or redact:

- API keys
- Passwords
- Access tokens
- SSH keys
- Private certificates
- Session cookies
- Personal IDs
- Phone numbers
- Home addresses
- Private email addresses unless strictly necessary
- Proprietary code or confidential material that should not leave the local workflow

Use explicit placeholders such as `[REDACTED_API_KEY]`, `[TOKEN]`, `[PRIVATE_EMAIL]`, and `[PROJECT_PATH]`.

## Validation Rules

Verify outsourced output before relying on it:

- Check commands before execution
- Review code before applying it
- Verify citations and links
- Re-read translations before submission
- Review rewritten report text before delivery
- Re-check any recent or time-sensitive claim against current primary sources

## Decision Rule

Use `chatgpt-outsource` when the task can be captured in a short, safe, self-contained prompt and the response can be validated independently. Do not use it as a replacement for local inspection, execution, testing, or final judgment.
