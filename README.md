# style-guide — a Claude Code skill by [Reblis.com](https://reblis.com)

Turn any website into a polished, standalone **brand style-guide HTML page** with one command:

```
/style-guide https://example.com
```

Claude scrapes the site, extracts its brand system — logo, colors, typography, voice — and generates a single self-contained HTML file: full-bleed gradient header with the logo as inline SVG, sticky anchor navigation, numbered sections (mission, voice & tone, language, logo usage, color palette with shade ramps, type system), and [Lucide.dev](https://lucide.dev) icons throughout.

Open [`template-example.html`](template-example.html) in a browser to see a finished guide — it's the exact layout the skill reproduces with your brand's tokens.

## Install

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/Reblis/style-guide-skill ~/.claude/skills/style-guide
```

Restart Claude Code (or start a new session) so the skill registers, then run:

```
/style-guide <url>
```

## What you get

- **One file, zero dependencies.** Everything — logo, icons, swatches — is inlined. The only external reference is Google Fonts. Email it, host it, drop it in a repo; it just works.
- **Logo as real SVG code.** Sanitized so multiple variants coexist on one page (`fill="currentColor"` instead of class-styled fills that clash document-wide).
- **Sticky anchor menu** with frosted-glass backdrop linking every section.
- **Color system done right.** Primary swatches with HEX/RGB specs, 5-step shade ramps grid-aligned under their parent colors, labeled gradient cards.
- **Lucide icons only.** Fetched as exact official SVG code — never emoji or HTML-entity placeholders.
- **Verified output.** The skill screenshots its own output headlessly and inspects it before calling the job done.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- `curl` (for fetching the target site and Lucide icons)
- Optional, for the self-verification step: `chromium` + ImageMagick

## License

MIT © Reblis.com
