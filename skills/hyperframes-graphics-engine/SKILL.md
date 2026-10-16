---
name: hyperframes-graphics-engine
description: |
  HyperFrames (HTML/CSS/JS) ব্যবহার করে word-level synchronized ক্যাপশন,
  অ্যানিমেটেড মোশন গ্রাফিক্স, সাবজেক্ট কাটআউট এবং প্রিমিয়াম টাইপোগ্রাফি তৈরি করে।
  ইমেজ, B-roll ও কাস্টম ফন্ট থেকে প্রিমিয়াম মোশন গ্রাফিক্স তৈরি করে।
  সোশ্যাল মিডিয়ার সেফ-জোন রুলস সবসময় মেনে চলে।
version: "2.0"
author: AI Systems Lab
---

# HyperFrames Graphics Engine — Skill Reference

## Overview

HyperFrames Graphics Engine হলো একটি সম্পূর্ণ HTML/CSS/JS-চালিত মোশন গ্রাফিক্স পাইপলাইন যা Roberto Nickson-এর সিগনেচার এডিটিং স্টাইলকে কোড দিয়ে রিক্রিয়েট করে। Roberto Nickson-এর কাজের বৈশিষ্ট্য হলো:

- **Minimal কিন্তু impactful** — প্রতিটি গ্রাফিক উদ্দেশ্যহীন নয়, প্রতিটি মুহূর্ত audio বা narrative-এর সাথে সরাসরি সংযুক্ত।
- **Word-level caption timing** — ক্যাপশন পুরো লাইনে একসাথে নয়, প্রতিটি শব্দ আলাদাভাবে beat এ আসে।
- **Subject-aware layering** — বক্তার cutout-এর পেছনে বড় টেক্সট বা গ্রাফিক রাখা হয় যাতে মনে হয় লোকটি গ্রাফিকের সামনে দাঁড়িয়ে আছে।
- **Restrained color palette** — শুধুমাত্র White `#FFFFFF`, Dark `#141413`, এবং Accent Orange `#D97757` ব্যবহার করা হয়। অতিরিক্ত রঙ যোগ করা হয় না।
- **Beat-synchronized graphics** — প্রতিটি visual effect audio-র একটি নির্দিষ্ট beat বা spoken keyword-এর সাথে frame-accurate sync করা।
- **Expo-out easing** — সমস্ত অ্যানিমেশন `cubic-bezier(0.16, 1, 0.3, 1)` ব্যবহার করে; কোনো bounce, spring বা overshoot নেই।

এই স্কিল HyperFrames renderer-এর জন্য HTML templates তৈরি করে এবং প্রতিটি frame-কে ffmpeg pipeline-এর মাধ্যমে final video-তে composite করে। ইনপুট হিসেবে SRT subtitle file, WAV/MP3 audio, B-roll footage, এবং assets folder গ্রহণ করে।

---

## 🎨 Vector Motion Graphics Subsystem (SVG, Charts & B-Roll Integration)

HyperFrames ইঞ্জিনটি আমাদের ইন্টিগ্রেটেড স্পেশালাইজড মোশন স্কিলগুলোকে সরাসরি সিনের কনটেক্সট অনুযায়ী কল করে এবং প্রতিটি ফ্রেম ভেক্টর ফরম্যাটে রেন্ডার করে:

### ১. পিওর ভেক্টর SVG অ্যানিমেশন (`skills/svg-animation`)
- **Hand-Drawn Circle / Underline:** কোনো গুরুত্বপূর্ণ শব্দ বা টেক্সটের নিচে হাত দিয়ে দাগ দেওয়ার অ্যানিমেশন `stroke-dasharray` এবং `stroke-dashoffset` দিয়ে ফ্রেম-অ্যাকিউরেট রেন্ডার হয়।
- **SVG Path Drawing:** পেন-টুল দিয়ে আঁকা ভেক্টর অ্যারো (Arrow), পয়েন্টার, বা ডায়াগ্রাম পাথ ১.২ সেকেন্ডে `expo-out` ইজিং দিয়ে ড্র-অন হয়।
- **Shape Morphing:** এক আইকন থেকে অন্য আইকন বা শেপে স্মুথ ভেক্টর মরফিং।
- **1080p ও 4K ক্রিস্পনেস:** ভেক্টর হওয়ায় কখনোই পিক্সেলেশন হয় না।

### ২. টকিং-হেড মোশন বি-রোল (`skills/talking-head-b-roll`)
- বক্তার কথা অনুযায়ী স্বয়ংক্রিয়ভাবে ৩টি ট্রিটমেন্ট নির্বাচন:
  - **Treatment A (Speaker):** শুধু বক্তা (কোনো ওভারলে নেই)।
  - **Treatment B (Overlay):** বক্তা দৃশ্যমান, একপাশে স্বচ্ছ অ্যানিমেটেড ভেক্টর কার্ড বা কি-ওয়ার্ড।
  - **Treatment C (Cover):** বক্তাকে ঢেকে ফুল-স্ক্রিন ক্যানভাস-চালিত মোশন সিন (ডাটা, প্রসেস বা উদাহরণের জন্য)।

