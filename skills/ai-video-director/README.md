# ai-video-director

> **The only AI skill that edits your short-form video — word by word, beat by beat — entirely in code.**

---

## স্কিলের পরিচয় ও কেন এটি অনন্য

`ai-video-director` হলো একটি **end-to-end ভিডিও প্রোডাকশন স্কিল** যা Claude Code-কে একজন পেশাদার ভিডিও এডিটরের সমতুল্য করে তোলে। সাধারণ AI ভিডিও টুলস যেখানে শুধু captions বা basic cut করে, এই স্কিল:

| অন্য টুলস | ai-video-director |
|-----------|-------------------|
| ক্যাপশন যোগ করে | প্রতিটি শব্দ ধরে ধরে ক্যাপশন animate করে |
| Basic cut করে | Word-level timestamp দিয়ে ০.১৫s পর্যন্ত নীরবতা কাটে |
| Color filter প্রয়োগ করে | Cinematic S-curve + warm skintone + film grain গ্রেড করে |
| একটি ক্লিপ নিয়ে কাজ করে | Multiple takes থেকে best take নির্বাচন করে |
| Human approval দরকার হয় বারবার | **মাত্র একবার** preview-এ থামে |
| শুধু ইংরেজি সাপোর্ট করে | বাংলা সহ **মাল্টিলিং্গুয়াল** সাপোর্ট |
| SRT থাকলেও ট্রান্সক্রাইব করে | **SRT detect করে, transcription skip করে** |

### কেন এটি Roberto Nickson (RPN) / Kallaway স্টাইল?

- **Tight cuts**: কোনো dead air নেই।
- **Sync map**: প্রতিটি visual এবং কথার মধ্যে সূক্ষ্ম সংযোগ।
- **Colour grade**: সিনেমাটিক, ওয়ার্ম, ফিল্মিক লুক।
- **Kinetic captions**: শব্দ ধরে ধরে হাইলাইট, active keyword কালার।
- **Story beats**: "Before vs After", "Raw vs Edited" moment — সব নাটকীয়ভাবে উপস্থাপিত।

---

## THE MASTER PROMPT

নিচে সম্পূর্ণ master prompt দেওয়া হলো। এটি word for word ব্যবহার করো। `[...]` দিয়ে চিহ্নিত অংশগুলো তোমার প্রজেক্ট অনুযায়ী পরিবর্তন করো।

---

