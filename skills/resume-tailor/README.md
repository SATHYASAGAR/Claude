# resume-tailor

A Claude skill that generates a **job-tailored, one-page resume**.

It reuses your most recent resume for formatting and for every section except **one focus experience entry** (usually your current/most-relevant employer). That focus section is rewritten from an accomplishments / STAR-stories document and tailored to a target job description — keeping bullets quantified, mirroring the JD's keywords for ATS, and never fabricating.

## Setup

Open `SKILL.md` and replace the placeholders in the **Configuration** table:

| Placeholder | Meaning |
|-------------|---------|
| `<RESUME_ROOT>` | Folder holding your resumes (and per-company subfolders) |
| `<STAR_STORIES_DOC>` | Your accomplishments / STAR-stories source doc |
| `<FOCUS_SECTION>` | The one experience entry to customize per job |
| `<RESUME_GLOB>` | Filename pattern matching your resumes (e.g. `Resume_*.docx`) |
| `<CANDIDATE_NAME>` | Name used in the output filename |

If you leave a placeholder unset, the skill will ask you for it on first run.

## Usage

Ask Claude something like *"tailor my resume for this job"* and paste or link the job description. The skill will pick your latest resume as the template, rewrite only the focus section to match the JD, verify it's one page, proofread it, save it, and report what it matched plus any gaps.

## Requirements

- A `docx` skill, or LibreOffice + `python-docx` + `pdfplumber` available in the environment (for editing, PDF conversion, and one-page validation).
