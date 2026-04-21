---
name: browser-automation
category: software-development
description: Production-grade browser automation framework for Hermes agent. 20+ workflows organized into 3 tiers — Control Flows, Cross-Cutting Concerns, and Orchestration. Includes implementation patterns, composition matrix, and decision tree.
version: 1.0
---

# Browser Automation Framework

This skill provides a structured framework for all browser automation tasks using Hermes's built-in browser tools. Load this skill before any web scraping, form filling, session management, or autonomous browsing task.

## Quick Reference: Decision Tree

```
What are you automating?
├── Extracting data → TIER 1.1 (Scraping) + TIER 2C (Data Pipeline)
├── Filling forms → TIER 1.2 (Form Fill) + TIER 2B (Interaction Patterns)
├── Authenticated browsing → TIER 1.3 (Session) + TIER 2A (Auth & State)
└── Goal-directed exploration → TIER 1.4 (Agent Loop) + TIER 2D (Resilience)
```

## TIER 1: Control Flows (4 Primary Patterns)

### 1.1 Structured Web Scraping
- NAVIGATE → SNAPSHOT → EXTRACT → TRANSFORM → NEXT_PAGE → loop
- Keys: `browser_navigate`, `browser_snapshot`, `browser_console(eval)` for JS extraction
- Always use schema-driven extraction (CSS/XPath → typed fields)
- Compose with: Pagination (3.1), Change Detection (4.2), Anti-Bot (5.1)

### 1.2 Multi-Step Form Automation
- NAVIGATE → FILL_FIELDS → SUBMIT → VERIFY → retry on failure
- Keys: `browser_type`, `browser_click`, `browser_press`, `browser_vision`
- Always clear fields before typing: click → Ctrl+A → Backspace → type
- Compose with: Auth (2.1), Dynamic Waits (3.2), Modal Handler (3.3)

### 1.3 Session-Based Browsing
- LOGIN → SAVE_COOKIES → NAVIGATE_SERIES → EXTRACT → CHECKPOINT
- Keys: `browser_console` for cookie ops, save/restore between sessions
- Always check session validity before each navigation, re-auth only when expired
- Compose with: Auth (2.1-2.3), Checkpointing (5.4)

### 1.4 Agent-Style Browsing Loop
- PLAN → EXECUTE → OBSERVE → DECIDE → loop until goal achieved or budget exhausted
- Keys: `browser_vision` (observe), `browser_snapshot` (parse), LLM decides next action
- CRITICAL: Set max_iterations to prevent infinite loops
- Compose with: All Tier 2 concerns, especially Resilience (5.x)

## TIER 2A: Authentication & State

### 2.1 Login Flow Preservation
- Login once → serialize cookies → inject on resume
- MFA: use `clarify()` to ask user for TOTP/SMS codes
- Check auth state before every navigation, re-auth only when needed
- Anti-pattern: logging in on every page visit

### 2.2 Multi-Account Rotation
- Sequential: for rate-limited APIs
- Random: for pattern avoidance
- Sticky: for stateful multi-tenant workflows
- Architecture: Account Pool → LLM Router → Browser Instance

### 2.3 Proxy & Identity Rotation
- 4 identity layers must be CONSISTENT: IP, fingerprint, timing, geography
- Japanese IP + en-US browser + EST timezone = flag
- Use residential proxies, rotate User-Agent/viewport/canvas, add timing jitter

## TIER 2B: Interaction Patterns

### 3.1 Pagination & Infinite Scroll
- Numbered pages: click "next" until disabled
- Infinite scroll: scroll down → wait for render → check for new items → repeat
- Cursor-based: follow API cursor/token URLs
- Always track seen items to detect end-of-list

### 3.2 Dynamic Content & Wait Strategies
- PREFER smart waits over fixed delays
- Selector exists → text present → network idle → vision confirm → fixed delay (last resort)
- Implementation: poll DOM in loop with 0.5s intervals, timeout after configurable seconds

### 3.3 Modal & Overlay Handling
- Run overlay dismissal before every primary action
- Common selectors: cookie accept, modal close, popup dismiss, aria-label Close/Dismiss
- Use `browser_console` to bulk-click all overlay dismiss buttons via JS