```
You are an expert AI video editor. Use HyperFrames, Remotion 3D Engine, and ffmpeg to edit a 9:16 vertical short-form video (Instagram Reels / TikTok) following the ai-video-director skill pipeline exactly.

PROJECT FOLDER: [your/project/folder/path]

FOLDER STRUCTURE:
- clips/         ← raw video takes (MP4/MOV)
- srt/           ← [OPTIONAL] pre-made SRT files with word-level timestamps
- images/        ← [OPTIONAL] logos, screenshots, graphics
- broll/         ← [OPTIONAL] cutaway footage
- old/           ← [OPTIONAL] old video to re-edit
- output/        ← all outputs go here

SCRIPT (word for word, as spoken):
"""
[Paste the exact script here, exactly as spoken in the video.
Each paragraph = one logical section. Include filler words if present.]
"""

SYNC MAP — STEP 6 MODE:
Choose one:
- OPTION A (RECOMMENDED FOR REAL VIDEOS — FULLY AUTONOMOUS): Write:
  "MODE: AUTONOMOUS SEMANTIC DIRECTOR & REASONING ENGINE.
   I do NOT say effect names in my video. Autonomously analyze my spoken script and SRT semantics.
   - Follow the 5-Stage Cognitive Reasoning Framework from SKILL.md.
   - For every scene: analyze intent, output the mandatory Chain-of-Thought (CoT) reasoning log, autonomously load the matching skill from skills/ (such as skills/remotion-motion-engine for hyper 3D graphics, spring cards & 3D maps, skills/chart-animation, skills/svg-animation, skills/diagram-animation, skills/lower-thirds, skills/talking-head-b-roll, etc.).
   - Generate scene assets in build/scenes/scene_NN/ in 1080x1920 (9:16).
   - Enforce 2-frame handle sync, 2–4s visual rhythm, Roberto Nickson color tokens (#FFFFFF, #141413, #D97757), and safe zone rules (Top: 220px, Bottom: 420px, Right: 140px)."
- OPTION B (FOR DEMO/META REEL):
  Paste custom sync map keywords, or write "Use Meta Demo sync map from SKILL.md".

SRT FILES:
[If srt/ folder has files, write: "SRT files present in srt/ folder. Use them. Do NOT transcribe."]
[If no SRT files, write: "No SRT files. Transcribe all clips."]

ASSETS:
[List any specific images/ or broll/ files and when to use them. Example:
- images/logo.png → watermark, top-right corner, opacity 60%, all video
- broll/screen_record.mp4 → use at "one prompt" beat
- images/screenshot.png → use at "This is what I gave it" beat]

LANGUAGE: [English / বাংলা / Hindi / Mixed]

CAPTION STYLE:
- Font: [Montserrat ExtraBold / Noto Sans Bengali Bold / other]
- Active keyword color: #D97757
- Caption groups: 2–4 words per group
- Fade-in: 80ms

OLD VIDEO RE-EDIT:
[If old/ folder has a video, write: "Re-edit old/[filename.mp4] with new style to output/old_reedit.mp4"]
[If not applicable, write: "No old video to re-edit."]

PIPELINE INSTRUCTIONS:
0. 📋 STEP 0: CREATIVE STORYBOARD & BLUEPRINT (MANDATORY APPROVAL GATE #1):
   - Before cutting footage, generating code, or assembling timeline, you MUST output a super-detailed Creative Storyboard & Execution Blueprint Table for the entire video scene-by-scene:
     * Scene ID & Exact Timing (Start/End)
     * Spoken Script Line
     * A-Roll Composition: When is A-Roll Full-Screen (100% wide / 120% close) vs PiP (corner floating pill / hex mask)?
     * Custom BG for PiP: When PiP is active, you MUST use my supplied background (`assets/images/bg.*` or `assets/broll/bg.*`) with cinematic vignette — NEVER plain black or solid flat color.
     * Strict Face Protection: 100% guarantee that NO text, graphic card, icon, or motion asset covers or overlaps the character's face/eyes.
     * Target Skill & Asset Sourcing: Explicit skill from `skills/` (`skills/remotion-motion-engine`, `skills/svg-animation`, `skills/chart-animation`, `skills/diagram-animation`, `skills/talking-head-b-roll`, etc.) and asset type (Lottie, Pure SVG, Web/local image download, or 3D MapLibre/Cesium map).
     * Asset BG Treatment: Cutout transparent alpha (rembg/canvas) vs keep background vs 3D glass card.
     * Visual Effect & Motion Mechanics: Spring physics (mass/damping), 3D perspective tilt, path-draw stroke-dashoffset, mask-reveal split, camera orbit.
     * Apple-Level 3D Hook: First 3 seconds MUST feature an Apple keynote-level 3D motion graphics hook (Remotion Three.js 3D glass particle hologram or 3D spring tilt) to stop the viewer's scroll instantly.
     * Anti-Box Rule & Punchy Typography: Motion graphics do NOT have to be trapped inside generic rounded boxes. Use borderless mask reveals (`clip-path`) and floating 3D elements. Typography MUST be large, punchy, easily readable (68-84px), with accent keyword glow in #D97757.
     * SFX & Audio Sync: Precise sound effects (`whoosh`, `pop`, `click`, `riser`) and ducking.
   - ⛔ STOP HERE (GATE #1): Present the complete Creative Blueprint table in chat and STOP. Wait for my approval ("Approve" or revisions) before proceeding to Step 1.
1. Follow the 10-step pipeline from the ai-video-director SKILL.md.
2. Check srt/ folder FIRST. If any .srt file matches a clip name, use it directly — do NOT call `npx hyperframes transcribe` for that clip.
3. Select the best take for each script line. Document every decision in takes.md.
4. Remove all pauses > 0.15s. Keep 2-frame handles on every cut.
5. Apply cinematic colour grade: S-curve, warm skintone, cool shadows, vignette, film grain.
6. Apply smart zooms: face-tracked, alternating 100% and 120%, max 115% for 1080p source.
7. Apply captions: word-level timing, active keyword in #D97757, safe zone enforced.
8. Execute the approved sync map beats — for 3D/maps use Remotion 3D Engine with WebGL Angle backend.
9. Clean voice audio, duck music to 12%, master to -14 LUFS.
10. Generate preview (540×960) + stills for every sync beat.
11. ⛔ STOP HERE (GATE #2). Show preview, stills, and 9-point QC report. Wait for final approval.
12. After approval: render output/final.mp4 at 1080×1920, H.264 ~18 Mbps, AAC 320k.
13. Print final report with real numbers.
```

---

## 🧠 Autonomous Context Understanding & Scene-by-Scene Reasoning Engine

