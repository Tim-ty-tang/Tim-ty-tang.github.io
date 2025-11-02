# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal website built with Hugo static site generator. It serves as a portfolio and blog for Tim (Timothy Tang), featuring:
- Personal about page
- Blog posts on data science, machine learning, and technical topics
- Resume/CV sections
- Photo gallery
- Custom Hugo theme "typo-tim" (based on the Typo theme)

## Development Commands

### Local Development
```bash
# Start development server using Docker
docker-compose up

# Alternative: Start Hugo development server directly (requires Hugo installation)
hugo server -D -p 8081
```

The site runs on port 8081 by default.

### Build Commands
```bash
# Build the site for production
hugo

# Build with drafts included
hugo -D
```

### Theme Management
The site uses a custom theme located in `themes/typo-tim/`. This is a modified version of the Typo theme with customizations.

## Architecture and Structure

### Content Organization
- `content/about/` - About page with personal information
- `content/posts/` - Blog posts (Markdown files)
- `content/resume/` - Resume sections (main, print, html versions)
- `content/gallery/` - Photo gallery with images

### Configuration
- `config/_default/hugo.toml` - Main Hugo configuration
- `data/resume.toml` - Resume data in TOML format
- `docker-compose.yml` - Docker setup for development

### Theme Structure
- `themes/typo-tim/` - Custom theme based on Typo
- `themes/typo-tim/layouts/` - HTML templates
- `themes/typo-tim/assets/` - CSS, JS, and other assets
- `themes/typo-tim/static/` - Static files (fonts, favicons)

### Key Features
- Responsive design with dark/light theme support
- Gallery functionality with image thumbnails
- Resume generation in multiple formats
- Social media integration
- Blog post pagination
- Mathematical notation support (enabled in config)

### Content Types
- **Posts**: Technical articles on data science, ML, and engineering
- **About**: Personal introduction and professional interests
- **Resume**: Professional experience and skills
- **Gallery**: Photography and travel images

### Deployment
The site is configured for GitHub Pages deployment (ttyt.me domain) with CNAME file included.

## Development Notes

### Adding New Content
- Blog posts: Add Markdown files to `content/posts/`
- Gallery images: Add to `content/gallery/` and update index.md
- Resume updates: Modify `data/resume.toml`

### Theme Customization
The theme is customized in `themes/typo-tim/` and should be modified there rather than overriding in the main site structure.

### Container Development
The project includes Docker setup for consistent development environment across platforms.