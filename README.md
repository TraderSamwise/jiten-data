# jiten-data

Pre-built dictionary assets for the Jiten app.

## Downloads

- **dictionary.db** — download from [Releases](https://github.com/TraderSamwise/jiten-data/releases)
- **[jlpt-words.csv](jlpt-words.csv)** — JLPT word classifications for all 22,575 common JMdict entries (Waller + audit + frequency). See [jlpt-words-README.md](jlpt-words-README.md) for methodology.
- **jlpt-audit-\*.csv** — audit override records with per-word reasoning. These are the provenance trail for corrections to Waller's JLPT classifications.
- **[counter-readings.csv](counter-readings.csv)** — verified counter word readings for 264 Japanese counters × 11 numbers (1–10 + 何) = 2,904 readings. See methodology below.
- **[noun-counter-mappings.csv](noun-counter-mappings.csv)** — maps 1,015 common Waller-set nouns to their appropriate counters (1,091 mappings across 88 counters). See methodology below.

## Counter readings methodology

264 counters were extracted from JMdict entries tagged with the "ctr" part-of-speech. For each counter, readings for numbers 1–10 and 何 were generated using phonological rules, then verified in two passes.

### Generation (pass 1)

A script (`scripts/build-counter-data.ts`) generated candidate readings by applying standard Japanese counter phonology:

- **Sokuon (っ)** for 1 (いち), 6 (ろく), 8 (はち), 10 (じゅう) before eligible consonants (k-row, s-row, t-row, h→p)
- **6 sokuon restriction**: only before k-row and h→p — NOT before s-row, t-row, or pure p-row loanwords
- **Rendaku** (h→b) for 3 (さん) and 何 (なん) before h-row
- **Half-voicing exceptions** (h→p instead of h→b) for specific counters like 泊(はく)

Generated readings were cross-referenced against JMdict's counted-form entries. Where JMdict had a match, that was used; where JMdict had a different reading, both were flagged for review. ~230 corrections were made in this pass.

### Manual audit (pass 2)

All 2,904 readings were reviewed in 7 batches of ~30 counters each, checking:

- Sokuon rules (especially the 6-before-s/t/p restriction)
- Rendaku and half-voicing rules
- Wago (native Japanese) numeral usage for counters that require it (e.g. 人: ひとり/ふたり, 日: ついたち/ふつか)
- Irregular readings for specialized counters (mahjong, traditional units)

11 additional corrections were made in this pass (0.38% error rate from pass 1).

### CSV columns

| Column | Description |
|--------|-------------|
| `counter_id` | JMdict sequence number |
| `counter_kanji` | Counter kanji/kana |
| `counter_reading` | Base reading of the counter |
| `counter_gloss` | English meaning |
| `number` | Number (1–10 or 何) |
| `number_kanji` | Kanji for the number |
| `combined_kanji` | Number + counter kanji |
| `generated_reading` | Reading from phonological rules |
| `jmdict_readings` | JMdict counted-form readings (if any) |
| `source` | `jmdict_match`, `jmdict_mismatch`, or `generated` |
| `notes` | Reasoning for mismatches or corrections |
| `verified_reading` | Final verified reading |

## Noun–counter mappings methodology

1,015 common nouns from the Waller set (5,303 total Waller nouns) were mapped to their standard Japanese counters.

### Generation

The 5,303 Waller-set nouns were split into 6 batches and each batch was classified against the full list of 264 counters. The prompt asked for only standard, natural counters — not generic fallbacks like つ.

This produced 1,279 candidate mappings.

### Manual audit

All 1,279 mappings were reviewed in 12 batches of ~100, checking for:

- **Wrong counter type**: e.g. 台 assigned to non-machines (stores, factories), 本 for non-cylindrical things (paintings), 枚 for non-flat things (terms)
- **Non-countable nouns**: abstract concepts (理解, 裏), na-adjectives (幼稚), adverbial expressions (一人一人, 種々)
- **Self-referential mappings**: counter expressions mapped to themselves (一人→人, 二人→人, 万人→人)
- **Missing counters**: entries referencing counter IDs not in the dataset
- **Wrong sense**: words with multiple meanings mapped using the wrong sense (品 as "elegance" → 品 counter)
- **Overuse patterns**: 台 for anything with a surface, 品 for non-food items, 人 for pronouns/collectives

188 mappings were removed (14.7%) and ~18 were corrected to different counters.

### CSV columns

| Column | Description |
|--------|-------------|
| `jmdict_id` | JMdict sequence number of the noun |
| `counter_id` | JMdict sequence number of the counter |