বাস্তব ভিডিওতে ক্রিয়েটর মুখে কখনো এডিটিং ইফেক্টের নাম বলে না। তাই এই স্কিলে যুক্ত করা হয়েছে **Autonomous Context Understanding & Scene-by-Scene Reasoning Engine**।

### এটি কীভাবে কাজ করে?

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                   AUTONOMOUS SCENE-BY-SCENE REASONING PIPELINE                   │
├──────────────────────────────────────────────────────────────────────────────────┤
│ 1. Subtitle/Script Ingestion  ➔ Word-level timestamp সহ লাইন ও পজ শনাক্ত করে    │
│ 2. Semantic Chunking           ➔ বাক্য ও ভাব অনুসারে সিন ভাগ করে (০.১৫s পজ)       │
│ 3. Cognitive Classification    ➔ সিনের উদ্দেশ্য (Data, Flow, Accent, B-roll ইত্যাদি) নির্ণয় │
│ 4. Chain-of-Thought (CoT)      ➔ প্রতিটি সিনের আগে কারণ ও ভিজ্যুয়াল স্ট্র্যাটেজি লগে লেখে  │
│ 5. Dynamic Skill Loading       ➔ skills/ ফোল্ডার থেকে প্রয়োজনীয় স্কিল মেমোরিতে লোড করে  │
│ 6. Isolated Asset Generation   ➔ 1080x1920 ক্যানভাসে কোড জেনারেট করে (build/scenes/)    │
│ 7. Sequential Timeline Blend   ➔ ২-ফ্রেম রুল ও সেফ জোনে মাস্টার ভিডিওতে অ্যাসেট সিঙ্ক করে │
└──────────────────────────────────────────────────────────────────────────────────┘
```

### কোন কনটেক্সটে কোন স্কিল লোড হবে?

| যখন স্ক্রিপ্টে থাকবে | এআই যে স্কিল লোড করবে | জেনারেটেড ভিজ্যুয়াল আউটপুট |
|----------------------|-----------------------|---------------------------|
| **সংখ্যা, পার্সেন্টেজ, ডাটা বৃদ্ধি/হ্রাস** | `skills/chart-animation` | ফ্রেম-ড্রিভেন বার/লাইন চার্ট, কাউন্টার টিকার |
| **সিস্টেম আর্কিটেকচার, স্টেপস, ফ্লো** | `skills/diagram-animation` | ভেক্টর নোড ডায়গ্রাম, কানেক্টিং ডাটা ফ্লো |
| **শব্দে জোর, সার্কেল বা আন্ডারলাইন** | `skills/svg-animation` | পেন-ড্রন ভেক্টর হাইলাইটার, অ্যারো, আন্ডারলাইন |
| **বোল্ড পাঞ্চলাইন ও স্মরণীয় উক্তি** | `skills/kinetic-typography` | হাই-স্পিড কাইনেটিক টাইপোগ্রাফি |
| **বক্তার পরিচয়, পদবী বা সোশ্যাল** | `skills/lower-thirds` | ট্রান্সপারেন্ট ব্রডকাস্ট লোয়ার থার্ড |
| **কাজের দৃশ্য, গল্প বা ডেমো** | `skills/talking-head-b-roll` | ক্যানভাস জেনারেটিভ বি-রোল (Treatment B/C) |
| **তুলনামূলক ফিচার বা লিস্ট** | `skills/animated-infographic` | মাল্টি-পার্ট ইনফোগ্রাফিক কার্ড |
| **ব্র্যান্ড লোগো বা টুল ইন্ট্রো** | `skills/logo-animation` | ভেক্টর লোগো স্টিং ও রিভিল |
| **সাবস্ক্রাইব বা কল-টু-অ্যাকশন** | `skills/lottie-animation` | লাইটওয়েট লটি মাইক্রো-ইন্টারঅ্যাকশন |
| **মুড শিফট বা ব্যাকগ্রাউন্ড ডেপথ** | `skills/motion-background` | পার্টিকেল ড্রাইভেন মোশন ব্যাকগ্রাউন্ড |
| **৩D সিন, মেটাবিক অবজেক্ট, ফ্লোটিং কার্ড** | `skills/remotion-motion-engine` | Three.js ৩D স্পেস, ThreeCanvas, স্প্রিং ফিজিক্স গ্লাস কার্ড |
| **মানচিত্র, বর্ডার, দেশ, ফ্লাইট রুট (ভক্স স্টাইল)** | `skills/remotion-motion-engine` | MapLibre 3D, Turf.js কার্ভড রুট, CesiumJS ৩D সিটি ফ্লাইওভার |

---

## STEP 6 কাস্টমাইজেশন গাইড (Meta / Demo Mode এর জন্য)

### নিজের স্ক্রিপ্ট অনুযায়ী Sync Map রিরাইট কিভাবে করতে হয়

Sync map হলো স্ক্রিপ্টের নির্দিষ্ট শব্দ/ফ্রেজের সাথে ভিজ্যুয়াল/অডিও ইভেন্টের একটি তালিকা। Default sync map RPN/Kallaway-এর "Editing with Claude Code" ভিডিওর জন্য। তোমার নিজের স্ক্রিপ্টের জন্য নিচের পদ্ধতি অনুসরণ করো:

#### ধাপ ১: স্ক্রিপ্ট বিশ্লেষণ

তোমার স্ক্রিপ্ট পড়ে নিচের প্রতিটি ক্যাটাগরির জন্য কিওয়ার্ড চিহ্নিত করো:

| ক্যাটাগরি | কী খুঁজবে | Default Effect |
|-----------|----------|----------------|
| Hard cut moment | "cut", "trim", "remove" | Jump cut + click SFX |
| Before/After reveal | "before", "used to", "old version" | Ungraded → graded wipe |
| Data/Stats | Numbers, percentages | Counter animation |
| CTA | "comment", "follow", "subscribe" | Kinetic text + bubble |
| Tool/App mention | Software names | App card overlay |
| Process reveal | "I did", "I used", "here's how" | Terminal card |
| Transformation | "now", "after", "result" | Riser + wipe |

#### ধাপ ২: Beat Template

প্রতিটি beat নিচের ফরম্যাটে লেখো:

```
[Keyword: "তোমার কিওয়ার্ড"]
- Trigger: এই শব্দটি বলার ঠিক সময়
- Visual: কী দেখাবে (overlay / transition / animation)
- Audio: কী শোনাবে (SFX name, দৈর্ঘ্য, ভলিউম)
- Duration: কতক্ষণ ইফেক্ট থাকবে
- Exit: কীভাবে শেষ হবে (fade-out / slide-out / cut)
```

#### ধাপ ৩: উদাহরণ — একটি "Product Review" ভিডিওর Sync Map

নিচে একটি কাস্টম sync map উদাহরণ:

```
CUSTOM SYNC MAP — Product Review Video

