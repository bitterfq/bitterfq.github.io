# How to Build and Deploy Your Blog

## Quick Start (Most Common)

### Writing a New Blog Post

1. **Create a new post:**
   ```bash
   hugo new content/posts/my-post-name.md
   ```

2. **Edit the post:**
   - Open `content/posts/my-post-name.md`
   - Update the frontmatter (title, date, tags)
   - Write your content in markdown
   - Set `draft: false` when ready to publish

3. **Add images (if needed):**
   - Place images in `static/images/`
   - Reference them in markdown: `![Alt text](/images/your-image.jpg)`

4. **Deploy:**
   ```bash
   git add .
   git commit -m "Add new blog post"
   git push
   ```

   **That's it!** GitHub Actions will automatically build and deploy your site in 1-2 minutes.

---

## Local Preview (Optional)

To preview your site locally before deploying:

```bash
# Preview including drafts
hugo server -D

# Preview production version
hugo server
```

Visit `http://localhost:1313` in your browser.

---

## Directory Structure

```
├── content/              # Your blog posts and pages (EDIT THIS)
│   └── posts/           # Blog posts go here
├── static/              # Static files (images, PDFs, etc.)
│   └── images/          # Blog images go here
├── themes/typo/         # Theme (Git submodule, don't edit)
├── config.toml          # Site configuration
└── .github/workflows/   # GitHub Actions (auto-deployment)
```

---

## Important Commands

```bash
# Create new post
hugo new content/posts/post-name.md

# Local preview with drafts
hugo server -D

# Build site (optional - GitHub Actions does this)
hugo

# Deploy (automatic via GitHub Actions)
git push
```

---

## Configuration

Edit `config.toml` to change:
- Site title and description
- Navigation menu
- Social media links
- Homepage introduction
- Color scheme

---

## Troubleshooting

**Images not showing?**
- Make sure images are in `static/images/`
- Use `/images/filename.jpg` in markdown (with leading slash)

**Post not appearing?**
- Check that `draft: false` in frontmatter
- Make sure date is not in the future

**Deployment failing?**
- Check GitHub Actions tab: `https://github.com/bitterfq/bitterfq.github.io/actions`
- Look for error messages in the workflow logs

---

## What Changed (From Old Workflow)

**OLD workflow:**
1. Run `hugo` locally
2. Manually sync `public/` to root with rsync
3. Commit everything including built HTML
4. Push to GitHub

**NEW workflow:**
1. Edit content
2. Commit and push source files only
3. GitHub Actions automatically builds and deploys

**Benefits:**
- No more manual rsync commands
- Cleaner git history (no built files)
- Automatic deployment on every push
- Smaller repository size