### ৩. ডায়নামিক ডাটা চার্ট ও ইনফোগ্রাফিক (`skills/chart-animation` & `skills/animated-infographic`)
- সংখ্যার ক্ষেত্রে `scaleLinear` এবং `interpolate` দিয়ে ফ্রেম-ড্রিভেন বার চার্ট বা লাইন গ্রাফ রিভিল।
- নাম্বার কাউন্টার / টিকার যা ০ থেকে শুরু হয়ে বাস্তব সংখ্যায় এসে থামে।

### ৪. ট্রান্সপারেন্ট লোয়ার থার্ডস (`skills/lower-thirds`)
- চ্যানেল লোগো, বক্তার নাম, সোশ্যাল হ্যান্ডেল এবং সাবস্ক্রাইব রিমাইন্ডার স্বচ্ছ আলফা চ্যানেলসহ রেন্ডার করে মূল ফুটেজে কম্পোজিট করা।

---

## Caption System (ক্যাপশন ডিজাইন সিস্টেম)

### Word-Boundary Timing Model

প্রতিটি ক্যাপশন লাইন পুরো sentence হিসেবে নয় — প্রতিটি **শব্দ** আলাদা timestamp-এ render হয়। প্রতি স্ক্রিনে সর্বোচ্চ **২–৪ শব্দ** দেখানো হয়। এর বেশি হলে readability কমে এবং Roberto Nickson স্টাইলের punch হারায়।

**Word-boundary timing algorithm (SRT থেকে):**

```
1. SRT ফাইল parse করো — প্রতিটি cue-এর start_ms, end_ms, এবং text বের করো।
2. প্রতিটি cue-এর text-কে শব্দে বিভক্ত করো (Unicode word boundary বা space-split)।
3. cue duration = end_ms - start_ms
4. প্রতিটি শব্দের estimated duration = cue_duration / word_count
5. প্রতিটি শব্দের start_ms = cue_start_ms + (word_index × estimated_word_duration)
6. active keyword হলে সেই শব্দটি #D97757 রঙে render করো।
7. যদি SRT-তে word-level timestamp থাকে (WhisperX বা Whisper word_timestamps=True):
   - সরাসরি সেই timestamp ব্যবহার করো, step 4–5 skip করো।
8. প্রতিটি শব্দের fade-in: 80ms, easing: expo-out।
9. পরবর্তী শব্দ আসার আগে পূর্বের শব্দটি screen-এ থাকে।
10. স্ক্রিনে word count 4 পূর্ণ হলে পুরো line clear করে নতুন শুরু।
```

**বাংলা word boundary detection:**

বাংলা ভাষায় শব্দের মাঝে space সবসময় থাকে না — Unicode ZWJ (Zero Width Joiner) এবং ZWNJ (Zero Width Non-Joiner) character বিবেচনা করতে হয়। Algorithm:

```javascript
function splitBengaliWords(text) {
  // Unicode normalize করো: NFC form
  const normalized = text.normalize("NFC");
  // Bengali Unicode range: U+0980–U+09FF
  // Space (\u0020) এবং Zero Width Space (\u200B) দিয়ে split
  const words = normalized.split(/[\u0020\u200B]+/).filter(Boolean);
  return words;
}

function normalizeBengali(text) {
  // Hasanta + consonant → correct conjunct rendering
  // Remove stray control chars except ZWJ/ZWNJ
  return text
    .normalize("NFC")
    .replace(/[\u200C\u200D]/g, (m) => m) // preserve ZWJ/ZWNJ
    .replace(/\s+/g, " ")
    .trim();
}
```

**Font fallback for Bengali:**

```css
.caption-text {
  font-family:
    "CustomFont", /* assets/fonts/ থেকে লোড করা */
    "Noto Sans Bengali",
    "Hind Siliguri",
    sans-serif;
  unicode-bidi: plaintext; /* Bengali RTL/LTR mixed support */
}
```

### Typography Specification

| Property        | Value                                               |
|-----------------|-----------------------------------------------------|
| Font source     | `assets/fonts/` (.woff2 → .ttf → system fallback)  |
| Default font    | SF Pro Display Bold (system) / Noto Sans Bengali    |
| Font size       | ~68px @ 1080px canvas width                         |
| Font weight     | 700–800 (Bold / ExtraBold)                          |
| Letter spacing  | -1px to -2px (tight, modern)                        |
| Line height     | 1.1–1.2 (compact)                                   |
| Text transform  | UPPERCASE বেশিরভাগ সময়                              |
| Active keyword  | `#D97757` (Claude orange)                            |
| Inactive word   | `#FFFFFF` (white)                                   |
| Text shadow     | `0 1px 2px rgba(0,0,0,0.6)` (subtle, 1–2px blur)   |
| Fade-in timing  | 80ms                                                |
| Easing          | `cubic-bezier(0.16, 1, 0.3, 1)` (expo-out)         |
| Bounce          | **নেই** — কোনো bounce বা spring effect নিষিদ্ধ       |

