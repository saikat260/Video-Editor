---
name: video-project-scaffolder
description: |
  AI ভিডিও এডিটিং প্রজেক্টের জন্য সম্পূর্ণ ফাইল ও ডিরেক্টরি স্ট্রাকচার স্বয়ংক্রিয়ভাবে তৈরি করে।
  Claude Code + HyperFrames + ffmpeg পাইপলাইনের জন্য প্রয়োজনীয় সমস্ত ফোল্ডার, টেমপ্লেট ফাইল,
  কনফিগারেশন এবং গাইড ফাইল জেনারেট করে। পরিবেশ (Node.js, ffmpeg, HyperFrames) যাচাই করে।
version: "2.0"
author: AI Systems Lab
triggers:
  - "নতুন ভিডিও প্রজেক্ট শুরু করো"
  - "প্রজেক্ট ফোল্ডার বানাও"
  - "set up my video project"
  - "create project structure"
  - "scaffold video project"
  - "initialize project"
requires:
  - Node.js >= 22
  - ffmpeg
  - npx hyperframes (via npm)
tags:
  - setup
  - scaffolding
  - project-init
  - video-editing
  - ffmpeg
  - hyperframes
---

## Overview

### এই স্কিলটি কী করে?

`video-project-scaffolder` স্কিলটি Claude Code-এর মাধ্যমে একটি সম্পূর্ণ, প্রোডাকশন-রেডি AI ভিডিও এডিটিং প্রজেক্টের কাঠামো (scaffold) স্বয়ংক্রিয়ভাবে তৈরি করে। একটি মাত্র কমান্ডে নিচের সব কিছু হয়ে যায়:

- **সমস্ত প্রয়োজনীয় ডিরেক্টরি** তৈরি হয় (raw, srt, assets, output ইত্যাদি)
- **টেমপ্লেট ফাইল** জেনারেট হয় (config.json, script.txt, recording-guide.md, takes.md)
- **পরিবেশ যাচাই** হয় — Node.js ≥22, ffmpeg, এবং HyperFrames ইনস্টলড আছে কিনা
- মিসিং টুলের জন্য **ইনস্টল নির্দেশনা** দেওয়া হয় এবং প্রজেক্ট সেটআপ থামিয়ে দেওয়া হয়

### কেন এই স্কিল দরকার?

Claude Code + HyperFrames + ffmpeg পাইপলাইনটি নির্দিষ্ট একটি ফোল্ডার কনভেনশন অনুসরণ করে। প্রতিটি নতুন ভিডিও এডিটিং সেশনে হাতে হাতে ফোল্ডার বানানো, JSON কনফিগ লেখা, এবং গাইড ফাইল তৈরি করা সময়সাপেক্ষ ও ভুলপ্রবণ। এই স্কিল সেই পুরো প্রক্রিয়াকে স্বয়ংক্রিয় করে, ফলে ব্যবহারকারী সরাসরি কন্টেন্ট তৈরিতে মনোযোগ দিতে পারেন।

### কখন ব্যবহার করতে হবে?

- যখনই একটি নতুন ভিডিও এডিটিং প্রজেক্ট শুরু করতে হবে
- যখন একটি নতুন Instagram Reel, TikTok ভিডিও, YouTube Short বা অন্য কোনো শর্ট-ফর্ম কন্টেন্ট তৈরির পরিকল্পনা করা হচ্ছে
- যখন কোনো ক্লায়েন্ট বা ব্র্যান্ডের জন্য ভিডিও প্রজেক্ট শুরু করতে হবে
- যখন পুরনো ওয়ার্কফ্লো রিসেট করে পরিষ্কার স্ট্রাকচার থেকে নতুনভাবে শুরু করতে হবে

---

## Prerequisites (পূর্বশর্ত)

এই স্কিল চালু করার আগে নিচের তিনটি টুল তোমার সিস্টেমে ইনস্টলড থাকতে হবে। স্কিলটি নিজেই এগুলো চেক করবে, কিন্তু আগে থেকে ইনস্টল করে রাখলে ভালো।

### 1. Node.js (version 22 বা তার বেশি)

