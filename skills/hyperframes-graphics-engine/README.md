# HyperFrames Graphics Engine

> **Version 2.0** — Roberto Nickson স্টাইলে word-level synchronized ক্যাপশন, beat-matched মোশন গ্রাফিক্স, সাবজেক্ট কাটআউট এবং প্রিমিয়াম টাইপোগ্রাফি — সম্পূর্ণ HTML/CSS/JS দিয়ে।

---

## স্কিলের পরিচয়

**HyperFrames Graphics Engine** একটি কোড-ফার্স্ট ভিডিও গ্রাফিক্স সিস্টেম। এটি traditional video editing software (Premiere Pro, DaVinci Resolve, CapCut) ছাড়াই প্রিমিয়াম সোশ্যাল মিডিয়া ভিডিও তৈরি করে। সমস্ত ভিজুয়াল — ক্যাপশন থেকে টাইটেল কার্ড, wipe transition থেকে kinetic text — HTML/CSS/JS template হিসেবে লেখা হয় এবং HyperFrames renderer + ffmpeg দিয়ে final MP4-এ convert করা হয়।

### কেন এই স্কিল তৈরি হয়েছে

Roberto Nickson-এর ভিডিও স্টাইলের মূল বৈশিষ্ট্য হলো:
1. **Audio-visual synchrony** — প্রতিটি গ্রাফিক একটি spoken word বা beat-এর সাথে exact sync করা।
2. **Word-by-word caption** — পুরো sentence একসাথে নয়, প্রতিটি শব্দ আলাদাভাবে appear করে।
3. **Extreme restraint** — একটি scene-এ একটিই effect। কখনো দুটো নয়।
4. **Subject-foreground layering** — বক্তা সবসময় গ্রাফিকের সামনে থাকে, গ্রাফিক পেছনে।
5. **Platform-aware safe zones** — Instagram/TikTok UI কখনো কোনো গ্রাফিক cover করে না।

এই স্কিল সেই পুরো workflow-কে Claude Code-এর একটি prompt দিয়ে চালানোর সুযোগ দেয়।

### Architecture Overview

```
Input:
  ├── footage/            ← Raw video clips (.mp4, .mov)
  ├── audio/              ← Audio file (.wav, .mp3)
  ├── subtitles/          ← SRT file with word timestamps
  ├── assets/
  │   ├── images/         ← Logos, screenshots, banners
  │   └── fonts/          ← Custom .woff2 / .ttf fonts
  └── sync-map.json       ← Beat → timestamp mapping

Processing:
  ├── HyperFrames renderer  ← HTML/CSS/JS → PNG frames
  ├── rembg / cutout        ← Subject background removal
  └── ffmpeg pipeline       ← Composite + encode

Output:
  └── final_loop.mp4        ← 1080×1920, loop-ready, platform-safe
```

---

## Graphics Revision Prompts

নিচে ৬টি ready-to-use revision prompt দেওয়া হলো। প্রতিটি prompt ব্যবহার করার সময় `[brackets]`-এ দেওয়া তথ্যগুলো নিজের content অনুযায়ী পূরণ করো।

---

### 1. Rebrand Prompt — Brand Color দিয়ে পুরো গ্রাফিক্স rebrand করো

**কখন ব্যবহার করবে:** যখন default Claude orange (`#D97757`) বাদ দিয়ে নিজের brand color দিয়ে পুরো গ্রাফিক সিস্টেম rebrand করতে চাও।

```
Rebrand the entire HyperFrames graphics output for my brand.

Replace ALL instances of the accent color #D97757 (Claude orange) with:
  Primary brand color:     [e.g., #1DB954 — Spotify green]
  Secondary brand color:   [e.g., #191414 — Spotify black]
  Brand font (if any):     [e.g., Circular Std Bold — file is at assets/fonts/CircularStd-Bold.woff2]

Rules:
- Active caption keywords → primary brand color
- All card borders, badges, highlights → primary brand color
- All dark card backgrounds → secondary brand color (if provided)
- Opening title brand highlight → primary brand color
- Terminal card accent → primary brand color
- Do NOT change white (#FFFFFF) or muted gray (#8A8A8A)
- Do NOT change any timing, easing, or animation logic
- Update all CSS :root variables so the change propagates everywhere
- Regenerate safe-zone check image after rebrand

Output: Updated HTML templates + updated sync-map CSS variables.
```

---

### 2. Motion on Every Line Prompt — প্রতিটি ক্যাপশন লাইনে motion যোগ করো

**কখন ব্যবহার করবে:** যখন caption-গুলো static মনে হচ্ছে এবং প্রতিটি নতুন লাইনে ছোট directional motion যোগ করতে চাও।

```
Add a subtle entry motion to every caption line in the video.

Current behavior: Words fade in with opacity 0→1, translateY 6px→0.
New behavior:
  - Odd lines (1st, 3rd, 5th...): slide in from LEFT — translateX(-20px)→(0)
  - Even lines (2nd, 4th, 6th...): slide in from RIGHT — translateX(20px)→(0)
  - Combined with existing opacity fade (keep it)
  - Duration: same 80ms per word
  - Easing: cubic-bezier(0.16, 1, 0.3, 1) — expo-out, no bounce
  - Safe zone: ensure no word starts its animation outside the safe boundary
  - Do NOT add motion to active keyword highlight — color change only

Apply this to ALL caption words in ALL beats.
Do NOT change font size, color, position, or shadow.
```

---

### 3. Better Hook Prompt — প্রথম ৩ সেকেন্ডের hook improve করো

**কখন ব্যবহার করবে:** যখন প্রথম few seconds দুর্বল মনে হচ্ছে এবং viewer retention বাড়াতে চাও।

