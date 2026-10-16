# ffmpeg-video-cutter

> AI-চালিত ভিডিও কাটিং, সিনেমাটিক গ্রেডিং ও ফেস-ট্র্যাকিং জুম — সম্পূর্ণ ffmpeg-ভিত্তিক পাইপলাইন

---

## ১. স্কিলের পরিচয়

`ffmpeg-video-cutter` হলো একটি production-ready AI skill যা raw talking-head footage-কে প্রফেশনাল-গ্রেড short-form ভিডিওতে রূপান্তর করে। এটি নিম্নলিখিত সমস্যাগুলো স্বয়ংক্রিয়ভাবে সমাধান করে:

| সমস্যা | সমাধান |
|--------|--------|
| Long pauses এবং "উম/আহ" | Silence detection + transcript matching |
| Flat, ungraded footage | Cinematic colour grading pipeline |
| Static, boring framing | Face-tracking dynamic zoom |
| Wrong takes included | `takes.md` driven take selection |
| Manual B-roll insertion | Keyword-based automatic B-roll matching |
| Jarring audio cuts | 3ms crossfade at every cut point |
| Timestamp drift | Waveform-verified cut points |

### স্কিলটি কোন ওয়ার্কফ্লোতে বসে

```
Raw Footage (mp4/mov)
        │
        ▼
[Phase 1] Audio Analysis ──── SRT/Transcript
        │
        ▼
[Phase 2] Take Selection ──── takes.md
        │
        ▼
[Phase 3] Silence Removal + Crossfade
        │
        ▼
[Phase 4] Colour Grading
        │
        ▼
[Phase 5] Face Detection + Zoom
        │
        ▼
[Phase 6] B-roll Integration ──── assets/broll/
        │
        ▼
[Phase 7] Final Assembly
        │
        ▼
output/assembled.mp4  +  output/ungraded_ref.mp4
```

### প্রয়োজনীয় ফাইল স্ট্রাকচার

```
project/
├── raw/
│   ├── raw_take_01.mp4
│   ├── raw_take_02.mp4
│   └── raw_take_03.mp4
├── assets/
│   └── broll/
│       ├── coding_screen_closeup_08s.mp4
│       └── coffee_morning_wide_05s.mp4
├── takes.md          ← take selection guide
├── script.txt        ← script / topic keywords
├── transcript.srt    ← word-level SRT (optional)
└── output/           ← generated output
    ├── assembled.mp4
    └── ungraded_ref.mp4
```

---

## ২. Cutting Revision Prompts

নিচে ৫টি উদাহরণ প্রম্পট দেওয়া হলো যা দিয়ে তুমি AI-কে নির্দিষ্ট সমস্যা ঠিক করতে বলতে পারবে।

---

### Prompt 1 — ভুল টেক ঠিক করা

**পরিস্থিতি:** অ্যাসেম্বল করা ভিডিওর segment 3-এ ভুল take ব্যবহার হয়েছে। Segment 3-এ `raw_take_01.mp4` ব্যবহার হওয়ার কথা ছিল কিন্তু `raw_take_02.mp4` ব্যবহার হয়েছে।

```
takes.md-এ segment 3 আপডেট করো — ফাইল পরিবর্তন করো raw_take_02.mp4 থেকে
raw_take_01.mp4-এ। In point: 00:01:14.220, Out point: 00:01:31.560।
শুধু segment 3 re-extract করো এবং final assembly-তে replace করো।
audio crossfade এবং 2-frame handle ঠিক রাখো।
```

**প্রত্যাশিত আউটপুট:**
- `takes.md` আপডেট হবে
- `segment_03.mp4` নতুন করে extract হবে
- Final assembly-তে নতুন segment_03 দিয়ে re-concat হবে
- শুধু segment 3 পরিবর্তন হবে, বাকি segments অপরিবর্তিত থাকবে

---

### Prompt 2 — জুম স্টাইল পরিবর্তন

**পরিস্থিতি:** বর্তমানে সব জায়গায় 120% zoom হচ্ছে। তুমি চাও সর্বোচ্চ 110% এবং zoom speed কমাতে।

