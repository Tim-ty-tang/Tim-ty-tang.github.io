# Tim-ty-tang.github.io

A personal website built with [Hugo](https://gohugo.io/) featuring a blog, resume, and photo gallery. Deployed to [ttyt.me](https://ttyt.me) via GitHub Pages.

## Tech Stack

| Category | Technology |
|----------|------------|
| **Framework** | Hugo 0.144.1 (Go-based static site generator) |
| **Theme** | typo-tim (custom fork of Typo theme) |
| **Styling** | Pure CSS3 with CSS Custom Properties |
| **Fonts** | Literata (serif), Monaspace (monospace) - self-hosted |
| **Icons** | Font Awesome 6.5.1 (CDN), inline SVGs |
| **Gallery** | gallerydeluxe (vanilla JS) |
| **CI/CD** | GitHub Actions |
| **Hosting** | GitHub Pages |

## Directory Structure

```
.
├── config/
│   └── _default/
│       └── hugo.toml          # Main Hugo configuration
├── content/
│   ├── about/index.md         # About page
│   ├── gallery/               # Photo gallery (images + index.md)
│   ├── posts/                 # Blog posts (22 markdown files)
│   └── resume/                # Resume pages (main, html, print)
├── data/
│   └── resume.toml            # Structured resume data
├── i18n/
│   └── en.yaml                # English translations
├── themes/
│   └── typo-tim/              # Custom theme
│       ├── assets/
│       │   ├── css/           # Stylesheets
│       │   └── js/            # JavaScript
│       ├── layouts/           # HTML templates
│       │   ├── _default/      # Base layouts
│       │   ├── partials/      # Reusable components
│       │   ├── shortcodes/    # Custom shortcodes
│       │   ├── resume/        # Resume templates
│       │   └── gallerydeluxe/ # Gallery templates
│       └── static/fonts/      # Self-hosted fonts
├── archetypes/
│   └── default.md             # Template for new posts
├── .github/workflows/
│   └── hugo.yaml              # CI/CD deployment workflow
├── docker-compose.yml         # Local dev environment
├── .pre-commit-config.yaml    # Image optimization hooks
└── CNAME                      # Custom domain (ttyt.me)
```

## Quick Start

### Prerequisites

- [Hugo Extended](https://gohugo.io/installation/) v0.144.1+
- [Docker](https://docs.docker.com/get-docker/) (optional, for containerized dev)
- [Git](https://git-scm.com/)

### Local Development

**Option 1: Docker (Recommended)**
```bash
docker-compose up
# Site available at http://localhost:8081
```

**Option 2: Direct Hugo**
```bash
hugo server -D
# Site available at http://localhost:1313
```

### Production Build

```bash
hugo --gc --baseURL https://ttyt.me/
# Output in /public directory
```

## Modification Guide

### Adding a New Blog Post

1. Create a new markdown file:
   ```bash
   hugo new posts/my-new-post.md
   ```

2. Or manually create `/content/posts/my-new-post.md`:
   ```markdown
   ---
   title: "My New Post"
   date: 2025-01-15
   tags: ["topic1", "topic2"]
   summary: "Brief description for list pages"
   toc: true           # Show table of contents
   readTime: true      # Show reading time
   autonumber: false   # Auto-number headings
   math: false         # Enable KaTeX math rendering
   draft: false        # Set true to hide from production
   ---

   Your content here...
   ```

3. Preview locally, then push to deploy.

---

### Editing the Homepage

**Change intro text:**
Edit `config/_default/hugo.toml`:
```toml
[params]
homeIntroTitle = 'Your Name'
homeIntroContent = 'Your bio/intro text here'
```

**Modify layout:**
Edit `themes/typo-tim/layouts/_default/home.html`

---

### Modifying the Resume

Resume data is stored separately from layout for easy updates.

**Edit content:** `data/resume.toml`
```toml
[profile]
name = "Your Name"
tagline = "Your Title"
summary = "Your professional summary..."

[[experience]]
title = "Job Title"
company = "Company Name"
location = "City, Country"
dates = "2020 - Present"
responsibilities = [
    "Accomplishment 1",
    "Accomplishment 2",
]

[[skills]]
name = "Category"
list = ["Skill 1", "Skill 2", "Skill 3"]
```

**Edit section headers:** `i18n/en.yaml`

**Modify layout:**
- Main template: `themes/typo-tim/layouts/resume/resume.html`
- Partials: `themes/typo-tim/layouts/partials/resume/`
- Styles: `themes/typo-tim/assets/css/resume.css`

---

### Changing Navigation Menu

Edit `config/_default/hugo.toml`:
```toml
[[params.menu]]
name = "home"
url = "/"

[[params.menu]]
name = "posts"
url = "/posts"

[[params.menu]]
name = "new page"
url = "/new-page"
```

---

### Adding Social Links

Edit `config/_default/hugo.toml`:
```toml
[[params.social.list]]
name = "GitHub"
icon = "fab fa-github-square"  # Font Awesome class
url = "https://github.com/username"

[[params.social.list]]
name = "LinkedIn"
icon = "fab fa-linkedin"
url = "https://linkedin.com/in/username"
```

Available icons: [Font Awesome](https://fontawesome.com/icons)

---

### Adding Gallery Images

1. Add JPG/JPEG images to `/content/gallery/`
2. Images are automatically optimized by pre-commit hooks (resized to max 2048px, quality 85)
3. Gallery index: `/content/gallery/index.md`

**Configure gallery settings in `hugo.toml`:**
```toml
[params.gallerydeluxe]
shuffle = false
reverse = true
enableExif = true
dateFormat = "January 2006"
[params.gallerydeluxe.watermark]
image = "images/watermark.png"  # Optional watermark
```

---

### Changing Colors & Theme

**Switch color palette:**
Edit `hugo.toml`:
```toml
[params]
colorPalette = 'default'  # Options: default, catpuccin, gruvebox, eink
theme = 'auto'            # Options: light, dark, auto
```

**Create a custom palette:**
1. Copy `themes/typo-tim/assets/css/colors/default.css`
2. Save as `themes/typo-tim/assets/css/colors/mypalette.css`
3. Modify CSS variables:
   ```css
   :root[data-color-palette="mypalette"][data-theme="light"] {
       --content-primary: #333;
       --content-secondary: #666;
       --background: #fff;
       --code-background: #f5f5f5;
       --primary-color: #007acc;
   }
   ```
4. Set `colorPalette = 'mypalette'` in `hugo.toml`

---

### Modifying Styles

| File | Purpose |
|------|---------|
| `themes/typo-tim/assets/css/vars.css` | Global CSS variables (spacing, typography) |
| `themes/typo-tim/assets/css/main.css` | Core layout and components |
| `themes/typo-tim/assets/css/custom.css` | Site-specific overrides |
| `themes/typo-tim/assets/css/resume.css` | Resume-specific styles |
| `themes/typo-tim/assets/css/gallery.css` | Gallery styles |
| `themes/typo-tim/assets/css/colors/*.css` | Color palettes |

**Key CSS Variables** (`vars.css`):
```css
:root {
    --font-primary: 'Literata', serif;
    --font-code: 'Monaspace Neon', monospace;
    --size-step-0: 1em;      /* Base font size */
    --size-step-1: 1.25em;   /* h3 */
    --size-step-2: 1.563em;  /* h2 */
    --size-step-3: 1.953em;  /* h1 */
    --max-width: 750px;      /* Content max width */
}
```

---

### Enabling Comments (Giscus)

1. Set up [Giscus](https://giscus.app/) for your repository
2. Edit `hugo.toml`:
   ```toml
   [params.giscus]
   enable = true
   repo = "username/repo"
   repoID = "R_..."
   category = "Announcements"
   categoryID = "DIC_..."
   mapping = "pathname"
   theme = "preferred_color_scheme"
   ```

---

### Enabling Math (KaTeX)

In post frontmatter:
```yaml
---
math: true
---
```

Then use LaTeX:
```markdown
Inline: $E = mc^2$

Block:
$$
\int_0^\infty e^{-x^2} dx = \frac{\sqrt{\pi}}{2}
$$
```

---

## Deployment

### Automatic (GitHub Actions)

Push to `main` branch triggers automatic deployment:
1. Hugo builds the site
2. Assets are fingerprinted and minified
3. Deployed to GitHub Pages

### Manual

```bash
hugo --gc --baseURL https://ttyt.me/
# Upload /public directory to your hosting
```

---

## File Reference

### Configuration Files

| File | Description |
|------|-------------|
| `config/_default/hugo.toml` | Main site configuration |
| `data/resume.toml` | Resume structured data |
| `i18n/en.yaml` | Translation strings |
| `.github/workflows/hugo.yaml` | CI/CD workflow |
| `.pre-commit-config.yaml` | Image optimization hooks |
| `docker-compose.yml` | Local dev container |

### Key Templates

| Template | Purpose |
|----------|---------|
| `layouts/_default/baseof.html` | Base HTML wrapper |
| `layouts/_default/home.html` | Homepage |
| `layouts/_default/single.html` | Single post/page |
| `layouts/_default/list.html` | Archive/list pages |
| `layouts/partials/head.html` | `<head>` section |
| `layouts/partials/header.html` | Site header/nav |
| `layouts/partials/footer.html` | Site footer |

### Post Frontmatter Options

| Option | Type | Description |
|--------|------|-------------|
| `title` | string | Post title |
| `date` | date | Publication date |
| `tags` | array | Post tags |
| `summary` | string | Short description |
| `description` | string | SEO description |
| `toc` | bool | Show table of contents |
| `readTime` | bool | Show estimated reading time |
| `autonumber` | bool | Auto-number headings |
| `math` | bool | Enable KaTeX |
| `draft` | bool | Hide from production |

---

## License

Theme: MIT License (see `themes/typo-tim/LICENSE`)
