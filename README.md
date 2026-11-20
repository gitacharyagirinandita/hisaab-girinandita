# Hisaab – Digital Udhar Khata

**Live app:** https://hisaab-girinandita.vercel.app · **Guided tour:** https://hisaab-girinandita.vercel.app/?tour=1 · **Sample date:** https://hisaab-girinandita.vercel.app/?today=2026-11-15

Hisaab is a simple, mobile-first credit ledger (udhar khata) for Indian kirana, grocery, stationery and general stores. It replaces the paper notebook: add customers, write down goods given on credit, record payments (with an optional UPI screenshot), see who owes what, and send a polite WhatsApp reminder in one tap.

- **No sign-up, no server, no AI API.** Everything is stored in the shopkeeper's own browser (localStorage for the ledger, IndexedDB for screenshots).
- **Plain static site:** `index.html`, `styles.css`, `app.js`, `i18n.js`. No build step. Deploys to Vercel as-is.
- **Languages:** English, हिंदी, తెలుగు, ಕನ್ನಡ. **Themes:** light and dark.

## Features

| Area | What it does |
|---|---|
| Dashboard | Total outstanding (`#total-outstanding`), customer count, total credit, total received, collection %, search, Add Customer, customer list, overdue list, business insights, recent transactions, import |
| Customers | Add (with optional opening balance and due date), view, edit, delete with confirmation, duplicate-phone prevention, 10-digit Indian mobile validation |
| Account page | Name, phone, total credit, total payments, outstanding, full history newest first with running balance, Add Credit / Add Payment, WhatsApp reminder, statement download (CSV) and print/PDF |
| Entries | Credit or payment, amount (> 0, up to 2 decimals), date, note, optional screenshot (`input#shot-input`, preview before saving, thumbnail `img[data-shot]` after saving, survives refresh). Edit and delete with automatic recalculation |
| WhatsApp | `a[data-remind="<phone>"]` → `https://wa.me/91<phone>?text=…` with the customer's name and live balance. Opens WhatsApp prefilled; nothing is sent automatically. Customers with zero/advance balance get a "settled" message, never a payment demand |
| UPI | Add your own UPI ID in Settings and reminders include a `upi://pay` link for the exact due amount. No UPI ID is ever invented |
| Overdue | Customers with dues sorted by days since their last payment (or since first credit if they never paid), plus due-date tracking |
| Import | `textarea#import-box` + `button#import-go`, or upload a `.json` file. Validates everything and explains errors. Replacing data keeps an automatic backup; merging data from a different shop is refused |
| Export / backup | Full ledger CSV, balances CSV, per-customer statement, JSON backup (includes screenshots) and restore |
| Guided tour | Opens automatically on the first visit. **Step 1 is choosing the language** (English, हिंदी, తెలుగు, ಕನ್ನಡ), then 10 short steps highlight each part of the screen: total, summary cards, Add Customer, search, customer list and Remind, quick entry, overdue, import, Guide and Settings. Skip, Back, Next, arrow keys and Esc all work. It is shown once per device and can be replayed from **Guide** |
| Guide (bottom bar) | Written step-by-step instructions in all 4 languages (set up shop, add customer, give udhar, receive payment, remind, fix mistakes, overdue, backup), a language picker and a **Start guided tour** button. The bottom bar is now Home · Guide · **+** · Overdue · Settings |
| Dates | `?today=YYYY-MM-DD` sets the default entry date and the "days since" calculations. Otherwise the device's local date is used. No UTC conversion anywhere (tested in UTC-10 and UTC+14) |

Balances are **never stored**. They are always computed from the transaction history: `Outstanding = Total Credit − Total Payments`. Totals use integer paise to avoid rounding drift. The dashboard total is the sum of positive balances only.

## How to use Hisaab (for the shopkeeper)

On the first visit Hisaab opens a short guided tour. Pick your language, then tap **Next** to see what each part does. You can watch it again any time: **Guide** (bottom bar) → **Start guided tour**.

