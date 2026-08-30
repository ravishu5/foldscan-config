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

Editing this file changes the app's behaviour within about half a day. Validation lives in
`src/premium/remoteConfig.ts` in the app repository — a change that fails it is ignored rather
than applied, so a mistake here is a no-op and not an outage.