### Caption Position Rules

- **Vertical**: lower-middle area — canvas height-এর ~75%–85% এর মধ্যে
- **Horizontal**: centered (text-align: center)
- **মুখের উপরে কখনো না**: subject face detection বা manual safe-zone দিয়ে মুখের region বাদ দাও
- **Safe zone মানতেই হবে**: নিচে 420px থেকে উপরে থাকো (নিচে UI element), উপরে 220px এর নিচে থাকো
- **Multi-line**: সর্বোচ্চ ২ লাইন, প্রতি লাইনে ২ শব্দ

### CSS Animation Template (Word Fade-In)

```css
@keyframes wordFadeIn {
  from {
    opacity: 0;
    transform: translateY(6px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.caption-word {
  display: inline-block;
  opacity: 0;
  animation: wordFadeIn 80ms cubic-bezier(0.16, 1, 0.3, 1) forwards;
  color: #ffffff;
  font-size: 68px;
  font-weight: 800;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.6);
  margin: 0 4px;
}

.caption-word.active-keyword {
  color: #d97757;
}
```

---

## Safe Zone Rules (CRITICAL)

> **এই নিয়মগুলো ভাঙলে Instagram/TikTok/YouTube Shorts UI elements ক্যাপশন বা গ্রাফিক্সকে ঢেকে ফেলবে। প্রতিটি graphic element এই boundary-এর মধ্যে থাকতেই হবে।**

### Platform Safe Zone Map (1080×1920 canvas)

```
┌─────────────────────────────────┐  ← y=0 (top of frame)
│         UNSAFE ZONE             │
│    (Platform UI: notch, etc)    │
│         [220px]                 │
├─────────────────────────────────┤  ← y=220 (SAFE TOP BOUNDARY)
│                                 │
│                                 │
│        ✅ SAFE CONTENT          │
│           ZONE                  │
│                                 │
│   ←140px→           ←140px→    │
│   UNSAFE             UNSAFE     │
│   (battery,          (gallery,  │
│    signal)           buttons)   │
│                                 │
│                                 │
├─────────────────────────────────┤  ← y=1500 (SAFE BOTTOM BOUNDARY)
│         UNSAFE ZONE             │
│  (Instagram like/comment bar,   │
│   TikTok nav, YT Shorts UI)     │
│         [420px]                 │
└─────────────────────────────────┘  ← y=1920 (bottom of frame)
```

### Numeric Boundaries

| Edge     | Unsafe zone size | Safe content starts/ends at |
|----------|------------------|-----------------------------|
| Top      | 220px            | y ≥ 220                     |
| Bottom   | 420px            | y ≤ 1500 (1920 - 420)       |
| Left     | 140px            | x ≥ 140                     |
| Right    | 140px            | x ≤ 940 (1080 - 140)        |

**Safe Content Area:** `140px` → `940px` horizontally × `220px` → `1500px` vertically  
**Safe Width:** 800px (1080 - 140×2)  
**Safe Height:** 1280px (1920 - 220 - 420)

### Animation Motion Constraint

- কোনো element তার **initial বা final position**-এ safe zone লঙ্ঘন করতে পারবে না।
- `translateY(-50px)` animation যদি element-কে safe top boundary-র বাইরে নিয়ে যায়, সেটি ব্যবহার নিষিদ্ধ।
- Shadow এবং glow effect-ও safe zone মেনে চলে — `box-shadow: 0 0 30px` radius হিসেব করতে হবে।

### CSS Safe Zone Variables

```css
:root {
  --safe-top: 220px;
  --safe-bottom: 420px;
  --safe-left: 140px;
  --safe-right: 140px;
  --safe-width: calc(1080px - var(--safe-left) - var(--safe-right)); /* 800px */
  --safe-height: calc(1920px - var(--safe-top) - var(--safe-bottom)); /* 1280px */
  --canvas-width: 1080px;
  --canvas-height: 1920px;
}

.safe-zone-element {
  position: absolute;
  left: var(--safe-left);
  top: var(--safe-top);
  width: var(--safe-width);
  max-height: var(--safe-height);
  overflow: hidden;
}
```

### Stress Test: Phone Crop Simulation

প্রতিটি final render-এর জন্য phone crop simulation চালাও:

```bash
# ffmpeg দিয়ে Instagram portrait crop simulate করো
ffmpeg -i output.mp4 \
  -vf "crop=1080:1920:0:0,drawbox=x=0:y=0:w=1080:h=220:color=red@0.3:t=fill,\
       drawbox=x=0:y=1500:w=1080:h=420:color=red@0.3:t=fill,\
       drawbox=x=0:y=0:w=140:h=1920:color=blue@0.2:t=fill,\
       drawbox=x=940:y=0:w=140:h=1920:color=blue@0.2:t=fill" \
  -frames:v 1 safe_zone_check.jpg
```

