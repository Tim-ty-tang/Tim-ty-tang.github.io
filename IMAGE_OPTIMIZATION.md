# Image Optimization Guide

## Current State

The gallery contains **24 high-resolution images** totaling approximately **123MB**. This creates issues:
- Large git repository size (slow clones)
- Longer Hugo build times
- Larger deployment size

## Recommended Solutions

### Option 1: Pre-Optimize Images Before Committing (Quick, Simple)

**Pros:** Simple, one-time operation, no external dependencies
**Cons:** Loses original quality, irreversible

**Steps:**
```bash
# Install ImageMagick (if not already installed)
# macOS: brew install imagemagick
# Ubuntu: sudo apt-get install imagemagick

# Resize images to max 2048px width and compress to 85% quality
cd content/gallery
mogrify -resize 2048x2048\> -quality 85 *.jpg *.JPG

# This will reduce file sizes significantly (typically 50-80% reduction)
```

**Result:** Images will be ~2-4MB instead of 8-17MB

---

### Option 2: Use Git LFS (Best for Large Files)

**Pros:** Keeps full quality, doesn't bloat git history
**Cons:** Requires Git LFS setup, some hosts have bandwidth limits

**Steps:**
```bash
# Install Git LFS
# macOS: brew install git-lfs
# Ubuntu: sudo apt-get install git-lfs

# Initialize Git LFS in repo
git lfs install

# Track large image files
git lfs track "content/gallery/*.jpg"
git lfs track "content/gallery/*.JPG"
git lfs track "content/gallery/*.png"

# This creates/updates .gitattributes
git add .gitattributes

# Migrate existing files to LFS
git lfs migrate import --include="content/gallery/*.jpg,content/gallery/*.JPG" --everything

# Push LFS files
git push origin main --all
```

**Result:** Git repository size dramatically reduced, images stored in LFS

**GitHub LFS Limits:**
- Free: 1GB storage, 1GB/month bandwidth
- Pro: 50GB storage, 50GB/month bandwidth
- Bandwidth usage: each page view = ~1-2MB (thumbnails)

---

### Option 3: Use External CDN (Best Performance)

**Pros:** Fastest loading, smallest repo, unlimited bandwidth
**Cons:** Costs money, requires migration, vendor lock-in

**Recommended Services:**
- **Cloudinary** - Free tier: 25GB storage, 25GB/month bandwidth
- **Imgix** - Starts at $10/month
- **Cloudflare Images** - $5/month for 100k images

**Steps:**
1. Upload images to CDN service
2. Update `content/gallery/index.md` with CDN URLs
3. Remove local image files from git
4. Add `.env` for CDN credentials

**Example with Cloudinary:**
```markdown
# content/gallery/index.md
---
title: "Gallery"
resources:
- src: https://res.cloudinary.com/yourname/image/upload/v1/gallery/image1.jpg
  title: "Image 1"
- src: https://res.cloudinary.com/yourname/image/upload/v1/gallery/image2.jpg
  title: "Image 2"
---
```

---

### Option 4: Hugo Image Processing (Current Approach, Good Enough)

**What's happening now:**
- Hugo automatically processes images on build
- Creates multiple sizes (thumbnails, medium, large)
- Stores processed versions in `resources/_gen/` (now gitignored)
- Serves optimized versions on site

**Pros:** Already working, no code changes needed
**Cons:** Large source files still in git, slow builds

**To improve:**
- Pre-optimize source images to ~4MB each (Option 1)
- This keeps Hugo processing but reduces git bloat

---

## Recommended Strategy

**For your use case (personal portfolio/blog):**

1. **Short-term (today):**
   - Pre-optimize existing images with ImageMagick (Option 1)
   - Reduces git size immediately without breaking anything
   ```bash
   cd content/gallery && mogrify -resize 2048x2048\> -quality 85 *.jpg *.JPG
   ```

