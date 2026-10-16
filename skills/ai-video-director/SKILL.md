---
name: ai-video-director
description: |
  End-to-end AI ভিডিও এডিটিং পাইপলাইনের মাস্টার অর্কেস্ট্রেটর।
  HyperFrames + ffmpeg ব্যবহার করে 9:16 শর্ট-ফর্ম ভিডিও সম্পূর্ণ কোডে এডিট করে।
  স্ক্রিপ্টের প্রতিটি শব্দের সাথে সিঙ্ক্রোনাইজড ইফেক্ট (sync map) বাস্তবায়ন করে।
  SRT ফাইল থাকলে ট্রান্সক্রিপশন বাদ দেয়। বাংলা সহ মাল্টিলিং্গুয়াল সাপোর্ট।
version: "2.0"
author: AI Systems Lab
tags:
  - video-editing
  - ffmpeg
  - hyperframes
  - short-form
  - 9:16
  - captions
  - sync-map
  - multilingual
  - bengali
tools:
  - hyperframes
  - ffmpeg
  - node
  - npx
inputs:
  - raw video clips (mp4, mov, mkv)
  - optional: srt/ folder with pre-made SRT files
  - optional: images/ folder with logos, graphics, screenshots
  - optional: broll/ folder with cutaway footage
outputs:
  - output/preview.mp4
  - output/final.mp4 (1080×1920, H.264 ~18 Mbps, AAC 320k)
  - output/old_reedit.mp4
  - takes.md
---

# ai-video-director

## Overview

এই স্কিল **Roberto Nickson (RPN)** ও **Kallaway**-এর স্টাইলে প্রিমিয়াম ক্রিয়েটর-গ্রেড এডিট তৈরি করে — সম্পূর্ণভাবে কোডের মাধ্যমে, কোনো গ্রাফিক্যাল এডিটর ছাড়া। এটি একটি **9-স্টেপ, ফুলি-অটোমেটেড পাইপলাইন** যা:

- Raw footage থেকে শুরু করে publication-ready 9:16 ভার্টিক্যাল ভিডিও তৈরি করে।
- স্ক্রিপ্টের **প্রতিটি শব্দের** সাথে সিঙ্ক্রোনাইজড ভিজ্যুয়াল ইফেক্ট প্রয়োগ করে (sync map)।
- SRT ফাইল উপস্থিত থাকলে স্বয়ংক্রিয়ভাবে ট্রান্সক্রিপশন স্কিপ করে — সময় ও API কোটা বাঁচায়।
- বাংলা, ইংরেজি ও অন্যান্য ভাষার কনটেন্ট সমানভাবে সাপোর্ট করে।
- ক্যাপশন, কালার গ্রেড, জুম, সাউন্ড ডিজাইন ও মোশন গ্রাফিক্স — সব **একটি পাইপলাইনে** সম্পন্ন করে।
- **মাত্র একবার** থামে: প্রিভিউ অনুমোদনের জন্য।

এই স্কিলের প্রধান দর্শন: **"No timeline. Only code."** — প্রতিটি কাট, ট্রানজিশন, ইফেক্ট ও টাইমিং একটি নির্ভুল, পুনরাবৃত্তিযোগ্য কোডবেসে এনকোড করা থাকে।

---

## The 9-Step Pipeline (সম্পূর্ণ কর্মপ্রবাহ)

### STEP 1: TRANSCRIPTION & BEST TAKE SELECTION

#### SRT Policy (সর্বোচ্চ অগ্রাধিকার)

**যদি `srt/` ফোল্ডারে `.srt` ফাইল থাকে:**

1. প্রতিটি SRT ফাইলের নামকে ক্লিপের নামের সাথে মিলিয়ে দেখো।
   - উদাহরণ: `srt/take_3.srt` → ক্লিপ `take_3.mp4`
   - ফাইল নাম হুবহু না মিললে fuzzy match করো (prefix/suffix trim)।
2. মিলে যাওয়া SRT ফাইল থেকে **word-level timestamps** সরাসরি পড়ো।
3. `npx hyperframes transcribe` কমান্ড একদম কল করো না।
4. বাংলা বা অন্য non-ASCII টেক্সট সরাসরি SRT থেকে পড়ো — কোনো transliteration করো না।

**যদি `srt/` ফোল্ডার না থাকে বা ফোল্ডার খালি থাকে:**

```bash
npx hyperframes transcribe --input clips/ --output srt/ --word-level true --language auto
```

- `--word-level true` ফ্ল্যাগ অবশ্যই দাও, word-level timestamps ছাড়া sync map কাজ করবে না।
- `--language auto` দিলে ভাষা স্বয়ংক্রিয় শনাক্ত হয়; বাংলার জন্য `--language bn` ব্যবহার করো।

#### Best Take Selection Algorithm

```
FOR each line in script:
  1. সর্বশেষ (সবচেয়ে বড় টেক নম্বর) take দিয়ে শুরু করো।
  2. নিচের যেকোনো সমস্যা থাকলে → আগের টেকে যাও:
     a. stumble: কোনো শব্দ দুবার বলা হয়েছে বা থেমে থেমে বলা হয়েছে।
     b. clipped word: শব্দের শুরু বা শেষ কাটা গেছে।
     c. flat energy: monotone, উৎসাহ নেই — সর্বশেষ বিকল্প হিসেবে বিবেচনা করো।
  3. কোনো টেকেই সন্তোষজনক না হলে → সবচেয়ে কম সমস্যাযুক্ত টেকটি নাও।
```