Node.js না থাকলে অথবা পুরনো ভার্সন থাকলে HyperFrames কাজ করবে না।

**macOS (Homebrew):**
```bash
brew install node@22
# অথবা nvm দিয়ে:
nvm install 22
nvm use 22
```

**Windows (winget):**
```powershell
winget install OpenJS.NodeJS.LTS
# অথবা nvm-windows দিয়ে:
nvm install 22
nvm use 22
```

**Linux (Ubuntu/Debian):**
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**ভার্সন যাচাই:**
```bash
node --version   # আউটপুট হওয়া উচিত: v22.x.x বা তার বেশি
npm --version    # আউটপুট হওয়া উচিত: 10.x.x বা তার বেশি
```

---

### 2. ffmpeg

ffmpeg ছাড়া ভিডিও এনকোড, ট্রান্সকোড, অডিও মিক্সিং কিছুই হবে না।

**macOS (Homebrew):**
```bash
brew install ffmpeg
```

**Windows (winget):**
```powershell
winget install Gyan.FFmpeg
# অথবা Chocolatey দিয়ে:
choco install ffmpeg
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get update
sudo apt-get install -y ffmpeg
```

**ভার্সন যাচাই:**
```bash
ffmpeg -version   # প্রথম লাইনে ffmpeg version দেখাবে
```

---

### 3. HyperFrames (npx দিয়ে)

HyperFrames npm-এ পাওয়া যায় এবং `npx` দিয়ে ব্যবহার করা হয়।

**গ্লোবাল ইনস্টল (ঐচ্ছিক কিন্তু দ্রুত):**
```bash
npm install -g hyperframes
```

**npx দিয়ে (ইনস্টল ছাড়া, সরাসরি রান):**
```bash
npx hyperframes --version
```

**ভার্সন যাচাই:**
```bash
npx hyperframes --version   # ভার্সন নম্বর দেখাবে
```

---

## Directory Structure (তৈরি হওয়া ফোল্ডার স্ট্রাকচার)

স্কিলটি নিচের সম্পূর্ণ স্ট্রাকচার তৈরি করে। প্রতিটি ফোল্ডার ও ফাইলের উদ্দেশ্য বিস্তারিতভাবে ব্যাখ্যা করা হয়েছে:

```
my-edit/
├── raw/                    # সব raw ভিডিও টেক (রেকর্ডিং অর্ডারে)
├── srt/                    # Pre-existing SRT/subtitle ফাইল (বাংলাসহ যেকোনো ভাষা)
├── script.txt              # ফাইনাল স্ক্রিপ্ট (optional কিন্তু recommended)
├── old/                    # আগের পুরনো রিল/ভিডিও (before/after এর জন্য)
├── assets/
│   ├── music/              # ব্যাকগ্রাউন্ড মিউজিক ট্র্যাক
│   ├── sfx/                # সাউন্ড ইফেক্ট (whoosh, pop, click, riser, swoosh)
│   ├── fonts/              # ক্যাপশন ফন্ট (.woff2/.ttf/.otf)
│   ├── images/             # ব্র্যান্ড লোগো, স্ট্যাটিক গ্রাফিক্স, বিজনেস কার্ড ইমেজ,
│   │                       # পণ্যের ছবি, স্ক্রিনশট ইত্যাদি
│   └── broll/              # B-roll ভিডিও ফুটেজ (সাপোর্টিং ফুটেজ, কাটওয়ে শট)
├── takes.md                # AI-জেনারেটেড টেক সিলেকশন লগ
├── config.json             # প্রজেক্ট কনফিগারেশন
├── recording-guide.md      # রেকর্ডিং গোল্ডেন রুলস
└── output/                 # ফাইনাল রেন্ডার ফাইল
    ├── preview/            # প্রিভিউ ও বিট স্টিলস
    └── final.mp4           # (প্লেসহোল্ডার — রেন্ডারের পরে তৈরি হবে)
```

### প্রতিটি ফোল্ডারের বিস্তারিত বিবরণ

