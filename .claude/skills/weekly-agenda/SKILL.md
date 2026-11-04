---
name: weekly-agenda
description: Prepare this week's p2poolv2 weekly call file from template.md, pulling merged PRs, new PRs, new issues and discussions from github.com/p2poolv2/p2poolv2 and github.com/p2poolv2/pdm into the Since Last Call section of today's YYYY-MM-DD.md. Use when the user runs /weekly-agenda or asks to update/prepare the meeting agenda.
disable-model-invocation: true
argument-hint: "[YYYY-MM-DD meeting date, defaults to today]"
allowed-tools: Bash(gh *), Bash(date *), Bash(ls *), Bash(cat *), Bash(cp *), Bash(jq *), Read, Write, Edit
---

# Weekly agenda for p2poolv2

Meeting date: `$ARGUMENTS` if given, otherwise today (`date +%F`). The file is
`<date>.md` in the repo root (Markdown), created from `template.md`. See
`AGENTS.md` for the file conventions.

## 1. Find the reporting window

- Previous notes: the latest `YYYY-MM-DD.md`, `.adoc` or `.org` file before
  the meeting date (ignore `*~` backups and `template.md`). Read it and use
  its date as the window start `S`.
- From the previous file take:
  - its Action Items table → this file's **Last Week's Action Items**
    (Status left empty, filled during the call);
  - its Next Call topic of the week and facilitator → this file's
    **Topic of the Week** heading and **Facilitator** line;
  - its Next Call carry-forward items and any still-open `[ ]` items →
    **Input Wanted** if they fit there. Otherwise leave them out and list
    them in the report so the user can place them.
- Also read any in-person / non-weekly notes in the window and mention issues
  created from them in the report.

## 2. Collect GitHub activity since `S`

Repos: `p2poolv2/p2poolv2` (main) and `p2poolv2/pdm` (deployment and
management tool). Run the queries below for each, with `R` and the graphql
`name:` set accordingly.

```sh
R=p2poolv2/p2poolv2
gh -R $R pr list --state merged --search "merged:>=$S" -L 100 --json number,title,author
gh -R $R pr list --state all --search "created:>=$S" -L 100 --json number,title,author,state,isDraft
gh -R $R issue list --state all --search "created:>=$S" -L 100 --json number,title,author,state
gh api graphql -f query='{repository(owner:"p2poolv2",name:"p2poolv2"){discussions(first:15,orderBy:{field:UPDATED_AT,direction:DESC}){nodes{number title createdAt updatedAt author{login}}}}}'
```

Keep discussions created or updated since `S`.

**Dedupe against the previous notes.** The window includes day `S`, so items
from before last week's call may already be recorded. Grep the previous file
for each number (in the matching repo's section) and drop it if it's already
listed with the same status. A PR listed as new last week and merged now goes
under Merged.

You don't need to read PR/issue bodies or comments: attendees read the items
themselves before the call.

## 3. Write the file

- If `<date>.md` doesn't exist, copy `template.md` to it.
- Title `# Weekly Call – <date> (<Day>)` (e.g. `2026-11-05 (Thu)`), Next
  Call date one week later. Leave Attendees and Note-taker empty.
- Keep the template's Agenda, section order, and `<!-- -->` comments as they
  are.
- Fill **Since Last Call**. Every item is one line: link with number, title,
  author handle in parentheses. No descriptions, summaries, or dates.

  ```md
  ### Merged

  - [#695](https://github.com/p2poolv2/p2poolv2/pull/695) Share identity binding using midstate trick (pool2win)
  ```

  - `### Merged`: PRs merged since `S`.
  - `### New PRs`: PRs created since `S` and not merged; append ` – draft`
    or ` – closed` when it applies.
  - `### New Issues`: issues created since `S`; append ` – closed` when it
    applies.
  - `### Discussions`: link to `/discussions/<n>`; append ` – new` for ones
    created since `S`.
  - `### PDM`: same three lists for p2poolv2/pdm, as `- Merged:`,
    `- New PRs:`, `- New Issues:` with items nested under each (or `none`
    on the same line).
  - Links go to `/pull/<n>`, `/issues/<n>` or `/discussions/<n>` in the
    matching repo. Label dependabot as `dependabot`.
  - An empty list is `- none`.
  - Update the `gh` command in the section's comment to use `merged:>=S`.
- Leave TL;DR, Open Questions and Comments, Input Wanted outcomes, Take to GitHub
  and Action Items for the call. Don't invent attendees, discussion or outcomes.
- Preserve anything the user already wrote in the file; merge, don't clobber.

## 4. Report

Summarise what was added, and list any carry-forward items you couldn't
place. Do not commit unless asked.
