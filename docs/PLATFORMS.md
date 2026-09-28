# Platform chrome measurements

Where the unsafe rects in `layout.platforms` come from, and how much to
trust them.

## Honesty note

These are **community-measured approximations**, not platform
specifications. TikTok, Instagram, and YouTube don't publish safe-zone
specs for organic video; the numbers below come from creator guides,
template makers, and frame-by-frame measurement of the apps' chrome on
a 1080×1920 reference device. Platforms redesign their chrome
regularly. Treat every unsafe rect as guidance with a margin, not
ground truth — the region templates already include margin (content
stops short of the measured chrome edges).

If a platform redesigns, the fix is data-only: update the `unsafe`
rects in `platforms.py`, bump the minor version, note it in the
changelog.

## TikTok (1080×1920)

| zone | measured rect (px, top-origin) | fraction |
|---|---|---|
| status bar / "Following · For You" tabs | y 0–150 | top 0.078 |
| right action rail (profile, like, comment, share, save) | x 925–1080, y 800–1440 | x 0.856–1.0, y 0.417–0.750 |
| caption / song marquee / progress bar | y 1560–1920 | bottom 0.1875 |

Sources: creator safe-zone templates (widely republished; e.g. the
"1080×1920 TikTok safe zone" guides used by CapCut/Premiere template
makers). The rail drifts vertically with video height; the template
uses the common midpoint band.

## Instagram Reels (1080×1920)

| zone | measured rect (px, top-origin) | fraction |
|---|---|---|
| status bar + Reels header (camera icon) | y 0–160 | top 0.083 |
| right action rail | x 945–1080, y 850–1420 | x 0.875–1.0, y 0.443–0.740 |
| caption / audio / progress | y 1500–1920 | bottom 0.219 |

Reels has the deepest bottom chrome of the three 9:16 platforms; the
shared 9:16 region set clears it.

## YouTube Shorts (1080×1920)

| zone | measured rect (px, top-origin) | fraction |
|---|---|---|
| status bar | y 0–120 | top 0.0625 |
| right action rail | x 935–1080, y 820–1400 | x 0.866–1.0, y 0.427–0.729 |
| title / channel / subscribe / progress | y 1540–1920 | bottom 0.198 |

## X portrait (1080×1350, 4:5)

4:5 is the tallest aspect X shows un-cropped in feed. Chrome is light:
top nav (~110px) and the reply/action bar (~120px). No side rail.

## Square 1:1 and widescreen 16:9

No platform chrome inside the canvas on the feed surfaces these
target; content may bleed to the edges. (Stories/reels *viewers* add
chrome, but these aspects are for feed and YouTube main, not stories.)

## The union rule for 9:16

`tiktok`, `instagram-reel`, and `youtube-shorts` share one region
template, validated against the **union** of the three chromes (top
0.083, bottom 0.219, rail x ≥ 0.856). Per-platform `unsafe` rects are
still exposed for custom layouts and for the UI to draw guides.
