# Lanes & User Context
## <<Full Name>>

A lane is a preset for section order + summary voice + which stories to
foreground. The Stage 2 brief picks a lane or synthesises one.

## Lane Selection

| Role keywords in JD title | Lane |
|---|---|
| <<AI Engineer, ML Engineer>> | <<AI Engineer>> |
| <<BI Analyst, Reporting, Power BI>> | <<BI / Reporting>> |
| <<Data Analyst, Business Analyst>> | <<Data Analyst>> |
| <<Graduate, Junior, Associate>> | <<Graduate>> |

## Per-Lane Rules

### <<Lane name>>
- **Order:** Summary → <<Skills → Experience → Projects → Education → Certs>>
- **Summary voice:** <<builder / commercial / analyst / MSc+outcome>>
- **Experience ordering:** reverse chronological, current role always first.
  <<Optional: relevance-forward ordering for roles below the current one.>>
- **Publications:** <<always / skip / only if research-adjacent>>
- **Story emphasis:** <<which STORY shortcodes get 4-5 bullets, which get 2>>
- **Projects:** <<2-3 projects, 3 bullets each (4 for AI roles)>>

### <<Next lane>> ...

## Page Target

- CV length: <<1 or 2 pages>> (2 is the usual choice past three years of experience; 1 for graduates or where the market expects it)
- Cover letter: 1 page

## Cross-Lane Rules (apply regardless of lane)

- Bullets fit ONE rendered line: hard limit ~112 characters per `\item`
  excluding markup. Check: `grep -n 'item' content.tex | awk 'length($0)>125'`
- Full-time roles: 4 bullets minimum. Internships: 2 minimum. Regardless of JD relevance.
- Match the JD's spelling variant (US vs British) across the whole CV.
- Skills section: curate per JD, ~5-6 categories, never dump the library.
- Skills placement is JD-shaped: checklist JD → Skills right after Summary;
  narrative JD → lane default.
- Never place an ended role above the current role.
- Experience entries carry a `\stack{}` line with the tools used in that role.

## Salary Bands (for application forms; anchor mid-band as a range)

| Role type | Band |
|---|---|
| <<Data Analyst>> | <<€xx–xxk>> |
| <<AI Engineer>> | <<€xx–xxk>> |

## Standard Form Answers

- Visa / right to work: <<"No sponsorship required" / "Requires sponsorship">>
- Notice period: <<...>>
- Preferred work mode: <<hybrid / remote / on-site>>

## Learning Loop Notes (the pipeline appends here after outcomes)

- <<date: angle X on role-type Y → interview / rejected>>
