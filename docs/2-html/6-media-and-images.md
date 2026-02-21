---
sidebar_position: 6
---

# Media and images

Optimize images and media for accessibility, performance, and visual stability. Serve modern formats with responsive delivery to reduce bandwidth while maintaining visual quality.

## Image formats

Modern image formats offer significantly better compression than legacy formats, reducing file sizes and improving load times.

| Format | Full Name | Best For | Browser Support |
|---|---|---|---|
| **AVIF** | AV1 Image File Format | Photos and complex images; best compression ratio (up to 50% smaller than JPEG) | Chrome 85+, Firefox 93+, Safari 17+ |
| **WebP** | Web Picture format | General-purpose; 25–30% smaller than JPEG with broad support | All modern browsers |
| **JPEG** | Joint Photographic Experts Group | Fallback for photos when modern formats are not supported | Universal |
| **PNG** | Portable Network Graphics | Graphics requiring transparency; lossless compression | Universal |
| **SVG** | Scalable Vector Graphics | Icons, logos, illustrations; scales without quality loss | Universal |
| **GIF** | Graphics Interchange Format | Short, simple animations (prefer video for longer animations) | Universal |

- **SHOULD** serve AVIF as the primary format where supported for maximum compression.
- **SHOULD** provide WebP as a fallback for browsers that do not support AVIF.
- **MAY** fall back to JPEG or PNG for legacy browser support.
- Avoid unoptimized BMP (Bitmap) or TIFF (Tagged Image File Format) in production.

## Responsive images

Different devices have different screen sizes and pixel densities. Responsive images serve appropriately sized files so mobile users do not download desktop-sized images.

### srcset and sizes

Use `srcset` with **width descriptors** (`w`) and the `sizes` attribute to let the browser choose the best image for the current viewport and display density:

```html
<img
  srcset="
    /img/hero-400.avif   400w,
    /img/hero-800.avif   800w,
    /img/hero-1200.avif 1200w,
    /img/hero-1600.avif 1600w
  "
  sizes="(max-width: 600px) 100vw, (max-width: 1200px) 50vw, 800px"
  src="/img/hero-800.jpg"
  alt="Team collaborating at a whiteboard"
  width="1600"
  height="900"
  loading="lazy"
  decoding="async"
/>
```

- `srcset` lists available image files with their intrinsic widths.
- `sizes` tells the browser how wide the image will be rendered at various viewport sizes.
- `src` provides a fallback for browsers that do not support `srcset`.
- Width descriptors (`w`) are preferred over density descriptors (`x`) for fluid layouts because they give the browser more flexibility.

### Art direction with `<picture>`

The `<picture>` element allows you to serve entirely different images based on viewport conditions (art direction) or provide format-based fallbacks:

```html
<!-- Format fallback -->
<picture>
  <source srcset="/img/card.avif" type="image/avif" />
  <source srcset="/img/card.webp" type="image/webp" />
  <img src="/img/card.jpg" alt="Product showcase" width="600" height="400" loading="lazy" decoding="async" />
</picture>

<!-- Art direction: different crop for mobile vs desktop -->
<picture>
  <source media="(max-width: 768px)" srcset="/img/hero-mobile.avif" type="image/avif" />
  <source media="(max-width: 768px)" srcset="/img/hero-mobile.webp" type="image/webp" />
  <source srcset="/img/hero-desktop.avif" type="image/avif" />
  <source srcset="/img/hero-desktop.webp" type="image/webp" />
  <img src="/img/hero-desktop.jpg" alt="Panoramic office view" width="1200" height="600" loading="lazy" />
</picture>
```

Use `<picture>` when:
- You need **different crops or layouts** per breakpoint (art direction).
- You need **format fallbacks** with explicit `type` hints.
- For simple resolution switching without art direction, `srcset`/`sizes` on `<img>` is sufficient.

## Alt text

The `alt` attribute provides a text alternative for users who cannot see the image — screen reader users, users with slow connections where images fail to load, and search engine crawlers.

