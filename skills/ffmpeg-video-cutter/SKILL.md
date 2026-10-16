---
name: ffmpeg-video-cutter
description: |
  ffmpeg ব্যবহার করে AI-নির্দেশিত ভিডিও কাটিং, সিনেমাটিক কালার গ্রেডিং এবং
  ফেস-ট্র্যাকিং ডায়নামিক জুম বাস্তবায়ন করে। অডিও ওয়েভফর্ম বিশ্লেষণ করে
  মিলিসেকেন্ড নির্ভুলতায় কাট করে। SRT-ভিত্তিক ওয়ার্ড-লেভেল সিঙ্ক সাপোর্ট করে।
version: "2.0"
author: AI Systems Lab
---

# ffmpeg-video-cutter Skill

## Overview

এই স্কিলটি একটি সম্পূর্ণ AI-চালিত ভিডিও পোস্ট-প্রোডাকশন পাইপলাইন যা ffmpeg-কে কেন্দ্র করে গড়ে উঠেছে। এটি শুধু ভিডিও কাটে না — বরং একটি প্রফেশনাল এডিটরের মতো প্রতিটি ধাপ বিশ্লেষণ করে, সিদ্ধান্ত নেয় এবং সেই সিদ্ধান্ত বাস্তবায়ন করে।

**মূল সক্ষমতাসমূহ:**

1. **অডিও ওয়েভফর্ম বিশ্লেষণ (Audio Waveform Analysis):** Raw অডিও ডেটা থেকে silence intervals, breath sounds, এবং verbal fillers (উম, আহ, হুম) শনাক্ত করে মিলিসেকেন্ড-স্তরের নির্ভুলতায়। ffmpeg-এর `silencedetect` ফিল্টার ব্যবহার করে নির্দিষ্ট dB threshold-এর নিচে থাকা অংশগুলো চিহ্নিত করা হয়।

2. **SRT/Transcript-সিঙ্ক্রোনাইজড কাটিং:** SubRip (SRT) বা raw transcript থেকে word-level timestamps পার্স করে প্রতিটি কাট পয়েন্ট নির্ধারণ করে। **সতর্কতা:** transcript timestamps সাধারণত ০.২–০.৩৩ সেকেন্ড আগে দেখায়; তাই প্রতিটি টাইমস্ট্যাম্পকে waveform-এর বিরুদ্ধে যাচাই করা বাধ্যতামূলক।

3. **সিনেমাটিক কালার গ্রেডিং:** ffmpeg filtergraph ব্যবহার করে S-curve contrast, skin-warm tone, cool shadow, saturation reduction, film grain, এবং soft vignette প্রয়োগ করা হয় — প্রতিটি ক্লিপে হিস্টোগ্রাম-ভিত্তিক exposure matching সহ।

4. **ফেস-ট্র্যাকিং ডায়নামিক জুম:** OpenCV বা `npx hyperframes face-detect` দিয়ে প্রতিটি ফ্রেমে চোখের অবস্থান শনাক্ত করে `zoompan` ফিল্টারের মাধ্যমে smooth, bounce-মুক্ত push-in/pull-out অ্যানিমেশন তৈরি করে।

5. **B-roll Integration:** `assets/broll/` ডিরেক্টরি স্ক্যান করে script keyword-এর সাথে ফাইল নাম মিলিয়ে স্বয়ংক্রিয়ভাবে B-roll কাট করে।

6. **ফাইনাল অ্যাসেম্বলি:** filter_complex concat দিয়ে সমস্ত সেগমেন্ট জোড়া দিয়ে 1080×1920 (9:16), H.264, ~18 Mbps, AAC 320k আউটপুট তৈরি করে।

**এই স্কিলটি কখন ব্যবহার করবেন:**
- Long-form talking-head footage থেকে short-form social content তৈরি করতে
- Multiple takes থেকে সেরা মুহূর্ত বেছে নিতে
- ম্যানুয়াল এডিটিং ছাড়াই প্রফেশনাল গ্রেড ও জুম পেতে
- Transcript-driven editing workflow-এ

---

## Step-by-Step Cutting Process

### Phase 1: Audio Analysis

অডিও বিশ্লেষণ হলো পুরো পাইপলাইনের ভিত্তি। এখানে যে তথ্য সংগ্রহ হয়, তার উপর নির্ভর করে পরবর্তী সমস্ত ধাপ।

#### ধাপ ১.১ — Silence Detection

