# 🕷️ Browser Automation Framework

> **Production-grade workflow architecture for agent-driven browsing**

A structured framework for all browser automation tasks — 20+ workflows organized into 3 tiers: **Control Flows**, **Cross-Cutting Concerns**, and **Orchestration**. Includes implementation patterns, composition matrix, and decision tree.

## Architecture

```
                          ┌──────────────────────┐
                          │  BROWSER AUTOMATION  │
                          │     FRAMEWORK        │
                          └──────────┬───────────┘
                                     │
          ┌──────────┬───────────┬────┴─────┬──────────┬──────────┐
          │          │           │          │          │          │
     ┌────┴────┐┌───┴────┐┌─────┴────┐┌────┴────┐┌────┴────┐┌────┴────┐
     │ CONTROL ││ STATE  ││INTERACT  ││  DATA   ││RESILIENCE││ ORCHES  │
     │ FLOWS   ││ & AUTH ││PATTERNS  ││PIPELINE ││& RECOVER ││ TRATION │
     │ (Tier1) ││(Tier2) ││ (Tier2)  ││(Tier2)  ││ (Tier2) ││ (Tier3) │
     └────┬────┘└───┬────┘└─────┬────┘└────┬────┘└────┬────┘└────┬────┘
```

## Tiers

### Tier 1 — Control Flows (4 Primary Patterns)

| Pattern | Intent | Key Ops |
|---------|---------|----------|
| **Structured Scraping** (1.1) | Extract structured data into typed schemas | `browser_navigate`, `browser_snapshot`, `browser_console` |
| **Form Automation** (1.2) | Fill and submit complex multi-page forms | `browser_type`, `browser_click`, `browser_press`, `browser_vision` |
| **Session Browsing** (1.3) | Authenticated browsing with cookie persistence | `browser_console` (cookie ops), state checkpointing |
| **Agent Loop** (1.4) | Goal-directed autonomous browsing | `browser_vision`, `browser_snapshot`, LLM decision loop |

### Tier 2 — Cross-Cutting Concerns

| Concern | Sections |
|---------|----------|
| **Auth & State** | Login preservation (2.1), Multi-account rotation (2.2), Proxy & identity (2.3) |
| **Interaction Patterns** | Pagination (3.1), Dynamic waits (3.2), Modal handling (3.3), File downloads (3.4) |
| **Data Pipeline** | Schema-driven extraction (4.1), Change detection (4.2), Rate-limiting (4.3), Validation (4.4) |
| **Resilience** | Anti-bot evasion (5.1), CAPTCHA handling (5.2), Retry logic (5.3), Checkpoint & resume (5.4) |

### Tier 3 — Orchestration

- **Parallel Processing** (6.1): `delegate_task` for true parallelism
- **Cross-Site Workflows** (6.2): Chain research → synthesize → publish
- **Export Pipelines** (6.3): File system, database, API, notification

## Decision Tree

```
What are you automating?
├── Extracting data → Tier 1.1 (Scraping) + Tier 2C (Data Pipeline)
├── Filling forms → Tier 1.2 (Form Fill) + Tier 2B (Interaction Patterns)
├── Authenticated browsing → Tier 1.3 (Session) + Tier 2A (Auth & State)
└── Goal-directed exploration → Tier 1.4 (Agent Loop) + Tier 2D (Resilience)
```

## Composition Matrix

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

## Full Documentation

See [`docs/full-framework.md`](docs/full-framework.md) for the complete 28KB framework document with all code examples, implementation patterns, and composition guides.

## Quick Start

```python
# Tier 1.1: Structured Scraping
async def scrape(url, schema):
    browser_navigate(url)
    snapshot = browser_snapshot(full=True)
    return extract_with_schema(snapshot, schema)

# Tier 1.2: Form Automation
async def fill_form(url, fields, submit_sel):
    browser_navigate(url)
    for selector, value in fields.items():
        browser_click(ref=selector)
        browser_press(key="Control+a")
        browser_press(key="Backspace")
        browser_type(ref=selector, text=value)
    browser_click(ref=submit_sel)
```

## Security

This repository includes comprehensive `.gitignore` rules to prevent credential and token leaks:
- Environment files (`.env`, `.env.*`)
- Private keys (`.pem`, `.key`, `.ppk`)
- Credential stores (`credentials.json`, `secret_*`)
- Cloud provider configs (`.aws/`, `.azure/`, `.gcloud/`)
- Session and debug data

**Always use `.env.example` files for documentation — never commit real secrets.**

## License

MIT License — see [LICENSE](LICENSE) for details.