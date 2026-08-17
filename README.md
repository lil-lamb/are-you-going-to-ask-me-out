# Are You Going to Ask Me Out? 💌

A playful, flirty single-page "quiz" that nudges someone toward saying yes to a date. Four screens, no reloads, zero cost to host. Built with plain HTML/CSS/JS — no framework.

## Quick start

1. Open `index.html` in a browser — it works out of the box.
2. Want to personalize the name? Edit one line in `index.html`:

   ```js
   const CONFIG = { askerName: "Alex", sheetWebAppUrl: "PASTE_APPS_SCRIPT_URL_HERE" };
   ```

3. To receive responses, set up the Google Sheet backend below and paste the web app URL into `CONFIG.sheetWebAppUrl`.

## How it works

- **Screen 1** — "are you going to ask me out, {name}?" with Yes/No. Hitting **No** runs a 3-strike system: the button shrinks, the label gets sadder 🥺, and on the 3rd click it goes grey, shrinks to minimum, and disables itself.
- **Screen 2** — pick 2–3 date plans (pills auto-lock at 3; unchecking frees a slot). Optional free-text "other suggestion" is always available.
- **Screen 3** — flowers? The only option is **Yes**.
- **Screen 4** — a personalized confirmation and a one-time fire-and-forget `POST` to your Google Sheet.

## Backend: Google Sheet + Apps Script (free)

### Step 1 — Create the Sheet
1. Make a new Google Sheet, e.g. `AskMeOut Responses`.
2. Row 1 headers: `Timestamp | No Clicks | Date Options | Custom Suggestion | Brought Flowers`

### Step 2 — Add the Script
1. In the Sheet: **Extensions → Apps Script**.
2. Replace the default code with:

```javascript
function doPost(e) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const data = JSON.parse(e.postData.contents);

  sheet.appendRow([
    new Date(),
    data.noClicks,
    (data.dateOptionsSelected || []).join(", "),
    data.customSuggestion || "",
    data.broughtFlowers
  ]);

  // Email alert
  MailApp.sendEmail({
    to: "YOUR_EMAIL@example.com",
    subject: "Someone said yes! 🎉",
    body: "New response submitted:\n\n" + JSON.stringify(data, null, 2)
  });

  return ContentService.createTextOutput(
    JSON.stringify({ status: "success" })
  ).setMimeType(ContentService.MimeType.JSON);
}
```

3. Replace `YOUR_EMAIL@example.com` with your real address.

### Step 3 — Deploy
1. Click **Deploy → New deployment**.
2. Type: **Web app**.
3. Execute as: **Me**.
4. Who has access: **Anyone**.
5. Click Deploy and authorize permissions when prompted.
6. Copy the **Web app URL** and paste it into `CONFIG.sheetWebAppUrl` in `index.html`.

### Step 4 — Test
- Submit the page once, then confirm a row appears in the Sheet and you get the email.

## Hosting (free) — GitHub Pages

This repo is already a GitHub Pages-friendly static site. To publish:

1. Push `index.html` and `README.md` to a GitHub repo (this repo was created as `are-you-going-to-ask-me-out`).
2. In the repo: **Settings → Pages → Deploy from a branch → `main` → Save**.
3. Your site will be live at `https://<your-username>.github.io/are-you-going-to-ask-me-out/`.
4. Update `CONFIG.sheetWebAppUrl` to your Apps Script URL and re-push.

> Alternatively, drag the folder into [Netlify Drop](https://app.netlify.com/drop) or [Vercel](https://vercel.com) — all free.
