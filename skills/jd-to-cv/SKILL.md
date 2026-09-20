---
name: jd-to-cv
description: Adversarial job application pipeline. Analyzer (ATS sim + fit score) → Hacker (strategy brief) → Writer (fresh prose from locked facts) → Render (2-page LaTeX PDF) → Gate (6-second sim + slop scan). Handles single JDs, batch triage of up to 10 JDs, cover letters, CV-vs-JD audits, application-form answers and recruiter outreach. Use when the user pastes a job description or job URL, says "analyze this", "should I apply", "write CV", "triage these", "cover letter", or any job application task. Requires payload/ built by /payload-setup.
---

# JD-to-CV Pipeline

Four stages, each with one job. Facts locked, prose free.

## Setup check (first thing, every session)

```bash
ls payload/payload.md payload/lanes.md payload/header.tex 2>&1
```
Missing → stop and run `/payload-setup`. Nothing in this skill works
without it.

**Before first use (once per session):** read `payload/payload.md` (the ONLY
source of CV facts) and `payload/lanes.md` (lanes, disqualifiers context,
salary bands, form answers). Cache both; never re-read per application. If
they are already in this conversation's context in any form, do not read
them again.

**Regime:** the payload is a fact library, not a bullet library. Every
number, tool, company, date and event is sacred. If a fact is not in the
payload, it does not exist. All prose is composed fresh per JD following
the payload's VOICE GUIDE.

**Paths:** `${CLAUDE_PLUGIN_ROOT}/templates/` holds the LaTeX design.
`payload/` holds the user's facts and header. `Applications/` and
`pipeline_log.csv` are created in the current working directory.

**Model note:** structure-carried; a mid-tier model handles every stage
for volume days. For a high-stakes target, suggest switching to a
stronger model before Stage 2 (Hacker), the only stage where model
quality visibly matters.

---

## Disqualifiers (ALWAYS first, before any stage)

Read `payload.md` → DISQUALIFIERS and PERSONAL → Work authorisation /
Target geography. Any hit → `🚫 SKIP — [reason from payload].` and stop.

**Location gate:** a posting that says only "Remote" with no country is
not in the target geography until proven. For a company headquartered
outside the target geography posting an unqualified remote role, resolve
eligibility before Stage 3 (application form location dropdown, salary
currency, sibling reqs). If it cannot be resolved, run Stage 1-2 only and
tell the user to confirm before "go".

---

## Flow Modes

| User intent | Run |
|---|---|
| Pastes 1 JD / "analyze" / "should I apply" | Stage 1 → Stage 2, then ask "Go?" |
| Pastes multiple JDs / "triage these" | Batch: Stage 1 only on each → ranked shortlist → user picks → Stage 2+ per pick |
| "go" / "write CV" | Stage 3 → 3.5 → 4 → log to tracker |
| "cover letter" | Stage 5 |
| "audit CV vs JD" | Audit |
| Form question / "message the recruiter" / outreach | Stage 6 |
| New JD mid-session | New Stage 1; nothing carries over between applications |

**JD intake:** if the user pastes a whole job-board page, use only the JD
body. Never quote, echo or process tracking URLs and page furniture.

---

## Stage 1 — ANALYZER (ATS simulator; mechanical, no strategy)

Read the JD the way the parser and a 6-second screener do. Output EXACTLY
this, max 40 lines:

```
## [Role] at [Company] | [Location] | [Level]

FIT: [0-100] → [PROCEED / BORDERLINE / STOP]

KEYWORD TABLE (exact JD spelling; weight = # of JD sections it appears in):
  MUST-HIT (title/requirements/duties, 2+ sections): [term(w), ...]
  NICE (1 section): [terms]
  EXACT PHRASES (mirror verbatim, 5-8 max): ["...", "..."]
  ACRONYM FORMS: [e.g. "Power BI" not "PowerBI"]

TONE: [1 line — register + 4-6 dominant JD verbs]
HARD REQUIREMENTS: [pass/fail items incl. years, certs, languages]
⚠️ GAP: [requirement not in payload] → MITIGATION: [honest framing] (one line each)
```

**Fit scoring:** score against payload FACTS, not job titles. Title is
irrelevant; evidence is everything.

**Hard stop below 60:** output the fit line + 2-line reason and STOP. The
cheapest CV is the one never written. User overrides with "do it anyway".

**Batch mode:** Stage 1 only, compressed to 8 lines per JD, then a ranked
table (Company | Role | Fit | one-line reason). Ask which to take forward.

---

## Stage 2 — HACKER (strategy; the thinking stage)

Input: Stage 1 output + payload. Output a brief, max 50 lines:

```
## BRIEF: [Role] at [Company]

THEY NEED: [1 line — the problem behind the posting]
THEY FEAR: [1 line — the mis-hire they're guarding against]
TYPICAL APPLICANT: [1-2 lines — what the other 200 CVs will say]
DIFFERENTIATION: [1-2 lines — how this CV deliberately won't say that]

BIN-TEST (why a screener rejects in 6 seconds → counter):
1. [reason] → [counter placed where]
... (up to 5)

LANE: [preset from lanes.md OR synthesized: section order + summary voice + story emphasis; state Skills placement reasoning]
HOOK: [the one thing that stops the reader] → position: [headline / summary / first bullet]
ANGLE: [1-2 lines — the single argument this CV makes]

KEYWORD PLACEMENT MAP (every MUST-HIT term gets ONE home):
  headline: [terms] | summary: [terms] | skills: [terms]
  [story/bullet target]: [terms]

FACT SELECTION:
  STORIES: [shortcodes + fact clusters, e.g. ACME: commercial + reporting]
  PROJECTS: [2-3 ranked, one-word rationale]
  SKILLS: [JD-priority order] | CERTS: [max 4]
```

**Lane synthesis:** no preset fits → compose one: section order by what
the screener needs first, summary voice matched to JD register, story
emphasis from payload ANGLES tags. Note it in the brief.

**Learning loop:** if `pipeline_log.csv` exists, grep it for similar role
types; reuse angles that got interviews, avoid angles with rejection streaks.

End with **"Go?"** — the only checkpoint in the pipeline.

---

## Stage 3 — WRITER (composition; brief + facts only)

**Compose from the Stage 2 brief and payload facts ONLY. Do not re-read or
quote the raw JD.** Keyword coverage comes from the placement map, nothing
else. This is what prevents keyword parroting.

### Writing rules
- Follow the payload VOICE GUIDE absolutely: plain verbs, banned list, no
  em-dashes, no pronouns on the CV.
- **Orientation first:** every experience and project entry makes clear
  what the company or product IS before any metric lands. No unexplained
  product names or internal jargon.
- **Vary bullet shapes.** Never repeat `verb + thing + by/through + metric`
  twice in a row.