| ফোল্ডার/ফাইল | উদ্দেশ্য | কী রাখতে হয় |
|---|---|---|
| `raw/` | সব raw রেকর্ডিং | `.mp4`, `.mov`, `.mkv` — রেকর্ডিং অর্ডারে নামকরণ করো (`take_001.mp4`, `take_002.mp4`) |
| `srt/` | সাবটাইটেল/ক্যাপশন ফাইল | `.srt` ফরম্যাটের ট্রান্সক্রিপশন ফাইল |
| `script.txt` | ভিডিওর স্ক্রিপ্ট | Claude-এর Sync Map হিসেবে ব্যবহৃত হয় |
| `old/` | পুরনো ভিডিও রেফারেন্স | before/after তুলনার জন্য আগের ভিডিও |
| `assets/music/` | ব্যাকগ্রাউন্ড মিউজিক | `.mp3`, `.wav`, `.aac` ট্র্যাক |
| `assets/sfx/` | সাউন্ড ইফেক্ট | whoosh, pop, click, riser, swoosh ইত্যাদি |
| `assets/fonts/` | ক্যাপশন ফন্ট | `.woff2`, `.ttf`, `.otf` ফরম্যাট |
| `assets/images/` | স্ট্যাটিক গ্রাফিক্স | লোগো, বিজনেস কার্ড, প্রোডাক্ট ছবি, স্ক্রিনশট |
| `assets/broll/` | সাপোর্টিং ভিডিও ফুটেজ | কাটওয়ে শট, B-roll ভিডিও ক্লিপ |
| `takes.md` | টেক সিলেকশন লগ | Claude AI-জেনারেটেড, কোন টেক ব্যবহার হয়েছে তার রেকর্ড |
| `config.json` | প্রজেক্ট কনফিগারেশন | রেজোলিউশন, FPS, কালার, অডিও সেটিংস |
| `recording-guide.md` | রেকর্ডিং গাইড | ৩টি গোল্ডেন রুল বিস্তারিতভাবে |
| `output/` | রেন্ডার আউটপুট | ফাইনাল ভিডিও এখানে সেভ হয় |
| `output/preview/` | প্রিভিউ ফাইল | লো-রেজ প্রিভিউ, বিট স্টিলস, থাম্বনেইল |

---

## Environment Check (পরিবেশ যাচাই)

স্কিলটি চালু হওয়ার সাথে সাথে, প্রজেক্ট ফোল্ডার তৈরি করার আগে, নিচের পরিবেশ যাচাই পদক্ষেপগুলো সম্পন্ন করতে হবে। কোনো একটি চেকে ব্যর্থ হলে সেটআপ থামিয়ে ইনস্টলেশন নির্দেশনা দেখাতে হবে।

### চেক ১: Node.js ভার্সন

```bash
node --version
```

**প্রত্যাশিত আউটপুট:** `v22.x.x` বা তার বেশি

**ব্যর্থ হলে করণীয়:**
```
❌ Node.js পাওয়া যায়নি বা ভার্সন 22-এর কম।
   ইনস্টল করো: https://nodejs.org/en/download
   অথবা: nvm install 22 && nvm use 22
   সেটআপ বন্ধ করা হচ্ছে।
```

### চেক ২: ffmpeg উপস্থিতি

```bash
ffmpeg -version
```

**প্রত্যাশিত আউটপুট:** `ffmpeg version X.X.X` দিয়ে শুরু হওয়া লাইন

**ব্যর্থ হলে করণীয়:**
```
❌ ffmpeg পাওয়া যায়নি।
   macOS:   brew install ffmpeg
   Windows: winget install Gyan.FFmpeg
   Linux:   sudo apt-get install ffmpeg
   সেটআপ বন্ধ করা হচ্ছে।
```

### চেক ৩: HyperFrames উপস্থিতি

```bash
npx hyperframes --version
```

**প্রত্যাশিত আউটপুট:** HyperFrames ভার্সন নম্বর

**ব্যর্থ হলে করণীয়:**
```
❌ HyperFrames পাওয়া যায়নি।
   ইনস্টল করো: npm install -g hyperframes
   অথবা npx দিয়ে সরাসরি ব্যবহার করো।
   সেটআপ বন্ধ করা হচ্ছে।
```

