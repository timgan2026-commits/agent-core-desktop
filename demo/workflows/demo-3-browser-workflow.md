# Demo 3 — Browser workflow

Shows real browser automation as a distinct, sandboxed capability from
filesystem/terminal tools — controlling a browser tab, not the whole
desktop.

## Goal

> "Open the project's documentation site, find the page about
> installation, and extract the exact command it recommends for a first
> install."

## Steps

1. Agent Core plans a sequence of browser actions: navigate to the site,
   locate the installation page (via link text / structure, not screen
   coordinates), and extract the relevant text.
2. Each browser action is classified before it runs. Navigation and
   content extraction are `ALLOW`. Anything that looks like it would
   submit a form, make a purchase, or touch a password/payment field
   would instead be `ASK` — this demo doesn't need any of those, so it
   completes without stopping for approval.
3. The agent extracts the install command text from the rendered page
   (not from a raw HTML dump it has to guess at).
4. The Verifier confirms the extraction step actually returned non-empty,
   plausible content — an empty or clearly-failed extraction is not
   silently treated as success.

## Expected result

- The run reports the extracted install command as its result.
- The run's evidence includes what page was visited and what was
  extracted from it — reproducible, not just "the agent said so."
- If the page structure doesn't have what was asked for, the run reports
  that honestly (e.g. blocked/failed with a reason) rather than
  fabricating a plausible-looking answer.

## Evidence

- A real, end-to-end verified browser backend (not just a mocked
  adapter) supports: navigate, click, type, select, scroll, extract,
  screenshot, multi-tab management, file download (verified on disk
  before being reported as success), and file upload.
- Password and payment-shaped fields, and buy/submit-looking controls,
  are always `ASK` — verified as part of the same security-policy
  pipeline every other tool call goes through, not a browser-specific
  special case.
- Mock mode exists for fully offline development/testing and is loudly
  labeled as such in the run's own status — the UI never presents mock
  browser activity as if it were real.