```
Improve the opening hook for this video. The first 3 seconds must stop the scroll.

Current opening: [describe current opening — e.g., "Plain title card with fade-in text"]

New hook requirements:
  1. Frame 0–0.5s: Black frame → instant cut to subject (no fade, hard cut)
  2. Frame 0.5s: First caption word appears in [KEYWORD] — color: #D97757 — size: 80px
  3. Frame 0.5–1.5s: Three rapid caption words, 80ms apart, each translateY(-8px)→(0)
  4. Frame 1.5s: Quick zoom push — scale(1.0)→scale(1.06) over 0.4s
  5. Frame 1.5–3s: [Opening title text here] builds word by word

The hook must:
  - Have NO music fade-in (audio starts at full volume, frame 0)
  - Use a maximum of 2 visual elements at any time
  - Keep all text within safe zones (top: 220px, bottom: 420px, sides: 140px)
  - End with a clear visual "question" that the rest of the video answers

Output: Updated opening sequence HTML + updated sync-map timing for 0–3s.
```

---

### 4. Make a Moment Pop Prompt — একটি নির্দিষ্ট beat-কে আরো impactful করো

**কখন ব্যবহার করবে:** যখন video-র মধ্যে একটি বিশেষ moment আছে যেটা এখনো যতটা হওয়া উচিত ততটা impactful না।

```
Make the moment at [timestamp, e.g., "0:00:14" / "the 'what it gave me back' beat"] pop harder.

Current treatment: [describe current — e.g., "Simple wipe transition with no audio cue"]

Upgrade this specific beat only. Do NOT change anything before or after it.

Enhancement options to apply (pick the most appropriate 1–2):
  Option A — Scale punch: Subject briefly scales 1.0→1.04→1.0 (0.3s total, expo-out)
  Option B — Text slam: Keyword text drops from y-20px with a 60ms delay after cut
  Option C — Color flash: Single-frame white flash (1 frame = 40ms) on the cut point
  Option D — Card reveal: Animated dark card slides up from bottom with key stat
  Option E — Audio sync: Ensure the visual hit lands on frame N where audio peak occurs

Rules:
  - Maximum 1 new animation layer added
  - No stacking of Options A+B+C simultaneously — pick 1 or 2 max
  - Safe zone compliance required for all new elements
  - Easing: expo-out only — no bounce, no spring
  - Test: run safe_zone_check after applying

Output: Updated HTML for that specific beat + explanation of which option was chosen and why.
```

---

### 5. Tease the Prompt Prompt — "one prompt" beat-কে আরো dramatic করো

**কখন ব্যবহার করবে:** যখন "one prompt" বা "AI did this" moment-টি terminal card দিয়ে দেখানো হচ্ছে কিন্তু আরো dramatic হওয়া দরকার।

```
Make the "one prompt" terminal card sequence more dramatically reveal the AI prompt.

Current behavior: Terminal card appears, first line types in at 30ms/char.

New behavior — staged reveal:
  Stage 1 (0–0.5s): Terminal card slides up from bottom — translateY(60px)→(0), opacity 0→1
  Stage 2 (0.5–0.8s): Cursor blinks 3 times (200ms on/off) — builds anticipation
  Stage 3 (0.8–2.5s): Prompt text types in at [25ms/char] — one character at a time
  Stage 4 (2.5–3.0s): Prompt text fully visible → brief highlight sweep (left to right, orange tint)
  Stage 5 (3.0–3.5s): Counter animates: [raw_word_count] → [final_word_count], 800ms, expo-out

The prompt text to type in:
  "[Paste the actual AI prompt here — e.g., 'Edit this video. Make every cut hit on a beat. Use only Claude orange and white. Keep it under 60 seconds.']"

Additional rules:
  - Font: monospace (Courier New or system mono) for the prompt text
  - Terminal card background: #141413, border: 1px solid rgba(255,255,255,0.08)
  - Cursor: white blinking block character ▋
  - No SFX on typing — silence is more dramatic here
  - Safe zone: card must stay within x:140–940, y:220–1500

Output: Updated terminal card HTML template + updated timing in sync-map.json.
```

---

### 6. Screen Glance Cover Prompt — ফোনের স্ক্রিন glance-এ যা দেখা যাবে তা cover করো

**কখন ব্যবহার করবে:** যখন video thumbnail বা first frame-এ subject-এর ফোন স্ক্রিন, sensitive info বা placeholder content দেখা যাচ্ছে যা cover করতে হবে।

```
Cover the phone screen / laptop screen / monitor visible in the footage at [timestamp range, e.g., "0:00:08–0:00:12"].

The screen area in frame is approximately at:
  x: [e.g., 300px], y: [e.g., 600px], width: [e.g., 380px], height: [e.g., 240px]
  (Use 1080×1920 coordinate system)

Cover method — choose the most premium option:
  Option A — Branded mockup: Replace the screen with a realistic device mockup showing
             the product/tool screenshot from assets/images/[filename].png
             (animate it with a subtle tilt and shadow — like a product ad)

  Option B — Blur + badge: Apply a gaussian blur (radius: 20px) over the screen area
             and place a branded badge overlay: "[Brand name] ✦" in white on dark card

  Option C — Content replace: Replace screen content with a relevant stat card or
             terminal showing "[relevant text that fits the narrative]"

Rules:
  - The cover must look intentional, not like a redaction
  - If Option A: use assets/images/ — check for a screenshot/product image first
  - The covered area must still respect safe zones
  - Composite via ffmpeg overlay filter, frame-accurate to [timestamp]
  - No harsh edges — feather/blend the overlay by 4–8px

Output: ffmpeg composite command + HTML overlay template for the screen cover.
```

---

## `assets/images/` ফোল্ডার Naming Convention

ফাইল নাম থেকে HyperFrames automatically সঠিক beat-এ image ব্যবহার করে। নিচের convention follow করো:

### Standard Naming Patterns

