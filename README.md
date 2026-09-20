# jd-to-cv

A Claude Code plugin that writes your CV for a specific job from a locked
file of your own facts. It does not invent numbers, it does not reuse
bullets, and it does not sound like a machine wrote it.

| Skill | When | Does |
|---|---|---|
| `/payload-setup` | once | Reads your current CV, asks you about the story behind each number, saves your fact file and your name block. |
| `/jd-to-cv` | per job | Scores the fit, writes a strategy brief, asks "Go?", then writes, renders and checks the PDF. Also cover letters, batch triage, form answers, recruiter messages. |

## Why

Most CV generators are template fillers. Paste a JD, get your old bullets
back with the JD's keywords pushed in, plus a layer of "leveraged
cross-functional stakeholders to drive impact". Recruiters recognise that
voice in a few seconds. Some tools also round your 38% up to 40% or add a
tool you never touched, and one checkable error ends the application.

Here the rule is simple: facts are locked, prose is free. Your numbers,
tools, titles and dates live in `payload/payload.md`. The writing stage can
only read that file and a strategy brief. It never sees the raw JD, so it
cannot echo it and cannot cite anything you did not put there. Every line is
written fresh for each job. A blacklist of AI vocabulary and patterns runs on
everything, and a final gate reads the CV the way a screener does before it
ships.

Each CV logs a row to `pipeline_log.csv`. Mark what got interviews and the
next brief for a similar role uses that.

## Install

Needs [Claude Code](https://claude.com/claude-code).

```
/plugin marketplace add ajinkyachintawar/jd-to-cv
/plugin install jd-to-cv@jd-to-cv
```

PDF rendering needs tectonic and poppler:

```bash
brew install tectonic poppler
```

Debian/Ubuntu: `sudo apt install poppler-utils`, tectonic from
[tectonic-typesetting.github.io](https://tectonic-typesetting.github.io).

The design uses the Charter font. Missing it? `brew install --cask
font-charter`, or change `\setmainfont{Charter}` in `templates/preamble.tex`.

## Set up your payload

Work in a folder of your own. The plugin writes there, never into itself.

```bash
mkdir ~/job-hunt && cd ~/job-hunt
claude
```
```
/payload-setup
```

Hand it your current CV (PDF, DOCX or text). It pulls out every role,
project, degree, cert and skill with the numbers and dates as written, then
asks you what each number measured, what the baseline was, and how it
happened. If you do not know a figure, it stays blank. It also asks you to
describe a project in your own words and keeps a few of those sentences as
the voice for every CV.

You set your lanes (section order per role type), page length (1 or 2, your
call), countries and companies to skip, work authorisation, salary bands and
standard form answers. It finishes by compiling a tiny test CV so you know
the LaTeX chain works.

```
job-hunt/payload/
  payload.md      your facts. The only source the writer may use.
  lanes.md        lanes, page target, salary bands, form answers, skip list
  header.tex      your name block for the CV
  header-cl.tex   same, for cover letters
```

Spend fifteen minutes on the interview. Every CV after this is only as good
as this file.

## Use it

```
/jd-to-cv
<paste the job description>
```

You get a fit score, a keyword table and a strategy brief. It ends with
"Go?". Say go. The PDF lands in `Applications/<date>_<Company>/` at the page
count you set, with every keyword checked against the PDF text and a
scorecard printed.

Then, if you want them:

| Say | Get |
|---|---|
| `cover letter` | one page, matched design |
| `triage these` + up to 10 JDs | ranked table, you pick |
| `audit CV vs JD` | covered / weak / missing, with fixes |
| a form question | one line, from your bands and standard answers |
| `message the recruiter` | connection note, DM, one-week follow-up |
| `add my new role at X` | interview for the new facts, written back dated |

## **Ten JDs per session, then open a new one**

**After about ten job descriptions in one Claude Code session, start a
fresh session.** Past that point the model starts borrowing phrasing from
the earlier CVs in the same conversation and the bullets drift toward each
other. A new session reloads the payload cold. You lose nothing:
`pipeline_log.csv` and `Applications/` persist on disk.

Triage first, run the full pipeline on the roles you actually want.

## Repo layout

```
.claude-plugin/          plugin and marketplace manifests
skills/payload-setup/    the setup interview + payload and lanes templates
skills/jd-to-cv/         the pipeline
templates/               LaTeX design, header templates, content skeleton
```

`payload/`, `Applications/` and `pipeline_log.csv` are yours and are
gitignored. Do not commit your payload to a public repo.

## Change the design

Edit `templates/preamble.tex`. Keep `\role`, `\proj`, `\stack` and
`\section`; the writer uses them. Section headers are bold capitals because
small caps break the text layer ATS parsers read.

## Licence

MIT.
