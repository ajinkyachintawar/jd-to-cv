---
name: payload-setup
description: One-time (and on-update) builder of the fact payload that /jd-to-cv writes CVs from. Use when the user wants to set up jd-to-cv, says "build my payload", "set up my CV facts", uploads or points at an existing CV/resume to seed from, or wants to add a new role/project/cert to the payload. Also run automatically when /jd-to-cv finds no payload/ folder.
---

# Payload Setup

Builds `payload/` in the current working directory. Everything /jd-to-cv
ever puts on a CV comes from these files, so this is where truth is locked.

```
payload/
  payload.md      fact library (from references/payload-template.md)
  lanes.md        lane presets + salary bands + form answers (from references/lanes-template.md)
  header.tex      CV name block (from ${CLAUDE_PLUGIN_ROOT}/templates/header.tex)
  header-cl.tex   cover-letter name block + footer
```

## Step 0 — Prerequisites check

```bash
for t in tectonic pdfinfo pdftotext; do command -v $t >/dev/null || echo "MISSING: $t"; done
```
If anything is missing: macOS `brew install tectonic poppler`; Debian/Ubuntu
`sudo apt install poppler-utils` + tectonic from https://tectonic-typesetting.github.io.
Report it once and continue; rendering is the only stage that needs them.

## Step 1 — Intake

Ask which mode, or infer from what the user gave you:

**A. Seed from existing CV(s)** (preferred; user gives PDF, DOCX, or text)
- PDF: `pdftotext -layout file.pdf -`. DOCX: `unzip -p file.docx word/document.xml | sed 's/<[^>]*>/ /g'`. Text/markdown: read directly.
- Extract into the template: one STORY per role, one PROJ per project,
  EDUCATION, CERTIFICATIONS, SKILLS. Copy numbers, dates, titles, tools
  *verbatim*. Existing bullet prose is NOT a fact; strip it back to the
  fact inside it ("Increased revenue 40% via dashboard rollout" → FACT:
  revenue +40%; MECHANISM: dashboard rollout; ask what the dashboard did).
- Multiple CVs: union the facts; flag any conflicts (same metric, two
  values) and ask which is true.

**B. Interview from scratch** — walk the template top to bottom, one
section per message. Never ask more than 4 questions in one turn.

## Step 2 — Mechanism interview (the part that makes it work)

For every STORY and PROJ, ask until each headline number has:
1. **What it measured** (revenue, cycle time, accuracy, users...)
2. **Baseline → result** with units, not just the delta
3. **How** (the mechanism: the test run, the automation built, the process changed)
4. **Your share** (sole owner / led / contributed)

Also ask, once per STORY: "What is the company / product, in one plain
sentence a stranger understands?" That line goes into "The story in one line".

If the user cannot give a number, record the fact without one. Never
suggest a plausible figure. Never round what they give you.

## Step 3 — Write the files

1. Copy `${CLAUDE_PLUGIN_ROOT}/skills/payload-setup/references/payload-template.md`
   → `payload/payload.md`, fill every `<<...>>`, delete unused blocks whole.
2. Same for `lanes-template.md` → `payload/lanes.md`. Propose 2-4 lanes
   from the user's target roles; ask them to confirm section order and
   voice per lane, and the Page Target (1 or 2 pages; do not assume).
   Fill DISQUALIFIERS and Work authorisation carefully: the pipeline skips
   roles on these alone.
3. `header.tex` / `header-cl.tex`: copy from `${CLAUDE_PLUGIN_ROOT}/templates/`,
   substitute `<<NAME>>`, `<<NAME_UPPER>>`, `<<LOCATION>>`, `<<PHONE>>`,
   `<<EMAIL>>`, `<<LINKEDIN>>`, `<<PORTFOLIO>>`. No portfolio → delete
   that `\href` line and the `\textbar` before it.
4. Confirm no `<<` remains: `grep -rn '<<' payload/`.

## Step 4 — Smoke render

Write a 6-line `payload/_smoke.tex` content file (Summary with one
sentence, one `\role`, one bullet, `\end{document}`), then:

```bash
cd payload && cat "${CLAUDE_PLUGIN_ROOT}/templates/preamble.tex" header.tex _smoke.tex > _smoke_full.tex && tectonic _smoke_full.tex && pdftotext _smoke_full.pdf - | head -5; rm -f _smoke*
```
Name, phone, and email must appear in the text output. If Charter is not
installed the font line fails: tell the user to edit `\setmainfont{Charter}`
in the plugin's `templates/preamble.tex` (or `brew install --cask font-charter`).

## Step 5 — Hand off

Print a 5-line summary: N roles, N projects, N certs, lanes defined,
disqualifiers set. Then: "Paste a job description or run /jd-to-cv."

## Updating later

"Add my new role at X", "I passed cert Y", "update payload": open the
relevant block, run the Step 2 interview for the new facts only, append.
Mark corrections in-place with `(user-confirmed YYYY-MM-DD)` so a later
session does not reintroduce the old value.

## Voice capture (do this during the interview)

The pipeline needs to sound like the user, not like a model. While they
answer, keep two or three sentences they actually said about their work,
verbatim, and put them in the payload VOICE GUIDE under "Add any personal
voice notes". Ask once: "Describe your favourite project to me the way you
would to a friend." That answer is the register every CV must keep. Never
write payload text in the blacklisted AI-slop vocabulary listed in
/jd-to-cv; facts recorded in slop get reproduced as slop.

## Rules

- Facts only. If the user says "about 30%", write "~30%" and ask if they
  have the exact figure. Never invent, never round, never upgrade a
  "contributed" to a "led".
- Certifications in progress are written as IN PROGRESS with a warning line;
  a false cert is checkable in one click.
- Do not copy CV prose into the payload. Facts in, prose stays out.
- The payload is the user's private data. It lives in their working
  directory, never in the plugin repo.
