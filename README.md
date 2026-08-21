# Speed to Lead Audit

A branded, interactive lead-magnet quiz for Amigo Marketing. Visitors answer 24 questions across five sections on how fast and consistently their business responds to new leads, get an instant score with a section-by-section breakdown, and are pushed toward booking a Business Growth Review.

## Files

- `index.html`, the whole quiz. Single file, no build step, no external dependencies apart from Google Fonts. Open it directly in a browser or embed it as-is.
- `ghl-setup-guide.md`, how to connect the quiz's lead capture to GoHighLevel: custom fields, the form, the automation workflow, and two ways to embed it on the website.
- `30-day-nurture-sequence.md`, the full 30-email nurture sequence for leads who complete the audit but do not book straight away.

## Editing

Everything lives in `index.html`: styles in the `<style>` block, quiz content and scoring logic in the `<script>` block near the bottom (`SECTIONS` array for the questions, the `showResults` function for the score bands and copy).

The GoHighLevel webhook endpoint is a placeholder until it's wired up:

```js
const GHL_FORM_WEBHOOK_URL = "REPLACE_WITH_YOUR_GHL_FORM_ENDPOINT";
```

See `ghl-setup-guide.md` for what to replace it with.