[ভিডিও শুরু → "unboxing" পর্যন্ত]
- ক্যাপশন: বন্ধ
- Title card: "HONEST REVIEW — NO SPONSORSHIP" (সাদা টেক্সট, কালো পিল)

[Keyword: "price"]
- Visual: Price card overlay — "৳২৫,০০০" বড় করে, counter animation
- Audio: cash register SFX (-15dB, 30ms)
- Duration: 2s
- Exit: slide-out left

[Keyword: "unboxing"]
- Visual: Caption চালু, title card fade-out
- Audio: রিস্টলিং paper SFX (-12dB, 0.8s)

[Keyword: "pros"]
- Visual: সবুজ চেকমার্ক (✓) list build করে — প্রতিটি pro একটি করে
- Audio: soft tick SFX প্রতিটি চেকমার্কে

[Keyword: "cons"]
- Visual: লাল X list build করে
- Audio: soft buzz SFX প্রতিটি X-এ

[Keyword: "verdict"]
- Visual: স্টার রেটিং কার্ড (X/5 stars)
- Audio: drumroll (0.5s) → sting

[Keyword: "Comment"]
- Visual: Kinetic text bounce + comment bubble
- Audio: notification ping SFX

[শেষ ফ্রেম = প্রথম ফ্রেম: seamless loop]
```

#### ধাপ ৪: Sync Map Prompt-এ পেস্ট করো

উপরের কাস্টম sync map তৈরি করার পরে, master prompt-এর `SYNC MAP` সেকশনে সরাসরি পেস্ট করো।

#### গুরুত্বপূর্ণ নিয়মাবলী

- **একসাথে দুটো ইফেক্ট কখনো না**: একটি শেষ হওয়ার পরেই পরেরটি শুরু।
- **২–৪ সেকেন্ড rule**: প্রতি ২–৪ সেকেন্ডে কিছু একটা পরিবর্তন হতে হবে।
- **Seamless loop**: শেষ ও প্রথম ফ্রেম সবসময় মেলাও।
- **Safe zone**: সব overlay safe zone-এর মধ্যে রাখো।

---

## SRT ফাইল ব্যবহারের নিয়ম

### ফাইল নামিং কনভেনশন

SRT ফাইলের নাম **হুবহু** ক্লিপের নামের সাথে মিলতে হবে (extension বাদে):

| ক্লিপ ফাইল | সঠিক SRT নাম | ভুল SRT নাম |
|-----------|-------------|-------------|
| `take_1.mp4` | `take_1.srt` ✅ | `Take1.srt` ❌ |
| `interview_final.mov` | `interview_final.srt` ✅ | `interview.srt` ⚠️ (fuzzy match) |
| `clip_20241015_003.mp4` | `clip_20241015_003.srt` ✅ | `clip_003.srt` ⚠️ |

> ⚠️ Fuzzy match (partial name similarity) কাজ করে কিন্তু নির্ভরযোগ্য নয়। Exact match সবসময় ভালো।

### Word-Level SRT ফরম্যাট

**সঠিক ফরম্যাট** (প্রতিটি entry = একটি শব্দ):

```srt
1
00:00:01,240 --> 00:00:01,480
This

