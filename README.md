# Love Bank for a practice

A multi-couple version of Love Bank for a therapist's clients. Each couple signs in with their own couple code and password and sees only their own space. The therapist has an admin page for managing couples and can open any couple's space read-only.

Forked from David and Beth's own Love Bank (`../love-bank`) on 2026-11-13. The two are separate: different site, different database. Changes to one don't reach the other.

## Pieces

| Piece | Where |
|---|---|
| Couples' app | `index.html` (GitHub Pages, repo `davidbrynbruce/love-bank-practice`) |
| Therapist admin | `admin.html` on the same site |
| Database | Supabase project `love-bank-practice`, ref `ktfkvvopmaptqgyvxwrs` ($10/month on the Pro plan) |

## What couples get

The fortnightly check-in and tracker, Talk it through, Love notes, and the three exercises (Fondness, What's underneath, After an argument). Their sign-in screen and the footer say that their therapist can see what they enter.

Switched off in this version (`FEATURES` in `index.html`): recording with AI write-ups, and phone alerts. Both send data to outside services (OpenAI, Anthropic, Apple), and nothing has been deployed for them in this project.

## Accounts and sessions

- `lb_couples`: one row per couple (`code`, `name_a`, `name_b`, bcrypt `pass_hash`, `active`). Inside the app and database the two partners are `p1` and `p2`; the names are filled in after sign-in.
- `lb_admin`: the therapist's admin password hash (single row).
- `lb_sessions`: session tokens, stored only as SHA-256. Three kinds:
  - `couple`: a couple's device. Lasts until 180 days unused, or until the couple is paused or their password reset.
  - `viewer`: the therapist looking at one couple. Read-only: every function that changes data refuses with `read_only`. Lasts 12 hours unused.
  - `admin`: the admin page. Lasts 12 hours unused, and is kept in the browser's sessionStorage, so it ends when the browser closes.
- Wrong passwords are counted in `lb_attempts`: 10 in 15 minutes for one couple code (or for admin) blocks further tries until the window passes.

## How isolation works

Every data table (`love_bank_checkins`, `love_bank_talks`, `love_bank_notes`, `love_bank_fondness`, `love_bank_under`, `love_bank_repair`) has a `couple_id`. No table is readable from the web. The app only calls functions; each one looks up the couple from the session token (`lb_couple`) and reads or writes that couple's rows only. Deleting a couple removes all their rows.

Tested on 2026-11-13 with two made-up couples: neither could read, edit or delete the other's check-ins, notes, pictures or exercises; a viewing session could read everything and change nothing; a couple's token was refused by the admin functions and the admin token by the couple functions.

## Setting the admin password

In the Supabase dashboard, project `love-bank-practice`, SQL Editor:

`select public.lb_admin_set_password('a long password here');`

At least 10 characters. Don't save the query as a snippet. This also ends any open admin and viewing sessions. After that the therapist can change it themselves at the bottom of the admin page.

## Admin page

Add a couple (names, code, password; the sign-in details are shown once to copy), open a couple read-only, reset a password (signs them out everywhere), rename, pause and resume (pausing signs them out and blocks sign-in but keeps their data), and delete (needs the couple code typed to confirm; removes everything).

## Things to know before real clients use it

- This holds therapy-related information about identifiable people. The therapist is responsible for whether their use meets their professional and legal obligations (in the US, HIPAA). Supabase signs a BAA only on certain paid plans and it is not in place for this project.
- The project is in David's Supabase account, so David can technically read the database. Moving it to an account the therapist controls would fix that.
- Couples share one sign-in, so within a couple "only you see your answer" is a courtesy, not a lock.

## Updating

Edit `index.html` or `admin.html`, commit, push to `main`. `index.html` was generated once from the original app by a conversion script and is now maintained here by hand.
