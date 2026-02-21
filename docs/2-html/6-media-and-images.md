---
sidebar_position: 6
---

# Media and images

Optimize for accessibility, performance, and visual stability. Prefer modern formats with responsive delivery.

## Formats

- **SHOULD** serve AVIF where supported; **SHOULD** provide WebP fallback; **MAY** fall back to JPEG/PNG.
- Avoid unoptimized BMP/TIFF/GIF for photos; use GIF/APNG only for short animations when video is not suitable.

## Responsive images

Use `srcset` with width descriptors and `sizes` to match layout.

```html
<img
  srcset="/img/hero-800.avif 800w, /img/hero-1200.avif 1200w"
  sizes="(max-width: 600px) 100vw, 50vw"
  src="/img/hero-800.jpg"
  alt="Team collaborating at a whiteboard"
  loading="lazy"
  decoding="async"
  fetchpriority="high"
/>
```

## Art direction and format switching

```html
<picture>
  <source srcset="/img/card.avif" type="image/avif" />
  <source srcset="/img/card.webp" type="image/webp" />
  <img src="/img/card.jpg" alt="Product card" loading="lazy" decoding="async" />
</picture>
```

Use `<picture>` when:
- You need different crops/layouts per breakpoint.
- You need format fallbacks with type hints.

## Alt text

- **MUST** provide meaningful `alt` for informative images.
- **SHOULD** use `alt=""` for decorative images to avoid noise for screen readers.
- Do not duplicate surrounding caption text verbatim; keep alt concise and specific.

## Preventing layout shift

- Provide intrinsic dimensions or `width`/`height` to establish aspect ratio.
- For CSS background images, reserve space via aspect-ratio boxes.

## Loading strategy

- `loading="lazy"` for below-the-fold images.
- `fetchpriority="high"` for hero images critical to LCP.
- `decoding="async"` to avoid blocking rendering.

## Video and audio

- Prefer `<video>`/`<audio>` with controls; avoid autoplay with sound.
- Include captions (`<track kind="captions">`) and transcripts.
- Provide poster images for video to avoid blank frames; set dimensions to prevent CLS.

## SVG

- Inline SVG for icons when you need CSS theming or accessibility control.
- Provide `role="img"` and `aria-label` or `<title>`/`<desc>` when SVG is meaningful.