```bash
# -40dB threshold এবং minimum 0.15 second duration দিয়ে silence detect করো
ffmpeg -i clip.mp4 \
  -af "silencedetect=n=-40dB:d=0.15" \
  -f null - 2>&1 | grep "silence_"
```

**আউটপুট উদাহরণ:**
```
silence_start: 4.23104
silence_end: 4.68291 | silence_duration: 0.45187
silence_start: 7.11003
silence_end: 7.28944 | silence_duration: 0.17941
```

এই লগ থেকে প্রতিটি silence interval সংগ্রহ করো এবং একটি structured list তৈরি করো:

```json
[
  { "start": 4.231, "end": 4.682, "duration": 0.451, "type": "pause" },
  { "start": 7.110, "end": 7.289, "duration": 0.179, "type": "breath" }
]
```

**প্যারামিটার বিশ্লেষণ:**
- `n=-40dB` — ৪০ dB-এর নিচে অডিও কে silence হিসেবে চিহ্নিত করে। রুমের ambient noise বেশি হলে `-35dB` ব্যবহার করো।
- `d=0.15` — minimum ০.১৫ সেকেন্ড স্থায়ী silence-ই গণনা করবে, ছোট ছোট word gaps বাদ পড়বে।
- Verbal fillers (উম, আহ): এগুলো সাধারণত low-energy কিন্তু silence নয়। এদের জন্য আলাদা pass দরকার — `volumedetect` দিয়ে RMS levels চেক করো।

#### ধাপ ১.২ — Transcript/SRT টাইমস্ট্যাম্পের সাথে সংযোগ

SRT বা transcript থেকে পাওয়া প্রতিটি শব্দের timestamp-কে waveform silence list-এর বিরুদ্ধে যাচাই করো।

> **⚠️ CRITICAL WARNING:** Transcript timestamps সাধারণত actual audio থেকে ১/৩ সেকেন্ড (≈ 0.33s) আগে আসে। এটি Whisper, AssemblyAI, বা Google STT সবার ক্ষেত্রে প্রযোজ্য। **কখনো শুধু transcript timestamp দিয়ে কাট করবে না।** সর্বদা:
> 1. Transcript থেকে approximate cut point নাও
> 2. ±0.5s window-এর মধ্যে waveform-এ nearest silence খোঁজো
> 3. সেই silence point-এ কাট করো

**যাচাইয়ের অ্যালগরিদম (Python pseudocode):**

```python
def find_verified_cut_point(transcript_ts, silence_list, window=0.5):
    candidates = [
        s for s in silence_list
        if abs(s['start'] - transcript_ts) <= window
    ]
    if candidates:
        # nearest silence start to transcript timestamp
        best = min(candidates, key=lambda s: abs(s['start'] - transcript_ts))
        return best['start'] + 0.02  # 20ms padding after silence starts
    else:
        # fallback: shift transcript ts forward by avg drift
        return transcript_ts + 0.25
```

#### ধাপ ১.৩ — Waveform Export for Manual Verification

```bash
# Low-res grayscale waveform image export করো (visual inspection-এর জন্য)
ffmpeg -i clip.mp4 \
  -filter_complex "showwavespic=s=1920x200:colors=0x00aaff" \
  -frames:v 1 waveform.png
```

---

### Phase 2: Take Selection Implementation

`takes.md` ফাইলটি হলো AI বা human editor-এর নির্দেশনা যেখানে প্রতিটি সেগমেন্টের জন্য কোন take ব্যবহার করতে হবে তা লেখা থাকে।

#### ধাপ ২.১ — takes.md ফরম্যাট পড়া

`takes.md` ফাইলের প্রত্যাশিত ফরম্যাট:

```markdown
## Segment 1
- File: raw_take_02.mp4
- In: 00:00:12.450
- Out: 00:00:28.910
- Note: Best energy, clear diction

## Segment 2
- File: raw_take_01.mp4
- In: 00:01:03.200
- Out: 00:01:19.780
- Note: Natural pause after "তাহলে"
```

প্রতিটি সেগমেন্ট পার্স করে structured object তৈরি করো এবং trim কমান্ড চালাও।

#### ধাপ ২.২ — Segment Extraction with 2-Frame Handle

**2-frame handle কী এবং কেন?**  
প্রতিটি segment-এর শুরু ও শেষে অতিরিক্ত ২ ফ্রেম রাখা হয় যাতে পরবর্তী ধাপে crossfade বা dissolve করার সময় কোনো hard cut visible না হয়।