```
assets/images/
├── logo.png                  → Opening title, brand card, all brand beats
├── logo.svg                  → Same as logo.png (SVG preferred for scaling)
│
├── before-[description].png  → "posted last year" beat — BEFORE card
├── after-[description].png   → "Claude's re-edit" beat — AFTER card
├── thumbnail-[description].jpg → Video thumbnail বা preview card
│
├── tool-screenshot.png       → "single editing app" বা tool demo beat
├── screenshot-[name].png     → যেকোনো screen recording demo beat
├── product-[name].png        → Product showcase beat
│
├── banner-[description].png  → Background overlay / context beat
├── map-[location].png        → Location বা context graphic beat
│
├── icon-[name].png           → Badge বা label-এ inline icon (24–48px)
├── icon-[name].svg           → Same, SVG preferred
│
└── bg-[description].jpg      → Full-bleed background texture বা gradient
```

### নামকরণের নিয়ম

1. **lowercase + hyphen** — কোনো space বা underscore নয়: `tool-screenshot.png` ✅, `Tool Screenshot.png` ❌
2. **descriptor prefix** — category prefix সবসময় দাও: `logo-`, `before-`, `after-`, `tool-`, `icon-`
3. **meaningful suffix** — generic নাম দিও না: `product-claude-interface.png` ✅, `image1.png` ❌
4. **PNG for transparency** — transparent background দরকার হলে সবসময় `.png` ব্যবহার করো
5. **SVG for logos/icons** — যেকোনো vector asset-এর জন্য `.svg` prefer করো

### Beat-to-Image Mapping Logic

```javascript
// HyperFrames auto-mapper
const beatImageMap = {
  "opening-title":    ["logo", "banner"],
  "posted-last-year": ["before-", "thumbnail-"],
  "claude-re-edit":   ["after-", "thumbnail-"],
  "tool-demo":        ["tool-screenshot", "screenshot-", "product-"],
  "one-prompt":       ["screenshot-", "terminal"],
  "raw-footage":      ["before-", "thumbnail-"],
  "context":          ["banner-", "map-"],
};

function findImageForBeat(beatName, imageFiles) {
  const patterns = beatImageMap[beatName] || [];
  return imageFiles.find(file =>
    patterns.some(pattern => file.toLowerCase().includes(pattern))
  ) || null;
}
```

---

## Subject Cutout — কখন এবং কোথায় ব্যবহার করতে হবে

### সংক্ষিপ্ত নীতি

> **Subject cutout = selective luxury, not default.**  
> প্রতিটি beat-এ cutout ব্যবহার করলে rendering time বাড়ে এবং ভুল জায়গায় দেখতে weird লাগে। শুধু নির্দিষ্ট beats-এ ব্যবহার করো।

### কখন ব্যবহার করবে ✅

| Beat | কারণ |
|------|------|
| Opening title | Headline টেক্সট বক্তার পেছনে layer করতে — premium intro |
| "motion graphics" | "MOTION GRAPHICS" বড় টেক্সট subject-এর পেছনে যাবে |
| "one prompt" | Terminal card subject-এর পেছনে — depth effect |
| যেকোনো beat যেখানে 72px+ font আছে | বড় টেক্সট subject overlap করলে |

### কখন ব্যবহার করবে না ❌

| Situation | কারণ |
|-----------|------|
| Fast-moving action shots | rembg ভুল mask তৈরি করে |
| Hair/fur/complex edge footage | Edge artifact দেখা যাবে — premium লুক নষ্ট হবে |
| Caption-only beats | Caption সামনে থাকে, পেছনে graphic নেই — cutout দরকার নেই |
| Transition beats (wipe, cut) | Transition-এ cutout flicker করে |
| Every single beat | Overkill — Roberto Nickson style হারায় |

### Cutout Quality Checklist

```
কাজ শেষে প্রতিটি cutout frame-এ check করো:
  [ ] মাথার চারপাশে edge smooth আছে কিনা (হালোহীন)
  [ ] হাতের আঙুলের মাঝে background সঠিকভাবে transparent
  [ ] Shadow এবং clothing edge realistic
  [ ] কোনো green fringe বা color bleed নেই
  [ ] Composite-এ subject layer Z=5 (সবকিছুর উপরে)
```

### Cutout → Composite Workflow

```bash
# Step 1: Subject cutout করো
rembg i footage/speaker_clip.png assets/cutout/speaker.png

# Step 2: Video frames-এ apply করো (batch)
mkdir -p frames/raw frames/cutout
ffmpeg -i footage/speaker.mp4 -vf fps=25 frames/raw/frame_%04d.png
for f in frames/raw/*.png; do
  rembg i "$f" "frames/cutout/${f##*/}"
done

# Step 3: Cutout frames → alpha video
ffmpeg -r 25 \
  -i frames/cutout/frame_%04d.png \
  -vf "format=rgba" \
  -c:v prores_ks -profile:v 4444 \
  assets/cutout/speaker_alpha.mov

# Step 4: Composite — background graphic + speaker cutout
ffmpeg \
  -i background_graphic.mp4 \
  -i assets/cutout/speaker_alpha.mov \
  -filter_complex "[0:v][1:v]overlay=x=0:y=0:format=auto" \
  -c:v libx264 -crf 16 \
  composite_output.mp4
```

---

## HyperFrames HTML Template উদাহরণ

নিচে একটি সম্পূর্ণ, production-ready HyperFrames HTML template দেওয়া হলো যা সমস্ত system-কে একসাথে একত্রিত করে।

