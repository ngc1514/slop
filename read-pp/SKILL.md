---
name: read-pp
description: Read and digest a privacy policy (URL, pasted text, or local file) and produce a short, security-conscious bullet digest in the "I read the privacy policy so you don't have to" format. Use when the user runs /read-pp or asks for a privacy policy / ToS / DPA summarized, audited, or turned into red-flag bullets.
allowed-tools: WebFetch, WebSearch, Read, Write, Bash, Glob, Grep
---

# read-pp — Privacy Policy Digest

Turn a privacy policy into a scannable list of what the service actually reserves
the right to do with user data. Written from a security/privacy-conscious point of
view: surface the things a paranoid reader would care about, not the marketing.

## 1. Get the source text

The argument to `/read-pp` may be any of these. Detect which, don't ask:

| Input | Action |
|---|---|
| URL (`https://...`) | `WebFetch` it. If the page is a JS shell, a nav stub, or under ~2000 chars, look for the real policy link (`/privacy`, `/legal/privacy`, `/privacy-policy`) and fetch that. |
| Local path | `Read` it (PDF, md, txt, html all fine). |
| Pasted text | Use it directly. |
| Just a product name ("telegram") | `WebSearch` for `<name> privacy policy`, fetch the official domain's policy. State which URL you used. |
| Nothing | Ask for a URL, file, or pasted text. Do not guess. |

Long policies: fetch in sections rather than truncating. Many services split
across several documents (Privacy Policy + Terms + Cookie Policy + a regional
addendum). If the main policy references a linked sub-policy that carries real
data-handling terms, fetch that too and fold it in. Say in the footer which
documents you read.

**Never invent a clause.** Every bullet must trace to text you actually read. If
the policy is silent on something important, that silence is itself a bullet
(see "Notable silences" below) — but mark it as silence, not as a stated practice.

## 2. What to look for

Read the whole thing, then sweep specifically for these. Not every category will
apply; skip the empty ones.

- **Encryption** — E2EE by default, opt-in, or absent? What's excluded (backups,
  cloud sync, metadata, attachments)?
- **Identity requirements** — phone number, government ID, real name, biometrics.
- **Retention** — concrete durations for logs, IPs, metadata, deleted content,
  backups. Grab the numbers; numbers are the most useful bullets.
- **Deletion** — what "delete" actually removes, what persists and for how long,
  what's kept in backups or on other users' devices.
- **Metadata** — IP, device IDs, user agent, contacts graph, timestamps, location
  inferred from IP.
- **Contacts / address book** — uploaded? hashed? retained for non-users?
- **Law enforcement** — what's disclosed, under what legal process, whether
  notification to the user happens, transparency reporting.
- **Third parties** — processors, ad partners, analytics SDKs, payment providers,
  AI/model vendors. Named or unnamed?
- **AI / training** — is user content used to train models? Opt-out? Human review?
- **Ads & profiling** — inferred interests, behavioral scoring, cross-app tracking,
  ad IDs, lookalike targeting.
- **Jurisdiction & transfers** — where data lives, cross-border transfer
  mechanisms, whose legal process applies.
- **Moderation** — human review of reported content, automated scanning, hash
  matching.
- **Extensions of the platform** — bots, plugins, integrations, business/enterprise
  accounts, mini-apps: what they can reach that the core product can't.
- **Children's data**, **sale/sharing** (CCPA-style), **change policy** (can terms
  change without notice?), **breach notification**, **security specifics**
  (or the absence of any).

## 3. Output format

Match this shape exactly:

```
I read the privacy policy because I'm an unpaid intern so you don't have to.

Episode: <Service Name>

- <finding>
- <finding>
- <finding>
```

Then a short footer:

```
Source: <url or file> (<policy version/date if stated>)
```

### Bullet rules

These are what make the format work. Follow them.

1. **10–15 bullets.** Fewer if the policy is genuinely thin. Never more than 18 —
   the value is in the cut.
2. **One fact per bullet.** No semicolons stacking two ideas.
3. **Short.** Target under 10 words. `Phone number required` is a perfect bullet.
4. **Capability, not intent.** Write what the policy *permits*, because permission
   is what actually binds. Prefer "can be", "may be", "aren't ... by default".
   `IP + phone number can be disclosed to authorities` — not "they share your data
   with police."
5. **Keep the numbers.** `retained for up to 12 months`, `can persist for up to
   48 hours`. Specific durations are the highest-signal bullets in the list.
6. **No hedging adverbs, no editorializing.** No "worryingly", "shockingly", "note
   that", "it appears". The facts carry it. No emoji, no severity icons, no bold.
7. **Neutral-but-pointed voice.** Deadpan. You are reporting, not warning.
8. **Sort by how much a privacy-conscious reader would care**, roughly: encryption
   → required identity → retention → disclosure → third parties/AI → ads →
   integrations → jurisdiction → deletion.
9. **No positives** unless a protection is unusually strong and load-bearing (real
   default E2EE, a no-logs claim, an explicit no-training commitment). One or two
   max, phrased flatly: `No cloud backup of message content`.
10. **No boilerplate.** "We take your privacy seriously", "we use industry-standard
    security", GDPR rights that every policy restates — cut all of it. If every
    company says it, it isn't a finding.

### Notable silences

If the policy conspicuously *fails* to address something material, add at most 2–3
bullets in a trailing section:

```
Not stated:
- No retention period given for server logs
- No mention of whether content trains models
```

## 4. Optional extended mode

If the user asks for detail, receipts, or "the long version", follow the bullet
list with a `Details` section: each bullet restated with a one-line paraphrase of
the governing clause and the section name/number it came from. Still no quoting
of long passages — cite the location and paraphrase.

## 5. Handling

- **Ambiguous clause** — report the broader reading (what it permits) and note the
  ambiguity in a single trailing line, not inside a bullet.
- **Regional variants** (GDPR/CCPA addenda) — digest the general/global policy as
  the main list; add a bullet only where a region materially differs.
- **Fetch fails / paywalled / bot-blocked** — say so plainly and ask for pasted
  text. Do not summarize from memory or from what you know about the company.
- **Dated policy** — include the effective date in the footer. If the document
  carries no date, say `(no effective date stated)`.