### 3.4 File Download & Upload
- Download: monitor via `browser_console`, intercept Content-Disposition
- Upload: use `browser_type` on file input or `browser_console` to set `.files` property
- Bulk: queue URLs with rate limiting, verify checksums after download

## TIER 2C: Data Pipeline

### 4.1 Schema-Driven Extraction
- Define schema: `{field: {selector, type, transform, required}}`
- Extract each field from snapshot, transform to target type, validate with Pydantic
- Always validate extracted data before storing

### 4.2 Change Detection & Incremental Scraping
- Content hash: hash key sections, only store if changed
- Timestamp poll: check last-modified headers
- Diff-based: full scrape → diff → store delta
- Event-driven: monitor RSS/webhooks, scrape on trigger

### 4.3 Rate-Limiting
- Default: 2-5s delay, 15-30 req/min, 1 tab
- Aggressive: 0.5-1s + jitter, 60-120 req/min, 3-5 tabs + proxy
- Always respect robots.txt unless critical need
- On 429: exponential backoff, switch proxy/account

### 4.4 Validation
- Validate every record with Pydantic BaseModel
- Separate valid/invalid into two streams
- Log validation errors with full context for debugging

## TIER 2D: Resilience & Recovery

### 5.1 Anti-Bot Evasion
- 4 layers: Basic (UA rotation, delays) → Intermediate (proxy, fingerprint) → Advanced (stealth plugins, residential proxies) → Expert (behavioral emulation)
- Pre-navigation stealth: remove `navigator.webdriver`, delete `__selenium_unwrapped` and `__playwright`, randomize canvas fingerprint

### 5.2 CAPTCHA Handling
- Escalation chain: Try AI vision solve → Try solving service → Ask user via `clarify()` → Skip task → FAIL
- reCAPTCHA v2: `browser_vision` + 2captcha
- hCaptcha: vision model classification
- Cloudflare Turnstile: stealth mode (avoid headless triggers)

### 5.3 Retry Logic
- Use tenacity or manual retry with exponential backoff
- 3 attempts, wait 2s → 4s → 8s
- Retry on: TimeoutError, ConnectionError, CaptchaError
- On timeout: try one page refresh before giving up

### 5.4 Checkpoint & Resume
- Save state: completed URLs, failed URLs, cookies, timestamp
- On resume: skip completed URLs, retry failed ones, restore cookies
- Persist to file after each successful extraction

## TIER 3: Orchestration

### 6.1 Parallel Processing
- Hermes uses single browser context → use `delegate_task` for true parallelism
- Each subagent gets own browser session
- Limit concurrency to 3-5 parallel tasks

### 6.2 Cross-Site Workflows
- Chain: Research (scrape N sites) → Synthesize (LLM) → Publish (form fill)
- Auth management: separate cookies per site, rotate identities

### 6.3 Export Pipelines
- File system: `write_file` (JSON/CSV/XML)
- Database: `terminal` + sqlite/postgres client
- API: `browser_console(fetch)` or `terminal(curl)`
- Notification: `send_message` to Telegram/Discord

## Composition Matrix (Quick)

| Flow → | Scrape | Form | Session | Agent |
|--------|--------|------|---------|-------|
| Auth | ✅ | ✅✅ | ✅✅✅ | ✅✅ |
| Proxy | ✅✅✅ | ✅ | ✅✅ | ✅✅ |
| Waits | ✅✅ | ✅✅✅ | ✅✅ | ✅✅✅ |
| Modals | ✅✅ | ✅✅✅ | ✅ | ✅✅✅ |
| Schema | ✅✅✅ | ✅ | ✅ | ✅✅ |
| Anti-Bot | ✅✅✅ | ✅✅ | ✅✅ | ✅✅✅ |
| Retry | ✅✅ | ✅✅ | ✅✅✅ | ✅✅✅ |
| Checkpoint | ✅✅✅ | ✅ | ✅✅✅ | ✅✅ |

## Implementation Priority

1. **Phase 1**: Scraping + Waits + Schema + Retry
2. **Phase 2**: Auth + Pagination + Anti-Bot + Rate-Limiting + Forms
3. **Phase 3**: Change Detection + Checkpointing + CAPTCHA + Sessions + Agent Loop
4. **Phase 4**: Multi-Account + Cross-Site + Parallel + Export

---

*Full 28KB framework document with all code examples: `/home/ubuntu/browser-automation-framework.md`*