#### takes.md ফরম্যাট

প্রতিটি নির্বাচিত লাইনের জন্য `takes.md` এ নিচের ফরম্যাটে লেখো:

```markdown
| Line | Script Text | Selected Take | Start | End | Reason |
|------|-------------|---------------|-------|-----|--------|
| 1 | "এটা সত্যিই কাজ করে" | take_3 | 00:12.4 | 00:15.1 | শেষ সম্পূর্ণ টেক, ভালো এনার্জি |
| 2 | "আমি অবাক হয়েছি" | take_2 | 00:08.7 | 00:11.2 | take_3 তে stumble ছিল |
```

- প্রতিটি সিদ্ধান্তের কারণ লেখা বাধ্যতামূলক।
- ফাইল পাথ পূর্ণ রাখো।

---

### STEP 2: TIGHT CUT

#### বিরতি কাটার নিয়ম

- **০.১৫ সেকেন্ডের বেশি** যেকোনো নীরবতা কাটো।
- শ্বাস-প্রশ্বাসের বিরতি (breath pause): ০.১৫s পর্যন্ত রাখো — এটা স্বাভাবিক শোনায়।
- একটি শব্দের মাঝখানে (mid-word) কাট করা **সম্পূর্ণ নিষেধ**।
- শব্দের শেষ থেকে পরের শব্দের শুরু পর্যন্ত গ্যাপ পরিমাপ করো।

#### 2-Frame Handle Rule

প্রতিটি কাটে:
- **In-point**: শব্দ শুরুর ঠিক **২ ফ্রেম আগে** (30fps = ~66ms)।
- **Out-point**: শব্দ শেষের ঠিক **২ ফ্রেম পরে**।
- এটি abrupt pop শব্দ এড়ায় এবং কাট স্মুথ রাখে।

#### ffmpeg Concat Script Template

```bash
# silence_remover.py দিয়ে প্রতিটি সেগমেন্ট নির্ধারণ করো
# তারপর ffmpeg concat demuxer দিয়ে জোড়া লাগাও:
ffmpeg -f concat -safe 0 -i segments.txt -c copy tight_cut.mp4
```

---

### STEP 3: COLOUR GRADE

#### গ্রেডিং প্যারামিটার (ffmpeg filtergraph)

```
curves=
  r='0/0 0.1/0.08 0.5/0.55 0.9/0.95 1/1':   ← S-curve (cinematic contrast)
  g='0/0 0.1/0.09 0.5/0.52 0.9/0.93 1/1':
  b='0/0 0.1/0.12 0.5/0.48 0.9/0.88 1/1',    ← cool shadows

colorbalance=
  rs=0.03:gs=-0.01:bs=-0.04:                  ← warm highlights (skintone)
  rm=0.01:gm=0:bm=-0.02:                      ← neutral mids
  rl=-0.02:gl=-0.01:bl=0.04,                  ← cool shadows

vignette=PI/5:                                 ← soft vignette (18% corner darkening)

noise=alls=3:allf=t+u                          ← film grain (temporal+uniform, strength 3)
```

#### Ungraded Copy সংরক্ষণ

```bash
cp tight_cut.mp4 output/ungraded.mp4
```

- এই কপি **STEP 6-এ** "colour grading" বিটে reveal করার জন্য ব্যবহার হবে।
- কখনো overwrite করো না।

---

### STEP 4: SMART ZOOMS

#### Face Tracking

```bash
npx hyperframes track-face \
  --input graded.mp4 \
  --output face_data.json \
  --target eyes \
  --smoothing 0.85
```

- `--target eyes`: চোখের কেন্দ্রবিন্দুতে ফোকাস রাখো।
- `--smoothing 0.85`: হঠাৎ ঝাঁকুনি এড়াতে উচ্চ স্মুথিং ব্যবহার করো।

#### Zoom Levels ও নিয়ম

| Condition | Zoom Level | Duration |
|-----------|-----------|----------|
| Normal talking head | 100% | Default |
| Key phrase / emphasis | 120% | ≥ 1.5s segment |
| Source is 1080p | Max 115% | — |
| Source is 4K | Max 130% | — |

- ১০০% ও ১২০% **alternate** করো — একটানা একই লেভেলে থাকো না।
- জুম ট্রানজিশন: `ease-in-out` curve, ০.৩s duration।
- ফেস সেন্টার থেকে ২০% এর বেশি সরে গেলে re-center করো।

#### ffmpeg Zoom Implementation

```bash
# face_data.json থেকে per-frame zoom ও pan ক্যালকুলেট করে
# zoompan filter দিয়ে প্রয়োগ করো:
ffmpeg -i graded.mp4 \
  -vf "zoompan=z='zoom_expr':x='x_expr':y='y_expr':d=1:s=1080x1920" \
  zoomed.mp4
```

---

### STEP 5: CAPTIONS & DESIGN

#### Caption Timing

- প্রতিটি ক্যাপশন গ্রুপে **২–৪ শব্দ** রাখো।
- প্রতিটি শব্দ তার নিজস্ব word-level timestamp অনুযায়ী **individually** হাইলাইট হবে।
- **Fade-in**: ৮০ms।
- **Fade-out**: পরবর্তী ক্যাপশনের শুরুর ঠিক আগে।

#### Design Spec

| Property | Value |
|----------|-------|
| Font | Montserrat ExtraBold / Noto Sans Bengali Bold |
| Font size | 72px (baseline) |
| Text color | #FFFFFF |
| Active keyword color | #D97757 (warm orange) |
| Background | Semi-transparent pill: rgba(0,0,0,0.45), border-radius 16px |
| Stroke | 3px #000000 (readability) |
| Shadow | 0px 4px 12px rgba(0,0,0,0.6) |

