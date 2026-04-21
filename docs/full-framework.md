# 🕷️ Browser Automation Framework
## Production-Grade Workflow Architecture for Agent-Driven Browsing

---

## 0. Taxonomy Overview

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
          │         │          │          │          │          │
     ┌────┴────┐   ...        ...        ...        ...        ...
     │Scrape   │
     │FormFill │
     │Session  │
     │AgentLoop│
     └─────────┘
```

**Tier 1 — Control Flows**: The 4 primary execution patterns that every workflow instantiates.
**Tier 2 — Cross-Cutting Concerns**: Capabilities that modify/enhance control flows. Composable.
**Tier 3 — Orchestration**: Multi-agent, multi-site, multi-tab coordination.

---

## 1. TIER 1: CONTROL FLOWS (Primary Patterns)

These are the four foundational execution paradigms. Every browser automation task is an instance of one (or a composition of several) of these.

### 1.1 Structured Web Scraping
**Intent**: Extract structured data from pages into typed schemas.

```
NAVIGATE → SNAPSHOT/SLECT → EXTRACT → TRANSFORM → NEXT_PAGE? ──┐
                ▲                                               │
                └───────────────────────────────────────────────┘
```

| Aspect | Detail |
|--------|--------|
| **Input** | URL or URL pattern, CSS/XPath selectors, output schema (JSON/Pydantic) |
| **Output** | Typed array of records (JSON, CSV, database rows) |
| **Key Ops** | `browser_navigate`, `browser_snapshot`, `browser_console(eval)` |
| **Failure Modes** | Selector drift, dynamic rendering, rate limits, CAPTCHAs |
| **Composes With** | Pagination (3.1), Change Detection (4.2), Anti-Bot (5.1) |

**Implementation Pattern (Hermes)**:
```python
# Pseudocode for structured scraping workflow
async def scrape_structured(url, schema, paginate=None):
    records = []
    page_url = url
    while page_url:
        browser_navigate(page_url)
        snapshot = browser_snapshot(full=True)
        # Extract using schema-defined selectors
        extracted = extract_with_schema(snapshot, schema)
        records.extend(extracted)
        # Pagination handling
        page_url = resolve_next_page(paginate) if paginate else None
    return records
```

### 1.2 Multi-Step Form Automation
**Intent**: Fill and submit complex forms including multi-page wizards.

```
NAVIGATE → DETECT_FORM → FILL_FIELD* → SUBMIT → VERIFY → NEXT_STEP?
                                          ▲         │
                                          └─RETRY───┘
```

| Aspect | Detail |
|--------|--------|
| **Input** | URL, field mapping `{selector: value}`, submit selector, verification condition |
| **Output** | Success/failure status, resulting page data, screenshot evidence |
| **Key Ops** | `browser_type`, `browser_click`, `browser_press`, `browser_vision` |
| **Failure Modes** | Dynamic validation, hidden fields, CAPTCHAs, multi-step state loss |
| **Composes With** | Auth (2.1), Dynamic Waits (3.2), Modal Handling (3.3) |

**Implementation Pattern (Hermes)**:
```python
async def automate_form(url, fields, submit_sel, verify_fn):
    browser_navigate(url)
    for selector, value in fields.items():
        # Clear existing value first
        browser_click(ref=selector)
        browser_press(key="Control+a")
        browser_press(key="Backspace")
        browser_type(ref=selector, text=value)
    browser_click(ref=submit_sel)
    # Wait for submission to process
    await wait_for_condition(verify_fn)
    return verify_fn()
```

### 1.3 Session-Based Browsing
**Intent**: Maintain authenticated state across multiple page visits.

```
LOGIN → STORE_COOKIES → NAVIGATE_SERIES* → EXTRACT/INTERACT → SAVE_STATE
  ▲                                                         │
  └─────────────── RESUME ──────────────────────────────────┘