লাল zone-এ কোনো গ্রাফিক বা ক্যাপশন থাকলে সেটি fix করতে হবে।

---

## Motion Graphics System

### Design Palette

| Role           | Color     | Hex       | Usage                                      |
|----------------|-----------|-----------|---------------------------------------------|
| Background     | Dark      | `#141413` | Dark cards, terminal backgrounds            |
| Primary text   | White     | `#FFFFFF` | Headlines, body captions                    |
| Accent/CTA     | Orange    | `#D97757` | Active keywords, brand highlights, CTAs     |
| Muted          | Gray      | `#8A8A8A` | Secondary info, timestamps, labels          |
| Overlay        | Dark      | `rgba(20,20,19,0.85)` | Semi-transparent card backgrounds |

**Rounded corners:** সব কার্ড এবং badge-এ `border-radius: 8px` থেকে `16px`।  
**Easing (সার্বজনীন):** `cubic-bezier(0.16, 1, 0.3, 1)` — expo-out। যেকোনো animation-এ এটাই ব্যবহার হবে।

---

### Beat-Specific Graphics (Sync Map)

প্রতিটি beat-এর জন্য নিচে HTML template এবং timing spec দেওয়া হলো।

---

#### 1. Opening Title — "EDITED 100% WITH CLAUDE CODE"

**Trigger:** Video শুরু (0:00:00)  
**Duration:** ~2.5s  
**Design:**

```
┌─────────────────────────────────┐
│                                 │
│   EDITED 100% WITH              │
│   ██████ CODE                   │  ← CLAUDE = #D97757 orange
│                                 │
└─────────────────────────────────┘
```

**Animation:** প্রতিটি শব্দ আলাদাভাবে 80ms expo-out fade-in করে। "CLAUDE CODE" শেষে আসে এবং orange glow সহ।

```css
.opening-title .highlight {
  color: #d97757;
  text-shadow: 0 0 20px rgba(217, 119, 87, 0.4);
}
```

---

#### 2. "cuts" — Hard Jump Cut

**Trigger:** "cuts" keyword audio-তে  
**Design:** কোনো extra graphic নেই — ffmpeg-এর `select` filter দিয়ে hard cut। ক্যাপশনে "CUTS" শব্দটি `#D97757` রঙে।

```bash
# ffmpeg hard cut — no transition
ffmpeg -i input.mp4 -vf "trim=start=2.1:end=2.15,setpts=PTS-STARTPTS" cut_segment.mp4
```

---

#### 3. "colour grading" — Vertical Line Sweep Wipe

**Trigger:** "colour grading" keyword  
**Duration:** 1.2s  
**Design:** ভার্টিক্যাল লাইন বাম থেকে ডানে sweep করে — বাম দিক ungraded (desaturated), ডান দিক graded (warm tones)।

```css
@keyframes wipeReveal {
  from { clip-path: inset(0 100% 0 0); }
  to   { clip-path: inset(0 0% 0 0); }
}

.graded-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(to right, transparent, rgba(217,119,87,0.08));
  animation: wipeReveal 1.2s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

.wipe-line {
  position: absolute;
  top: 0; bottom: 0;
  width: 3px;
  background: #ffffff;
  box-shadow: 0 0 12px rgba(255,255,255,0.8);
  animation: wipeLineMove 1.2s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

@keyframes wipeLineMove {
  from { left: 0%; }
  to   { left: 100%; }
}
```

---

#### 4. "captions" — Caption Switch On, Title Fade Out

**Trigger:** "captions" keyword  
**Design:** Title graphic fade out (200ms), caption system fade in (80ms per word)।

```javascript
// Transition sequence
timeline.at(keyword_ms)
  .fadeOut('.title-graphic', { duration: 200, easing: 'expo-out' })
  .fadeIn('.caption-system', { duration: 80, stagger: 80, easing: 'expo-out' });
```

---

#### 5. "zoom ins" / "zoom outs" — Push/Pull + Whoosh

**Trigger:** "zoom ins" বা "zoom outs" keyword  
**Duration:** 0.6s  
**Design:**

- **Zoom in:** `scale(1.0)` → `scale(1.12)`, `cubic-bezier(0.16, 1, 0.3, 1)`
- **Zoom out:** `scale(1.12)` → `scale(1.0)`, `cubic-bezier(0.16, 1, 0.3, 1)`
- Whoosh SFX: short high-pass filtered audio `[whoosh.wav]`

```css
@keyframes pushIn {
  from { transform: scale(1.0); }
  to   { transform: scale(1.12); }
}
@keyframes pullOut {
  from { transform: scale(1.12); }
  to   { transform: scale(1.0); }
}
.video-layer.zoom-in  { animation: pushIn  0.6s cubic-bezier(0.16, 1, 0.3, 1) forwards; }
.video-layer.zoom-out { animation: pullOut 0.6s cubic-bezier(0.16, 1, 0.3, 1) forwards; }
```

---

#### 6. "motion graphics" — MOTION GRAPHICS Text Build from Orange Bar

