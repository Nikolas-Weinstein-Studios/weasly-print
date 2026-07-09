# weasly-print

Public host for the **Weasly** label-print page.

`print.html` prints asset labels from the [Weasly](https://github.com/Nikolas-Weinstein-Studios/Weasly)
inventory app to a NIIMBOT printer (M2 / M2-H / B1 family) over **Web Bluetooth**.

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

## Editing / testing

This is the canonical copy of `print.html` — edit it here, not in the Weasly repo.
See **LABEL-PRINTING.md** in the Weasly repo for device-test notes (M2 vs M2-H protocol,
print-task name, orientation).