```

| Aspect | Detail |
|--------|--------|
| **Input** | Credentials, target URLs, session persistence config |
| **Output** | Cookie/token store, page data, session state checkpoint |
| **Key Ops** | `browser_navigate`, `browser_console(cookie_ops)`, `browser_vision` |
| **Failure Modes** | Session expiry, token refresh, MFA challenges, IP rotation |
| **Composes With** | Auth (2.1-2.2), Checkpointing (5.4), Proxy Rotation (2.3) |

**Implementation Pattern (Hermes)**:
```python
async def session_browse(creds, urls, resume_from=None):
    # Load or create session
    if resume_from:
        load_cookies(resume_from)
        browser_navigate(base_url)  # Set domain context
        inject_cookies(loaded_cookies)
    else:
        login_flow(creds)
        save_cookies(session_id)
    
    results = []
    for url in urls:
        browser_navigate(url)
        # Check session validity
        if session_expired():
            login_flow(creds)
            save_cookies(session_id)
        results.append(extract_page_data())
    return results
```

### 1.4 Agent-Style Browsing Loop
**Intent**: Goal-directed autonomous browsing with plan-execute-observe cycle.

```
PLAN ──→ EXECUTE ──→ OBSERVE ──→ DECIDE ──→ DONE?
             ▲                              │
             └──────────────────────────────┘ (continue loop)
```

| Aspect | Detail |
|--------|--------|
| **Input** | Natural language goal, constraints, max iterations |
| **Output** | Goal achievement status, trace log, extracted findings |
| **Key Ops** | `browser_vision`, `browser_snapshot`, `browser_click`, `browser_console` |
| **Failure Modes** | Infinite loops, goal drift, unexpected page states, budget exhaustion |
| **Composes With** | All Tier 2 concerns, especially Resilience (5.x) |

**Implementation Pattern (Hermes)**:
```python
async def agent_browse(goal, max_iterations=20):
    context = {"goal": goal, "iterations": 0, "findings": []}
    plan = decompose_goal(goal)
    
    for step in plan:
        while context["iterations"] < max_iterations:
            # Execute current step
            action = decide_next_action(context)
            execute_action(action)
            
            # Observe result
            observation = browser_vision(question="What happened?")
            context["findings"].append(observation)
            
            # Decide: continue, retry, or move on
            if goal_achieved(context):
                return context["findings"]
            context["iterations"] += 1
    return context["findings"]
```

---

## 2. TIER 2A: AUTHENTICATION & STATE

### 2.1 Login Flow Preservation
**Intent**: Handle authentication once, persist session across restarts.

| Component | Strategy |
|-----------|----------|
| **Cookie Store** | Serialize all cookies after login. Inject before next session. |
| **Token Refresh** | Monitor 401/403 responses. Auto-refresh OAuth tokens. |
| **MFA Handling** | Prompt user for TOTP/SMS via `clarify()`. Store TOTP secret for automation. |
| **Session Resume** | Check auth state on each navigation. Re-authenticate only when needed. |

**Anti-Pattern**: Logging in on every page visit. **Always** check session validity first.

```python
# Cookie persistence pattern
async def login_once(site_config):
    browser_navigate(site_config.login_url)
    fill_credentials(site_config.creds)
    if site_config.requires_mfa:
        mfa_code = get_totp(site_config.totp_secret)  # or clarify(user)
        browser_type(ref=mfa_field, text=mfa_code)
    browser_click(ref=submit_btn)
    
    # Persist full cookie jar
    cookies = browser_console(expression="document.cookie")
    save_to_vault(site_config.domain, cookies)
