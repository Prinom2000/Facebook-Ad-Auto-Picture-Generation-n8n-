# Facebook Ad Auto-Poster & Picture Generation (n8n)

An end-to-end n8n workflow that turns a Google Sheets content calendar into finished Facebook posts for **Kasri**, a Bangladeshi natural pain-relief oil brand. It generates the ad image with OpenAI, writes the caption, publishes to a Facebook Page, and logs everything back to the sheet, with no manual steps.
<img width="1863" height="362" alt="image" src="https://github.com/user-attachments/assets/0077719e-ab56-4c69-8f91-3b6f14a5e7a6" />

## What it does

Twice a day (10:00 and 20:00), the workflow:

1. Reads the next unprocessed row from the content calendar (Google Sheets).
2. Generates a branded **1024x1024 ad image** (or a **quiz post**) with an OpenAI image model, using the Kasri logo and the real product photo as references.
3. Resizes the image to **1080x1350** (Facebook portrait format).
4. Saves the image to Google Drive.
5. Writes a short **Bangla or English caption** with an OpenAI text model (hook question, Kasri answer, safety note, CTA), then appends a fixed contact footer.
6. Publishes the photo and caption to the Facebook Page via the Graph API.
7. Updates the sheet with status, caption, image link, post link and timestamp.

If any step fails, a Telegram alert is sent with a link to the failed execution.

## Workflow overview

```
Schedule (10 AM / 8 PM)
  -> Read Content Calendar (Google Sheets)
  -> new? (Status is empty)  -> Limit (1 row)
  -> If Product type is empty?
       |- Yes (Quiz post):   Download logo -> Edit image (quiz) -> Resize
       |- No  (Product ad):  Download logo + Search/Download product image
                             -> Merge -> Edit image (ad) -> Resize
  -> Upload image to Google Drive
  -> Generate caption (OpenAI) -> Add contact footer (Code)
  -> Update sheet: Status = Generated
  -> Restore image binary + caption (Code)
  -> Post to Facebook Page (Graph API)
  -> Update sheet: Status = Uploaded, Post Link, Date & Time
```

## Content calendar (Google Sheet)

| Column | Purpose |
|---|---|
| `Topic` | Main idea of the ad, or the subject of the quiz |
| `Product type` | Used to find the product image in Drive. Leave empty for a quiz post |
| `Model` | `male`, `female`, `none` / `product only`, `quiz`, or blank (AI decides) |
| `Language` | `Bangla` (default) or `English` |
| `Headline` / `Subheadline` | Optional locked ad text (reproduced exactly) |
| `Quiz or Normal picture` | Tells the caption writer which style to use |
| `Status` | Empty = pending, `Generated`, `Uploaded` |
| `Generated caption` | Filled by the workflow |
| `Generated image link` | Filled by the workflow |
| `Post Link` | Filled by the workflow |
| `Date & Time` | Filled by the workflow (Asia/Dhaka) |

Only rows with an empty `Status` are picked up, one per run.

## Prompt design highlights

- **Reference fidelity:** the logo and product packaging must be reproduced exactly, with no redesign or recolor.
- **Text style variety:** six headline styles (red label, clean bold, brush-stroke, ribbon, two-tone, stacked) so the feed does not look repetitive.
- **Text lock and fallback:** locked text is reproduced character by character. If the model cannot render it accurately, it leaves the area empty instead of producing gibberish.
- **Health-product safety:** no cure or guarantee claims, no graphic imagery, no diagnosis-style questions, and a doctor-advice note for pain topics.
- **Quiz mode:** a clean quiz card with exactly 4 options, a raw-material photo (no packaging), and no hint at the correct answer.

## Tech stack

- n8n (self-hosted)
- OpenAI (image edit + text generation)
- Google Sheets and Google Drive
- Facebook Graph API (v21.0, `/{page-id}/photos`)
- Telegram Bot (error alerts)

## Setup

1. Import `workflow.json` into n8n.
2. Create credentials in n8n for **OpenAI**, **Google Sheets**, **Google Drive** and **Telegram**.
3. Update these values to match your own accounts:
   - Google Sheet ID and sheet tab
   - Drive file ID of the logo and the folder ID for generated images
   - Facebook Page ID in the HTTP Request URL
   - Telegram chat ID and your n8n execution URL in the alert nodes
   - Phone numbers and website in the prompts and the caption footer
4. Store your Facebook Page access token in an n8n credential (for example Header Auth or a Generic credential), **not** inline in the node.
5. Put your product photos in Drive, named so that a search on `Product type` finds them.
6. Add rows to the sheet and activate the workflow.

## Security notes

- Never commit access tokens, API keys or credential files to the repository.
- Use a long-lived Page access token with only the permissions it needs, and rotate it periodically.

## Project status

Running in production.

## Author

**Prinom Mojumder**, AI/ML Automation Engineer
