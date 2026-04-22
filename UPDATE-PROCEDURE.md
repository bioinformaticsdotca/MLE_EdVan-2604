# MLE_EdVan-2604: Updating the GitHub Repository and Website

This document describes the complete workflow for making changes to the workshop website at `https://bioinformaticsdotca.github.io/MLE_EdVan-2604/`, from editing source files to getting changes live on the site.

---

## Overview

The site is built with **bookdown** (an R-based static site generator) and published via **GitHub Pages**.

**Key architecture facts:**
- **Source files** (`.Rmd`, `.csv`, etc.) live at the repo root — you edit these
- **Output files** (`.html`, `.css`, etc.) live in `docs/` — GitHub Pages serves this folder as the website root
- **GitHub Pages source:** `main` branch, `/docs` directory
- **URL mapping:** A file at `docs/content-files/Workshop_location_in_Edmonton.pptx` is accessible at `https://bioinformaticsdotca.github.io/MLE_EdVan-2604/content-files/Workshop_location_in_Edmonton.pptx` (the `docs/` prefix is dropped because GitHub Pages uses `docs/` as the site root)
- The site uses the **legacy GitHub Pages build type**

---

## Critical Rule: Don't Edit `docs/` Manually

**The `docs/` directory is auto-generated.** Never edit HTML, CSS, or other files in `docs/` directly. All changes must be made to the **source files** (`.Rmd`, `_output.yml`, `style.css`, etc.) and then the site must be **rebuilt** with bookdown. The only exception is adding large binary files (see Adding Downloadable Files below).

**Note on `.nojekyll`:** The file `docs/.nojekyll` must be present (it is already in the repo). It tells GitHub Pages not to run Jekyll processing on the `docs/` folder, which is required for the site to work correctly. Do not remove it.

---

## Standard Update Workflow

### Step 1: Edit the Source Files

Edit the appropriate `.Rmd` file or other source file. The main files you will modify:

| File | Purpose |
|---|---|
| `index.Rmd` | Front page content (welcome text, schedule, pre-work link) |
| `001-faculty.Rmd` | Faculty and TA bios |
| `003-logistics.Rmd` | Logistics parent page |
| `003-logistics.Rmd` | Contains Edmonton location info (no separate page) |
| `005-vancouver.Rmd` | Vancouver venue/location info |
| `010-module-1.Rmd` – `080-module-8.Rmd` | Module content pages |
| `schedule.csv` | Workshop schedule data |
| `style.css` | CSS styling |
| `_output.yml` | Sidebar, header, and theme configuration |
| `_bookdown.yml` | Bookdown output directory configuration |

### Step 2: Rebuild the Bookdown Site

After editing source files, you must rebuild the site to regenerate the HTML in `docs/`.

**Prerequisites:** R with the `bookdown`, `readr`, `jsonlite`, and `htmltools` packages installed. If packages are missing, install them with:
```r
install.packages(c('bookdown', 'readr', 'jsonlite', 'htmltools'), repos='https://cloud.r-project.org')
```

**Rebuild command:**
```bash
cd /path/to/MLE_EdVan-2604
Rscript -e "bookdown::render_book('index.Rmd', 'bookdown::gitbook')"
```

This will regenerate all HTML files in `docs/` based on the current `.Rmd` source files.

### Step 3: Commit and Push All Changes

The workflow requires pushing **both source files and the rebuilt `docs/` files** to GitHub. GitHub Pages serves the `docs/` folder, so the HTML changes won't appear on the website until `docs/` is pushed.

```bash
git add <modified-source-files> docs/
git commit -m 'description of changes'
git push origin main
```

For example, if you edited `schedule.csv` and `index.Rmd`:
```bash
git add schedule.csv index.Rmd docs/
git commit -m 'update schedule with new times'
git push origin main
```

### Step 4: Wait for GitHub Pages to Rebuild

GitHub Pages will automatically rebuild after a push. This typically takes 30–60 seconds. You can monitor the build status via:
```bash
gh api repos/bioinformaticsdotca/MLE_EdVan-2604/pages
```

To manually trigger a rebuild:
```bash
gh api -X POST repos/bioinformaticsdotca/MLE_EdVan-2604/pages/builds
```

### Step 5: Verify the Changes