```html
<!DOCTYPE html>
<html lang="bn">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=1080" />
  <title>HyperFrames — Graphics Engine v2.0</title>

  <!-- ═══════════════════════════════════════════
       FONT LOADING — assets/fonts/ প্রথমে
  ═══════════════════════════════════════════ -->
  <style>
    @font-face {
      font-family: "CustomFont";
      src:
        url("assets/fonts/CustomFont-Bold.woff2") format("woff2"),
        url("assets/fonts/CustomFont-Bold.ttf")   format("truetype");
      font-weight: 700 800;
      font-display: block;
    }
    @font-face {
      font-family: "NotoSansBengali";
      src: url("assets/fonts/NotoSansBengali-Bold.woff2") format("woff2");
      font-weight: 700;
      unicode-range: U+0980-U+09FF;
      font-display: block;
    }
  </style>

  <style>
    /* ═══════════════════════════════════════
       ROOT VARIABLES — Design System
    ═══════════════════════════════════════ */
    :root {
      /* Canvas */
      --canvas-w: 1080px;
      --canvas-h: 1920px;

      /* Safe Zones */
      --safe-top:    220px;
      --safe-bottom: 420px;
      --safe-left:   140px;
      --safe-right:  140px;
      --safe-w: calc(var(--canvas-w) - var(--safe-left) - var(--safe-right));
      --safe-h: calc(var(--canvas-h) - var(--safe-top)  - var(--safe-bottom));

      /* Color Palette */
      --color-dark:    #141413;
      --color-white:   #FFFFFF;
      --color-accent:  #D97757;
      --color-muted:   #8A8A8A;
      --color-overlay: rgba(20, 20, 19, 0.85);

      /* Typography */
      --font-primary: "CustomFont", "NotoSansBengali", "Noto Sans Bengali",
                      "SF Pro Display", "Segoe UI", sans-serif;
      --font-mono:    "Courier New", "JetBrains Mono", monospace;
      --font-caption: 68px;
      --font-title:   96px;
      --font-badge:   24px;

      /* Animation */
      --ease-expo-out: cubic-bezier(0.16, 1, 0.3, 1);
      --dur-word:      80ms;
      --dur-card:      500ms;
      --dur-wipe:      1200ms;

      /* Cards */
      --radius-card:   16px;
      --radius-badge:  8px;
    }

    /* ═══════════════════════════════════════
       BASE RESET & CANVAS
    ═══════════════════════════════════════ */
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      width:  var(--canvas-w);
      height: var(--canvas-h);
      background: #000000;
      overflow: hidden;
      font-family: var(--font-primary);
    }

    .frame {
      position: relative;
      width:  var(--canvas-w);
      height: var(--canvas-h);
      overflow: hidden;
    }

    /* ═══════════════════════════════════════
       LAYER SYSTEM
    ═══════════════════════════════════════ */
    .layer { position: absolute; inset: 0; }
    .layer-1-bg      { z-index: 1; }
    .layer-2-broll   { z-index: 2; }
    .layer-3-graphic { z-index: 3; }
    .layer-4-caption { z-index: 4; }
    .layer-5-subject { z-index: 5; }

    /* ═══════════════════════════════════════
       SAFE ZONE CONTAINER
    ═══════════════════════════════════════ */
    .safe-zone {
      position: absolute;
      left:   var(--safe-left);
      top:    var(--safe-top);
      width:  var(--safe-w);
      height: var(--safe-h);
      /* border: 1px dashed rgba(255,0,0,0.3); ← debug mode only */
    }

    /* ═══════════════════════════════════════
       CAPTION SYSTEM
    ═══════════════════════════════════════ */
    .caption-container {
      position: absolute;
      bottom: calc(var(--safe-bottom) + 80px);
      left:   var(--safe-left);
      width:  var(--safe-w);
      text-align: center;
      z-index: 4;
    }

    .caption-line {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      align-items: baseline;
      gap: 0 10px;
      min-height: 84px;
    }

    @keyframes wordFadeIn {
      from { opacity: 0; transform: translateY(8px); }
      to   { opacity: 1; transform: translateY(0);   }
    }

    .caption-word {
      display: inline-block;
      opacity: 0;
      font-family: var(--font-primary);
      font-size: var(--font-caption);
      font-weight: 800;
      letter-spacing: -1.5px;
      line-height: 1.1;
      color: var(--color-white);
      text-shadow: 0 1px 2px rgba(0, 0, 0, 0.6);
      animation: wordFadeIn var(--dur-word) var(--ease-expo-out) forwards;
      text-transform: uppercase;
    }

    .caption-word.keyword {
      color: var(--color-accent);
    }

    /* Bengali script — disable uppercase */
    .caption-word[lang="bn"] {
      text-transform: none;
      font-family: "NotoSansBengali", "Noto Sans Bengali", "Hind Siliguri", sans-serif;
      letter-spacing: 0;
    }

    /* Stagger delays via JS — example inline style:
       style="animation-delay: 0ms"
       style="animation-delay: 80ms"
       style="animation-delay: 160ms" */

    /* ═══════════════════════════════════════
       OPENING TITLE
    ═══════════════════════════════════════ */
    .opening-title {
      position: absolute;
      top: 50%;
      left: var(--safe-left);
      width: var(--safe-w);
      transform: translateY(-50%);
      text-align: center;
      z-index: 3;
    }

    .opening-title .line {
      display: block;
      font-size: var(--font-title);
      font-weight: 800;
      color: var(--color-white);
      letter-spacing: -2px;
      text-transform: uppercase;
    }

    .opening-title .highlight {
      color: var(--color-accent);
      text-shadow: 0 0 24px rgba(217, 119, 87, 0.35);
    }

    /* ═══════════════════════════════════════
       STAT CARD (Beat 8: "editing apps")
    ═══════════════════════════════════════ */
    @keyframes cardReveal {
      from { opacity: 0; transform: translateY(24px) scale(0.97); }
      to   { opacity: 1; transform: translateY(0)    scale(1.00); }
    }

    .stat-card {
      background: var(--color-dark);
      border: 1px solid rgba(255, 255, 255, 0.08);
      border-radius: var(--radius-card);
      padding: 28px 36px;
      display: inline-block;
      animation: cardReveal var(--dur-card) var(--ease-expo-out) forwards;
    }

    .stat-label {
      font-size: 28px;
      font-weight: 500;
      color: var(--color-muted);
      text-transform: uppercase;
      letter-spacing: 1px;
      margin-bottom: 8px;
    }

    .stat-value {
      font-size: 100px;
      font-weight: 800;
      color: var(--color-accent);
      line-height: 1;
    }

    /* ═══════════════════════════════════════
       TERMINAL CARD (Beat 10: "one prompt")
    ═══════════════════════════════════════ */
    .terminal-card {
      background: var(--color-dark);
      border: 1px solid rgba(255, 255, 255, 0.1);
      border-radius: var(--radius-card);
      overflow: hidden;
      animation: cardReveal var(--dur-card) var(--ease-expo-out) forwards;
      width: 100%;
    }

    .terminal-header {
      background: rgba(255, 255, 255, 0.05);
      padding: 12px 16px;
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .terminal-dot {
      width: 12px; height: 12px;
      border-radius: 50%;
    }
    .terminal-dot.red    { background: #ff5f57; }
    .terminal-dot.yellow { background: #febc2e; }
    .terminal-dot.green  { background: #28c840; }

    .terminal-title-bar {
      flex: 1;
      text-align: center;
      font-size: 22px;
      color: var(--color-muted);
      font-family: var(--font-mono);
    }

    .terminal-body {
      padding: 20px 24px;
      font-family: var(--font-mono);
      font-size: 30px;
      color: var(--color-white);
      min-height: 120px;
    }

    .terminal-cursor {
      display: inline-block;
      color: var(--color-accent);
      animation: cursorBlink 500ms step-end infinite;
    }
    @keyframes cursorBlink {
      0%, 100% { opacity: 1; }
      50%       { opacity: 0; }
    }

    .terminal-counter {
      padding: 16px 24px;
      border-top: 1px solid rgba(255,255,255,0.06);
      display: flex;
      align-items: center;
      gap: 12px;
      font-size: 28px;
      color: var(--color-muted);
    }
    .terminal-counter .value {
      font-size: 36px;
      font-weight: 800;
      color: var(--color-white);
    }
    .terminal-counter .arrow { color: var(--color-accent); }
    .terminal-counter .final { color: var(--color-accent); }

    /* ═══════════════════════════════════════
       RAW BADGE (Beat 11: "This is what...")
    ═══════════════════════════════════════ */
    .badge {
      display: inline-flex;
      align-items: center;
      font-size: var(--font-badge);
      font-weight: 700;
      letter-spacing: 2.5px;
      padding: 6px 16px;
      border-radius: var(--radius-badge);
      text-transform: uppercase;
    }
    .badge-raw    { background: #c0392b; color: #ffffff; }
    .badge-before { background: var(--color-muted); color: #ffffff; }
    .badge-claude { background: var(--color-accent); color: #ffffff; }

    .timecode {
      font-family: var(--font-mono);
      font-size: 26px;
      color: var(--color-white);
      background: rgba(0, 0, 0, 0.72);
      padding: 5px 12px;
      border-radius: 4px;
    }

    /* ═══════════════════════════════════════
       WAVEFORM (Beat 7: "sound effects")
    ═══════════════════════════════════════ */
    .waveform {
      display: flex;
      align-items: center;
      gap: 6px;
      height: 60px;
    }

    @keyframes waveformPulse {
      from { height: 8px;  opacity: 0.35; }
      to   { height: 48px; opacity: 1.00; }
    }

    .waveform-bar {
      width: 5px;
      border-radius: 3px;
      background: var(--color-accent);
      animation: waveformPulse 480ms var(--ease-expo-out) infinite alternate;
    }
    /* stagger delays set via inline style */

    /* ═══════════════════════════════════════
       TIMELINE STRIP (Beat 9: "raw footage")
    ═══════════════════════════════════════ */
    .timeline-strip {
      display: flex;
      height: 22px;
      border-radius: 11px;
      overflow: hidden;
      width: 100%;
      gap: 2px;
    }
    .ts-bad   { background: #c0392b; }
    .ts-pause { background: #555555; }
    .ts-good  { background: rgba(255,255,255,0.85); }

    @keyframes tsReveal {
      from { transform: scaleX(0); }
      to   { transform: scaleX(1); }
    }
    .timeline-strip .ts-seg {
      transform-origin: left;
      animation: tsReveal 0.6s var(--ease-expo-out) forwards;
      border-radius: 4px;
    }

    /* ═══════════════════════════════════════
       COLOUR GRADE WIPE (Beat 3)
    ═══════════════════════════════════════ */
    @keyframes wipeReveal {
      from { clip-path: inset(0 100% 0 0); }
      to   { clip-path: inset(0 0%   0 0); }
    }
    .graded-layer {
      position: absolute; inset: 0;
      animation: wipeReveal var(--dur-wipe) var(--ease-expo-out) forwards;
    }
    @keyframes wipeLineSweep {
      from { left: 0%; }
      to   { left: 100%; }
    }
    .wipe-line {
      position: absolute; top: 0; bottom: 0;
      width: 3px; background: #ffffff;
      box-shadow: 0 0 14px rgba(255,255,255,0.9);
      animation: wipeLineSweep var(--dur-wipe) var(--ease-expo-out) forwards;
    }

    /* ═══════════════════════════════════════
       MOTION GRAPHICS BUILD (Beat 6)
    ═══════════════════════════════════════ */
    @keyframes barExpand {
      from { transform: scaleX(0); }
      to   { transform: scaleX(1); }
    }
    .mg-bar {
      height: 6px;
      background: var(--color-accent);
      border-radius: 3px;
      transform-origin: left;
      animation: barExpand 0.4s var(--ease-expo-out) forwards;
    }
    @keyframes mgTextRise {
      from { clip-path: inset(100% 0 0 0); transform: translateY(16px); }
      to   { clip-path: inset(0%   0 0 0); transform: translateY(0);    }
    }
    .mg-text {
      font-size: 76px;
      font-weight: 800;
      color: var(--color-white);
      text-transform: uppercase;
      letter-spacing: -2px;
      clip-path: inset(100% 0 0 0);
      animation: mgTextRise 0.4s 0.38s var(--ease-expo-out) forwards;
    }

    /* ═══════════════════════════════════════
       CHAT BUBBLE (Beat 13: "bigger captions")
    ═══════════════════════════════════════ */
    @keyframes bubblePop {
      from { transform: scale(0.75); opacity: 0; }
      to   { transform: scale(1.00); opacity: 1; }
    }
    .chat-bubble {
      background: var(--color-accent);
      color: var(--color-white);
      border-radius: 16px 16px 4px 16px;
      padding: 16px 24px;
      font-size: 34px;
      font-weight: 600;
      display: inline-block;
      animation: bubblePop 300ms var(--ease-expo-out) forwards;
    }

    /* ═══════════════════════════════════════
       BEFORE / AFTER CARD (Beats 14 & 15)
    ═══════════════════════════════════════ */
    .media-card {
      border-radius: var(--radius-card);
      overflow: hidden;
      border: 2px solid rgba(255, 255, 255, 0.1);
      position: relative;
      animation: cardReveal var(--dur-card) var(--ease-expo-out) forwards;
    }
    .media-card img, .media-card video {
      width: 100%;
      display: block;
    }
    .media-card .badge {
      position: absolute;
      top: 14px; left: 14px;
    }

    /* ═══════════════════════════════════════
       KINETIC TEXT (Beat 16: "Comment CLAUDE")
    ═══════════════════════════════════════ */
    @keyframes kineticZoom {
      from { transform: scale(0.4); opacity: 0; }
      to   { transform: scale(1.0); opacity: 1; }
    }
    .kinetic-word {
      font-size: 128px;
      font-weight: 800;
      color: var(--color-accent);
      text-transform: uppercase;
      letter-spacing: -4px;
      line-height: 1;
      animation: kineticZoom 0.5s var(--ease-expo-out) forwards;
    }

    .comment-bubble-ui {
      background: var(--color-overlay);
      border: 1px solid rgba(255,255,255,0.12);
      border-radius: 12px;
      padding: 14px 22px;
      font-size: 30px;
      color: var(--color-white);
      display: flex;
      align-items: center;
      gap: 12px;
      animation: cardReveal 0.4s 0.45s var(--ease-expo-out) forwards;
      opacity: 0;
    }
  </style>
</head>
<body>

<div class="frame" id="frame">

  <!-- ─── LAYER 1: Background ─── -->
  <div class="layer layer-1-bg" id="bg-layer">
    <!-- Base video বা solid color এখানে -->
    <video id="base-video" autoplay muted playsinline
           style="width:100%;height:100%;object-fit:cover;">
      <source src="footage/base.mp4" type="video/mp4">
    </video>
  </div>

  <!-- ─── LAYER 2: B-Roll / Colour Grade ─── -->
  <div class="layer layer-2-broll" id="broll-layer" style="display:none;">
    <!-- Beat-specific B-roll footage -->
    <div class="graded-layer">
      <video autoplay muted style="width:100%;height:100%;object-fit:cover;">
        <source src="footage/broll.mp4" type="video/mp4">
      </video>
    </div>
    <div class="wipe-line" id="wipe-line"></div>
  </div>

  <!-- ─── LAYER 3: Motion Graphics ─── -->
  <div class="layer layer-3-graphic" id="graphic-layer">
    <div class="safe-zone" id="graphic-safe">
      <!-- Dynamic beat graphics inserted here by JS -->
    </div>
  </div>

  <!-- ─── LAYER 4: Captions ─── -->
  <div class="layer layer-4-caption">
    <div class="caption-container" id="caption-container">
      <div class="caption-line" id="caption-line">
        <!-- Words injected dynamically by caption engine -->
      </div>
    </div>
  </div>

  <!-- ─── LAYER 5: Subject Cutout ─── -->
  <div class="layer layer-5-subject" id="subject-layer" style="display:none;">
    <video id="subject-video" autoplay muted playsinline
           style="width:100%;height:100%;object-fit:contain;">
      <source src="assets/cutout/speaker_alpha.webm" type="video/webm">
    </video>
  </div>

</div><!-- /.frame -->

<script>
// ═══════════════════════════════════════════════════════════
//  HyperFrames Caption Engine v2.0
// ═══════════════════════════════════════════════════════════

const ACTIVE_KEYWORDS = [
  "claude", "code", "edited", "captions", "motion", "graphics",
  "single", "one", "prompt", "comment",
];

/**
 * Parse SRT file and return word-level timed entries.
 * @param {string} srtContent - Raw SRT file content
 * @returns {Array<{wordMs: number, text: string, isKeyword: boolean}>}
 */
function parseSRT(srtContent) {
  const cues = [];
  const blocks = srtContent.trim().split(/\n\n+/);

  blocks.forEach(block => {
    const lines = block.split('\n');
    if (lines.length < 3) return;

    const timeLine = lines[1];
    const text     = lines.slice(2).join(' ').replace(/<[^>]+>/g, '').trim();

    const [startStr, endStr] = timeLine.split(' --> ');
    const startMs = srtTimeToMs(startStr.trim());
    const endMs   = srtTimeToMs(endStr.trim());

    // Word-boundary split (supports Bengali via Unicode normalization)
    const words = splitWords(text);
    const wordDuration = (endMs - startMs) / words.length;

    words.forEach((word, i) => {
      cues.push({
        wordMs:    startMs + i * wordDuration,
        text:      word,
        isKeyword: isActiveKeyword(word),
        lang:      detectLang(word),
      });
    });
  });

  return cues.sort((a, b) => a.wordMs - b.wordMs);
}

function srtTimeToMs(timeStr) {
  // HH:MM:SS,mmm → ms
  const [hms, ms] = timeStr.split(',');
  const [h, m, s] = hms.split(':').map(Number);
  return (h * 3600 + m * 60 + s) * 1000 + Number(ms);
}

function splitWords(text) {
  // Unicode NFC normalize
  const normalized = text.normalize('NFC');
  // Split on spaces, zero-width spaces, and zero-width non-joiners
  return normalized.split(/[\u0020\u200B\u200C]+/).filter(Boolean);
}

function isActiveKeyword(word) {
  const clean = word.toLowerCase().replace(/[^a-z\u0980-\u09FF]/g, '');
  return ACTIVE_KEYWORDS.some(kw => clean.includes(kw));
}

function detectLang(word) {
  // Bengali Unicode range U+0980–U+09FF
  return /[\u0980-\u09FF]/.test(word) ? 'bn' : 'en';
}

/**
 * Render word at given DOM position.
 * @param {Object} wordEntry - {text, isKeyword, lang, delay}
 */
function renderWord(wordEntry, delayMs = 0) {
  const container = document.getElementById('caption-line');
  const span = document.createElement('span');
  span.className = 'caption-word' + (wordEntry.isKeyword ? ' keyword' : '');
  span.textContent = wordEntry.text;
  span.lang = wordEntry.lang;
  span.style.animationDelay = `${delayMs}ms`;
  container.appendChild(span);

  // Safe zone check: max 4 words per line
  const words = container.querySelectorAll('.caption-word');
  if (words.length > 4) clearCaptionLine();
}

function clearCaptionLine() {
  document.getElementById('caption-line').innerHTML = '';
}

// ─── SYNC MAP PLAYER ───────────────────────────────────────
// syncMap.json format:
// [{ "beatName": "opening-title", "startMs": 0, "endMs": 2500, "graphicId": "beat-1" }, ...]

async function loadSyncMap(url) {
  const res = await fetch(url);
  return res.json();
}

function playSyncMap(syncMap, videoEl) {
  syncMap.forEach(beat => {
    const startSec = beat.startMs / 1000;
    const endSec   = beat.endMs   / 1000;

    // Check video time and show/hide graphic
    videoEl.addEventListener('timeupdate', () => {
      const t = videoEl.currentTime;
      const el = document.getElementById(beat.graphicId);
      if (!el) return;

      if (t >= startSec && t < endSec) {
        el.style.display = 'block';
      } else {
        el.style.display = 'none';
      }
    });
  });
}

// ─── COUNTER ANIMATION ─────────────────────────────────────
function animateCounter(element, from, to, durationMs = 1000) {
  const start = performance.now();
  const tick  = (now) => {
    const t    = Math.min((now - start) / durationMs, 1);
    const ease = 1 - Math.pow(1 - t, 4); // expo-out
    element.textContent = Math.round(from + (to - from) * ease);
    if (t < 1) requestAnimationFrame(tick);
  };
  requestAnimationFrame(tick);
}

// ─── TYPEWRITER ────────────────────────────────────────────
function typeWriter(element, text, charDelayMs = 28) {
  let i = 0;
  const cursor = document.querySelector('.terminal-cursor');
  return new Promise(resolve => {
    const interval = setInterval(() => {
      element.textContent += text[i++];
      if (i >= text.length) {
        clearInterval(interval);
        if (cursor) cursor.style.display = 'none';
        resolve();
      }
    }, charDelayMs);
  });
}

// ─── INIT ───────────────────────────────────────────────────
(async () => {
  // 1. Load sync map
  const syncMap = await loadSyncMap('sync-map.json');

  // 2. Load SRT and parse words
  const srtRes  = await fetch('subtitles/captions.srt');
  const srtText = await srtRes.text();
  const words   = parseSRT(srtText);

  // 3. Setup caption engine
  const baseVideo = document.getElementById('base-video');
  let lastRenderedWordIdx = -1;

  baseVideo.addEventListener('timeupdate', () => {
    const nowMs = baseVideo.currentTime * 1000;

    words.forEach((wordEntry, idx) => {
      if (idx <= lastRenderedWordIdx) return;
      if (nowMs >= wordEntry.wordMs) {
        renderWord(wordEntry, 0);
        lastRenderedWordIdx = idx;
      }
    });
  });

  // 4. Play sync map
  playSyncMap(syncMap, baseVideo);
})();
</script>

</body>
</html>
```

