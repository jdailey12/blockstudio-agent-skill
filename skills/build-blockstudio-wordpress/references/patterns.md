# Blockstudio Implementation Patterns

These are stable patterns distilled from Blockstudio docs. Treat them as starting points, then confirm details against official docs or the project's `blockstudio-llm.txt`.

## Project Triage

Before editing:

1. Look for `blockstudio.json`, `blockstudio/`, `pages/`, `patterns/`, and theme files.
2. Inspect existing blocks for namespace, field naming, template style, Tailwind usage, and escaping conventions.
3. Prefer existing local helpers over inventing new abstractions.
4. If the project exposes `/blockstudio-llm.txt`, use it for framework details tied to the installed version.

## Block Folder

Default shape:

```text
blockstudio/
  example-block/
    block.json
    index.php
    style.css
```

Use `index.twig` when the project already uses Timber/Twig. Use `index.blade.php` only when the project already has Blade support.

## Block JSON

Minimum Blockstudio block:

```json
{
  "$schema": "https://blockstudio.dev/schema/block",
  "name": "my-theme/example",
  "title": "Example",
  "category": "widgets",
  "icon": "star-filled",
  "blockstudio": true
}
```

With fields:

```json
{
  "$schema": "https://blockstudio.dev/schema/block",
  "name": "my-theme/card",
  "title": "Card",
  "category": "widgets",
  "icon": "id",
  "blockstudio": {
    "attributes": [
      { "id": "heading", "type": "text", "label": "Heading" },
      { "id": "body", "type": "textarea", "label": "Body" }
    ]
  }
}
```

## Template Variables

Common PHP template variables:

- `$a` - attributes/field values.
- `$b` - block metadata.
- `$c` - parent context when configured.
- `$content` - inner block content for nested/rendered blocks.

Use normal WordPress escaping: `esc_html()`, `esc_attr()`, `esc_url()`, `wp_kses_post()`.

## Field Organization

Use groups and tabs for complex inspector UIs. Anonymous groups organize fields visually while keeping field access direct. Named groups prefix child field IDs; use Blockstudio helpers such as `bs_get_group()` when appropriate in an existing project.

Conditional logic uses arrays of conditions. Inner arrays are AND groups, outer arrays are OR groups.

```json
{
  "id": "buttonUrl",
  "type": "link",
  "label": "Button Link",
  "conditions": [[{ "id": "showButton", "operator": "==", "value": true }]]
}
```

## Repeaters

Use repeaters for structured lists. In PHP templates, iterate over the array and escape each field.

```php
<?php foreach (($a['items'] ?? []) as $item) : ?>
  <h3><?php echo esc_html($item['heading'] ?? ''); ?></h3>
<?php endforeach; ?>
```

For RichText inside repeaters, use the official RichText docs and bracket notation like `items[index].heading`.

## File-Based Pages

Use `page.json` plus `index.php` for pages controlled by files.

```json
{
  "name": "landing",
  "title": "Landing Page",
  "postType": "page",
  "postStatus": "publish",
  "templateLock": "all",
  "blockEditingMode": "disabled"
}
```

Use `key` attributes for sections whose editor-modified content should survive template syncs.

```html
<h1 key="title" blockEditingMode="contentOnly">Default Title</h1>
```

## Tailwind

Enable Tailwind only when it fits the project. Blockstudio compiles Tailwind server-side and previews it in the editor.

```json
{
  "$schema": "https://blockstudio.dev/schema/blockstudio",
  "tailwind": {
    "enabled": true
  }
}
```

Use a `classes` field with `"tailwind": true` when editors should choose utility classes.

## Components

Use component blocks for reusable UI pieces rendered programmatically and hidden from the editor inserter.

```json
{
  "$schema": "https://blockstudio.dev/schema/block",
  "name": "my-theme/card",
  "title": "Card",
  "blockstudio": {
    "component": true,
    "attributes": [
      { "id": "title", "type": "text", "label": "Title" }
    ]
  }
}
```

Render components with Blockstudio rendering helpers or `<bs:...>` syntax after confirming the project's preferred surface.

## Full-Stack Blocks

For application-like blocks, Blockstudio supports co-located server and client logic such as `db.php`, `rpc.php`, `cron.php`, `script.inline.js`, and Interactivity API markup. Read the full-stack and interactivity docs before implementing because these APIs are more sensitive to the installed version.
