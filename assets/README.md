# PureO — Brand assets

The mark: a thick ring (the object's interface — the only visible surface)
enclosing a dot (its encapsulated state). Pure objects, drawn.

## Colors
- Brand blue (light backgrounds): #2563EB
- Brand blue (dark backgrounds):  #60A5FA
- Wordmark ink (light): #0F172A · (dark): #E2E8F0
- Wordmark typeface: Inter Medium (outlined to paths — no font dependency)

## Files
svg/
  icon.svg              adaptive (auto light/dark via prefers-color-scheme) — use in README
  icon-{light,dark}.svg fixed-color variants
  icon-mono-{black,white}.svg  single-color (print, stamps, badges)
  logo.svg              horizontal lockup, adaptive — use in README / website header
  logo-{light,dark}.svg fixed-color lockups
  logo-mono-{black,white}.svg
png/
  icon-{16..512}.png    raster exports (light variant), icon-512-dark.png
  logo-{light,dark}-1024.png
  github-avatar-512.png     upload as the pure-objects org avatar
  social-preview-1280x640.png  upload in repo Settings → Social preview
favicon.ico             16/32/48 multi-size, for pureo.org

## Usage
- README header:  <img src="assets/svg/logo.svg" height="56" alt="PureO">
- Favicon (site): <link rel="icon" href="/favicon.ico">
                  <link rel="icon" type="image/svg+xml" href="/icon.svg">
- Keep clear space around the mark of at least the dot's diameter.
- Don't stretch, recolor outside the palette, or add effects.
