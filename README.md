# foldscan-config

Runtime settings for the [Foldscan](https://play.google.com/store) Android app, served as a
static file from GitHub Pages.

**This repository contains no user data and collects none.** The app fetches `config.json` with a
plain `GET` — no request body, no identifiers, no cookies, no credentials. Nothing about the
person making the request is sent, and nothing is logged back to us. It is the same shape of
request as downloading a file from a website.

## Why it exists

The rules below were compiled into the app, which meant correcting any of them required a store
release, a review queue, and a wait for people to update. Worse, an install that never updates
would have been frozen on whatever shipped with it — so the users most in need of a corrected
trial date were exactly the ones who could never receive one.

## What the app does with it

- Fetched at most twice a day, never on a schedule the user can feel.
- **Fails closed.** If this file is missing, unreachable, malformed, or fails validation, the app
  uses the values compiled into the binary and behaves exactly as it shipped.
- **Cannot revoke a purchase.** A purchase is checked before any of this is read. Nothing that
  can be written here will take away what somebody paid for.
- A config that leaves no free light theme or no free dark theme is rejected, because that would
  put dark mode behind a purchase.

## Fields

| Field | Meaning |
|---|---|
| `trialEnds` | ISO 8601. When the launch trial ends and the paid tier begins. |
| `premiumTools` | Tool ids that need Pro. Anything absent is free. |
| `freeThemes` | Theme presets available without Pro. Must include at least one light and one dark. |
| `adsEnabled` | Whether the app may offer a rewarded ad in place of a purchase. |
| `passMinutes` | How long one rewarded ad unlocks a tool for. 5–1440. |
| `minVersionCode` | Oldest Android `versionCode` allowed to run. `0` means no minimum. |

### `adsEnabled` does not install an ad network

Setting it to `true` only permits the app to *offer* a rewarded ad where one is already
available. No advertising SDK ships in the app, so with nothing able to serve an ad the offer
never appears. This field is the kill switch for after an SDK exists — a way to withdraw ads
everywhere without waiting for a store release — not the switch that turns them on.

### `minVersionCode` blocks people out of their own documents

Scans live in the app's private storage, so a blocked build is not an inconvenience: it is
somebody's records behind a door they cannot open. Leave it at `0` unless an old build would
actively corrupt data by continuing. Use the store's own update prompt for everything else.

The app fails **open** here — an absent, zero, negative or non-numeric value, or a build whose
own version cannot be read, all mean "do not block".

Editing this file changes the app's behaviour within about half a day. Validation lives in
`src/premium/remoteConfig.ts` in the app repository — a change that fails it is ignored rather
than applied, so a mistake here is a no-op and not an outage.
