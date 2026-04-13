# ScoreBlur License Activation Bridge

A tiny single-page HTTPS bridge that receives the Dodo Payments checkout redirect and forwards the license key to the [ScoreBlur Chrome extension](https://chromewebstore.google.com/) via Chrome's `externally_connectable` messaging API.

## Why this exists

Dodo Payments accepts an HTTPS `return_url` for post-checkout redirects. Chrome extensions live at `chrome-extension://<id>/...`, which most payment processors do not allow as a redirect destination. This bridge sits in between:

```
Dodo checkout → return_url=https://thornografi.github.io/scoreblur-license-bridge/?ext_id=<EXTENSION_ID>
              → Dodo appends &license_key=<UUID>
              → bridge JS reads both query params
              → chrome.runtime.sendMessage(extensionId, { type: 'ACTIVATE_LICENSE_FROM_BRIDGE', licenseKey })
              → ScoreBlur service worker activates the license, returns success
              → bridge shows "License activated"
```

The extension's `manifest.json` whitelists this origin via:

```json
"externally_connectable": {
  "matches": ["https://thornografi.github.io/*"]
}
```

The extension ID is passed in the `ext_id` query param so the same bridge works for every install (extension IDs differ between dev installs; production gets a stable ID from the Chrome Web Store).

## Manual fallback

If anything fails (no `license_key` in URL, extension not installed, messaging error, activation rejected by Dodo), the bridge displays the license key in a copy-able `<code>` block so the user can paste it into the extension options page manually.

## Hosting

Served via GitHub Pages from `main` branch root.

URL: https://thornografi.github.io/scoreblur-license-bridge/