### সব চেক পাস হলে

```
✅ Node.js v22.x.x — OK
✅ ffmpeg 6.x.x — OK
✅ HyperFrames 1.x.x — OK
🚀 প্রজেক্ট স্ট্রাকচার তৈরি শুরু হচ্ছে...
```

---

## Template Files (জেনারেটেড ফাইলের বিষয়বস্তু)

স্কিলটি নিচের টেমপ্লেট কন্টেন্ট দিয়ে প্রতিটি ফাইল তৈরি করে।

### `config.json` টেমপ্লেট

এই ফাইলটি পুরো প্রজেক্টের "কন্ট্রোল প্যানেল"। HyperFrames ও Claude এখান থেকে সব সেটিংস পড়ে।

```json
{
  "project": {
    "name": "my-edit",
    "created": "AUTO_DATE",
    "platform": "instagram_tiktok",
    "aspect_ratio": "9:16",
    "resolution": "1080x1920",
    "target_fps": "source",
    "language": "bn"
  },
  "palette": {
    "primary": "#FFFFFF",
    "dark": "#141413",
    "accent": "#D97757",
    "easing": "expo-out"
  },
  "safe_zone": {
    "top_px": 220,
    "bottom_px": 420,
    "right_px": 140
  },
  "caption": {
    "words_per_screen": "2-4",
    "font_size_1080": 68,
    "fade_ms": 80,
    "position": "lower_middle"
  },
  "cuts": {
    "silence_threshold_sec": 0.15,
    "handle_frames": 2,
    "zoom_wide_pct": 100,
    "zoom_close_pct": 120,
    "zoom_cap_1080p_pct": 115
  },
  "audio": {
    "target_lufs": -14,
    "peak_dbtp": -1,
    "highpass_hz": 80,
    "music_duck_db": -20
  },
  "export": {
    "codec": "h264",
    "bitrate_mbps": 18,
    "audio_codec": "aac",
    "audio_bitrate": "320k"
  }
}
```

**`config.json` ফিল্ড ব্যাখ্যা:**

| সেকশন | ফিল্ড | বিবরণ |
|---|---|---|
| `project` | `name` | প্রজেক্টের নাম (ফোল্ডার নামের সাথে মিলবে) |
| `project` | `created` | স্বয়ংক্রিয়ভাবে তৈরির তারিখ বসবে |
| `project` | `platform` | টার্গেট প্ল্যাটফর্ম |
| `project` | `aspect_ratio` | ভিডিওর অ্যাসপেক্ট রেশিও |
| `project` | `resolution` | আউটপুট রেজোলিউশন |
| `project` | `target_fps` | `"source"` মানে ইনপুট FPS রাখবে |
| `project` | `language` | ক্যাপশন ভাষা কোড (`bn` = বাংলা) |
| `palette` | `primary` | মূল টেক্সট/UI রঙ |
| `palette` | `dark` | ডার্ক ব্যাকগ্রাউন্ড রঙ |
| `palette` | `accent` | হাইলাইট/অ্যাকসেন্ট রঙ |
| `palette` | `easing` | অ্যানিমেশন ইজিং ফাংশন |
| `safe_zone` | `top_px` | উপরের UI-মুক্ত এলাকা (পিক্সেলে) |
| `safe_zone` | `bottom_px` | নিচের UI-মুক্ত এলাকা (পিক্সেলে) |
| `safe_zone` | `right_px` | ডানের UI-মুক্ত এলাকা (পিক্সেলে) |
| `caption` | `words_per_screen` | প্রতি স্ক্রিনে শব্দের সংখ্যা |
| `caption` | `font_size_1080` | 1080px রেজোলিউশনে ফন্ট সাইজ |
| `caption` | `fade_ms` | ক্যাপশন ফেড ইন/আউট সময় (মিলিসেকেন্ড) |
| `caption` | `position` | ক্যাপশনের অবস্থান |
| `cuts` | `silence_threshold_sec` | এতটুকু নীরবতা কাটা যাবে |
| `cuts` | `handle_frames` | কাটের আগে/পরে রাখা ফ্রেম সংখ্যা |
| `cuts` | `zoom_wide_pct` | ওয়াইড শটে জুম শতাংশ |
| `cuts` | `zoom_close_pct` | ক্লোজ শটে জুম শতাংশ |
| `cuts` | `zoom_cap_1080p_pct` | 1080p-তে সর্বোচ্চ জুম |
| `audio` | `target_lufs` | লাউডনেস টার্গেট (LUFS) |
| `audio` | `peak_dbtp` | পিক লিমিট (dBTP) |
| `audio` | `highpass_hz` | হাই-পাস ফিল্টার কাটঅফ |
| `audio` | `music_duck_db` | কথা থাকলে মিউজিক কমানোর পরিমাণ |
| `export` | `codec` | ভিডিও এনকোডার |
| `export` | `bitrate_mbps` | ভিডিও বিটরেট (Mbps) |
| `export` | `audio_codec` | অডিও এনকোডার |
| `export` | `audio_bitrate` | অডিও বিটরেট |

