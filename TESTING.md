# Testing Checklist

Use this checklist to verify the site works correctly after making changes.

## Pre-Deployment Testing

### Build & Serve
- [ ] Hugo builds without errors: `hugo`
- [ ] Dev server starts: `hugo server -D -p 8081`
- [ ] No console errors in terminal
- [ ] No broken resource references

---

## Visual Testing

### Homepage
- [ ] Homepage loads correctly at http://localhost:8081
- [ ] Site title displays correctly
- [ ] Navigation links work
- [ ] Footer displays correctly

### Blog Posts
- [ ] Blog index page loads (`/posts/`)
- [ ] Individual blog posts load correctly
- [ ] Blog post formatting looks correct (headings, paragraphs, lists)
- [ ] Code blocks render with syntax highlighting
- [ ] Mathematical notation renders (if `math: true` in post)
- [ ] Images in posts load correctly
- [ ] Tags display correctly

### About Page
- [ ] About page loads (`/about/`)
- [ ] Content displays correctly
- [ ] Images load (if any)

### Resume/CV
**Desktop (1920px):**
- [ ] Resume page loads (`/resume/`)
- [ ] Header displays with name and tagline
- [ ] Contact information displays with icons
- [ ] Two-column layout works (main content left, sidebar right)
- [ ] Experience section displays correctly
- [ ] Projects section displays correctly
- [ ] Education section displays in sidebar
- [ ] Skills section displays in sidebar
- [ ] Font Awesome icons display correctly
- [ ] Section headings have colored bar accent
- [ ] Spacing and typography look professional

**Tablet (768px):**
- [ ] Resume layout adapts responsively
- [ ] Header stacks properly
- [ ] Contact info moves below name
- [ ] Main and sidebar stack vertically
- [ ] Text remains readable

**Mobile (375px):**
- [ ] Resume is readable on small screen
- [ ] No horizontal scrolling
- [ ] Touch targets are adequate size
- [ ] Font sizes are appropriate
- [ ] Spacing is comfortable

### Gallery
- [ ] Gallery page loads (`/gallery/`)
- [ ] All thumbnail images load
- [ ] Back arrow (←) displays and works
- [ ] Back arrow returns to homepage
- [ ] Clicking thumbnail opens lightbox
- [ ] Lightbox navigation works (next/previous)
- [ ] Lightbox close button works
- [ ] Images display at good quality
- [ ] Page loads reasonably fast

---

## Theme & Colors

