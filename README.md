# Punk Portal — Vercel Deploy Folder

This is a self-contained deploy folder for the Punk Portal. Drop it on Vercel and you're live.

## What's inside

| File | Purpose |
|------|---------|
| `index.html` | The Portal — single-file HTML/CSS/JS, including the embedded PunkGrid wallet |
| `vercel.json` | Security headers + cache rules (X-Frame-Options, HSTS, no-cache on HTML) |
| `package.json` | Minimal stub so Vercel recognises it as a project (no build step) |
| `.vercelignore` | Keeps junk out of the deploy bundle |

## First-time setup (one time only)

You need the Vercel CLI installed and to be logged in. If you've already deployed PunkSwap from this same machine, you can skip both:

```cmd
npm install -g vercel
vercel login
```

Then from this `portal-deploy` folder, link it to a Vercel project:

```cmd
cd path\to\portal-deploy
vercel link
```

Vercel will ask:
- **Set up and deploy?** → `Y`
- **Which scope?** → your account
- **Link to existing project?** → `N` (create new)
- **Project name?** → `punk-portal` (or whatever you want)
- **Directory?** → `./` (current folder)

This creates a hidden `.vercel/` folder that links this directory to the Vercel project. **Do not commit `.vercel/` to git or share it** — it contains project credentials.

## Every deploy after that

One command from inside this folder:

```cmd
vercel --prod --force
```

`--prod` deploys to the production URL (not a preview). `--force` skips Vercel's cache so the new HTML is actually served, which matters because the Portal is large and Vercel sometimes deduplicates aggressively.

When it finishes, Vercel prints the production URL. The first time, it'll look like `https://punk-portal-xxxxx.vercel.app`. You can add a custom domain in the dashboard later.

## Updating the Portal HTML

When Claude (or you) generates a new `Punk_Portal_Live.html`, copy it into this folder as `index.html`:

```cmd
copy "C:\path\to\Punk_Portal_Live.html" "C:\path\to\portal-deploy\index.html" /Y
cd "C:\path\to\portal-deploy"
vercel --prod --force
```

You can also chain it on one line:

```cmd
copy "C:\Users\wilso\Downloads\Punk_Portal_Live.html" "C:\Users\wilso\firebase-indexes\escrow-backend\portal-deploy\index.html" /Y && cd "C:\Users\wilso\firebase-indexes\escrow-backend\portal-deploy" && vercel --prod --force
```

(Same shape as your PunkSwap deploy line.)

## Custom domain (when ready)

1. Vercel dashboard → `punk-portal` project → Settings → Domains
2. Click **Add Domain**, enter e.g. `portal.punkswap.app` (or whatever you want)
3. Vercel shows you a DNS record to add at your domain registrar (CNAME for subdomains, A record for apex)
4. Add the record at your registrar; takes 5–30 minutes to propagate
5. Once Vercel says it's verified, the site is live on that domain

After the custom domain is live, update the `PUNKSWAP_URL` constant inside `index.html` (search for `PUNKSWAP_URL = `) to point at the matching PunkSwap domain. That powers the "View on PunkSwap" buttons in the Portal.

## Security headers explained

`vercel.json` sets these for every response:

- **X-Frame-Options: DENY** — prevents the Portal from being embedded in an `<iframe>` on another site. Critical because the Portal hosts an in-page wallet; iframing would enable clickjacking attacks against seed input.
- **X-Content-Type-Options: nosniff** — stops browsers from guessing MIME types, which can sidestep CSP.
- **Referrer-Policy: strict-origin-when-cross-origin** — only sends the origin (not the full URL) when navigating to other sites, so query params can't leak.
- **Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()** — denies access to sensors and the payment API. The Portal doesn't need them.
- **Strict-Transport-Security: max-age=31536000; includeSubDomains** — forces HTTPS for a year. Vercel issues SSL automatically.

And caching:

- `index.html` and `/` have `Cache-Control: must-revalidate` so users always check for a fresh version. This is important because the Portal updates often and you don't want a one-week-old cached copy serving after you push a hotfix.

## Troubleshooting

**"Command failed: vercel link"** — run `vercel login` first.

**Deploy succeeds but the site shows the old version** — hard-refresh with `Ctrl+Shift+R`, or open DevTools → Application → Storage → "Clear site data" → reload. Service workers can hold onto old assets.

**"Module not found" or build errors** — Vercel might be auto-detecting a framework. In the dashboard go to Settings → General → Framework Preset → set to "Other" or leave blank.

**The PunkGrid wallet tab fails to load fonts/SDK** — check the browser console. The Portal loads:
- Tailwind CSS (cdn.tailwindcss.com)
- Chart.js (cdn.jsdelivr.net)
- KeetaNet SDK (static.network.keeta.com)
- BIP-39 wordlist for wallet creation (cdn.jsdelivr.net)

If any of these CDNs is blocked (corporate firewall, Pi-hole, etc.), wallet creation will fail with a clear error message in the console.

## What's NOT in this folder

This deploy folder doesn't include:
- Firebase Cloud Functions (those deploy separately via `firebase deploy`)
- Firestore security rules (deploy via Firebase Console or `firebase deploy --only firestore:rules`)
- The KeePunks Chrome Extension (separate ZIP from the wallet integration session)

Those live in your `escrow-backend` folder and deploy independently of the Portal frontend.
