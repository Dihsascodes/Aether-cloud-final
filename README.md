# Aether Cloud

Frontend prototype for **Aether Cloud**, a private cloud storage service.

This is a **frontend-only build**. No storage server, database or backend is connected yet.
All data is demo data in your browser, and uploads stay in the open tab. That makes it safe
to open, explore and test before the server is purchased.

---

## Run it locally

You need **Node.js 20 or newer** ([nodejs.org](https://nodejs.org) — choose the LTS download).

1. Open a terminal in this folder.
2. Install dependencies (first time only):

   ```bash
   npm install
   ```

3. Start the development server:

   ```bash
   npm run dev
   ```

4. Open **http://localhost:5173** in your browser.

Press `Ctrl + C` in the terminal to stop the server.

### Signing in

On the **Sign in** page, use the two demo buttons:

| Button       | Opens                                                      |
| ------------ | ---------------------------------------------------------- |
| **Customer** | Customer dashboard, file manager and settings (`/app`)     |
| **Admin**    | Admin console with user management (`/admin`)              |

You can also sign in with any email and a password of 6+ characters, or create an account
on the registration page. Accounts exist only in your browser's local storage.

### Other commands

```bash
npm run build     # type-check and build a production bundle into dist/
npm run preview   # serve the built bundle locally (http://localhost:4173)
npm test          # run unit tests (pricing, formatting, file sorting)
```

The dev and preview servers listen on `127.0.0.1` only, so the site isn't exposed to your network.

---

## What's included

| Area                 | Route(s)                                    |
| -------------------- | ------------------------------------------- |
| Landing page         | `/`                                         |
| Pricing              | `/pricing`                                  |
| Sign in / Register / Forgot password | `/login`, `/register`, `/forgot-password` |
| Customer dashboard   | `/app`                                      |
| File manager         | `/app/files`, `/app/files/:folderId`, `/app/recent`, `/app/starred`, `/app/shared`, `/app/trash` |
| Account settings     | `/app/settings` (Profile, Account, Security, Plan & billing, Notifications) |
| Admin console        | `/admin`, `/admin/users`                    |
| Legal placeholders   | `/privacy`, `/terms`                        |

What works in the demo:
- **File manager:** folder navigation, search, sort, type filters, list and grid views, multi-select (checkbox, Ctrl/⌘-click, Shift-click), rename, star, trash with undo, restore, delete forever, new folders and a details panel.
- **Uploads:** pick or drag-and-drop real files. They are held in the open browser tab only, and you can download them again during that session.
- **Sample files:** these have no real content. Downloading one explains that downloads arrive with the server.
- **Share dialog:** invite by email, set view/edit access and expiry, and copy a preview link. The link does not work for anyone yet.
- **Settings:** profile edits, notification preferences, 2FA toggle, session sign-out, and plan changes. Plan changes update the storage quota everywhere.
- **Admin:** user search, filters, sorting, pagination, a user detail drawer, suspend/reactivate, plan change, delete and invite.

Keyboard shortcuts: `⌘K` / `Ctrl+K` or `/` focuses search · `Esc` clears selection · `⌘A` / `Ctrl+A` selects all · `Delete` moves the selection to trash.

Demo data resets when you reload the page. Your sign-in and notification preferences persist.

---

## Changing prices

All prices live in one file: **`src/config/pricing.ts`**.

```ts
{ id: '50gb', name: 'Personal', storageGB: 50, price: { monthly: 260, yearly: 2600 }, ... }
```

Edit `monthly` / `yearly` (whole LKR). The landing page, pricing page, comparison table,
registration, settings and admin revenue figures all update automatically. Savings,
"months free" and cost-per-GB are calculated, not typed.

`src/config/pricing.test.ts` locks in the confirmed price list. If you change a price on
purpose, update the table in that test too, then run `npm test`.

Plan names (Essential, Personal, Plus, Studio, Vault), taglines and the shared feature list
are placeholders in the same file. Rename them freely.

---

## The logo

- `public/brand/aether-logo.png` is the official logo file, unchanged. It is also the favicon.
- `public/brand/aether-logo-transparent.png` is a lossless copy with the black background made
  transparent, so the mark sits cleanly on any dark surface. On black it is identical to the
  original (verified pixel-by-pixel by `scripts/make-transparent-logo.py`).
- `src/components/brand/Logo.tsx` is the only component that renders the logo. It crops the
  empty margin around the mark without changing its proportions.

If you replace the logo file, re-run `python3 scripts/make-transparent-logo.py` (needs Pillow).

---

## Project structure

```
src/
  config/        pricing.ts (prices), site.ts (nav, company info), faq.ts (FAQ copy)
  services/      ← the ONLY boundary between UI and data
    index.ts     service interfaces + which implementation is active
    types.ts     shared data types (files, users, usage, activity…)
    mock/        in-browser mock implementations + demo data
  pages/         marketing/, auth/, app/ (customer), admin/
  layouts/       MarketingLayout, AuthLayout, AppLayout (sidebar + top bar)
  components/
    brand/       Logo, opening animation
    ui/          Button, Input, Switch, Modal, Drawer, Menu, Tabs, Badge…
    motion/      Reveal, KineticHeading, page transitions, CountUp, Magnetic…
    charts/      AreaChart, BarList, UsageRing, Sparkline
    marketing/   Navbar, Footer, pricing cards, landing sections
    app/         Sidebar, Topbar, upload dialog, file manager pieces
  context/       Auth, Toasts, Uploads
  hooks/ lib/    small utilities (formatting, sorting, file types)
```

**Stack:** React 19 · TypeScript · Vite · Tailwind CSS v4 · React Router · Motion (animation) ·
Lucide icons · Geist typeface (self-hosted, no external requests).

---

## Connecting the real backend later

Pages and components never read data directly. Everything goes through three interfaces
in `src/services/index.ts`:

- `AuthService`: sign in/up, password reset, profile, sessions, notification preferences
- `StorageService`: list, search, upload, download, share, rename, trash, usage, activity
- `AdminService`: users, statistics, user management

To connect the server:

1. Create e.g. `src/services/http/httpStorageService.ts` implementing `StorageService`, calling your API.
2. Change the assignment at the bottom of `src/services/index.ts`:
   ```ts
   export const storageService: StorageService = httpStorageService
   ```
3. Set `IS_PROTOTYPE = false` in the same file once nothing is mocked.

Do the same for auth and admin. The types in `src/services/types.ts` describe exactly what the
UI expects back. Keep API URLs in a `.env` file (already git-ignored), never in source code.

### Before going public

This build is a private prototype. Review these before launch:

- **Marketing copy:** the security claims (encryption at rest, redundant storage, 2FA, session control) should match what the server actually provides.
- **Contact details and legal pages:** the support email (`src/config/site.ts`) is a placeholder, and the Privacy/Terms pages are placeholders.
- **Search engines:** `index.html` sets `noindex, nofollow`. Remove it when you want the site to be found.
