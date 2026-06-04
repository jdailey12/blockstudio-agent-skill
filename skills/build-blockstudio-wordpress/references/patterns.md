# Blockstudio Implementation Patterns

These are stable patterns distilled from Blockstudio docs. Treat them as starting points, then confirm details against official docs or the project's `blockstudio-llm.txt`.

## Contents

- Project triage
- Block folders, block JSON, template variables, fields, repeaters, and file-based pages
- Tailwind, component blocks, and full-stack blocks
- Static/Astro clone workflow
- Content model decision tree and CPT-backed sections
- RichText, media fields, and editor CSS gotchas
- Editor block-error debugging and verification matrix

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

## Static Or Astro Clone Workflow

When converting a static or Astro site into WordPress, inspect the running source site and the running WordPress site. Screenshots are useful references, but the live source site is the source of truth for layout, breakpoints, states, and assets.

Recommended flow:

1. Build or run the Astro/static source site and capture desktop and mobile reference screenshots.
2. Inventory sections, repeated records, assets, links, forms, menus, animations, and responsive states.
3. Decide the content model before writing `block.json` files.
4. Import clone assets into the WordPress Media Library when editors should replace them.
5. Build locked section-level blocks or file-based pages for fixed page structure.
6. Verify the frontend against the running source site.
7. Verify Gutenberg editor usability with the sidebar open, sidebar closed, and each block selected.

Do not rely on manual wp-admin layout assembly for a cloned homepage unless the user wants a flexible page-builder style result. For pixel-perfect clones, locked section blocks with clear fields are usually more stable.

## Content Model Decision Tree

Choose the storage model before choosing fields:

- One-off text, media, links, and style options for a section: Blockstudio attributes.
- Small decorative repeated lists, such as three stats or two CTA buttons: Blockstudio repeater.
- Real records with titles, images, URLs, order, categories, or independent management: custom post type or taxonomy.
- Values used across the site: option storage or WordPress options.
- Queryable values attached to one page or post: post meta.

Avoid giant repeaters for portfolios, projects, team members, services, locations, products, testimonials, articles, or other records editors may manage independently. A section block can configure presentation, filters, limits, and headings while the records live in WordPress content.

When a block renders external data such as CPT records, add a `message` field in the sidebar explaining where editors manage those records. Use `refreshOn: ["save"]` when the editor preview depends on saved external data.

## CPT-Backed Section Pattern

Use this pattern for portfolio grids, project cards, team directories, location lists, testimonial libraries, and similar records.

Register the CPT:

```php
register_post_type('project', [
  'label' => __('Projects', 'theme'),
  'public' => false,
  'show_ui' => true,
  'show_in_menu' => true,
  'supports' => ['title', 'thumbnail', 'page-attributes', 'revisions'],
  'menu_position' => 20,
]);
```

Use `public: false` when records are only cards and do not need public single URLs. Use `show_in_rest: false` when a simple classic edit screen is better than Gutenberg for record management. If order matters, expose `menu_order`, query by it, and make the admin list sortable. Add admin columns for thumbnail, URL/meta, and order; those columns are editing UX, not optional polish.

In the Blockstudio section block:

- Keep headings, intro copy, layout options, filters, limits, and CTA text as block attributes.
- Query the CPT for cards.
- Sort by `menu_order` when editorial ordering matters.
- Add a message field that tells editors to manage records in the CPT menu.
- Keep any old repeater data hidden as fallback until migration is confirmed.

## RichText Gotchas

`<RichText />` can introduce Gutenberg wrappers, inline styles, `contenteditable`, and `white-space: pre-wrap`. Test the actual editor output; do not assume a styled frontend span will behave the same once editable.

Use direct canvas RichText for stable headings, paragraph copy, and short labels. Prefer sidebar fields for:

- Animated or duplicated text, including marquees.
- Structural line breaks and split-line headings.
- URLs, form endpoints, IDs, tracking values, and technical settings.
- Badges or pills where RichText wrappers make styling unpredictable.
- Repeated values where only one canonical source should be editable.

For exact split-line headings, use fields such as `desktopTitleLines` and `mobileTitleLines`, render a useful `aria-label` on the heading, and mark decorative line spans `aria-hidden="true"`. If the editor layout is too constrained, render a simplified editor preview from the same fields.

## Media Field Gotchas

Programmatic defaults for `files` fields can crash previews when values are shaped differently than expected. Attributes might be attachment IDs, arrays, objects, URLs, or stale theme-relative strings depending on how they were seeded.

Guard every media access:

```php
$image = $a['image'] ?? null;
$image_url = '';
$image_alt = '';

if (is_numeric($image)) {
  $image_url = wp_get_attachment_image_url((int) $image, 'large') ?: '';
  $image_alt = get_post_meta((int) $image, '_wp_attachment_image_alt', true) ?: '';
} elseif (is_array($image)) {
  $image_url = $image['url'] ?? '';
  $image_alt = $image['alt'] ?? '';
} elseif (is_string($image)) {
  $image_url = $image;
}
```

For cloned assets, sideload images into the Media Library and preserve alt text when possible. Verify backend media controls show a selected image, not just a frontend fallback URL.

## Editor CSS

Editor CSS is separate from frontend CSS. Gutenberg sidebars, iframes, RichText wrappers, admin chrome, selection outlines, and constrained canvases can change layout.

Use editor-only stylesheets, `add_editor_style()`, Blockstudio editor assets, or small `$isEditor`-conditioned attributes for deterministic preview fixes. Verify styles are present inside the editor iframe. Do not output `<style>` tags from Blockstudio PHP templates as a quick editor fix; they may be stripped or printed as text depending on render context.

Frontend CSS should remain pixel-perfect. Editor CSS may be slightly more compact when necessary to keep editing usable. Large headings often need editor-only max-width or font-size adjustments. Do not let hover states, focus outlines, RichText wrappers, or preview labels resize fixed-format UI.

When fighting stale editor CSS, bump every relevant theme/enqueue version the project uses, including the theme stylesheet header version and any theme constants used for cache busting.

## Debugging Editor Block Errors

Use this checklist for "This block has encountered an error":

- Validate `block.json` against the Blockstudio schema.
- PHP lint every block template.
- Render each block individually with `render_block()`.
- Inspect the actual `$a` shape in post content.
- Confirm media values are valid and guarded.
- Check RichText attribute paths, especially inside repeaters.
- Check for unguarded array access.
- Read the browser console.
- Read `wp-content/debug.log`.

WP-CLI snippet for a page render pass:

```bash
ddev wp eval '
$post = get_page_by_path("home", OBJECT, "page");
foreach (parse_blocks($post->post_content) as $i => $block) {
  $html = render_block($block);
  printf("%02d %s %d bytes\n", $i, $block["blockName"] ?? "(freeform)", strlen($html));
}
'
```

When not using DDEV, drop the `ddev` prefix. A clean PHP lint does not prove attributes are shaped correctly. A frontend render does not prove Gutenberg preview is healthy.

## Verification Matrix

For full page builds and site clones, verify:

- Frontend desktop against the running source site.
- Frontend mobile against the running source site.
- Interactive states such as menu open, expanded cards, carousels, forms, and hover/focus states.
- No horizontal overflow.
- Gutenberg editor with settings sidebar open.
- Gutenberg editor with settings sidebar closed.
- Each section block selected, with useful fields visible.
- Backend media fields show selected/replacable media.
- Repeated record admin list, including columns and ordering.
- Repeated record single edit screen.
- No "This block has encountered an error" notices.

Use browser automation after implementation, not only WP-CLI. Backend verification means usability, not just absence of crashes.
