# Secret Santa

A small web app for running a family Secret Santa: wish lists, invite links and a name draw that nobody can peek at. It runs at [gifts.timdayley.com](https://gifts.timdayley.com).

## What it does

- **Sign in by email.** No passwords; you get a link in your inbox.
- **Exchanges.** Start one with a name and, if you like, a budget and date, then share its invite link. The organizer can edit the details later. People can join until names are drawn.
- **Wish lists.** Add items with a link, a price, a note and a photo. Pasting a link fills in the title, picture and price when the shop's page allows it.
- **The draw.** Names are drawn inside the database, so nobody using the app, the organizer included, ever sees the full list. The organizer can rule out pairs who shouldn't draw each other. A draw needs at least three people.
- **Claims.** Mark an item as "I'm getting this" so two people don't buy the same thing. The person whose list it is can't see claims.
- **Email.** Everyone is emailed when names are drawn. If the exchange has a date, a reminder goes out about a week before: to everyone, or only to the organizer if names haven't been drawn yet. The emails never say who anyone is buying for.
- **Install to the home screen.** It works as an app on phones.

## Built with

Next.js (App Router) and React, with Supabase for the database, sign-in, access rules and photo storage. Supabase sends the sign-in emails; the app's own two emails go through Resend. It is hosted on AWS Amplify.

## Run it

Needs Node 20.9 or newer, and a Supabase project (see [Set up Supabase](#set-up-supabase)).

1. Create `.env.local` in the repo root:

   ```bash
   NEXT_PUBLIC_SUPABASE_URL=https://<your-project>.supabase.co
   NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=sb_publishable_...
   ```

2. Install and start:

   ```bash
   npm install
   npm run dev
   ```

3. Open <http://localhost:3000/gifts>.

Locally the app lives under `/gifts`. On the live site it sits at the top level of its own subdomain; see [Addresses](#addresses). `http://localhost:3000/` shows the old timdayley.com home page, which is still in this repo; ignore it.

## Settings

| Name | Needed | What it is for |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Always | The Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | Always | The Supabase publishable key. Public by design; the access rules protect the data |
| `GIFTS_HOST` | Live site | The hostname the app answers on at the top level, `gifts.timdayley.com`. Hostname only, no `https://`. Also used for links in emails |
| `SUPABASE_SECRET_KEY` | For email | Looks up members' email addresses. Bypasses the access rules, so never give it a `NEXT_PUBLIC_` name |
| `RESEND_API_KEY` | For email | Sends the draw and reminder emails |
| `CRON_SECRET` | For reminders | The daily reminder job must present it |
| `EMAIL_FROM` | Optional | Sender address. Defaults to `Secret Santa <gifts@timdayley.com>` |
| `SITE_URL` | Optional | Base address for links in emails when `GIFTS_HOST` is not set. Defaults to `https://timdayley.com` |
| `GIFTS_REDIRECT_OLD` | Optional | Set to `1` to forward `/gifts/...` on any other host that reaches this app to `GIFTS_HOST`. Not needed for timdayley.com itself, which is a separate site with its own redirects |

Without the email settings the app sends no draw or reminder emails; everything else, sign-in included, still works.

Two more exist for local testing only and must never be set on the live site: `RESEND_API_URL` and `LINK_PREVIEW_ALLOW_LOCAL`.

## Set up Supabase

[`supabase/README.md`](supabase/README.md) has the full steps: applying the migrations, the sign-in email template, photo storage, email and the daily reminder job.

Parts of it predate the move to gifts.timdayley.com. Use `https://gifts.timdayley.com` wherever it says `https://timdayley.com`, and skip the two-stage subdomain section.

## Checks

```bash
npm run lint             # ESLint
supabase/tests/run.sh    # access rules and the draw; needs createdb, psql and dropdb
```

There is no separate typecheck or test script; `npm run build` checks the types.

## Deploy

Amplify builds and deploys every push to `main`, using [`amplify.yml`](amplify.yml).

- Settings go in Amplify under Hosting → Environment variables. Redeploy after changing one.
- Server-only settings reach the running site only if they are named in the `grep` line in `amplify.yml`. Add any new one there.
- The daily reminder is a GitHub Action, [`gift-reminders.yml`](.github/workflows/gift-reminders.yml). It needs `CRON_SECRET` in the repo's Actions secrets, and skips quietly without it.
- GitHub pauses scheduled workflows after 60 days with no activity in the repo. Re-enable it under the Actions tab each November.

## Addresses

The pages live in `app/gifts/`. [`proxy.ts`](proxy.ts) runs before every page request and, for the app's pages:

- maps short addresses such as `/wishlist` onto those pages when the request arrives on `GIFTS_HOST`, and forwards `/gifts/...` addresses on that host to the short form,
- keeps the sign-in session fresh,
- sends signed-out visitors to the sign-in page.

Every link in the app is written with `at(base, '/wishlist')` from [`lib/gifts-path.ts`](lib/gifts-path.ts), so it comes out right on either kind of address. `base` comes from `giftsBase()` in `lib/gifts-base.ts` on the server and `useGiftsPath()` in `components/gifts/GiftsBase.tsx` in the browser.

## How it is put together

| Path | What it holds |
|---|---|
| `app/gifts/` | The pages and the actions behind their forms |
| `app/auth/confirm/` | Where the sign-in email's link lands |
| `app/api/gifts/reminders/` | The endpoint the daily reminder job calls |
| `components/gifts/` | Shared pieces of the pages |
| `lib/` | Email, link previews, photos, address helpers and Supabase clients |
| `proxy.ts` | Addresses, sessions and the sign-in check |
| `public/` | The web manifest and app icons |
| `scripts/copy-semantic.mjs` | Copies Semantic UI into `public/vendor/` before `dev` and `build`; the pages use its icons |
| `supabase/migrations/` | The database: tables, access rules and the draw |
| `supabase/tests/` | Tests for the access rules and the draw |
| `app/page.tsx`, `components/*.tsx` | The old timdayley.com home page. Not reachable on the live site, but still shown at `/` when running locally. timdayley.com is now a separate site in its own repo |