```

### 2.2 Multi-Account Rotation
**Intent**: Switch identities to distribute load, avoid rate limits, or access multi-tenant data.

| Strategy | When to Use |
|----------|-------------|
| **Sequential Rotation** | Rate-limited APIs, content that varies by account |
| **Random Selection** | Avoiding pattern detection, distributed data collection |
| **Sticky Sessions** | Stateful workflows requiring account continuity within a session |

**Rotation Architecture**:
```
┌──────────────┐    ┌─────────┐    ┌──────────────┐
│ Account Pool │───→│ Router  │───→│ Browser Inst. │
│ (creds+proxy)│    │ (LLM)   │    │ (stateful)    │
└──────────────┘    └─────────┘    └──────────────┘
                          │
                    ┌─────┴──────┐
                    │ Cooldown   │
                    │ Tracker    │
                    └────────────┘
```

### 2.3 Proxy & Identity Rotation
**Intent**: Rotate IP addresses and browser fingerprints to avoid detection.

| Concern | Implementation |
|---------|---------------|
| **IP Rotation** | Residential proxy pools (BrightData, Oxylabs). Rotate per-request or per-session. |
| **Fingerprint Spoofing** | Rotate User-Agent, viewport size, WebGL renderer, canvas hash via browser_console. |
| **Timing Jitter** | Add random delays (1-5s) between actions. Vary interaction patterns. |
| **Geographic Consistency** | Match proxy location to timezone, language, currency settings. |

**Key Rule**: All four identity layers (IP, fingerprint, timing, geography) must be internally consistent. A Japanese IP with en-US browser and 9-5 EST timing will get flagged.

---

## 3. TIER 2B: INTERACTION PATTERNS

### 3.1 Pagination & Infinite Scroll
**Intent**: Traverse multi-page or infinitely-scrolling content.

| Pattern | Trigger | Extraction |
|---------|---------|------------|
| **Numbered Pagination** | "Next" button, numbered pages | Click next → extract until no next |
| **Infinite Scroll** | Scroll threshold, load-more button | Scroll → wait for render → extract new items |
| **Cursor-Based** | API cursor/token in URL or response | Follow cursor URLs until exhausted |
| **Date-Range Pagination** | Calendar/date selectors | Iterate through date ranges |

```python
# Infinite scroll pattern
async def scrape_infinite_scroll(url, item_selector, max_scrolls=50):
    browser_navigate(url)
    seen = set()
    for i in range(max_scrolls):
        snapshot = browser_snapshot(full=True)
        new_items = extract_items(snapshot, item_selector)
        unseen = [item for item in new_items if item['id'] not in seen]
        if not unseen and i > 3:  # No new items after initial load
            break
        seen.update(item['id'] for item in new_items)
        browser_scroll(direction="down")
        await wait_for_render(2)
    return list(seen)
```

### 3.2 Dynamic Content & Wait Strategies
**Intent**: Handle SPAs, lazy-loaded content, and async rendering reliably.

| Wait Strategy | Function | Use Case |
|---------------|----------|----------|
| **Fixed Delay** | `await sleep(N)` | Last resort, when no signal exists |
| **Selector Exists** | Poll DOM for element | Standard: wait for content to appear |
| **Network Idle** | Monitor outgoing requests | SPA page transitions, AJAX loads |
| **Text Present** | Poll for text content | Confirm data rendered correctly |
| **Vision Confirm** | `browser_vision(question=...)` | Complex visual state verification |

**Implementation**:
```python
async def smart_wait(condition_type, value, timeout=30):
    """Wait for dynamic content using the most appropriate strategy."""
    start = time.time()
    while time.time() - start < timeout:
        if condition_type == "selector":
            result = browser_console(expression=f"document.querySelector('{value}') !== null")
            if result: return True
        elif condition_type == "network_idle":
            result = browser_console(expression="window.__networkIdle === true")
            if result: return True
        elif condition_type == "text":
            snapshot = browser_snapshot(full=True)
            if value in snapshot: return True
        await sleep(0.5)
    raise TimeoutError(f"Wait condition {condition_type}:{value} not met")
