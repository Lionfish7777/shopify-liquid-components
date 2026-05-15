# Shopify Liquid Components

We built these as production ready Shopify theme components to understand how
Liquid actually works inside a live Shopify 2.0 theme architecture. Every piece
is configurable through the theme editor with no code changes required after
install. Working through schema integration, section scoping, and snippet
composition with real merchant use cases in mind sharpened how we think about
building for people who will never touch the code.

---

## Components

### `sections/featured-collection.liquid`

Renders a responsive product grid from any merchant-selected collection. Every
display option is configurable through the Shopify theme editor. No code changes
required after installation.

**What it demonstrates**
- `collections[]` global object. Resolve a collection by handle from schema settings
- `{% for product in collection.products limit: n %}`. Loop with dynamic limit from settings
- `{% render 'product-card', product: product %}`. Snippet composition with parameter passing
- `{% if collection.products_count > section.settings.products_to_show %}`. Conditional view-all logic
- Section-scoped CSS via `.section-{{ section.id }}`. Prevents style collisions across multiple instances
- `{% schema %}` JSON block. Full theme editor integration with collection picker, range sliders, and checkbox toggles

### `snippets/product-card.liquid`

A self-contained product card that handles every standard display case. Regular
price, sale price with discount percentage, sold-out state, missing image
fallback, and products with price ranges.

**What it demonstrates**
- `{% assign %}`. Pre-compute sale state and discount percentage once, reference throughout
- Arithmetic filter chaining. `| minus: | times: 100 | divided_by:` for discount calculation
- `| money`. Locale-aware currency formatting
- `| img_url: '500x'`. Shopify CDN image resizing
- `| divided_by: product.featured_image.aspect_ratio | round`. Calculated image height for layout stability
- `| default:`. Safe fallback for missing alt text
- `| upcase`. Vendor display formatting
- `{% unless product.available %}`. Inverted conditional for sold-out badge
- `{{ 'product-1' | placeholder_svg_tag }}`. Editor-safe placeholder for products without images
- `product.price_varies`. Handles multi-variant price ranges correctly
- `aria-label` on price containers. Accessible markup

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

**Drop into any Shopify 2.0 theme**

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

**Or install via Shopify CLI**

```bash
shopify theme push --only sections/featured-collection.liquid snippets/product-card.liquid
```

---

## Schema Reference

`featured-collection.liquid` exposes the following settings in the theme editor:

| Setting | Type | Default | Description |
|---|---|---|---|
| `heading` | text | "Featured Collection" | Section heading |
| `subheading` | textarea | none | Optional subheading below heading |
| `collection` | collection | none | Collection to display products from |
| `products_to_show` | range (2–12) | 4 | Number of products to render |
| `columns_desktop` | range (2–4) | 4 | Grid columns on desktop |
| `show_view_all` | checkbox | true | Show link to full collection page |

---

## Design Decisions

**Why section-scoped CSS?**
Using `.section-{{ section.id }}` as a CSS prefix means merchants can add multiple
instances of the same section to a page without styles colliding. Each instance
gets its own scoped variable for column count, allowing different grid layouts
side by side.

**Why pre-compute sale state with `{% assign %}`?**
Rather than evaluating `product.compare_at_price > product.price` multiple times
across the template, assigning it to `on_sale` once keeps the logic centralized
and the template readable.

**Why calculate image height from aspect ratio?**
Setting explicit `width` and `height` attributes on `<img>` tags prevents layout
shift as images load. This is a Core Web Vitals requirement. The height is
computed from the image's actual aspect ratio using Liquid's arithmetic filters.

---

## Status

Complete. Ready to drop into any Shopify 2.0 theme.

---
