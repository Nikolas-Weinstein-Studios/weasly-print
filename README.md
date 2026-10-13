# weasly-print

Public host for the **Weasly** label-print page.

`print.html` prints asset labels from the [Weasly](https://github.com/Nikolas-Weinstein-Studios/Weasly)
inventory app to a NIIMBOT printer (M2 / M2-H / B1 family) over **Web Bluetooth**.

It also prints **batches of Code 128 labels** for other studio tools — currently the NYU
[spine roller](https://github.com/Nikolas-Weinstein-Studios/spine-roller), a Claude artifact
with the same sandboxed-iframe problem as Weasly. See [Batches](#batches) below.

## Why it lives in its own public repo

The Weasly app is served by Google Apps Script, which renders inside a sandboxed
cross-origin iframe where Web Bluetooth is blocked. The print page must therefore run as a
top-level page on a public origin. The Weasly repo is private on a free plan (can't publish
Pages), so this small **public** repo hosts the page instead. It contains no keys or app
code — Weasly opens it with the item's WID, name, and QR target passed as URL params.

## Live URL

Served via GitHub Pages:

```
https://nikolas-weinstein-studios.github.io/weasly-print/print.html
```

Weasly's `LABEL_PRINT_URL` constant (in its `Index.html`) points here.

## The tab stays open

Weasly opens this page **once** and then posts each subsequent label into it
(`postMessage`, `{type:'weasly-label', wid, name, url}`). It does not reopen the page per
label, and this page does not reload — a Bluetooth connection belongs to the page that
made it, so a reload means another trip through the device chooser for every single label.

- The page answers `weasly-print-ready` to its opener once its listener is attached, so a
  label requested while it was still loading is delivered rather than dropped.
- Senders are accepted only from `*.googleusercontent.com` (the Apps Script iframe's
  per-deployment host) and only in that message shape.
- There is no "back to Weasly" button. One shipped and was reverted the same day: Chromium
  does not let a page move focus to another tab, so `window.opener.focus()` did nothing at
  all on Android/Brave. A note tells the user to switch tabs themselves, which costs nothing
  — the connection belongs to this page and survives as long as the tab does. **Close this
  tab** is the explicit way out, and is what forces a reconnect.
- Where `navigator.bluetooth.getDevices()` exists (Chrome on Android) the last printer is
  remembered in `localStorage` and reconnected without the chooser even after a reload.
  Bluefy doesn't implement it, so on iOS the open tab is what does the work.

## Batches

A tool that can't reach Bluetooth itself links here with the labels in the URL fragment:

```
print.html#from=Spine+roller&labels=[{"code":"S047","title":"47","lines":["Center  L-3","R 2905    cut 1543"]}, …]
```

(URL-encoded, as `URLSearchParams` writes it.) Each label is a Code 128 barcode of `code`
across the bottom, `title` large at top left, and up to four `lines` beside it, drawn at
whatever label size is set on the page. At most 500 labels; anything else in a bad shape is
refused with a message rather than printed.

- **Print all** sends the batch as one print task over one connection. Step through with
  ‹ ›: the button then prints from the label on screen to the end, which is how a batch that
  stopped part way is resumed. **Print only this label** is for a reprint.
- A plain link has no opener, so a batch tab gets no further labels; the next batch opens a
  tab of its own. On Android Chrome the remembered printer reconnects without the chooser.
- If the tab opens with an opaque origin — a sandboxed frame that couldn't let the popup
  escape — Bluetooth can't work there, and the page says to copy its address into a new tab.

The spine roller's `npm run e2e` loads this repo's `print.html` from a sibling checkout and
scans the barcodes it draws, so run it after changing the batch layout.

## Editing / testing

This is the canonical copy of `print.html` — edit it here, not in the Weasly repo.
See **LABEL-PRINTING.md** in the Weasly repo for device-test notes (M2 vs M2-H protocol,
print-task name, orientation).
