---
name: weekly-agenda
description: Prepare this week's p2poolv2 weekly call agenda by pulling merged PRs, new PRs, new issues and discussions from github.com/p2poolv2/p2poolv2 and github.com/p2poolv2/pdm into today's YYYY-MM-DD.adoc file. Use when the user runs /weekly-agenda or asks to update/prepare the meeting agenda.
disable-model-invocation: true
argument-hint: "[YYYY-MM-DD meeting date, defaults to today]"
allowed-tools: Bash(gh *), Bash(date *), Bash(ls *), Bash(cat *), Bash(cp *), Bash(asciidoctor *), Read, Write, Edit
---

# Weekly agenda for p2poolv2

Meeting date: `$ARGUMENTS` if given, otherwise today (`date +%F`). The file is
`<date>.adoc` in the repo root (AsciiDoc). See `AGENTS.md` for the file
conventions.

## 1. Find the reporting window

- Previous notes: the latest `YYYY-MM-DD.adoc` or `YYYY-MM-DD.org` (notes
  before 2026-10-08 are org) before the meeting date (ignore `*~` backups).
  Read it and use its date as the window start `S`.
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

Keep discussions created or updated since `S`.

**Dedupe against the previous notes.** The window is inclusive of `S` so that
activity after last week's call is caught, which means items from before the
call on day `S` are already recorded. Grep the previous file for each number
(`"num"` or `#num`, in the matching repo's section) and:
- drop it if it's already listed with the same status (e.g. merged then and
  merged now, closed issue already noted, discussion with no new comments);
- keep it if its status changed (e.g. open last week → merged now), with a
  sub-bullet saying so;
- for PRs still open and already listed, keep them under "Open PRs awaiting
  review" marked `(carried over)`, summarising only what's new since `S`.

For every PR, issue and discussion that will be listed under Updates (both
repos), read its body *and* its conversation so you can summarise it — what it
does/why, plus where the review or discussion stands:

```sh
gh -R $R pr view <n> --json body,comments,reviews \
  --jq '.body, (.comments[]|select(.author.login!="codecov")|.author.login+": "+.body), (.reviews[]|select(.body!="")|.author.login+" "+.state+": "+.body)'
gh api repos/$R/pulls/<n>/comments --jq '.[]|.user.login+" "+.path+": "+.body'   # inline review comments
gh -R $R issue view <n> --json body,comments,stateReason
gh api graphql -f query='{repository(owner:"p2poolv2",name:"p2poolv2"){discussion(number:<n>){body comments(first:50){nodes{author{login} body}}}}}'
```

For dependabot PRs just note the bumped packages. Skip codecov/bot noise, but
do note the outcome of Copilot reviews (e.g. "all items addressed").

## 3. Write the agenda

- If `<date>.adoc` doesn't exist, copy `template.adoc` to it.
- Set the title to `= Weekly Call – <date> (<Day>)` (e.g. `2026-10-08 (Thu)`)
  and the `Next Call` date to one week later. Leave `:attendees:` empty.
- Update the `// gh ...` comment above "Merged / Shipped" to use `merged:>=S`.
- Fill sections under `== Updates` with AsciiDoc tables, one per group,
  in this shape (`|Issue` header for issues):

  ```adoc
  ==== Share chain work

  [cols="1,2,8"]
  |===
  |PR |Author |Title / description

  |https://github.com/p2poolv2/p2poolv2/pull/695[#695]
  |pool2win
  a|*Share identity binding using midstate trick from original p2pool*

  * What it does and why.
  * Review / discussion status, who said what, next step.
  |===
  ```

  - First column links the number to `/pull/<n>` or `/issues/<n>` in the
    matching repo (pdm links for PDM). Second column is the author handle;
    label dependabot as `dependabot`.
  - Third column is an `a|` cell: bold title, blank line, then 1–4 short
    bullets (nested `**` allowed) summarising the body and comments/review.
    Wrap source lines at ~78 cols. Escape a literal `|` in text as `\|`.
  - No dates in rows. For non-merged PRs the first bullet is the state
    (`Open.`, `Draft, carried over.`, `Closed in favour of …`).
  - Group rows under `====` theme headings (e.g. Share chain work,
    Dependencies PRs, Cleanup; for issues by theme, from in-person meeting,
    or contributor), each heading with its own table.
  - Sections: `=== Merged / Shipped`; `=== New PRs Opened` (with a
    `==== Open PRs awaiting review` table for carried-over open PRs);
    `=== New Issues` (note issues closed in the window);
    `=== Discussions` (link to `/discussions/<n>`).
  - For dependabot PRs the summary is just the bumped packages.
  - An empty section is a plain `* none` / `* No new ...` line, not a table;
    a placeholder bullet is `* {empty}`.
- Add a `=== PDM (p2poolv2/pdm)` section before `=== Blocked`: one-line
  `* Merged: none` style entries for empty parts, and `====` tables for
  Merged, New PRs Opened, Open PRs awaiting review and New Issues that have
  entries.
- Check it renders without warnings:
  `asciidoctor --failure-level WARN -o /dev/null <date>.adoc`.
- Fill `== Agenda` with `* [ ]` items, starting with `* [ ] Go through
  Updates`. **Do not repeat anything listed under Updates** (merged PRs, new
  PRs, new/closed issues, discussions, PDM items) — those are covered by going
  through Updates and repeating them is confusing. The agenda is only for
  things not in Updates: carried-forward topics, open questions from earlier
  meetings, older open PRs still waiting on review/action, follow-ups on
  issue groups from earlier meetings, stale-PR triage. End with
  `* [ ] PDM – go through PDM updates below`.
- Leave Discussion Notes, Decisions, Action Items, Blocked for the meeting.
- Preserve anything the user already wrote in the file; merge, don't clobber.

## 4. Report

Summarise what was added. Do not commit unless asked.
