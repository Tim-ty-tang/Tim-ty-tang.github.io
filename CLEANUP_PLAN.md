# Hugo Site Cleanup Plan

## Overview

This site currently has technical debt from integrating multiple Hugo packages (Typo theme + Bootstrap resume templates + Gallery Deluxe) without proper CSS dependency management. This plan provides a phased approach to clean up and stabilize the codebase.

**Current State:** Functional but fragile - CSS broken, missing dependencies, maintenance difficult
**Goal:** Cohesive, maintainable Hugo site with clear architecture
**Estimated Effort:** 2-3 focused work sessions

---

## Phase 1: Critical Fixes (Required for proper functionality)

### 1.1 Fix CSS Dependencies

**Problem:** Resume uses Bootstrap classes but Bootstrap CSS is never included.

**Options:**
- **Option A (Quick):** Include Bootstrap CSS
  ```bash
  # Add to themes/typo-tim/layouts/partials/head/css.html
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  ```

- **Option B (Clean, Recommended):** Rewrite resume templates to use custom CSS
  - Create `assets/css/resume.css` with custom grid/utilities
  - Remove Bootstrap classes from templates
  - Align with existing theme variables

**Files to modify:**
- `/themes/typo-tim/layouts/shortcodes/resume.html`
- `/themes/typo-tim/layouts/partials/resume/*.html`
- `/themes/typo-tim/layouts/partials/head/css.html`

---

### 1.2 Fix CSS Variables

**Problem:** `reset.css` uses undefined `--primary` and `--theme` variables.

**Solution:**
```css
/* Add to themes/typo-tim/assets/css/vars.css */
:root {
  --primary: #54B689;      /* Your brand color */
  --theme: var(--background);  /* Or define separately */
}
```

**Files to modify:**
- `/themes/typo-tim/assets/css/vars.css` (add variables)
- OR `/themes/typo-tim/assets/css/reset.css` (replace with defined variables)

---

### 1.3 Fix custom.css Syntax Errors

**Problem:** Invalid CSS (quoted colors, SCSS syntax, incomplete values).

**Solution:**
```css
/* Line 4, 11, 25, 31: Remove quotes from colors */
color: #54B689;  /* NOT "54B689" */

/* Line 40: Add numeric value */
padding: 1rem;  /* NOT padding: rem; */

/* Lines 26-35: Convert SCSS nesting to valid CSS */
.resume-section-content .resume-timeline-item-header h3:before {
  content: "";
  position: absolute;
  /* ... rest of properties */
}
```

**Files to modify:**
- `/themes/typo-tim/assets/css/custom.css`

---

### 1.4 Add Font Awesome

**Problem:** Icons used in resume (`fas fa-phone-square`, etc.) but CSS never loaded.

**Solution:**
```html
<!-- Add to themes/typo-tim/layouts/partials/head/css.html -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">
```

**Files to modify:**
- `/themes/typo-tim/layouts/partials/head/css.html`

---

### 1.5 Handle Missing main.js

**Problem:** Template tries to load `js/main.js` but file doesn't exist.

**Options:**
- **Option A:** Create empty `assets/js/main.js` with a comment explaining future use
- **Option B:** Remove the reference from `themes/typo-tim/layouts/partials/head/js.html`

---

## Phase 2: Refactoring for Maintainability

### 2.1 Consolidate Inline Styles

**Problem:** Gallery template has inline styles with 8 `!important` declarations.

**Solution:**
```css
/* Add to assets/css/gallery.css or custom.css */
.gallery-back-nav {
  text-align: left;
  margin: 20px;
  position: relative;
  z-index: 1000;
}

.gallery-back-link {
  text-decoration: none;
  color: var(--content-primary);
  margin-left: 10px;
  margin-right: 50px;
  font-size: 48px;
  font-weight: bold;
}
```

Then update `/themes/typo-tim/layouts/gallerydeluxe/gallery.html` to use classes instead of inline styles.

**Files to modify:**
- `/themes/typo-tim/layouts/gallerydeluxe/gallery.html` (lines 32-33)
- `/themes/typo-tim/assets/css/custom.css` or create `gallery.css`