```
সমস্ত segments-এর zoompan ফিল্টার আপডেট করো:
- Maximum zoom: 1.10 (110%) — 1.20 এর বদলে
- Zoom increment প্রতি ফ্রেমে: 0.0008 (আগে ছিল 0.0015, এখন আরো ধীরে)
- Push-in duration এখন হবে প্রায় 8 seconds (আগে ছিল ~4.4s)
- 1080p source limit মেনে চলো
zoomed.mp4 নতুন করে generate করো এবং final assembly update করো।
```

**প্রত্যাশিত আউটপুট:**
- zoompan z parameter আপডেট: `min(z+0.0008,1.10)`
- সমস্ত segments re-render হবে নতুন zoom settings দিয়ে
- Slower, more subtle push-in effect

---

### Prompt 3 — গ্রেডিং পরিবর্তন

**পরিস্থিতি:** বর্তমান গ্রেডিং অনেক warm মনে হচ্ছে। Skin tone একটু কম orange, shadows একটু কম warm করতে হবে।

```
colour grading filtergraph আপডেট করো:
- gamma_r: 1.02 → 0.99 (warm কমাও, neutral-এ আনো)
- gamma_b: 0.97 → 0.99 (shadows less cool করো, neutral-এর কাছাকাছি)
- saturation: 0.88 → 0.92 (একটু বেশি saturated রাখো)
- noise grain: 8 → 6 (grain কমাও)
এই নতুন parameters দিয়ে graded.mp4 নতুন করে render করো।
ungraded_ref.mp4 অপরিবর্তিত রাখো।
```

**প্রত্যাশিত আউটপুট:**
- filtergraph parameter আপডেট হবে
- নতুন `graded.mp4` তৈরি হবে
- More neutral, slightly more saturated look

---

### Prompt 4 — B-roll যোগ করা

**পরিস্থিতি:** Segment 5-এ "ফোনে স্ক্রল করা" উল্লেখ আছে। `assets/broll/phone_scroll_hands_12s.mp4` ফাইল আছে। Segment 5-এর ৩.০s থেকে ৯.০s পর্যন্ত এই B-roll দিয়ে replace করতে হবে।

```
segment_05_final.mp4-এ B-roll integration করো:
- B-roll file: assets/broll/phone_scroll_hands_12s.mp4
- Overlay duration: 3.0s → 9.0s (6 seconds)
- B-roll-এও একই colour grading apply করো (curves, eq, unsharp, vignette, noise)
- Main audio বজায় রাখো B-roll-এর সময়েও
- Safe zone: 1080x1920 crop নিশ্চিত করো
segment_05_final.mp4 update করো এবং final assembly re-run করো।
```

**প্রত্যাশিত আউটপুট:**
- B-roll graded এবং cropped হবে
- Segment 5-এ 3s-9s পর্যন্ত B-roll দেখাবে
- Original audio চলতে থাকবে
- Final assembly আপডেট হবে

---

### Prompt 5 — নির্দিষ্ট সময়ের সমস্যা ঠিক করা

**পরিস্থিতি:** Final video-তে 1:23.450 timestamp-এ একটা audible pop/click শোনা যাচ্ছে। সম্ভবত crossfade ঠিকমতো কাজ করেনি।

```
output/assembled.mp4-এর 1:23.450 এ একটা audio pop আছে।
এটা segment 4 ও 5-এর junction হওয়ার কথা।
নিচের steps follow করো:
1. segment_04_final.mp4 এর শেষ 0.5s এবং segment_05_final.mp4-এর প্রথম 0.5s extract করো
2. ওই junction point-এ waveform check করো — silence আছে কিনা দেখো
3. acrossfade duration বাড়িয়ে 0.008s (8ms) করো এই specific junction-এ
4. শুধু segment 4-5 re-join করো নতুন crossfade দিয়ে
5. Final assembly-তে পুরনো junction replace করো
6. 1:23 এর আশপাশে আর কোনো pop আছে কিনা scan করো
```

**প্রত্যাশিত আউটপুট:**
- Junction point-এ বড় crossfade (8ms)
- Click/pop মুক্ত smooth transition
- শুধু affected region re-render হবে

---

## ৩. ffmpeg ইনস্টলেশন গাইড

### Windows

**Option A — winget (Recommended):**
```powershell
winget install --id=Gyan.FFmpeg -e
# PATH স্বয়ংক্রিয়ভাবে যোগ হবে
ffmpeg -version
```