---

## Bengali Caption Support Guide

HyperFrames Graphics Engine বাংলা ভাষার ক্যাপশনকে সম্পূর্ণভাবে support করে। নিচে step-by-step guide দেওয়া হলো।

### Step 1: SRT ফাইল বাংলায় লেখো

```srt
1
00:00:01,200 --> 00:00:02,800
এটা কীভাবে সম্ভব হলো

2
00:00:02,900 --> 00:00:04,500
শুধু একটা প্রম্পট দিয়ে

3
00:00:04,600 --> 00:00:06,000
Claude Code এডিট করেছে
```

- SRT ফাইল **UTF-8 encoding**-এ save করো (BOM ছাড়া)
- প্রতিটি শব্দের মাঝে **standard space** ব্যবহার করো
- Zero Width Joiner (ZWJ, U+200D) শুধু conjunct consonants-এ রাখো

### Step 2: Bengali Font লোড করো

```bash
# Noto Sans Bengali download করো (ওজন 700)
curl -o assets/fonts/NotoSansBengali-Bold.woff2 \
  "https://fonts.gstatic.com/s/notosansbengali/v20/Cn-SJsCGWQxOjaGwMQ6fIiMywrNJIky6nvd8BjzVMvJx2mcSPVFpVEqE-6KmsolLudCk8izI0lc.woff2"

# অথবা Hind Siliguri (lighter, more modern)
curl -o assets/fonts/HindSiliguri-Bold.woff2 \
  "https://fonts.gstatic.com/s/hindsiliguri/v12/N0bX2SlFPv1weGeLZDtQIfTTkdbJYA.woff2"
```

