Meridian Home Lending — Predictive Engagement demo site (Web Messaging)
A single-file, premium-looking mortgage-lender landing page built to demo Genesys Predictive Engagement digital user tracking on Web Messaging (Messenger). The lead form is the demo instrument: when a visitor submits it, their name shows up in Live Now instead of "Unknown".
Everything is in `index.html`. No build step, no dependencies.
> **Your setup:** Your org is on **Web Messaging**, so this site uses the **Messenger bootstrap** (`genesys.min.js` / the `Genesys(...)` global) and the **Journey plugin**. It does **not** use the legacy `ac(...)` tracking snippet — those are two different, incompatible SDKs.
---
1. The one thing you must edit
Open `index.html` in Visual Studio, find the block marked `GENESYS WEB MESSAGING`, and set:
`deploymentId` — replace `YOUR-DEPLOYMENT-ID` with your Messenger Deployment ID.
`environment` — already `prod-usw2` (US West 2), which matches your `apps.usw2.pure.cloud` region. Leave it unless your region changes.
Where to get the Deployment ID: Genesys Cloud → Admin ▸ Message ▸ Messenger Deployments → open your deployment → copy the Deployment ID.
---
2. Why your name showed as "Unknown" — and how this fixes it
Two problems were stacked on top of each other:
a) SDK mismatch. The previous file had the Messenger bootstrap (`Genesys(...)`, `deploymentId`) but kept legacy `ac('forms:track', ...)` / `globalTraitsMapper` lines under it. With Messenger the global function is `Genesys` — so the `ac(...)` calls referenced a function that doesn't exist, and the loose `globalTraitsMapper: [...]` fragment was a JavaScript syntax error. None of the identity code ran. (Page views still tracked, because Messenger's Journey plugin captures those automatically once deployed — that's why Live Now showed a session but an empty name.)
b) No trait mapping. Even correctly written, identity in Live Now is built from the traits `givenName`, `familyName`, and `email` (plus phone traits like `cellPhone`). There is no trait literally called `name` or `fullName`. You have to map your form fields onto those trait names.
The fix, now in the file, is the Messenger-native command:
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
The form's inputs are already named `givenName`, `familyName`, `email`, and `cellPhone`, so they line up with the mapper. The name populates only after the visitor submits the form — before that, "Unknown" is expected and correct.
---
3. Make sure journey tracking is on for the deployment
For the Journey plugin to run at all, journey/Predictive Engagement tracking must be enabled on the Messenger deployment (Admin ▸ Message ▸ Messenger Deployments ▸ your deployment). Yours already is — your screenshots show page views in Live Now — so there's nothing to change here. Noted only for completeness.
---
4. Deploy to GitHub Pages
Create a GitHub repo (e.g. `pe-demo-site`).
Add `index.html` at the repo root, commit, and push. (Visual Studio: Git Changes → Commit → Push.)
On GitHub: Settings ▸ Pages → Source Deploy from a branch → Branch main / folder / (root) → Save.
Wait ~1 minute. Live at `https://<your-username>.github.io/<repo-name>/`.
If your Messenger deployment uses Restrict domain access, add your `<your-username>.github.io` domain to its allowed list, or Messenger won't load there. (If tracking already works on your Pages URL, the domain is fine.)
---
5. Verify in Live Now
Open your live GitHub Pages URL in a normal browser tab.
In Genesys Cloud open Live Now (Orchestration ▸ Predictive Engagement ▸ Live Now). It refreshes every few seconds.
You appear as a visitor. Before submitting the form, Name = "Unknown" — expected.
Fill first name, last name, email, phone → click See my rate.
Within a few seconds the session updates: Name = givenName + familyName, plus email/phone in the Customer Summary.
Still Unknown after submitting? Check in order: (1) real `deploymentId` set, (2) `environment` matches your region, (3) the input `name` attributes weren't changed, (4) you actually clicked submit, (5) open DevTools ▸ Console for Genesys/Journey errors, and DevTools ▸ Network to confirm the tracking request fires on submit.
---
6. Notes
GDPR / consent: In production, collect tracking consent before Predictive Engagement tracks visitors. This demo omits it for simplicity.
Guest vs authenticated: Mapping traits via `Journey.formsTrack` is enough to show the name in Live Now for a guest (unauthenticated) visitor — which is your goal. If you later want the session firmly linked to an External Contact record, that's authenticated web messaging (an OAuth identity provider), which is a larger setup.
Fictional content: "Meridian Home Lending," the sample rates, and NMLS #000000 are placeholders for a demo — not a real lender or an offer to lend.