**Option B — Chocolatey:**
```powershell
# Chocolatey আগে ইনস্টল করো
choco install ffmpeg-full
```

**Option C — Manual (Full Build with all codecs):**
1. https://www.gyan.dev/ffmpeg/builds/ থেকে `ffmpeg-release-full.7z` ডাউনলোড করো
2. `C:\ffmpeg\` ফোল্ডারে extract করো
3. System PATH-এ `C:\ffmpeg\bin` যোগ করো:
   ```powershell
   [Environment]::SetEnvironmentVariable(
     "PATH",
     $env:PATH + ";C:\ffmpeg\bin",
     [EnvironmentVariableTarget]::Machine
   )
   ```
4. নতুন terminal খুলে verify করো: `ffmpeg -version`

### macOS

```bash
# Homebrew দিয়ে (সব codec সহ)
brew install ffmpeg

# অথবা specific options সহ
brew install ffmpeg --with-libvpx --with-libx265

# Verify
ffmpeg -version
ffprobe -version
```

### Linux (Ubuntu/Debian)

```bash
# Standard package (limited codecs)
sudo apt update && sudo apt install ffmpeg

# Full build (all codecs)
sudo add-apt-repository ppa:savoury1/ffmpeg4
sudo apt update
sudo apt install ffmpeg

# Build from source (maximum control)
sudo apt install build-essential yasm cmake libtool libc6 \
    libc6-dev unzip wget libnuma1 libnuma-dev libx264-dev

git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg/
cd ffmpeg
./configure --enable-gpl --enable-libx264 --enable-libx265 \
    --enable-libvpx --enable-libopus --enable-filter=drawtext
make -j$(nproc)
sudo make install
```

### Required Versions

| Component | Minimum Version | Recommended |
|-----------|----------------|-------------|
| ffmpeg | 4.4 | 6.0+ |
| ffprobe | 4.4 | 6.0+ |
| libx264 | 155 | 164+ |
| libx265 | 3.4 | 3.5+ |

### Additional Tools

```bash
# OpenCV (face detection)
pip install opencv-python numpy

# Hyperframes (alternative face detection)
npm install -g hyperframes

# SRT parsing
pip install srt