**Trigger:** "motion graphics" keyword  
**Duration:** 0.8s  
**Design:** একটি orange horizontal bar (`#D97757`) নিচ থেকে উপরে reveal করে, তারপর "MOTION GRAPHICS" টেক্সট বার থেকে উপরে build হয়।

```css
.mg-bar {
  position: absolute;
  bottom: 0; left: 0; right: 0;
  height: 6px;
  background: #d97757;
  transform-origin: left center;
  animation: barExpand 0.4s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
@keyframes barExpand {
  from { transform: scaleX(0); }
  to   { transform: scaleX(1); }
}

.mg-text {
  position: absolute;
  bottom: 20px;
  font-size: 72px;
  font-weight: 800;
  color: #ffffff;
  clip-path: inset(100% 0 0 0);
  animation: textRise 0.4s 0.35s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
@keyframes textRise {
  from { clip-path: inset(100% 0 0 0); transform: translateY(20px); }
  to   { clip-path: inset(0% 0 0 0);   transform: translateY(0); }
}
```

---

#### 7. "sound effects" — Minimal Waveform Pulse

**Trigger:** "sound effects" keyword  
**Duration:** 1s loop (2–3 cycles)  
**Design:** ছোট waveform bars (5–7টি vertical bar), pulse animation। meme-style নয়, minimal।

```css
.waveform-bar {
  width: 4px;
  border-radius: 2px;
  background: #d97757;
  animation: waveformPulse 0.5s cubic-bezier(0.16, 1, 0.3, 1) infinite alternate;
}
@keyframes waveformPulse {
  from { height: 8px;  opacity: 0.4; }
  to   { height: 40px; opacity: 1.0; }
}
/* প্রতিটি bar-এর animation-delay ভিন্ন: 0ms, 60ms, 120ms, 60ms, 0ms */
```

---

#### 8. "single editing app" — Dark Card "Editing apps opened: 0"

**Trigger:** "single editing app" keyword  
**Duration:** 2s display  
**Design:** Dark card (`#141413`, `border-radius: 16px`), subtitle text, counter "0" bold orange।

```html
<div class="stat-card">
  <p class="stat-label">Editing apps opened</p>
  <p class="stat-value">
    <span class="stat-number orange">0</span>
  </p>
</div>
```

```css
.stat-card {
  background: #141413;
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 16px;
  padding: 24px 32px;
  animation: cardReveal 0.5s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
.stat-value { font-size: 96px; font-weight: 800; }
.orange { color: #d97757; }
```

---

#### 9. "raw footage" Timeline — Real Data Timeline Strip

**Trigger:** "raw footage" keyword  
**Duration:** 2–3s  
**Design:** Horizontal timeline strip — লাল = bad take, ধূসর = pause/gap, সবুজ/white = usable।

```
[████░░░████████░░██░░░░███░░████████]
 ↑                                   ↑
 bad  pause  good  pause bad  good   end
```

```css
.timeline-strip {
  display: flex;
  height: 20px;
  border-radius: 10px;
  overflow: hidden;
  width: 100%;
}
.timeline-segment.bad   { background: #c0392b; } /* লাল — bad take */
.timeline-segment.pause { background: #555555; } /* ধূসর — pause */
.timeline-segment.good  { background: #ffffff; opacity: 0.85; } /* white — usable */

/* Segments fade in left-to-right */
.timeline-segment {
  animation: segReveal 0.6s cubic-bezier(0.16, 1, 0.3, 1) forwards;
  transform-origin: left;
}
```

Raw footage data থেকে segment প্রপোর্শন calculate করে render করো।

---

#### 10. "one prompt" — Dark Terminal Card + Counter

**Trigger:** "one prompt" keyword  
**Duration:** 2.5s  
**Design:**
1. Dark terminal card appear হয়
2. First prompt line type করে (character-by-character, 30ms/char)
3. Counter animate করে: raw word count → final word count

```html
<div class="terminal-card">
  <div class="terminal-header">
    <span class="dot red"></span>
    <span class="dot yellow"></span>
    <span class="dot green"></span>
    <span class="terminal-title">Claude Sonnet 4</span>
  </div>
  <div class="terminal-body">
    <span class="prompt-line" id="typeTarget"></span>
    <span class="cursor">▋</span>
  </div>
  <div class="counter-row">
    <span class="counter-label">Words:</span>
    <span class="counter-value" id="wordCounter">2847</span>
    <span class="counter-arrow">→</span>
    <span class="counter-final">312</span>
  </div>
</div>
```

```javascript
// Typewriter effect
function typeWriter(element, text, speed = 30) {
  let i = 0;
  const interval = setInterval(() => {
    element.textContent += text[i++];
    if (i >= text.length) clearInterval(interval);
  }, speed);
}

// Counter animation: raw → final
function animateCounter(el, from, to, duration = 1200) {
  const start = performance.now();
  const update = (now) => {
    const t = Math.min((now - start) / duration, 1);
    const ease = 1 - Math.pow(1 - t, 4); // expo-out
    el.textContent = Math.round(from + (to - from) * ease);
    if (t < 1) requestAnimationFrame(update);
  };
  requestAnimationFrame(update);
}
```

