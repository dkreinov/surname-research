# surname-research

A [Claude Code](https://claude.com/claude-code) skill that researches a family surname end-to-end and produces a designed, cited HTML dossier — etymology, geographic origin, real archival records, reconstructed family trees, and (when the trail allows) the user's own documented direct line.

> All screenshots and examples in this repo use **fictional data** (surname "Silbermann"). The skill itself works on any surname; it is strongest for Ashkenazi / Eastern-European names because of its JewishGen integration.

![Mock dossier](docs/screenshots/full.png)

## What it does

Five phases, each feeding the next:

| Phase | What happens | Output |
|---|---|---|
| 1. Deep web research | Multi-agent fan-out over etymology, origin, notable bearers, variant spellings — every claim adversarially verified | verified findings + sources |
| 2. HTML dossier | Archival-dossier design (parchment/gold, serif, confidence chips), optional trilingual EN/HE/RU switcher, optional AI hero image, PDF export | `<surname>-report.html` |
| 3. Archive search | Drives the user's Chrome through JewishGen's Unified Search: revision lists, vital records, voter lists, business directories, burial registries, Holocaust datasets — with all the site's quirks encoded | `<surname>-records.md` |
| 4. Trees & chain-tracing | Reconstructs family nests per town; links records **across databases and cities** via patronymic + birth-year + spouse-name matching; solid = documented, dashed = labeled hypothesis | family-tree sections |
| 5. Fold-in | Findings merged back into the dossier; follow-up map (Yad Vashem, WW2 records, archive scans, emigration trails); skill self-updates with new pitfalls | updated report + next steps |

## Why it's more than a prompt

The skill encodes **hard-won operational knowledge** that would otherwise burn hours per session:

- JewishGen's login/profile/review gates and which databases each gate blocks
- Popup-blocker-proof form submission (`HTMLFormElement.prototype.submit` — the site shadows `.submit`), tightest-row selection in nested result tables
- Phonetic-search noise filtering (how to spot an unrelated family by patronymic style)
- **Chain-tracing**: matching a birth record in one city to a death record in another decades later — how Pale-of-Settlement families are actually followed to Kiev/Kharkov/Odessa
- The ~1911 civil-register horizon and how to state the gap to living generations honestly
- CSS-only family trees (no libraries — artifact CSP-safe), RTL Hebrew layout rules, headless-Chrome PDF export

## Install

Copy `SKILL.md` into your personal Claude Code skills directory:

```
~/.claude/skills/surname-research/SKILL.md
```

Then in any project: *"research the surname X"* — or invoke `/surname-research`.

Requirements: Claude Code with the Claude-in-Chrome extension (for the archive phase), a free JewishGen account (the skill walks the user through registration), optionally a text-to-image API key for hero images.

## Repo layout

```
SKILL.md                     # the skill — pipeline + pitfalls
examples/mock-report.html    # fictional dossier demonstrating the output design
docs/screenshots/            # renders of the mock example
```

## Honesty rules baked in

- Every tree edge is either **documented** (cites a record) or **dashed + labeled** with the inference reason
- Etymology without a direct dictionary entry is labeled as inference from sibling surnames
- Claims that fail adversarial verification are excluded and listed as refuted
- Record gaps (e.g. post-1911) are stated, never papered over

## License

MIT
