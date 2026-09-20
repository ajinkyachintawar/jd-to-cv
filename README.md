# jd-to-cv

A Claude Code plugin that turns a job description into a two-page, ATS-tuned
CV that sounds like you and never invents a fact about you.

Two skills, one folder of your facts:

| Skill | When | What it does |
|---|---|---|
| `/payload-setup` | Once | Reads your existing CV, interviews you for the story behind every number, writes your fact library (`payload/`) and your name block for the LaTeX design. |
| `/jd-to-cv` | Every job | Analyzer → Hacker → Writer → Render → Humanize → Gate. Fit score, strategy brief, one "Go?", then a verified PDF in `Applications/`. Also batch triage, cover letters, form answers, recruiter messages, CV-vs-JD audit. |

---

## Why this exists

Most CV tools are template fillers. You paste a JD, they paste your bullets
back with the JD's keywords sprinkled in, and the result reads like the other
two hundred applications the recruiter saw that morning: "leveraged
cross-functional stakeholders to drive impactful, data-driven outcomes". A
screener spots that in six seconds and bins it. Worse, the generator will
happily round your 38% to "40%+" or add a tool you never used, and one
checkable lie ends the application.

This plugin takes the opposite bet:

**Facts locked, prose free.** Your numbers, tools, titles and dates live in
one file, `payload/payload.md`. The writer stage is only allowed to read the
strategy brief and that file. It never sees the raw JD, so it cannot parrot
it, and it cannot cite anything that is not in the payload. Every sentence is
composed fresh for each application; the facts never move.

**A strategy before a single bullet.** The Hacker stage works out what the
posting is really hiring for, what mis-hire they are afraid of, what the
typical applicant will say, and how this CV deliberately will not say that.
The CV is an argument, not a list.

**Humanized by rule, not by hope.** A word and pattern blacklist (leverage,
delve, "not just X but Y", the rule of three, participle tails, em-dashes,
uniform bullet rhythm) is applied to everything the pipeline writes, and a
final Gate reads the CV the way a screener does: name, headline, summary,
two bullets, interview or bin. Below 8/10 it rewrites the failing lines,
never the whole CV, and never inflates the score to pass.

**It learns.** Every generated CV logs one row to `pipeline_log.csv`. You
mark outcomes. The next brief for a similar role reuses the angles that got
interviews and avoids the ones that did not.

---

## Install