```

### 3.3 Modal & Overlay Handling
**Intent**: Dismiss cookie banners, newsletter popups, paywalls, and other interruptions.

```python
# Universal overlay handler — run before every primary action
async def dismiss_overlays():
    overlay_selectors = [
        '[class*="cookie"] [class*="accept"], [class*="cookie"] button',
        '[class*="modal"] [class*="close"]',
        '[class*="popup"] [class*="dismiss"]',
        '[class*="overlay"] [class*="close"]',
        'button[aria-label="Close"]',
        'button[aria-label="Dismiss"]',
    ]
    for sel in overlay_selectors:
        try:
            browser_console(expression=f"""
                document.querySelectorAll('{sel}').forEach(el => el.click())
            """)
        except: pass
```

### 3.4 File Download & Upload
**Intent**: Handle browser file dialogs for download and upload workflows.

| Operation | Strategy |
|-----------|----------|
| **Download** | Monitor download URLs via `browser_console`. Intercept `Content-Disposition` headers. |
| **Upload** | Use `browser_type` on file inputs or `browser_console` to set `.files` property. |
| **Bulk Download** | Queue URLs, download with rate limiting, verify checksums. |

---

## 4. TIER 2C: DATA PIPELINE

### 4.1 Structured Extraction (CSS/XPath → Schema)
**Intent**: Convert raw DOM into typed, validated data structures.

```python
# Schema-driven extraction
EXTRACTION_SCHEMA = {
    "product_name": {"selector": "h1.product-title", "type": "string", "required": True},
    "price": {"selector": ".price-current", "type": "decimal", "transform": "parse_price"},
    "availability": {"selector": ".stock-status", "type": "enum", "values": ["in_stock", "out_of_stock"]},
    "images": {"selector": ".product-gallery img", "type": "array", "attribute": "src"},
    "specs": {"selector": ".spec-table tr", "type": "dict", "key_col": 0, "val_col": 1},
}

async def extract_with_schema(url, schema):
    browser_navigate(url)
    snapshot = browser_snapshot(full=True)
    results = {}
    for field, config in schema.items():
        raw = extract_dom(snapshot, config["selector"], config.get("attribute"))
        results[field] = transform(raw, config.get("type"), config.get("transform"))
    return validate(results, schema)
```

### 4.2 Change Detection & Incremental Scraping
**Intent**: Only scrape what's new or changed since last run.

| Strategy | Implementation | Best For |
|----------|---------------|----------|
| **Content Hash** | Hash key sections. Only store if hash differs. | Pages that update in-place |
| **Timestamp Poll** | Track `last-modified` or embed timestamps. | Blogs, forums, feeds |
| **Diff-Based** | Full scrape → diff against previous. Store delta. | Versioned content, wikis |
| **Event-Driven** | Monitor RSS/Webhooks. Scrape on event. | News, releases, alerts |

```python
async def incremental_scrape(url, previous_hash=None):
    browser_navigate(url)
    content = browser_snapshot(full=True)
    current_hash = hash(content)
    
    if current_hash == previous_hash:
        return {"status": "unchanged", "data": None}
    
    # Extract only if changed
    data = extract_with_schema(content, SCHEMA)
    return {"status": "changed", "data": data, "hash": current_hash}
```

### 4.3 Rate-Limiting & Politeness
**Intent**: Be a good citizen. Avoid overwhelming servers and getting blocked.

| Tactic | Default | Aggressive |
|--------|---------|------------|
| **Inter-request delay** | 2-5s | 0.5-1s + jitter |
| **Concurrent tabs** | 1 | 3-5 (with proxy rotation) |
| **Requests per minute** | 15-30 | 60-120 |
| **Retry on 429** | Wait 60s, then retry with backoff | Switch proxy + account |
| **Robots.txt respect** | Always check | Override if critical |

### 4.4 Data Validation & Cleaning
**Intent**: Ensure extracted data is complete, consistent, and usable.

```python
# Validation pipeline
class ScrapedRecord(BaseModel):
    url: HttpUrl
    title: str = Field(min_length=1)
    price: Optional[Decimal] = None
    scraped_at: datetime = Field(default_factory=datetime.utcnow)

