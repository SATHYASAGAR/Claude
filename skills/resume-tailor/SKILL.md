---
name: resume-tailor
description: Create a job-tailored, one-page resume. Use whenever the user wants to tailor, customize, or generate a resume for a specific job/company, or asks for a resume for a new role. Reuses the user's latest existing resume for formatting and all sections except one focus experience section, which is rewritten from an accomplishments/STAR-stories doc to match the target job description.
---

# Resume Tailor

Generate a one-page resume tailored to a specific job description. Everything except **one focus experience section** is reused verbatim from the user's most recent resume (for identical formatting and content). The focus section is rewritten from an accomplishments/STAR-stories source and tailored to the target job description.

> **Setup — replace these placeholders before use.** Edit the values in the "Configuration" block below (or, on first run, ask the user for any that are unset). Everything else in this skill is generic.

## Configuration

| Placeholder | What it is | Example |
|-------------|-----------|---------|
| `<RESUME_ROOT>` | Folder that holds the user's resumes (and per-company subfolders) | `~/Documents/Resumes` |
| `<STAR_STORIES_DOC>` | Path to the accomplishments / STAR-stories source for the focus section | `<RESUME_ROOT>/STAR-stories.docx` |
| `<FOCUS_SECTION>` | The one experience entry to customize per job (usually current/most-relevant employer) | `BILL` |
| `<RESUME_GLOB>` | Filename pattern that matches the user's resumes | `Resume_*.docx` |
| `<CANDIDATE_NAME>` | Name used in the output filename | `Jane_Doe` |

On first use, if any placeholder is still literally `<...>`, ask the user for it (a one-time setup), then proceed.

## Fixed sources (do not ask the user for these once configured)

- **Resume root folder:** `<RESUME_ROOT>`
- **Focus-section content source (always):** `<STAR_STORIES_DOC>`
- **Formatting / template source:** the **latest** resume matching `<RESUME_GLOB>` in `<RESUME_ROOT>` or its subfolders (most recently modified). Reuse all of its content as-is **except** the `<FOCUS_SECTION>` experience bullets.

## What to ask the user

1. **Job description** — required. Ask them to paste it or give a file path (`.pdf`, `.docx`, `.txt`, or a URL). Do not proceed without it.
2. **Company / role name** — used for the output filename. If a JD file is given, infer it and confirm.
3. (Optional) Confirm which resume to use as the template if there are several recent ones; default to the most recently modified.

## Workflow

### 1. Locate the template resume
List files matching `<RESUME_GLOB>` across the root and subfolders, sorted by modification time, and pick the newest (skip lock files starting with `~$`). Confirm the choice with the user if ambiguous.

```bash
cd "<RESUME_ROOT>"
ls -t **/<RESUME_GLOB> 2>/dev/null | grep -v '~\$'
```

### 2. Read the three inputs
- Extract the JD (use `pdfplumber` for PDF, `python-docx` for docx, plain read for txt). Pull out: required skills, technologies, responsibilities, security/compliance asks, and recurring keywords.
- Extract the template resume's full text and **run-level formatting** of the focus-section bullets (see structure below).
- Skim `<STAR_STORIES_DOC>` for `<FOCUS_SECTION>` accomplishments relevant to the JD.

### 3. Identify the focus section structure in the template
The `<FOCUS_SECTION>` experience block runs from its role/company header to the next role header. It typically contains:
- An optional `Normal` "Languages/Tools:" line (run[0] = label, run[1] = the tools list).
- A series of `List Paragraph` bullets, each with **run[0] = bold lead-in label** (e.g. `"Architecture leadership: "`) and **run[1..] = normal description text**.

Inspect the actual runs first — formatting conventions vary between resumes.

### 4. Rewrite the focus-section bullets, tailored to the JD
- Keep roughly the same number of bullets as the template so it stays one page.
- Reorder and reword so bullets **lead with the skills, technologies, and responsibilities the JD emphasizes**; mirror the JD's terminology for ATS without keyword-stuffing.
- Keep every claim **quantified** (metrics, scale, impact) and lead with strong action verbs.
- **Never fabricate.** Only use facts grounded in `<STAR_STORIES_DOC>` / the existing resume. If the JD names a skill the user lacks in the focus role, leave it out and report it as a gap.
- Optionally retune the focus section's "Languages/Tools:" line to surface JD-relevant tech first (only tech that is actually true).

### 5. Build the output by editing the template (preserves formatting exactly)
Load the template with `python-docx`, then for each focus-section bullet set `run[0].text` = new bold label and `run[1].text` = new description, and blank out any extra runs (`run[2:].text = ''`). Do **not** touch any other paragraph. This guarantees identical fonts, spacing, margins, and section styling.

```python
from docx import Document
d = Document(TEMPLATE_PATH)
ps = d.paragraphs
# find the <FOCUS_SECTION> header index and the next role header index;
# edit only the bullets between them:
#   tools line:  ps[tools_idx].runs[1].text = "<tailored tools>"
#   each bullet: runs[0].text = label; runs[1].text = desc
#                for r in runs[2:]: r.text = ''
d.save(OUTPUT_PATH)
```

### 6. Verify
- Convert to PDF and confirm it is exactly **one page**:
  ```bash
  python3 .claude/skills/docx/scripts/office/soffice.py --headless --convert-to pdf OUTPUT.docx
  python3 -c "import pdfplumber; print(len(pdfplumber.open('OUTPUT.pdf').pages))"
  ```
  If it spills to a second page, trim wording (don't shrink fonts/margins — keep the template's exact styling).
- Render `pdftoppm -jpeg -r 110 OUTPUT.pdf pg` and read the image to confirm layout and content.
- Proofread the full text for spelling, grammar, parallel verb structure, and consistent punctuation.

### 7. Deliver
- Output filename: `<CANDIDATE_NAME>_Resume_<Company>.docx`.
- Save it into the company's subfolder under `<RESUME_ROOT>` (create the subfolder if needed), and present it.
- Then report: which JD requirements were matched (and where), plus any gaps the user should be aware of.

## Notes
- The output must stay strictly one page with formatting identical to the template.
- Only the `<FOCUS_SECTION>` experience section changes; all other sections (Summary, Skills, other roles, Education, contact) are reused verbatim.
- This skill assumes a `docx` skill (or LibreOffice + python-docx + pdfplumber) is available for editing, PDF conversion, and validation.
