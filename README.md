# Flouri.sh Documentation

This is the documentation website for [Flouri.sh](https://github.com/guilyx/flouri), an AI-powered terminal environment.

## Development

### Prerequisites

- Ruby 3.0+ (check with `ruby --version`)
- Bundler (Ruby gem manager)

### Installing Bundler

If you don't have Bundler installed:

```bash
# Install Bundler (user installation, no sudo required)
gem install bundler --user-install

# Add to your PATH (add this to your ~/.zshrc or ~/.bashrc to make it permanent)
export PATH="$HOME/.local/share/gem/ruby/3.0.0/bin:$PATH"
```

To make the PATH change permanent, add this line to your shell configuration file:
- For zsh: `~/.zshrc`
- For bash: `~/.bashrc`

```bash
echo 'export PATH="$HOME/.local/share/gem/ruby/3.0.0/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Local Setup

1. Install dependencies:
   ```bash
   bundle install
   ```

2. Run Jekyll locally:
   ```bash
   bundle exec jekyll serve
   ```

3. Open http://localhost:4000 in your browser

### Building

```bash
bundle exec jekyll build
```

The site will be built to `_site/`.

## Deployment

This site is automatically deployed to GitHub Pages when changes are pushed to the `main` branch.

The deployment is handled by GitHub Actions (see `.github/workflows/pages.yml`).

## Structure

- `_config.yml` - Jekyll configuration
- `index.md` - Homepage
- `getting-started.md` - Getting started guide
- `configuration.md` - Configuration documentation
- `security.md` - Security documentation
- `architecture.md` - Architecture documentation
- `api.md` - API reference
- `plugins.md` - Plugin system documentation
- `developer.md` - Developer guide

## Theme

This site uses the [Just the Docs](https://just-the-docs.github.io/just-the-docs/) theme for Jekyll.

## Contributing

To contribute to the documentation:

1. Make your changes
2. Test locally with `bundle exec jekyll serve`
3. Submit a pull request