- Metrics carry their mechanism (from the story's MECHANISM line).
- **10-15 words per bullet, hard cap; ~112 characters** so it renders on one
  line. Longer idea → two bullets. Exception: ONE origin-story bullet on the
  flagship project may run to ~25 words.
- One bullet per role may be plain and metric-free.
- **Summary is not a story:** who + years + core tools + one proof point,
  2-3 sentences, **80 words max**, contains the HOOK, never a role-by-role
  narrative and never "[Title] with X years of experience".
- No floating prose outside sections; everything is a bullet or a header.
- **Bold rationed: 5-8 load-bearing terms across the whole CV**, first use
  only. Skills category names are exempt.
- Placement-map keywords appear at their assigned home, exact spelling, once.
- Match the JD's spelling variant (US vs British) across the whole CV.

### Section rules
- Experience: current role first, then per lane. **Full-time roles: 4-5
  bullets, minimum 4. Internships: 2-3, minimum 2.** These floors hold
  regardless of JD relevance. Each role gets a `\stack{}` line with its
  tools. Titles print exactly as the payload says; never "Intern" in a
  title.
- Projects: 2-3 per brief, 3 bullets each (AI-heavy roles: 4). Include
  payload links; a DeepWiki page, if listed, is added as
  `\href{...}{Chat with repo}`.
- **Long-URL rule:** DOIs, arXiv IDs and any URL over ~30 chars go in
  `\href{full-url}{short-label}`, never raw.
- Skills: brief's subset only, grouped, ~5-6 categories, bold category
  names, depth qualifiers inline (`UiPath (working knowledge)`).
- Publications: per lane rule. Certs: max 4, in-progress ones labelled.

---

## Stage 3.5 — RENDER (LaTeX → verified 2-page PDF)

Design lives in `${CLAUDE_PLUGIN_ROOT}/templates/preamble.tex` (bold caps
section headers, never small caps: pdftotext breaks them). Skeleton:
`${CLAUDE_PLUGIN_ROOT}/templates/cv-template.tex`. Header:
`payload/header.tex` (carries `\begin{document}`).

**Token discipline: never write the preamble into context.** Write ONLY a
content file, then assemble on disk:

1. `mkdir -p Applications/[YYYY-MM-DD]_[Company]`, write `content.tex`
   there (Summary onward, `\role`/`\proj`/`\stack`/itemize per the
   skeleton; escape `% & # _`; ends with `\end{document}`).
2. Line-length check: `grep -n 'item' content.tex | awk 'length($0)>125'` → shorten hits.
3. Assemble + compile:
   ```bash
   cat "${CLAUDE_PLUGIN_ROOT}/templates/preamble.tex" ../../payload/header.tex content.tex > CV_[Company]_[Role]_[Surname].tex && tectonic CV_*.tex
   ```
   Fix errors in content.tex, reassemble.
4. `pdfinfo CV_*.pdf | grep Pages` → must be exactly 2. Over → trim bullets
   by angle-relevance, never below the floors; under-filled page 2 is fine,
   a 3rd page is not.
5. `pdftotext CV_*.pdf - | grep -c -i '<term>'` for name, phone, email and
   every MUST-HIT. Grep, don't dump the text into context.
6. Visual check (`pdftoppm -png -r 80`, view, delete) **only when warranted**:
   first CV of the session, a template change, or an anomaly in steps 4-5.

---

## Stage 4 — GATE (runs automatically after Stage 3.5)

Failures → rewrite ONLY the failing lines, never regenerate the CV.

1. **ATS coverage:** every MUST-HIT present at its mapped home, exact spelling? X/Y.
2. **6-second sim:** read ONLY name, headline, summary, first two bullets.
   Does it answer "can this person do THIS job" and give one reason to keep
   reading? Interview or bin.
3. **Slop scan:** banned verbs, repeated skeletons, em-dashes, bold > 8,
   uniform bullet lengths, "responsible for", metric without mechanism in a
   hook position, any pronoun on the CV (org names containing "My" exempt),
   any entry whose first bullet does not orient the reader.
4. **JD-echo check:** any JD phrase not in the placement map → remove.
5. **Length check:** bullet word counts, summary ≤ 80, bullet floors met,
   total bullets ≤ ~35.

```
GATE SCORECARD
  ATS coverage: [X/Y must-hits] = [%]  (missing term + where added)
  6-second sim: [INTERVIEW / BIN] — [one line why]
  Slop scan: [clean / fixed N] | Echo: [clean / fixed] | Length: [pass / trimmed]
  Content: [x/10] — differentiation, credibility, human voice
  Honest risks: [1-2 lines — things wording can't fix]
  OVERALL: [x/10] → [SHIP / FIX FIRST]
Filenames: CV_[Company]_[RoleShorthand]_[Surname].pdf | CL_..._[Surname].pdf
```
Ship threshold: ATS ≥ 90% AND overall ≥ 8. Below → fix named lines, rescore
once. Never inflate; a true 7 with named risks beats a fake 9.

### Tracker (after every gated CV)
Append one row to `./pipeline_log.csv` (create with header if missing):
```
date,company,role,location,source,fit_score,lane,angle,hook,cv_file,cl_file,status
```
Bash heredoc/echo append. `status` starts `generated`; the user updates
outcomes later. Never edit any spreadsheet the user keeps alongside it.

---

## Stage 5 — Cover Letter (on request only)

Requires CV in context. 200-280 words, 4 paragraphs, no bullets, no
em-dashes, first person is normal here. Voice guide applies.

Content file starts at the date and uses `\recipient{}{}{}` and
`\subjectline{Re: [Exact Role Title]}`, then:
P1 hook + strongest proof point (never "I am writing to apply").
P2 1-2 experiences mapped to JD responsibilities, tool + metric + mechanism.
P3 why this company, something real from the JD.
P4 location, availability, one sentence.
Sign-off: name + LinkedIn/portfolio line. Ends with `\end{document}`.

Render: `cat "${CLAUDE_PLUGIN_ROOT}/templates/preamble-cl.tex" ../../payload/header-cl.tex content-cl.tex > CL_[Company]_[Role]_[Surname].tex && tectonic CL_*.tex`
→ must be 1 page. Report word count + filename; update the tracker row.

---

## Stage 6 — Application Questions & Outreach (on request)

First person is normal here. Voice guide still applies; facts from payload only.

**Form questions:** answer at the length the field wants, default ONE line.
Salary → bands in `lanes.md`, anchor mid-band as a range, append "flexible
depending on the overall package". Visa → the standard answer in `lanes.md`.
"Why us" → 2 sentences max: one real observation + the brief's strongest proof.

**Outreach:**
- LinkedIn connection note (≤300 chars): name the role, one proof point, no flattery.
- Post-application DM / email (≤80 words): applied for [role], the single most
  relevant thing built with a link, one sentence why this team, ask only that
  they glance at it.
- Follow-up (1 week, ≤40 words): one new fact or link, never "just checking in".
Reuse the brief's HOOK and ANGLE; never send identical text to two people at one company.

---

## Audit — CV vs JD (on request only)

```
## Audit: [Role] at [Company]
COVERED: [req] ✅ (bullet evidence) | WEAK: [req] ⚠️ (skills-only) | MISSING: [req] ❌
ATS ESTIMATE: ~[X]% → after fixes: ~[X]%
HIGH-IMPACT FIXES: 1. [swap] → +Xpp ...
VERDICT: [GO ✅ / FIX THEN GO ⚠️ / NO-GO ❌]
```

---

## Multi-application sessions

- References read once; rules never re-explained.
- Stage outputs stay inside their line caps.
- Nothing carries between applications except the learning-loop lookup.
- Batch triage first, full pipeline only on the shortlist.

## Rules (absolute)

1. Metrics are sacred: never round, approximate or change any number.
2. Tools are facts: only claim what the payload lists for that role.
3. No fabrication: not in the payload, not on the CV.
4. Honest about gaps: mitigate framing, never oversell.
5. Prose composed fresh per application; never reuse old bullet text.
6. Active voice, human voice, short sentences, real verbs.
7. Work authorisation stays off the CV unless the user explicitly asks.
8. New fact learned mid-session (user corrects a number, adds a tool) →
   write it into `payload/payload.md` with `(user-confirmed YYYY-MM-DD)`
   before continuing.