---

#### 11. "This is what I gave it" — Ungraded + RAW Tag + Timecode

**Trigger:** "This is what I gave it" keyword  
**Design:** B-roll footage overlay-এ:
- কোণে `RAW` badge (red background `#c0392b`, white text, `border-radius: 8px`)
- Timecode burn-in: `HH:MM:SS:FF` monospace font, white on dark background
- Desaturated/flat color grade simulate করো ffmpeg filter দিয়ে

```css
.raw-badge {
  background: #c0392b;
  color: #ffffff;
  font-size: 24px;
  font-weight: 700;
  letter-spacing: 3px;
  padding: 6px 14px;
  border-radius: 8px;
  position: absolute;
  top: calc(var(--safe-top) + 20px);
  left: calc(var(--safe-left) + 20px);
}
.timecode {
  font-family: "Courier New", monospace;
  font-size: 28px;
  color: #ffffff;
  background: rgba(0,0,0,0.7);
  padding: 4px 10px;
  border-radius: 4px;
  position: absolute;
  bottom: calc(var(--safe-bottom) + 20px);
  right: calc(var(--safe-right) + 20px);
}
```

---

#### 12. "what it gave me back" — Riser + Wipe to Final

**Trigger:** "what it gave me back" keyword  
**Duration:** 1s  
**Design:** Audio riser SFX সহ white wipe transition। Left to right reveal করে graded/edited version।

```css
@keyframes riserWipe {
  0%   { clip-path: inset(0 100% 0 0); }
  100% { clip-path: inset(0 0%   0 0); }
}
.final-reveal {
  position: absolute;
  inset: 0;
  animation: riserWipe 1s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
```

---

#### 13. "Make the captions bigger" — Typed Message + Scale 140%

**Trigger:** "Make the captions bigger" keyword  
**Design:**
1. Chat bubble appear — "Make the captions bigger" type করো
2. ক্যাপশন `font-size: 68px` → `font-size: 95px` (140%) animate করো

```css
.chat-bubble {
  background: #d97757;
  color: #ffffff;
  border-radius: 16px 16px 4px 16px;
  padding: 14px 22px;
  font-size: 32px;
  font-weight: 600;
  animation: bubblePop 0.3s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
@keyframes bubblePop {
  from { transform: scale(0.8); opacity: 0; }
  to   { transform: scale(1.0); opacity: 1; }
}

/* Caption scale up */
.caption-text {
  transition: font-size 0.5s cubic-bezier(0.16, 1, 0.3, 1);
}
.caption-text.enlarged { font-size: 95px; }
```

---

#### 14. "posted last year" — Old Video Card + Grey BEFORE Tag

**Trigger:** "posted last year" keyword  
**Duration:** 2s  
**Design:** Rounded card (`border-radius: 16px`) ভেতরে old video thumbnail, কোণে grey "BEFORE" badge।

```css
.before-card {
  border-radius: 16px;
  overflow: hidden;
  border: 2px solid rgba(255,255,255,0.12);
  position: relative;
  animation: cardReveal 0.5s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
.before-badge {
  position: absolute;
  top: 12px; left: 12px;
  background: #8a8a8a;
  color: #ffffff;
  font-size: 22px;
  font-weight: 700;
  letter-spacing: 2px;
  padding: 5px 12px;
  border-radius: 8px;
}
```

---

#### 15. "Claude's re-edit" — Wipe + Orange CLAUDE RE-EDIT Tag

**Trigger:** "Claude's re-edit" keyword  
**Design:** Wipe transition (same as beat 12) + "CLAUDE RE-EDIT" orange badge।

```css
.claude-badge {
  background: #d97757;
  color: #ffffff;
  font-size: 22px;
  font-weight: 800;
  letter-spacing: 2px;
  padding: 5px 14px;
  border-radius: 8px;
}
```

---

#### 16. "Comment CLAUDE" — Kinetic Text + Comment Bubble

**Trigger:** "Comment CLAUDE" keyword  
**Duration:** 2s  
**Design:** "CLAUDE" শব্দটি kinetic zoom-in করে, তারপর comment bubble appear হয়।

```css
.kinetic-word {
  font-size: 120px;
  font-weight: 800;
  color: #d97757;
  animation: kineticZoom 0.5s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
@keyframes kineticZoom {
  from { transform: scale(0.5); opacity: 0; }
  to   { transform: scale(1.0); opacity: 1; }
}

.comment-bubble {
  background: rgba(20, 20, 19, 0.9);
  border: 1px solid rgba(255,255,255,0.15);
  border-radius: 12px;
  padding: 12px 20px;
  font-size: 28px;
  display: flex;
  align-items: center;
  gap: 10px;
}
.comment-bubble::before {
  content: "💬";
  font-size: 24px;
}
```

