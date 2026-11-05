# 4. Privacy, terms, cookies, age and consent

Last verified: 2026-11-05. **This is engineering guidance, not legal advice.** Laws differ by country and change. Every text produced from this file is a *draft pending review by a qualified person* in the owner's jurisdiction. See `09-when-to-ask-a-professional.md`.

## Principles

1. **Collect the minimum.** The strongest legal position is the data you do not hold. Decide each field's purpose before adding it.
2. **Write what the code really does.** Generate the privacy text from the code and the provider list, not from a generic template. When the code changes, the text changes (we once wrote that a provider identifier was received; the code never read it, and we corrected the text).
3. **Mark owner facts and proposals** with `[PLACEHOLDER: …]` and `[PROPOSAL: …]` (see `templates/legal-markers.md`). Show a visible "draft pending legal review" banner and a version string on every legal page.
4. **A checkbox is a declaration, not verification.** Say so in the texts and in your own head.
5. Never write "compliant with X". Write "draft".

## Who is the data controller?

Someone must be named: usually the site's owner (a person or a company), **not the freelance developer** who builds it. The developer acts on the owner's behalf; write the arrangement down (who owns content, who pays, who has access, who answers privacy requests). Show at least a legal name and a working contact address; whether a postal address or ID number must be public depends on the jurisdiction (ask a professional). Keep the account of the domain registrar and key services in the owner's name.

**If the repository is public**, the owner's legal name and contact details must not live in code, tests, fixtures or commit messages. Store them in an admin-only site setting that the owner (or an administrator) fills in on the production site, and render the legal pages from it. Tests use obviously fake names.

## Documents to draft

- **Privacy policy**: who is responsible; what is collected (account, age bracket, progress, contact messages, technical logs); why; legal basis where relevant; retention per data type; who receives data (provider table: what each receives, where it is based if known, link to their policy); international transfers; rights and how to exercise them, with a response time `[PROPOSAL]`; deletion; security; incident notice `[PROPOSAL]`; minors; changes and versions.
- **Terms of use**: who may use the service (age); account rules; user content; intellectual property; third-party sellers if you link to them (say the purchase happens on their site under their terms); limitation of liability and governing law as `[PLACEHOLDER]` for a lawyer; contact.
- **Cookie policy**: a table of every cookie and storage key (name, purpose, duration, first/third party). If you only use strictly necessary cookies and no analytics or ads, a short notice is enough in many places; any analytics, advertising or tracking pixel changes that and usually requires prior consent (see the ePrivacy rules in the EU/UK and local equivalents).
- **Third parties list** (hosting, database, mail, media, sign-in, CDN, registrar) with the data each receives. A browser fetching images directly from a media CDN gives that CDN the visitor's IP: include it.
- **Deletion request procedure**, covering both the account and contact messages. Make the in-app deletion actually delete (test it, including game state or derived records; keep only de-identified aggregates).
- **Fonts and images**: licences (for example the SIL Open Font License) and a statement that images are the owner's or licensed.

## Age and consent flow (design that worked)

Decide the minimum age from the audience and law (the number differs: for example the EU lets member states set 13–16 for digital consent, US children's privacy rules start at 13, other countries differ; check locally).

- Ask for **month and year of birth** only, never the day or the exact age. Compute age **conservatively** (you turn N at the end of your birth month) on the server, with the same function in the client for UI only. The server decides.
- Below the minimum: **reject before creating anything**. No account, no password hash, no email sent, no record, and nothing in the logs (no email, name or date). The client can avoid calling the server at all. Use one gentle, uniform message that reveals nothing else.
- Rate-limit rejections per network so dates cannot be probed until one passes, but size the limit for shared networks.
- Between minimum and majority: require a guardian-authorization checkbox; at majority: an "I am an adult" checkbox. These are declarations.
- **Separate checkboxes** for accepting the terms and for authorising data processing (some regimes require consent to be independent and revocable as easily as it was given). Do not pre-tick.
- Store only: birth year and month, the guardian-authorization flag, timestamps of each acceptance, and the version string of the texts accepted. Never the day or the computed age.
- Existing users and staff accounts created by an administrator: make the new fields optional so nobody is locked out; decide separately whether to re-ask existing users when texts change.
- Social sign-in for new users: two steps. First call returns "needs profile" and creates nothing; the second completes the declarations and re-verifies the provider credential. Existing accounts sign in as before.
- If someone else could have made the declarations (an unverified account later claimed by its owner), **ask again** at the owner's next sign-in.

## Reviews and endorsements

If you publish reviews taken from elsewhere (for example a retailer), get the reviewer's **permission**, record it (who, when, how), keep the text unaltered or mark it as an excerpt, show the source, use the name the person agrees to, and give a way to withdraw. Do not present a hand-picked set as the average opinion: no aggregate rating or star average, a title such as "Selected reviews", and no incentives. Consumer-protection and advertising rules about misleading endorsements vary by country; have the policy reviewed.

## Links to a retailer

If the site only links to third-party sellers, say so in the terms. If the links are **affiliate** links (the owner earns a commission) disclose it near the link. A site that sells directly has more obligations (seller identity, prices, returns, payment terms): get advice before enabling sales.

## What to hand a professional

A one-page package: the data inventory (fields, purpose, retention, recipients), the provider table, the age/consent flow description, the draft texts with all `[PLACEHOLDER]`/`[PROPOSAL]` marks listed, the jurisdiction assumption, and the questions you could not verify (registration duties with the data authority, treatment of minors, whether using foreign hosting counts as a transfer). That makes a short paid consultation far more useful.