def validate_batch(records: List[dict]) -> Tuple[List[ScrapedRecord], List[dict]]:
    valid, invalid = [], []
    for r in records:
        try:
            valid.append(ScrapedRecord(**r))
        except ValidationError as e:
            invalid.append({"record": r, "errors": e.json()})
    return valid, invalid
```

---

## 5. TIER 2D: RESILIENCE & RECOVERY

### 5.1 Anti-Bot Evasion
**Intent**: Navigate bot detection systems without being blocked.

| Layer | Technique | Sophistication |
|-------|-----------|---------------|
| **Basic** | Random User-Agent, request delay, respect robots.txt | Low |
| **Intermediate** | Proxy rotation, fingerprint randomization, headless flags | Medium |
| **Advanced** | Stealth plugins, residential proxies, human-like mouse curves | High |
| **Expert** | Browser fingerprint consistency, behavioral analysis emulation | Very High |

**Hermes-Specific Evasion**:
```python
# Pre-navigation stealth setup
async def setup_stealth():
    # Remove webdriver flag
    browser_console(expression="""
        Object.defineProperty(navigator, 'webdriver', {get: () => undefined});
        // Remove automation indicators
        delete window.__selenium_unwrapped;
        delete window.__playwright;
        // Randomize canvas fingerprint
        const orig = HTMLCanvasElement.prototype.toDataURL;
        HTMLCanvasElement.prototype.toDataURL = function() {
            return orig.apply(this, arguments) + Math.random().toString(36).slice(2,4);
        };
    """)
```

### 5.2 CAPTCHA Delegation
**Intent**: Handle CAPTCHAs without blocking the automation pipeline.

| CAPTCHA Type | Strategy | Tool |
|-------------|----------|------|
| **reCAPTCHA v2** | Solve via AI vision or delegate to solving service | `browser_vision` + 2captcha |
| **reCAPTCHA v3** | Accumulate solving score over multiple visits | Behavior emulation |
| **hCaptcha** | Image classification via vision model | `browser_vision` + solving service |
| **Cloudflare Turnstile** | Usually invisible, avoid headless triggers | Stealth mode |
| **Custom/Image** | Vision model classification | `browser_vision(question=...)` |
| **Text CAPTCHA** | OCR + context | `browser_vision` |

**Escalation Pattern**:
```
Try AI solve (vision) → Try solving service → Ask user (clarify) → Skip task → FAIL
```

### 5.3 Retry Logic & Error Recovery
**Intent**: Recover from transient failures without manual intervention.

```python
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=30),
    retry=retry_if_exception_type((TimeoutError, ConnectionError, CaptchaError)),
    before_sleep=log_retry_attempt
)
async def resilient_navigate(url):
    try:
        browser_navigate(url)
        # Verify page loaded correctly
        snapshot = browser_snapshot()
        if is_captcha_page(snapshot):
            raise CaptchaError("CAPTCHA detected")
        if is_error_page(snapshot):
            raise PageLoadError(f"Error page: {extract_error(snapshot)}")
        return snapshot
    except BrowserTimeout:
        # Try refreshing once
        browser_navigate(url)
        return browser_snapshot()
```

### 5.4 Checkpoint & Resume
**Intent**: Save progress so interrupted scraping runs can resume without re-processing.

```python
class Checkpoint:
    def __init__(self, run_id, storage_path):
        self.run_id = run_id
        self.storage = storage_path
        self.state = self._load() or {
            "completed_urls": [],
            "failed_urls": {},
            "last_url": None,
            "cookies": None,
            "timestamp": None
        }
    
    def mark_complete(self, url, data):
        self.state["completed_urls"].append(url)
        self._save()
        self._persist_data(url, data)
    
    def get_next_url(self, url_queue):
        completed = set(self.state["completed_urls"])
        return next((u for u in url_queue if u not in completed), None)
    
    def restore_session(self):
        if self.state["cookies"]:
            browser_console(expression=f"document.cookie = '{self.state['cookies']}'")