# Verify all tools
python3 -c "import cv2; print(f'OpenCV {cv2.__version__}')"
npx hyperframes --version
```

---

## ৪. B-roll ফাইল নামিং কনভেনশন

সঠিক নামিং ছাড়া automatic keyword matching কাজ করবে না।

### মূল ফরম্যাট

```
{primary_keyword}_{secondary_keyword}_{shot_type}_{duration}s.mp4
```

### ফিল্ড ব্যাখ্যা

| ফিল্ড | নিয়ম | উদাহরণ |
|-------|------|---------|
| `primary_keyword` | Script-এ যে বিষয়ের কথা বলা হচ্ছে | `coding`, `coffee`, `phone` |
| `secondary_keyword` | Context বা descriptor | `screen`, `morning`, `scroll` |
| `shot_type` | Framing type | `closeup`, `wide`, `medium`, `overhead`, `handheld` |
| `duration` | ক্লিপের দৈর্ঘ্য সেকেন্ডে (2 digit) | `05`, `08`, `12`, `20` |

### উদাহরণ ফাইল নামসমূহ

```
assets/broll/
├── coding_screen_closeup_08s.mp4      ← "কোডিং" কীওয়ার্ড
├── coding_keyboard_hands_05s.mp4      ← "কোডিং" alternate
├── coffee_morning_wide_10s.mp4        ← "কফি" কীওয়ার্ড
├── coffee_steam_macro_06s.mp4         ← "কফি" alternate
├── phone_scroll_hands_12s.mp4         ← "ফোন" কীওয়ার্ড
├── phone_notification_closeup_04s.mp4 ← "ফোন" alternate
├── city_traffic_wide_15s.mp4          ← "শহর/ট্রাফিক" কীওয়ার্ড
├── desk_setup_overhead_08s.mp4        ← "ডেস্ক/সেটাপ" কীওয়ার্ড
├── notebook_writing_medium_07s.mp4    ← "নোটবুক/লেখা" কীওয়ার্ড
└── plant_window_bokeh_10s.mp4         ← "প্রকৃতি/উইন্ডো" কীওয়ার্ড
```

### sync_map.json ফরম্যাট

```json
{
  "segments": [
    {
      "segment_id": 3,
      "srt_start": 45.2,
      "srt_end": 62.8,
      "broll_keywords": ["coding", "screen"],
      "broll_file": null,
      "use_broll": false
    },
    {
      "segment_id": 5,
      "srt_start": 98.4,
      "srt_end": 115.0,
      "broll_keywords": ["phone", "scroll"],
      "broll_file": null,
      "use_broll": false
    }
  ]
}
```

### নামিং নিয়ম (DO/DON'T)

| ✅ করো | ❌ করো না |
|--------|----------|
| lowercase শুধু | CamelCase বা UPPERCASE |
| underscore `_` separator | space বা hyphen `-` |
| English keywords | Bengali keywords (matching fails) |
| Duration সবসময় দাও | Duration বাদ দেওয়া |
| `.mp4` extension | `.mov`, `.avi` (convert করো আগে) |

---

## ৫. জুম ক্যালকুলেশন রেফারেন্স টেবিল

### Source Resolution অনুযায়ী Maximum Safe Zoom

| Source Resolution | Safe Max Zoom | Effective Resolution at Max Zoom | Notes |
|-------------------|--------------|----------------------------------|-------|
| 4K (3840×2160) | 200% (2.00) | 1920×1080 → নিরাপদ | 4K-তে aggressive zoom সম্ভব |
| 2.7K (2704×1520) | 170% (1.70) | 1590×893 → acceptable | |
| 1080p (1920×1080) | 115% (1.15) | 1669×939 → ঠিক আছে | **এর বেশি blur হবে** |
| 720p (1280×720) | 100% (1.00) | No zoom | 720p source-এ zoom করো না |

### Zoom Speed Reference (30fps)

| Increment/Frame | 1.0→1.2 Time | ব্যবহার |
|-----------------|-------------|---------|
| +0.0005 | ~13.3s | Very slow, cinematic |
| +0.0008 | ~8.3s | Slow, subtle |
| +0.0012 | ~5.6s | Medium |
| +0.0015 | ~4.4s | Default (recommended) |
| +0.0020 | ~3.3s | Fast, energetic |
| +0.0030 | ~2.2s | Very fast, jarring |

### Zoom Speed Reference (24fps)

| Increment/Frame | 1.0→1.2 Time | ব্যবহার |
|-----------------|-------------|---------|
| +0.0005 | ~16.7s | Very slow |
| +0.0010 | ~8.3s | Slow |
| +0.0015 | ~5.6s | Medium |
| +0.0020 | ~4.2s | Fast |

### Zoom Level → Use Case Mapping

| Zoom Level | Scale | কখন ব্যবহার |
|------------|-------|------------|
| Wide | 1.00 | Segment শুরু, general talk, setup |
| Slight push | 1.05 | Casual emphasis |
| Medium push | 1.10 | Clear emphasis |
| Close push | 1.15 | Strong emphasis, emotional peak |
| Maximum (1080p) | 1.15 | Absolute max for 1080p |
| Close (4K) | 1.20 | Standard close shot |
| Very close (4K) | 1.30 | Intense moment |

### Eye Position থেকে Zoom Center Calculate করা

```
# Face detected: face_x=400, face_y=300, face_w=280, face_h=350
# Eye position (roughly 35% down from face top):
eye_x = face_x + (face_w / 2)     = 400 + 140  = 540
eye_y = face_y + (face_h * 0.35)  = 300 + 122  = 422

# zoompan x/y formula:
x = eye_x - (iw / zoom / 2)
y = eye_y - (ih / zoom / 2)

# At zoom=1.10, iw=1080, ih=1920:
x = 540 - (1080 / 1.10 / 2) = 540 - 490.9 = 49.1
y = 422 - (1920 / 1.10 / 2) = 422 - 872.7 = -450.7 → clamp to 0
```

---

## ৬. গ্রেডিং প্যারামিটার কাস্টমাইজেশন গাইড

### Base Filtergraph (Reference)

```bash
-vf "curves=all='0/0 0.1/0.06 0.5/0.53 0.9/0.96 1/1',\
     eq=saturation=0.88:gamma_r=1.02:gamma_b=0.97,\
     unsharp=3:3:0.3,\
     vignette=PI/4,\
     noise=alls=8:allf=t+u"
