# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Hugo-based static site blog** for a data engineer's personal website and blog. The site is deployed to **GitHub Pages** at `https://bitterfq.github.io/` and uses the [Typo theme](https://github.com/tomfran/typo) as a Git submodule.

**Hugo Version**: v0.111.3+extended

## Key Commands

### Building and Development
```bash
# Generate the static site (outputs to public/ directory)
hugo

# Run local development server with drafts enabled
hugo server -D

# Run local development server (production mode)
hugo server

# Clean build artifacts
rm -rf public/
```

### Content Management
```bash
# Create a new blog post
hugo new content/posts/my-new-post.md

# Create a new page
hugo new content/about.md
```

### GitHub Pages Deployment
The site has a **dual-directory structure** where both the root directory and `public/` contain built site files. This is due to GitHub Pages deployment requirements.

**After building the site:**
```bash
hugo                                    # Build site to public/
rsync -av --delete public/ ./ \
  --exclude public \
  --exclude themes \
  --exclude content \
  --exclude static \
  --exclude .git                        # Sync public/ to root
git add .
git commit -m "Rebuild site"
git push
```

### Theme Management
```bash
# Update the Typo theme submodule
git submodule update --remote themes/typo

# Initialize submodules (if cloning fresh)
git submodule init
git submodule update
```

## Architecture

### Directory Structure
- **`content/`** - All markdown content source files
  - `content/posts/` - Blog posts with frontmatter
  - `content/about.md` - About page
  - `content/resume.md` - Resume page
- **`themes/typo/`** - Git submodule containing the Typo theme
- **`static/`** - Static assets (currently contains `resume.pdf`)
- **`css/`** - Custom CSS overrides (`custom.css`)
- **`js/`** - Custom JavaScript files (`theme-switch.js`, `copy-code.js`, `mermaid.js`)
- **`public/`** - Hugo build output (Git-tracked for GitHub Pages)
- **Root directory** - Also contains built site files (synced from `public/`)

### Configuration
All site configuration lives in **`config.toml`**:
- Site metadata (title, baseURL, description)
- Theme configuration (color palette, appearance)
- Homepage intro text and collection settings
- Navigation menu structure
- Social media links
- Giscus comments integration
- Syntax highlighting settings

### Content Frontmatter Structure
Blog posts use YAML frontmatter:
```yaml
---
title: "Post Title"
date: 2025-02-01T14:00:00-08:00
draft: false
tags: ["tag1", "tag2"]  # Optional
---
```

### Deployment Architecture
This site uses an **unconventional deployment approach**:
1. Hugo builds to `public/` directory
2. Built files are synced from `public/` to root directory (excluding source files)
3. Both `public/` and root contain built site files
4. GitHub Pages serves from the root directory
5. All changes are committed to the `main` branch

**Important**: When making site changes, always:
1. Edit source files in `content/`, `config.toml`, or `themes/`
2. Run `hugo` to rebuild
3. Sync `public/` to root using rsync (see command above)
4. Commit both the source changes and built files

### Custom Features
- **Giscus comments**: Integrated via GitHub Discussions
- **Theme switching**: Custom JavaScript for light/dark mode
- **Code copy functionality**: Custom JS for copying code blocks
- **Mermaid diagrams**: Support for Mermaid diagram rendering
- **Custom CSS**: Additional styling in `css/custom.css`

## Important Notes

- The theme is a **Git submodule**, not a direct dependency
- **Never edit files in `themes/typo/` directly** - those changes will be lost
- Custom theme overrides should go in root-level `css/` or `js/` directories
- The site uses **taxonomy for tags** defined in `config.toml`
- Google Analytics is configured but placeholder ID needs updating if real tracking is needed
- Giscus comment system requires the GitHub repo to have Discussions enabled
