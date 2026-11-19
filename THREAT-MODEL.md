# Threat Model — Farm ERP

Version 1, November 2026. Review this page at the start of every phase.

## 1. What are we building?

A farm ERP for a mixed farm in Kenya. A Spring Boot API runs on a Contabo VPS
behind Caddy. PostgreSQL runs inside Docker and is never reachable from the
internet. A Flutter app on phones works offline and syncs later.

Users: the owner, farm workers, later a vet and an accountant. Later still,
other farms.

What we must protect:

- Workers' personal details: names, phone numbers, ID numbers
  (personal data under the Kenya Data Protection Act 2019)
- Money records and M-Pesa payments
- Farm history: animals, harvests, events (must never be secretly changed)
- Secrets: passwords, login tokens, Daraja keys, AI API keys

## 2. What can go wrong?

| # | Threat | Weak spot | Damage |
|---|--------|-----------|--------|
| 1 | Stranger guesses a password (brute force, password spraying) | Login page | Full access to the farm |
| 2 | Stranger floods the login or API (DoS) | Public API | Farm can't record work or pay workers |
| 3 | Workers' phone numbers leak, then get used for SIM swaps and fake M-Pesa messages | Personal data, CSV exports | Workers scammed, legal breach |
| 4 | Worker edits or deletes records to hide theft (insider threat) | Event records | The books lie |
| 5 | A user of one farm reads another farm's records (IDOR) | Record IDs in requests | Data leak between farms |
| 6 | Fake M-Pesa callback says "paid" | Callback URL | Money recorded that never arrived |
| 7 | Lost or stolen phone | Login token on the phone | Someone acts as that worker |
| 8 | Secrets pushed to GitHub | Git repository | Database or Daraja account taken over |
| 9 | SQL injection | Database queries | Records read or wiped |

## 3. What will we do about it?

| # | Defence | When |
|---|---------|------|
| 1 | Argon2 password hashing, limit login attempts, two-factor login for the owner | Phase 1 (Dec 2026) |
| 2 | Rate limiting in Caddy and the API | Phase 1 (Dec 2026) |
| 3 | Personal details in their own table; only roles that need them can see them; never sent to the AI; exports logged | Phase 1, Phase 4, Phase 5 |
| 4 | Events are append-only (corrections are new events); audit log of who did what and when; roles | Phase 1 |
| 5 | Farm and role check on every request, on the server | Phase 1 |
| 6 | Confirm every payment with Daraja; process each receipt number exactly once | Phase 2 |
| 7 | Tokens in secure storage on the phone; owner can sign a phone out remotely | Flutter app |
| 8 | Secrets in environment variables only; `.gitignore`; secret and dependency scans in CI | Now, then CI |
| 9 | Queries only through JPA parameters, never by gluing strings together | Phase 1 |

## 4. Did we do it well?

- Saturday security sprints check one row of this table against the real code.
- January 2027: attack the API myself with OWASP ZAP and Burp Suite.
- Any new feature adds its threats here before it is built.
