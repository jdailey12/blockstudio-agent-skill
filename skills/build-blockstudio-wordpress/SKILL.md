---
name: build-blockstudio-wordpress
description: Build with Blockstudio, also written Block Studio, for WordPress and Gutenberg. Use when the user mentions Blockstudio/Block Studio, WordPress custom blocks, block themes, block.json, file-based pages, patterns, Tailwind in Blockstudio, ACF block migration, Blockstudio attributes/fields, RichText, InnerBlocks, Interactivity API, RPC, database, cron, component blocks, or Blockstudio registries.
---

# Build Blockstudio WordPress

Use this skill to build WordPress blocks, pages, patterns, and reusable block systems with Blockstudio.

Always consult the official Blockstudio docs for current API details and examples: https://www.blockstudio.dev/docs. When a project exposes `blockstudio-llm.txt`, fetch or read it before making framework-specific decisions; it is the canonical full context file for the installed Blockstudio version. If no project-specific file exists and network access is available, use the upstream context file as a fallback: https://raw.githubusercontent.com/inline0/blockstudio/refs/heads/main/includes/llm/blockstudio-llm.txt.

## Quick Reference

Blockstudio is file-system based. Prefer writing files over asking the user to configure blocks in wp-admin.

Common locations:

- `blockstudio/` - Theme or configured directory for blocks and components.
- `blockstudio/<block>/block.json` - WordPress block metadata plus Blockstudio configuration.
- `blockstudio/<block>/index.php` - PHP template.
- `blockstudio/<block>/index.twig` - Twig template when Timber is installed.
- `blockstudio/<block>/style.css`, `style.scoped.css`, `script.js`, `script.inline.js` - Block-owned assets.
- `pages/<page>/page.json` and `pages/<page>/index.php` - File-based pages.
- `patterns/<pattern>/pattern.json` and `patterns/<pattern>/index.php` - File-based patterns.
- `blockstudio.json` - Global Blockstudio settings, including Tailwind and dev tools.

Important URLs:

- Block schema: `https://blockstudio.dev/schema/block`
- Settings schema: `https://blockstudio.dev/schema/blockstudio`
- Main docs: https://www.blockstudio.dev/docs
- AI context docs: https://www.blockstudio.dev/docs/dev/ai
- Upstream LLM context fallback: https://raw.githubusercontent.com/inline0/blockstudio/refs/heads/main/includes/llm/blockstudio-llm.txt

Use `references/docs-map.md` to choose official docs pages. Use `references/patterns.md` for compact implementation patterns.

## Build A Block

Create a folder with `block.json` and a template. Include the Blockstudio block schema and keep the block name namespaced to the theme or plugin.

```json
{
  "$schema": "https://blockstudio.dev/schema/block",
  "name": "my-theme/testimonial",
  "title": "Testimonial",
  "category": "widgets",
  "icon": "format-quote",
  "description": "Quote with author details.",
  "blockstudio": {
    "attributes": [
      { "id": "quote", "type": "textarea", "label": "Quote" },
      { "id": "authorName", "type": "text", "label": "Author name" },
      { "id": "authorRole", "type": "text", "label": "Author role" },
      {
        "id": "photo",
        "type": "files",
        "label": "Photo",
        "multiple": false,
        "allowedTypes": ["image"],
        "returnFormat": "object"
      }
    ]
  }
}
```

In PHP templates, field values are available through `$a`, the shorthand for Blockstudio attributes. Escape output like normal WordPress PHP.

```php
<figure class="rounded-xl border border-slate-200 p-6">
  <?php if (!empty($a['quote'])) : ?>
    <blockquote class="text-lg italic">
      <?php echo esc_html($a['quote']); ?>
    </blockquote>
  <?php endif; ?>

  <figcaption class="mt-4 flex items-center gap-3">
    <?php if (!empty($a['photo']['url'])) : ?>
      <img class="h-12 w-12 rounded-full object-cover" src="<?php echo esc_url($a['photo']['url']); ?>" alt="<?php echo esc_attr($a['photo']['alt'] ?? ''); ?>">
    <?php endif; ?>
    <span>
      <strong><?php echo esc_html($a['authorName'] ?? ''); ?></strong>
      <small class="block text-slate-500"><?php echo esc_html($a['authorRole'] ?? ''); ?></small>
    </span>
  </figcaption>
</figure>
```

For editable text directly in the editor canvas, use a `richtext` field and `<RichText />`. For nested content, use `<InnerBlocks />`. Check the official component docs before writing complex component props.

## Build File-Based Pages

Use pages when the user wants complete page scaffolds, locked templates, client-editable content, reusable landing pages, or version-controlled page structure.

```json
{
  "name": "about",
  "title": "About Us",
  "slug": "about",
  "postType": "page",
  "postStatus": "publish",
  "templateLock": "all",
  "blockEditingMode": "disabled"
}
```

Use `key` attributes to preserve editor content across template syncs. Use `blockEditingMode="contentOnly"` on text the client should edit.

```html
<block name="core/cover" key="hero">
  <h1 blockEditingMode="contentOnly">About Us</h1>
  <p blockEditingMode="contentOnly">Our story.</p>
</block>
```

## Use Tailwind

Blockstudio supports Tailwind CSS v4 without a Node build step. Enable it in `blockstudio.json` when the project wants Tailwind utilities in block templates or page templates.

```json
{
  "$schema": "https://blockstudio.dev/schema/blockstudio",
  "tailwind": {
    "enabled": true
  }
}
```

If the project uses theme tokens, put durable tokens in the Tailwind v4 CSS-first config or follow the local project conventions. Avoid inventing a second styling system when the theme already has CSS, SCSS, design tokens, or utility helpers.

## Migrate From ACF Blocks

Translate ACF concepts into Blockstudio files:

- ACF field group -> `blockstudio.attributes` in `block.json`.
- `get_field('title')` -> `$a['title']`.
- `have_rows()` / `the_row()` -> `foreach ($a['items'] as $item)`.
- ACF image field -> `files` field, usually with `multiple: false`.
- Editable WYSIWYG or formatted text -> `richtext` or `wysiwyg` field based on editor needs.

Do not keep ACF helper calls in a Blockstudio template unless the user explicitly wants hybrid legacy behavior.

## Use Dev Tools

When available, Blockstudio dev tools help agent workflows:

- Enable AI context generation in `blockstudio.json` so the project exposes `blockstudio-llm.txt`.
- Use Canvas to visually inspect file-based pages and registered blocks.
- Use Element Grabber to map a rendered frontend element back to the template file that created it.

Confirm the site's Blockstudio settings and local conventions before enabling dev-only features.
