# Lim Family Tree - GitHub Pages

This directory contains the GitHub Pages site for the Lim Family Tree.

## Structure

```
docs/
├── _config.yml          # Jekyll configuration
├── index.md             # Home page
├── generations/         # Family tree diagrams by generation
│   ├── g1-g2.md
│   ├── g3-s1.md
│   ├── g3-s2.md
│   ├── g4-s1.md
│   ├── g4-s2.md
│   ├── g4-s3.md
│   ├── g4-s4.md
│   ├── p0.md
│   ├── p1.md
│   └── p2.md
├── images/              # Family tree diagrams (JPG files)
└── photos/              # Reference photos

```

## GitHub Pages Setup

This site is published from the `/docs` directory on the `main` branch.

### To enable GitHub Pages:
1. Go to repository Settings
2. Under "GitHub Pages" section
3. Select "Deploy from a branch"
4. Choose `main` branch and `/docs` folder
5. Click Save

The site will be available at: `https://betanik.github.io/family/`

## Theme

The site uses the `jekyll-theme-minimal` theme.

## Navigation

- **Home** - Overview of all generations
- **Generations** - Access different family tree sections

## Images

All family tree diagrams are stored in the `images/` folder and are automatically displayed on their corresponding generation pages.

## Local Development

To run locally:
```bash
cd docs
bundle install
bundle exec jekyll serve
```

Then visit `http://localhost:4000/family/` in your browser.
