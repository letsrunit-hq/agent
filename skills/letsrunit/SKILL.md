---
name: letsrunit
description: Generate and execute browser tests using Gherkin (Given/When/Then) syntax via a real Playwright browser. Use when asked to write, run, or debug browser tests, or to automate and verify web UI behaviour.
compatibility: Requires the letsrunit MCP server to be configured. See https://github.com/letsrunit/letsrunit.
---

## MCP Tools

| Tool | Description |
|------|-------------|
| `letsrunit_session_start` | Launch a browser. Returns `{ sessionId }`. Does **not** navigate. |
| `letsrunit_run` | Execute Gherkin steps or a full feature. Returns `{ status, steps, reason?, journal }`. Does **not** return HTML. |
| `letsrunit_snapshot` | Get scrubbed page HTML on demand. Accepts `selector` and scrub options. |
| `letsrunit_screenshot` | Take a screenshot. Accepts `selector` (crop to element) and `mask` (spotlight elements). |
| `letsrunit_diff` | Diff the current live page against the HTML snapshot from the last passing test of a scenario. Pass the `scenarioId` returned by `letsrunit_run`. |
| `letsrunit_debug` | Evaluate JavaScript on the current page. Returns `{ result, error? }`. For debugging only. |
| `letsrunit_session_close` | Close the browser and release its resources. |
| `letsrunit_list_sessions` | List all active sessions. |
| `letsrunit_list_steps` | List available step definitions for the current session. Optional `type`: `Given`, `When`, `Then`. |

## Writing Tests

Tests are written in Gherkin. Every test must start with a `Given I'm on the homepage` or `Given I'm on page` step to navigate.
Do not assume step names from memory. Always call `letsrunit_list_steps` first and use that output as the source of truth.

Relative paths (e.g. `"/login"`) require a `baseURL` set in `cucumber.js`:

```js
export default {
  timeout: 30_000,
  worldParameters: { baseURL: 'http://localhost:3000' },
};
```

```gherkin
Feature: Login

Scenario: User logs in with valid credentials
  Given I'm on page "/login"
  When I set field "email" to "user@example.com"
  And I set field "password" to "secret"
  And I click button "Sign in"
  Then the page contains text "Dashboard"
  And I should be on page "/dashboard"
```

## Locators

Locators identify elements on the page. Prefer natural language over raw CSS selectors.

| Pattern | Example | Description |
|---------|---------|-------------|
| `button "text"` | `button "Sign in"` | Button by visible text or aria-label |
| `link "text"` | `link "Home"` | `<a>` tag by its text |
| `field "label"` | `field "Email"` | Input by label text, placeholder, or aria-label |
| `text "content"` | `text "Welcome"` | Any element containing the text |
| `image "alt"` | `image "Logo"` | Image by alt text |
| `date "value"` | `date "2025-01-22"` | Element by specific date |
| `date of {expr}` | `date of tomorrow` | Relative dates (`date of 3 days ago`, `date of 1 week from now at 20:00`) |
| `{role} "name"` | `menuitem "Profile"` | Element by ARIA role |
| `{tag}` | `div`, `section` | HTML tag name |
| `` `selector` `` | `` `.btn-primary` `` | Raw Playwright selector (last resort) |

**Scoping:** `` button "Submit" within `#checkout-form` `` — scope to a parent element.
**Filtering:** `section with button "Save"` / `section without text "Expired"` — filter by descendants.

**Rules:** prefer descriptive locators over attribute-based ones; use `link` for `<a>` tags; ensure locators are unambiguous.

## Values

- **String** — `"hello"`
- **Number** — `42`
- **Boolean** — `true` / `false`
- **Date** — `date of tomorrow`, `date of 3 days ago`, `date of 8 weeks from now` or `date "2025-01-22"`
- **Generated password** — `password of "some-user"` (requires `LETSRUNIT_PASSWORD_SEED` env var)
- **Array** — `["Option A", "Option B"]`

## Key Combinations

Standard key names: `Enter`, `Tab`, `Escape`, `Space`, `ArrowDown`.
Modifier combos: `Control+A`, `Shift+Tab`, `Meta+K`.

## When to use letsrunit vs Cucumber

**Use letsrunit** for the development loop: writing a scenario, running it, debugging failures, iterating. One scenario at a time with a live browser session you can inspect.

**Use Cucumber** for pass/fail suite runs across all scenarios. If the project has Cucumber configured, prefer it for running the full feature file.

When running Cucumber from the CLI for agent analysis, use the agent formatter:

```bash
yarn test --format @letsrunit/cucumber/agent
```

This formatter emits machine-oriented NDJSON with structured failure details and baseline diff context.

**Cucumber not configured?** Suggest running `npx letsrunit init` to set it up. If the user declines, fall back to running each scenario individually via the MCP, iterating through them one at a time.

## Workflow

1. `letsrunit_session_start` — launch the browser (no navigation)
2. `letsrunit_list_steps` — discover all steps available in this runtime
3. `letsrunit_run` with `Given I'm on the homepage` or `Given I'm on page "/path"` — navigate to the target URL
4. Call `letsrunit_snapshot` when you need to inspect the DOM
5. Propose **When** steps in small batches, run them, observe `status` and `steps`
6. Generate **Then** assertions based on what actually happened
7. `letsrunit_session_close` when done
8. Return the complete Gherkin feature

## Debugging

- Step failed → `letsrunit_snapshot` with `selector` to inspect the relevant DOM subtree
- Visual confirmation → `letsrunit_screenshot` with `mask` to spotlight the element
- Arbitrary JS → `letsrunit_debug`, e.g. `document.querySelector('#btn')?.textContent`
- Locator not found → try a broader selector or inspect the HTML for the actual text
- Regression → `letsrunit_diff` with `scenarioId` to see what changed compared to the last passing test
- Keep the session open after a failure to inspect state and run follow-up steps
- When encountering a failure running cucumber, explicitly classify the fix path:
  - **Test issue**: expected behavior/selector/assertion is stale or too strict; update the Gherkin test.
  - **Code issue**: application behavior regressed or is incorrect; update product code.
  - If uncertain, gather evidence (NDJSON failure payload, diff, snapshot, screenshot) and state which side is most likely before editing.

## Gherkin Rules

- Use `And` after the first step of a type — the runner treats it as the same type as the preceding keyword
- `But` is also accepted as an alias
- One scenario per `letsrunit_run` call
