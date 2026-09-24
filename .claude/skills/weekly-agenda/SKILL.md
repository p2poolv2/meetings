---
name: weekly-agenda
description: Prepare this week's p2poolv2 weekly call agenda by pulling merged PRs, new PRs, new issues and discussions from github.com/p2poolv2/p2poolv2 and github.com/p2poolv2/pdm into today's YYYY-MM-DD.org file. Use when the user runs /weekly-agenda or asks to update/prepare the meeting agenda.
disable-model-invocation: true
argument-hint: "[YYYY-MM-DD meeting date, defaults to today]"
allowed-tools: Bash(gh *), Bash(date *), Bash(ls *), Bash(cat *), Bash(cp *), Read, Write, Edit
---

# Weekly agenda for p2poolv2

Meeting date: `$ARGUMENTS` if given, otherwise today (`date +%F`). The file is
`<date>.org` in the repo root. See `AGENTS.md` for the file conventions.

## 1. Find the reporting window

- Previous notes: the latest `YYYY-MM-DD.org` before the meeting date (ignore
  `*.org~` backups). Read it and use its date as the window start `S`.
- Also read any in-person / non-weekly notes in the window; issues created from
  them should be grouped under "From the <date> meeting" and recapped in the agenda.
- Read the previous file's "Next Call → Topics to carry forward" and any open
  `[ ]` agenda items; carry them into the new agenda.

## 2. Collect GitHub activity since `S`

Repos: `p2poolv2/p2poolv2` (main) and `p2poolv2/pdm` (deployment and
management tool). Run the queries below for each, with `R` and the graphql
`name:` set accordingly. For pdm also list *all* open PRs (`--state open`
without a date filter) since its PRs often wait weeks for review.

```sh
R=p2poolv2/p2poolv2
gh -R $R pr list --state merged --search "merged:>=$S" -L 100 --json number,title,author,mergedAt \
  | jq -r '.[]|[(.number|tostring),.title,.author.login,.mergedAt[:10]]|@csv'
gh -R $R pr list --state all --search "created:>=$S" -L 100 --json number,title,author,state,isDraft
gh -R $R pr list --state open --search "updated:>=$S" -L 100 --json number,title,author,isDraft
gh -R $R issue list --state all --search "created:>=$S" -L 100 --json number,title,author,state,createdAt
gh -R $R issue list --state closed --search "closed:>=$S" -L 100 --json number,title
gh -R $R release list -L 3
gh api graphql -f query='{repository(owner:"p2poolv2",name:"p2poolv2"){discussions(first:15,orderBy:{field:UPDATED_AT,direction:DESC}){nodes{number title createdAt updatedAt author{login}}}}}'
```

Keep discussions created or updated since `S`. Skim bodies (`gh api
repos/$R/discussions/<n> --jq .body`, `gh pr view <n>`) only as needed to write
a one-line summary.

## 3. Write the agenda

- If `<date>.org` doesn't exist, copy `template.org` to it.
- Set the heading timestamp to `<date Day>` (e.g. `<2026-09-24 Thu>`), and
  `Next Call` date to one week later. Leave `:ATTENDEES:` empty.
- Update the `# gh ...` comment above "Merged / Shipped" to use `merged:>=S`.
- Fill sections under `** Updates`:
  - `*** Merged / Shipped` — CSV lines `- "num","title","author","YYYY-MM-DD"` (existing style).
  - `*** New PRs Opened` — same CSV style with state instead of date; label
    dependabot PRs as `dependabot`.
  - `*** New Issues` — `- #num title`, grouped by theme or author (e.g. wallet,
    from in-person meeting, contributor name). Note issues closed in the window.
  - `*** Discussions` — `- #num title – one line summary (author, date)`.
- Add a `*** PDM (p2poolv2/pdm)` section before `*** Blocked` with: Merged,
  New PRs Opened, Open PRs awaiting review, New Issues (CSV style, "none" if empty).
- Fill `** Agenda` with `- [ ]` items: releases, PRs needing review, notable
  discussions, themed issue groups, carried-forward topics, and per-contributor
  items. Reference issue/PR numbers. End with a `- [ ] PDM` item listing its
  open PRs (`#num title (author, note)`) and a one-line status.
- Leave Discussion Notes, Decisions, Action Items, Blocked for the meeting.
- Preserve anything the user already wrote in the file; merge, don't clobber.

## 4. Report

Summarise what was added. Do not commit unless asked.