2
00:00:01,480 --> 00:00:01,720
is

3
00:00:01,720 --> 00:00:02,100
amazing

4
00:00:02,100 --> 00:00:02,500
content
```

**বাক্য-স্তরের SRT** (word-level timing ছাড়া) — এটি sync accuracy কমায়:

```srt
1
00:00:01,240 --> 00:00:02,500
This is amazing content
```

> যদি word-level SRT না থাকে কিন্তু sentence-level SRT থাকে, তাহলে:
> - Sentence-level SRT ব্যবহার করো pause detection-এর জন্য।
> - Word-level animation-এর জন্য proportional interpolation করো।
> - বা `npx hyperframes transcribe` চালাও শুধু word-timestamps-এর জন্য।

### বাংলা SRT উদাহরণ

```srt
1
00:00:00,800 --> 00:00:01,050
আমি

2
00:00:01,050 --> 00:00:01,380
আজকে

3
00:00:01,380 --> 00:00:01,820
দেখাবো

4
00:00:01,820 --> 00:00:02,100
কীভাবে

5
00:00:02,100 --> 00:00:02,650
Claude

6
00:00:02,650 --> 00:00:03,200
ভিডিও

7
00:00:03,200 --> 00:00:03,800
এডিট

8
00:00:03,800 --> 00:00:04,200
করে
```

- ফাইল সেভ করো **UTF-8 encoding** দিয়ে (BOM ছাড়া)।
- Windows-এ Notepad++ → Encoding → UTF-8 without BOM।
- VS Code-এ status bar-এ "UTF-8" নিশ্চিত করো।

### SRT ফাইল তৈরির টুলস

| টুল | মূল্য | Word-Level | বাংলা |
|-----|------|-----------|-------|
| HyperFrames CLI | Free (self-hosted) | ✅ | ✅ |
| Whisper (OpenAI) | API cost | ✅ | ✅ |
| Kapwing | Freemium | ❌ | ⚠️ |
| Adobe Premiere | Paid | ✅ | ⚠️ |
| Descript | Paid | ✅ | ❌ |

**সবচেয়ে ভালো**: HyperFrames বা Whisper দিয়ে নিজে generate করো, তারপর `srt/` ফোল্ডারে রাখো।

---

## Assets ব্যবহার প্রম্পট

### images/ ফোল্ডার — Prompt উদাহরণ

#### লোগো ওয়াটারমার্ক হিসেবে:

```
ASSETS:
- images/logo.png → watermark, position: top-right (60px from edges),
  opacity: 60%, size: 120px wide, visible throughout entire video
```

#### স্ক্রিনশট "proof" হিসেবে:

```
ASSETS:
- images/analytics_screenshot.png → use at keyword "results",
  show as overlay card (640px wide, centered), duration: 3s,
  label: "REAL ANALYTICS" in orange pill above image
```

#### চার্ট motion graphic হিসেবে:

```
ASSETS:
- images/growth_chart.png → use at keyword "growth",
  animate: slide up from bottom (0.5s ease-out), hold 2.5s,
  then slide down. Orange bar above with text "REAL DATA"
```

#### একাধিক images/:

```
ASSETS:
- images/before.png → use at "before" keyword, left side, BEFORE label
- images/after.png → use at "after" keyword, right side, AFTER label
- images/logo.png → watermark bottom-right, opacity 40%, entire video
```

### broll/ ফোল্ডার — Prompt উদাহরণ

#### সিম্পল broll cut:

```
ASSETS:
- broll/hands_typing.mp4 → cut to this at "typing" keyword,
  duration: match the script segment (≈ 2s), then cut back
```

#### broll-কে zoom-এর বিকল্প হিসেবে:

```
ASSETS:
- broll/product_closeup.mp4 → use instead of zoom at "look at this" beat.
  Cut from main camera to broll, hold 3s, cut back.
  No zoom animation needed for this segment.
