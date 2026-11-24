# Cross-link and terminology ledger

## What is checked and why

This ledger checks whether readers can navigate the book, whether links and anchors resolve with exact path casing,
whether Markdown and Mermaid syntax parse, whether required IDs are unique, and whether unfinished-marker words hide
missing content. These checks matter because broken navigation and ambiguous terminology can make correct lessons
unusable, while clean syntax does not by itself prove technical accuracy.

## Status interpretation

- **Pass:** the automated check completed with zero actionable failures under the stated rules.
- **Pass with review remaining:** structure is valid, but a human check such as terminology or accessibility is still
  required.
- **Not yet performed:** there is no retained check for the current frozen candidate.

A passing link check proves that a target exists, not that its claim is correct. A passing lint check proves conformity
to formatting rules, not that prose is clear or code runs.

## Latest validation record

**Checked:** 2026-11-24. The final learner-first pass covered every Markdown file, every numbered top-level section,
all 47 chapters, and all 47 assessments. Automated checks used the shared configuration shown below; code and table
line length is excluded because commands, URLs, hashes, JSONL records, and wide comparison tables must remain intact.

<!-- markdownlint-disable MD013 -->
| Check | What counts as passing | Result | Interpretation |
| --- | --- | --- | --- |
| Repository Markdown lint | Configured rules report zero errors across all Markdown | 302 files checked; 0 errors | Pass |
| Links, anchors, path case, and navigation | Every Markdown target and fragment resolves with exact case; every file is reachable from the root learning map | 1,887 unique internal edges; 0 actionable failures; 302/302 reachable | Pass |
| Tables | Every detected table has a separator, consistent columns, and no unexplained empty result cell | 448 tables; 0 empty cells | Pass |
| Fences, headings, and equations | Fences balance, heading levels do not skip, inline math uses `$...$`, and display math uses standalone `$$` delimiters | 302 files; 525 display pairs and 1,657 inline pairs normalized; 0 failures | Pass |
| Structured listings | JSON, JSONL, YAML, and Python fences parse with their language parser | 317 listings: 116 JSON, 16 JSONL, 68 YAML, and 117 Python; 0 failures | Pass for syntax, not execution |
| Unfinished and replacement markers | No unresolved content/deployment token remains; literal HTML/prompt tags and language interpolation remain identifiable | 511 replacement tokens resolved; 0 unresolved markers | Pass |
| External links | Each unique HTTP(S) target receives a bounded HEAD/GET check | 1,016 unique URLs; 931 accessible; access-limited, local, illustrative, endpoint-only, and timeout cases separated | Pass with external access limits |
| Mermaid | Every Mermaid block renders with Mermaid CLI 11.9.0 | 121/121 diagrams rendered; 0 failures | Pass |
| Mandatory IDs | Each required table/figure ID has one canonical owner | 25/25 tables, 26/26 prompt visuals, and 7/7 CH-47 visuals retained | Pass |
| Terminology and prose | Marker, slang, status-language, acronym, and learner-sequence scans plus all-chapter review | Objective findings corrected; independent copy edit remains optional | Pass with editorial review remaining |
<!-- markdownlint-restore -->

The internal scanner checks Markdown destinations and fragments and builds a reachability graph from `README.md` and
`BOOKMAP.md`. External URLs are checked separately so anti-bot responses, local examples, and API endpoints are not
misreported as broken reader navigation.

## Reproducible lint command

```powershell
$cfg = Join-Path $env:TEMP 'slm.markdownlint-cli2.jsonc'
@'
{
  "config": {
    "default": true,
    "MD013": false,
    "MD024": { "siblings_only": true },
    "MD033": { "allowed_elements": ["a", "system", "user", "assistant", "intent-filter"] },
    "MD049": { "style": "asterisk" },
    "MD050": { "style": "asterisk" },
    "MD060": false
  }
}
'@ | Set-Content -Encoding utf8 $cfg
npx --yes markdownlint-cli2@0.19.0 --config $cfg "**/*.md" "#node_modules"
$exit = $LASTEXITCODE
Remove-Item $cfg
exit $exit
```

## Passing and failing evidence

**Pass:** a relative link resolves to the exact file and normalized heading anchor on a case-sensitive comparison; the
validator output and candidate hash are retained.

**Fail:** the link opens only on a case-insensitive workstation, points to a renamed heading, or is written inside a
table that the scanner silently skipped. Manual clicking of a few links is also insufficient for repository coverage.

For unfinished-marker review, a blank measured-result cell explicitly marked “not yet performed; populate from
`RUN-*`” passes. An unexplained token in a command, ownership field, or required result cell fails.

## How to close remaining gaps

Freeze and hash the publication candidate, then retain these command outputs with it. The remaining external work is
rendered accessibility review, optional independent copy/terminology review, and the real execution, device, AWS,
rights, safety, and domain reviews listed in the other ledgers.
