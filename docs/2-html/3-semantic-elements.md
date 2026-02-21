---
sidebar_position: 3
---

# Semantic elements

Semantic HTML means choosing elements based on the **meaning** of the content rather than its visual appearance. This is fundamental to accessibility, SEO (Search Engine Optimization), and long-term maintainability. Screen readers, search engine crawlers, and browser features (like reader mode) all rely on semantic structure to understand and present content correctly.

MUST prefer semantic elements over generic containers (`<div>`, `<span>`) whenever a meaningful alternative exists.

## Landmarks and layout

Landmark elements define the major regions of a page. Assistive technologies expose these as navigation shortcuts, allowing users to jump directly to sections like navigation or main content.

| Element | Purpose | Notes |
|---|---|---|
| `<header>` | Introductory content or navigational aids | Typically contains logo, site title, primary nav. Can also be used inside `<article>`/`<section>`. |
| `<nav>` | Major navigation blocks | Use `aria-label` when multiple `<nav>` elements exist on one page (e.g., "Primary", "Footer"). |
| `<main>` | Primary page content | Exactly one per page. Screen readers use it as a skip target. |
| `<aside>` | Tangentially related content | Sidebars, related links, pull quotes, ads. |
| `<footer>` | Footer for the nearest sectioning ancestor | Copyright, legal links, secondary nav. Can also appear inside `<article>`/`<section>`. |

:::tip[good]

```html
<header>
  <a href="/" aria-label="Homepage">Logo</a>
  <nav aria-label="Primary">
    <ul>
      <li><a href="/about">About</a></li>
      <li><a href="/contact">Contact</a></li>
    </ul>
  </nav>
</header>
<main>
  <article>...</article>
</main>
<aside aria-label="Related articles">...</aside>
<footer>...</footer>
```

:::

:::warning[bad]

```html
<div class="header">
  <div class="logo">Logo</div>
  <div class="nav">
    <div class="nav-item"><a href="/about">About</a></div>
    <div class="nav-item"><a href="/contact">Contact</a></div>
  </div>
</div>
<div class="main">
  <div class="article">...</div>
</div>
<div class="sidebar">...</div>
<div class="footer">...</div>
```

:::

## Sectioning content

Sectioning elements create sub-regions within the document, each with their own heading hierarchy:

- **`<article>`** — Self-contained content that makes sense independently (blog post, news story, product card, comment). A useful test: would this content make sense if syndicated in an RSS feed?
- **`<section>`** — Thematic grouping of content within a page or article. Each section SHOULD start with a heading. If you cannot think of a meaningful heading, a `<div>` may be more appropriate.
- **`<figure>` and `<figcaption>`** — An image, diagram, code snippet, or other media with an optional caption. The caption describes or annotates the figure content.
- **`<aside>`** — Content that is related to but separate from the surrounding flow — sidebars, pull quotes, glossary terms, advertisement blocks.

```html
<article>
  <h2>Building Accessible Forms</h2>
  <p>Forms are one of the most critical areas for accessibility...</p>

  <section>
    <h3>Label Association</h3>
    <p>Every input must have an associated label...</p>
  </section>

  <figure>
    <img src="/img/form-example.png" alt="A form with properly associated labels" />
    <figcaption>Example of a form with explicit label-input associations.</figcaption>
  </figure>
</article>
```

## Text-level semantics

These inline elements convey specific meaning about their content. Using them correctly helps assistive technologies communicate intent and allows browsers/search engines to process content more intelligently.

| Element | Purpose | Example |
|---|---|---|
| `<strong>` | Strong importance (not just bold) | `<strong>Warning:</strong> This action is irreversible.` |
| `<em>` | Stress emphasis (not just italic) | `You <em>must</em> agree to the terms.` |
| `<code>` | Inline code fragment | `Use <code>Array.from()</code> to convert iterables.` |
| `<kbd>` | Keyboard input | `Press <kbd>Ctrl</kbd>+<kbd>S</kbd> to save.` |
| `<samp>` | Sample output from a program | `The console displayed <samp>OK</samp>.` |
| `<var>` | Mathematical or programming variable | `Let <var>x</var> be the total count.` |
| `<time>` | Machine-readable date/time | `<time datetime="2025-03-15">March 15, 2025</time>` |
| `<abbr>` | Abbreviation with expansion | `<abbr title="Hypertext Markup Language">HTML</abbr>` |
| `<mark>` | Highlighted/relevant text | Search results highlighting a matched term. |
| `<cite>` | Title of a creative work | `<cite>The Pragmatic Programmer</cite>` |

## Headings

Headings create a hierarchical outline that screen readers expose as a navigable table of contents. Users frequently navigate pages by jumping between headings.

- Exactly one `<h1>` per page, representing the page's primary topic.
- Descend levels sequentially: `<h1>` → `<h2>` → `<h3>`. Never skip levels (e.g., `<h1>` → `<h3>`).
- Each `<section>` or `<article>` SHOULD start with a heading at the appropriate level.
- **MUST NOT** use heading elements purely for visual sizing. Use CSS for font sizes; headings communicate document structure, not appearance.

## Lists and tables

### Lists

- Use `<ul>` (unordered list) for items where order does not matter.
- Use `<ol>` (ordered list) for items where sequence is significant (steps, rankings).
- Use `<dl>` (description list) for key-value pairs (glossary terms, metadata).
- MUST NOT use `<br>` tags or paragraphs to simulate list formatting.

### Tables

- Use `<table>` **only** for tabular data — never for page layout.
- Include `<thead>` for header rows and `<tbody>` for data rows.
- Use `<th scope="col">` for column headers and `<th scope="row">` for row headers so screen readers announce cell associations correctly.
- Add `<caption>` for a brief, accessible description of the table.

```html
<table>
  <caption>Quarterly revenue by region</caption>
  <thead>
    <tr>
      <th scope="col">Region</th>
      <th scope="col">Q1</th>
      <th scope="col">Q2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">North America</th>
      <td>$1.2M</td>
      <td>$1.5M</td>
    </tr>
  </tbody>
</table>
```

## Avoiding div/span soup

"Div soup" refers to markup that uses `<div>` and `<span>` for everything, stripping the document of any semantic meaning. This harms accessibility, makes the code harder to maintain, and gives search engines less information to work with.

- `<div>` is acceptable as a **layout primitive** — a wrapper for CSS Grid/Flexbox containers — when no semantic element fits the purpose.
- `<span>` is for inline grouping when you need a styling hook but the content carries no inherent structural meaning.
- When in doubt, ask: "Does this content have a specific role or meaning?" If yes, choose the semantic element. If no, a `<div>` or `<span>` is appropriate.