```

---

## 6. TIER 3: ORCHESTRATION

### 6.1 Parallel Tab Management
**Intent**: Process multiple pages concurrently for throughput.

```
┌─────────┐    ┌─────────┐    ┌─────────┐
│  Tab 1  │    │  Tab 2  │    │  Tab 3  │
│ (Scrape)│    │ (Fill)   │    │(Scrape) │
└────┬────┘    └────┬────┘    └────┬────┘
     │              │              │
     └──────────────┼──────────────┘
                    │
              ┌─────┴─────┐
              │ Coordinator│
              │ (LLM)     │
              └───────────┘
```

**Note**: Hermes uses a single browser context. For true parallelism, use `delegate_task` to spawn subagents, each with their own browser session.

### 6.2 Cross-Site Workflows
**Intent**: Chain operations across multiple websites (e.g., research → aggregate → publish).

```python
# Cross-site workflow: Research → Synthesize → Publish
async def cross_site_workflow(topic):
    # Phase 1: Research across multiple sources
    results = await gather(*[
        scrape_structural(source.url, source.schema)
        for source in SOURCES
    ])
    
    # Phase 2: Synthesize findings
    synthesized = llm_synthesize(results, prompt=f"Summarize {topic}")
    
    # Phase 3: Publish to target
    automate_form(PUBLISH_URL, {
        "title_field": synthesized.title,
        "content_field": synthesized.content,
        "tags_field": synthesized.tags
    }, submit_sel="#publish-btn")
```

### 6.3 File Download Pipelines
**Intent**: Bulk download and process files (PDFs, images, datasets).

```python
async def download_pipeline(urls, output_dir, max_concurrent=3):
    semaphore = asyncio.Semaphore(max_concurrent)
    
    async def download_one(url):
        async with semaphore:
            browser_navigate(url)
            # Trigger download
            browser_click(ref=download_btn)
            # Wait for download to complete
            await wait_for_file(output_dir, expected_extension)
            return verify_download(output_dir / filename)
    
    results = await gather(*[download_one(url) for url in urls])
    return results
