# Cam Gallery — S3 video/photo viewer

A single static page (`index.html`) that lists your camera clips and snapshots
from S3, filtered by time range, and plays them in the browser. Hostable on
GitHub Pages. Your S3 bucket stays **private** — the page talks to a small
read-only Lambda that returns short-lived presigned URLs.

```
Browser (GitHub Pages)  ──GET ?from&to&token──▶  cam-viewer Lambda ──▶ S3 (private)
        ▲                                                 │
        └────────── JSON: [{time, type, url(presigned)}] ─┘
```

## 1. Deploy the listing API (once)

In **AWS CloudShell** (same account/region as the bucket):

```bash
# upload lambda/viewer_lambda.py and lambda/deploy_viewer.sh into CloudShell, then:
bash deploy_viewer.sh
```

It prints an **API URL** and a **Token** at the end. Copy both.
(Re-running updates the code and keeps the same token.)

## 2. Publish the page

Push `web/index.html` to a GitHub repo and enable **Settings → Pages → Deploy
from branch** (root or `/web`). Or just open `index.html` locally — it works
from `file://` too.

## 3. Connect

Open the page → click **⚙ (Settings)** → paste the **API URL** and **Token** →
Save. They're stored only in your browser (localStorage), never committed.

## Using it

- Pick a **From/To** range or a preset chip (Last hour / 24 h / Today / 7 days / All).
- Filter by **Type** (video / photo).
- Click any tile to play/enlarge; use **⬇** to download.
- Times shown are IST, matching the camera OSD and the `clip-YYYYMMDD-HHMMSS` filenames.

## Notes

- The Function URL is public but guarded by the unguessable token, and the role
  is **list + read only** — it cannot modify or delete anything.
- Presigned URLs default to a 1-hour lifetime (`expires` query param, max 7 days).
- To rotate the token: see the comment at the top of `deploy_viewer.sh`.
