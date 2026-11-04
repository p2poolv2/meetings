# p2poolv2 meetings

Meeting notes for the p2poolv2 project (https://github.com/p2poolv2/p2poolv2)
and its deployment and management tool PDM (https://github.com/p2poolv2/pdm).
Notes are written in Markdown (`.md`). Older notes are AsciiDoc
(2026-10-08 to 2026-10-29) or org-mode (up to 2026-10-01); leave them as
they are.

## Layout

- `YYYY-MM-DD.md` — one file per meeting, named by meeting date.
  - Weekly calls are on Thursdays, with `**Tags:** weekly`.
  - Other meetings (e.g. in person) use their own tag such as `in-person`.
- `YYYY-MM-DD.adoc`, `YYYY-MM-DD.org` — older notes; don't convert them.
- `template.md` — starting point for a new weekly call file.

## Weekly call file structure

See template.md

Run `/weekly-agenda` (skill in `.claude/skills/weekly-agenda/`). It collects
GitHub activity from both repos since the previous meeting file with `gh` and fills today's
file. Don't commit unless asked.
