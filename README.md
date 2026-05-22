# Gratuity — Tip Calculator

Live tip/bill splitter. Single HTML file, zero dependencies, no build step.

**Deployed:** _(add URL)_

---

## Run locally

```bash
git clone https://github.com/YOUR_USERNAME/gratuity-tip-calculator.git
cd gratuity-tip-calculator
```

Then either:

```bash
# Option A — open directly (no server needed)
open index.html

# Option B — local server via Node
npx serve .
# → http://localhost:3000

# Option C — local server via Python
python3 -m http.server 8080
# → http://localhost:8080
```

No `npm install`. No build step. Works offline after the Google Fonts request.

---

## Features

- Live calculation — no Calculate button
- 5 preset tip %s + custom % input
- Bill splitter up to 500 people
- Inline validation (no alerts, no browser tooltips)
- Rounding policy: ceiling to nearest cent per person (group never underpays); overpayment shown
- Keyboard accessible — full tab navigation, `aria-live` results, `aria-pressed` tip presets
- Responsive: 360px phone through 1440px desktop

## Stack

Vanilla HTML / CSS / JS — no framework, no bundler. Google Fonts (DM Serif Display, DM Mono, Outfit) loaded via CDN.

## Files

```
index.html   ← entire app
README.md
ANSWERS.md
```
