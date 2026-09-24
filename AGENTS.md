# p2poolv2 meetings

Meeting notes for the p2poolv2 project (https://github.com/p2poolv2/p2poolv2)
and its deployment and management tool PDM (https://github.com/p2poolv2/pdm).
Notes are written in Emacs org-mode.

## Layout

- `YYYY-MM-DD.org` — one file per meeting, named by meeting date.
  - Weekly calls are on Thursdays, tagged `:weekly:`.
  - Other meetings (e.g. in person) use their own tag such as `:in-person:`.
- `template.org` — starting point for a new weekly call file.
- `template.md` — older markdown template; not used for new notes.
- `*.org~` — Emacs backup files; ignore them, never edit or commit them.

## Weekly call file structure

`** Agenda` (`- [ ]` checkboxes, ticked during the call), `** Updates`
(`*** In Progress`, `*** Merged / Shipped`, `*** New PRs Opened`,
`*** New Issues`, `*** Discussions`, `*** PDM (p2poolv2/pdm)`, `*** Blocked`), `** Discussion Notes`,
`** Decisions`, `** Action Items` (org table), `** Next Call`.

Conventions:
- Reference GitHub issues/PRs/discussions as `#123` (numbers under the PDM
  heading/agenda item refer to p2poolv2/pdm).
- Merged PRs are listed as CSV lines `- "num","title","author","date"`.
- Refer to people by their handle as used in previous notes.
- Don't invent attendees, decisions or discussion content — those are filled
  in by the user during or after the call.

## Preparing the agenda

Run `/weekly-agenda` (skill in `.claude/skills/weekly-agenda/`). It collects
GitHub activity from both repos since the previous meeting file with `gh` and fills today's
file. Don't commit unless asked.