```

---

### Curves — Contrast কাস্টমাইজেশন

Curves format: `input/output` pairs (0.0 to 1.0)

**Flat/Minimal Contrast (vlog/neutral look):**
```
curves=all='0/0 0.5/0.5 1/1'
```

**Medium S-Curve (default):**
```
curves=all='0/0 0.1/0.06 0.5/0.53 0.9/0.96 1/1'
```

**Strong S-Curve (dramatic look):**
```
curves=all='0/0 0.15/0.05 0.5/0.55 0.85/0.97 1/1'
```

**Lifted Blacks (Instagram matte look):**
```
curves=all='0/0.05 0.1/0.10 0.5/0.55 0.9/0.96 1/1'
```
> `0/0.05` → pure black কে 5% উজ্জ্বল করে → "matte" effect

**Per-channel Control (Teal & Orange Grade):**
```
curves=r='0/0 0.5/0.58 1/1':g='0/0 0.5/0.50 1/1':b='0/0 0.5/0.42 1/1'
```
> Red বাড়ানো + Blue কমানো = Teal & Orange cinematic look

---

### EQ — Saturation, Gamma, Brightness

```bash
eq=saturation=S:gamma=G:gamma_r=GR:gamma_g=GG:gamma_b=GB:brightness=B:contrast=C
```

| Parameter | Range | Default | Effect |
|-----------|-------|---------|--------|
| `saturation` | 0.0–3.0 | 1.0 | 1.0=unchanged, 0=grayscale, 2.0=vivid |
| `gamma` | 0.1–10.0 | 1.0 | Overall gamma (brightness in midtones) |
| `gamma_r` | 0.1–10.0 | 1.0 | Red channel gamma |
| `gamma_g` | 0.1–10.0 | 1.0 | Green channel gamma |
| `gamma_b` | 0.1–10.0 | 1.0 | Blue channel gamma |
| `brightness` | -1.0–1.0 | 0.0 | Overall brightness offset |
| `contrast` | -1000–1000 | 1.0 | Overall contrast multiplier |

**Look Presets:**

| Look | saturation | gamma_r | gamma_b | brightness |
|------|-----------|---------|---------|------------|
| **Default (Warm Natural)** | 0.88 | 1.02 | 0.97 | 0.0 |
| **Cool/Moody** | 0.80 | 0.97 | 1.03 | -0.02 |
| **Vivid/Social** | 1.10 | 1.00 | 1.00 | 0.03 |
| **B&W Film** | 0.00 | 1.00 | 1.00 | 0.0 |
| **Vintage Warm** | 0.85 | 1.05 | 0.93 | 0.02 |
| **Clinical/Cool** | 0.90 | 0.96 | 1.04 | 0.0 |

---

### Unsharp — Sharpening কাস্টমাইজেশন

```bash
unsharp=luma_msize_x:luma_msize_y:luma_amount:chroma_msize_x:chroma_msize_y:chroma_amount
```

| Setting | Value | Use Case |
|---------|-------|----------|
| Minimal | `3:3:0.15` | 4K footage যা already sharp |
| Default | `3:3:0.30` | Standard 1080p |
| Medium | `5:5:0.50` | Slightly soft footage |
| Strong | `7:7:0.80` | Web camera or phone footage |
| **Max (no artifacts)** | `5:5:1.00` | Soft lens footage |

> ⚠️ `luma_amount > 1.5` → ringing artifacts দেখা যাবে। Avoid করো।

---

### Vignette — Strength কাস্টমাইজেশন

```bash
vignette=angle:x0:y0:mode:eval:dither:aspect
# Simple form:
vignette=PI/4          # Standard
vignette=PI/5          # Lighter
vignette=PI/3          # Stronger
vignette=PI/2.5        # Very strong (cinematic)
```

| Angle Value | Approx Effect |
|-------------|--------------|
| `PI/6` (30°) | Very subtle vignette |
| `PI/5` (36°) | Light vignette |
| `PI/4` (45°) | **Default — medium** |
| `PI/3` (60°) | Strong vignette |
| `PI/2` (90°) | Very strong, heavy corner darkening |

---

### Film Grain — কাস্টমাইজেশন

```bash
noise=alls=STRENGTH:allf=FLAGS
```

| Strength | Look |
|----------|------|
| `alls=4` | Barely perceptible, digital clean |
| `alls=6` | Subtle grain, preferred for interviews |
| `alls=8` | **Default — visible grain, filmic** |
| `alls=12` | Heavy grain, 35mm film look |
| `alls=20` | Very grainy, lo-fi aesthetic |

**Flags:**
- `t` = Temporal (per-frame random → organic motion)
- `u` = Uniform (even across frame)
- `t+u` = Both → most natural (default)
- সুধু `u` → grain pattern flickers too regularly

---

### Custom Look Presets — Complete Commands

**🎬 Cinematic Warm (Default):**
```bash
-vf "curves=all='0/0 0.1/0.06 0.5/0.53 0.9/0.96 1/1',\
     eq=saturation=0.88:gamma_r=1.02:gamma_b=0.97,\
     unsharp=3:3:0.3,vignette=PI/4,noise=alls=8:allf=t+u"