Check that the website reflects your changes. Use `curl` to confirm files are accessible — look for `HTTP/2 200`:

```bash
# Check the homepage
curl -sI 'https://bioinformaticsdotca.github.io/MLE_EdVan-2604/' 2>&1 | head -3

# Check a specific page (no docs/ prefix — it's stripped)
# Edmonton info is now in logistics.html - no separate page
# curl -sI 'https://bioinformaticsdotca.github.io/MLE_EdVan-2604/logistics.html' 2>&1 | head -3

# Check a downloadable file
curl -sI 'https://bioinformaticsdotca.github.io/MLE_EdVan-2604/content-files/file.pptx' 2>&1 | head -3
```

If you get `HTTP/2 404`, the file either wasn't pushed to GitHub or GitHub Pages hasn't rebuilt yet. Wait 30–60 seconds and try again, or trigger a manual rebuild (see Step 4).

---

## Adding Downloadable Files

If you need to add downloadable files (PDFs, PPTXs, etc.) that visitors can download **without a GitHub account**:

### Option A: Add to `docs/content-files/` (Recommended)

> **Why `docs/content-files/` and not the root `content-files/`?**
> GitHub Pages serves `docs/` as the website root. Files in `docs/content-files/` are accessible at `/content-files/` on the live site. A root-level `content-files/` (untracked, for local use only) also exists — this is separate from `docs/content-files/`. Always put downloadable files in `docs/content-files/` so they are tracked and pushed to GitHub.

1. Copy the file to `docs/content-files/` at the repo root
2. Commit and push the file:
   ```bash
   cp /path/to/file.pptx docs/content-files/
   git add docs/content-files/file.pptx docs/
   git commit -m 'add workshop location PPTX'
   git push origin main
   ```
3. Link to the file using a **relative path from the HTML output location**. Since HTML pages are in `docs/`, use the path relative to `docs/`:
   ```markdown
   [Download File](content-files/file.pptx)
   ```
   **Do NOT use `../content-files/`** — this resolves to the repo root, not the `docs/` directory, and will result in a 404.

4. Verify the URL. A file at `docs/content-files/file.pptx` will be accessible at:
   ```
   https://bioinformaticsdotca.github.io/MLE_EdVan-2604/content-files/file.pptx
   ```
   (the `docs/` prefix is removed by GitHub Pages)

5. Trigger a GitHub Pages rebuild if the new subdirectory isn't being served:
   ```bash
   gh api -X POST repos/bioinformaticsdotca/MLE_EdVan-2604/pages/builds
   ```

### Option B: Use GitHub Releases

For large files (e.g., >50 MB), consider using GitHub Releases instead:
1. Go to the GitHub repository → Releases → Draft a new release
2. Upload the file as a release asset
3. Link to the release URL: `https://github.com/bioinformaticsdotca/MLE_EdVan-2604/releases/tag/v1.0`

### Option C: Use Raw GitHub URLs

Files tracked in git can be accessed via raw GitHub:
```
https://raw.githubusercontent.com/bioinformaticsdotca/MLE_EdVan-2604/main/docs/content-files/file.pptx
```
This requires no special setup, but may not be as user-friendly for non-technical visitors.

---

## Adding New Pages

To add a new page to the website:

1. Create a new `.Rmd` file with a numbered prefix to control its position in the sidebar:
   - Use `003-` to `009-` for introductory/cross-cutting pages (003 is already used by Logistics)
   - Use `010-` to `090-` for module pages (010–080 are used by the 8 modules)
   - Use `100-` and above for appendix/reference pages

2. Add content to the `.Rmd` file. The basic structure is:
   ```markdown
   # Page Title

   ## Section 1

   Content here...

   ## Section 2

   More content...
   ```

3. For embedded videos, use iframe placeholders:
   ```html
   <iframe width='640' height='360' src='YOUTUBE EMBED LINK' title='YouTube video player' frameborder='0' allow='accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share' referrerpolicy='strict-origin-when-cross-origin' allowfullscreen></iframe>
   ```

4. For download links, see the section above on Adding Downloadable Files.

5. Rebuild and push the site:
   ```bash
   Rscript -e "bookdown::render_book('index.Rmd', 'bookdown::gitbook')"
   git add <new-file>.Rmd docs/
   git commit -m 'add new page'
   git push origin main
   ```

