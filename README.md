# Washington Yacht Club Technical Handbook

**Last updated:** 2026-08-23  
**Contact:** Eshan Arora

This handbook documents the club's production software, infrastructure, and current
technical migration work.

## Principles

- **Resilience:** Systems should remain reliable through officer turnover and for
  years with minimal recurring maintenance. Prefer managed services, automation,
  documented recovery, and clear ownership.
- **Cost:** Prefer services with generous free tiers or nonprofit discounts when
  they meet the club's operational needs.

## Current systems

| System                             | Platform                       | Current status                                                                                 | Links                                                                                                                                                          |
| ---------------------------------- | ------------------------------ | ---------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Public website                     | WordPress on DreamHost         | The production marketing website.                                                              | [Website](https://washingtonyachtclub.org/)                                                                                                                    |
| Database v2                        | Vercel with MySQL on DreamHost | The current member and club-operations web application.                                        | [database.washingtonyachtclub.org](https://database.washingtonyachtclub.org/) is a CNAME for [wyc-database-v2.vercel.app](https://wyc-database-v2.vercel.app/) |
| Replacement public website         | Astro                          | Under development and not deployed.                                                            | —                                                                                                                                                              |
| Keelboat calendar and reservations | DreamHost                      | The current calendar and reservation application. It has not yet been moved into database v2.  | —                                                                                                                                                              |
| Checkout                           | Database v2 on Vercel          | The new checkout is in beta. The legacy checkout will redirect to it after the beta ends.      | [New checkout](https://database.washingtonyachtclub.org/checkout) · [Legacy checkout](https://checkout.washingtonyachtclub.org/)                               |
| Novice written tests               | WordPress                      | Being ported to database v2 and nearly ready.                                                  | [Current](https://washingtonyachtclub.org/written-tests/) · [Replacement](https://database.washingtonyachtclub.org/written-tests)                              |
| Documents                          | Public GitHub repository       | Club documents are maintained in the `documents` repository.                                   | [Documents](https://documents.washingtonyachtclub.org/)                                                                                                        |
| Payments and commerce              | Square                         | Handles membership payments, donations, keelboat reservations, merchandise, and other sales.   | [Square storefront](https://washington-yacht-club.square.site/)                                                                                                |
| Transactional email                | Resend                         | Database v2 sends club email through Resend.                                                   | —                                                                                                                                                              |
| Club email                         | Google Workspace               | Hosts the club's Gmail accounts.                                                               | —                                                                                                                                                              |
| Photos and galleries               | WordPress and Google Drive     | WordPress galleries contain photos through 2022. The club also maintains a photo Google Drive. | [Gallery](https://washingtonyachtclub.org/gallery/) · [Photo Drive](https://photos.washingtonyachtclub.org/)                                                   |

## Hosting and accounts

### Vercel

Database v2 is deployed on Vercel and connects to the production MySQL database on
DreamHost. The Vercel account uses `webmaster@washingtonyachtclub.org`.

### DreamHost

DreamHost currently provides:

- WordPress hosting for the public website;
- MySQL hosting for the production database;
- hosting for the keelboat calendar and reservation application;
- registration and authoritative DNS for `washingtonyachtclub.org`;
- DNS records for Google Workspace and Resend;
- cron jobs for database backups.

These responsibilities must all be moved or replaced before DreamHost can be
retired. Existing subdomains and email forwarding addresses must also be inventoried
during the migration.

### Google Workspace and email

Google Workspace hosts the club's Gmail accounts. Its administrator account is
`admin2@washingtonyachtclub.org`. The main domain's incoming mail is routed to
Google.

Database v2 sends transactional email through Resend as
`database@mail.washingtonyachtclub.org`. The sending domain is configured through
DreamHost DNS.

### Square

Square handles membership payments, donations, keelboat reservation payments,
merchandise, and other club sales. It also handles employee and payroll tracking for
ratings examiners.

Square is expected to remain the club's payment platform. Stripe has a better
developer experience, but splitting payments between providers would make club
operations and reporting more complicated.

### Account ownership

Many newer service accounts use `webmaster@washingtonyachtclub.org`, including
Vercel and Backblaze. Credentials and recovery codes must be stored separately from
this public repository. Google Workspace is administered through
`admin2@washingtonyachtclub.org`; the planned shared password manager is for
non-Google services.

## WordPress dependencies

Moving the public website away from WordPress also requires replacing or migrating:

- waivers and their associated workflow;
- novice written tests;
- photo storage and curated galleries;
- any remaining WordPress-managed content or integrations.

## Backups and recovery

The production systems use one MySQL database. It is relatively small, approximately
100 MB.

DreamHost cron jobs create automatic database backups and store them in Backblaze
under the webmaster account:

- hourly backups are retained for 24 hours;
- daily backups are retained for 30 days.

The backup jobs must be moved to another scheduler when the database leaves
DreamHost.

## Photo publishing

The future website needs a photo system that separates technical infrastructure from
editorial work. A technical maintainer should build and operate the publishing
pipeline, while a media director should be able to upload, organize, select, and
caption photos without working directly with code.

The existing photo Google Drive may serve as the upload and organization interface.
The final image host and publishing integration have not been selected. A small
number of website images can remain in Git, but the full gallery should use dedicated
image storage.

## To-dos

- [ ] Retire DreamHost after migrating its [current responsibilities](#dreamhost):
  - [ ] Select a managed MySQL provider; PlanetScale is one candidate.
  - [ ] Migrate the MySQL database.
  - [ ] Move the [backup jobs](#backups-and-recovery) to another scheduler.
  - [ ] Move the keelboat calendar and reservations into database v2.
  - [ ] Complete and deploy the Astro website.
  - [ ] Replace the remaining [WordPress dependencies](#wordpress-dependencies).
  - [ ] Establish the new [photo publishing](#photo-publishing) workflow.
  - [ ] Move the domain registration and DNS records.
  - [ ] Preserve Google Workspace mail routing, Resend authentication, subdomains,
        and required forwarding addresses.
  - [ ] Confirm that no production system or scheduled task still uses DreamHost.
- [ ] Set up a shared password manager for non-Google service accounts that:
  - [ ] Make the Commodores the vault administrators.
  - [ ] Grant position holders access only to the accounts they need.
