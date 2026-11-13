# Universal 5-Day Inter-Run Precision App

A Streamlit application that extracts numeric assay results from instrument PDF reports and populates a five-date precision workbook using an uploaded Excel template.

## Features
- Upload an original Excel precision template on the first run.
- Upload one or more instrument PDF reports.
- Reads the actual `Result Date/Time` from each result row; SID labels such as `INTER DAY2`, `DAY301`, and `DAY302` do **not** determine the workbook row.
- Groups results by analyte + actual calendar date + QC level.
- Selects up to five observations closest to the median within each analyte/date/QC-level group.
- Writes actual dates into the template's DATE column (five chronological date rows per QC level).
- Detects analyte headings dynamically rather than using a fixed analyte list.
- Separates Level 1 and Level 2 using the configured QC identifiers; unknown controls remain `Unassigned` for review.
- Preserves all extracted observations in `Raw_Data_Audit`, with an exact Yes/No flag identifying the five selected source rows.
- Creates `Upload_Summary` showing raw and populated replicate counts by actual date.

## Run locally
```bash
pip install -r requirements.txt
streamlit run app.py
```

## Deploy with GitHub + Streamlit Community Cloud
1. Create a GitHub repository.
2. Upload `app.py`, `requirements.txt`, `DUMMY_TEMPLATE.xlsx`, and `README.md` to the repository root (or configure the app's main file path accordingly).
3. Open https://share.streamlit.io/ and sign in with GitHub.
4. Choose **New app**, select the repository, branch `main`, and file `app.py`.
5. Deploy. Streamlit installs packages from `requirements.txt`.

## How to use
1. First run: upload your original `.xlsx` precision template.
2. Upload the PDF report(s). The app reads the result date from the PDF; the date selector is only a fallback when a result row has no date.
3. Click **Process PDFs** and download the populated workbook.
4. For later uploads, upload the latest populated workbook as the previous workbook, then upload the next PDF(s). Keep the latest downloaded workbook as the master file.

## Selection logic
For each analyte + actual date + QC level, the app calculates the median of all extracted numeric results and selects the five results with the smallest absolute distance from that median. Ties retain report order. If five or fewer results exist, all available results are populated. The audit sheet preserves every extracted row and marks the exact selected rows.

This is the requested selection rule, not an assertion that observations outside the selected five are invalid. Do not exclude results solely because they are statistically inconvenient; review the approved precision protocol and all raw observations.

## QC-level rules
Starter identifiers:
- Level 1: `101`, `301`, `AB1`, `K5972`, `6304961`
- Level 2: `102`, `302`, `AB2`, `K5973`, `6304962`

Other identifiers remain `Unassigned` and must be reviewed in `Raw_Data_Audit` before use. Modify `infer_level()` for a verified local QC naming convention.

## Important limitations
- The template has five date rows per QC level. If more than five distinct dates are detected for one analyte/level, the first five chronological dates are populated and a warning is shown.
- PDF layouts vary between instrument software versions; inspect the audit sheet.
- Formula recalculation is performed by Excel/LibreOffice when the downloaded workbook is opened.
- For regulated use, the application requires formal verification/validation, access control, versioning, and audit governance. This is an assisted extraction tool, not a validated laboratory information system.
