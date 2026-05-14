# html-ppt-generator

A Claude Code skill for generating self-contained HTML presentation slideshows (PPT-style). Produces single `.html` files — no build tools, no dependencies, no CDN required.

## Features

- **Zero dependencies** — everything in one HTML file
- **Dark theme** with customizable color accents
- **Responsive scaling** — auto-fits any viewport (16:9)
- **Multiple navigation** — keyboard, mouse wheel, click, touch, dots
- **Fullscreen mode** with position persistence
- **Smooth transitions** with CSS animations

## Usage

This is a Claude Code skill. Place the `html-ppt-generator` directory in your `.claude/skills/` folder, or let Claude Code load it automatically.

When you ask Claude to create a presentation, slideshow, or PPT in HTML format, this skill guides the generation process.

## Output

A single `.html` file containing:
- Cover slide, content slides, comparison tables, and ending slide
- Built-in slide engine (JS) for navigation and transitions
- Bottom navigation bar with page dots and page numbers

## Files

- `SKILL.md` — Complete skill definition with templates, patterns, and workflow
- `references/` — Reference materials