**Handle calculation:**
```
handle_seconds = 2 / fps
# 1080p 30fps → 2/30 = 0.0667 seconds
# 4K 24fps   → 2/24 = 0.0833 seconds
# 1080p 60fps → 2/60 = 0.0333 seconds
```

**Extraction command:**
```bash
# [start] এবং [end] Phase 1-এ verified cut points
# handle যোগ করো: start - handle_seconds, end + handle_seconds
ffmpeg -i raw_take_02.mp4 \
  -ss 00:00:12.383 \
  -to 00:00:28.977 \
  -c copy \
  segment_01.mp4
```

> **NOTE:** `-c copy` ব্যবহার করা হয় কারণ এই ধাপে re-encoding প্রয়োজন নেই। তবে `-c copy` দিয়ে keyframe-aligned cut হয়, ফলে ১-২ ফ্রেম offset হতে পারে। চূড়ান্ত assembly-তে এটি handle দিয়ে ঠিক হয়ে যাবে।

#### ধাপ ২.৩ — Cut Point Validation

প্রতিটি segment extraction-এর পরে:

```bash
# Segment duration verify করো
ffprobe -v error -show_entries format=duration \
  -of default=noprint_wrappers=1:nokey=1 segment_01.mp4
```

প্রত্যাশিত duration = (out - in) + (2 × handle) ± 0.1s tolerance।  
এর বাইরে হলে re-extract করো।

---

### Phase 3: Silence Removal

এই phase-এ প্রতিটি segment-এর ভেতরের অপ্রয়োজনীয় silence, breath, এবং verbal fillers বাদ দেওয়া হয়।

#### ধাপ ৩.১ — Internal Silence Detection

```bash
# প্রতিটি segment-এ silence scan করো
ffmpeg -i segment_01.mp4 \
  -af "silencedetect=n=-40dB:d=0.15" \
  -f null - 2>&1 | grep "silence_" > segment_01_silence.txt
```

#### ধাপ ৩.২ — Sub-segment Split এবং Reassemble

প্রতিটি silence interval-এ segment কেটে sub-segments তৈরি করো:

```bash
# উদাহরণ: segment_01 এ 3.21s-3.89s silence আছে
# sub-segment A: 0.000 → 3.143 (silence_start - 0.067s handle)
ffmpeg -i segment_01.mp4 -ss 0.000 -to 3.143 -c copy seg01_a.mp4

# sub-segment B: 3.957 → end (silence_end + 0.067s handle)  
ffmpeg -i segment_01.mp4 -ss 3.957 -c copy seg01_b.mp4
```

#### ধাপ ৩.৩ — Micro Audio Crossfade (3ms) — Click/Pop Prevention

দুটি sub-segment জোড়া দেওয়ার সময় **3 millisecond audio crossfade** বাধ্যতামূলক। এছাড়া junction point-এ audible click বা pop হবে।

```bash
ffmpeg -i seg01_a.mp4 -i seg01_b.mp4 \
  -filter_complex "
    [0:a]aformat=sample_fmts=fltp,atrim=0:3.143[a0];
    [1:a]aformat=sample_fmts=fltp[a1];
    [a0][a1]acrossfade=d=0.003:c1=tri:c2=tri[aout];
    [0:v][1:v]concat=n=2:v=1:a=0[vout]
  " \
  -map "[vout]" -map "[aout]" \
  seg01_joined.mp4
```

**কেন ৩ms?**  
- ১ms — অনেক ক্ষেত্রে click এখনো শোনা যায়
- ৫ms — সামান্য pitch artifact হতে পারে fast speech-এ
- ৩ms — perceptually transparent, no artifacts

#### ধাপ ৩.৪ — Verbal Filler Detection

উম, আহ, হুম, ইত্যাদি সাধারণত silence নয়, কিন্তু বাদ দিতে হবে। Transcript-এ এগুলো চিহ্নিত থাকলে সেই timestamps ব্যবহার করো। না থাকলে:

```bash
# Very low RMS regions detect করো (filler sounds)
ffmpeg -i segment_01.mp4 \
  -af "volumedetect" \
  -f null - 2>&1 | grep "mean_volume"
```

Transcript-এ `[UH]`, `[UM]`, `[HMM]` token থাকলে সেই word-interval সরিয়ে দাও।

---

### Phase 4: Colour Grading

কালার গ্রেডিং একটি সিনেমাটিক look তৈরি করে যা YouTube Shorts এবং Instagram Reels-এ high retention নিশ্চিত করে।

#### সম্পূর্ণ Filtergraph:

