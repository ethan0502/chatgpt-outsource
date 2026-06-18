# chatgpt-outsource

Use ChatGPT as a bounded external advisor for safe, self-contained tasks such as rewriting, translation, summarization, research, and debugging advice, while keeping local inspection, execution, and verification as the final authority.

This repository contains a Codex skill definition that helps an agent decide when outsourcing is appropriate, how to sanitize prompts before sending them to ChatGPT, and how to validate the returned answer before using it.

## What This Is

`chatgpt-outsource` is a skill for Codex-style agents. It is designed for cases where:

- the task is compact and self-contained
- the task is non-sensitive
- the answer is useful as advice, not as final authority
- the result can be checked locally against code, tests, runtime behavior, or primary sources

The skill is intentionally conservative. It does not treat ChatGPT as a trusted executor, code reviewer of record, or source of truth for recent facts.

## What This Is Not

This repository is not:

- a browser automation project
- a Playwright test suite
- a general-purpose ChatGPT API wrapper
- a replacement for local debugging, testing, or code review

Playwright support in this README is optional and only for users who want to validate browser-driven flows around the ChatGPT webpage.

## Repository Layout

```text
chatgpt-outsource/
├── README.md
├── SKILL.md
└── agents/
    └── openai.yaml
```

- `SKILL.md`: the actual skill definition, rules, and prompt templates
- `agents/openai.yaml`: metadata for the OpenAI/Codex agent interface
- `README.md`: installation, usage, and verification guidance

## When To Use This Skill

Good fits:

- rewriting or condensing text without adding facts
- translating self-contained passages
- summarizing documentation into actionable notes
- asking for debugging hypotheses or architecture alternatives
- asking for structured research summaries with links
- drafting prompts, checklists, or outlines

Bad fits:

- editing local files precisely
- reasoning over a large repository state that cannot be compressed safely
- sending secrets, credentials, cookies, certificates, or personal data
- relying on unverified current facts, regulations, prices, or product behavior
- delegating final security judgment or command execution decisions

## How It Works

The intended workflow is:

1. Decide whether the task is safe and compact enough to outsource.
2. Reduce the prompt to the minimum necessary context.
3. Remove secrets and sensitive data.
4. Ask ChatGPT for a response in a format that is easy to verify.
5. Validate the answer locally before using it.
6. Prefer local evidence whenever the outsourced answer conflicts with code, tests, runtime behavior, or primary sources.

## Installation

### Option 1: Install from a local checkout

If you already cloned this repository:

```bash
mkdir -p ~/.codex/skills
cp -R chatgpt-outsource ~/.codex/skills/
```

If the repository contains this skill under a subdirectory such as `GitHub-to_publish/chatgpt-outsource`:

```bash
mkdir -p ~/.codex/skills
cp -R GitHub-to_publish/chatgpt-outsource ~/.codex/skills/chatgpt-outsource
```

### Option 2: Install from a GitHub clone

```bash
git clone <your-repo-url>
cd <your-repo>
mkdir -p ~/.codex/skills
cp -R chatgpt-outsource ~/.codex/skills/
```

### Verify Installation

After installation, these files should exist:

- `~/.codex/skills/chatgpt-outsource/SKILL.md`
- `~/.codex/skills/chatgpt-outsource/agents/openai.yaml`

You can verify with:

```bash
ls ~/.codex/skills/chatgpt-outsource
ls ~/.codex/skills/chatgpt-outsource/agents
```

## Requirements

Minimum practical requirements:

- Codex or a compatible local agent that loads skills from `~/.codex/skills`
- access to ChatGPT in Chrome
- a working browser path if your Codex setup uses a Chrome extension or browser bridge

Optional tooling:

- Playwright, if you want to automate or verify browser-side behavior

## Quick Start

Once the skill is installed, use it only when the task is safe to send out and the response can be verified locally.

Example request:

```text
Use $chatgpt-outsource to rewrite this paragraph into concise technical English.

Constraints:
- Preserve meaning.
- Do not add facts.
- Keep commands and filenames unchanged.

Output:
Return only the rewritten paragraph.
```

Another example:

```text
Use $chatgpt-outsource to summarize this documentation into an implementation checklist.

Constraints:
- Preserve exact function names and CLI flags.
- Do not invent information.
- Keep the output short and actionable.

Output:
A flat checklist.
```

## Prompt Design

The skill works best when prompts follow a predictable structure:

```text
Task:
[What ChatGPT should do]

Context:
[Only the minimum needed context]

Constraints:
- [Formatting, tone, scope, factual limits]
- [What not to do]

Output:
[Exact output shape]
```

### Prompt Design Rules

- keep the prompt compact
- include only the context needed to complete the task
- state what must not change
- state the exact desired output format
- avoid sending large repository dumps
- do not send secrets or sensitive identifiers

### Redaction Examples

Replace sensitive values with placeholders such as:

- `[REDACTED_API_KEY]`
- `[TOKEN]`
- `[PRIVATE_EMAIL]`
- `[PRIVATE_PHONE]`
- `[PROJECT_PATH]`

## Validation Rules

Treat every outsourced answer as advisory until it is checked.

Validate based on the task type:

- commands: inspect before running
- code: review before applying
- research: verify links and claims against primary sources
- translations: re-read before submitting
- rewrites: confirm tone, meaning, and factual fidelity
- recent or time-sensitive claims: verify again before relying on them

If the outsourced answer conflicts with local evidence, trust the local evidence.

## Browser-Side Verification

If your Codex setup reaches ChatGPT through a Chrome extension or browser bridge, a successful page open is not enough. You need proof that the page can receive a prompt and return a visible assistant response.

Recommended smoke test:

1. Open ChatGPT through the intended browser path.
2. Send `Reply with exactly: OK`.
3. Confirm that a visible assistant message returns `OK`.
4. Only after that, send the real outsourced prompt.

Treat the run as failed if:

- the prompt never submits
- the assistant response area stays empty
- the page shows a generic error such as `Something went wrong`
- the browser transport works but ChatGPT itself does not return usable content

## Optional: Playwright Setup

Playwright is not required for this skill itself. Use it only if you want to:

- verify that the browser path is healthy
- inspect the ChatGPT page state during failures
- build your own smoke tests around the ChatGPT webpage

### Official Installation Path

Based on the official Playwright documentation, the standard setup is:

```bash
npm init playwright@latest
```

For a minimal scratch environment, you can also do:

```bash
mkdir -p /tmp/chatgpt-outsource-playwright
cd /tmp/chatgpt-outsource-playwright
npm init -y
npm install -D @playwright/test
npx playwright install
```

Verify the CLI:

```bash
npx playwright --version
```

### Notes for Browser Downloads

Playwright browser binaries are installed separately from the npm package. After updating Playwright, you may need to run:

```bash
npx playwright install
```

If you only need Chromium:

```bash
npx playwright install chromium
```

If your network path to the default download host is unstable, use an already installed Chrome or Chromium, or configure Playwright download settings in your own environment. See the official Playwright browser installation docs.

Useful references:

- Playwright installation: `https://playwright.dev/docs/intro`
- Playwright browser installation: `https://playwright.dev/docs/browsers`
- Playwright CLI reference: `https://playwright.dev/docs/test-cli`

### Minimal Playwright Probe

If you want a minimal browser probe, create a file such as `smoke.spec.ts`:

```ts
import { test, expect } from '@playwright/test';

test('chatgpt page opens', async ({ page }) => {
  await page.goto('https://chatgpt.com/');
  await expect(page).toHaveURL(/chatgpt\.com|chat\.openai\.com/);
});
```

Run it with:

```bash
npx playwright test smoke.spec.ts
```

This only proves the page opens. It does not prove your ChatGPT session is healthy, your browser bridge is healthy, or that the assistant can return a real answer. For that, use the browser smoke test described earlier.

## Usage Examples

### Rewrite

```text
Rewrite the following text.

Goal:
concise technical English

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
Translate the following text from Chinese to English.

Requirements:
- Preserve technical meaning.
- Keep commands, code, filenames, and URLs unchanged.
- Use natural phrasing.
- Flag any ambiguity briefly if needed.

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

### Research Summary

```text
Find current information about [topic].

Requirements:
- Prefer official or primary sources when possible.
- Include links.
- Distinguish verified facts from inference.
- End with a short practical recommendation.
```

## Troubleshooting

### The page opens, but no answer appears

Possible causes:

- ChatGPT session is unhealthy
- browser bridge connected, but response rendering failed
- prompt submission worked, but ChatGPT errored server-side

What to do:

1. reload once
2. retry the minimal probe `Reply with exactly: OK`
3. if the probe fails too, stop relying on that ChatGPT session for the current task

### The browser bridge times out on first connection

Possible cause:

- stale browser state or handshake failure

What to do:

1. open a fresh Chrome window on the intended profile
2. retry once
3. if the retry still fails, stop and investigate the browser path rather than looping

### The answer looks plausible but is still risky

That is expected. This skill is designed around the assumption that plausible answers still need verification.

## Security and Privacy

Before sending anything to ChatGPT, remove or redact:

- API keys
- passwords
- access tokens
- SSH keys
- private certificates
- session cookies
- personal IDs
- private phone numbers
- private addresses
- private email addresses unless truly necessary
- proprietary code or confidential material that should not leave the local workflow

This skill is only safe when the operator is disciplined about prompt minimization and redaction.

## Scope and Limitations

This skill can improve efficiency, but it has strict limits:

- it helps with advisory work, not final decisions
- it cannot replace local verification
- it is unsuitable for sensitive or high-context tasks
- it should not be used as evidence that a command, code change, or factual claim is correct

## Files

- `SKILL.md`: the skill definition and operating rules
- `agents/openai.yaml`: the OpenAI agent interface metadata

## Notes

- This repository was initialized from an installed local skill at `~/.codex/skills/chatgpt-outsource`.
- The browser-specific lessons are intentionally documented in both `SKILL.md` and this README so public users understand the failure modes.
- Playwright is optional support tooling for your own validation workflow. It is not required for the skill definition itself.

## License

MIT. See [LICENSE](/Users/xuyijun/.openclaw/workspace/GitHub-to_publish/chatgpt-outsource/LICENSE).
