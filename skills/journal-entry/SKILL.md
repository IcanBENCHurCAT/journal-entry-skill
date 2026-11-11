---
name: journal-entry
description: Record an entry in an agent journal (for example the .jules/ persona journals bolt.md, palette.md, sentinel.md, tux.md used by Google Jules, or any markdown journal with dated headings). Use this every time you write a journal, changelog-of-learnings, or work-log entry. The bundled script stamps the real current date automatically, so entries are never misdated.
license: MIT
---

# Journal entries: never type the date yourself

Agents asked to head a journal entry with today's date will generate a
plausible date instead of reading the clock. In production repos this
produced entries dated weeks, months, and years in the future (and some
in the past), copied and compounded by later sessions reading the file.
The heading date must come from the system clock, not from the model.
`scripts/journal.py` (bundled with this skill) owns the date; you own
the words.

## Rules

1. **Never** write or edit a journal heading date by hand. Not today's
   date as you believe it to be, not a date copied from a neighboring
   entry, not an incremented date. If a date did not come out of
   `journal.py`, it is wrong by construction.
2. **Never** fix a wrong date by editing the heading yourself. Report
   it; corrections go through a reviewed change.
3. Journal content rules still apply: for example, performance claims
   need before/after measurements. The script fixes dates, not evidence.

## How to add an entry

Run the bundled script from the repo root. (Projects may also copy
`journal.py` into their own `scripts/` folder; either location works,
it resolves journals relative to the working directory.)

Formatted journals (Learning/Action style, e.g. bolt, palette):

```bash
python3 scripts/journal.py bolt \
  --title "Avoid X in hot path" \
  --learning "What you measured or discovered" \
  --action "What you changed and where"
```

Freeform journals (e.g. tux, sentinel): write the body to a temp file,
then:

```bash
python3 scripts/journal.py tux --title "Short title" --body-file /tmp/entry.md
```

The script appends to the end of `.jules/<persona>.md` (override the
journal directory with `--dir`):

```markdown
## <real today's date> - Short title

<body>
```

Match the journal's existing title convention (some wrap the title in
`[brackets]`); pass the title exactly as it should appear after the
date. There is deliberately **no `--date` option**.

## Verify afterwards

```bash
python3 scripts/journal.py --check-last <your-persona>
```

This fails if the latest entry in the journal is future-dated. Run it
before opening a PR.

(`--check` without `--last` scans every heading in every journal. Some
journals carry legacy future-dated entries from before this tooling
existed; those are tracked cleanup work, not yours to fix inside an
unrelated PR.)

## For repo maintainers

To make this mandatory in a repo, add a section like this to AGENTS.md:

```markdown
## Journal entries

- Journal entries in `.jules/` must be written with `scripts/journal.py`.
  Never type a journal heading date by hand.
- Before opening a PR, run `python3 scripts/journal.py --check-last
  <your-persona>` and make sure the latest entry is not future-dated.
```