---

#### 17. Loop Frame — Last Frame = Frame 0

**Critical:** Video-র শেষ frame হুবহু প্রথম frame-এর মতো হতে হবে। এটা loop playback-এর জন্য।

```bash
# ffmpeg দিয়ে first frame extract করো
ffmpeg -i output.mp4 -frames:v 1 -q:v 2 first_frame.jpg

# Last frame replace করো first frame দিয়ে
# Python script দিয়ে frame-accurate concatenation
python loop_close.py --input output.mp4 --output final_loop.mp4
```

```python
# loop_close.py
import subprocess

def close_loop(input_path, output_path):
    """Replace last frame with first frame for seamless loop."""
    # Get total frame count
    probe = subprocess.run([
        'ffprobe', '-v', 'error', '-select_streams', 'v:0',
        '-count_packets', '-show_entries', 'stream=nb_read_packets',
        '-of', 'csv=p=0', input_path
    ], capture_output=True, text=True)
    total_frames = int(probe.stdout.strip())

    # Extract first frame as image
    subprocess.run([
        'ffmpeg', '-i', input_path, '-frames:v', '1',
        '-q:v', '1', '/tmp/first_frame.png', '-y'
    ])

    # Replace last frame: concat all-but-last + first frame image
    subprocess.run([
        'ffmpeg',
        '-i', input_path,
        '-i', '/tmp/first_frame.png',
        '-filter_complex',
        f'[0:v]trim=0:{total_frames-1}/25,setpts=PTS-STARTPTS[main];'
        f'[1:v]loop=1:1:0,trim=0:1/25,setpts=PTS-STARTPTS+({total_frames-1}/25)/TB[last];'
        f'[main][last]concat=n=2:v=1[out]',
        '-map', '[out]',
        '-map', '0:a',
        '-c:v', 'libx264', '-c:a', 'copy',
        output_path, '-y'
    ])
```

---

## Subject Cutout (Background Separation)

### কখন ব্যবহার করবে

Subject cutout শুধুমাত্র সেই beat-এ ব্যবহার করো যেখানে background-এ **বড় গ্রাফিক বা টেক্সট** রাখা হবে। প্রতিটি beat-এ নয় — selective ব্যবহার premium look দেয়।

**ব্যবহারের beats:**
- Beat 6: "motion graphics" — MOTION GRAPHICS টেক্সট বক্তার পেছনে
- Beat 1: Opening title — headline টেক্সট subject-এর পেছনে layer
- Beat 10: "one prompt" — terminal card subject-এর পেছনে

### Cutout Method

**Option A: rembg (recommended for static shots)**
```bash
# Python rembg দিয়ে background remove
pip install rembg
rembg i input_frame.png output_cutout.png

# Video-র প্রতিটি frame-এ apply করো
ffmpeg -i input.mp4 -vf fps=25 frames/frame_%04d.png
for f in frames/*.png; do rembg i "$f" "cutout_${f##*/}"; done
ffmpeg -r 25 -i cutout_frame_%04d.png -c:v prores_ks -alpha_bits 16 subject.mov
```

**Option B: npx hyperframes cutout**
```bash
npx hyperframes cutout \
  --input footage/speaker.mp4 \
  --output assets/cutout/speaker_cutout.webm \
  --format webm-alpha \
  --fps 25
```

**Option C: ffmpeg chromakey (green screen footage)**
```bash
ffmpeg -i input.mp4 \
  -vf "chromakey=green:0.1:0.05" \
  -c:v libvpx-vp9 -b:v 0 -crf 10 \
  -auto-alt-ref 0 \
  subject_cutout.webm
```

### Composite Layer Order (Z-Index)

```
Layer 5 (top):    Subject cutout (বক্তার foreground)
Layer 4:          Caption text
Layer 3:          Motion graphic / big text
Layer 2:          B-roll or color overlay
Layer 1 (bottom): Base video / background
```

```css
.layer-background  { z-index: 1; }
.layer-broll       { z-index: 2; }
.layer-graphic     { z-index: 3; }
.layer-caption     { z-index: 4; }
.layer-subject     { z-index: 5; }
```

---

## Images Folder Integration (`assets/images/`)

### Supported Formats

| Format | Use Case                            |
|--------|--------------------------------------|
| `.png` | Logo, screenshot, transparency-required assets |
| `.svg` | Logo, icon, scalable graphics        |
| `.jpg` | Thumbnail, photo reference           |
| `.webp`| Optimized web-format images          |

### File Naming Convention → Beat Mapping

ফাইল নামের keyword থেকে কোন beat-এ ব্যবহার হবে তা automatically detect করো:

| File name pattern       | Maps to beat                         |
|-------------------------|--------------------------------------|
| `logo.*`                | Opening title, brand card            |
| `tool-screenshot.*`     | "single editing app" বা tool demo beat |
| `before.*`              | "posted last year" beat (BEFORE card)|
| `after.*`               | "Claude's re-edit" beat              |
| `thumbnail.*`           | "posted last year" card              |
| `banner.*`              | Background overlay beat              |
| `icon-*.*`              | Badge বা label-এ inline icon         |
| `product-*.*`           | Product showcase beat                |
| `map-*.*`               | Location/context beat                |
| `screenshot-*.*`        | Screen recording demo beat           |

