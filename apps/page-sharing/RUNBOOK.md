# Screenshot runbook — Page Sharing docs

> **Where this lives (since 2026-06-29).** The published docs are in the **cloudscript.io site repo** at
> `cloudscript.io/apps/page-sharing/` (this folder), served at `https://www.cloudscript.io/apps/page-sharing`.
> The screenshots are produced by driving the **Page Sharing dev app** (the sibling `page-sharing/` repo —
> Forge install + the three Playwright-MCP browsers). It's a **cross-repo** refresh: drive the dev app from
> the `page-sharing/` side, then land the PNGs here. If the Playwright file-access roots block writing
> straight into `cloudscript.io/`, capture into a working dir first (e.g. `page-sharing/.shots/`) and `cp`
> the finished PNGs into `cloudscript.io/apps/page-sharing/`.

The screenshots in [`index.html`](./index.html) are **colocated in this folder**
(`cloudscript.io/apps/page-sharing/*.png`, next to `index.html`) and are **regenerated on demand** (not in CI).
When a documented UI surface changes, re-run the relevant entries below and overwrite the PNG in
place — the filenames are stable, so `index.html`'s `<img src>` never changes.

> **Output locations — Playwright defaults to the repo root; always pass an explicit path.**
> Screenshots → `cloudscript.io/apps/page-sharing/<name>.png` (colocated with `index.html`).
> Accessibility snapshots (`browser_snapshot` `.yml`) → a scratch dir (e.g. `/tmp`), not the site repo —
> they're intermediate capture aids, never published.

## Tools

Driven through the three Playwright-MCP browsers already wired for the cross-instance E2E lane. Each
is a Chrome debug profile signed into a different account:

| Browser | Account / role | Site (install) |
|---|---|---|
| `playwright-admin` | site administrator on the **producer** | `cloudscript.atlassian.net` (or current producer dev install) |
| `playwright-user`  | non-admin editor on the **subscriber** | `cloudscript-forge.atlassian.net` (or current consumer install) |

> `playwright-space` exists for the E2E lane (a second space-admin account) but is not needed for
> these captures.

Capture with `browser_take_screenshot`, passing `filename: "cloudscript.io/apps/page-sharing/<name>.png"`. UI Kit
surfaces (modals, admin tabs, byline dialog) render in an iframe — `browser_snapshot` pierces it, so
snapshot first to get the element `ref` (save the snapshot to `a scratch dir (e.g. /tmp) <name>.yml`), then screenshot
the dialog/region (pass `element` + `target` for a tight crop, or full-page for context).

## Prerequisites

Both installs must be live, and **a full share→approve→sync flow (E2E scenario S1) must have run**, so
that:

- the producer has at least one **shared page** with an **active subscriber** (needed for `admin-active`),
- the subscriber site has a **synced read-only replica** (needed for `subscriber-replica` / `subscriber-byline`),
- the **Allowed instances** list is non-empty (needed for `admin-allowlist`).

To stage a **pending** request for `admin-pending` **without disturbing the working subscription**: the
producer's own cloudId is not in its allow-list, so subscribe to the producer's own share *from the
producer site itself* (a fresh share code, the `⬇️ Subscribe` action on any producer page). It lands as
a pending request from the producer's cloudId. Capture it, then **Deny** to clean up — no allow-list or
synced page is touched. (Denying leaves a harmless "unavailable" ref in the producer's own Subscribed
Pages.) Avoid removing the real consumer from Allowed — that can disrupt the live demo subscription.

**Label caveat:** these screenshots were captured from the **development/staging** install, so menu
items and lozenges read e.g. "Share Page Externally **(Staging)**" / "Read-only replica **(Staging)**".
Recapture from a production install for unsuffixed labels.

## Captures

Save every screenshot into `cloudscript.io/apps/page-sharing/` (colocated with `index.html`).

### Publisher — `playwright-admin` (producer)

| File | Surface | How to reach it |
|---|---|---|
| `publisher-menu.png` | The `•••` action menu | Open a shareable page → page header **More actions (•••)** → **Apps** submenu → snapshot full page so the portal menu is visible → screenshot the menu showing **⬆️ Share Page Externally**. |
| `publisher-share.png` | Share dialog with the minted link | Click **⬆️ Share Page Externally** → in the Forge modal click **Approve & generate link** → wait for the link/textarea → screenshot the dialog. |

### Admin — `playwright-admin` (producer)

