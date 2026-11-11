# journal-entry

An agent skill that stops AI coding agents from misdating their own
journal entries.

## The problem

Repos that let agents keep a work journal (the `.jules/` persona
journals used with Google Jules are the canonical example: `bolt.md`,
`palette.md`, `sentinel.md`, `tux.md`) all hit the same failure. The
agent is told to head each entry with today's date, and instead of
reading a clock it generates a plausible one. Observed in the wild,
in a single production repo:

- entries dated **2027-01-10** and **2027-01-15**, written in 2026
- entries dated three weeks in the future, repeatedly (a whole run of
  them: Nov 15, 20, 25, 26, written in October and early November)
- entries backdated to **2025** and **2023**
- wrong dates copied and compounded by later sessions, because each
  new agent treats the existing headings as the pattern to continue

It is not a timezone bug. A clock skew is off by a day; this is off by
whatever makes a convincing-looking heading. To a language model, the
date is prose, not data, and prose gets autocompleted.

Downstream, the bad dates poison everything that reads the journal as
a timeline: audits, "what changed this week" summaries, and automated
guards. (The repo this was built for ended up adding a merge-time
guard that rejects PRs whose journal entries are future-dated. The
guard kept firing, because the agents kept doing it.)

## The fix

Take the date away from the model.

`scripts/journal.py` appends the entry and stamps the heading from the
system clock (America/New_York) at run time. The agent supplies only a
title and the entry text. There is deliberately **no `--date` option**:
a date the agent can pass is a date the agent can invent.

```bash
# Learning/Action style journals
python3 scripts/journal.py bolt \
  --title "Avoid X in hot path" \
  --learning "What you measured or discovered" \
  --action "What you changed and where"

# Freeform journals
python3 scripts/journal.py tux --title "Short title" --body-file entry.md

# Pre-PR sanity checks
python3 scripts/journal.py --check-last tux   # latest entry not future-dated
python3 scripts/journal.py --check            # audit every heading
```

The bundled `SKILL.md` teaches the agent the rules (never type a date,
never hand-fix one, verify with `--check-last`), and includes an
AGENTS.md snippet so maintainers can make script use mandatory.

## Install

As an Agent Skill (skills.sh ecosystem):

```bash
npx skills add IcanBENCHurCAT/journal-entry-skill
```

Or copy `skills/journal-entry/` into your agent's skills directory, or
just copy `scripts/journal.py` into your repo and point your AGENTS.md
at it. The script is dependency-free Python 3 (stdlib only).

## Provenance

Built and battle-tested in
[IcanBENCHurCAT/ipfs-pay-to-pin](https://github.com/IcanBENCHurCAT/ipfs-pay-to-pin),
the repo with the worst journal-date record: the tooling landed there
first (PR #439), and a Jules session immediately used it end to end to
log a correctly dated entry (PR #440, `## 2026-11-11`, written on
2026-11-11).

## License

MIT
