# Kanha Blog - Hugo Setup

This is a personal blog using Hugo with the Terminal theme.

## Project Structure

```
content/          # Blog posts (Markdown files)
  posts/          # Individual blog posts
  test-note.md    # Example shortcode test

layouts/          # Override theme templates
  shortcodes/     # Custom shortcodes
    note.html     # Note/warning shortcode

assets/           # Custom assets
  css/            # Custom CSS files
    note.css      # Note shortcode styling

themes/terminal/  # Terminal theme (git submodule)
```

## Adding Content

1. **New blog post**: Create `.md` file in `content/posts/` with front matter:
   ```yaml
   ---
   title: "Post Title"
   date: YYYY-MM-DD
   ---
   ```

2. **Assets**: Place in `static/` (images, PDFs) or `assets/` (CSS, JS).

3. **Custom CSS**: Add to `assets/css/` - automatically loaded by theme.

## Shortcodes

- **Note shortcode**: `{{< note type="info|warning|success|danger|tip" title="Optional Title" >}}Content{{< /note >}}`
- CSS: `assets/css/note.css`
- Template: `layouts/shortcodes/note.html`

## Development

- Run locally: `hugo server`
- Build site: `hugo`
- Output: `public/` directory

## Theme Customization

- Override theme files by placing them in project `layouts/` directory
- CSS variables in `:root` (main.css) control colors
- Terminal theme loads all CSS from `assets/css/` automatically