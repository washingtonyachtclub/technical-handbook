# Washington Yacht Club Technical Handbook

**Last updated:** 2026-11-27

**Contact:** Eshan Arora

This handbook documents the club's production software and infrastructure.

## Principles

- **Resilience:** Systems should remain reliable through officer turnover and for
  years with minimal recurring maintenance. Prefer managed services, automation,
  documented recovery, and clear ownership.
- **Cost:** Prefer services with generous free tiers or nonprofit discounts.

## Current systems

| System                | Platform                               | Purpose                                                                              | Links                                                                                                         |
| --------------------- | -------------------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| Public website        | EmDash and Astro on Cloudflare Workers | Public pages, photos, galleries, and navigation, edited in EmDash.                   | [Website](https://washingtonyachtclub.org/) · [Editor](https://washingtonyachtclub.org/_emdash/admin)         |
| Database              | Cloudflare Workers and D1              | Member records and club operations.                                                  | [Production](https://database.washingtonyachtclub.org/) · [Development](https://wyc-database-dev.wyc-emdash-website.workers.dev/) |
| Documents             | Public GitHub repository               | Club documents maintained in the `documents` repository.                             | [Documents](https://documents.washingtonyachtclub.org/)                                                       |
| Payments and commerce | Square                                 | Membership payments, donations, keelboat reservations, merchandise, and other sales. | [Square storefront](https://washington-yacht-club.square.site/)                                               |
| Database emails       | Resend                                 | Email sent by the database application.                                              | —                                                                                                             |
| Club email            | Google Workspace                       | Club Gmail accounts.                                                                 | —                                                                                                             |

The database application includes:

- [Boat checkout](https://database.washingtonyachtclub.org/checkout).
- [Keelboat calendar and reservations](https://database.washingtonyachtclub.org/keelboat-calendar).
- [Written tests](https://database.washingtonyachtclub.org/tests).
- [Lessons and signups](https://database.washingtonyachtclub.org/lessons).
- [Membership signup](https://database.washingtonyachtclub.org/join), renewals, payments,
  and [officer approval](https://database.washingtonyachtclub.org/membership-approvals).
- Member and [guest waivers](https://database.washingtonyachtclub.org/guest-waiver),
  with executed PDFs stored privately in Cloudflare R2.

## Hosting and accounts

### Public website

Cloudflare Workers runs the EmDash and Astro website. EmDash stores content in
Cloudflare D1 and images in R2. Editors manage pages, photos, and navigation through
EmDash without a code deployment. Cloudflare Builds deploys code changes from the
website repository's `main` branch.

### Database hosting

Cloudflare Workers Builds deploys the database application from the `main` branch
of `wyc-database-v2`. Production uses the `wyc-database-production` D1 database.

`npm run dev` runs the Worker locally. It reads and writes the same
`wyc-database-dev` D1 database as hosted development. R2 files stay local.
Sign in to Wrangler with the club Cloudflare account and Workers preview access.

Use `npm run db:migrate:dev` for shared dev schema updates. To refresh shared dev
from production, run `./sync-prod-to-dev.mjs` from `wyc-context`. The `--local`
flag refreshes only the isolated SQLite copy.

The production Worker runs daily reminders at 15:00 UTC.
[D1 Time Travel](https://developers.cloudflare.com/d1/reference/time-travel/) provides
30 days of point-in-time recovery on the club’s Workers Paid plan.

### Domains and DNS

`washingtonyachtclub.org`, `wayc.org`, and `wycsail.org` are registered with Cloudflare
and use Cloudflare DNS under `webmaster@washingtonyachtclub.org`.

Cloudflare redirects `wayc.org` and
`wycsail.org` (including `www`) to `http://www.washingtonyachtclub.org`

### Google Workspace and email

Google Workspace hosts the club's Gmail accounts. Its administrator account is
`admin2@washingtonyachtclub.org`. The main domain's incoming mail is routed to
Google.

The database application sends transactional email through Resend as
`database@mail.washingtonyachtclub.org`. The sending domain is configured through
Cloudflare DNS.

### Square

Square handles membership payments, donations, keelboat reservation payments,
merchandise, and other club sales. It also handles employee and payroll tracking for
ratings examiners.

### Cloudflare R2

The database Worker uses R2 bindings to access private buckets for signed waivers
and member profile photos.

### Account ownership

The club’s Cloudflare account uses `webmaster@washingtonyachtclub.org`. Credentials and recovery codes must be stored separately from
this public repository. Google Workspace is administered through
`admin2@washingtonyachtclub.org`; the planned shared password manager is for
non-Google services.

## To-dos

- [ ] Fully Retire DreamHost and cancel subscription
- [ ] Set up a shared password manager for non-Google service accounts:
  - [ ] Make the Commodores the vault administrators.
  - [ ] Grant position holders access only to the accounts they need.