### Step 3: Unicode Normalization

```javascript
/**
 * Bengali text-কে render করার আগে normalize করো।
 * NFC form — precomposed characters নিশ্চিত করে।
 */
function prepareBengaliText(text) {
  return text
    .normalize('NFC')               // Canonical decomposition + composition
    .replace(/\r\n?/g, '\n')        // Line ending normalize
    .replace(/[\u00A0]/g, '\u0020') // NBSP → regular space
    .trim();
}

/**
 * বাংলা শব্দ boundary detect করো।
 * বাংলায় শব্দের সীমানা মূলত space দিয়ে।
 * Hasanta (্) + consonant = conjunct — এটা split করবে না।
 */
function splitBengaliWords(text) {
  const normalized = prepareBengaliText(text);

  // ZWJ/ZWNJ ছাড়া সব space-এ split
  // \u09CD = Hasanta — এর পরের character conjunct-এর অংশ
  const words = [];
  let current = '';

  for (let i = 0; i < normalized.length; i++) {
    const char = normalized[i];
    const next = normalized[i + 1];

    if (char === '\u0020' || char === '\u200B') {
      // Hasanta-র পরে space এলেও split করো না — edge case
      if (current) {
        words.push(current);
        current = '';
      }
    } else {
      current += char;
    }
  }
  if (current) words.push(current);

  return words.filter(w => w.trim().length > 0);
}
```

