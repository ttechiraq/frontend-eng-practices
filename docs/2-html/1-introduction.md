---
sidebar_position: 1
---

# Introduction

This HTML Engineering Guide defines the standards for authoring semantic, accessible, and performant HTML across our frontend projects. It focuses on modern practices aligned with the [WHATWG](https://whatwg.org/) (Web Hypertext Application Technology Working Group) HTML Living Standard, [WCAG](https://www.w3.org/TR/WCAG22/) 2.2 (Web Content Accessibility Guidelines), and current guidance from [MDN](https://developer.mozilla.org/) (Mozilla Developer Network) and [web.dev](https://web.dev/).

## Terminology

This guide uses [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) (Request for Comments) terms for normative language:

- **MUST / MUST NOT**: Absolute requirements with no exceptions.
- **SHOULD / SHOULD NOT**: Strong recommendations with rare, documented exceptions.
- **MAY**: Optional suggestions or acceptable alternatives.

The terms **prefer** and **avoid** correspond to **should** and **should not** respectively. Imperative statements (for example, "Use `<main>` for page content") and declarative statements (e.g., "Documents declare a charset") represent absolute requirements equivalent to **must**.

When exceptions to **should** rules are necessary, document the rationale in code comments or team decision records.

## Guide Notes

- Examples are illustrative, not exhaustive. Alternative implementations are acceptable if they comply with the stated rules.
- Optional formatting in examples (line breaks, attribute ordering) is non-normative unless explicitly stated.
- Prefer semantic HTML over ARIA (Accessible Rich Internet Applications). When ARIA is required, follow the first rule of ARIA: **no ARIA is better than bad ARIA**. Misused ARIA can make accessibility worse than having no ARIA at all.

## Glossary

Key abbreviations used throughout this guide:

| Abbreviation | Full Name | Description |
|---|---|---|
| **a11y** | Accessibility | Numeronym: "a" + 11 letters + "y". The practice of making web content usable by everyone, including people with disabilities. |
| **ARIA** | Accessible Rich Internet Applications | A W3C specification that defines attributes to make web content more accessible to assistive technologies such as screen readers. |
| **AVIF** | AV1 Image File Format | A modern image format based on AV1 video compression offering superior compression ratios compared to JPEG and WebP. |
| **CDN** | Content Delivery Network | A geographically distributed network of servers that delivers cached content to users from the nearest location. |
| **CLS** | Cumulative Layout Shift | A Core Web Vital metric measuring unexpected layout movement during the page lifecycle. |
| **CSS** | Cascading Style Sheets | The language used for describing the presentation of HTML documents. |
| **DNS** | Domain Name System | The protocol that translates human-readable domain names to IP addresses. |
| **DOM** | Document Object Model | The tree-structured API that browsers create from parsed HTML, enabling scripts to read and modify page content. |
| **INP** | Interaction to Next Paint | A Core Web Vital metric measuring responsiveness — the delay between user interaction and the next visual update. |
| **JSON-LD** | JavaScript Object Notation for Linked Data | A format for embedding structured data in HTML that search engines use for rich results. |
| **LCP** | Largest Contentful Paint | A Core Web Vital metric measuring perceived load speed — the render time of the largest visible element. |
| **OTP** | One-Time Password | A temporary code used for single-use authentication, commonly sent via SMS or authenticator apps. |
| **RUM** | Real User Monitoring | Performance measurement collected from actual user sessions in production, as opposed to synthetic/lab testing. |
| **SEO** | Search Engine Optimization | Practices that improve a page's visibility and ranking in search engine results. |
| **SVG** | Scalable Vector Graphics | An XML-based vector image format that scales without loss of quality. |
| **TTFB** | Time to First Byte | A performance metric measuring the time from the initial request to the first byte of the server's response. |
| **WCAG** | Web Content Accessibility Guidelines | The W3C standard (current version 2.2) defining success criteria for accessible web content at levels A, AA, and AAA. |
| **WebP** | Web Picture format | A modern image format developed by Google providing lossy and lossless compression with broad browser support. |
| **WHATWG** | Web Hypertext Application Technology Working Group | The standards body that maintains the HTML Living Standard. |
| **XSS** | Cross-Site Scripting | A security vulnerability where attackers inject malicious scripts into web pages viewed by other users. |

## What This Guide Covers

- **Document structure** — Required head metadata, DOCTYPE, language, and charset declarations.
- **Semantic elements** — Landmark, sectioning, and text-level elements for meaningful markup.
- **Accessibility** — WCAG 2.2 compliance, ARIA usage, keyboard support, and assistive technology compatibility.
- **Forms** — Native input types, validation, labeling, grouping, and accessible error handling.
- **Media and images** — Responsive patterns, modern formats (AVIF/WebP), lazy loading, and alt text.
- **Meta tags and SEO** — Open Graph, Twitter Cards, structured data (JSON-LD), and crawl directives.
- **Performance** — Resource hints, script loading strategies, critical rendering path, and Core Web Vitals (LCP, INP, CLS).
- **Modern HTML features** — Dialog, Popover API, details/summary, search element, inert, and template/slot.
