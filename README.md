# quantike.xyz

Personal website for Ike (@quantike) - thoughts on systems and software.

## Local Development

This site is built with [Zola](https://www.getzola.org/), a fast static site generator written in Rust.

### Prerequisites

Install Zola:
```bash
# macOS
brew install zola

# Or download from https://www.getzola.org/documentation/getting-started/installation/
```

### Running Locally

Start the development server:
```bash
zola serve
```

The site will be available at `http://127.0.0.1:1111` with hot-reloading enabled.

### Building

Build the site for production:
```bash
zola build
```

Output will be in the `public/` directory.

### Project Structure

- `content/` - Markdown content for pages and posts
- `static/` - Static assets (images, etc.)
- `templates/` - Custom template overrides
- `themes/`
  - `serene/` - Serene theme (git submodule)
  - `custom/` - Custom Gruvbox syntax themes (JSON)
- `config.toml` - Site configuration

## Customization

### Theme & Colors

This site uses the [Serene](https://github.com/isunjn/serene) theme by isunjn as a base, with extensive customization:

**Color Scheme**: Full Gruvbox palette (light and dark modes)
- Light mode: Cream background (`#fbf1c7`) with dark gray text
- Dark mode: Hard variant dark background (`#1d2021`) with light cream text
- Aqua accent color (`#8ec07c` dark / `#689d6a` light)

**Customization Files**:
- `templates/_custom_css.html` - Gruvbox color variables and CSS overrides
- `templates/_head_extend.html` - SEO and Open Graph meta tags
- `templates/home.html`, `templates/post.html`, `templates/prose.html` - Template overrides for Zola 0.22+ syntax highlighting
- `themes/custom/gruvbox-light.json` - Gruvbox Light syntax highlighting theme
- `themes/custom/gruvbox-dark.json` - Gruvbox Dark syntax highlighting theme

### Syntax Highlighting

Using Zola 0.22+ with the Giallo highlighter and custom Gruvbox themes:
- Light theme uses Gruvbox Light colors for code blocks
- Dark theme uses Gruvbox Dark Hard colors for code blocks
- Automatic theme switching based on system preferences or manual toggle

### Why `themes/custom/`?

The custom Gruvbox syntax themes are stored in `themes/custom/` (tracked by this repo) rather than `themes/serene/highlight_themes/` (inside the submodule) to:
1. Keep the Serene submodule clean and updatable
2. Ensure custom themes are version-controlled in this repository
3. Avoid CI/CD issues where submodule files wouldn't be available during builds

## Deployment

This site is deployed to GitHub Pages. Push to `main` branch to trigger deployment via GitHub Actions.

The workflow uses Zola 0.22.0 to build the site and deploy to GitHub Pages.
