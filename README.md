# Tim's Personal Website

Personal website and blog built with Hugo. Features blog posts on data science and machine learning, resume/CV sections, photo gallery, and custom theme.

**Live Site:** https://ttyt.me

---

## Quick Start

### Local Development

**With Docker (recommended):**
```bash
docker-compose up
```

**With Hugo directly:**
```bash
hugo server -D -p 8081
```

Visit: http://localhost:8081

### Build for Production
```bash
hugo
# Output in public/ directory
```

---

## Project Status

⚠️ **This site is functional but has technical debt.** See [CLEANUP_PLAN.md](CLEANUP_PLAN.md) for details.

**Known Issues:**
- Resume uses Bootstrap classes but Bootstrap CSS not included
- Some CSS variables undefined
- Syntax errors in custom.css
- Large image files in git (123MB)

**If you're new to this codebase:** Read [THEME_ARCHITECTURE.md](THEME_ARCHITECTURE.md) first to understand the structure.

---

## Site Features

- **Blog:** Technical posts on data science, ML, and software engineering
- **Resume/CV:** Interactive resume with multiple output formats
- **Photo Gallery:** Lightbox gallery with responsive images
- **About Page:** Personal introduction and professional interests
- **Dark/Light Mode:** Automatic theme based on OS preference
- **Responsive Design:** Mobile-friendly layouts

---

## Key Files & Directories

```
.
├── config/
│   └── _default/
│       └── hugo.toml          # Main site configuration
├── content/
│   ├── about/                 # About page
│   ├── posts/                 # Blog posts (Markdown)
│   ├── resume/                # Resume pages
│   └── gallery/               # Photo gallery with images
├── data/
│   └── resume.toml            # Resume data (structured)
├── themes/
│   └── typo-tim/              # Custom theme (fork of Typo)
├── docker-compose.yml         # Docker development setup
├── CLAUDE.md                  # Project instructions for AI assistants
├── CLEANUP_PLAN.md           # Technical debt cleanup roadmap
└── THEME_ARCHITECTURE.md     # Theme documentation
```

---

## Documentation

| Document | Purpose |
|----------|---------|
| [CLAUDE.md](CLAUDE.md) | Project overview and development guide |
| [CLEANUP_PLAN.md](CLEANUP_PLAN.md) | Known issues and cleanup roadmap |
| [THEME_ARCHITECTURE.md](THEME_ARCHITECTURE.md) | Theme structure and customization guide |

---

## Common Tasks

### Adding a Blog Post
```bash
hugo new posts/my-new-post.md
```

Edit `content/posts/my-new-post.md`:
```yaml
---
title: "My New Post"
date: 2024-01-15
tags: ["data-science", "python"]
draft: false
---

Your content here...
```

### Updating Resume
Edit `data/resume.toml` with your information, then rebuild.

### Adding Gallery Images
1. Add images to `content/gallery/`
2. Update `content/gallery/index.md` with image metadata
3. Rebuild site

### Changing Theme Colors
Edit `themes/typo-tim/assets/css/colors/default.css`

See [THEME_ARCHITECTURE.md](THEME_ARCHITECTURE.md) for detailed customization guide.

---

## Technology Stack

