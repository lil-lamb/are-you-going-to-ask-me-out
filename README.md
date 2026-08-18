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

- **Screen 0 (fish)** — "what is the name of the fish?🐟". Type a name and press **Enter** (or Continue) to lock it in. A **copy link** button then appears — it copies the page URL with `?fish=<name>` baked in, so user A can send that link to user B, whose fish box opens pre-filled. The name also appears in the Screen 1 title: "are you going to ask me out, {name}? 💗" and is saved to the sheet.
- **Screen 1** — Yes/No. Hitting **No** runs a 3-strike system: the button shrinks, the label gets sadder 🥺, and on the 3rd click it goes grey, shrinks to minimum, and disables itself.
- **Screen 2** — pick 2–3 date plans (pills auto-lock at 3; unchecking frees a slot). Optional free-text "other suggestion" is always available; **Enter** also advances when valid.
- **Screen 3** — flowers? The only option is **Yes** — 30 bubbles drift across the box and dodge your cursor; catch one.
- **Screen 4** — a personalized confirmation and a one-time invisible-GET submission to your Google Sheet (image-load GET — reliable through Apps Script's redirect in all browsers).

## Backend: Google Sheet + Apps Script

### Step 1 — Create the Sheet
1. Make a new Google Sheet, e.g. `AskMeOut Responses`.
2. Row 1 headers: `Timestamp | No Clicks | Date Options | Custom Suggestion | Brought Flowers | Fish Name`

### Step 2 — Add the Script
1. In the Sheet: **Extensions → Apps Script**.
2. Replace the default code with:

```javascript
function doGet(e) {
  const p = (e && e.parameter) || {};
  return record({
    fishName: p.fishName || "",
    noClicks: parseInt(p.noClicks, 10) || 0,
    dateOptionsSelected: (p.dateOptionsSelected || "").split(",").map(s => s.trim()).filter(Boolean),
    customSuggestion: p.customSuggestion || "",
    broughtFlowers: String(p.broughtFlowers).toLowerCase() === "true"
  });
}

function doPost(e) {
  if (!e || !e.postData) return;
  return record(JSON.parse(e.postData.contents));
}

function record(data) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName("AskMeOut Responses") || ss.getSheets()[0];

  sheet.appendRow([
    new Date(),
    data.noClicks,
    (data.dateOptionsSelected || []).join(", "),
    data.customSuggestion || "",
    data.broughtFlowers,
    data.fishName || ""
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

function listCustomSuggestions() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName("AskMeOut Responses") || ss.getSheets()[0];
  const values = sheet.getRange(2, 4, Math.max(sheet.getLastRow() - 1, 0), 1)
    .getValues().flat().filter(String);
  Logger.log("Custom Suggestions (" + values.length + "):\n" + values.join("\n"));
}
```

> Note: the script writes to the tab named **"AskMeOut Responses"** (falling back to the first tab if missing), so rows land in the right place no matter which tab is active.

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

## Hosting — GitHub Pages

This repo is already a GitHub Pages-friendly static site. To publish:

1. Push `index.html` and `README.md` to a GitHub repo (this repo was created as `are-you-going-to-ask-me-out`).
2. In the repo: **Settings → Pages → Deploy from a branch → `main` → Save**.
3. Your site will be live at `https://<your-username>.github.io/are-you-going-to-ask-me-out/`.
4. Update `CONFIG.sheetWebAppUrl` to your Apps Script URL and re-push.

> Alternatively, drag the folder into [Netlify Drop](https://app.netlify.com/drop) or [Vercel](https://vercel.com).