```

#### broll loop করা:

```
ASSETS:
- broll/city_timelapse.mp4 → use as background during intro (0s–5s).
  Loop if shorter than 5s. Main speaker overlay on top (50% opacity).
```

### images/ ও broll/ একসাথে:

```
ASSETS:
- images/app_logo.png → at "app" keyword: slide in from left (0.3s),
  hold 2s, slide out right
- broll/app_demo.mp4 → immediately after app_logo exits: cut to broll,
  show app demo for 4s, cut back to main camera
- images/download_qr.png → at "download" keyword: bottom-center overlay,
  150px, fade-in 200ms, hold until end of sentence
```

---

## বাংলা ভিডিও প্রম্পট

বাংলা কনটেন্টের জন্য master prompt-এ নিচের অতিরিক্ত নির্দেশনা যোগ করো:

### বাংলা Caption Spec

```
CAPTION STYLE (বাংলা):
- Primary Font: Noto Sans Bengali Bold
- Fallback Font: Hind Siliguri Bold
- Font size: 76px (বাংলা অক্ষর ইংরেজির চেয়ে সামান্য বড় দেখায়, তাই 4px বাড়াও)
- Active keyword color: #D97757
- Text direction: LTR (বাংলা LTR)
- Word grouping: 2–3 শব্দ প্রতি ক্যাপশন গ্রুপে
  (বাংলা শব্দ সাধারণত ইংরেজির চেয়ে বেশি জায়গা নেয়)
- Line height: 1.4 (বাংলা descenders-এর জন্য)
- Encoding: UTF-8, no BOM
- Subtitle format: ASS (SRT নয় — ASS-এ বাংলা ফন্ট বেটার রেন্ডার করে)
```

### বাংলা Transcription Command

```bash
npx hyperframes transcribe \
  --input clips/ \
  --output srt/ \
  --word-level true \
  --language bn \
  --model large-v3
```

- `--language bn`: বাংলা ভাষা specify করো।
- `--model large-v3`: ছোট মডেল বাংলায় কম accurate — large-v3 ব্যবহার করো।

### বাংলা Script Prompt উদাহরণ

```
LANGUAGE: বাংলা

SCRIPT:
"""
আজকে আমি দেখাবো কীভাবে Claude Code দিয়ে
একটা পুরো ভিডিও এডিট করা যায়।
কোনো editing app লাগবে না।
শুধু একটা prompt।
এটাই আমি দিয়েছিলাম।
আর এটাই আমি পেয়েছিলাম।
Comment করো CLAUDE।
"""

SYNC MAP:
[Keyword: "editing app"]
- Visual: Card overlay — "Editing apps opened: ০" (Bengali numeral)
- Audio: negative beep SFX (-15dB)
- Duration: 2s, exit: fade-out

[Keyword: "prompt"]
- Visual: Terminal card — first line of Bengali prompt types out
- Counter: Raw length → Final length (real numbers)

[Keyword: "দিয়েছিলাম"]  ← "gave it" in Bengali
- Visual: Ungraded raw footage + RAW tag (red) + timecode
- Audio: none

[Keyword: "পেয়েছিলাম"]  ← "gave me back" in Bengali
- Visual: Riser (1.5s) → wipe to final edit

[Keyword: "CLAUDE"]
- Visual: Kinetic text bounce + comment bubble with "CLAUDE"
- Audio: notification ping

SRT FILES: [উল্লেখ করো]

CAPTION FONT: Noto Sans Bengali Bold, 76px
```

### বাংলা-ইংরেজি মিশ্র ভিডিও

```
LANGUAGE: Mixed (বাংলা primary, English technical terms)

CAPTION STYLE:
- Bangla words: Noto Sans Bengali Bold
- English words (in same caption): Montserrat ExtraBold
- Use font fallback chain in ASS: "Noto Sans Bengali,Montserrat,sans-serif"
- Active keyword: #D97757 for BOTH languages

Note: Do NOT transliterate Bengali to Latin. Render Bengali text natively.
```

---

## উদাহরণ takes.md ফরম্যাট

`takes.md` ফাইলটি STEP 1-এ তৈরি হয় এবং পুরো পাইপলাইন জুড়ে আপডেট হয়। নিচে একটি সম্পূর্ণ উদাহরণ:

```markdown
# takes.md — AI Video Director
Generated: 2024-10-15 14:32:18
Project: my_viral_video
Total clips: 5 (take_1.mp4 through take_5.mp4)
SRT status: take_3.srt, take_5.srt found in srt/ — transcription skipped for these.
            take_1.mp4, take_2.mp4, take_4.mp4 transcribed via HyperFrames.