---

## Common Pitfalls

### ❌ Editing `docs/` Directly

Never edit HTML files in `docs/` directly. Your changes will be overwritten the next time the site is rebuilt with bookdown.

### ❌ Wrong Relative Paths for Downloads

When adding download links in `.Rmd` files:
- ✅ **Correct:** `content-files/file.pptx` (relative to `docs/` where the HTML is generated)
- ❌ **Wrong:** `../content-files/file.pptx` (resolves to the repo root, not `docs/`)

### ❌ Forgetting to Push `docs/`

Changes to `.Rmd` files won't appear on the website unless you also push the rebuilt `docs/` folder. Always run `git add docs/` along with your source file changes.

### ❌ Pushing Large Binary Files to Git

If a file is very large (>100 MB), consider using GitHub Releases instead of committing to the repository. Git has size limits and large files can slow down clones.

### ❌ Breaking the Schedule

The schedule is loaded dynamically from `schedule.csv` via R code in `index.Rmd`. The CSV must have these columns:
- `day_label` — e.g., `April 22`
- `date` — e.g., `2026-04-22`
- `activity` — e.g., `Module 1 (Instructor)`
- `type` — either `activity` or `break`
- `time_1` — PT time in `HH:MM` format
- `timezone_1_label` — label for PT (e.g., `Time (PT)`)
- `time_2` — MT time in `HH:MM` format
- `timezone_2_label` — label for MT (e.g., `Time (MT)`)

---

## Repository Structure

```
MLE_EdVan-2604/
├── index.Rmd              # Front page source
├── _bookdown.yml          # Build config (output_dir: docs)
├── _output.yml            # Theme, sidebar, and navigation config
├── style.css              # Custom CSS styling
├── schedule.csv           # Workshop schedule data
├── 001-faculty.Rmd        # Faculty & TA bios
├── 002-computing.Rmd      # Computing setup page
├── 003-logistics.Rmd      # Logistics parent page
# (004-edmonton.Rmd removed - consolidated into 003-logistics.Rmd)
├── 005-vancouver.Rmd      # Vancouver venue info
├── 010-module-1.Rmd       # Module 1 content
├── 020-module-2.Rmd       # Module 2 content
├── ...
├── 080-module-8.Rmd       # Module 8 content
├── content-files/         # (Not tracked by git — for local use)
└── docs/                  # GitHub Pages output directory
    ├── index.html         # Served as the site homepage
    ├── module-1.html      # Served as /module-1.html
    ├── content-files/     # Downloadable files (accessible at /content-files/)
    ├── img/               # Images
    ├── libs/              # JavaScript/CSS libraries
    ├── style.css          # Site stylesheet
    ├── search_index.json  # Search index
    └── .nojekyll          # Prevents Jekyll processing
```

---

## Quick Reference

**Rebuild the site:**
```bash
Rscript -e "bookdown::render_book('index.Rmd', 'bookdown::gitbook')"
```

**Commit and push all changes:**
```bash
git add <source-files> docs/
git commit -m 'your message'
git push origin main
```

**Trigger manual Pages rebuild:**
```bash
gh api -X POST repos/bioinformaticsdotca/MLE_EdVan-2604/pages/builds
```

**Check Pages status:**
```bash
gh api repos/bioinformaticsdotca/MLE_EdVan-2604/pages
```

**Check if a file is accessible:**
```bash
curl -sI 'https://bioinformaticsdotca.github.io/MLE_EdVan-2604/path/to/file' 2>&1 | head -3
```

---

## Website URL Reference

| Resource | URL |
|---|---|
| Website homepage | `https://bioinformaticsdotca.github.io/MLE_EdVan-2604/` |
| Module 1 page | `https://bioinformaticsdotca.github.io/MLE_EdVan-2604/module-1.html` |
| Logistics page | `https://bioinformaticsdotca.github.io/MLE_EdVan-2604/logistics.html` |
| Logistics page (Edmonton section) | `https://bioinformaticsdotca.github.io/MLE_EdVan-2604/logistics.html` |
| Downloadable file | `https://bioinformaticsdotca.github.io/MLE_EdVan-2604/content-files/filename.ext` |

**Note:** The `docs/` prefix is stripped from URLs because GitHub Pages serves the `docs/` directory as the site root.