```bash
ffmpeg -i input_clean.mp4 \
  -vf "curves=all='0/0 0.1/0.06 0.5/0.53 0.9/0.96 1/1',\
       eq=saturation=0.88:gamma_r=1.02:gamma_b=0.97,\
       unsharp=3:3:0.3,\
       vignette=PI/4,\
       noise=alls=8:allf=t+u" \
  -c:v libx264 -crf 16 -preset slow \
  -c:a copy \
  graded.mp4
```

#### প্রতিটি ফিল্টারের বিস্তারিত ব্যাখ্যা:

**১. `curves=all='0/0 0.1/0.06 0.5/0.53 0.9/0.96 1/1'` — S-Curve Contrast**

এটি একটি ক্লাসিক S-curve যা shadows কে একটু গাঢ় এবং highlights কে একটু উজ্জ্বল করে।

| Input Value | Output Value | প্রভাব |
|-------------|-------------|--------|
| 0.0 (pure black) | 0.0 | কালো কালোই থাকে |
| 0.1 (dark shadow) | 0.06 | Shadows গাঢ় হয় → ক্রাশড ব্ল্যাক feel |
| 0.5 (midtone) | 0.53 | Midtones সামান্য উজ্জ্বল |
| 0.9 (bright highlight) | 0.96 | Highlights উজ্জ্বল → blown-out নয় |
| 1.0 (pure white) | 1.0 | সাদা সাদাই থাকে |

এই curve দিয়ে ফুটেজে "filmic depth" আসে — flat সোনি বা ক্যানন footage-কে cinema-grade look দেয়।

**২. `eq=saturation=0.88` — Saturation -12%**

Saturation 100% থেকে 88%-এ নামানো হয়। কারণ:
- Full saturation → overly "social media" look যা viewer-কে ক্লান্ত করে
- -12% → colors আরও naturalistic, skin tone আরও pleasing
- বিশেষত Bengali/South Asian skin tones-এ এই value সবচেয়ে ভালো কাজ করে

**৩. `eq=gamma_r=1.02` — Warm Skin (Gamma Red +2%)**

Red channel-এর gamma সামান্য বাড়ানো হয়:
- Skin tone warm হয় (orange-amber shift)
- Midtones-এ একটা golden hour feel আসে
- Cold/clinical look দূর হয়

**৪. `eq=gamma_b=0.97` — Cool Shadows (Gamma Blue -3%)**

Blue channel-এর gamma কমানো হয়:
- Shadows-এ blue tint কমে → shadows warmer হয়
- কিন্তু combined effect: warm highlights, slightly cooler shadows = split-tone look
- Hollywood blockbuster grading-এ এই pattern খুব সাধারণ

**৫. `unsharp=3:3:0.3` — Subtle Sharpening**

- `3:3` → 3×3 pixel kernel
- `0.3` → sharpening strength (0 = none, 1 = aggressive)
- এটি over-sharpened "YouTuber" look এড়িয়ে subtle detail enhancement করে
- চোখের পাপড়ি, চুল, এবং texture আরও defined হয়

**৬. `vignette=PI/4` — Soft Vignette**

- `PI/4` ≈ 45° → medium-strength vignette
- Corner থেকে ধীরে ধীরে center-এর দিকে ফেড করে
- Viewer-এর চোখ স্বাভাবিকভাবে center (face)-এর দিকে আকৃষ্ট হয়
- খুব strong vignette এড়াতে `PI/5` বা `PI/6` ব্যবহার করা যায়

**৭. `noise=alls=8:allf=t+u` — Film Grain**

- `alls=8` → সমস্ত channel-এ strength 8 (0-100 scale)
- `allf=t+u` → temporal + uniform noise (প্রতি ফ্রেমে আলাদা pattern = organic grain)
- Digital noise-এর বদলে analog film grain feel দেয়
- 8 হলো sweet spot — কম হলে অদৃশ্য, বেশি হলে distracting

#### Exposure Matching Across Takes

একাধিক take-এ আলোর তারতম্য থাকলে histogram comparison করো:

```bash
# প্রতিটি segment-এর histogram stats বের করো
ffprobe -v quiet -of json \
  -select_streams v:0 \
  -show_frames \
  -read_intervals "%+#5" \
  segment_01.mp4 | python3 -c "
import sys, json
data = json.load(sys.stdin)
# mean luminance calculate করো
for f in data['frames']:
    print(f.get('pkt_pts_time', 'N/A'))
"
```

অথবা `eq` ফিল্টারে `brightness` প্যারামিটার দিয়ে per-segment adjustment করো:

```bash
# Segment_02 যদি 0.05 unit বেশি bright হয়
-vf "eq=brightness=-0.05:saturation=0.88:gamma_r=1.02:gamma_b=0.97"
```

#### Ungraded Copy সংরক্ষণ

```bash
# Graded ভার্সন তৈরির আগে ungraded কপি রাখো
cp input_clean.mp4 output/ungraded_ref.mp4
```

**কেন?**
- Before/After "reveal" beat-এর জন্য (TikTok/Reels-এ very popular)
- Client review-এর জন্য
- গ্রেডিং পুনরায় করতে হলে re-encode এড়াতে

---

### Phase 5: Face Detection & Smart Zooms

#### ধাপ ৫.১ — Face/Eye Detection

**Option A — OpenCV (Python):**

```python
import cv2

cap = cv2.VideoCapture('graded.mp4')
face_cascade = cv2.CascadeClassifier(
    cv2.data.haarcascades + 'haarcascade_frontalface_default.xml'
)
eye_cascade = cv2.CascadeClassifier(
    cv2.data.haarcascades + 'haarcascade_eye.xml'
)

face_positions = []
frame_idx = 0

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break
    
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    faces = face_cascade.detectMultiScale(gray, 1.1, 4)
    
    for (x, y, w, h) in faces:
        eyes = eye_cascade.detectMultiScale(gray[y:y+h, x:x+w])
        if len(eyes) >= 2:
            eye_center_x = x + w // 2
            eye_center_y = y + int(h * 0.35)
            face_positions.append({
                'frame': frame_idx,
                'eye_x': eye_center_x,
                'eye_y': eye_center_y,
                'face_w': w
            })
    
    frame_idx += 1

cap.release()
```

**Option B — Hyperframes CLI:**

```bash
npx hyperframes face-detect graded.mp4 --output face_data.json
```

#### ধাপ ৫.২ — Zoom Level নির্ধারণ

| Zoom Level | Scale Factor | কখন ব্যবহার করবে |
|------------|-------------|-----------------|
| Wide shot | 1.00 (100%) | Segment শুরুতে, general talk |
| Close shot | 1.20 (120%) | Emphasis words, emotional moments |
| Maximum (1080p source) | 1.15 (115%) | Quality loss এড়াতে — ১০৮০p source-এ ১১৫%-এর বেশি নয় |

> **⚠️ 1080p Source Limit:** 1080×1920 সোর্স থেকে 120% zoom করলে effective resolution 900×1600 হয়ে যায়, যা upscale করলে ব্লারি দেখায়। তাই **1080p source-এ সর্বোচ্চ 115% zoom** নিরাপদ।

#### ধাপ ৫.৩ — Smooth Push-In Animation

**0.4 second ease-in-out push-in:**

```bash
ffmpeg -i graded.mp4 \
  -vf "zoompan=\
    z='if(lte(on,1),1,min(z+0.0015,1.2))':\
    x='iw/2-(iw/zoom/2)':\
    y='ih/2-(ih/zoom/2)':\
    d=1:\
    s=1080x1920,\
    setsar=1" \
  -c:v libx264 -crf 16 -preset slow \
  -c:a copy \
  zoomed.mp4
```

**প্যারামিটার ব্যাখ্যা:**

- `z='if(lte(on,1),1,min(z+0.0015,1.2))'`
  - `on` = output frame number
  - প্রথম ফ্রেমে zoom = 1.0 থেকে শুরু
  - প্রতি ফ্রেমে +0.0015 বাড়ে → 30fps এ 0.0015×30 = 0.045/sec rate
  - 1.0 থেকে 1.2 পর্যন্ত যেতে (0.2/0.045) ≈ **4.4 seconds** লাগে
  - দ্রুত push চাইলে `+0.003` করো → ≈ 2.2 seconds

- `x='iw/2-(iw/zoom/2)'` — horizontal center tracking (face-center)
- `y='ih/2-(ih/zoom/2)'` — vertical center tracking
- `d=1` — প্রতিটি output ফ্রেমে ১টি করে input ফ্রেম process করে (smooth)
- `s=1080x1920` — output size নির্দিষ্ট করে

**Face-centered zoom (detected eye position ব্যবহার করে):**

```bash
# eye_x=540, eye_y=420 ধরে (detected values)
-vf "zoompan=\
  z='if(lte(on,1),1,min(z+0.0015,1.15))':\
  x='540-(iw/zoom/2)':\
  y='420-(ih/zoom/2)':\
  d=1:\
  s=1080x1920"
```

