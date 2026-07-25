# Meridian Home Lending — Predictive Engagement demo site

A single-file, premium-looking mortgage-lender landing page built to demo **Genesys Predictive Engagement digital user tracking**. The lead form is the demo instrument: when a visitor submits it, their name shows up in **Live Now** instead of "Unknown".

Everything is in `index.html`. No build step, no dependencies.

---

## 1. The only two things you must edit

Open `index.html` in Visual Studio and find the block near the top marked
`GENESYS PREDICTIVE ENGAGEMENT`. Edit two values:

1. **Org ID** — replace `PASTE-YOUR-ORG-ID-HERE` with your organization's ID (a GUID).
2. **region** — set it to your org's region (e.g. `use1`, `use2`, `euw1`, `euc1`, `aps1`, `apne2`).

**Where to get them:** Genesys Cloud → **Admin ▸ Predictive Engagement ▸ Global Settings ▸ Tracking Snippet**. The snippet Genesys generates there already has the correct script URL, Org ID, and region baked in.

> Safest approach: paste your org's ready-made snippet over the one in the file, then keep the two lines marked **★ NAME CAPTURE** (the `globalTraitsMapper` array and the `ac('forms:track', ...)` call). Those two lines are what make the name appear.

---

## 2. Why your name showed as "Unknown" last time — and how this fixes it

Predictive Engagement shows **Unknown** in Live Now whenever it can't tie the visitor to identity traits. Three things have to be true for the name to appear. The old page was almost certainly missing one:

**a) The form fields must map to real Genesys identity traits.**
There is **no trait literally called `name` or `fullName`.** Identity is built from `givenName`, `familyName`, and `email` (plus phone traits like `cellPhone`). If your previous form had a single `<input name="fullName">` or `name="name">`, PE captured the text but had nothing to bind identity to — so you stayed "Unknown."
This site names its inputs `givenName`, `familyName`, `email`, `cellPhone`, and also declares a `globalTraitsMapper` so the mapping is explicit.

**b) Every input needs a proper `name` attribute.**
Genesys forms tracking only captures fields that have a `name`. All inputs here have one.

**c) The form must actually be tracked, and the visitor must submit it.**
`ac('forms:track', '#rate-quote-form', { captureFormDataOnSubmit: true })` binds tracking to the form. Until a visitor **submits**, Live Now will correctly show "Unknown" — that's expected. The name populates on submit.

Also worth checking: password, hidden, and "sensitive" fields (anything matching card/ssn/cvv/etc.) are never tracked by design.

---

## 3. Deploy to GitHub Pages

1. Create a new GitHub repository (e.g. `pe-demo-site`).
2. Add `index.html` at the **repo root** and commit/push. (In Visual Studio: Git Changes → Commit → Push.)
3. In the repo on GitHub: **Settings ▸ Pages**.
4. Under **Build and deployment**, set Source = **Deploy from a branch**, Branch = **main**, folder = **/ (root)**. Save.
5. Wait ~1 minute. Your site goes live at `https://<your-username>.github.io/<repo-name>/`.

---

## 4. Point Predictive Engagement at your domain

Add your GitHub Pages domain to the allowed list, or PE won't track it:

**Admin ▸ Predictive Engagement ▸ Global Settings ▸ Tracking Settings ▸ Allowed domains** → add `<your-username>.github.io`.

Tip from Genesys: start with just the allowed domain, confirm tracking works in Live Now, then configure the rest of the tracking settings.

---

## 5. Verify in Live Now

1. Open your live GitHub Pages URL in a normal browser tab.
2. In Genesys Cloud, open **Live Now** (Performance ▸ Workspace ▸ Live Now, or the Journey view). Live Now refreshes every few seconds.
3. You'll appear as a visitor. **Before** you submit the form, your name is "Unknown" — expected.
4. Fill in first name, last name, email, phone and click **See my rate**.
5. Within a few seconds your session should update with your **name** (from `givenName` + `familyName`) and email.

If it still says Unknown after submitting, check in this order: (1) Org ID/region correct, (2) your domain is in Allowed domains, (3) the field `name` attributes weren't changed, (4) you actually clicked submit, (5) look for the tracking request in the browser DevTools ▸ Network tab.

---

## 6. Notes

- **GDPR / consent:** For a real site you must collect tracking consent (a banner/dialog) before Predictive Engagement tracks visitors. This demo omits it for simplicity.
- **Legacy vs Web Messaging:** This page uses the **Predictive Engagement tracking snippet (Journey JavaScript SDK)** — the classic approach that populates Live Now and where traits mapping decides name-vs-Unknown. Genesys is steering Genesys Cloud CX customers toward **Web Messaging / Messenger**. If your org is set up that way, the identity concept is identical but you'd deploy Messenger and map traits via the Journey plugin instead of the tracking snippet. Ask and I'll wire that variant.
- **Fictional content:** "Meridian Home Lending," the rates, and NMLS #000000 are all placeholders for a demo — not a real lender or an offer to lend.