### Light Theme (default)
- [ ] Background is light colored
- [ ] Text is dark colored (high contrast)
- [ ] Links are visible and colored
- [ ] Headings use primary color (#54B689)
- [ ] Resume section headings have colored accent
- [ ] Code blocks have subtle background

### Dark Theme (OS preference)
**To test:** Set OS to dark mode, then reload site

- [ ] Background is dark colored
- [ ] Text is light colored (high contrast)
- [ ] Links are visible in dark mode
- [ ] Headings maintain good contrast
- [ ] Resume remains readable
- [ ] Gallery back arrow is visible (white)
- [ ] No elements become invisible

---

## Responsive Design

Test at these breakpoints:
- [ ] **Mobile:** 375px width
- [ ] **Tablet:** 768px width
- [ ] **Desktop:** 1024px width
- [ ] **Large Desktop:** 1920px width

For each breakpoint verify:
- [ ] Layout adapts appropriately
- [ ] Text remains readable
- [ ] No horizontal scroll
- [ ] Images scale correctly
- [ ] Navigation works
- [ ] Spacing is appropriate

---

## Browser Compatibility

Test in:
- [ ] Chrome/Chromium (latest)
- [ ] Firefox (latest)
- [ ] Safari (latest, if on macOS)
- [ ] Mobile Safari (iPhone)
- [ ] Chrome Mobile (Android)

For each browser check:
- [ ] Site loads correctly
- [ ] CSS renders properly
- [ ] JavaScript works (gallery)
- [ ] Fonts load correctly
- [ ] No console errors

---

## Performance

### Page Load Speed
- [ ] Homepage loads in < 2 seconds (local)
- [ ] Resume page loads in < 2 seconds (local)
- [ ] Gallery page loads in < 5 seconds (local)
- [ ] Images load progressively (don't block render)

### Build Performance
- [ ] `hugo` build completes in < 30 seconds
- [ ] No warnings about slow templates
- [ ] Image processing completes successfully
- [ ] Generated CSS is minified
- [ ] Generated JS is minified

### Browser Console
Open browser dev tools (F12) and check:
- [ ] No JavaScript errors
- [ ] No CSS errors
- [ ] No 404 errors for resources
- [ ] No mixed content warnings (HTTP/HTTPS)
- [ ] Font Awesome CSS loads correctly
- [ ] All CSS variables are defined

---

## Functionality Testing

### Links
- [ ] All internal links work
- [ ] All external links work
- [ ] Links open in appropriate target (_blank for external)
- [ ] Social media links work (if any)

### Navigation
- [ ] Home link works
- [ ] About link works
- [ ] Posts/Blog link works
- [ ] Resume link works
- [ ] Gallery link works
- [ ] Back to top scrolls to top (if present)

### Resume Features
- [ ] Contact links are clickable
- [ ] Email links open mail client (`mailto:`)
- [ ] Phone links work on mobile (`tel:`)
- [ ] External links open in new tab
- [ ] Experience items are well-formatted
- [ ] Lists render correctly

### Gallery Features
- [ ] Thumbnails load as grid
- [ ] Clicking image opens full size
- [ ] Arrow keys navigate in lightbox (desktop)
- [ ] Swipe gestures work (mobile)
- [ ] ESC key closes lightbox
- [ ] Click outside image closes lightbox
- [ ] EXIF data displays (if enabled)

---

## Content Validation

### Resume Data
- [ ] Name displays correctly
- [ ] Job title/tagline is current
- [ ] Contact information is up to date
- [ ] Experience entries are accurate
- [ ] Education is correct
- [ ] Skills are current
- [ ] No typos in resume content

### Blog Posts
- [ ] Dates are correct
- [ ] Tags are appropriate
- [ ] No draft posts published (unless intended)
- [ ] Content is well-formatted
- [ ] No broken image links
- [ ] No placeholder text ("Lorem ipsum")

### Metadata
- [ ] Page titles are descriptive
- [ ] Meta descriptions exist
- [ ] Social media tags present (if using)
- [ ] Favicon displays correctly
- [ ] Site title in browser tab is correct

---

## Accessibility

### Keyboard Navigation
- [ ] Can tab through all interactive elements
- [ ] Focus indicators are visible
- [ ] Skip to content link works (if present)
- [ ] Gallery can be navigated with keyboard

### Screen Reader
**Test with screen reader (optional but recommended):**
- [ ] Headings are properly structured (H1 → H2 → H3)
- [ ] Images have alt text (or decorative images are hidden)
- [ ] Links have descriptive text (not "click here")
- [ ] Form inputs have labels (if any)

### Color Contrast
- [ ] Text has sufficient contrast on background
- [ ] Links are distinguishable from text
- [ ] Headings are readable
- [ ] Button text is readable

---

## CSS Fixes Verification

### Bootstrap Replacement
- [ ] Resume grid layout works without Bootstrap
- [ ] Columns stack correctly on mobile
- [ ] Spacing utilities work (mb-*, py-*, etc.)
- [ ] Text utilities work (text-uppercase, text-muted, etc.)
- [ ] No broken layouts

### CSS Variables
- [ ] `--primary-color` displays correctly (#54B689)
- [ ] `--theme` background works
- [ ] Headings use `--heading-color`
- [ ] Links use `--link-color`
- [ ] No `undefined` or blank colors

### Custom CSS
- [ ] No syntax errors in custom.css
- [ ] Heading colors apply correctly
- [ ] Resume styles work
- [ ] Gallery styles apply

### Font Awesome Icons
- [ ] Icons display in resume (not boxes or X's)
- [ ] fa-phone-square shows phone icon
- [ ] fa-envelope-square shows envelope
- [ ] fa-globe shows globe
- [ ] fa-map-marker-alt shows location pin

---

## Git & Deployment

### Repository
- [ ] `.gitignore` includes `resources/_gen/`
- [ ] No generated files in git (check `git status`)
- [ ] Large images optimized (< 5MB each)
- [ ] Commit messages are descriptive

### Build for Production
- [ ] `hugo` builds successfully
- [ ] `public/` directory created
- [ ] All pages present in `public/`
- [ ] Assets are minified
- [ ] No .DS_Store or temp files

### Deployment (GitHub Pages)
- [ ] Site builds on GitHub Actions (if using)
- [ ] CNAME file included for custom domain
- [ ] Site accessible at https://ttyt.me
- [ ] HTTPS works correctly
- [ ] No mixed content warnings

---

## Regression Testing

After making changes, verify these haven't broken:
- [ ] Existing blog posts still render correctly
- [ ] Gallery still works
- [ ] Resume data still displays
- [ ] Dark/light theme toggle still works
- [ ] Mobile layout still responsive

---

## Documentation

- [ ] CLAUDE.md is up to date
- [ ] README.md reflects current state
- [ ] CLEANUP_PLAN.md tasks completed/updated
- [ ] THEME_ARCHITECTURE.md is accurate
- [ ] IMAGE_OPTIMIZATION.md instructions work

---

## Known Issues Resolved

Verify these critical fixes from CLEANUP_PLAN.md:

### Phase 1: Critical Fixes
- [ ] ✅ Bootstrap CSS dependency resolved (custom CSS replaces it)
- [ ] ✅ Font Awesome CSS included and working
- [ ] ✅ CSS variables `--primary` and `--theme` defined
- [ ] ✅ custom.css syntax errors fixed (no quoted colors)
- [ ] ✅ main.js file exists (no 404 error)

### Phase 2: Refactoring
- [ ] ✅ Gallery inline styles converted to CSS classes
- [ ] ✅ resume_contact.html documented (or implemented)
- [ ] ✅ Design system CSS variables complete

### Phase 3: Performance
- [ ] ✅ resources/_gen/ in .gitignore
- [ ] ✅ Generated resources removed from git
- [ ] ⏳ Images optimized (see IMAGE_OPTIMIZATION.md)

### Phase 4: Documentation
- [ ] ✅ theme.toml updated with correct info
- [ ] ✅ Testing checklist created (this file!)

---

## Pre-Commit Checklist

Before committing changes:
- [ ] Tested locally with `hugo server`
- [ ] No console errors
- [ ] Visual inspection looks good
- [ ] Relevant tests from above completed
- [ ] Commit message is descriptive

---

## Production Deployment Checklist

Before deploying to production:
- [ ] All critical tests pass
- [ ] Build succeeds: `hugo`
- [ ] Site tested on multiple devices
- [ ] Images optimized
- [ ] No broken links
- [ ] Analytics working (if enabled)
- [ ] Backup taken (if applicable)

---

## Quick Smoke Test (30 seconds)

Minimal test for quick changes:
1. [ ] `hugo server -D`
2. [ ] Open http://localhost:8081
3. [ ] Click through: Home → Posts → Resume → Gallery
4. [ ] Check browser console (no errors)
5. [ ] Looks good? ✅

---

## Full Test (10 minutes)

Comprehensive test before major releases:
1. [ ] All visual tests
2. [ ] All responsive breakpoints
3. [ ] Both light and dark themes
4. [ ] All functionality tests
5. [ ] CSS/JS validation
6. [ ] One browser from each family (Chrome, Firefox, Safari)
7. [ ] Production build test
8. [ ] Deploy to staging (if available)

---

## Test Results Log

Track test results here:

### Test Run: [Date]
- **Tester:** [Your name]
- **Hugo Version:** [Run `hugo version`]
- **Branch:** [Git branch]
- **Result:** ✅ Pass / ❌ Fail
- **Issues Found:** [None / List issues]
- **Notes:** [Any observations]

---

**Example:**
### Test Run: 2024-11-24
- **Tester:** Tim
- **Hugo Version:** 0.144.1
- **Branch:** claude/review-hugo-cleanup-*
- **Result:** ✅ Pass
- **Issues Found:** None
- **Notes:** Full refactor complete, all CSS dependencies resolved

---

## Continuous Testing

Set up automated testing (optional but recommended):

### HTML Validation
```bash
# Install validator
npm install -g html-validator-cli

# Validate built site
hugo && html-validator --url=http://localhost:8081 --verbose
```

### Link Checking
```bash
# Install link checker
npm install -g broken-link-checker

# Check for broken links
blc http://localhost:8081 -ro
```

### Accessibility Testing
```bash
# Install pa11y
npm install -g pa11y

# Test accessibility
pa11y http://localhost:8081
```

---

## Questions or Issues?

If tests fail:
1. Check browser console for errors
2. Review recent changes in git
3. Consult THEME_ARCHITECTURE.md
4. Review CLEANUP_PLAN.md for known issues
5. Test in isolated environment
6. Roll back changes if needed

---

**Last Updated:** 2024-11-24
**Refactor Phase:** Complete
**Status:** All critical fixes implemented ✅