```

**🌊 Moody Cool:**
```bash
-vf "curves=all='0/0 0.15/0.05 0.5/0.52 0.85/0.97 1/1',\
     eq=saturation=0.78:gamma_r=0.96:gamma_b=1.04:brightness=-0.02,\
     unsharp=3:3:0.2,vignette=PI/3,noise=alls=10:allf=t+u"
```

**☀️ Bright & Vivid (Social Media):**
```bash
-vf "curves=all='0/0.03 0.5/0.55 0.9/0.97 1/1',\
     eq=saturation=1.08:gamma_r=1.01:gamma_b=0.99:brightness=0.03,\
     unsharp=3:3:0.4,vignette=PI/5,noise=alls=4:allf=t+u"
```

**🎞️ Vintage Film:**
```bash
-vf "curves=r='0/0.04 0.5/0.58 1/0.98':g='0/0.02 0.5/0.52 1/0.97':b='0/0.01 0.5/0.45 1/0.94',\
     eq=saturation=0.82:gamma_r=1.04:gamma_b=0.94,\
     unsharp=5:5:0.5,vignette=PI/3.5,noise=alls=14:allf=t+u"
```

**⬛ Monochrome/B&W:**
```bash
-vf "hue=s=0,\
     curves=all='0/0 0.1/0.05 0.5/0.55 0.9/0.97 1/1',\
     unsharp=5:5:0.6,vignette=PI/3,noise=alls=12:allf=t"
```

---

## Quick Reference Cheat Sheet

```bash
# ============================================
# PHASE 1: Silence Detection
# ============================================
ffmpeg -i clip.mp4 -af "silencedetect=n=-40dB:d=0.15" -f null - 2>&1 | grep silence_

# ============================================
# PHASE 2: Segment Extract (with 2-frame handle)
# ============================================
ffmpeg -i input.mp4 -ss [start-0.067] -to [end+0.067] -c copy segment_N.mp4

# ============================================
# PHASE 3: 3ms Crossfade Join
# ============================================
ffmpeg -i a.mp4 -i b.mp4 \
  -filter_complex "[0:a][1:a]acrossfade=d=0.003:c1=tri:c2=tri[aout];[0:v][1:v]concat=n=2:v=1:a=0[vout]" \
  -map "[vout]" -map "[aout]" joined.mp4

# ============================================
# PHASE 4: Colour Grade
# ============================================
ffmpeg -i input.mp4 \
  -vf "curves=all='0/0 0.1/0.06 0.5/0.53 0.9/0.96 1/1',eq=saturation=0.88:gamma_r=1.02:gamma_b=0.97,unsharp=3:3:0.3,vignette=PI/4,noise=alls=8:allf=t+u" \
  -c:v libx264 -crf 16 -preset slow graded.mp4

# ============================================
# PHASE 5: Zoom
# ============================================
ffmpeg -i graded.mp4 \
  -vf "zoompan=z='if(lte(on,1),1,min(z+0.0015,1.15))':x='iw/2-(iw/zoom/2)':y='ih/2-(ih/zoom/2)':d=1:s=1080x1920" \
  -c:v libx264 -crf 16 zoomed.mp4

# ============================================
# PHASE 7: Final Concat
# ============================================
ffmpeg -f concat -safe 0 -i concat_list.txt \
  -c:v libx264 -b:v 18000k -maxrate 20000k -bufsize 36000k \
  -c:a aac -b:a 320k -movflags +faststart \
  output/assembled.mp4
```