2. **Medium-term (next month):**
   - Set up Git LFS for future large files (Option 2)
   - Migrate existing optimized images to LFS
   - Establish workflow: optimize → commit → push to LFS

3. **Long-term (if traffic grows):**
   - Consider Cloudinary free tier if serving thousands of visitors
   - Migrate when you approach bandwidth limits

---

## Current Gallery Images

These images are particularly large:

| File | Size | Recommended Action |
|------|------|-------------------|
| `_DSF1405.JPG` | 17MB | Resize to 2048px, compress to 85% → ~2MB |
| `_DSF1513.JPG` | 17MB | Resize to 2048px, compress to 85% → ~2MB |
| `_DSF1503.JPG` | 17MB | Resize to 2048px, compress to 85% → ~2MB |
| `IMG_1123.jpg` | 8.9MB | Resize to 2048px, compress to 85% → ~1.5MB |
| All others | 2-8MB | Same treatment |

**Total after optimization: ~30-40MB** (down from 123MB)

---

## Automated Workflow for Future Images

Create a script to optimize images before committing:

**`.githooks/pre-commit-image-check.sh`:**
```bash
#!/bin/bash
# Check for large images and optimize them automatically

for file in $(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(jpg|jpeg|png|JPG|JPEG|PNG)$'); do
    if [ -f "$file" ]; then
        size=$(wc -c < "$file")
        # If file is larger than 5MB, optimize it
        if [ $size -gt 5242880 ]; then
            echo "Optimizing large image: $file ($(numfmt --to=iec $size))"
            mogrify -resize 2048x2048\> -quality 85 "$file"
            git add "$file"
            echo "✓ Optimized: $file"
        fi
    fi
done
```

**Install the hook:**
```bash
chmod +x .githooks/pre-commit-image-check.sh
git config core.hooksPath .githooks
```

Now all images >5MB will be auto-optimized on commit!

---

## Hugo Configuration for Image Processing

Your current setup (in `config/_default/hugo.toml`):

```toml
[imaging]
  quality = 85
  resampleFilter = "Lanczos"
  anchor = "Smart"
```

This is already good! Hugo creates responsive image sizes automatically.

---

## Testing After Optimization

1. Run local build: `hugo server -D`
2. Check gallery page loads correctly
3. Verify images display at good quality
4. Confirm file sizes reduced
5. Test on mobile devices for performance

---

## Decision Matrix

| Criteria | Pre-Optimize | Git LFS | CDN |
|----------|--------------|---------|-----|
| **Setup time** | 5 minutes | 30 minutes | 2 hours |
| **Cost** | Free | Free* | $0-10/mo |
| **Repo size** | Medium | Small | Smallest |
| **Performance** | Good | Good | Best |
| **Complexity** | Simple | Medium | Complex |
| **Recommended for** | Quick fix | Long-term | High traffic |

*GitHub LFS: Free tier usually sufficient for personal sites

---

## Immediate Action Recommended

Run this now to reduce git size by 60-70%:

```bash
cd /home/user/Tim-ty-tang.github.io/content/gallery
# Backup originals first (optional)
mkdir -p ~/gallery-originals
cp *.{jpg,JPG} ~/gallery-originals/ 2>/dev/null || true

# Optimize all images
mogrify -resize 2048x2048\> -quality 85 *.jpg *.JPG

# Verify results
ls -lh

# Commit the optimized images
git add *.jpg *.JPG
git commit -m "Optimize gallery images for web (resize + compress)"
```

**Before:** 123MB total
**After:** ~30-40MB total
**Savings:** ~80MB (65% reduction)

---

## Questions?

- **Will quality be acceptable?** Yes, 2048px at 85% quality is excellent for web
- **What about retina displays?** 2048px is sufficient for 1024px displayed size at 2x retina
- **Can I keep originals?** Yes, backup to external drive or cloud storage
- **Will this break the gallery?** No, Hugo will regenerate thumbnails automatically

---

**Next Steps:** See CLEANUP_PLAN.md Phase 3.1 for integration with cleanup workflow.