---

## Best Take Selection

| # | Script Line | Selected Take | File | In | Out | Duration | Reason |
|---|-------------|---------------|------|----|-----|----------|--------|
| 1 | "আজকে আমি দেখাবো" | take_5 | take_5.mp4 | 00:02.1 | 00:04.8 | 2.7s | শেষ সম্পূর্ণ টেক, সবচেয়ে ভালো এনার্জি |
| 2 | "কীভাবে Claude Code দিয়ে" | take_5 | take_5.mp4 | 00:04.8 | 00:07.6 | 2.8s | Continuous from line 1, no stumble |
| 3 | "একটা পুরো ভিডিও এডিট করা যায়" | take_3 | take_3.mp4 | 00:08.2 | 00:12.1 | 3.9s | take_5-এ "ভিডিও" শব্দে stumble ছিল |
| 4 | "কোনো editing app লাগবে না" | take_5 | take_5.mp4 | 00:12.5 | 00:15.0 | 2.5s | শেষ সম্পূর্ণ টেক |
| 5 | "শুধু একটা prompt" | take_4 | take_4.mp4 | 00:03.1 | 00:04.9 | 1.8s | take_5 clipped word on "prompt" |
| 6 | "এটাই আমি দিয়েছিলাম" | take_5 | take_5.mp4 | 00:15.8 | 00:18.2 | 2.4s | শেষ সম্পূর্ণ টেক, ভালো pause |
| 7 | "আর এটাই আমি পেয়েছিলাম" | take_5 | take_5.mp4 | 00:18.2 | 00:21.0 | 2.8s | Continuous, ভালো dramatic pause |
| 8 | "Comment করো CLAUDE" | take_5 | take_5.mp4 | 00:21.0 | 00:23.5 | 2.5s | Energetic delivery |

---

## Rejected Takes Log

| Take | Line # | Rejection Reason | Details |
|------|--------|-----------------|---------|
| take_1 | ALL | flat energy | একঘেয়ে delivery, উৎসাহ নেই |
| take_2 | 3 | stumble | "ভিডিও" শব্দ দুবার বলা হয়েছে |
| take_5 | 3 | stumble | "ভিডিও" → "ভিডি-ভিডিও" detected |
| take_5 | 5 | clipped word | "prompt" এর শেষ অক্ষর clip হয়েছে |
| take_3 | 4 | flat energy | "editing app" সেগমেন্টে low energy |

---

## Pause Removal Log (STEP 2)

| Location | Pause Duration | Action |
|----------|---------------|--------|
| Line 1 → Line 2 | 0.42s | REMOVED (> 0.15s threshold) |
| Line 2 → Line 3 (cross-take) | 0.18s | REMOVED |
| Line 3 → Line 4 | 0.11s | KEPT (breath pause, ≤ 0.15s) |
| Line 4 → Line 5 | 0.67s | REMOVED |
| Line 5 → Line 6 | 0.23s | REMOVED |
| Line 6 → Line 7 | 0.09s | KEPT (natural rhythm) |
| Line 7 → Line 8 | 0.31s | REMOVED |

Total pauses removed: 5
Total time cut from pauses: 1.81s

---

## Timestamp Adjustments (STEP 6)

| Beat | Script Word | Transcript Timestamp | Waveform Timestamp | Adjustment |
|------|-------------|---------------------|-------------------|------------|
| editing app card | "app" | 8.920s | 8.780s | -140ms (waveform used) |
| RAW tag | "দিয়েছিলাম" | 15.340s | 15.190s | -150ms (waveform used) |
| riser | "পেয়েছিলাম" | 18.220s | 18.100s | -120ms (waveform used) |

---

## Final Stats

| Metric | Value |
|--------|-------|
| Raw footage total | 4m 12s |
| Final edit length | 23.5s |
| Takes rejected | 2 (take_1, partially take_2) |
| Segments used | 8 from 3 different takes |
| Pauses removed | 5 (total: 1.81s) |
| SRT files used | 2 (take_3, take_5) |
| Clips transcribed | 3 (take_1, take_2, take_4) |
```

---

## FAQ

### ❓ SRT ফাইল না থাকলে কি হবে?

`srt/` ফোল্ডার খালি থাকলে বা না থাকলে, পাইপলাইন স্বয়ংক্রিয়ভাবে `npx hyperframes transcribe` চালাবে। তোমাকে আলাদা কিছু করতে হবে না।

---

### ❓ আমার script অনেক ছোট, sync map দরকার নেই — কি করবো?

Master prompt-এ লেখো:
```
SYNC MAP: Minimal. Apply only captions, tight cuts, colour grade, and smart zooms.
No beat-specific overlays needed.
```

---

### ❓ শুধু captions চাই, বাকি কিছু না — কি করবো?

```
SYNC MAP: Captions only. Skip all beat overlays, kinetic text, and motion graphics.
Apply tight cut, colour grade, captions, and final render only.
```

---

### ❓ Multiple speakers আছে (interview format) — কীভাবে handle করবো?

Master prompt-এ যোগ করো:
```
SPEAKERS: 2
Speaker 1: [Name] — left side of frame
Speaker 2: [Name] — right side of frame