#### ধাপ ৫.৪ — Zoom Timing Logic

```
Emphasis word detected (transcript) → push-in শুরু
Next cut point → pull-out (zoom 1.0-এ ফেরা)
```

**কোনো shake বা bounce নেই:** `zoompan` ফিল্টারের linear/eased interpolation নিশ্চিত করে smooth motion। যদি jitter দেখা যায়:

```bash
# Stabilize করো আগে, zoom করো পরে
-vf "vidstabdetect=stepsize=6:shakiness=5,\
     vidstabtransform=smoothing=30,\
     zoompan=..."
```

---

### Phase 6: B-roll Integration

#### ধাপ ৬.১ — B-roll Library Scan

```bash
# assets/broll/ ফোল্ডারের সমস্ত ভিডিও ফাইল list করো
find assets/broll/ -type f \( -name "*.mp4" -o -name "*.mov" \) | sort
```

**B-roll ফাইল নামিং কনভেনশন (required):**
```
{keyword}_{variant}_{duration}s.mp4
# উদাহরণ:
coding_screen_closeup_08s.mp4
coffee_morning_wide_05s.mp4
phone_scroll_hands_12s.mp4
```

#### ধাপ ৬.২ — Script-to-B-roll Matching

`script.txt` বা `sync_map.json` থেকে keyword extract করো:

```python
import json, os, re

with open('sync_map.json') as f:
    sync_map = json.load(f)

broll_files = os.listdir('assets/broll/')

for entry in sync_map['segments']:
    keywords = entry.get('broll_keywords', [])
    for keyword in keywords:
        matches = [f for f in broll_files if keyword.lower() in f.lower()]
        if matches:
            entry['broll_file'] = f"assets/broll/{matches[0]}"
            entry['use_broll'] = True
```

#### ধাপ ৬.৩ — B-roll Insertion

```bash
# Main footage-এর নির্দিষ্ট segment-এ B-roll ওভারলে করো
ffmpeg -i main_segment_03.mp4 -i assets/broll/coding_screen_closeup_08s.mp4 \
  -filter_complex "
    [1:v]scale=1080:1920:force_original_aspect_ratio=increase,
         crop=1080:1920,
         setsar=1[broll];
    [0:v][broll]overlay=0:0:enable='between(t,2.5,10.5)'[vout]
  " \
  -map "[vout]" -map "0:a" \
  segment_03_with_broll.mp4
```

> **NOTE:** B-roll-এও একই color grading filtergraph প্রযোজ্য এবং 1080×1920 safe zone মেনে চলতে হবে।

---

### Phase 7: Concatenation & Final Assembly

#### ধাপ ৭.১ — Segment List File তৈরি

```bash
# concat_list.txt তৈরি করো
cat > concat_list.txt << EOF
file 'segment_01_final.mp4'
file 'segment_02_final.mp4'
file 'segment_03_final.mp4'
EOF
```

#### ধাপ ৭.২ — Simple Concat (No Transition)

```bash
ffmpeg -f concat -safe 0 -i concat_list.txt \
  -c copy \
  output/assembled_raw.mp4
```

#### ধাপ ৭.৩ — Filter_Complex Concat with Audio Crossfade

```bash
ffmpeg \
  -i segment_01_final.mp4 \
  -i segment_02_final.mp4 \
  -i segment_03_final.mp4 \
  -filter_complex "
    [0:v][0:a][1:v][1:a][2:v][2:a]
    concat=n=3:v=1:a=1[vout][aout]
  " \
  -map "[vout]" \
  -map "[aout]" \
  -c:v libx264 -b:v 18000k -maxrate 20000k -bufsize 36000k \
  -c:a aac -b:a 320k \
  -movflags +faststart \
  output/assembled.mp4
```

#### ধাপ ৭.৪ — Video Dissolve (Optional, Emotional Transitions)

```bash
# Segment A → B তে 0.5s dissolve
ffmpeg -i seg_a.mp4 -i seg_b.mp4 \
  -filter_complex "
    [0:v]trim=0:duration_a,setpts=PTS-STARTPTS[v0];
    [1:v]trim=0:duration_b,setpts=PTS-STARTPTS[v1];
    [v0][v1]xfade=transition=fade:duration=0.5:offset=fade_start[vout];
    [0:a][1:a]acrossfade=d=0.5[aout]
  " \
  -map "[vout]" -map "[aout]" \
  transition_out.mp4
```

---

## Output Specs

