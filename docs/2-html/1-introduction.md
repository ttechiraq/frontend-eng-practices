---
sidebar_position: 1
---

# Introduction

This HTML Engineering Guide defines the standards for authoring semantic, accessible, and performant HTML across our frontend projects. It focuses on modern practices aligned with the WHATWG HTML Living Standard, WCAG 2.2, and current guidance from MDN and web.dev.

## Terminology

This guide uses [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) terms:

- **MUST / MUST NOT**: Absolute requirements with no exceptions.
- **SHOULD / SHOULD NOT**: Strong recommendations with rare, documented exceptions.
- **MAY**: Optional suggestions or acceptable alternatives.

Imperative statements (for example, “Use `<main>` for page content”) are normative. Document any justified exceptions in code comments or team decision records.

## Guide Notes

- Examples are illustrative, not exhaustive. Alternative implementations are acceptable if they comply with the stated rules.
- Optional formatting in examples (line breaks, attribute ordering) is non-normative unless explicitly stated.
- Prefer semantic HTML over ARIA. When ARIA is required, follow the first rule of ARIA: **no ARIA is better than bad ARIA**.

## What This Guide Covers

- Document structure and required head metadata.
- Semantic elements for layout and content.
- Accessibility practices aligned with WCAG 2.2 and ARIA Authoring Practices.
- Forms, validation, and accessible error handling.
- Media and images, responsive patterns, and modern formats (AVIF/WebP).
- Meta tags for SEO, social sharing, and structured data.
- Performance techniques (resource hints, critical rendering path, Core Web Vitals).
- Modern HTML features (dialog, popover, details/summary, inert, template/slot).