Go to **Confluence Settings → Apps → Page Sharing Admin**. Tabs are **Active / Instances / Pending /
Links / Disabled**. The Forge admin app cold-starts slowly — wait for tab text before reading. Tag the
inner app container (the first ancestor of the `Page Sharing Admin` `h1` that also holds the `Active (n)`
tab button, ~816px wide) and screenshot that, so the shot excludes the outer "(Staging)" chrome heading.

| File | Surface | How to reach it |
|---|---|---|
| `admin-active.png` | **Active** tab | Default tab. Screenshot a shared page (lozenge **SHARED**) with its subscriber table (Subscriber / Shared by / Subscribed / Last publish) + **Stop sharing**. One clean subscriber row is best — extra rows are test residue. |
| `admin-allowlist.png` | **Instances** tab | Click **Instances (n)** (the renamed allow-list). Screenshot the cloudId / Label / Approved-by / Pages table and the **Add** form. |
| `admin-pending.png` | **Pending** tab | Click **Pending (n)**. Screenshot the request row (Requesting instance / Page / **Approve** · **Deny**). Stage per Prerequisites, then **Deny** to restore. |
| `admin-links.png` | **Links** tab | Click **Links (n)**. Screenshot the reusable-links table (Page / Link / Created by / Created / Uses / Last used / **Revoke**). Trim to ~3 rows for legibility (hide extra `tr`s via DOM). |

### Subscriber — `playwright-user` (consumer)

| File | Surface | How to reach it |
|---|---|---|
| `subscriber-menu.png` | The `•••` action menu | Open the intended parent page → **More actions (•••)** → **Apps** submenu → snapshot full page → screenshot the menu showing **⬇️ Subscribe to External Page**. |
| `subscriber-modal.png` | Subscribe dialog | Click **⬇️ Subscribe to External Page** → screenshot the modal with the paste field + **Subscribe** button (capture *before* submitting). |
| `subscriber-replica.png` | A synced replica page | Open the synced page (from **Subscribed Pages**) → tag the content column (the ancestor of the `h1` holding both the title **and** body text like `PS-1` / `Out of scope`) and screenshot it, so the shot includes the **Read-only replica** byline lozenge + the synced body. |
| `subscriber-byline.png` | Byline dialog | Click the **Read-only replica** byline lozenge → screenshot the `[data-testid="forge-action-popup"]` popup naming the source page. |
| `subscriber-list.png` | **Subscribed Pages** | Confluence left nav → **Apps → Subscribed Pages** → screenshot the cloud ID banner + the subscription status list. |

> **Gotchas (learned 2026-06-28):**
> - **Test-residue rows.** After heavy testing the consumer's **Subscribed Pages** can show several stale
>   `UNAVAILABLE` rows and the admin **Active** tab a duplicate subscriber. They don't auto-purge. For a
>   clean shot, hide the stale rows in the live DOM before capturing (`display:none` / `visibility:hidden`)
>   — local-only, non-destructive. Don't rely on **Disable** (a live mutation, and the safety layer blocks it
>   unprompted).
> - **Zombie subscriptions won't recover from a push.** A push only *updates* an existing replica; it won't
>   recreate one that's missing. If a subscription is stuck `UNAVAILABLE` (e.g. pre-fix residue), make a fresh
>   **subscribe** to get a working replica rather than re-pushing. (The earlier "title edit trashes the replica"
>   report was the pre-fix on-subscribe race — fixed in `fix-on-subscribe-push-race`; healthy replicas now
>   update their title in place.)
> - **Degraded browser captures.** If `browser_take_screenshot` times out ("element not stable" / hangs after
>   "fonts loaded"), that browser session is degraded — full-viewport and tall-element captures fail while
>   small element captures still work. `cs-admin` is a member on **both** sites, so the reliable
>   `playwright-admin` browser can navigate to the consumer replica/pages and capture the subscriber shots
>   too. Restarting the Chrome profiles also clears it.

## After capturing

1. Confirm the 11 `*.png` files sit next to `index.html` in `cloudscript.io/apps/page-sharing/`, each matching an
   `<img src>` (e.g. `<img src="publisher-menu.png">`).
2. Open `index.html` locally and eyeball each section.
3. Commit the changed PNGs with a one-line note of what UI changed. The `a scratch dir (e.g. /tmp) *.yml` snapshots are
   intermediate aids — keep or discard, but don't reference them from the page.