---

### `script.txt` টেমপ্লেট

```
# এখানে তোমার ফাইনাল স্ক্রিপ্ট লেখো
# প্রতিটি লাইন = স্ক্রিনে একটি আলাদা কথা
# ইফেক্টের নাম মুখে বলো (cuts, colour grading, zoom ins, zoom outs,
# captions, motion graphics, sound effects ইত্যাদি)
# Claude এই স্ক্রিপ্টকে Sync Map হিসেবে ব্যবহার করবে

[তোমার স্ক্রিপ্ট এখানে লেখো]
```

**গুরুত্বপূর্ণ নির্দেশনা:**
- প্রতিটি লাইন ভিডিওর একটি আলাদা বাক্য বা বিরতি প্রতিনিধিত্ব করে
- ইফেক্টের নাম সরাসরি স্ক্রিপ্টে লেখো (যেমন: `[zoom in]`, `[colour grade - warm]`, `[sfx: whoosh]`)
- Claude এই ফাইলকে Sync Map হিসেবে ব্যবহার করে raw ফুটেজের সাথে মেলায়

---

### `recording-guide.md` টেমপ্লেট

```markdown
# রেকর্ডিং গোল্ডেন রুলস 🎬

ভালো ভিডিও রেকর্ড করার জন্য মাত্র ৩টি নিয়ম মনে রাখো।

---

## নিয়ম ১: Repeat (পুনরাবৃত্তি) 🔁

**ভুল হলে ক্যামেরা বন্ধ করো না।**

রেকর্ডিং করার সময় যদি ভুল বলো বা জড়িয়ে যাও, সাথে সাথে ক্যামেরা বন্ধ করার দরকার নেই।
একটু থামো, ৩ সেকেন্ড নীরব থাকো, তারপর **একই লাইন আবার বলো** যেন কিছুই হয়নি।

Claude সেই নীরবতার গ্যাপ খুঁজে বের করে শেষ সবচেয়ে ভালো টেকটি স্বয়ংক্রিয়ভাবে নেবে।
এতে এডিটিং সেশনে একটু বাড়তি ফুটেজ আসবে, কিন্তু ফলাফল অনেক বেশি মসৃণ হবে।

**কী করবে:** ভুল → বিরতি → আবার বলো → এগিয়ে যাও
**কী করবে না:** ভুল → থামো → কাটো → আবার শুরু করো

---

## নিয়ম ২: Name It (নাম বলো) 🏷️

**প্রতিটি ইফেক্ট মুখে স্পষ্ট করে বলো।**

যদি চাও কোথাও zoom in হোক, কথায় কথায় বলো: *"এখানে zoom in করো"* বা *"এবার colour grade করো warm।"*
Claude স্ক্রিপ্টের সাথে মিলিয়ে সেই জায়গায় ঠিক সেই ইফেক্টটি বসাবে।

সাপোর্টেড ইফেক্ট কমান্ড:
- `zoom in` / `zoom out`
- `cut here` / `jump cut`
- `colour grade: warm` / `colour grade: cool` / `colour grade: cinematic`
- `sfx: whoosh` / `sfx: pop` / `sfx: riser`
- `caption on` / `caption off`
- `b-roll: [বিষয়]`
- `motion graphic: [বিষয়]`
- `music: fade in` / `music: fade out`

---

## নিয়ম ৩: Glance (স্ক্রিনে তাকাও) 👀

**ল্যাপটপে তাকালে কোনো সমস্যা নেই।**

রেকর্ড করার সময় যদি স্ক্রিপ্ট পড়ার জন্য ল্যাপটপে তাকাতে হয়, তাহলে নিশ্চিন্তে তাকাও।
Claude সেই মুহূর্তে ফুল-স্ক্রিন গ্রাফিক্স, B-roll ফুটেজ বা ওভারলে দিয়ে সেই অংশ ঢেকে দেবে।

দর্শক কখনো বুঝতে পারবে না যে তুমি স্ক্রিন দেখছিলে।
এটি একটি প্রো-লেভেল টেকনিক।
```