| Parameter | Value | বিস্তারিত |
|-----------|-------|----------|
| File | `output/assembled.mp4` | Primary deliverable |
| Ungraded ref | `output/ungraded_ref.mp4` | Before/after comparison |
| Resolution | 1080 × 1920 | 9:16 vertical (mobile-first) |
| Frame Rate | Source-matched | 24/30/60fps যা সোর্সে আছে |
| Video Codec | H.264 (libx264) | Universal compatibility |
| Video Bitrate | ~18 Mbps (18,000k) | High quality, reasonable file size |
| Max Bitrate | 20 Mbps | Buffer overflow এড়াতে |
| Buffer Size | 36 MB | 2× maxrate = standard |
| Audio Codec | AAC | Universal compatibility |
| Audio Bitrate | 320 kbps | Maximum AAC quality |
| Container | MP4 (faststart) | Streaming-optimized |
| Color Space | yuv420p | Maximum compatibility |

**চূড়ান্ত encode command:**

```bash
ffmpeg -i assembled_raw.mp4 \
  -c:v libx264 \
  -b:v 18000k \
  -maxrate 20000k \
  -bufsize 36000k \
  -vf "format=yuv420p" \
  -c:a aac \
  -b:a 320k \
  -movflags +faststart \
  -metadata title="Assembled Video" \
  output/assembled.mp4
```

---

## SRT Integration

### SRT ফরম্যাট বিশ্লেষণ

Standard SRT format:

```
1
00:00:01,240 --> 00:00:03,890
আমরা আজকে কথা বলব

2
00:00:03,920 --> 00:00:07,450
ffmpeg-এর সবচেয়ে শক্তিশালী ফিচারগুলো নিয়ে
```

### Word-Level Timestamp Parse করার নিয়ম

Whisper-generated word-level SRT (extended format):

```
1
00:00:01,240 --> 00:00:01,680
আমরা

2
00:00:01,700 --> 00:00:02,140
আজকে

3
00:00:02,180 --> 00:00:02,890
কথা
```

**Python SRT Parser:**

```python
import re
from dataclasses import dataclass
from typing import List

@dataclass
class SRTEntry:
    index: int
    start_sec: float
    end_sec: float
    text: str

def parse_srt(filepath: str) -> List[SRTEntry]:
    entries = []
    with open(filepath, encoding='utf-8') as f:
        content = f.read()
    
    blocks = content.strip().split('\n\n')
    time_pattern = re.compile(
        r'(\d{2}):(\d{2}):(\d{2}),(\d{3})\s+-->\s+'
        r'(\d{2}):(\d{2}):(\d{2}),(\d{3})'
    )
    
    for block in blocks:
        lines = block.strip().split('\n')
        if len(lines) < 3:
            continue
        
        idx = int(lines[0].strip())
        m = time_pattern.match(lines[1])
        if not m:
            continue
        
        h1,m1,s1,ms1,h2,m2,s2,ms2 = map(int, m.groups())
        start = h1*3600 + m1*60 + s1 + ms1/1000
        end   = h2*3600 + m2*60 + s2 + ms2/1000
        text  = ' '.join(lines[2:]).strip()
        
        entries.append(SRTEntry(idx, start, end, text))
    
    return entries
```

### SRT থেকে Cut Points বের করার অ্যালগরিদম

```python
def extract_cut_points(
    srt_entries: List[SRTEntry],
    silence_list: List[dict],
    min_segment_duration: float = 2.0,
    max_segment_duration: float = 45.0
) -> List[dict]:
    """
    SRT entries এবং silence intervals মিলিয়ে optimal cut points বের করে।
    
    Rules:
    1. প্রতিটি sentence boundary-তে cut candidate তৈরি করো
    2. ওই boundary-র ±0.5s window-এ silence খোঁজো
    3. Silence থাকলে সেখানে কাটো, না থাকলে transcript ts + 0.25s drift correction
    4. Segment duration MIN/MAX চেক করো
    """
    cut_points = []
    
    for i, entry in enumerate(srt_entries[:-1]):
        # Sentence end candidate
        candidate_ts = entry.end_sec
        
        # ±0.5s window-এ silence খোঁজো
        window_start = candidate_ts - 0.5
        window_end   = candidate_ts + 0.5
        
        nearby_silences = [
            s for s in silence_list
            if s['start'] >= window_start and s['start'] <= window_end
        ]
        
        if nearby_silences:
            # Nearest silence শুরুতে কাটো
            best = min(
                nearby_silences,
                key=lambda s: abs(s['start'] - candidate_ts)
            )
            verified_ts = best['start'] + 0.020  # 20ms padding
        else:
            # Drift correction: +0.25s
            verified_ts = candidate_ts + 0.25
        
        cut_points.append({
            'index': i,
            'srt_ts': candidate_ts,
            'verified_ts': verified_ts,
            'text_before': entry.text,
            'text_after': srt_entries[i+1].text
        })
    
    return cut_points
```