---

### 2.2 Complete Resume Contact Section

**Problem:** `resume_contact.html` is empty but referenced.

**Solution:**
- Either populate with contact info layout
- Or remove the include from `resume_header.html` line 28

**Files to modify:**
- `/themes/typo-tim/layouts/partials/resume/resume_contact.html`
- OR `/themes/typo-tim/layouts/partials/resume/resume_header.html`

---

### 2.3 Create Cohesive Design System

**Problem:** CSS variables are scattered and incomplete.

**Solution:** Document and standardize all CSS variables in `vars.css`:

```css
/* themes/typo-tim/assets/css/vars.css */

/* === COLORS === */
:root {
  /* Brand colors */
  --primary: #54B689;
  --secondary: #6C757D;

  /* Content colors (inherited from Typo theme) */
  --content-primary: /* existing */;
  --content-secondary: /* existing */;

  /* Background colors */
  --background: /* existing */;
  --theme: var(--background);

  /* Semantic colors */
  --link-color: var(--primary);
  --heading-color: var(--content-primary);
}

/* === SPACING === */
:root {
  --spacing-xs: 0.5rem;
  --spacing-sm: 1rem;
  --spacing-md: 1.5rem;
  --spacing-lg: 2rem;
  --spacing-xl: 3rem;
}

/* === TYPOGRAPHY === */
:root {
  /* existing h1-font-size, etc. */
  --font-weight-normal: 400;
  --font-weight-bold: 700;
}

/* === LAYOUT === */
:root {
  --main-width: /* existing */;
  --content-max-width: 800px;
  --resume-max-width: 1000px;
}
```

**Files to modify:**
- `/themes/typo-tim/assets/css/vars.css`

---

## Phase 3: Performance & Organization

### 3.1 Optimize Image Handling

**Problem:** 123MB of large images in git (bloats repository).

**Solutions:**
- **Option A:** Use Git LFS for large image files
  ```bash
  git lfs track "content/gallery/*.jpg"
  git lfs track "content/gallery/*.JPG"
  ```

- **Option B:** Pre-optimize images before committing
  ```bash
  # Install imagemagick
  mogrify -resize 2048x2048\> -quality 85 content/gallery/*.jpg
  ```

- **Option C:** Move images to external CDN (Cloudinary, Imgix, etc.)

**Files to modify:**
- `.gitattributes` (if using Git LFS)
- `.gitignore` (add optimized image outputs if creating build step)

---

### 3.2 Clean Up Generated Resources

**Problem:** `/resources/_gen/` contains Hugo-generated images (should not be in git).

**Solution:**
```bash
# Add to .gitignore
echo "resources/_gen/" >> .gitignore
git rm -r --cached resources/_gen/
git commit -m "Remove generated resources from git"
```

**Files to modify:**
- `.gitignore`

---

### 3.3 Reorganize CSS Files

**Current structure:**
- reset.css
- vars.css
- utils.css
- fonts.css
- main.css
- custom.css
- colors/*.css

**Proposed structure:**
```
assets/css/
├── 00-reset.css          # Browser resets
├── 01-variables.css      # All CSS custom properties
├── 02-fonts.css          # Font imports
├── 03-base.css           # Base element styles
├── 04-layout.css         # Grid, container, spacing utilities
├── 05-components.css     # Reusable components
├── 06-resume.css         # Resume-specific styles
├── 07-gallery.css        # Gallery-specific styles
└── themes/
    ├── light.css
    └── dark.css
```

Benefits:
- Clear load order (numbered prefixes)
- Separation of concerns
- Easy to find and modify styles

---

## Phase 4: Documentation & Testing

### 4.1 Update Theme Documentation

**Files to create/update:**
- `themes/typo-tim/README.md` - Theme architecture and customization guide
- `themes/typo-tim/theme.toml` - Update author info to reflect "typo-tim" fork
- `THEME_ARCHITECTURE.md` - Document CSS variables, layouts, partials structure

---

### 4.2 Create Testing Checklist

**Create:** `TESTING.md`

```markdown
# Testing Checklist

Before deploying, verify:

## Visual Testing
- [ ] Homepage loads correctly
- [ ] Blog posts render with proper formatting
- [ ] Resume displays correctly on desktop (1920px)
- [ ] Resume displays correctly on tablet (768px)
- [ ] Resume displays correctly on mobile (375px)
- [ ] Gallery loads and images display
- [ ] Gallery navigation works
- [ ] About page renders correctly

## Theme Switching
- [ ] Light theme works
- [ ] Dark theme works
- [ ] Theme toggle functions
- [ ] Colors are readable in both themes

## Icons & Assets
- [ ] Font Awesome icons display in resume
- [ ] All images load
- [ ] Fonts load correctly

## Build
- [ ] `hugo` builds without errors
- [ ] No CSS errors in browser console
- [ ] No JS errors in browser console
```

---

### 4.3 Add CSS Comments

**Add to key CSS files:**
```css
/* ============================================
   RESUME STYLES
   Styling for resume/CV pages
   Dependencies: vars.css, layout.css
   ============================================ */
