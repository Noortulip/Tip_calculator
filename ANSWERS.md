# ANSWERS.md

---

## Q1 — How to run

**No install needed.** The project is a single self-contained HTML file with zero runtime dependencies.

### Simplest — just open it

```bash
git clone https://github.com/YOUR_USERNAME/gratuity-tip-calculator.git
cd gratuity-tip-calculator
open index.html          # macOS
# start index.html       # Windows
# xdg-open index.html   # Linux
```

### With a local server (recommended — avoids file:// quirks)

**Node.js (v16+):**
```bash
npx serve .
# → http://localhost:3000
```

**Python 3 (no install, ships with macOS/Linux):**
```bash
python3 -m http.server 8080
# → http://localhost:8080
```

**Deployed URL:** _(add Vercel/Netlify URL after deployment)_

---

## Q2 — Stack & design choices

**Why vanilla HTML/CSS/JS:**
A tip calculator is a pure calculation engine with four inputs and four outputs. Introducing React, Vue, or Svelte would add framework overhead — bundle size, hydration, component lifecycle complexity — to something that fits comfortably in under 300 lines of JS. Vanilla keeps the deployment a single file you can open directly from disk, which also makes the "fresh machine" setup trivially simple.

The app uses the Module Pattern (IIFE with `'use strict'`) rather than global variables, giving the encapsulation of a framework's component without the dependency cost.

**Design decision 1 — Two-column layout with results on the left on mobile:**
The inputs and results live in two equal-width panels on desktop. On mobile (≤620px), the grid collapses to a single column — but I reversed the DOM order with `order: -1` on the results panel so it appears _above_ the inputs. This matters because on mobile the on-screen keyboard pushes the viewport upward. If results were below inputs, the moment a user taps the bill field and the keyboard opens, the result panel would scroll completely off screen. By putting results above, the user can see the live update with the keyboard open. This affects the `.app` grid and the `.results-panel { order: -1 }` rule in the responsive block.

**Design decision 2 — Dark results panel against a light input panel:**
Rather than one card with inputs on top and results below (the obvious layout), I split them into two side-by-side panels with an intentional dark/light contrast. The results panel uses `background: var(--ink)` (near-black) while inputs stay on white. This does two things: it creates a visual hierarchy where "where you put things in" and "what you get out" feel like distinct surfaces, not a continuous form; and it makes the per-person amount — rendered in DM Serif Display at 3rem in white — immediately dominant without the user having to scan for it. The hierarchy is: giant white number (per person) → smaller mono numbers (tip, subtotal, grand total) → inputs. The dark panel creates that contrast cheaply with just a background color change. It affects `.results-panel` and all `.result-row` descendant styles.

---

## Q3 — Responsive & accessibility

**360px phone:**
- The two-column grid collapses to a single column via `grid-template-columns: 1fr`
- The results panel reorders to the top (`order: -1`) so it stays visible when the keyboard opens
- Font sizes use `clamp()` so the headline scales gracefully
- The tip presets remain a 5-column row (they're narrow enough to fit); their font drops to 11px
- Padding on `.panel` reduces from 1.75rem to 1.25rem to reclaim horizontal space
- Touch targets on stepper buttons are at least 44px tall

**1440px laptop:**
- The two columns stretch to `max-width: 960px` with a 24px gap
- Panel padding increases to 2rem 2.25rem
- The per-person amount scales to 3rem without clipping
- The layout doesn't reflow or change structure — it simply breathes more

**Accessibility handled — keyboard navigation and ARIA state on tip presets:**
Each preset tip button has `aria-pressed="true/false"` that updates when clicked. This tells screen readers whether a preset is currently selected rather than just that it's a button. The custom tip input clears the `aria-pressed` state on all presets when the user types, so there's never a mismatch between visual state and announced state. Tab order follows the visual reading order: bill → presets → custom tip → people stepper → reset. All error messages use `role="alert"` and `aria-live="polite"` so they're announced without stealing focus.

**Accessibility knowingly skipped — focus-visible ring on the hero number:**
The per-person result (`<span>` inside the results panel) doesn't have a focus ring because it's not interactive — it's display-only text inside an `aria-live` region. Screen readers will read updated values automatically when they change. I did not add any skip-link-to-results because the results panel is positioned first in DOM order on mobile anyway, and on desktop the two panels are side-by-side. A skip link was added for keyboard users navigating the page header, but not mid-panel. The trade-off: a keyboard user on desktop may have to tab through all inputs before hearing results announced. With more time I would test with NVDA and VoiceOver to confirm the `aria-live` regions fire as expected.

---

## Q4 — AI usage

**Tool used: Claude (this conversation)**

1. **Generated the initial app structure (first version above).** The AI produced a single-column card layout with a dark results panel as a sidebar. I restructured it to a proper two-column CSS Grid because the original used `display: flex` with `width: 50%` on both panels — that breaks below 480px without a media query. I replaced it with `grid-template-columns: 1fr 1fr` and `@media (max-width: 620px) { grid-template-columns: 1fr }`, which reflows correctly and lets me use `order: -1` on the results panel for the mobile keyboard fix. The AI's flex version had no mobile reorder — the results would always render below the keyboard.

2. **Generated the validation pattern.** The AI's first pass used `window.alert()` for invalid input (which the spec explicitly forbids) and CSS class `hidden` toggled with `display: none`. I replaced it with animated inline error messages using `opacity` and `translateY` transitions so errors appear/disappear gracefully without layout shift. I also changed the error announcements from `aria-live="assertive"` (which interrupts screen readers mid-sentence) to `aria-live="polite"` to not be disruptive.

3. **Generated the rounding logic.** The AI initially used `Math.round()`. I changed it to `Math.ceil()` at the cent level (`Math.ceil(n * 100) / 100`) and added a "remainder note" that tells the user how much the group is overpaying due to rounding. This is both more correct for the real-world use case (a restaurant never cares if you tip slightly more than the exact %; they do care if you tip slightly less) and more transparent.

4. **Generated the `keydown` handler for number inputs.** The AI blocked `e`, `E`, `+` but missed `-`. On a standard number input, typing `-` in a bill field produces a negative number. I added `-` to all the block lists, and also blocked `.` specifically on the people input (people must be integers — decimals are meaningless there).

---

## Q5 — Honest gap

**What isn't polished enough:** the mobile keyboard / scroll interaction.

When a user on a phone taps the bill input, the on-screen keyboard opens and the browser viewport shrinks. The results panel is positioned above the inputs in the DOM, but depending on the browser and OS (especially Safari on iOS with its dynamic toolbar), the panel can still get partially obscured. I moved results above inputs in DOM order but haven't tested on physical devices — I don't know whether `dvh` (dynamic viewport height) is enough to prevent the per-person number from getting clipped behind the keyboard.

**What I'd do with another day:** Test on an actual iPhone (Safari) and Android (Chrome). iOS Safari's behavior around `100dvh`, `scrollIntoView`, and the Virtual Keyboard API is notoriously inconsistent. I'd use the [VirtualKeyboard API](https://developer.mozilla.org/en-US/docs/Web/API/VirtualKeyboard_API) where available to get the keyboard's geometry and explicitly scroll the results panel into view when any input gains focus. On iOS, where that API isn't available, I'd add a `focusin` event that calls `document.getElementById('results').scrollIntoView({ behavior: 'smooth', block: 'start' })` after a short delay. That's a two-hour fix that would make the mobile experience genuinely reliable rather than "probably fine."