```

### 6.4 Export & Output Pipelines
**Intent**: Transform scraped data into usable formats and destinations.

| Destination | Format | Method |
|-------------|--------|--------|
| **File System** | JSON, CSV, XML | `write_file` |
| **Database** | SQL rows | `terminal` + sqlite/postgres client |
| **API** | JSON POST | `browser_console(fetch)` or `terminal(curl)` |
| **Cloud Storage** | Various | S3/GCS via CLI |
| **Notification** | Summary | `send_message` to Telegram/Discord |
| **Report** | Markdown/PDF | `write_file` + template rendering |

---

## 7. COMPOSITION MATRIX

Which concerns compose with which control flows:

| Concern ↓ / Flow → | Scraping | Form Fill | Session | Agent Loop |
|---------------------|----------|-----------|---------|------------|
| **Auth & Login** | ✅ | ✅✅ | ✅✅✅ | ✅✅ |
| **Multi-Account** | ✅✅ | ✅ | ✅✅ | ✅ |
| **Proxy/Identity** | ✅✅✅ | ✅ | ✅✅ | ✅✅ |
| **Pagination** | ✅✅✅ | ✅ | ✅ | ✅ |
| **Dynamic Waits** | ✅✅ | ✅✅✅ | ✅✅ | ✅✅✅ |
| **Modal Handling** | ✅✅ | ✅✅✅ | ✅ | ✅✅✅ |
| **File Ops** | ✅✅ | ✅ | ✅ | ✅✅ |
| **Schema Extract** | ✅✅✅ | ✅ | ✅ | ✅✅ |
| **Change Detect** | ✅✅✅ | — | ✅✅ | ✅ |
| **Rate Limiting** | ✅✅✅ | ✅ | ✅✅ | ✅✅ |
| **Anti-Bot** | ✅✅✅ | ✅✅ | ✅✅ | ✅✅✅ |
| **CAPTCHA** | ✅✅ | ✅✅✅ | ✅✅ | ✅✅✅ |
| **Retry/Recovery** | ✅✅ | ✅✅ | ✅✅✅ | ✅✅✅ |
| **Checkpoint** | ✅✅✅ | ✅ | ✅✅✅ | ✅✅ |
| **Parallel Tabs** | ✅✅✅ | ✅ | ✅ | ✅✅ |
| **Cross-Site** | ✅✅ | ✅✅ | ✅✅✅ | ✅✅✅ |
| **Export Pipeline** | ✅✅✅ | ✅ | ✅ | ✅✅ |

*✅ = useful | ✅✅ = recommended | ✅✅✅ = essential*

---

## 8. DECISION TREE

```
What are you automating?
├── Extracting data from pages
│   ├── Single page → 1.1 Scraping + 4.1 Schema Extraction
│   ├── Multiple pages → 1.1 Scraping + 3.1 Pagination + 4.3 Rate Limiting
│   └── Monitoring changes → 1.1 Scraping + 4.2 Change Detection + 5.4 Checkpoint
│
├── Filling forms / submitting data
│   ├── Single form → 1.2 Form Fill + 3.2 Dynamic Waits + 3.3 Modal Handling
│   ├── Multi-step wizard → 1.2 Form Fill + 5.3 Retry + 3.2 Waits
│   └── Authenticated forms → 1.2 Form Fill + 2.1 Auth + 5.2 CAPTCHA
│
├── Browsing with login state
│   ├── Simple session → 1.3 Session + 2.1 Auth
│   ├── Multi-account → 1.3 Session + 2.2 Account Rotation + 2.3 Proxy
│   └── Long-running → 1.3 Session + 5.4 Checkpoint + 5.3 Retry
│
└── Goal-directed exploration
    ├── Research task → 1.4 Agent Loop + 1.1 Scraping + 3.2 Waits
    ├── Interactive task → 1.4 Agent Loop + 3.3 Modals + 5.2 CAPTCHA
    └── Multi-site task → 1.4 Agent Loop + 6.2 Cross-Site + 2.1 Auth
```

---

## 9. IMPLEMENTATION PRIORITIES

### Phase 1: Foundation (Build First)
1. **Structured Scraping** (1.1) — The most common workflow
2. **Dynamic Waits** (3.2) — Every workflow needs this
3. **Schema Extraction** (4.1) — Makes scraped data usable
4. **Retry Logic** (5.3) — Essential for reliability

### Phase 2: Hardening (Build Second)
5. **Auth & Login** (2.1) — Unlock authenticated content
6. **Pagination** (3.1) — Scale beyond single pages
7. **Anti-Bot** (5.1) — Access protected sites
8. **Rate Limiting** (4.3) — Be a good citizen
9. **Form Automation** (1.2) — Data submission workflows

### Phase 3: Scale (Build Third)
10. **Change Detection** (4.2) — Enable monitoring
11. **Checkpointing** (5.4) — Resume interrupted runs
12. **CAPTCHA Handling** (5.2) — Unblock the hard sites
13. **Session Persistence** (1.3) — Long-running workflows
14. **Agent Loop** (1.4) — Autonomous browsing

### Phase 4: Advanced (Build Last)
15. **Multi-Account** (2.2) + **Proxy Rotation** (2.3) — Scale identity
16. **Cross-Site Orchestration** (6.2) — Complex pipelines
17. **Parallel Tabs** (6.1) — Throughput optimization
18. **Export Pipelines** (6.4) — Data destination flexibility

---

*Framework v1.0 — Built for Hermes Agent browser tools. Tool-agnostic patterns applicable to Playwright, Puppeteer, Selenium, or any headless browser automation.*