# p2poolv2 meetings

Meeting notes for the p2poolv2 project (https://github.com/p2poolv2/p2poolv2)
and its deployment and management tool PDM (https://github.com/p2poolv2/pdm).
Notes are written in AsciiDoc (`.adoc`, edited with Emacs adoc-mode).
Notes before 2026-10-08 are org-mode files; leave them as they are.

## Layout

- `YYYY-MM-DD.adoc` — one file per meeting, named by meeting date.
  - Weekly calls are on Thursdays, with `:tags: weekly`.
  - Other meetings (e.g. in person) use their own tag such as `in-person`.
- `YYYY-MM-DD.org` — older notes (up to 2026-10-01); don't convert them.
- `template.adoc` — starting point for a new weekly call file.
- `template.org`, `template.md` — older templates; not used for new notes.
- `*~` — Emacs backup files; ignore them, never edit or commit them.

## Weekly call file structure

`== Agenda` (`* [ ]` checklist, ticked `* [x]` during the call),
`== Updates` (`=== In Progress`, `=== Merged / Shipped`, `=== New PRs Opened`,
`=== New Issues`, `=== Discussions`, `=== PDM (p2poolv2/pdm)`, `=== Blocked`),
`== Discussion Notes`, `== Decisions`, `== Action Items` (table),
`== Next Call`.

Conventions:
- Reference GitHub issues/PRs/discussions as `#123` (numbers under the PDM
  heading/agenda item refer to p2poolv2/pdm).
- PRs, issues and discussions under Updates are AsciiDoc tables
  (`[cols="1,2,8"]`, columns PR / Author / Title / description), one per
  `====` theme heading. The number links to GitHub; the third column is an
  `a|` cell with the bold title and wrapped summary bullets; rows carry no
  dates. See `template.adoc`.
- Refer to people by their handle as used in previous notes.
- Don't invent attendees, decisions or discussion content — those are filled
  in by the user during or after the call.

## Preparing the agenda

Run `/weekly-agenda` (skill in `.claude/skills/weekly-agenda/`). It collects
GitHub activity from both repos since the previous meeting file with `gh` and fills today's
file. Don't commit unless asked.
