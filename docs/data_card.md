# Data card - ITFFC Part 1

## Source
- U.S. Department of Justice News API press releases (public).
- API: https://www.justice.gov/api/v1/press_releases.json
- Docs: https://www.justice.gov/developer/api-documentation/api_v1
- Frozen snapshot used: `releases_merged.json` (instructor shared file, everyone uses same file).

## Snapshot identity
- File: `releases_merged.json`
- Records (raw): 31450
- Records (deduped by `uuid`): 15586
- Size: 223.2 MB
- SHA-256: `d4f7fe58e46cf1b7c93e18f3064f1b1dfebdca12f5f6647a6f7d9d776d29e461`
- Period (from `date` unix): 2025-11-05T12:00:00+00:00 to 2026-10-06T12:00:00+00:00

## Schema (raw JSON per record)
- `title`, `date` (unix timestamp stored as text), `body` (HTML), `topic` (list of {name}), `component`/office (list of {name}), `url`, `number`, `uuid`, `teaser`, `created`, `changed`, `attachment`, `image`.

## Cleaning decisions
- QUALITY ISSUE: raw file contains ~2x duplicate `uuid`s (raw 31450 rows -> 15586 unique). Deduplicated by `uuid` keep-first; all stats below are deduped.
- `body` HTML cleaned with BeautifulSoup get_text; scripts/styles removed; whitespace collapsed. No other text altered.
- `date` parsed to ISO UTC + `year_month`. Invalid -> empty.
- `topic` / `component` flattened to `|`-joined name strings; missing -> empty string.
- `number` often empty (12463 missing) - kept as-is, not used for filtering.
- 16 records have empty body; kept with body_chars=0 and excluded from simple-case pool.
- Output: `releases_clean.csv` (15586 rows), `fraud_selected.csv` (subset).

## Fraud selection (justification)
- Rule: `fraud_topic` (topic string contains "fraud", case-insensitive) OR `fraud_title` (title contains "fraud").
- Counts: topic=2518, title=2061, union=3004 (19.273707173104068%).
- Why: topic tags cover Fraud, Financial Fraud, Healthcare Fraud, COVID-Related Fraud, Benefit Fraud, Securities/Investment Fraud. Title rule recovers fraud cases mistagged (e.g. "Local physician indicted in $30M COVID-19 fraud scheme" tagged only Tax). Kept deliberately narrow (title keyword = fraud only) to stay reproducible; broader keywords (scam/embezzle/Ponzi) left for Week 3+ discussion.
- Note: 2,299 of the 15,586 deduped records have no topic tag; they can only enter via the title rule. Of the 3,004 selected fraud cases, 486 were recovered by the title rule alone.

## EDA highlights
- Monthly volume + fraud share: see `figs/monthly_volume.png`, `figs/fraud_share.png`.
- Top topics: Drug Trafficking (2801), Operation Take Back America (2364), Firearms Offenses (2211), Project Safe Childhood (1896), Violent Crime (1526), Financial Fraud (1344), Immigration (939), Project Safe Neighborhoods (836).
- Body length: median 2517 chars; simple-case pool restricted to 800-4000 chars.
- Offices: top issuer [('Federal Bureau of Investigation (FBI)', 2716), ('Bureau of Alcohol, Tobacco, Firearms and Explosives (ATF)', 1236), ('Drug Enforcement Administration (DEA)', 931)].

## Facts sheet
- `facts_sheet.csv`: 30 simple fraud cases (short, contain >=1 dollar amount).
- Fields per case: uuid/title/date/office/topics/url/body_chars + regex-extracted `amounts_found`, `dates_found`, `stages_found` + full `body_text`.
- Regex: amounts `\$[\d,]+... (million|billion|thousand)?`; dates `Month DD, YYYY`; stages keyword list (indicted/charged/pleaded/guilty/convicted/sentenced/...).
- Purpose: ground truth for Weeks 3-4 generators + Weeks 5-6 claim checker. "not stated" policy applies at generation time.

## Limits / ethics
- Releases name real persons. Use only release text; never add outside info; do not speculate (per workshop instructions).
- Regex facts are noisy (amounts may include forfeiture/restitution duplicates; names not extracted yet). Checker must verify against source text, not against facts sheet alone.
- Instructor keeps a held-out evaluation set; never tune prompts/rules on it.
- Terms of use of DOJ source respected; cite source in final report. Test API use <=4 req/s, <=50/page with clear User-Agent.