---

### `takes.md` টেমপ্লেট

```markdown
# Takes Selection Log

**Project:** my-edit
**Generated:** AUTO_DATE
**Status:** Pending Analysis

---

## Raw Takes Inventory

Claude এই ফাইলটি `raw/` ফোল্ডার বিশ্লেষণ করার পরে পূরণ করবে।

| Take | File | Duration | Quality Score | Selected | Notes |
|------|------|----------|---------------|----------|-------|
| — | — | — | — | — | analysis pending |

---

## Selection Summary

*(Claude দ্বারা পূরণ করা হবে)*
```

---

## SRT Folder Policy (বাংলা ও মাল্টিলিঙ্গুয়াল সাপোর্ট)

> [!CAUTION]
> **CRITICAL RULE:** `srt/` ফোল্ডারে যদি একটিও `.srt` ফাইল থাকে, তাহলে Claude **কোনো ক্লিপ ট্রান্সক্রাইব করবে না।** এই নিয়মটি ওভাররাইড করা যাবে না।

### কেন এই নিয়ম?

বাংলাসহ অনেক ভাষার ক্ষেত্রে স্বয়ংক্রিয় AI ট্রান্সক্রিপশন নির্ভুল নয়। তাই যদি ব্যবহারকারী নিজে `.srt` ফাইল প্রদান করেন, Claude সেটিকে সর্বোচ্চ অগ্রাধিকার দেয় এবং ট্রান্সক্রিপশন ধাপটি সম্পূর্ণ এড়িয়ে যায়।

### কীভাবে কাজ করে?

```
srt/ ফোল্ডার চেক করো
  │
  ├── .srt ফাইল আছে? ──YES──▶ সেই SRT ব্যবহার করো, ট্রান্সক্রিপশন বাদ দাও
  │
  └── .srt ফাইল নেই? ──NO───▶ raw/ ফুটেজ থেকে AI ট্রান্সক্রিপশন চালাও
```

### SRT ফাইলের নামকরণ কনভেনশন

```
srt/
├── take_001.srt     # take_001.mp4 এর জন্য
├── take_002.srt     # take_002.mp4 এর জন্য
└── full_script.srt  # পুরো ভিডিওর জন্য একটি SRT
```

### মাল্টিলিঙ্গুয়াল সাপোর্ট

`srt/` ফোল্ডার যেকোনো ভাষার সাবটাইটেল সাপোর্ট করে:
- **বাংলা (bn)** — পূর্ণ সাপোর্ট
- **English (en)** — পূর্ণ সাপোর্ট
- **Hindi (hi)** — পূর্ণ সাপোর্ট
- **Arabic (ar)** — RTL সাপোর্টসহ
- যেকোনো ISO 639-1 ভাষা কোড

ভাষা `config.json`-এ `project.language` ফিল্ডে সেট করো।

---

## Assets Usage Guide (Assets কীভাবে ব্যবহার হবে)

### `assets/images/` — স্ট্যাটিক গ্রাফিক্স

এই ফোল্ডারে রাখা ইমেজ ফাইলগুলো Claude নিচের কাজে ব্যবহার করে:

| ব্যবহার | উদাহরণ | Claude-এর আচরণ |
|---|---|---|
| **ব্র্যান্ড লোগো** | `logo.png`, `logo-white.png` | ওভারলে, ওয়াটারমার্ক, আউট্রো-তে স্বয়ংক্রিয়ভাবে যোগ করে |
| **বিজনেস কার্ড** | `business-card.jpg` | শেষে ফুল-স্ক্রিন বা কর্নারে রাখে |
| **পণ্যের ছবি** | `product-001.png` | স্ক্রিপ্টে উল্লেখ থাকলে সেই মুহূর্তে ওভারলে করে |
| **স্ক্রিনশট** | `app-screenshot.png` | tutorial ভিডিওতে স্ক্রিন শেয়ার হিসেবে ব্যবহার করে |
| **স্ট্যাটিক গ্রাফিক্স** | `intro-card.png` | ইন্ট্রো/আউট্রো কার্ড হিসেবে |

**সাপোর্টেড ফরম্যাট:** `.png` (transparent background সহ), `.jpg`, `.jpeg`, `.webp`, `.svg`

**নামকরণ টিপস:**
- `logo.png` বা `logo-dark.png` — Claude স্বয়ংক্রিয়ভাবে লোগো হিসেবে চেনে
- বর্ণনামূলক নাম দাও: `product-iphone-mockup.png` > `img001.png`

---

### `assets/broll/` — B-roll ভিডিও ফুটেজ

B-roll হলো সেই সাপোর্টিং ভিডিও ক্লিপ যা মূল ভিডিওর (A-roll) উপরে কাটওয়ে শট হিসেবে বসানো হয়।

| ব্যবহার | উদাহরণ | Claude-এর আচরণ |
|---|---|---|
| **কাটওয়ে শট** | `city-timelapse.mp4` | স্ক্রিপ্টে `[b-roll: শহর]` থাকলে বসায় |
| **প্রোডাক্ট শট** | `product-closeup.mp4` | পণ্যের কথা বলার সময় কাটওয়ে করে |
| **অ্যাম্বিয়েন্ট** | `cafe-ambience.mp4` | ব্যাকগ্রাউন্ড ভিজ্যুয়াল হিসেবে |
| **স্ক্রিন রেকর্ডিং** | `app-demo.mp4` | টিউটোরিয়ালে স্ক্রিন শেয়ার হিসেবে |

**সাপোর্টেড ফরম্যাট:** `.mp4`, `.mov`, `.mkv`, `.webm`

**গুরুত্বপূর্ণ:** B-roll ফাইলের নামে বিষয়বস্তু উল্লেখ করো যাতে Claude স্ক্রিপ্টের সাথে মেলাতে পারে।

---

### `assets/music/` ও `assets/sfx/`

| ফোল্ডার | কী রাখবে | ব্যবহার |
|---|---|---|
| `music/` | ব্যাকগ্রাউন্ড ট্র্যাক | `config.json`-এর `audio.music_duck_db` অনুযায়ী অটো-ডাক করা হয় |
| `sfx/` | সাউন্ড ইফেক্ট | স্ক্রিপ্টে `[sfx: whoosh]` লিখলে `sfx/whoosh.mp3` খোঁজা হয় |

**SFX নামকরণ কনভেনশন:**

```
assets/sfx/
├── whoosh.mp3      # [sfx: whoosh]
├── pop.mp3         # [sfx: pop]
├── click.mp3       # [sfx: click]
├── riser.mp3       # [sfx: riser]
├── swoosh.mp3      # [sfx: swoosh]
└── transition.mp3  # [sfx: transition]
```

---

### `assets/fonts/`

ক্যাপশন রেন্ডারিংয়ের জন্য ফন্ট ফাইল এখানে রাখো।

```
assets/fonts/
├── caption-regular.woff2    # প্রধান ক্যাপশন ফন্ট
├── caption-bold.woff2       # বোল্ড হাইলাইটের জন্য
└── display.ttf              # শিরোনাম/হেডলাইনের জন্য
```

`config.json`-এ `caption.font_size_1080: 68` দিয়ে ডিফল্ট সাইজ নিয়ন্ত্রণ করো।