CAPTION STYLE: Speaker-differentiated
- Speaker 1 captions: position bottom-left quadrant
- Speaker 2 captions: position bottom-right quadrant
- Speaker 1 active color: #D97757
- Speaker 2 active color: #57A0D9
```

---

### ❓ ৯:১৬ ছাড়া অন্য aspect ratio চাই — কি করবো?

```
OUTPUT FORMAT: 16:9 landscape (YouTube long-form)
Resolution: 1920×1080
Safe zone: top 80px, bottom 120px, left 60px, right 60px
Caption position: bottom-center (y ≈ 85% of height)
```

---

### ❓ Preview পছন্দ না হলে কী করবো?

Preview দেখার পরে নির্দিষ্ট feedback দাও:

```
Preview feedback:
1. Caption font size বাড়াও — 72px → 90px
2. "editing app" beat-এ card 1s বেশি থাকুক (2s → 3s)
3. Colour grade কম saturated করো — vivid লাগছে
4. Line 3-এ zoom level কমাও — 120% → 110%
Re-render preview only (not final render).
```

---

### ❓ Error: "face not detected" — কী করবো?

```
ZOOM FALLBACK: If face tracking fails for any segment,
use center-crop zoom (center of frame) instead.
Do not fail the pipeline — log the fallback in takes.md.
```

---

### ❓ Output ফাইল সাইজ অনেক বড় হচ্ছে — কী করবো?

```
OUTPUT QUALITY: Web-optimized
- Video bitrate: 8 Mbps (instead of 18 Mbps)
- CRF: 23 (instead of 18)
- Preset: medium (instead of slow)
- Audio: AAC 192k (instead of 320k)
Target file size: < 50 MB for 60s video
```

---

### ❓ বাংলা ক্যাপশন সঠিকভাবে রেন্ডার হচ্ছে না — কী করবো?

1. **Noto Sans Bengali** ফন্ট ইনস্টল নিশ্চিত করো।
2. ASS subtitle ফরম্যাট ব্যবহার করো (SRT নয়)।
3. FFmpeg compile check:
   ```bash
   ffmpeg -fonts | grep -i bengali
   ```
4. যদি ফন্ট না পাওয়া যায়:
   ```bash
   # Ubuntu/Debian
   apt-get install fonts-noto-core
   # Windows: Download from Google Fonts, install system-wide
   ```

---

### ❓ পাইপলাইন মাঝপথে থেমে গেলে কীভাবে resume করবো?

```
RESUME FROM: Step [X]
Previous completed steps output:
- tight_cut.mp4 exists ← Step 2 complete
- graded.mp4 exists ← Step 3 complete
- face_data.json exists ← Step 4 started
Resume from Step 4 (Smart Zooms). Do not redo Steps 1-3.
```

---

### ❓ takes.md কি manually এডিট করতে পারি?

হ্যাঁ। যদি তুমি জানো যে কোন take সেরা, manually লিখে দাও:

```
TAKES OVERRIDE (manually specified):
- Line 1-4: use take_3.mp4
- Line 5-8: use take_5.mp4
- Do not run automatic best-take selection algorithm.
Read timestamps from the specified takes' SRT files.
```

---

### ❓ `-14 LUFS` কেন? আমার platform আলাদা নরম করতে বলছে।

| Platform | Target LUFS | True Peak |
|---------|-------------|-----------|
| Instagram Reels | -14 LUFS | -1.5 dBTP |
| TikTok | -14 LUFS | -1.0 dBTP |
| YouTube Shorts | -14 LUFS | -1.0 dBTP |
| Spotify (podcast) | -16 LUFS | -1.5 dBTP |
| Broadcast (EBU R128) | -23 LUFS | -1.0 dBTP |

Platform specify করো:
```
MASTERING TARGET: TikTok (-14 LUFS, -1.0 dBTP)
```

---

*ai-video-director v2.0 — AI Systems Lab*
*"No timeline. Only code."*