### Step 4: Bengali Active Keywords

```javascript
// বাংলা active keyword list — এই শব্দগুলো orange হবে
const BENGALI_ACTIVE_KEYWORDS = [
  'ক্লড',     // Claude
  'কোড',      // Code
  'এডিট',     // Edit
  'প্রম্পট',  // Prompt
  'একটা',     // One (emphasis)
  'শুধু',     // Only (emphasis)
  'সম্পূর্ণ', // Complete
  'স্বয়ংক্রিয়', // Automatic
];

function isBengaliKeyword(word) {
  return BENGALI_ACTIVE_KEYWORDS.some(kw =>
    word.normalize('NFC').includes(kw.normalize('NFC'))
  );
}
```

### Step 5: Mixed Bengali-English Caption

ক্যাপশনে Bengali এবং English একসাথে থাকলে:

```javascript
function renderMixedCaption(words) {
  return words.map(word => {
    const isBengali = /[\u0980-\u09FF]/.test(word);
    const isKw = isBengali ? isBengaliKeyword(word) : isActiveKeyword(word);

    return `<span
      class="caption-word${isKw ? ' keyword' : ''}"
      lang="${isBengali ? 'bn' : 'en'}"
      style="${isBengali ? 'text-transform:none;font-family:var(--bengali-font);' : ''}"
    >${word}</span>`;
  }).join('');
}
```