1. **Open the app** on your phone. Optionally tap ⚙ **Settings** to type your shop name and your UPI ID.
2. **Add a customer:** tap the green **+** button, enter the name and 10-digit mobile number. If they already owe you money from your old notebook, type it in **Opening balance**.
3. **Give goods on udhar:** open the customer, tap **Add Credit**, type the amount and a short note (e.g. "rice, oil"), and tap **Save entry**.
4. **Customer pays:** tap **Add Payment**, type the amount. If they paid by UPI you can tap **Attach payment screenshot** to keep the proof.
5. **Remind:** tap the green **WhatsApp Reminder** button. WhatsApp opens with a polite message showing the exact balance. You check it and press send yourself.
6. **Fix a mistake:** in the customer's history tap ✏️ to edit or 🗑 to delete an entry. Balances update instantly.
7. **Check who is late:** the **Overdue** list on the home screen shows who hasn't paid for the longest time.
8. **Back up weekly:** Settings → **Download backup (JSON)**. Keep the file safe. Your data lives only on this phone/browser.

## Importing another shop's records

Paste JSON into the **Import customer records** box at the bottom of the home screen and tap **Import**, or tap **Upload JSON file**. The expected format (same as the official `sample-data.json`):

```json
{
  "shop": "Sri Sai Stationery",
  "customers": [
    { "name": "Arjun Mehta", "phone": "9876543210" }
  ],
  "entries": [
    { "phone": "9876543210", "type": "credit",  "amount": 500, "date": "2026-11-01", "note": "notebooks" },
    { "phone": "9876543210", "type": "payment", "amount": 200, "date": "2026-11-10", "note": "cash" }
  ]
}
```

Rules:

- `phone`: 10 digits. `+91`, spaces and a leading `0` are cleaned up automatically.
- `type`: `credit` or `payment` (also accepts `udhar` / `paid` / `received` / `jama`).
- `amount`: a number greater than 0. Strings like `"1,200"` are accepted.
- `date`: `YYYY-MM-DD`. `DD/MM/YYYY` is also accepted.
- `transactions` is accepted instead of `entries`, and entries can also be nested inside each customer.
- Invalid rows are skipped and listed, so nothing changes silently. If an entry's phone is not in the customer list, a customer is created for it and a note is shown.
- **Replace current data** (default): the old shop is saved to *Settings → Automatic backups* (last 3 kept) and can be restored.
- **Add to current shop**: merges customers by phone and skips exact duplicate entries. It is refused if the file belongs to a different shop name.

## Running locally

```bash
python3 -m http.server 8765      # then open http://127.0.0.1:8765/?today=2026-11-15
```

## Tests

`tests/acceptance.py` drives headless Chrome through all 20 competition checks plus extra checks: validation, XSS escaping, timezones, merge refusal and close/reopen persistence.

```bash
pip install playwright
python3 tests/acceptance.py http://127.0.0.1:8765
```

`tests/tour.py` checks the guided tour and the Guide page.

The latest results are in [TESTING.md](TESTING.md).

> The tour does not open by itself for automated browsers (`navigator.webdriver`) or for links that set `?today=`, so it never covers the page during scripted competition tests. Add `?tour=1` to force it or `?tour=0` to suppress it.

## Project structure

```
index.html            app shell
styles.css            design system (light/dark)
app.js                application logic
i18n.js               English / Hindi / Telugu / Kannada strings
icon.svg, manifest.webmanifest
vercel.json           static hosting config + security headers
samples/              official sample-data.json and sample-expected.json
tests/                Playwright acceptance tests, report, screenshots
```

## Privacy and security

- All user text is HTML-escaped before rendering. CSV export guards against spreadsheet formula injection.
- A strict Content-Security-Policy is set in `vercel.json`. The app makes no network requests.
- No credentials or API keys are used or stored.