```

---

## Implementation Priority

### Week 1: Critical Fixes (Phase 1)
**Impact:** High - Site will function correctly
**Effort:** 2-4 hours
**Focus:** Get CSS working, fix syntax errors

### Week 2: Refactoring (Phase 2)
**Impact:** Medium - Easier to maintain
**Effort:** 3-5 hours
**Focus:** Clean up technical debt, consolidate styles

### Week 3: Optimization (Phase 3)
**Impact:** Medium - Better performance and git hygiene
**Effort:** 2-3 hours
**Focus:** Image optimization, file organization

### Week 4: Documentation (Phase 4)
**Impact:** Low - Easier onboarding and future maintenance
**Effort:** 2-3 hours
**Focus:** Document decisions, create guides

---

## Quick Start: Minimum Viable Fixes

If you only have 1-2 hours, do these in order:

1. **Fix custom.css syntax** (15 min)
   - Remove quotes from colors
   - Fix padding value
   - Convert SCSS nesting to valid CSS

2. **Add Bootstrap CSS** (5 min)
   - Include CDN link in head/css.html

3. **Add Font Awesome** (5 min)
   - Include CDN link in head/css.html

4. **Define missing CSS variables** (10 min)
   - Add `--primary` and `--theme` to vars.css

5. **Test resume page** (15 min)
   - Verify layout works on mobile/desktop

This will get your site from "broken but works" to "actually works correctly."

---

## Success Criteria

You'll know the cleanup is complete when:

- ✅ Resume displays correctly on mobile and desktop
- ✅ No CSS errors in browser console
- ✅ No undefined CSS variables
- ✅ Icons display correctly
- ✅ Can modify styles without breaking other pages
- ✅ New developer can understand the theme structure
- ✅ Git repository is under 50MB
- ✅ Build completes without warnings

---

## Questions to Answer Before Starting

1. **Bootstrap decision:** Include Bootstrap or rewrite templates?
   - Including is faster (30 min)
   - Rewriting is cleaner long-term (3-4 hours)

2. **Image optimization:** Git LFS, pre-process, or CDN?
   - Git LFS: Best for large images, requires LFS setup
   - Pre-process: One-time shrink, simpler
   - CDN: Best performance, costs money

3. **CSS reorganization:** Keep current structure or reorganize?
   - Keep: Faster, less disruption
   - Reorganize: Better maintainability, worth it if doing full refactor

4. **Contact section:** Complete it or remove it?
   - Complete: Better resume
   - Remove: Faster, can add later

---

## Getting Help

If you get stuck on any phase:

1. Check the detailed file paths and line numbers in this doc
2. Test changes locally with `hugo server -D` before committing
3. Use browser dev tools to inspect CSS issues
4. Refer to `THEME_ARCHITECTURE.md` for theme structure
5. Check Hugo docs: https://gohugo.io/documentation/

---

## Rollback Plan

Before making changes:

```bash
# Create a backup branch
git checkout -b backup-before-cleanup
git push origin backup-before-cleanup

# Return to working branch
git checkout claude/review-hugo-cleanup-*
```

If something breaks:
```bash
git reset --hard backup-before-cleanup
```
