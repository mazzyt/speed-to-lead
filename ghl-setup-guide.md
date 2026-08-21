# GHL setup guide: The Speed to Lead Audit

This covers what to build inside GoHighLevel so the quiz (`2026-08-21_speed-to-lead-audit_v01.html`) saves leads to your CRM, sends an immediate results email, and starts a nurture sequence. The quiz itself is a self-contained HTML file. It cannot create GHL forms or workflows on its own, so this is the part you (or your GHL admin) build by hand inside the dashboard.

## 1. Create custom contact fields

Go to **Settings > Custom Fields > Contact** and add:

| Field name          | Key (approx)          | Type                                |
| ------------------- | --------------------- | ----------------------------------- |
| Speed to Lead Score | `speed_to_lead_score` | Number                              |
| Speed to Lead Max   | `speed_to_lead_max`   | Number                              |
| Speed to Lead Band  | `speed_to_lead_band`  | Text (values: `high`, `mid`, `low`) |
| Business Name       | `business_name`       | Text (skip if you already have one) |

## 2. Create the form

Go to **Sites > Forms > Builder**, create a new form called "Speed to Lead Audit" with fields: First Name, Email, Business Name, and hidden fields for the three score fields above (hidden fields let JavaScript fill them in without the visitor seeing them).

Open the form's **Integrate** tab and copy its **Post URL**. That is the value that goes into the quiz file.

## 3. Wire the quiz to the form

In `2026-08-21_speed-to-lead-audit_v01.html`, find this line near the top of the `<script>` block:

```js
const GHL_FORM_WEBHOOK_URL = "REPLACE_WITH_YOUR_GHL_FORM_ENDPOINT";
```

Replace the placeholder with the Post URL from step 2. The quiz already sends this payload on submit:

```json
{
  "first_name": "Alex",
  "email": "alex@example.com",
  "business_name": "Alex's Plumbing",
  "speed_to_lead_score": 24,
  "speed_to_lead_max": 48,
  "speed_to_lead_band": "mid",
  "section_scores": [{ "name": "Speed to lead", "score": 8, "max": 10 }, "..."]
}
```

If your form's Post URL expects form-encoded fields instead of JSON, tell me and I will change the `fetch` call to match, since GHL's native form endpoints usually accept both.

## 4. Build the workflow

Go to **Automation > Workflows > New Workflow**, trigger: **Form Submitted** (the form from step 2).

**Step 1: Send immediate results email.**
Use the merge fields `{{contact.speed_to_lead_score}}`, `{{contact.speed_to_lead_max}}` and `{{contact.speed_to_lead_band}}` in the email body. A simple version:

> Subject: Your Speed to Lead score: {{contact.speed_to_lead_score}}/{{contact.speed_to_lead_max}}
>
> Hi {{contact.first_name}}, thanks for running the Speed to Lead Audit. Your score puts you in the "{{contact.speed_to_lead_band}}" band. [Insert one paragraph per band, or use a GHL conditional/If-Else step to send one of three pre-written emails instead.]

The cleanest version uses an **If/Else** step right after the trigger, branching on `speed_to_lead_band = high / mid / low`, sending a different templated email down each branch. That way each contact gets a result email that matches their actual band and section breakdown, rather than one generic email.

**Step 2: Add a wait step**, then branch into the 30-day nurture sequence in `2026-08-21_speed-to-lead-audit_30-day-nurture-sequence_v01.md`.

**Step 3: Add an exit condition.** Under the workflow's settings, add "Remove from workflow if opportunity created" or "if appointment booked" so anyone who books a Growth Review stops receiving nurture emails. This matters: nothing looks worse than nurturing someone who already said yes.

## 5. PDF results

The quiz has a **"Download my results as a PDF"** button on the results screen. It uses the browser's native print-to-PDF (no plugin, works everywhere), styled to hide all the interactive quiz chrome and just print the score, verdict, breakdown and a mention of the booking page. This happens entirely on the visitor's device, so there's nothing to configure on the GHL side for it.

If you'd rather have GHL generate and attach a PDF inside the automated email (instead of relying on the visitor to click the button), that needs a GHL document/PDF template built from a saved design, which can't be dynamically populated with someone's individual quiz answers without a paid PDF-generation integration (e.g. via Zapier/Make with a service like PDFMonkey or DocSpring). Worth doing later if this becomes a serious lead source. Not necessary to launch.

## 6. Embedding on your website

The quiz is a single self-contained HTML file, no build step, no external dependencies except Google Fonts. Two ways to put it on your GHL site:

**Option A, GHL "Custom HTML/Code" element (simplest):** In the Sites/Funnel builder, add a Custom HTML element to a page and paste the entire contents of `2026-08-21_speed-to-lead-audit_v01.html` inside it (everything from `<!doctype html>` to `</html>` works fine dropped into most code blocks, though some builders only accept the `<body>` contents plus a `<style>`/`<script>` tag; if the whole document is rejected, paste just the code inside `<body>...</body>` and the `<style>...</style>` block from `<head>` into the element instead).

**Option B, iframe embed:** Host the file somewhere with a stable URL (e.g. this GitHub repo via GitHub Pages, or your own hosting) and embed it with:

```html
<iframe
  src="https://your-hosting-url/speed-to-lead-audit.html"
  style="width:100%; min-height:900px; border:0;"
  title="The Speed to Lead Audit"
></iframe>
```

Option B is more reliable long-term since GHL's HTML editor sometimes strips `<script>` tags, and it lets you update the quiz by pushing to GitHub without re-pasting code into GHL each time.
