# WhatsApp Lead Outreach — n8n Workflow

An open-source [n8n](https://n8n.io) workflow that reads a CSV of business leads, cleans and normalises their phone numbers, and sends **one personalised WhatsApp message per business** through the **official WhatsApp Business Cloud API** — with rate limiting, deduplication, a per-run cap, and optional Google Sheets logging.

Built for freelancers (designers, developers, marketers) doing small-scale, personalised local outreach — e.g. offering graphic design services to local businesses.

## How it works

```
Start (manual)
  └─> Read Leads File (CSV)
        └─> Parse CSV
              └─> Prepare Leads  (code node: skip "sent", normalise +91 numbers, dedupe, cap at 40/run)
                    └─> One at a Time (batch of 1)
                          └─> Send WhatsApp Template  (HTTP POST to Graph API)
                                └─> Sent OK?
                                      ├─ yes ─> Log to Google Sheet (optional) ─┐
                                      └─ no  ─> (continues anyway)             │
                                                Wait 45s ───────────────────────┘
                                                    └─> back to "One at a Time"
```

## Files

| File | What it is |
|---|---|
| `whatsapp-outreach-workflow.json` | The n8n workflow — import this |
| `data/leads_template.csv` | CSV format the workflow expects (sample rows) |
| `data/agartala_leads.csv` | Real sample lead list: 113 Agartala businesses (from public Google Maps listings) |

## Quick start

### 1. Set up WhatsApp Business Cloud API (free tier available)

1. Go to [developers.facebook.com](https://developers.facebook.com) → **Create App** → type **Business**.
2. Add the **WhatsApp** product. You get a test number, a **Phone Number ID**, and a temporary access token.
3. Create a **message template** named `lead_intro` (category: *Marketing*):
   ```
   Hi {{1}}! I'm David, a local graphic designer.
   I noticed {{2}} — I'd love to help with that.
   If you're interested, just reply here. No pressure!
   ```
   `{{1}}` = business name, `{{2}}` = the "opportunity" column from the CSV.
4. Submit it for approval (usually approved within minutes to a few hours — keep it short, no spam words, clear opt-out).

### 2. Import into n8n

1. n8n → **Workflows → Import from File** → pick `whatsapp-outreach-workflow.json`.
2. Open the **Send WhatsApp Template** node and replace `YOUR_PHONE_NUMBER_ID` in the URL with your Phone Number ID.
3. Create a **Header Auth credential**:
   - Name: `Authorization`
   - Value: `Bearer YOUR_ACCESS_TOKEN`
   - Assign it to the node (already set to use Header Auth).
4. Put your leads CSV on the n8n instance (self-hosted: use the `/data` folder; n8n Cloud: replace the *Read Leads File* node with a Google Sheets/Drive trigger).
5. If you want logging, enable the **Log to Google Sheet** node, create a sheet with columns `Timestamp, Business, Phone, Category, Status`, and paste your Sheet ID into the node.

### 3. Test, then run

- Always test with **2–3 of your own numbers first**.
- Press **Execute Workflow**. Each send is followed by a 45-second pause and the run is capped at **40 messages** by default (edit `MAX_PER_RUN` in the *Prepare Leads* node).

## Costs

WhatsApp Cloud API template messages are charged per message by Meta (marketing templates in India cost well under ₹1 each at the time of writing; check [Meta's pricing page](https://developers.facebook.com/docs/whatsapp/pricing) for current rates). Conversations started by the business are also rate-limited by Meta. Keep runs small and personalised.

## ⚠️ Compliance — read this before your first run

This tool uses the **official, approved** WhatsApp Business Cloud API, which is the only legitimate way to send business-initiated WhatsApp messages. Even so:

- **Opt-in matters.** WhatsApp's Business Policy expects recipients to have agreed to hear from you. Messaging numbers scraped from public directories is *not* consent — expect some reports, and stop messaging anyone who asks you to.
- **Approved templates only.** Business-initiated messages must use templates Meta has approved. Do not try to send free-form messages via unofficial tools — that is the fastest way to get your number banned.
- **Small + personalised beats big + spammy.** 20 relevant, personalised messages will outperform 1,000 identical blasts — in replies *and* in account health.
- **Reply and stop.** The moment someone replies, move them to a real conversation; never re-blast people who ignored the first message.
- For scale beyond warm outreach, use **email** (much more tolerant of cold contact) or paid lead-gen platforms — and always respect local rules (in India, promotional messaging norms incl. TRAI/DND considerations apply).

## Extending it

- Swap the CSV reader for a **Google Sheets trigger** to log + read from the same sheet.
- Add an **onReply** branch using the WhatsApp webhook node.
- A/B test two templates by alternating on `{{ $itemIndex % 2 }}`.

## License

[MIT](LICENSE) — free to use, modify, and build on.
