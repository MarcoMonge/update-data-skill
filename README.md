# update-data

A [Claude Code](https://claude.ai/code) skill that updates all datasets in a research project to their latest available versions.

## What it does

Given a list of raw data files and a README describing each dataset, the skill:

1. **Audits** every dataset — identifies the provider, current vintage, and whether it is an internal table that needs no update
2. **Downloads** the latest version from the original source, handling ZIPs, large files, PDFs that need extraction, and API endpoints automatically
3. **Reports** what was updated, what changed in coverage or time range, and gives step-by-step instructions for any datasets that require manual download (login walls, interactive forms, etc.)

## Installation

```bash
git clone https://github.com/MarcoMonge/update-data-skill.git ~/.claude/skills/update-data
```

On Windows:

```powershell
git clone https://github.com/MarcoMonge/update-data-skill.git "$env:USERPROFILE\.claude\skills\update-data"
```

## Usage

```
/update-data --list <list-file> --readme <readme-file> --raw-data <raw-data-folder> --output <output-folder>
```

All four arguments are required:

| Argument | Description |
|----------|-------------|
| `--list` | File listing all raw data files in the project (paths or glob patterns) |
| `--readme` | Project's data README describing each dataset, its provider, and reference URLs |
| `--raw-data` | Existing raw data folder — local path, cloud link (Google Drive, Dropbox), or both space-separated |
| `--output` | Destination folder for all updated files and the report |

## Output

The skill writes all updated files to `--output`, mirroring the original folder structure and preserving original filenames so downstream code does not break. It also saves `update_report.md` with a per-dataset summary, a files-downloaded table, and manual instructions for any blocked datasets.
