# Recipe: Scrape Page

## When to use
User wants to extract structured data from a web page (tables, prices, text content, lists).

## Required capabilities
Token-efficient page reading, structured data extraction.

## Preferred tool -> Fallback
**agent-browser** (accessibility tree is token-efficient and semantic) -> native -> Playwright (use targeted extraction with `locator().textContent()`, NOT `page.content()`)

## Prerequisites
- Target URL accessible
- If authenticated content, handle auth first (see SKILL.md auth modes)

## Steps (agent-browser)
1. Navigate to the target page:
   ```bash
   agent-browser open https://target-url.com
   ```
2. Snapshot to get the accessibility tree:
   ```bash
   agent-browser snapshot -i
   ```
3. Identify target elements from the snapshot (tables, lists, text blocks)
4. Extract data from each element:
   ```bash
   agent-browser get text @e1
   agent-browser get text @e2
   ```
5. For tables or repeated elements, extract each row/item individually
6. Structure the data as JSON, markdown table, or whatever format the user requested

## Steps (Playwright fallback)
Use targeted extraction — never full `page.content()`:
```javascript
import { chromium } from 'playwright';
const browser = await chromium.launch();
const page = await browser.newPage();
await page.goto('https://target-url.com');

const rows = await page.locator('table tr').allTextContents();
const price = await page.locator('.price').textContent();

await browser.close();
```

## Pagination
If data spans multiple pages:
1. Extract current page data
2. Re-snapshot and look for pagination controls:
   ```bash
   agent-browser snapshot -i
   ```
3. Click next:
   ```bash
   agent-browser click @eN
   ```
4. Re-snapshot and extract
5. Repeat until no more pages or user-specified limit reached

Always show a preview before extracting large datasets.

## Output
Structured data in the format the user requested (JSON, markdown table, plain text). Show a preview of the first few items before extracting everything.

## Failure modes
| Failure | Cause | Fix |
|---------|-------|-----|
| Anti-bot detection | Site blocks automated browsers | Try different user agent, or ask user to load page manually |
| Dynamic content not loaded | JS hasn't rendered yet | Add `waitForLoadState('networkidle')` before snapshotting |
| Login required | Content behind auth wall | Handle auth first (see SKILL.md auth modes), warn user about authenticated scraping |
| Rate limiting | Too many requests | Add delays between page loads |

## Cleanup
Close browser session after extraction. No persistent processes needed.
- agent-browser: `agent-browser close`
- Playwright: `await browser.close()`
