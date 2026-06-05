---
name: update-data
description: Updates a research project's datasets to their latest available versions. Use this skill whenever the user wants to refresh, update, or download new versions of research datasets, mentions needing to update their raw data folder, or asks to check which datasets have newer versions available. Invoke it with /update-data --list <list-file> --readme <readme-file> --raw-data <local-path-or-cloud-link> --output <output-folder>.
---

# update-data

Help a research team update all the datasets in a project to their latest available versions. This is a careful, three-phase process: first understand exactly what data exists and what each dataset is, then find and download updates, then produce a detailed report.

## Invocation

```
/update-data --list <path-to-list-file> --readme <path-to-readme> --raw-data <local-path-and-or-cloud-link> --output <output-folder>
```

All 4 arguments are required.
- `--list`: a file listing all raw data files in the project (with paths or glob patterns)
- `--readme`: the project's data README, which describes each dataset, its provider, and reference URLs
- `--raw-data`: the existing raw data folder. Can be a local disk path, a cloud link (Google Drive, Dropbox, etc.), or both space-separated. Access all sources provided.
- `--output`: destination folder for all updated files and the report

---

## Phase 1 — Audit (understand before acting)

Read and fully internalize all inputs before touching anything:

1. **The `--list` file**: every dataset the project uses, including file paths and glob patterns
2. **The `--readme` file**: treat this as your primary knowledge base. It describes each dataset's provider, content, geographic coverage, and has reference URLs. Important: README URLs are *starting points*, not authoritative download links — they may be outdated.
3. **The `--raw-data` reference**: inspect the actual files (local) and/or access the cloud link to understand the real folder structure, subfolder layout, file naming conventions, column names, and current data vintage. Use both the README and the actual files — they complement each other.

For each dataset in the list, determine:
- Provider (OECD, World Bank, BLS, Census Bureau, etc.)
- Current version/vintage on disk (from filenames, internal headers, README descriptions, file metadata)
- Geographic/country coverage currently in the data
- **Whether it is an internal table**: correspondence tables, crosswalks, and classification files created by the research team should be skipped entirely (no update needed). Identify these using two signals:
  - *Primary*: the README says it was created internally
  - *Secondary*: inspect the file — if it has no external provider, no URL in the README, and looks like a short lookup table (codes ↔ names, no time dimension), ask the user before skipping

Write a short internal checkpoint (a few bullet points per dataset) summarizing what you found in Phase 1 before proceeding to Phase 2.

---

## Phase 2 — Research and Download

For each non-internal dataset, find the latest version and download it. Work through them sequentially.

### The 3-strike rule: exhaust all options before declaring "manual download required"

You must attempt all three steps below before concluding a dataset cannot be downloaded automatically:

**Strike 1 — Explore from the README URL**
The README URL is a starting point, not the answer. From it, actively explore: navigate from landing pages into download sections, try URL pattern variations (change `2017` → `2023` in the path, `ICIO2021` → `ICIO2025`, etc.), follow "download", "latest version", or "most recent" links one level deep. Many datasets that seem hard to find are actually one click away from the README URL.

**Strike 2 — Broader web search**
If Strike 1 fails, search broadly for the dataset's current location. The provider may have reorganized their website. Search using the dataset name, provider, and indicator code from the README.

**Strike 3 — Programmatic/API access**
Try API endpoints, direct file URLs, bulk download paths, or programmatic access (requests, wget-style fetch). Many providers (World Bank, BEA, BLS, Census Bureau) have stable API or bulk download URLs even when their web portals change.

Only after all three strikes fail — or when the source *structurally* requires it (login wall with credentials, IPUMS custom extract, interactive multi-step web form) — may you declare "requires manual download." **Before finalizing that decision, ask the user**: "I've tried three approaches and couldn't download [dataset] automatically. Do you agree it requires manual download, or should I try something else?"

### Handling specific cases

**ZIP files**: Download and extract automatically. After extraction, keep only the files that correspond to datasets in the `--list`, guided by the README and the raw data folder structure. Don't dump the entire ZIP contents.

**Large files**: Always attempt downloads regardless of file size. Never skip a download because it's large — some datasets the team needs are genuinely multi-GB.

**Files not on local disk** (e.g., stored on Google Drive): The team may not have a local copy. Still download the updated version to the output folder. Use the README and the cloud link in `--raw-data` to understand the current version.

**PDF-to-structured-data**: When the original raw data file is a `.csv`, `.xlsx`, or `.dta` that was derived from a PDF report (signal: the raw data folder contains a processed file where only a PDF exists at the source, or the README describes the data as coming from a PDF/report):
1. Download the latest PDF
2. Check if `pdfplumber` is installed in the active conda environment. If not: `pip install pdfplumber`. Try this first.
3. If pdfplumber fails: check/install `tabula-py` (`pip install tabula-py`, requires Java) and try that.
4. Extract the relevant table(s) and produce a structured file matching the **exact column structure** of the original file — same column names, same units, same format. The original file is your template.
5. Only if both libraries fail should you consider this manual — and ask the user first.

Write a short internal checkpoint after Phase 2 (what was downloaded, what needs manual action) before writing the report.

---

## Phase 3 — Report

Save as `update_report.md` inside the `--output` folder. Follow this structure exactly:

```markdown
# Database Update Report

**Report generated:** [YYYY-MM-DD]
**Project:** [inferred from README title or raw data folder name]
**Updated files saved to:** `[output folder path]`

---

> **Note on internal tables:** Files [list them] are internal correspondence/classification tables created by the research team. No update check was performed for these files.

---

## [N]. [Dataset Name]
**List items:** [item numbers and filenames from --list]

- **Provider:** ...
- **Version on disk:** ...
- **Latest available version:** ...
- **Updated:** Yes / No
- **Download URL:** [exact URL used to download, or "N/A"]
- **Temporal scope note:** [only if relevant — e.g., "Original covered 2015–2019; updated file covers 1997–2025 — researcher should decide whether to trim"]
- **Coverage changed:** Yes / No
  - **Countries/areas added:** [list or "none"]
  - **Countries/areas removed:** [list or "none"]

---

## Summary Table

| # | File | Provider | Version on Disk | Latest Available | Updated | Downloaded |
|---|------|----------|-----------------|------------------|---------|------------|

---

## Files Downloaded to `[output folder]`

| File | Source | Size |
|------|--------|------|

---

## Requires User Action

The following datasets need manual steps. Complete in order:

1. **[Dataset name]** — Navigate to [exact URL] — [step-by-step instructions for a research assistant to follow]
```

---

## Output folder structure

Mirror the original raw data folder exactly:
- Reproduce the same subfolder hierarchy (e.g., `SAGDP/`, `Agriculture_Census/`, `Fips/`)
- **Keep the original filenames** from the raw data folder (e.g., `data_agriculture.csv`, not the source's native filename). Downstream Stata and Python code hardcodes these names — renaming breaks things.
- Save the full updated file without trimming to the original time window. If the original was trimmed, note the scope difference in the report and let the researcher decide.

---

## What not to do

- Don't skip a dataset because its README URL is broken or because the file is large
- Don't declare "manual download required" without exhausting all 3 strikes and confirming with the user
- Don't rename files away from the original naming convention
- Don't trim data to the original time window
- Don't skip ZIP extraction or PDF-to-CSV conversion
- Don't skip or flag an internal table without checking both the README and the file itself
