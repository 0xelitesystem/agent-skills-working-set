---
name: csv-cleaner
description: Use when the user provides a CSV or TSV file that is broken, malformed, or messy. Triggers on phrases like "fix this CSV", "clean up this data", "this CSV won't import", or when a file with .csv/.tsv extension is shared and the contents show obvious problems (mixed delimiters, encoding artifacts like ÃÂ, ragged rows, headers in the middle, etc.).
---

# CSV Cleaner

Detect and fix common CSV problems without losing data.

## Process

1. **Inspect the first 200 lines.** Don't load the whole file blindly. Look for:
   - Encoding artifacts (`ÃÂ`, `â€™`, `\ufffd`) → likely UTF-8 read as Latin-1 or vice versa
   - Mixed delimiters (commas in some rows, tabs in others) → file is mislabeled
   - Quoted fields containing the delimiter
   - Headers appearing more than once (file is concatenated dumps)
   - Trailing commas (Excel quirk)
   - BOM at start of file

2. **State the diagnosis to the user before changing anything.** "I see X, Y, Z problems. I will fix them by doing A, B, C. Confirm or tell me what to skip."

3. **Fix in this order:**
   - Re-decode encoding if artifacts present (try cp1252 → utf-8, latin-1 → utf-8)
   - Strip BOM
   - Detect actual delimiter via `csv.Sniffer` or character-frequency analysis
   - Re-parse with the correct delimiter and quote character
   - Drop duplicate header rows (keep first)
   - Pad ragged rows with empty strings rather than dropping them; flag the row count
   - Trim whitespace from cell values but NOT from quoted strings

4. **Type inference (only if user requested):**
   - For each column, sample 100 rows
   - Try int → float → date (ISO, US, EU formats) → bool → string
   - If >95% of non-empty values fit a type, suggest that type
   - Never coerce silently; report the inference and let the user confirm

5. **Output:** save as a NEW file (don't overwrite the original). Default name: `<original>_cleaned.csv`. Always UTF-8, always LF line endings, always RFC 4180 quoting.

## What NOT to do

- **MUST NOT** silently drop rows. Pad them or flag them.
- **MUST NOT** rename columns unless the user asks.
- **MUST NOT** change cell values (other than re-encoding) unless explicitly requested.
- **MUST NOT** assume the first row is a header without checking. Ask if the file has no consistent header.

## Edge cases worth flagging

- Files larger than 100MB: ask if streaming is okay before loading into memory
- Files with no clear delimiter (likely fixed-width): say so and ask the user to confirm
- Files where the header has fewer columns than the data: data row with the most columns sets the count
