---
name: numbers-test-matrix-to-readable
description: Translate an Apple Numbers (.numbers) checkbox or "x" test-matrix workbook (quote lines, option grids, configuration sheets) into a readable canvas or plain-text export, and compare it against a translated PDF or other version. Use when the user uploads a .numbers file or asks to map one translated document against another.
---

# Numbers test matrix to readable

Use this for spreadsheets where each row is one configuration, each marked cell (usually `x`) selects one option, and numeric cells are dimensions.

## 1. Parse the workbook by cell index

- A `.numbers` file is a zip of IWA protobuf. Install the parser once: `pip install numbers-parser`.
- Open it with `from numbers_parser import Document`, then iterate `doc.sheets` and `sheet.tables`. For each table, read `table.rows(values_only=True)` and index by `(row, col)`.
- Do not dump rows as joined strings to scan visually. Empty cells disappear and columns shift, so a visual pass miscounts. Index-based reads are authoritative.
- The `Calibri-Bold` unsupported-font warning is harmless. Values still parse.
- Row 0 is often merged category headers and row 1 the option labels. Confirm this per workbook.
- Data rows are the rows that contain marks or dimensions. Ignore stray characters in empty columns (for example, a lone `f` in an otherwise empty green column).
- Numbers shows 1-based row numbers. Report line numbers as the sheet row you use and say so, for example "line n is row n + 2".

## 2. Build a column map before decoding

- Write out `column index -> category header -> option label` for every column with a header.
- Duplicate header names are common (for example two columns both named `Vent`). Tell them apart by position and by their category header, and record which one is which.
- A blank category header that is shaded as a new group still starts a new product. Check the cell fill and neighbors.
- For each data row, list the marked options per category. A category with no mark is information. Say so explicitly, for example "service not marked".
- Check exceptions: a row with two marks in an either/or group, a row missing a whole group, values that look wrong (for example an unusual height). Report them as entered. Do not correct them.

## 3. Write the readable output

- Group lines by product family. One sentence per line. Keep sheet spellings, including typos such as "Prarie" and "Glass Configuation". Do not silently fix them.
- State fields that are blank on every row once, in a note, instead of repeating them per line.
- Put the source sheet name and the last-saved date in the header.
- Deliverables: a canvas (WriteCanvas) when the user will scan or filter it. A plain-text file when they ask for an export. Commit to the repo only when asked.

## 4. Compare against a translated version

- Align lines by configuration (product, dimensions, colorway), not by line number. Translated documents are often numbered with an offset.
- Build a term map from the source wording to the translation wording. Cover product names, operation words, frame profile words, glass words, grille pattern words, and lite-count notation.
- Lite-count notation: a width-by-height grid of N x M lites usually equals `(N-1)v(M-1)h` (for example 2 x 2 = 1v1h, 3 x 4 = 2v3h).
- Report two lists separately: wording differences (same meaning, different words) and content differences (an attribute present on one side and missing on the other). Content differences are the ones that can change a quote.
- Flag every "?" or "ask about" in the translation as an open question. Do not resolve it silently.
- If the translation states a value the sheet does not mark, or drops a value the sheet marks, call it out as a conflict.

## 5. Verify before reporting

- Count the output lines and compare with the parser's count of data rows.
- Grep the output file for each line label to confirm none are missing.
- For a TSX canvas, check that braces balance and that only `cursor/canvas` is imported.

## Pitfalls seen so far

- Visual scanning of joined strings miscounted columns. Always index.
- Two columns with the same name are different products. Do not merge them.
- Line numbers in a translation can be offset from the sheet. Match on configuration.
- A blank service-request cell is meaningful. Do not treat it as missing data.
- A glass attribute marked on one row may be absent on another. Check each row's cells directly before stating a glass spec.
- Do not use a single shell heredoc or inline string to pass a large canvas source. Write it to a temp file and check its length first.

## Reference: Fortress Black-White and Bronze-Bronze test plan

- Sheet `CWS Black_White, Bronze_Bronze`: 22 window and door configuration rows (data rows 3 to 24). Odd lines black exterior with white interior, even lines bronze with bronze.
- Sheet `Mulls`: 13 mull lines (rows 3 to 15).
- Column groups: product, configuration, size (width, height, short side), facing, frame profile, operation, tracks, venting height, service, frame type, exterior and interior colors, glass configuration, strength, insulated glass, hardware, grille type, bar, grille colors, pattern, radius, spokes, hub, lites wide and high.
- Stored translation in this repo: `fortress-black-white-bronze-bronze-units.txt`.