- **Hugo:** 0.144.1+ (static site generator)
- **Theme:** typo-tim (custom fork of [Typo](https://github.com/francescotomaselli/typo))
- **Gallery:** [Gallery Deluxe](https://github.com/linode/gallerydeluxe) plugin
- **Deployment:** GitHub Pages
- **Domain:** ttyt.me (custom domain via CNAME)

### External Dependencies (CDN)
**Currently missing but needed:**
- Bootstrap CSS (for resume layout) - See [CLEANUP_PLAN.md](CLEANUP_PLAN.md)
- Font Awesome (for resume icons) - See [CLEANUP_PLAN.md](CLEANUP_PLAN.md)

---

## Development Setup

### Requirements
- Hugo 0.116.0+ (extended version not required)
- Git
- Docker + Docker Compose (optional, for containerized development)

### Installation

**1. Clone repository:**
```bash
git clone https://github.com/Tim-ty-tang/Tim-ty-tang.github.io.git
cd Tim-ty-tang.github.io
```

**2. Start development server:**
```bash
# With Docker
docker-compose up

# OR with Hugo directly
hugo server -D -p 8081
```

**3. Make changes and test locally**

**4. Build for production:**
```bash
hugo
```

---

## Theme Architecture

This site uses **typo-tim**, a customized theme that integrates:

1. **Typo Theme** (base) - Minimal typography-focused Hugo theme
2. **Bootstrap Utilities** (partial) - Grid and spacing classes for resume
3. **Gallery Deluxe** (plugin) - Photo gallery with lightbox

**⚠️ Integration Issue:** Bootstrap classes are used in HTML but Bootstrap CSS was never included. This is one of the main issues to fix. See [CLEANUP_PLAN.md](CLEANUP_PLAN.md) for details.

### CSS Architecture
CSS is loaded in this order:
1. reset.css - Browser normalization
2. vars.css - CSS custom properties
3. utils.css - Utility classes
4. fonts.css - Font imports
5. main.css - Core theme styles
6. colors/default.css - Color palette
7. custom.css - Site-specific overrides

See [THEME_ARCHITECTURE.md](THEME_ARCHITECTURE.md) for complete documentation.

---

## Deployment

**Production site:** https://ttyt.me (GitHub Pages)

### GitHub Pages Setup
1. Build site: `hugo`
2. Push `public/` directory to GitHub Pages
3. CNAME file included for custom domain

### Custom Domain
- Domain: ttyt.me
- DNS configured to point to GitHub Pages
- CNAME file in repository root

---

## Known Issues & Roadmap

See [CLEANUP_PLAN.md](CLEANUP_PLAN.md) for detailed list and prioritized fixes.

**High Priority:**
- [ ] Fix CSS dependencies (Bootstrap, Font Awesome)
- [ ] Fix undefined CSS variables
- [ ] Fix custom.css syntax errors
- [ ] Consolidate inline styles

**Medium Priority:**
- [ ] Optimize images (123MB → <50MB)
- [ ] Remove generated files from git
- [ ] Complete resume contact section
- [ ] Create cohesive design system

**Low Priority:**
- [ ] Reorganize CSS files
- [ ] Add theme switcher button
- [ ] Improve documentation

---

## Troubleshooting

### CSS not updating
```bash
rm -rf resources/_gen/
hugo server
```

### Resume layout broken
The resume uses Bootstrap classes but Bootstrap CSS isn't included. Quick fix:
```html
<!-- Add to themes/typo-tim/layouts/partials/head/css.html -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
```

### Icons not showing
Font Awesome CSS not included. Quick fix:
```html
<!-- Add to themes/typo-tim/layouts/partials/head/css.html -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">
```

### Images not loading
Check image paths are relative to content file and images exist in `content/` or `static/`.

### Port already in use
```bash
# Change port in docker-compose.yml or use different port
hugo server -D -p 8082
```

---

## Contributing

This is a personal website, but if you notice issues:

1. Check [CLEANUP_PLAN.md](CLEANUP_PLAN.md) to see if it's already documented
2. Test changes locally before committing
3. Keep commits focused and descriptive
4. Update documentation if adding new features

---

## Project History

**Theme Origin:** Fork of [Typo](https://github.com/francescotomaselli/typo) Hugo theme
**Customizations:** Resume system, gallery integration, custom styling
**Integration Approach:** "Smashed a couple Hugo packages together" (user's description)
**Current State:** Functional but fragile - needs cleanup

---

## Resources

### Hugo Documentation
- [Hugo Quick Start](https://gohugo.io/getting-started/quick-start/)
- [Hugo Templates](https://gohugo.io/templates/)
- [Content Management](https://gohugo.io/content-management/)

### This Project
- [CLAUDE.md](CLAUDE.md) - Project overview for AI assistants
- [CLEANUP_PLAN.md](CLEANUP_PLAN.md) - Technical debt and fixes
- [THEME_ARCHITECTURE.md](THEME_ARCHITECTURE.md) - Theme structure guide

### Original Theme
- [Typo Theme](https://github.com/francescotomaselli/typo)

---

## License

This is a personal website. Theme (typo-tim) is based on [Typo](https://github.com/francescotomaselli/typo) by Francesco Tomaselli.

---

## Contact

**Author:** Timothy Tang
**Website:** https://ttyt.me
**GitHub:** [@Tim-ty-tang](https://github.com/Tim-ty-tang)

---

## Quick Reference

```bash
# Start dev server
hugo server -D -p 8081

# Build production site
hugo

# Create new post
hugo new posts/my-post.md

# Clear cache
rm -rf resources/_gen/

# Check Hugo version
hugo version

# Validate config
hugo config
```

---

**Last Updated:** 2024-11-24
**Hugo Version:** 0.144.1
**Status:** Functional, needs cleanup (see [CLEANUP_PLAN.md](CLEANUP_PLAN.md))