### Step 6: WhisperX দিয়ে Bengali Word Timestamps

সবচেয়ে accurate word timing-এর জন্য:

```bash
# WhisperX install
pip install whisperx

# Bengali audio থেকে word-level timestamp সহ SRT
whisperx audio.wav \
  --language bn \
  --model large-v2 \
  --word_timestamps True \
  --output_format srt \
  --output_dir subtitles/

# Output: subtitles/audio.srt — প্রতিটি word-এর exact timestamp
```

WhisperX-generated SRT format (word-level):
```srt
1
00:00:01,240 --> 00:00:01,720
এটা

2
00:00:01,720 --> 00:00:02,180
কীভাবে

3
00:00:02,180 --> 00:00:02,800
সম্ভব
```

এই format HyperFrames সরাসরি consume করতে পারে — কোনো estimation দরকার নেই।

### Bengali Caption Checklist

```
বাংলা ক্যাপশন render করার আগে:
  [ ] SRT ফাইল UTF-8 encoding-এ save করা হয়েছে
  [ ] NotoSansBengali-Bold.woff2 assets/fonts/-এ আছে
  [ ] splitBengaliWords() দিয়ে words split করা হয়েছে
  [ ] Bengali words-এ text-transform: none set করা হয়েছে
  [ ] Active keywords Bengali-তেও orange হচ্ছে
  [ ] Mixed content (EN+BN) সঠিকভাবে render হচ্ছে
  [ ] Safe zone মানা হচ্ছে (Bengali text generally taller than Latin)
  [ ] Font fallback chain কাজ করছে: CustomFont → NotoSansBengali → Hind Siliguri
```

---

## Quick Reference — Commands

```bash
# Full render pipeline
npx hyperframes render \
  --template templates/main.html \
  --sync-map sync-map.json \
  --srt subtitles/captions.srt \
  --assets assets/ \
  --output output/final.mp4 \
  --resolution 1080x1920 \
  --fps 25

# Subject cutout only
npx hyperframes cutout \
  --input footage/speaker.mp4 \
  --output assets/cutout/speaker.webm

# Safe zone check (generates overlay image)
npx hyperframes safezone-check --input output/final.mp4 --frame 1

# Bengali SRT generation with WhisperX
whisperx audio.wav --language bn --word_timestamps True --output_format srt --output_dir subtitles/

# Loop close (last frame = first frame)
python loop_close.py --input output/final.mp4 --output output/final_loop.mp4
```

---

> **এই স্কিলের লক্ষ্য হলো একটি prompt থেকে production-ready, Roberto Nickson স্টাইলের ভিডিও তৈরি করা — zero manual editing, zero safe zone violation, zero meme-edit chaos।**
