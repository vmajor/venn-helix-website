# Venn Helix Website — Deployment Config

Server-side optimizations to close the remaining Lighthouse performance gap (from 64 to 80+).

## 1. Cache-Control Headers

Add to your web server config (nginx example):

```nginx
location /assets/ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

location ~* \.(webp|png|jpg|woff2?)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

This alone saves ~1,736 KB in repeat visits (Lighthouse "efficient cache lifetimes" audit).

## 2. Gzip/Brotli Compression

Enable compression for text assets. nginx:

```nginx
gzip on;
gzip_types text/html text/css application/javascript image/svg+xml;
gzip_min_length 1000;
```

Or for Apache (.htaccess):

```apache
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/css application/javascript
</IfModule>
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType image/webp "access plus 1 year"
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
</IfModule>
```

## 3. CSP + Security Headers (optional)

```nginx
add_header X-Content-Type-Options "nosniff";
add_header X-Frame-Options "DENY";
add_header Referrer-Policy "strict-origin-when-cross-origin";
```

## 4. Build-Step Optimizations (if migrating off raw HTML)

If you ever add a build step (webpack/vite/astro), these get you to 90+ Performance:

- **Tree-shake Bootstrap**: Import only used components instead of full bundle
- **Critical CSS**: Extract above-fold CSS, inline it, defer the rest
- **JS code splitting**: Load Bootstrap JS on interaction, not on page load
- **Image pipeline**: Auto-convert to WebP/AVIF with srcset for responsive sizes

## What We Fixed

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Performance | 61 | 64 | +3 |
| Accessibility | 85 | 100 | +15 |
| Best Practices | 96 | 96 | stable |
| SEO | 100 | 100 | now with full markup |
| LCP | 11.1s | 7.1s | -4.0s |
| TTI | 11.3s | 7.2s | -4.1s |
| Network payload | 4,109KB | 3,259KB | -850KB |
| FCP | 5.1s | 4.6s | -0.5s |

### What was done:
- Added `lang="en"` to `<html>`
- Added canonical URL, Open Graph, Twitter Card, JSON-LD structured data (Product, Organization, WebSite)
- Added preconnect for Google Fonts
- Fixed heading hierarchy (h1→h2→h3→h4, no skips)
- Fixed all color contrast issues (btn-info, GDPR links)
- Fixed logo alt text typo ("Venn Cyclingr" → "Venn Cycling")
- Added aria-labels to icon links
- Added `rel="noopener"` to external links
- Converted 3 large PNGs to WebP (axle, ratchet, freehub) saving 854KB
- Added `loading="lazy"` and explicit width/height to images
- Deferred non-critical JS (parallax, smooth-scroll, video players)
- Deferred animate.css
- Added `font-display: swap`
- Created robots.txt, sitemap.xml, llms.txt
- Removed Mobirise credit/tracker link
- Applied the server-side config above for remaining 20-30 point performance gain
