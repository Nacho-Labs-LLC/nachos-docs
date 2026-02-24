---
title: "Agent Browser"
description: "Production-grade browser automation using Playwright for web interaction and data extraction."
---

# Agent Browser

The agent-browser skill provides production-grade browser automation powered by Playwright. It enables the AI to navigate websites, click elements, fill forms, extract data, take screenshots, and perform complex multi-step web workflows — all with security-first design.

## Overview

This skill wraps Playwright in an AI-friendly interface, giving your assistant the ability to:

- Navigate URLs and wait for page states
- Click buttons, fill forms, select options
- Extract text, tables, and structured data
- Take screenshots and visual verification
- Handle multi-tab workflows
- Manage sessions with cookies and storage
- Automate login flows
- Scrape dynamic content

**Security features:**
- Domain allowlisting
- Credential protection
- Sandboxed execution
- SSRF prevention
- Timeout controls

## When to Use

**Use agent-browser when:**
- You need to interact with web applications (click, type, submit)
- Content requires JavaScript execution (SPAs, dynamic sites)
- You need visual verification (screenshots)
- Login flows or complex multi-step automation required
- Testing web applications
- Extracting data from tables or complex DOM structures

**Use simpler tools when:**
- Static HTML pages → [Web Fetch](/tools/web-fetch)
- Just need to search for URLs → [Web Search](/tools/web-search)
- Simple API calls → Direct HTTP requests

## Core Capabilities

### 1. Navigation

Visit URLs and wait for content to load:

```python
page.goto('https://example.com')
page.wait_for_load_state('networkidle')
```

**Load states:**
- `load` — HTML loaded
- `domcontentloaded` — DOM ready
- `networkidle` — No more network requests (best for SPAs)

### 2. Element Interaction

Click, type, select, hover:

```python
# Click button
page.click('button[type="submit"]')

# Fill text input
page.fill('input[name="email"]', 'user@example.com')

# Select dropdown
page.select_option('select[name="country"]', 'US')

# Hover for tooltips
page.hover('.info-icon')

# Check checkbox
page.check('input[name="terms"]')
```

### 3. Data Extraction

Get text, attributes, structured data:

```python
# Extract text
title = page.title()
content = page.locator('main').text_content()

# Extract table as list of dicts
def scrape_table(page, selector):
    headers = page.locator(f'{selector} thead th').all_text_contents()
    rows = []
    for row in page.locator(f'{selector} tbody tr').all():
        cells = row.locator('td').all_text_contents()
        rows.append(dict(zip(headers, cells)))
    return rows
```

### 4. Screenshots

Visual verification and debugging:

```python
# Full page screenshot
page.screenshot(path='page.png')

# Element screenshot
page.locator('.chart').screenshot(path='chart.png')

# Full page with scroll
page.screenshot(path='full_page.png', full_page=True)
```

### 5. Multi-Tab Workflows

Coordinate actions across multiple tabs:

```python
# Create multiple pages
page1 = browser.new_page()
page2 = browser.new_page()

# Extract from first tab
data = page1.locator('.data').text_content()

# Use in second tab
page2.fill('input', data)
page2.click('button[type="submit"]')
```

### 6. Session Management

Save and restore browser state:

```python
# Save session
session_data = {
    'cookies': page.context().cookies(),
    'localStorage': page.evaluate('() => Object.entries(localStorage)'),
}

# Restore session
page.context().add_cookies(session_data['cookies'])
for key, value in session_data['localStorage']:
    page.evaluate(f'localStorage.setItem("{key}", "{value}")')
```

## Common Workflows

### Login Flow

```python
def login(page, username, password, login_url):
    page.goto(login_url)
    page.wait_for_load_state('networkidle')
    
    page.fill('input[name="username"]', username)
    page.fill('input[name="password"]', password)
    page.click('button[type="submit"]')
    
    page.wait_for_url('**/dashboard**')
    assert page.locator('.user-profile').is_visible()
```

### Form Submission

```python
def submit_form(page, form_data):
    for field, value in form_data.items():
        page.fill(f'input[name="{field}"]', value)
    
    page.select_option('select[name="country"]', 'US')
    page.check('input[name="terms"]')
    page.set_input_files('input[type="file"]', 'document.pdf')
    
    page.click('button[type="submit"]')
    page.wait_for_selector('.success-message, .error-message')
```

### Data Scraping

```python
def scrape_products(page, url):
    page.goto(url)
    page.wait_for_selector('.product-item')
    
    products = []
    for item in page.locator('.product-item').all():
        products.append({
            'name': item.locator('.product-name').text_content(),
            'price': item.locator('.product-price').text_content(),
            'url': item.locator('a').get_attribute('href')
        })
    return products
```

## Element Selection Strategies

**Priority order (most stable first):**

1. **Accessible roles** (best): `role=button[name="Submit"]`
2. **Test IDs**: `data-testid=submit-button`
3. **Labels**: `label >> input` or `text="Submit"`
4. **CSS selectors**: `button.submit-btn`
5. **XPath** (last resort): `//button[@class='submit']`