---

## Common ffmpeg Errors & Fixes

### Error 1: Pixel Format Mismatch

**Error message:**
```
[libx264 @ ...] Specified pixel format yuv444p is not supported by the libx264 encoder.
Incompatible pixel format 'yuv444p' for codec 'libx264', auto-selecting format 'yuv420p'
```

**কারণ:** Source footage-এ yuv444p বা অন্য pixel format আছে, কিন্তু H.264 শুধু yuv420p বা yuv422p সাপোর্ট করে।

**Fix:**
```bash
ffmpeg -i input.mp4 \
  -vf "format=yuv420p" \
  -c:v libx264 -crf 18 \
  output.mp4
```

অথবা filtergraph-এর শেষে যোগ করো:
```
-vf "...[other filters]...,format=yuv420p"
```

---

### Error 2: Audio Sync Drift

**লক্ষণ:** ভিডিওর ১-২ মিনিট পরে অডিও ০.৫-১.৫ সেকেন্ড পিছিয়ে যায়।

**কারণ:**
- `-c copy` দিয়ে concat করলে variable frame rate (VFR) footage misalign হয়
- Sample rate mismatch (48000Hz vs 44100Hz)

**Fix 1 — Constant Frame Rate নিশ্চিত করো:**
```bash
# VFR → CFR convert করো প্রথমে
ffmpeg -i input.mp4 \
  -vf "fps=30" \
  -c:v libx264 -crf 18 \
  -c:a aac -ar 48000 \
  input_cfr.mp4
```

**Fix 2 — Audio Sample Rate Normalize:**
```bash
ffmpeg -i input.mp4 \
  -c:v copy \
  -af "aresample=48000,aformat=sample_fmts=fltp" \
  -c:a aac -b:a 320k \
  normalized.mp4
```

**Fix 3 — Async হলে Manual Delay:**
```bash
# অডিও ০.৫ সেকেন্ড পিছিয়ে গেলে
ffmpeg -i drifted.mp4 \
  -itsoffset 0.5 -i drifted.mp4 \
  -map 0:v -map 1:a \
  -c copy \
  fixed.mp4
```

---

### Error 3: Codec Compatibility Issues

**Error message:**
```
Encoder libx265 not found. Try the codec 'hevc_nvenc' or 'hevc_amf' for hardware encoding.
```
অথবা:
```
moov atom not found
```

**Fix — libx265 নেই:**
```bash
# Fallback to libx264
-c:v libx264 -crf 18 -preset slow
```

**Fix — moov atom (corrupted/incomplete file):**
```bash
# qt-faststart দিয়ে মেরামত করো
ffmpeg -i corrupted.mp4 -c copy -movflags +faststart repaired.mp4
```

**Fix — Codec mismatch in concat:**
```bash
# সব segment একই codec-এ re-encode করো আগে
for f in segment_*.mp4; do
  ffmpeg -i "$f" \
    -c:v libx264 -crf 18 \
    -c:a aac -ar 48000 \
    "normalized_${f}"
done
```

**Fix — H.264 Profile সমস্যা (পুরনো device):**
```bash
# Baseline profile দিয়ে maximum compatibility
ffmpeg -i input.mp4 \
  -c:v libx264 -profile:v baseline -level 3.1 \
  -c:a aac \
  compatible_output.mp4
```

---

### Error 4: zoompan Filter OOM (Out of Memory)

**কারণ:** `zoompan` ফিল্টার internally বড় frame buffer তৈরি করে।

**Fix:**
```bash
# d=1 ব্যবহার করো (প্রতি ফ্রেমে process, buffer জমা করে না)
zoompan=z='...':d=1:s=1080x1920
```

---

### Error 5: concat Timestamp Gap

**Error:**
```
DTS 12345, next:12300, invalid dropping
```

**Fix:**
```bash
ffmpeg -f concat -safe 0 \
  -i concat_list.txt \
  -c copy \
  -fflags +genpts \
  output.mp4
```
`-fflags +genpts` → নতুন করে PTS generate করে, gaps ঠিক করে।