Requires [Claude Code](https://claude.com/claude-code).

```
/plugin marketplace add ajinkyachintawar/jd-to-cv
/plugin install jd-to-cv@jd-to-cv
```

Rendering needs a LaTeX compiler and the poppler PDF tools:

```bash
brew install tectonic poppler
```
Debian/Ubuntu: `sudo apt install poppler-utils` and tectonic from
[tectonic-typesetting.github.io](https://tectonic-typesetting.github.io).

The design uses the Charter typeface. If it is not on your system:
`brew install --cask font-charter`, or edit `\setmainfont{Charter}` in
`templates/preamble.tex` to a font you have.

---

## First run: build your payload

Make a folder for your job hunt. Everything the plugin writes lands here,
nothing lands in the plugin.

```bash
mkdir ~/job-hunt && cd ~/job-hunt
claude
```
```
/payload-setup
```

Give it your current CV (PDF, DOCX, or plain text; several CVs are fine, it
unions the facts and flags conflicts). It will:

1. Extract every role, project, degree, cert and skill, copying numbers,
   dates and titles verbatim. Existing bullet prose is thrown away; only the
   fact inside it is kept.
2. Interview you for the mechanism behind each number: what it measured,
   the baseline and the result, how it happened, whether you owned it or
   contributed. If you do not have a number, it records the fact without
   one. It never suggests a plausible figure.
3. Capture your voice. It asks you to describe your favourite project the
   way you would to a friend, and stores a few of your own sentences as the
   register every CV must keep.
4. Set your lanes (section order and summary voice per role type), your
   disqualifiers (countries, companies), work authorisation, salary bands
   and standard form answers.
5. Smoke-render a six-line CV to prove the LaTeX chain works on your machine.

Result:

```
job-hunt/
  payload/
    payload.md      the only place CV facts come from
    lanes.md        lane presets, salary bands, form answers, disqualifiers
    header.tex      your name block for the CV
    header-cl.tex   your name block and footer for cover letters
```

Take fifteen minutes on the interview. The quality of every CV afterwards is
capped by the quality of this file.

---

## Every job: paste a JD

```
/jd-to-cv
<paste the job description>
```

What you see:

**Stage 0, gates.** Wrong country, blocked company, or a "Remote" posting
whose country is unclear: it stops and tells you why. No CV is written.

**Stage 1, Analyzer.** A fit score out of 100 scored against your facts, not
your job titles. A keyword table in the JD's exact spelling. Hard
requirements. Gaps, each with an honest mitigation. Below 60 it stops; "do it
anyway" overrides.

**Stage 2, Hacker.** A 50-line brief: what they need, what they fear, what
the typical applicant will say, the one hook, the lane, and a placement map
giving every must-hit keyword exactly one home on the CV. Ends with **Go?**
This is the only question you answer.

**Stage 3 to 4, unattended.** Writer composes from the brief and the payload.
Render assembles the design, your header and the content into
`Applications/<date>_<Company>/CV_<Company>_<Role>_<Surname>.pdf`, compiles
it, confirms exactly two pages, and greps the PDF text layer to prove every
keyword and your contact details survived. Humanize pass runs the blacklist.
Gate scores ATS coverage, the six-second read, slop, JD echo and length, then
prints a scorecard and logs the row.

Then, on request:

| Say | Get |
|---|---|
| `cover letter` | 200 to 280 words, matched design, one page, `CL_….pdf` |
| `triage these` + up to 10 JDs | Analyzer only on each, ranked table, you pick the shortlist |
| `audit CV vs JD` | Covered / weak / missing table with the highest-impact fixes |
| a form question | One line, salary from your bands, visa from your standard answer |
| `message the recruiter` | Connection note, post-application DM, one-week follow-up |
| `add my new role at X` / `update payload` | Runs the interview for the new facts only, writes them back with a confirmation date |

---

## **Ten JDs per session, then start a new one**

**Run at most ten job descriptions in a single Claude Code session. After
that, open a fresh session before continuing.**

Each application adds a brief, a CV and a scorecard to the conversation.
Past ten, the model starts reusing phrasings and angles from earlier CVs in
the same window, bullets drift toward each other, and the Humanize pass has
more to catch. A new session reloads the payload cold and the output is
sharp again. Nothing is lost: `pipeline_log.csv` carries the learning loop
across sessions, and `Applications/` keeps every PDF.

Use batch triage first (`triage these`), take the top few forward, and
save the full pipeline for roles you actually want.

---

## What is in the repo

```
.claude-plugin/plugin.json        plugin manifest
.claude-plugin/marketplace.json   lets this repo act as its own marketplace
skills/payload-setup/SKILL.md     the setup interview
skills/payload-setup/references/  payload and lanes templates (every <<field>> explained)
skills/jd-to-cv/SKILL.md          the pipeline, stage by stage
templates/preamble.tex            CV design: fonts, spacing, \role \proj \stack \section
templates/preamble-cl.tex         cover letter design, matched
templates/header.tex              name-block templates that /payload-setup fills in
templates/header-cl.tex
templates/cv-template.tex         the content skeleton the writer follows
```

Your data (`payload/`, `Applications/`, `pipeline_log.csv`) stays in your
working directory and is `.gitignore`d here. Never commit your payload to a
public repo.

---

## Customising the design

Edit `templates/preamble.tex`. Keep the command names `\role{}{}{}`,
`\proj{}{}{}`, `\stack{}` and `\section{}`; the writer depends on them.
Section headers are bold capitals on purpose: small caps break the PDF text
layer that ATS parsers read, so "Experience" would parse as "E XPERIENCE".
Ligatures are disabled for the same reason.

To change the header layout, edit `templates/header.tex` and re-run
`/payload-setup` (or edit your own `payload/header.tex` directly).

---

## Rules the pipeline will not break

1. Metrics are sacred: never rounded, approximated or changed.
2. Tools are facts: only what the payload lists for that role.
3. Not in the payload, not on the CV.
4. Gaps are mitigated in framing, never papered over.
5. Prose is composed fresh every time; no bullet is ever reused.
6. No pronouns, no em-dashes, no blacklisted words on the CV.
7. Work authorisation stays off the CV unless you ask.
8. A fact you correct mid-session is written back to the payload before the
   pipeline continues, dated, so it never regresses.

---

## Licence

MIT. Fork it, change the design, keep the regime.
