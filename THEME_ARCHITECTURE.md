# Theme Architecture Documentation

## Overview

This site uses **typo-tim**, a customized fork of the [Typo Hugo theme](https://github.com/francescotomaselli/typo) with additional features for resume, gallery, and blog functionality.

**Base Theme:** Typo by Francesco Tomaselli
**Customizations:** Resume sections, Gallery Deluxe integration, custom styling
**Hugo Version:** 0.144.1+ recommended

---

## Directory Structure

```
themes/typo-tim/
├── archetypes/          # Content templates for 'hugo new'
│   └── default.md
├── assets/              # Source files (processed by Hugo Pipes)
│   ├── css/
│   │   ├── reset.css       # Browser resets
│   │   ├── vars.css        # CSS custom properties
│   │   ├── utils.css       # Utility classes
│   │   ├── fonts.css       # Font imports
│   │   ├── main.css        # Core theme styles
│   │   ├── custom.css      # Site-specific customizations
│   │   └── colors/
│   │       └── default.css # Color palette (light/dark)
│   └── js/
│       └── gallerydeluxe/  # Gallery JavaScript
├── layouts/             # HTML templates
│   ├── _default/
│   │   ├── baseof.html    # Base template (all pages inherit)
│   │   ├── list.html      # List pages (blog index)
│   │   ├── single.html    # Single pages (blog posts)
│   │   └── terms.html     # Taxonomy pages (tags)
│   ├── partials/
│   │   ├── head/          # <head> components
│   │   ├── footer/        # Footer components
│   │   ├── resume/        # Resume section templates
│   │   └── gallerydeluxe/ # Gallery components
│   ├── shortcodes/
│   │   └── resume.html    # Resume shortcode
│   ├── gallerydeluxe/
│   │   └── gallery.html   # Gallery page template
│   └── index.html         # Homepage template
├── static/              # Static files (copied as-is to public/)
│   ├── fonts/
│   ├── apple-touch-icon.png
│   └── favicon.ico
├── hugo.toml            # Theme configuration
└── theme.toml           # Theme metadata

Root site structure:
├── config/              # Site configuration
│   └── _default/
│       └── hugo.toml    # Main config (merged with theme config)
├── content/             # Content files
│   ├── about/
│   ├── posts/           # Blog posts
│   ├── resume/          # Resume pages
│   └── gallery/         # Gallery images and metadata
├── data/
│   └── resume.toml      # Resume data (structured)
└── public/              # Generated site (DO NOT EDIT)
```

---

## Theme Components

### 1. CSS Architecture

#### Load Order
CSS files are concatenated in this order (see `layouts/partials/head/css.html`):

1. **reset.css** - Normalize browser defaults
2. **vars.css** - CSS custom properties (variables)
3. **utils.css** - Utility classes
4. **fonts.css** - Font imports (@font-face)
5. **main.css** - Core theme styles
6. **colors/default.css** - Color palette (can swap for other themes)
7. **custom.css** - Site-specific overrides

#### CSS Variables (Design Tokens)

**Location:** `themes/typo-tim/assets/css/vars.css`

**Available variables:**
```css
/* Layout */
--main-width: 65ch;
--main-padding: 0 1rem;
--content-height: calc(100vh - 60px);

/* Typography */
--h1-font-size: 2rem;
--h2-font-size: 1.5rem;
--h3-font-size: 1.25rem;
--small-font-size: 0.875rem;
--font-family: "Recursive", system-ui, sans-serif;

/* Spacing */
--separator-space: 3rem;
```

**Color variables** (in `colors/default.css`):
```css
/* Light theme */
--content-primary: hsl(0 0% 0%);
--content-secondary: hsl(0 0% 45%);
--background: hsl(0 0% 100%);

/* Dark theme (prefers-color-scheme: dark) */
--content-primary: hsl(0 0% 90%);
--content-secondary: hsl(0 0% 70%);
--background: hsl(0 0% 10%);
```

**⚠️ Known Issues:**
- `--primary` and `--theme` are referenced but not defined (see CLEANUP_PLAN.md)
- No variables for resume-specific styling

---

### 2. Layout System

#### Base Template: `layouts/_default/baseof.html`

All pages inherit from this template. Structure:

```html
<!DOCTYPE html>
<html>
  <head>
    {{- partial "head/meta.html" . -}}
    {{- partial "head/css.html" . -}}
    {{- partial "head/js.html" . -}}
  </head>
  <body>
    <main>
      {{- block "main" . }}{{- end }}
    </main>
    {{- partial "footer/index.html" . -}}
  </body>
</html>
```

#### Page Templates

**Single Post:** `layouts/_default/single.html`
- Used for: Blog posts
- Layout: Header, content, footer

**List Page:** `layouts/_default/list.html`
- Used for: Blog index, taxonomy pages
- Layout: Title, post list with dates

**Homepage:** `layouts/index.html`
- Custom layout for homepage
- Different from other pages

---

### 3. Resume System

#### Data Source
**Location:** `data/resume.toml`

**Structure:**
```toml
[header]
name = "Timothy Tang"
headline = "Data Science and Machine Learning"
avatar = "/path/to/image.jpg"

[[header.contact]]
icon = "fas fa-phone-square"
text = "XXX-XXX-XXXX"
link = "tel:XXX-XXX-XXXX"

[[summary]]
text = "Summary paragraph..."

[[experience]]
role = "Job Title"
company = "Company Name"
start = "2020"
end = "2023"
location = "City, State"
description = "Job description..."
```

#### Resume Templates

**Main Template:** `layouts/shortcodes/resume.html`
- Entry point, wraps everything in container
- **⚠️ Uses Bootstrap classes but Bootstrap CSS not included**

**Partials:**
- `resume_header.html` - Name, contact, avatar
- `resume_sidebar.html` - Skills, education (sidebar)
- `resume_main.html` - Experience, projects (main content)
- `resume_contact.html` - **Empty file, not implemented**

#### Usage in Content
```markdown
<!-- content/resume/_index.md -->
---
title: "Resume"
layout: resume
---

{{< resume >}}
```

---

### 4. Gallery System

**Plugin:** [Gallery Deluxe](https://github.com/linode/gallerydeluxe)

**Components:**
- `layouts/gallerydeluxe/gallery.html` - Gallery page template
- `layouts/partials/gallerydeluxe/*.html` - Gallery partials
- `assets/js/gallerydeluxe/*.js` - Gallery JavaScript (PhotoSwipe wrapper)

**Data Source:** `content/gallery/index.md`

**Structure:**
```
content/gallery/
├── index.md                # Gallery metadata
├── image1.jpg              # Images
├── image2.jpg
└── _DSF1234.JPG
```

**Features:**
- Lightbox/modal view
- Touch/swipe navigation
- Lazy loading
- Responsive images (Hugo processes images)

**⚠️ Known Issues:**
- Inline styles with excessive `!important` declarations (see CLEANUP_PLAN.md)
- Large source images (2-17MB each) in git

---

### 5. Blog System

**Content Location:** `content/posts/*.md`

**Front Matter:**
```yaml
---
title: "Post Title"
date: 2024-01-15
tags: ["data-science", "python"]
draft: false
math: true  # Enable KaTeX for mathematical notation
---
```

**Features:**
- Tags/taxonomy support
- Date-based sorting
- Draft posts (not published)
- Mathematical notation (KaTeX, if `math: true`)

---

## Customization Guide

### Changing Colors

**Edit:** `themes/typo-tim/assets/css/colors/default.css`

```css
/* Light theme */
:root {
  --content-primary: hsl(YOUR_HUE YOUR_SAT% YOUR_LIGHT%);
  --content-secondary: hsl(YOUR_HUE YOUR_SAT% YOUR_LIGHT%);
  --background: hsl(YOUR_HUE YOUR_SAT% YOUR_LIGHT%);
}

/* Dark theme */
@media (prefers-color-scheme: dark) {
  :root {
    --content-primary: hsl(YOUR_HUE YOUR_SAT% YOUR_LIGHT%);
    --content-secondary: hsl(YOUR_HUE YOUR_SAT% YOUR_LIGHT%);
    --background: hsl(YOUR_HUE YOUR_SAT% YOUR_LIGHT%);
  }
}
```

**To add more colors:**
1. Define in `vars.css`: `--primary: #54B689;`
2. Use in CSS: `color: var(--primary);`
3. Use in templates: (not directly, use CSS classes)

---

### Changing Fonts

**Edit:** `themes/typo-tim/assets/css/fonts.css`

**Current font:** Recursive (variable font from Google Fonts)

**To change:**
1. Add new `@font-face` or Google Fonts import
2. Update `--font-family` in `vars.css`
3. Clear browser cache and rebuild

**Example:**
```css
/* fonts.css */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap');

/* vars.css */
:root {
  --font-family: "Inter", system-ui, sans-serif;
}
```

---

### Changing Layout Width

**Edit:** `themes/typo-tim/assets/css/vars.css`

```css
:root {
  --main-width: 65ch;  /* Change to 80ch for wider, 50ch for narrower */
}
```

**Common values:**
- `60ch` - Narrow, easier reading
- `65ch` - Current (good for text)
- `80ch` - Wider (good for code)
- `1200px` - Fixed pixel width
- `90vw` - Responsive to viewport

---

### Adding New Pages

**Create content:**
```bash
hugo new about/index.md
hugo new posts/my-new-post.md
```

**Front matter:**
```yaml
---
title: "Page Title"
date: 2024-01-15
layout: single  # or custom layout name
---

Content here...
```

**Custom layout (optional):**
1. Create `themes/typo-tim/layouts/custom/single.html`
2. Set `layout: custom` in front matter

---

### Adding New CSS

**Best practices:**
1. Define variables in `vars.css` first
2. Add styles to `custom.css` (for site-specific) or `main.css` (for theme-wide)
3. Use CSS classes, avoid inline styles
4. Follow existing naming conventions

**Example:**
```css
/* custom.css */
.my-custom-section {
  max-width: var(--main-width);
  margin: var(--separator-space) auto;
  color: var(--content-primary);
}
```

---

## Configuration

### Site Config: `config/_default/hugo.toml`

**Key settings:**
```toml
baseURL = "https://ttyt.me/"
languageCode = "en-us"
title = "Tim's Website"
theme = "typo-tim"

[params]
  description = "Personal website and blog"
  author = "Timothy Tang"

[outputs]
  home = ["HTML", "RSS"]

[taxonomies]
  tag = "tags"
```

### Theme Config: `themes/typo-tim/hugo.toml`

**Module settings:**
```toml
[module]
  [module.hugoVersion]
    extended = false
    min = "0.116.0"
```

---

## Build Process

### Development
```bash
# Start dev server (auto-reload)
hugo server -D -p 8081

# Or with Docker
docker-compose up
```

**Access at:** http://localhost:8081

### Production
```bash
# Build site
hugo

# Output in public/ directory
# Deploy public/ to web server
```

### What Hugo Does
1. Reads `config/` and merges with theme config
2. Processes `content/` Markdown files
3. Applies layouts from `themes/typo-tim/layouts/`
4. Compiles CSS/JS from `assets/` (Hugo Pipes)
5. Copies `static/` files as-is
6. Generates HTML in `public/`

---

## Troubleshooting

### CSS not updating
```bash
# Clear Hugo cache
rm -rf resources/_gen/
hugo server
```

### Resume layout broken
**Cause:** Bootstrap classes used but Bootstrap CSS not included
**Fix:** See CLEANUP_PLAN.md Phase 1.1

### Icons not showing
**Cause:** Font Awesome CSS not included
**Fix:** See CLEANUP_PLAN.md Phase 1.4

### Dark theme not working
**Check:** Browser theme preference (not a site toggle currently)
```css
/* Force dark mode for testing */
:root {
  color-scheme: dark;
}
```

### Images not loading
**Check:**
1. Image path in Markdown (relative to content file)
2. Image exists in `content/` or `static/`
3. No broken links in HTML source

---

## Dependencies

### Required
- Hugo 0.116.0+ (extended version not required)
- Git (for version control)

### Optional
- Docker + Docker Compose (for containerized dev)
- Node.js (if adding npm scripts for CSS/JS processing)

### External Assets (CDN)
**Currently none, but recommended to add:**
- Bootstrap CSS (for resume layout) - or remove Bootstrap classes
- Font Awesome (for icons in resume)

---

## Package Integration Summary

This theme integrates three main systems:

1. **Typo Theme (Base)**
   - Minimal, typography-focused Hugo theme
   - Provides: base layouts, CSS reset, font loading, dark mode
   - Files: Most of `layouts/_default/`, `assets/css/main.css`

2. **Bootstrap Utilities (Partial)**
   - **⚠️ INCOMPLETE INTEGRATION**
   - Used in: Resume templates (grid, spacing, utilities)
   - Problem: HTML uses classes but CSS never included
   - Fix needed: Add Bootstrap CSS or rewrite templates

3. **Gallery Deluxe (Plugin)**
   - Hugo gallery plugin with lightbox
   - Provides: Gallery layouts, PhotoSwipe JavaScript wrapper
   - Files: `layouts/gallerydeluxe/`, `assets/js/gallerydeluxe/`

**The "smashing together" problem:** These three systems weren't fully integrated - Bootstrap classes added without CSS, custom styles without planning, resulting in fragile codebase.

---

## Code Style Guide

### CSS
- Use CSS custom properties for all repeated values
- Use `rem` for spacing (not `px`)
- Use HSL for colors (easier to adjust lightness for dark mode)
- Avoid `!important` (indicates specificity issue)
- Prefix custom classes to avoid conflicts (`.site-*`, `.resume-*`)

### HTML Templates
- Keep templates small (use partials)
- Use Hugo's template functions (`.Title`, `.Date`, `.Content`)
- Use semantic HTML5 elements (`<article>`, `<nav>`, `<section>`)
- Prefer CSS classes over inline styles
- Use descriptive class names (`resume-header`, not `rh`)

### Markdown Content
- Use YAML front matter (easier to read than TOML in content files)
- Keep front matter minimal (inherit defaults from config)
- Use relative image paths
- Use Hugo shortcodes for complex elements

---

## Future Improvements

### Recommended Enhancements
1. **Add theme switcher button** (currently relies on OS preference)
2. **Implement search** (using Fuse.js or similar)
3. **Add RSS feed styling**
4. **Create print-optimized CSS** for resume
5. **Add skip-to-content link** for accessibility
6. **Implement service worker** for offline support
7. **Add build-time image optimization** (automatic resize/compress)

### Modernization Ideas
1. Replace custom CSS with Tailwind CSS
2. Add TypeScript for JavaScript
3. Use Hugo modules for theme dependency management
4. Implement critical CSS inlining
5. Add automated accessibility testing

---

## Resources

### Hugo Documentation
- [Hugo Quick Start](https://gohugo.io/getting-started/quick-start/)
- [Hugo Templates](https://gohugo.io/templates/)
- [Hugo Pipes (Asset Processing)](https://gohugo.io/hugo-pipes/)

### Theme Resources
- [Original Typo Theme](https://github.com/francescotomaselli/typo)
- [Gallery Deluxe](https://github.com/linode/gallerydeluxe)

### CSS Resources
- [CSS Custom Properties (MDN)](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)
- [Modern CSS Reset](https://piccalil.li/blog/a-modern-css-reset/)

---

## Questions?

If something isn't clear:
1. Check `CLEANUP_PLAN.md` for known issues
2. Look at `CLAUDE.md` for project overview
3. Read Hugo documentation
4. Inspect the generated HTML in browser dev tools
5. Check Git history for context on changes

## Maintenance Log

**Last Updated:** 2024-11-24
**Theme Version:** typo-tim (fork of Typo, version unknown)
**Hugo Version:** 0.144.1
**Known Issues:** See CLEANUP_PLAN.md Phase 1