### PNG Transparency Composite

```bash
# ffmpeg overlay filter — transparent PNG composite করো video-র উপরে
ffmpeg -i base_video.mp4 -i logo.png \
  -filter_complex \
  "[0:v][1:v] overlay=x=(W-w)/2:y=240:format=auto,format=yuv420p" \
  -c:v libx264 -crf 18 output_with_logo.mp4
```

### Animated Card with Image

```html
<div class="image-card" style="animation: cardReveal 0.5s cubic-bezier(0.16,1,0.3,1) forwards;">
  <img src="assets/images/tool-screenshot.png"
       alt="Tool demo"
       style="width:100%; border-radius:12px 12px 0 0; display:block;">
  <div class="image-card-label">
    <span class="label-text">Claude Code in action</span>
  </div>
</div>
```

---

## Font Integration

### Font Discovery Algorithm

```
1. assets/fonts/ ফোল্ডার scan করো
2. .woff2 ফাইল খোঁজো → সর্বোচ্চ priority
3. .ttf ফাইল খোঁজো → দ্বিতীয় priority
4. .otf ফাইল খোঁজো → তৃতীয় priority
5. কোনো ফাইল না থাকলে: system font "SF Pro Display" (macOS) / "Segoe UI" (Windows)
6. বাংলা ক্যাপশন detect করলে: Bengali-compatible font যাচাই করো
   - Unicode range U+0980–U+09FF cover করে কিনা দেখো
   - না করলে: Google Fonts "Noto Sans Bengali" বা "Hind Siliguri" load করো
```

### CSS @font-face Template

```css
/* Primary font from assets/fonts/ */
@font-face {
  font-family: "CustomFont";
  src:
    url("../assets/fonts/CustomFont-Bold.woff2") format("woff2"),
    url("../assets/fonts/CustomFont-Bold.ttf")   format("truetype");
  font-weight: 700;
  font-display: block; /* layout shift এড়াতে */
}

/* Bengali fallback */
@font-face {
  font-family: "NotoSansBengali";
  src: url("../assets/fonts/NotoSansBengali-Bold.woff2") format("woff2");
  font-weight: 700;
  unicode-range: U+0980-U+09FF; /* Bengali Unicode block */
  font-display: block;
}

/* Global caption font stack */
.caption-text {
  font-family:
    "CustomFont",
    "NotoSansBengali",
    "Noto Sans Bengali",
    "Hind Siliguri",
    "SF Pro Display",
    "Segoe UI",
    sans-serif;
}
```

### Font Size Scaling

```javascript
// 1080px canvas-এ 68px → অন্য resolution-এ scale
function scaleFontSize(basePx, canvasWidth) {
  const referenceWidth = 1080;
  return Math.round(basePx * (canvasWidth / referenceWidth));
}
// scaleFontSize(68, 1080) → 68px
// scaleFontSize(68, 720)  → 45px
// scaleFontSize(68, 1440) → 91px
```

---

## Restraint Rules

এই নিয়মগুলো Pablo Escobar বা meme-edit স্টাইল থেকে দূরে রাখে এবং Roberto Nickson-এর premium look নিশ্চিত করে।

### The Three Laws of Restraint

**Law 1: এক বিটে একটি effect।**
একই frame-এ একাধিক animation, transition বা overlay stack করা সম্পূর্ণ নিষিদ্ধ। যদি দুটো effect একসাথে লাগে, তাহলে একটি বাদ দাও।

**Law 2: Minimum SFX।**
প্রতিটি SFX-এর পরে নিজেকে জিজ্ঞেস করো — "এটা মুছলে video কি আরো premium লাগবে?" যদি উত্তর হ্যাঁ, মুছে দাও। SFX অর্ধেক কমানো মানে double premium।

**Law 3: ২–৪ সেকেন্ডে পরিবর্তন।**
প্রতি ২–৪ সেকেন্ডে screen-এ কিছু একটা change হতে হবে — caption update, graphic switch, cut — কিন্তু সেটা সবসময় intentional।

### Effect Stack Check

```
BEFORE rendering, প্রতিটি frame-এর জন্য check করো:
- [ ] এই frame-এ কতটি animation চলছে? → 1 এর বেশি হলে, 1 বাদ দাও
- [ ] SFX কি এই beat-এ necessary? → না হলে, volume = 0
- [ ] এই effect safe zone মানছে? → না হলে, position adjust করো
- [ ] এই effect bounce বা spring ব্যবহার করছে? → হ্যাঁ হলে, expo-out দিয়ে replace করো
- [ ] শেষ visual change কত সেকেন্ড আগে হয়েছিল? → 4s+ হলে, একটি ছোট change যোগ করো
```
