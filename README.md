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
- `static/` - Static assets (CSS, images, etc.)
- `themes/serene/` - Serene theme (submodule)
- `config.toml` - Site configuration

## Deployment

This site is deployed to GitHub Pages. Push to `main` branch to trigger deployment via GitHub Actions.

## Theme

Using the [Serene](https://github.com/isunjn/serene) theme by isunjn.