#### Safe Zone (9:16 = 1080×1920)

```
Top margin:    220px   ← ইন্সটাগ্রাম UI এড়াতে
Bottom margin: 420px   ← TikTok / Reels CTA বাটন এড়াতে
Left margin:   60px
Right margin:  140px   ← TikTok sidebar icons এড়াতে
```

**Caption placement**: ফ্রেমের উল্লম্ব কেন্দ্রের সামান্য নিচে (y ≈ 55% of height = ~1056px from top)। Safe zone-এর মধ্যে থাকা বাধ্যতামূলক।

#### ffmpeg Subtitle Overlay

```bash
ffmpeg -i zoomed.mp4 \
  -vf "subtitles=captions.ass:force_style='FontName=Montserrat,FontSize=72,...'" \
  captioned.mp4
```

- ASS ফরম্যাট ব্যবহার করো — SRT-এর চেয়ে বেশি স্টাইলিং কন্ট্রোল দেয়।
- প্রতিটি active word-এর জন্য আলাদা `\c&H5777D9&` (BGR: #D97757) কালার কোড সেট করো।

---

### STEP 6: SYNC MAP & AI DIRECTOR MODES (সবচেয়ে ক্রিটিক্যাল স্টেপ)

বাস্তব জীবনে কোনো ক্রিয়েটর ভিডিওতে মুখে "এখন কাট করো", "এখন জুম করো" বলে না। তাই এই স্কিলটি **দুটি শক্তিশালী মোডে** কাজ করে:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        SYNC MAP OPERATING MODES                        │
├─────────────────────────────────────────┬──────────────────────────────┤
│ MODE 1: Semantic Director (DEFAULT)     │ MODE 2: Meta / Demo Mode     │
│ ক্রিয়েটর স্বাভাবিক কথা বলবে। এআই স্ক্রিপ্ট ও   │ ক্রিয়েটর মুখে ইফেক্টের নাম বলে      │
│ কনটেক্সট বুঝে স্বয়ংক্রিয়ভাবে কাট, জুম ও       │ ডেমো রিল তৈরি করে (আর্টিকেলের        │
│ গ্রাফিক্স তৈরি করবে।                    │ সেলফ-রেফারেন্সিয়াল স্টাইল)।        │
└─────────────────────────────────────────┴──────────────────────────────┘
```

---

#### 🌟 MODE 1: Semantic Director Mode (বাস্তব কনটেন্টের জন্য ডিফল্ট)

এই মোডে বক্তা স্বাভাবিকভাবে যেকোনো টপিক নিয়ে কথা বলবে। এআই স্ক্রিপ্ট/SRT-এর অর্থ, বাক্য ও ব্যাকরণ বিশ্লেষণ (Semantic & Linguistic Analysis) করে একটি স্বয়ংক্রিয় `auto_sync_map.json` তৈরি করবে:

##### ১. বাক্য ও চিন্তা পরিবর্তন (Sentence & Thought Boundaries)
- **নিয়ম:** প্রতিটি বাক্য শেষ হলে বা নতুন ভাবনা শুরু হলে একটি স্বাভাবিক **জাম্প কাট** হবে।
- **ফ্রেমিং অল্টারনেশন:** জাম্প কাটের সাথে সাথে ক্যামেরা অ্যাঙ্গেল অল্টারনেট করবে: **Wide (100%) ↔ Close (120%)**। এতে কাটটি ইচ্ছাকৃত ও সিনেমাটিক মনে হয়।

##### ২. গুরুত্বপূর্ণ শব্দ ও জোর প্রদান (Emphasis & Key Takeaways)
- **নিয়ম:** যখন বক্তা কোনো গুরুত্বপূর্ণ তথ্য বা সতর্কবার্তা দেয় (যেমন: *"সবচেয়ে বড় কারণ"*, *"বিশাল পরিবর্তন"*, *"গোপন ট্রিক"*, *"Warning"*, *"Critical"*):
  - **ইফেক্ট:** ০.৪ সেকেন্ডের স্মুথ পুশ-ইন জুম (100% → 115%)।
  - **ক্যাপশন:** ক্যাপশনের ওই অ্যাক্টিভ শব্দটি অ্যাকসেন্ট রঙে (#D97757) বড় হবে।
  - **সাউন্ড:** হালকা সফট `whoosh.wav` SFX।

##### ৩. সংখ্যা ও পরিসংখ্যান ডিটেকশন (Numbers & Statistics)
- **নিয়ম:** স্ক্রিপ্টে যেকোনো সংখ্যা, শতাংশ বা পরিমাপ এলে (যেমন: *"৮০% মানুষ"*, *"১০ হাজার ডলার"*, *"৩টি স্টেপ"*, *"১৫ সেকেন্ড"*):
  - **ইফেক্ট:** এআই স্বয়ংক্রিয়ভাবে একটি বড় বোল্ড **কাউন্টার কার্ড বা কাইনেটিক টাইপোগ্রাফি** স্ক্রিনে আনবে।
  - **প্লেসমেন্ট:** সাবজেক্ট কাটআউট দিয়ে বক্তার মাথার পেছনে অথবা সেফ জোনে।
  - **সাউন্ড:** হালকা `pop.wav` বা `click.wav`।

##### ৪. টুল, ব্র্যান্ড ও প্রোডাক্ট মেনশন (Tool & Brand Triggers)
- **নিয়ম:** স্ক্রিপ্টে কোনো সফটওয়্যার, কোম্পানি বা পণ্যের নাম এলে (যেমন: *"Python"*, *"ChatGPT"*, *"Google"*, *"iPhone"*, ইত্যাদি):
  - **অ্যাকশন:** এআই `assets/images/` ফোল্ডার স্ক্যান করে সম্পর্কিত লোগো বা স্ক্রিনশট বের করবে।
  - **ডিসপ্লে:** একটি ১৬px রাউন্ডেড গ্লাস-মরফিজম কার্ডে ইমেজটি স্ক্রিনের একপাশে ০.৮ সেকেন্ডের জন্য প্রদর্শিত হবে।

##### ৫. কাজের দৃশ্য ও পরিস্থিতি বর্ণনা (B-roll Triggers)
- **নিয়ম:** বক্তা যখন কোনো ঘটনা, কাজের দৃশ্য বা উদাহরণের বর্ণনা দেয় (যেমন: *"আমি যখন কোড লিখছিলাম"*, *"বাজারে গিয়ে দেখলাম"*):
  - **অ্যাকশন:** মূল ক্যামেরা থেকে কাট করে `assets/broll/` ফোল্ডারের সংশ্লিষ্ট B-roll ফুটেজে চলে যাবে (১.৫ থেকে ৩.০ সেকেন্ডের জন্য)।
  - **গ্রেডিং:** B-roll-এও মূল ফুটেজের একই সিনেমাটিক S-curve গ্রেড স্বয়ংক্রিয়ভাবে বসে যাবে। মূল ভয়েস ব্যাকগ্রাউন্ডে নিরবচ্ছিন্ন চলবে।

##### ৬. অডিয়েন্স রিটেনশন রুল (The 3-4s Rule)
- **নিয়ম:** একটানা ৩ থেকে ৪ সেকেন্ড যদি স্ক্রিনে কোনো কাট, জুম বা গ্রাফিক্স না থাকে, দর্শককে ধরে রাখতে এআই স্বয়ংক্রিয়ভাবে একটি সূক্ষ্ম জাম্প কাট বা ১০০% ↔ ১১৫% ফ্রেমিং শিফট করবে।

##### ৭. দৃষ্টি সরানোর মুহূর্ত ঢাকা (Glance Detection)
- **নিয়ম:** ফেস-ট্র্যাকিংয়ের মাধ্যমে এআই বক্তার চোখের পিউপিল ট্র্যাক করবে। বক্তা ল্যাপটপ বা অন্য কোথাও তাকালে সেই ১–২ সেকেন্ড ফুল-স্ক্রিন মোশন গ্রাফিক্স বা B-roll দিয়ে ঢেকে দেওয়া হবে।

---

#### 🎬 MODE 2: Meta / Demo Reel Mode (সেলফ-রেফারেন্সিয়াল ডেমো রিলের জন্য)

যখন ক্রিয়েটর ইচ্ছাকৃতভাবে এডিটিং নিয়ে কোনো ডেমো ভিডিও বানায় (যেখানে মুখে ইফেক্টের নাম বলা হয়), তখন নিচের হার্ডকোডেড Sync Map কার্যকর হবে:

> **গুরুত্বপূর্ণ নিয়ম**: ২–৪ সেকেন্ড ব্যবধানে সবসময় কিছু একটা পরিবর্তন হবে। কিন্তু **একই সময়ে দুটো ইফেক্ট কখনো না** (একটি শেষ হওয়ার পর পরেরটি শুরু হবে)।

#### সম্পূর্ণ Beat-by-Beat Sync Map (Meta Mode)

---

**[ভিডিও শুরু → "captions" শব্দ পর্যন্ত]**
- ক্যাপশন: **বন্ধ** (caption layer invisible)।
- ফ্রেমের উপরে টাইটেল কার্ড: `EDITED 100% WITH CLAUDE CODE` — সাদা টেক্সট, কালো পিল ব্যাকগ্রাউন্ড, স্ক্রিনের উপরের ২২০px safe zone-এর মধ্যে।
- কোনো অন্য ওভারলে নেই।

---

**[কিওয়ার্ড: "cuts" / "cut"]**
- ভিডিও ইফেক্ট: **হার্ড জাম্প কাট** — কোনো ট্রানজিশন নেই, ফ্রেম-পারফেক্ট।
- অডিও SFX: **sharp click** (≈ 1kHz, 20ms, -12dB) — শব্দটি শোনার ঠিক ২ ফ্রেমের মধ্যে।
- জুম: পরিবর্তন নেই।

---

**[কিওয়ার্ড: "colour grading" / "color grade"]**
- ফ্রেজটি শুরু হওয়ার সাথে সাথে:
  1. **আনগ্রেডেড** ফুটেজ (`output/ungraded.mp4`) দেখাও।
  2. ০.৮s পরে: বাম থেকে ডানে **ভার্টিক্যাল লাইন সোয়াইপ** ট্রানজিশন।
  3. সোয়াইপের পরে: **গ্রেডেড** ফুটেজ দেখাও।
- ভার্টিক্যাল লাইন: ৪px সাদা, full-height, ease-in-out।
- পুরো ট্রানজিশন সময়: ০.৬s।

---

**[কিওয়ার্ড: "captions"]**
- ক্যাপশন লেয়ার: **চালু** (fade-in 80ms)।
- টাইটেল কার্ড (`EDITED 100% WITH CLAUDE CODE`): **ফেড আউট** (300ms)।
- এই দুটো ইফেক্ট sequential হবে: টাইটেল fade-out → তারপর caption fade-in।

---

**[কিওয়ার্ড: "zoom in" / "zoom ins"]**
- ইফেক্ট: **স্মুথ পুশ-ইন** — ease-in, ১.২s duration, ১০০% → ১২০%।
- অডিও SFX: **whoosh** (high-to-low sweep, ≈ 200–800Hz, ০.৪s, -10dB)।
- ফেস ট্র্যাকিং সক্রিয় থাকবে।

---

**[কিওয়ার্ড: "zoom out" / "zoom outs"]**
- ইফেক্ট: **পুল-আউট** — ease-out, ১.২s duration, ১২০% → ১০০%।
- অডিও SFX: **whoosh** (low-to-high sweep, ≈ 800–200Hz, ০.৪s, -10dB)।

---

**[কিওয়ার্ড: "motion graphics"]**
- ফ্রেজটি শুরু হওয়ার সাথে সাথে:
  1. **অরেঞ্জ বার** (#D97757) ফ্রেমের বাম প্রান্ত থেকে স্লাইড ইন করে (ease-out, ০.৩s)।
  2. বার সম্পূর্ণ হলে: `MOTION GRAPHICS` টেক্সট অক্ষর-অক্ষর করে বিল্ড হয় — প্রতিটি অক্ষর ০.০৫s ব্যবধানে (typewriter effect)।
- পুরো অ্যানিমেশন: ০.৮s।
- বার height: ৮px, full-width।

---

**[কিওয়ার্ড: "sound effects"]**
- ওভারলে: **মিনিমাল ওয়েভফর্ম পালস** — সাদা, ৩ লাইন, ফ্রেমের নিচের অংশে।
- পালস অ্যানিমেশন: scale Y oscillation (1.0 → 1.8 → 1.0), ০.৫s cycle, ২ cycle।
- কোনো টেক্সট ওভারলে নেই।

---

**[কিওয়ার্ড: "single editing app"]**
- কার্ড ওভারলে: মিনিমাল কার্ড — ডার্ক ব্যাকগ্রাউন্ড (rgba 0,0,0,0.8), rounded 24px।
- কার্ড টেক্সট:
  ```
  Editing apps opened: 0
  ```
- কাউন্টার টাইপড-অ্যানিমেশন: `0` স্থির (জোর দেওয়ার জন্য)।
- স্লাইড-ইন: ডান থেকে বামে, ০.৪s ease-out।
- Duration: ২s দৃশ্যমান, তারপর fade-out।

---

**[কিওয়ার্ড: "raw footage" থেকে "long pauses" পর্যন্ত]**
- ওভারলে: **রিয়েল ডাটা দিয়ে টাইমলাইন স্ট্রিপ** — ফ্রেমের নিচের অংশে।
- টাইমলাইন স্ট্রিপ ডিজাইন:
  - প্রতিটি টেক একটি সেগমেন্ট।
  - **ভুল/বাতিল টেক**: লাল রং (#FF4444)।
  - **নীরবতা / দীর্ঘ বিরতি**: ধূসর রং (#888888)।
  - **নির্বাচিত টেক**: সবুজ রং (#44FF88)।
  - রিয়েল টাইমকোড দেখাও প্রতিটি সেগমেন্টের নিচে।
- `takes.md` থেকে ডাটা পড়ে রিয়েল নম্বর দেখাও।

---

**[কিওয়ার্ড: "one prompt"]**
- ওভারলে: **টার্মিনাল কার্ড** — কালো ব্যাকগ্রাউন্ড, monospace font (Fira Code/Courier)।
- অ্যানিমেশন:
  1. প্রম্পটের প্রথম লাইন typewriter effect-এ টাইপ হয় (প্রতি অক্ষর ৩০ms)।
  2. কার্ডের নিচে: `Raw: Xs → Final: Ys` কাউন্টার — raw length থেকে final length-এ গণনা করে (২s duration)।
- কাউন্টারের X ও Y রিয়েল সেকেন্ড (STEP 9-এর রিপোর্ট থেকে)।

---

**[কিওয়ার্ড: "This is what I gave it"]**
- তিনটি একযোগে (কিন্তু sequential) ইফেক্ট:
  1. **আনগ্রেডেড রাও মোমেন্ট** দেখাও।
  2. উপরে ওভারলে: `RAW` ট্যাগ — লাল পিল (#FF4444), সাদা টেক্সট।
  3. নিচে ওভারলে: টাইমকোড — যে ক্লিপ থেকে নেওয়া তার মূল টাইমকোড।
- সব তিনটি: fade-in 150ms।

---

**[কিওয়ার্ড: "what it gave me back"]**
- ইফেক্ট সিকোয়েন্স:
  1. **Riser** অডিও: ০–১.৫s, pitch sweep up (-12dB থেকে -6dB)।
  2. ১.৫s পরে: **Wipe** ট্রানজিশন বাম-থেকে-ডানে (১.০s, ease-in-out)।
  3. Wipe শেষে: **Final edited** ভার্সন দেখাও।
- RAW ট্যাগ ও টাইমকোড ওভারলে: fade-out wipe শুরুর সাথে সাথে।

---

**[কিওয়ার্ড: "Make the captions bigger"]**
- ওভারলে: **টাইপড মেসেজ বাবল** — iMessage স্টাইল, ডার্ক ব্যাকগ্রাউন্ড।
- বাবলে টেক্সট typewriter effect: "Make the captions bigger"
- যখন "bigger" শব্দটি সম্পূর্ণ টাইপ হয়:
  - ক্যাপশন font-size: ৭২px → **১০০.৮px** (১৪০% scale)।
  - Scale অ্যানিমেশন: ০.৩s ease-out।
  - একটি "pop" sound effect (soft): ৪০ms, -15dB।

---

**[কিওয়ার্ড: "posted last year"]**
- ওভারলে: **পুরনো ভিডিও প্রিভিউ কার্ড** — `old/` ফোল্ডারের ভিডিওর প্রথম ফ্রেম।
- কার্ড ডিজাইন: rounded corners (24px), ৩২০×১৮০px, ফ্রেমের মাঝামাঝিতে।
- কার্ডের উপরে: `BEFORE` ট্যাগ — হলুদ পিল (#FFD700), কালো টেক্সট, Bold।
- Duration: ২s দৃশ্যমান।
- Fade-in: ২০০ms।

---

**[কিওয়ার্ড: "Claude's re-edit" / "Claude re-edit"]**
- ইফেক্ট সিকোয়েন্স:
  1. **Wipe** ট্রানজিশন: বাম থেকে ডান, ০.৮s।
  2. Wipe শেষে: **CLAUDE RE-EDIT** অরেঞ্জ ট্যাগ (#D97757) overlay করো।
  3. ট্যাগ অ্যানিমেশন: scale-in (০.৫ → ১.০), ০.৩s ease-out।
- BEFORE কার্ড: wipe শুরুর সাথে slide-out।

---

**[কিওয়ার্ড: "Comment CLAUDE" / "comment"]**
- দুটো sequential ইফেক্ট:
  1. **Kinetic Text**: "COMMENT" শব্দটি স্ক্রিনের উপর থেকে bounce করে নামে (ease-in → elastic), ০.৬s।
  2. তারপর: **কমেন্ট বাবল** ওভারলে — রিয়েল ইন্সটাগ্রাম/TikTok কমেন্ট বাবল UI।
     - বাবলে টেক্সট: "CLAUDE" (ইউজারনেম স্টাইলে)।
     - Slide-up from bottom: ০.৪s ease-out।
- CLAUDE RE-EDIT ট্যাগ: এই পয়েন্টে fade-out।

---

**[শেষ ফ্রেম = প্রথম ফ্রেম: SEAMLESS LOOP]**
- ভিডিওর শেষ ফ্রেম ভিজ্যুয়ালি প্রথম ফ্রেমের সাথে মিলতে হবে।
- বাস্তবায়ন:
  1. শেষ ২s: সব ওভারলে fade-out।
  2. ভিডিও ফুটেজ শেষ ফ্রেমে ফ্রিজ।
  3. Crossfade: শেষ ফ্রেম → প্রথম ফ্রেম (১s)।
  4. ffmpeg `-loop 1` দিয়ে loop point সেট করো।

---

### STEP 7: OLD VIDEO RE-EDIT

#### প্রক্রিয়া

```bash
# 1. old/ ফোল্ডার থেকে ভিডিও পড়ো
# 2. একই পাইপলাইন প্রয়োগ করো (Step 2-5)
# 3. আলাদা output-এ সেভ করো
ffmpeg -i old/original.mp4 [same grading + captions pipeline] output/old_reedit.mp4
```

- **Sync map প্রয়োজন নেই** এই ভিডিওর জন্য — শুধু tight cut + grade + captions।
- Original ভিডিওর aspect ratio 16:9 হলে: crop to 9:16 (face-centered)।
- `BEFORE` ও `CLAUDE RE-EDIT` overlay: STEP 6-এর "posted last year" বিটে ব্যবহারের জন্য।

---

### STEP 8: SOUND DESIGN

#### ভয়েস ক্লিনআপ

```bash
# ffmpeg + afftdn (adaptive FFT denoiser)
ffmpeg -i tight_cut.mp4 \
  -af "afftdn=nf=-25,equalizer=f=200:t=q:w=1:g=-3,compand=attacks=0.01:decays=0.5:points=-80/-80|-12/-12|-6/-6|0/-3:gain=2" \
  voice_clean.mp4
```

- **Noise reduction**: -25dB noise floor।
- **EQ**: ২০০Hz -3dB (মাডি লো-মিড কমাও)।
- **Compression**: 2:1 ratio, নরম অ্যাটাক।

#### Music Ducking

```bash
# সাইডচেইন কম্প্রেশন সিমুলেশন:
ffmpeg -i voice_clean.mp4 -i background_music.mp3 \
  -filter_complex "[1:a]volume=0.12[bg];[0:a][bg]amix=inputs=2:weights=10 1[out]" \
  -map "[out]" audio_mixed.mp4
```

- Background music: **১২% ভলিউম** (ভয়েস চলাকালীন)।
- SFX সিঙ্ক: প্রতিটি SFX-এর নির্দিষ্ট timestamp sync map থেকে নাও।

#### Mastering

```bash
ffmpeg -i audio_mixed.mp4 \
  -af "loudnorm=I=-14:TP=-1.5:LRA=11" \
  mastered.mp4
```

- **Target LUFS**: -14 LUFS (Instagram/TikTok/YouTube standard)।
- **True Peak**: -1.5 dBTP।
- **LRA**: ১১ LU।

---

### STEP 9: PREVIEW → FINAL RENDER

#### Preview Generation

```bash
ffmpeg -i mastered.mp4 \
  -vf "scale=540:960" \
  -crf 28 -preset fast \
  output/preview.mp4
```

- Preview: ৫৪০×৯৬০ (half resolution), ছোট ফাইল সাইজ।
- প্রতিটি sync map beat-এর স্টিল ইমেজ export করো: `output/stills/beat_XX.jpg`।

#### Self-Check Checklist

প্রিভিউ generate করার পর, ফাইনাল রেন্ডারের আগে নিচের প্রতিটি পয়েন্ট যাচাই করো:

- [ ] **Sync accuracy**: প্রতিটি ইফেক্ট কি শব্দ শুরুর ২ ফ্রেমের মধ্যে?
- [ ] **Face overlap**: ক্যাপশন বা কার্ড কি ফেসের উপর পড়ছে?
- [ ] **Safe zone**: সব টেক্সট কি safe zone-এর মধ্যে?
- [ ] **Cuts**: কোনো mid-word cut আছে কি? (থাকলে ফিক্স করো)
- [ ] **Audio peaks**: কোনো peak কি -1.5 dBTP ছাড়িয়েছে?
- [ ] **Loop**: শেষ ফ্রেম কি প্রথম ফ্রেমের সাথে মিলছে?
- [ ] **Duration**: ইন্টারেস্ট ধরে রাখার জন্য ৬০s-এর মধ্যে আছে কি?

#### ⛔ ONE STOP: এখানে থামো

প্রিভিউ ও stills ইউজারকে দেখাও। **অনুমোদন না পাওয়া পর্যন্ত ফাইনাল রেন্ডার করো না।**

#### Final Render

অনুমোদনের পরে:

```bash
ffmpeg -i mastered.mp4 \
  -c:v libx264 -preset slow -crf 18 \
  -b:v 18M -maxrate 22M -bufsize 44M \
  -c:a aac -b:a 320k \
  -movflags +faststart \
  -s 1080x1920 \
  output/final.mp4
```

| Parameter | Value | কারণ |
|-----------|-------|-------|
| Codec | H.264 (libx264) | সর্বজনীন সামঞ্জস্যতা |
| Resolution | 1080×1920 | 9:16 vertical |
| Video bitrate | ~18 Mbps | Instagram/TikTok max quality |
| Audio codec | AAC | সার্বজনীন |
| Audio bitrate | 320 kbps | Lossless-near quality |
| faststart | হ্যাঁ | Progressive streaming |

#### Final Report

রেন্ডার শেষে কনসোলে রিয়েল ডাটা রিপোর্ট করো:

```
=== AI VIDEO DIRECTOR — FINAL REPORT ===
Raw footage length:      4m 32s (272s)
Final video length:      58s
Total pauses removed:    47 (cumulative: 1m 18s)
Takes rejected:          3
Best takes used:         12 segments
Output file:             output/final.mp4 (142 MB)
Output stills:           output/stills/ (18 images)
Old re-edit:             output/old_reedit.mp4 (38 MB)
Mastered LUFS:           -13.9 LUFS
========================================
```

---

## SRT File Priority (CRITICAL)

> ⚠️ **srt/ ফোল্ডারে যেকোনো `.srt` ফাইল থাকলে ট্রান্সক্রিপশন কমান্ড চালানো সম্পূর্ণ নিষেধ।**

### SRT ব্যবহারের সম্পূর্ণ নিয়ম

#### নিয়ম ১: ম্যাচিং লজিক

```python
import os, difflib

clip_name = "take_3"          # ক্লিপ ফাইলের নাম (extension বাদ)
srt_files = os.listdir("srt/")

# Exact match প্রথমে চেষ্টা করো
if f"{clip_name}.srt" in srt_files:
    srt_path = f"srt/{clip_name}.srt"
else:
    # Fuzzy match
    best = difflib.get_close_matches(clip_name, srt_files, n=1, cutoff=0.6)
    srt_path = f"srt/{best[0]}" if best else None

if srt_path:
    print(f"✅ SRT found: {srt_path} — transcription SKIPPED")
else:
    print("⚠️ No SRT found — running transcription")
    os.system("npx hyperframes transcribe ...")
```

#### নিয়ম ২: SRT থেকে Word-Level Timestamps পড়া

SRT ফাইলের উদাহরণ (word-level):

```srt
1
00:00:01,240 --> 00:00:01,480
এটা

2
00:00:01,480 --> 00:00:01,920
সত্যিই

3
00:00:01,920 --> 00:00:02,680
কাজ

4
00:00:02,680 --> 00:00:03,100
করে
```

Python পার্সিং:

```python
import re

def parse_srt_words(srt_path):
    words = []
    with open(srt_path, encoding='utf-8') as f:
        content = f.read()
    blocks = content.strip().split('\n\n')
    for block in blocks:
        lines = block.strip().split('\n')
        if len(lines) >= 3:
            time_match = re.match(
                r'(\d{2}:\d{2}:\d{2},\d{3}) --> (\d{2}:\d{2}:\d{2},\d{3})',
                lines[1]
            )
            if time_match:
                start = srt_time_to_seconds(time_match.group(1))
                end = srt_time_to_seconds(time_match.group(2))
                text = ' '.join(lines[2:])
                words.append({'word': text, 'start': start, 'end': end})
    return words
```

#### নিয়ম ৩: বাংলা টেক্সট অটোমেটিক হ্যান্ডেলিং

- বাংলা ইউনিকোড সরাসরি ব্যবহার করো।
- `utf-8` এনকোডিং সবখানে নিশ্চিত করো।
- Transliteration (বাংলা → Latin) কখনো করো না।
- **Noto Sans Bengali** ফন্ট ব্যবহার করো ASS/SRT subtitle-এ।

#### বিস্তারিত উদাহরণ

**ফোল্ডার স্ট্রাকচার:**
```
project/
├── clips/
│   ├── take_1.mp4
│   ├── take_2.mp4
│   └── take_3.mp4
├── srt/
│   ├── take_1.srt       ← exact match: take_1.mp4
│   └── take_3.srt       ← exact match: take_3.mp4
│   # take_2.srt নেই → take_2.mp4 ট্রান্সক্রাইব হবে
├── images/
├── broll/
└── old/
```

**পাইপলাইন লজিক:**
```
take_1.mp4 → srt/take_1.srt ✅ (no transcription)
take_2.mp4 → srt/take_2.srt ❌ not found → transcribe take_2.mp4only
take_3.mp4 → srt/take_3.srt ✅ (no transcription)
```

---

## Assets Integration (images/ ও broll/ ফোল্ডার)

### images/ ফোল্ডার

| ফাইল টাইপ | ব্যবহার |
|-----------|---------|
| লোগো (PNG, transparent) | Motion graphics ব্যাকগ্রাউন্ড বা watermark |
| স্ক্রিনশট | "This is what I gave it" বিটে overlay |
| স্ট্যাটিক গ্রাফিক্স | কার্ড ব্যাকগ্রাউন্ড বা ইলাস্ট্রেশন |
| চার্ট/ডায়াগ্রাম | "sound effects" বা "motion graphics" বিটে |

**নামকরণ কনভেনশন:**
- `images/logo.png` → ওয়াটারমার্ক হিসেবে।
- `images/screenshot_prompt.png` → "one prompt" বিটে।
- `images/chart_stats.png` → "motion graphics" বিটে।

**ffmpeg overlay:**
```bash
ffmpeg -i main_video.mp4 -i images/logo.png \
  -filter_complex "[0:v][1:v]overlay=W-w-20:20:enable='between(t,0,5)'" \
  output.mp4
```

### broll/ ফোল্ডার

| ফাইল নামকরণ | ব্যবহার |
|------------|---------|
| `broll/typing.mp4` | "one prompt" বিটে কাটঅ্যাওয়ে |
| `broll/phone_scroll.mp4` | "posted last year" বিটে |
| `broll/screen_record.mp4` | "colour grading" reveal-এ |

**ম্যাচিং নিয়ম:**
1. **নাম দিয়ে ম্যাচিং**: sync map-এ উল্লেখিত কিওয়ার্ডের সাথে broll ফাইল নামের মিল দেখো।
2. **Sync map-এ explicit উল্লেখ**: `broll/typing.mp4 at "one prompt" beat` লিখলে সেটি অগ্রাধিকার পাবে।
3. **Duration matching**: broll ক্লিপ যদি beat-এর চেয়ে ছোট হয়, loop করো; বড় হলে trim করো।

**broll insert:**
```bash
# "one prompt" beat-এ broll/typing.mp4 কাট করো
ffmpeg -i main_video.mp4 -i broll/typing.mp4 \
  -filter_complex "[0:v]trim=start=15.2:end=18.7[main];[1:v]trim=end=3.5[broll];[main][broll]concat" \
  output.mp4
```

---

## One Stop Policy

এই পাইপলাইনে **মাত্র একটি** user confirmation point আছে:

```
✅ Step 1  → চালিয়ে যাও
✅ Step 2  → চালিয়ে যাও
✅ Step 3  → চালিয়ে যাও
✅ Step 4  → চালিয়ে যাও
✅ Step 5  → চালিয়ে যাও
✅ Step 6  → চালিয়ে যাও
✅ Step 7  → চালিয়ে যাও
✅ Step 8  → চালিয়ে যাও
⛔ Step 9 (Preview) → STOP — ইউজারের অনুমোদন নাও
✅ Step 9 (Final Render) → অনুমোদনের পরে চালাও
```

**কোনো intermittent question জিজ্ঞেস করো না।** যদি কোনো অস্পষ্টতা থাকে (যেমন কোন টেক বেছে নেবে), takes.md-তে সিদ্ধান্ত লেখো এবং এগিয়ে যাও। প্রিভিউতে explanation দাও।

---

## Sync Accuracy Rule

### অডিও ওয়েভফর্ম থেকে কাট মাপো

ট্রান্সক্রিপ্টের timestamp প্রায়ই **১/৩ সেকেন্ড (≈ ৩৩৩ms)** দেরিতে বা আগে হতে পারে। সঠিক কাট পেতে:

```bash
# ffmpeg দিয়ে ওয়েভফর্ম ডাটা বের করো
ffmpeg -i audio.mp3 -af "silencedetect=n=-40dB:d=0.15" -f null - 2>&1 | grep silence
```

- `n=-40dB`: এই লেভেলের নিচে গেলে নীরবতা ধরো।
- `d=0.15`: ০.১৫s-এর কম নীরবতা ignore করো।

### 2-Frame Rule (পুনরাবৃত্তি)

```
Beat trigger timestamp (from transcript) = T
Actual effect start = T - 66ms (2 frames at 30fps)
Acceptable range = T ± 66ms
```

যদি ওয়েভফর্ম দেখে মনে হয় transcript timestamp ভুল:
- ওয়েভফর্ম থেকে প্রাপ্ত timestamp ব্যবহার করো।
- takes.md-তে note করো: "Timestamp adjusted from Xs to Ys based on waveform analysis"।