- **MUST** provide meaningful `alt` for all informative images. Describe the content or function, not the visual appearance. For example, "Bar chart showing Q3 revenue growth of 15%" is better than "chart image."
- **SHOULD** use `alt=""` (empty string) for purely decorative images. This tells screen readers to skip the image entirely. Omitting `alt` altogether causes screen readers to announce the file name, which is worse.
- Do not start alt text with "Image of" or "Picture of" — screen readers already announce the element as an image.
- Do not duplicate surrounding caption text verbatim; keep alt text complementary and concise.

## Preventing layout shift (CLS)

CLS (Cumulative Layout Shift) measures unexpected visual movement as a page loads. Images without explicit dimensions are a leading cause of layout shift.

- **MUST** provide `width` and `height` attributes on `<img>` elements. Modern browsers use these to calculate the aspect ratio and reserve space before the image loads, preventing content from jumping.
- For CSS-based sizing, set `width: 100%; height: auto;` in your stylesheet — the browser will still use the HTML attributes to compute the aspect ratio.
- For CSS background images, reserve space using the `aspect-ratio` property or padding-based ratio boxes.

```html
<!-- Browser reserves a 16:9 space before the image loads -->
<img src="/img/photo.jpg" alt="Sunset over the ocean" width="1600" height="900" loading="lazy" />
```

## Loading strategy

Control when and how images load to balance performance with user experience:

| Attribute | Purpose | When to use |
|---|---|---|
| `loading="lazy"` | Defers loading until the image is near the viewport | Below-the-fold images — most images on the page |
| `loading="eager"` | Loads immediately (browser default) | Above-the-fold hero images |
| `fetchpriority="high"` | Tells the browser this resource is high priority | The LCP (Largest Contentful Paint) hero image |
| `fetchpriority="low"` | Tells the browser this resource is low priority | Thumbnails, decorative images far down the page |
| `decoding="async"` | Allows the browser to decode the image off the main thread | Most images — avoids blocking rendering |

Do NOT use `loading="lazy"` on the hero/LCP image — it delays the most important visual element on the page.

## Video and audio

- Prefer native `<video>` and `<audio>` elements with the `controls` attribute so users can play, pause, and seek.
- Avoid `autoplay` with sound — it creates a poor experience and violates WCAG guidelines. If autoplay is needed, add `muted` and provide play/pause controls.
- Include `<track kind="captions">` for all video content to support deaf and hard-of-hearing users. Provide a `srclang` attribute to specify the caption language.
- Provide text transcripts for audio-only content.
- Set `width`, `height`, and `poster` on `<video>` to reserve space and avoid CLS.

```html
<video
  width="1280"
  height="720"
  poster="/img/video-poster.jpg"
  controls
  preload="metadata"
>
  <source src="/video/demo.mp4" type="video/mp4" />
  <source src="/video/demo.webm" type="video/webm" />
  <track kind="captions" src="/captions/demo-en.vtt" srclang="en" label="English" default />
  Your browser does not support the video element.
</video>
```

## SVG (Scalable Vector Graphics)

SVG images are resolution-independent and ideal for icons, logos, and illustrations.

- **Inline SVG** — Embed directly in HTML when you need CSS theming (fill/stroke colors), animation, or accessibility control. Use `role="img"` and either `aria-label` or a `<title>` element inside the SVG for meaningful graphics.
- **`<img>` with SVG src** — Use when the SVG is static and does not need CSS interaction. Provide an `alt` attribute.
- **Decorative SVGs** — Use `aria-hidden="true"` to hide from screen readers.

```html
<!-- Accessible inline SVG icon -->
<svg role="img" aria-label="Warning" viewBox="0 0 24 24" width="24" height="24">
  <title>Warning</title>
  <path d="M12 2L1 21h22L12 2zm0 3.5L19.5 19H4.5L12 5.5z" />
</svg>

<!-- Decorative SVG hidden from assistive technology -->
<svg aria-hidden="true" focusable="false" viewBox="0 0 24 24" width="24" height="24">
  <circle cx="12" cy="12" r="10" />
</svg>
```
