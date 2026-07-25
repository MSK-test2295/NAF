Meridian Home Lending — Predictive Engagement demo site
A single-file, dark-themed mortgage-lender site built to demo Genesys Predictive Engagement digital user tracking on Web Messaging (Messenger). It captures two things in Live Now:
Visitor identity — submitting the rate-quote form shows the visitor's name (not "Unknown").
Site searches — using the search box shows terms under "Searches performed."
Everything is in `index.html`. No build step, no dependencies.
---
1. Set your Deployment ID (do this first)
Open `index.html`, find the `GENESYS WEB MESSAGING` block near the top, and replace `YOUR-DEPLOYMENT-ID` with your Messenger Deployment ID — the same one you used before, when the name started appearing in Live Now.
Where: Genesys Cloud → Admin ▸ Message ▸ Messenger Deployments.
`environment: 'prod-usw2'` already matches your `apps.usw2.pure.cloud` region — leave it.
> This file ships with the placeholder, so if you don't paste your Deployment ID back in, nothing will track. That's the #1 thing to check if it "stops working."
---
2. Name capture (already working for you)
Identity in Live Now is built from the traits `givenName`, `familyName`, `email`, and `cellPhone` — there is no trait literally called "name." The `Journey.formsTrack` call maps the form's fields onto those traits:
```javascript
Genesys('subscribe', 'Journey.ready', function () {
  Genesys('command', 'Journey.formsTrack', {
    selector: '#rate-quote-form',
    formName: 'rate quote',
    captureFormDataOnAbandon: false,
    traitsMapper: [
      { fieldName: 'givenName',  traitName: 'givenName'  },
      { fieldName: 'familyName', traitName: 'familyName' },
      { fieldName: 'email',      traitName: 'email'      },
      { fieldName: 'cellPhone',  traitName: 'cellPhone'  }
    ]
  });
});
```
The name populates after the visitor submits the form — before that, "Unknown" is correct.
---
3. Search capture — one Genesys setting to flip (NEW)
Genesys reads searches from the URL: when the page URL carries the search term in a query parameter, it appears under "Searches performed." This site's search boxes are native `GET` forms that navigate to `?term=<what they typed>` — a real URL change Genesys can see.
You must tell Genesys which parameter holds the term:
> **Admin ▸ Predictive Engagement ▸ (Global / Tracking) Settings ▸ Site search settings → enter `term`** → Save.
(In your left nav this lives under Predictive Engagement ▸ Predictive Engagement Settings ▸ Tracking. The field is a small text box labeled "Site search settings.")
Also make sure `term` is not listed under "URL query parameters to ignore."
How to trigger a search in the demo: use the header search box, the big search box in the "Find answers fast" section, or click any of the popular-search chips / resource cards (they link to `?term=...`). Each one changes the URL and renders a results list.
After configuring `term`, do a search on the live site, then open the visitor in Live Now → the Searches panel lists the term(s).
---
4. Journey tracking on the deployment (already on)
The Journey plugin only runs if journey/Predictive Engagement tracking is enabled on the Messenger deployment. Yours already is — page views and identity are showing — so nothing to change here.
---
5. Deploy to GitHub Pages
Put `index.html` at your repo root, commit, push.
GitHub → Settings ▸ Pages → Source Deploy from a branch → main / / (root) → Save.
Live at `https://<your-username>.github.io/<repo-name>/` in ~1 minute.
If your Messenger deployment uses Restrict domain access, make sure your `github.io` domain is allowed. (It already tracks there, so you're fine.)
---
6. Verify in Live Now
Identity: open the live page → fill and submit the rate-quote form → within seconds, Name + email + mobile appear in the Customer Summary.
Search: run a search (or click a chip) → open the visitor → the Searches panel shows the term. If it stays empty, confirm Site search settings = `term`, and that `term` isn't in the ignore list.
---
7. What changed in this version
Look: rebuilt in a dark black-and-blue theme with subtle glow (Space Grotesk + Inter), glassmorphism cards, and glowing accents — professional, not flashy.
More content: added a knowledge-base search with results, an interactive monthly-payment calculator, an expanded loan-options grid, a full rates table, a "why us" feature grid, an FAQ accordion, testimonials, and resource cards — so it reads like a real lender site.
Unchanged: the Genesys snippet approach and the identity form field names, so your existing setup keeps working.
Notes
GDPR / consent: collect tracking consent before tracking in production. This demo omits it.
Guest vs authenticated: trait mapping shows the name in Live Now for a guest visitor (your goal). Linking to an External Contact record is authenticated web messaging — a larger setup.
Fictional content: "Meridian Home Lending," the rates, and NMLS #000000 are demo placeholders — not a real lender or an offer to lend.
