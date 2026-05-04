# Shopify Liquid Components

A collection of production-ready Shopify theme components built with Liquid, HTML, and CSS. Each component follows Shopify 2.0 section architecture — schema-driven settings, reusable snippets, responsive layout, and graceful empty states.

---

## Components

### `sections/featured-collection.liquid`

Renders a responsive product grid from any merchant-selected collection. Every display option is configurable through the Shopify theme editor — no code changes required after installation.

**What it demonstrates:**
- `collections[]` global object — resolve a collection by handle from schema settings
- `{% for product in collection.products limit: n %}` — loop with dynamic limit from settings
- `{% render 'product-card', product: product %}` — snippet composition with parameter passing
- `{% if collection.products_count > section.settings.products_to_show %}` — conditional view-all logic
- Section-scoped CSS via `.section-{{ section.id }}` — prevents style collisions across multiple instances
- `{% schema %}` JSON block — full theme editor integration with `collection` picker, `range` sliders, `checkbox` toggles

### `snippets/product-card.liquid`

A self-contained product card that handles every standard display case: regular price, sale price with discount percentage, sold-out state, missing image fallback, and products with price ranges.

**What it demonstrates:**
- `{% assign %}` — pre-compute sale state and discount percentage once, reference throughout
- Arithmetic filter chaining — `| minus: | times: 100 | divided_by:` for discount calculation
- `| money` — locale-aware currency formatting
- `| img_url: '500x'` — Shopify CDN image resizing
- `| divided_by: product.featured_image.aspect_ratio | round` — calculated image height for layout stability
- `| default:` — safe fallback for missing alt text
- `| upcase` — vendor display formatting
- `{% unless product.available %}` — inverted conditional for sold-out badge
- `{{ 'product-1' | placeholder_svg_tag }}` — editor-safe placeholder for products without images
- `product.price_varies` — handles multi-variant price ranges correctly
- `aria-label` on price containers — accessible markup

---

## Liquid Concepts Covered

| Concept | Where Used |
|---|---|
| Objects | `product`, `collection`, `section.settings`, `product.featured_image` |
| Tags | `for`, `if`, `unless`, `assign`, `render`, `comment` |
| Filters | `money`, `img_url`, `escape`, `upcase`, `default`, `minus`, `times`, `divided_by`, `round`, `placeholder_svg_tag` |
| Schema | `collection` picker, `text`, `textarea`, `range`, `checkbox`, `presets` |
| Snippets | Parameter passing via `render` tag |

---

## Installation

**Drop into any Shopify 2.0 theme:**

```bash
# Copy files into your theme
cp sections/featured-collection.liquid  your-theme/sections/
cp snippets/product-card.liquid         your-theme/snippets/
```

Then add the section through the theme editor:
1. Open **Online Store → Themes → Customize**
2. Click **Add section**
3. Select **Featured Collection**
4. Choose a collection, set product count and columns

**Or install via Shopify CLI:**

```bash
shopify theme push --only sections/featured-collection.liquid snippets/product-card.liquid
```

---

## Schema Reference

`featured-collection.liquid` exposes the following settings in the theme editor:

| Setting | Type | Default | Description |
|---|---|---|---|
| `heading` | text | "Featured Collection" | Section heading |
| `subheading` | textarea | — | Optional subheading below heading |
| `collection` | collection | — | Collection to display products from |
| `products_to_show` | range (2–12) | 4 | Number of products to render |
| `columns_desktop` | range (2–4) | 4 | Grid columns on desktop |
| `show_view_all` | checkbox | true | Show link to full collection page |

---

## Design Decisions

**Why section-scoped CSS?**
Using `.section-{{ section.id }}` as a CSS prefix means merchants can add multiple instances of the same section to a page without styles colliding. Each instance gets its own scoped variable for column count, allowing different grid layouts side by side.

**Why pre-compute sale state with `{% assign %}`?**
Rather than evaluating `product.compare_at_price > product.price` multiple times across the template (once for the badge, once for the price display, once for discount math), assigning it to `on_sale` once keeps the logic centralized and the template readable.

**Why calculate image height from aspect ratio?**
Setting explicit `width` and `height` attributes on `<img>` tags prevents layout shift (CLS) as images load — a Core Web Vitals requirement. The height is computed from the image's actual aspect ratio using Liquid's arithmetic filters.

---

## Status

Complete. Ready to drop into any Shopify 2.0 theme.

---