**Why accessible roles win:**
- Semantic meaning doesn't change when styling changes
- Works even if classes are renamed
- Accessible to screen readers (good for testing accessibility)

## Security Best Practices

### 1. Domain Allowlisting

```python
ALLOWED_DOMAINS = ['example.com', 'api.example.com']

def is_allowed(url):
    from urllib.parse import urlparse
    domain = urlparse(url).netloc
    return any(domain == d or domain.endswith(f'.{d}') 
               for d in ALLOWED_DOMAINS)
```

### 2. Credential Protection

- Use environment variables, never hardcode
- Never log sensitive fields
- Use separate "bot" accounts for automation

```python
import os
password = os.getenv('APP_PASSWORD')  # ✅

password = "hardcoded_secret"  # ❌ Never do this
```

### 3. Sandboxed Execution

```python
browser = playwright.chromium.launch(
    headless=True,
    args=[
        '--disable-dev-shm-usage',
        '--disable-gpu',
        '--no-sandbox'  # Only in controlled Docker environments
    ]
)
```

### 4. Timeout Controls

```python
page.set_default_timeout(30000)  # 30 seconds
page.set_default_navigation_timeout(60000)  # 60 seconds

try:
    page.wait_for_selector('.slow-element', timeout=10000)
except TimeoutError:
    print("Element did not appear")
```

## Error Handling

```python
from playwright.sync_api import TimeoutError, Error

def robust_automation(page, url):
    try:
        page.goto(url, wait_until='networkidle', timeout=60000)
    except TimeoutError:
        # Retry or handle
        page.goto(url, timeout=120000)
    except Error as e:
        if 'CONNECTION_REFUSED' in str(e):
            raise ConnectionError(f"Cannot connect to {url}")
        raise
```

## Performance Optimization

### Block Unnecessary Resources

```python
# Block images, fonts, CSS to speed up automation
context.route('**/*.{png,jpg,jpeg,gif,svg,css,font}', 
              lambda route: route.abort())
```

### Parallel Execution

```python
import asyncio
from playwright.async_api import async_playwright

async def scrape_multiple(urls):
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        async def scrape(url):
            page = await browser.new_page()
            await page.goto(url)
            content = await page.content()
            await page.close()
            return content
        
        results = await asyncio.gather(*[scrape(url) for url in urls])
        await browser.close()
    return results
```

## Debugging

### Visual Mode

```python
# Launch with visible browser
browser = playwright.chromium.launch(
    headless=False,
    slow_mo=1000  # Slow down by 1 second per action
)
```

### Console Logging

```python
page.on('console', lambda msg: print(f'Console: {msg.text}'))
page.on('request', lambda req: print(f'→ {req.url}'))
page.on('response', lambda res: print(f'← {res.url} {res.status}'))
```

### Trace Recording

```python
context.tracing.start(screenshots=True, snapshots=True)
# ... perform automation ...
context.tracing.stop(path='trace.zip')
# View at: https://trace.playwright.dev
```

## Common Pitfalls

❌ **Don't navigate before previous action completes:**

```python
page.click('button')
page.goto('https://example.com')  # May navigate too early
```

✅ **Wait for navigation:**

```python
page.click('button')
page.wait_for_url('**/next-page**')
```

❌ **Don't use brittle selectors:**

```python
page.click('div > div > div > button')  # Breaks easily
```

✅ **Use semantic selectors:**

```python
page.click('role=button[name="Submit"]')
```

❌ **Don't assume instant availability:**

```python
element = page.locator('.dynamic')
element.click()  # May fail if not loaded
```

✅ **Locators auto-wait:**

```python
page.locator('.dynamic').click()  # Waits automatically
```

## Installation

The skill requires Playwright to be installed:

```bash
# Install Playwright
pip install playwright

# Install browsers
playwright install chromium

# Optional: Firefox and WebKit
playwright install firefox webkit
```

## Integration with Nachos

When invoked by the assistant:

1. **Task analysis** — Determine required actions from user request
2. **Script generation** — Generate Playwright code using skill patterns
3. **Execution** — Run in sandboxed Docker container
4. **Result extraction** — Return structured data or screenshots
5. **Error handling** — Retry with adjustments if needed

## Full Skill Documentation

For comprehensive examples, advanced patterns, and reference scripts, see the full skill documentation:

```bash
cat skills/agent-browser/SKILL.md
```

Includes:
- Network interception
- JavaScript evaluation
- Dynamic content handling
- Session persistence patterns
- Testing and debugging workflows
- Complete reference scripts

## Related

- [Web Fetch](/tools/web-fetch) — Simple HTML fetching without browser
- [Web Search](/tools/web-search) — Find URLs before automation
- [Cron Jobs](/tools/cron) — Schedule periodic browser automation
- [Security Policies](/security/policies) — Domain and access controls
