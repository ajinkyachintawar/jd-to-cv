# jd-to-cv

A Claude Code plugin that turns a job description into an ATS-tuned,
human-voiced, 2-page LaTeX CV in about a minute, without ever inventing a
fact about you.

Two skills:

| Skill | What it does |
|---|---|
| `/payload-setup` | One-time. Builds `payload/` (your fact library, lanes, LaTeX header) from your existing CV and a short interview. |
| `/jd-to-cv` | Per job. Analyzer → Hacker → Writer → Render → Gate. Also batch triage, cover letters, form answers, recruiter outreach, CV-vs-JD audit. |

## Install

In Claude Code:

```
/plugin marketplace add <your-github-user>/jd-to-cv
/plugin install jd-to-cv@jd-to-cv
```

Rendering needs [tectonic](https://tectonic-typesetting.github.io) and
poppler (`pdfinfo`, `pdftotext`):

```bash
brew install tectonic poppler
```

The CV uses the Charter typeface. If it is missing:
`brew install --cask font-charter`, or change `\setmainfont{Charter}` in
`templates/preamble.tex`.

## First run

```bash
mkdir ~/job-hunt && cd ~/job-hunt
claude
```
```
/payload-setup
```
Point it at your current CV (PDF, DOCX or text). It extracts the facts,
interviews you for the mechanisms behind each number, and writes:

```
job-hunt/
  payload/
    payload.md      ← the only place CV facts come from
    lanes.md        ← section-order presets per role type, salary bands, form answers
    header.tex      ← your name block
    header-cl.tex
```

Then paste a job description:

```
/jd-to-cv
<paste JD>
```

You get a fit score and a strategy brief, then "Go?". Say go and it
writes, renders and gates the PDF into `Applications/<date>_<Company>/`
and logs a row in `pipeline_log.csv`.

## Why a payload

CV generators hallucinate. This one cannot: the writer stage is only
allowed to read the strategy brief and `payload.md`. If a number, tool or
title is not in the payload, it does not exist. Prose is composed fresh
every time; facts never change.

## Layout

```
.claude-plugin/plugin.json      plugin manifest
.claude-plugin/marketplace.json lets this repo act as its own marketplace
skills/payload-setup/           setup skill + payload/lanes templates
skills/jd-to-cv/                the pipeline
templates/preamble.tex          CV design (fonts, spacing, \role \proj \stack)
templates/preamble-cl.tex       cover letter design
templates/header*.tex           name-block templates filled by payload-setup
templates/cv-template.tex       content skeleton the writer follows
```

Your data (`payload/`, `Applications/`, `pipeline_log.csv`) lives in your
working directory, never in this repo.

## Customising the design

Edit `templates/preamble.tex`. Keep `\role{}{}{}`, `\proj{}{}{}`,
`\stack{}` and `\section{}` names; the writer depends on them. Bold caps
section headers are deliberate: small caps break the PDF text layer that
ATS parsers read.
