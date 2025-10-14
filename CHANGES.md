# Changes Made to Repository (October 13, 2025)

## Summary

Modernized the blog deployment workflow from manual builds to automated GitHub Actions deployment. Cleaned up repository bloat and simplified the workflow significantly.

---

## Major Changes

### 1. **Upgraded Hugo**
- **From:** v0.111.3 (March 2023)
- **To:** v0.139.3 (November 2024)
- **Why:** Old Hugo version was incompatible with latest theme updates
- **How:** Downloaded and installed Hugo binary to `/usr/local/bin/hugo`

### 2. **Implemented GitHub Actions CI/CD**
- **Added:** `.github/workflows/deploy.yml`
- **What it does:**
  - Automatically builds Hugo site on every push to main
  - Deploys to GitHub Pages without manual intervention
  - Uses Hugo 0.139.3 for consistent builds
- **Replaced:** Manual `hugo && rsync` workflow

### 3. **Repository Cleanup**
**Removed ~100 files including:**
- All built HTML/CSS/JS files from root directory
- Duplicate `images/` directory (15MB)
- `public/` directory contents (was 908KB)
- `posts/`, `tags/`, `about/`, `assets/`, `fonts/`, `js/` (all were build output)
- Root `404.html`, `index.html`, `index.xml`, `sitemap.xml`

**Why:** These files are now generated automatically by GitHub Actions. No need to commit build artifacts.

### 4. **Fixed Image Storage**
- **Moved images from:** Root `images/` directory
- **To:** `static/images/` directory
- **Why:** Hugo serves files from `static/` directory. Files in `static/` are copied to site root at build time.

### 5. **Added Proper .gitignore**
Now ignores:
- `public/` (Hugo build output)
- `resources/` (Hugo cache)
- `.hugo_build.lock`
- Editor and OS files

### 6. **Fixed Theme Submodule**
- Reinitialized `themes/typo` Git submodule
- Ensures theme is properly tracked and updated

---

## New Files Created

1. **`.github/workflows/deploy.yml`** - GitHub Actions workflow for automated deployment
2. **`.gitignore`** - Prevents build artifacts from being committed
3. **`.nojekyll`** - Tells GitHub Pages not to use Jekyll processing
4. **`HOW_TO_DEPLOY.md`** - Simple deployment instructions
5. **`CHANGES.md`** - This file, documenting all changes
6. **`CLAUDE.md`** - Technical documentation for Claude Code (updated)
7. **`content/posts/mt-shasta-ai-lessons.md`** - New blog post about hiking

---

## Configuration Changes

### GitHub Pages Settings (Manual)
- **Changed Source:** From "Deploy from branch" → "GitHub Actions"
- **Location:** Repository Settings → Pages → Build and deployment

### config.toml
- No changes to configuration
- All existing settings preserved

---

## Workflow Comparison

### Old Workflow
```bash
# Edit content
vim content/posts/my-post.md

# Build site
hugo

# Manually sync public/ to root
rsync -av --delete public/ ./ --exclude public --exclude themes --exclude content --exclude static --exclude .git

# Commit EVERYTHING (source + built files)
git add .
git commit -m "Update site"
git push
```

### New Workflow
```bash
# Edit content
vim content/posts/my-post.md

# Commit ONLY source files
git add content/posts/my-post.md
git commit -m "Add new post"
git push

# Done! GitHub Actions builds and deploys automatically
```

---

## Benefits

1. **Simpler:** No more manual build/sync steps
2. **Cleaner:** Git only tracks source files, not build output
3. **Smaller:** Repository size reduced by ~15MB
4. **Automated:** Push to main = automatic deployment
5. **Consistent:** Same Hugo version used for all builds
6. **Faster:** No need to commit/push large built files

---

## Breaking Changes

**None.** The site content and configuration remain unchanged. Only the deployment process changed.

---

## Testing

1. ✅ Created new blog post with images
2. ✅ Images display correctly on live site
3. ✅ GitHub Actions workflow runs successfully
4. ✅ Site deploys automatically within 2 minutes of push
5. ✅ All existing posts remain intact
6. ✅ Theme styling works correctly

---

## Files You Should Care About

**Edit these:**
- `content/posts/*.md` - Your blog posts
- `static/images/*` - Your images
- `config.toml` - Site configuration

**Don't edit these:**
- `themes/typo/*` - Theme files (Git submodule)
- `.github/workflows/deploy.yml` - Unless changing deployment
- `public/` - Auto-generated, not in Git

**Auto-generated (ignored by Git):**
- `public/` - Hugo build output
- `resources/` - Hugo cache
