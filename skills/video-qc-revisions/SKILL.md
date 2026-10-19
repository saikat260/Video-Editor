---
name: video-qc-revisions
description: |
  ফাইনাল রেন্ডারের আগে 9-দফা QC অডিট এবং 8টি স্পেশালাইজড রিভিশন প্রম্পট।
  প্রতিটি রিভিশন প্রম্পট copy-paste-ready, ধাপে ধাপে কাজ করে।
  SRT ফাইল থাকলে বাংলা/মাল্টিলিঙ্গুয়াল ক্যাপশন যাচাই ও ভিজ্যুয়াল বৈচিত্র্য অডিট করে।
version: "2.0"
author: AI Systems Lab
---

# video-qc-revisions

## Overview

এই স্কিলটি একটি সম্পূর্ণ ভিডিও কোয়ালিটি-কন্ট্রোল (QC) পাইপলাইন এবং রিভিশন সিস্টেম।
ফাইনাল রেন্ডার করার আগে প্রতিটি ভিডিও প্রজেক্টে এই ৯-দফা QC চেকলিস্ট রান করতে হবে।
চেকলিস্টে কোনো পয়েন্ট FAIL হলে, সংশ্লিষ্ট রিভিশন প্রম্পটটি সরাসরি AI-তে paste করো।
প্রতিটি রিভিশন প্রম্পট standalone ও copy-paste-ready — আলাদা কোনো ব্যাখ্যা ছাড়াই কাজ করে।
SRT / VTT ক্যাপশন ফাইল থাকলে বাংলা ও মাল্টিলিঙ্গুয়াল টেক্সটের জন্য বিশেষ QC স্টেপও আছে।

মূল নীতি:
- **একবারে একটি রিভিশন প্রম্পট** — পাঠাও, প্রিভিউ দেখো, তারপর পরেরটা।
- **QC log** সবসময় JSON ফর্ম্যাটে সেভ করো যাতে audit trail থাকে।
- **ব্র্যান্ড config** (নাম, রঙ, ফন্ট, ট্যাগলাইন) আলাদা ফাইলে রাখো এবং প্রতিটি প্রম্পটে রেফার করো।

---

## The 9-Point Final QC Checklist

ফাইনাল রেন্ডারের আগে নিচের ৯টি পয়েন্ট ক্রমানুসারে চেক করো। প্রতিটি পয়েন্টের পাশে কমান্ড / টুল এবং JSON লগ ফর্ম্যাট দেওয়া আছে।

---

### 1. Sync Check — সব ইফেক্ট শব্দ ২ ফ্রেমের মধ্যে কিনা যাচাই

**কী চেক করতে হবে:**
প্রতিটি অন-স্ক্রিন ভিজুয়াল ইভেন্ট (cut, transition, title card appear, graphic pop) এবং তার সাথে সংযুক্ত SFX / whoosh / ding / impact sound-এর মধ্যবর্তী গ্যাপ যেন ≤ 2 frames হয়।
ফ্রেম রেট 24fps হলে 2 frames = ~83ms। 30fps হলে 2 frames = ~67ms।
Audio waveform-এর transient peak এবং ভিজুয়াল ইভেন্টের টাইমকোড পাশাপাশি রেখে তুলনা করো।

**Command / Tool:**
```bash
# Step 1: ভিডিও থেকে প্রতিটি ফ্রেম বের করো (I-frames only for speed)
ffmpeg -i input.mp4 -vf "select=eq(pict_type\,I)" -vsync vfr frames/frame_%04d.png

# Step 2: অডিও ট্র্যাকের transient peaks বের করো
ffmpeg -i input.mp4 -af "silencedetect=noise=-30dB:d=0.05" -f null - 2>&1 | tee sync_log.txt

# Step 3: Manual review — প্রতিটি SFX-এর timecode নোট করো
# Timeline-এ marker দাও, visual event marker-এর সাথে align করো
```

**JSON লগ ফর্ম্যাট:**
```json
{
  "check": "sync",
  "status": "FAIL",
  "issues": [
    {
      "timecode": "00:00:12:04",
      "description": "Impact SFX fires 4 frames after graphic pop",
      "severity": "HIGH"
    }
  ]
}
```

**Pass criteria:** সব SFX ± 2 frames-এর মধ্যে। কোনো একটিও ৩+ frame দূরে থাকলে FAIL।

---

### 2. Face Coverage — মুখের উপরে কোনো টেক্সট/গ্রাফিক নেই

**কী চেক করতে হবে:**
যেকোনো ফ্রেমে যদি subject-এর মুখ, চোখ, বা ঠোঁটের উপরে কোনো lower-third, caption, animated graphic, shadow, বা overlay থাকে — সেটা FAIL।
বিশেষভাবে মনোযোগ দাও:
- Caption যদি 70% safe zone মানো তাহলেও মুখ নিচে হলে সমস্যা হতে পারে।
- Drop shadow এবং glow effect caption-এর বাইরেও জায়গা নেয় — shadow radius হিসাব করো।
- Animated graphic যদি mid-flight মুখের উপর দিয়ে যায় সেটাও FAIL।

**Command / Tool:**
```bash
# Step 1: Scene থেকে নির্দিষ্ট ফ্রেম extract করো
ffmpeg -i input.mp4 -ss 00:00:12 -vframes 1 face_check_0012.png

# Step 2: Face detection (Python + OpenCV)
python3 - <<'EOF'
import cv2
img = cv2.imread("face_check_0012.png")
face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + "haarcascade_frontalface_default.xml")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
faces = face_cascade.detectMultiScale(gray, 1.1, 4)
for (x, y, w, h) in faces:
    print(f"Face bbox: x={x}, y={y}, w={w}, h={h}")
EOF

# Step 3: উপরের bbox-এর সাথে overlay region তুলনা করো (manually বা script দিয়ে)
```

**JSON লগ ফর্ম্যাট:**
```json
{
  "check": "face_coverage",
  "status": "FAIL",
  "issues": [
    {
      "timecode": "00:00:12:00",
      "description": "Caption 'Subscribe Now' overlaps subject chin (y=620, face_bottom=640)",
      "shadow_radius_px": 8,
      "severity": "MEDIUM"
    }
  ]
}
```

**Pass criteria:** কোনো ফ্রেমে face bbox এবং text/graphic bbox overlap নেই (shadow radius সহ)।

---

### 3. Safe Zone — সব বিটের সমস্ত ফ্রেমের সেফ জোন যাচাই

**কী চেক করতে হবে:**
Action safe zone: ফ্রেমের প্রতিটি প্রান্ত থেকে 5% ভেতরে।
Title safe zone: প্রতিটি প্রান্ত থেকে 10% ভেতরে।
1920×1080 ফ্রেমে:
  - Action safe: 96px inset প্রতি দিক
  - Title safe: 192px inset প্রতি দিক
ইনস্টাগ্রাম / TikTok vertical (1080×1920) হলে top ও bottom safe zone আরও বড় কারণ UI overlay থাকে।

**Command / Tool:**
```bash
# ffprobe দিয়ে resolution confirm করো
ffprobe -v quiet -print_format json -show_streams input.mp4 | python3 -c "
import json, sys
data = json.load(sys.stdin)
for s in data['streams']:
    if s.get('codec_type') == 'video':
        print(f\"Width: {s['width']}, Height: {s['height']}\")
"

# ffmpeg দিয়ে safe zone overlay সহ preview render করো
ffmpeg -i input.mp4 -vf "
  drawbox=x=96:y=96:w=1728:h=888:color=yellow@0.5:t=2,
  drawbox=x=192:y=192:w=1536:h=696:color=green@0.5:t=2
" -t 5 safezone_preview.mp4
```

**JSON লগ ফর্ম্যাট:**
```json
{
  "check": "safe_zone",
  "status": "FAIL",
  "issues": [
    {
      "timecode": "00:00:45:12",
      "element": "Lower-third title",
      "position": "x=20, y=980",
      "violation": "Left edge outside action safe (x < 96)",
      "severity": "HIGH"
    }
  ]
}
```

**Pass criteria:** সমস্ত টেক্সট, লোগো, এবং গুরুত্বপূর্ণ গ্রাফিক্স title safe zone-এর ভেতরে।

---

### 4. Cut Quality — clipped word, mid-word cut, audio pop নেই

**কী চেক করতে হবে:**
প্রতিটি edit point-এ:
- কোনো শব্দ মাঝ থেকে cut হয়নি (speech waveform দেখো)
- কোনো "pop" বা "click" নেই — edit point-এ waveform zero-cross করছে কিনা
- ভয়েসের শেষ phoneme এবং edit point-এর মধ্যে ন্যূনতম ৩ frame silence থাকা ভালো
- J-cut / L-cut সঠিকভাবে fade-in / fade-out করছে কিনা

**Command / Tool:**
```bash
# Silence এবং abrupt cut detect করো
ffmpeg -i input.mp4 -af "silencedetect=noise=-45dB:d=0.02,ametadata=print:file=-" \
  -f null - 2>&1 | grep silence

# Audio pop/click detection (sox দিয়ে)
sox input_audio.wav -n stat 2>&1 | grep "Maximum amplitude"

# Zero-crossing check at edit point (Python)
python3 - <<'EOF'
import wave, struct
with wave.open("edit_region.wav", "rb") as f:
    frames = f.readframes(f.getnframes())
    samples = struct.unpack(f"{len(frames)//2}h", frames)
    # প্রথম ও শেষ 10 sample চেক করো
    print("First 10 samples:", samples[:10])
    print("Last 10 samples:", samples[-10:])
EOF
```

**JSON লগ ফর্ম্যাট:**
```json
{
  "check": "cut_quality",
  "status": "FAIL",
  "issues": [
    {
      "timecode": "00:01:03:08",
      "description": "Word 'import-ant' clipped, second syllable lost",
      "type": "mid_word_cut",
      "severity": "HIGH"
    },
    {
      "timecode": "00:02:15:00",
      "description": "Audio pop at cut, waveform not at zero-cross",
      "type": "audio_pop",
      "severity": "MEDIUM"
    }
  ]
}
```

**Pass criteria:** কোনো edit point-এ mid-word cut বা audible pop নেই।

---

### 5. Caption Spelling — নাম ও brand বানান যাচাই (config থেকে)

**কী চেক করতে হবে:**
Brand config ফাইল (brand.json) থেকে সমস্ত proper noun, brand name, product name, ট্যাগলাইন লোড করো।
SRT / VTT ফাইলের প্রতিটি লাইনে সেগুলোর বানান ঠিক আছে কিনা যাচাই করো।
বিশেষ মনোযোগ দাও:
- Capitalization (যেমন: "YouTube" না "Youtube")
- Hyphenation (যেমন: "e-commerce" না "ecommerce")
- Bengali ব্র্যান্ড নামের ইউনিকোড বানান

**Command / Tool:**
```bash
# brand.json উদাহরণ
cat brand.json
# {
#   "brand_names": ["YouTube", "TikTok", "Adobe Premiere", "After Effects"],
#   "product_names": ["ক্রিয়েটিভ স্যুট Pro", "AI Edit Engine"],
#   "taglines": ["Edit Smarter, Not Harder"]
# }

# Python দিয়ে SRT-এর বিরুদ্ধে brand name যাচাই
python3 - <<'EOF'
import json, re

with open("brand.json") as f:
    brand = json.load(f)

with open("captions.srt", encoding="utf-8") as f:
    srt_text = f.read()

issues = []
for name in brand["brand_names"] + brand["product_names"]:
    # case-insensitive match করো কিন্তু exact case না হলে flag করো
    pattern = re.compile(re.escape(name), re.IGNORECASE)
    for m in pattern.finditer(srt_text):
        if m.group() != name:
            issues.append({"found": m.group(), "expected": name, "pos": m.start()})

print(json.dumps(issues, ensure_ascii=False, indent=2))
EOF
```

**JSON লগ ফর্ম্যাট:**
```json
{
  "check": "caption_spelling",
  "status": "FAIL",
  "issues": [
    {
      "timecode": "00:00:34:00",
      "found": "Youtube",
      "expected": "YouTube",
      "severity": "HIGH"
    },
    {
      "timecode": "00:01:22:00",
      "found": "after effects",
      "expected": "After Effects",
      "severity": "HIGH"
    }
  ]
}
```

**Pass criteria:** সমস্ত brand name / proper noun brand config-এর সাথে exact match।

---

### 6. Audio Levels — -14 LUFS, -1 dBTP, voice চলাকালীন music ducked

**কী চেক করতে হবে:**
- **Integrated loudness:** -14 LUFS ± 1 LU (YouTube / standard streaming target)
- **True Peak:** -1 dBTP বা তার নিচে (কোনো clipping নেই)
- **Music ducking:** ভয়েস চলাকালীন background music যেন ভয়েসের চেয়ে ন্যূনতম -18 dB নিচে থাকে
- **Dynamic range (LRA):** 7–15 LU (খুব বেশি হলে শ্রোতা volume ঠিক রাখতে পারে না)

**Command / Tool:**
```bash
# Integrated LUFS, True Peak, LRA একসাথে মাপো
ffmpeg -i input.mp4 -af ebur128=framelog=verbose -f null - 2>&1 | tail -20

# শুধু summary দেখতে চাইলে
ffmpeg -i input.mp4 \
  -af "loudnorm=I=-14:TP=-1:LRA=11:print_format=json" \
  -f null - 2>&1 | grep -A 20 "{"

# Music ducking যাচাই: voice track ও music track আলাদা আলাদা measure করো
ffmpeg -i voice_only.wav -af ebur128 -f null - 2>&1 | grep "I:"
ffmpeg -i music_only.wav -af ebur128 -f null - 2>&1 | grep "I:"
# পার্থক্য ≥ 18 dB হওয়া উচিত ভয়েস চলাকালীন
```

**JSON লগ ফর্ম্যাট:**
```json
{
  "check": "audio_levels",
  "status": "FAIL",
  "issues": [
    {
      "measurement": "integrated_lufs",
      "measured": -11.2,
      "target": -14.0,
      "tolerance": 1.0,
      "severity": "HIGH"
    },
    {
      "measurement": "true_peak_dbtp",
      "measured": -0.2,
      "target": -1.0,
      "severity": "MEDIUM"
    },
    {
      "measurement": "music_duck_db",
      "measured": -12.0,
      "target": -18.0,
      "timecode": "00:00:45:00",
      "severity": "HIGH"
    }
  ]
}
```

**Pass criteria:** Integrated = -14 ±1 LUFS, True Peak ≤ -1 dBTP, music ducked ≥ -18 dB during voice.

---

### 7. Loop Check — শেষ ফ্রেম = frame 0 exactly

**কী চেক করতে হবে:**
লুপিং ভিডিও (social media bumper, background loop, animated logo) হলে শেষ ফ্রেম এবং প্রথম ফ্রেম pixel-perfect identical হতে হবে।
ভিজুয়াল glitch ছাড়াও audio loop point যাচাই করো — শেষ sample এবং প্রথম sample-এর মধ্যে কোনো pop নেই।

**Command / Tool:**
```bash
# প্রথম ও শেষ ফ্রেম extract করো
ffmpeg -i input.mp4 -vf "select=eq(n\,0)" -vframes 1 first_frame.png
ffmpeg -i input.mp4 -vf "select=eq(n\,last_frame_number)" -vframes 1 last_frame.png
# (last_frame_number = total_frames - 1, ffprobe দিয়ে বের করো)

# Pixel difference চেক করো (ImageMagick)
magick compare -metric PSNR first_frame.png last_frame.png diff.png 2>&1
# Output: inf = identical, কোনো সংখ্যা = পার্থক্য আছে

# Python দিয়ে pixel-level diff
python3 - <<'EOF'
from PIL import Image, ImageChops
import numpy as np

img1 = Image.open("first_frame.png")
img2 = Image.open("last_frame.png")
diff = ImageChops.difference(img1, img2)
arr = np.array(diff)
print(f"Max pixel diff: {arr.max()}")
print(f"Mean pixel diff: {arr.mean():.4f}")
# দুটোই 0 হলে PASS
EOF

# Audio loop point চেক
ffmpeg -i input.mp4 -af "aformat=sample_fmts=s16,atrim=end_sample=100" \
  -f s16le first_100_samples.raw - 2>/dev/null | xxd | head
ffmpeg -i input.mp4 -af "aformat=sample_fmts=s16,atrim=start_sample=-100" \
  -f s16le last_100_samples.raw - 2>/dev/null | xxd | tail
```

**JSON লগ ফর্ম্যাট:**
```json
{
  "check": "loop",
  "status": "FAIL",
  "issues": [
    {
      "type": "visual",
      "max_pixel_diff": 12,
      "mean_pixel_diff": 0.003,
      "description": "Last frame does not match frame 0 exactly",
      "severity": "HIGH"
    },
    {
      "type": "audio",
      "description": "Audio pop at loop point, samples not matching",
      "severity": "MEDIUM"
    }
  ]
}
```

**Pass criteria:** `max_pixel_diff = 0`, audio sample match at loop point।

---

### 8. Artifact Check — black frames, flashes, frozen frames, visual glitches

**কী চেক করতে হবে:**
- **Black frames:** কোনো unexpected black frame (≥ 1 frame duration)
- **Flash / strobe:** ≥ 3 Hz flash (accessibility + epilepsy risk — WCAG 2.3.1)
- **Frozen frames:** একই ফ্রেম ২+ consecutive times (video stutter)
- **Encoding artifacts:** macroblock corruption, green tint, pixel smearing
- **Color banding:** gradient-এ visible steps

**Command / Tool:**
```bash
# Black frame detection
ffmpeg -i input.mp4 -vf "blackdetect=d=0.05:pic_th=0.98" -f null - 2>&1 | grep black_

# Flash / strobe detection
ffmpeg -i input.mp4 -vf "flashdetect" -f null - 2>&1 | grep flash

# Frozen frame detection (duplicate frame detection)
ffmpeg -i input.mp4 -vf "mpdecimate,metadata=print:file=dup_log.txt" -f null - 2>&1

# Full artifact scan (একসাথে)
ffmpeg -i input.mp4 -vf "
  blackdetect=d=0.05:pic_th=0.98,
  flashdetect,
  mpdecimate
" -f null - 2>&1 | tee artifact_scan.log

# Encoding artifact visual check: একটি suspect timecode-এ ফ্রেম extract করো
ffmpeg -i input.mp4 -ss 00:01:30 -vframes 10 artifacts/frame_%02d.png
# ম্যানুয়ালি inspect করো macroblock, banding, smearing
```

**JSON লগ ফর্ম্যাট:**
```json
{
  "check": "artifacts",
  "status": "FAIL",
  "issues": [
    {
      "type": "black_frame",
      "timecode_start": "00:01:45:12",
      "timecode_end": "00:01:45:16",
      "duration_frames": 4,
      "severity": "HIGH"
    },
    {
      "type": "frozen_frame",
      "timecode": "00:02:10:00",
      "duplicate_count": 3,
      "severity": "MEDIUM"
    },
    {
      "type": "flash",
      "timecode": "00:00:58:00",
      "frequency_hz": 4.5,
      "wcag_violation": true,
      "severity": "CRITICAL"
    }
  ]
}
```

**Pass criteria:** কোনো black frame (unintentional), flash ≥ 3 Hz, frozen frame, বা visible encoding artifact নেই।

---

### 9. Motion Diversity & Anti-Template Check (ভিজ্যুয়াল বৈচিত্র্য ও অ্যান্টি-টেমপ্লেট অডিট)

**কী চেক করতে হবে:**
ভিডিওর সবগুলো মোশন গ্রাফিক্স একই রঙের বক্স বা একই ডার্ক কার্ডের মতো দেখতে হয়েছে কি না।
- যদি ৩টির বেশি সিনে একই ডার্ক পিল/কার্ড শেপ ব্যবহার করা হয় → **FAIL**।
- যদি `kinetic-typography` কোনো বক্সের ভেতরে সাধারণ টেক্সট হিসেবে থাকে → **FAIL**।
- যদি `svg-animation`-এ কোনো স্ট্রোক ড্রয়িং না থাকে → **FAIL**।
- যদি কোনো আর্কাইভাল ডকুমেন্টে রিয়েল পেপার টেক্সচার বা হাইলাইটার না থাকে → **FAIL**।

**Command / Tool:**
```bash
# সব জেনারেটেড সিন ডিরেক্টরি স্ক্যান করো:
ls -d build/scenes/*/
# রেন্ডার করা স্টিল ফ্রেমগুলো ইনস্পেক্ট করো:
ls output/stills/*.jpg
```

**JSON লগ ফর্ম্যাট:**
```json
{
  "check": "visual_monotony",
  "status": "FAIL",
  "issues": [
    {
      "scene": "scene_07",
      "error": "Kinetic typography enclosed in a generic dark card instead of borderless mask reveal",
      "remediation": "Re-render using pure SVG/CSS split-text with clip-path"
    }
  ]
}
```

**Pass criteria:** প্রতিটি দৃশ্য আলাদা ভিজ্যুয়াল পরিচয় (আর্কাইভাল ডকুমেন্ট, ভেক্টর মার্কার, লাইভ কাউন্টার, বড় মাস্কড টেক্সট) বহন করছে। ৩টির বেশি সিনে একই সাধারণ কার্ডের পুনরাবৃত্তি নেই।

---

### QC Pass/Fail Master Table

প্রতিটি চেকের পরে এই টেবিল আপডেট করো:

| পয়েন্ট | স্ট্যাটাস | সমস্যা |
|---|---|---|
| 1. Sync | PASS | — |
| 2. Face Coverage | FAIL | 0:12 তে caption মুখে |
| 3. Safe Zone | PASS | — |
| 4. Cut Quality | FAIL | 1:03 তে mid-word cut |
| 5. Caption Spelling | FAIL | "Youtube" → "YouTube" |
| 6. Audio Levels | PASS | — |
| 7. Loop Check | PASS | — |
| 8. Artifact Check | PASS | — |
| 9. Motion Diversity | FAIL | scene_07 তে কাইনেটিক টাইপোগ্রাফি কার্ডে আবদ্ধ |

সব PASS হলে ফাইনাল রেন্ডারে যাও। যেকোনো FAIL থাকলে সংশ্লিষ্ট রিভিশন প্রম্পট পাঠাও।

---

## The 8 Revision Prompts (Full, Copy-Paste Ready)

> **One-at-a-Time Rule:** একটি প্রম্পট পাঠাও → প্রিভিউ দেখো → PASS হলে পরেরটা।
> কখনো একসাথে একাধিক revision prompt পাঠাবে না।

---

### 01 — Push the Energy

```
You are editing a social-media short video. The current cut feels flat and slow.
Your job: push the energy of this edit UP without changing any of the footage or dialogue.

OPENING (first 3 seconds):
- Start on the loudest, most visually striking frame in the entire video.
- If there is an ambient/background sound, raise it +3 dB for the first beat, then duck it under voice.
- Add a single, sharp 1-frame white flash on frame 1 to punch the viewer in.

THROUGHOUT the video:
- Every time there is a natural pause in dialogue (> 0.3 seconds of silence), cut it by 50%.
- Every cut: add a 1-frame zoom-in (scale 100% → 103%) on the incoming clip.
- Every title card / caption appearance: add a 2-frame horizontal motion blur using the direction of the nearest camera movement.
- Music BPM: identify the BPM of the background track and ensure every major cut lands on a beat or half-beat.
- If there is no music, add a royalty-free electronic/hip-hop bed at -18 LUFS under the voice.

KEEP these rules:
- Do NOT remove or rearrange any dialogue.
- Do NOT change any title card text or captions.
- Do NOT exceed -1 dBTP on any output channel.
- Final integrated loudness target: -14 LUFS.

Output: revised edit decision list (EDL) with timecodes for every change made.
```

---

### 02 — Use Your Brand

```
You are a brand designer working on a social-media short video.
The current video uses default or placeholder colors, fonts, and styles.
Your job: replace everything visual with the brand identity pulled from brand.json.

STEP 1 — Remove all non-brand colors:
- Identify every color used in title cards, lower-thirds, backgrounds, and graphic elements.
- Replace each one with the nearest brand color from brand.json > colors.
- If a color has no close brand match, use the brand's primary color.

STEP 2 — Apply brand typography:
- Replace all on-screen fonts with brand.json > fonts.heading for titles.
- Replace all caption/body fonts with brand.json > fonts.body.
- Maintain existing font sizes; only change the typeface.

STEP 3 — Apply brand assets:
- Place the brand logo (brand.json > assets.logo_path) in the bottom-right corner inside title safe zone.
- Logo opacity: 80%. Do not animate the logo unless brand.json > logo_animated = true.
- Replace any generic outro card with the brand outro template (brand.json > assets.outro_template).

STEP 4 — Apply brand tagline:
- In the final 2 seconds, display brand.json > tagline in the brand heading font, centered, brand primary color.

STEP 5 — QC after rebrand:
- Confirm no non-brand color remains (tolerance: ΔE < 5 in CIELAB).
- Confirm logo is inside title safe zone on all target aspect ratios.
- Confirm all text passes WCAG AA contrast ratio (≥ 4.5:1) against its background.

Output: list of every visual element changed, with before/after color hex codes and font names.
```

---

### 03 — Motion on Every Line

```
You are a motion graphics artist. The current video has static title cards and captions.
Your job: add purposeful, on-brand motion to every line of text and every graphic element.

Use this table format to plan every motion beat before applying it:

| Timecode In | Timecode Out | Element | Motion Type | Direction | Duration | Easing |
|---|---|---|---|---|---|---|
| 00:00:03:00 | 00:00:03:12 | Title "Hook Line" | Slide in | Left → Center | 12 frames | Ease Out Expo |
| 00:00:03:12 | 00:00:05:00 | Title "Hook Line" | Hold | — | 48 frames | — |
| 00:00:05:00 | 00:00:05:08 | Title "Hook Line" | Slide out | Center → Right | 8 frames | Ease In Expo |

RULES for every motion beat:
1. Every line of dialogue caption: appear with a 4-frame vertical wipe from bottom, disappear with a 2-frame fade.
2. Every section title card: slide in from left (ease out expo), hold, slide out to right (ease in expo).
3. Every data point / stat graphic: count-up animation from 0 to final value, duration = (value / 1000) seconds, minimum 0.5 seconds.
4. Every logo appearance: scale from 95% → 100% over 6 frames (ease out back) on entry; no exit animation.
5. Every background graphic / texture: subtle parallax drift — move at 20% the speed of any camera movement.
6. No motion element should begin or end on the same frame as a cut — offset by ≥ 2 frames.

MOTION STYLE CONSTRAINTS:
- Maximum motion distance: 5% of frame dimension (no flying across the whole screen).
- No rotation > 3° unless it is a brand-specified spin.
- All easing curves: prefer exponential. No linear, no bounce (unless brand specifies).

Output: completed motion table for every element in the video, ready to hand off to an animator.
```

---

### 04 — Better Hook & Loop

```
You are a video editor specialising in social-media retention.
The current video has a weak opening hook and no satisfying loop.
Your job: redesign frame 0 and the final loop point.

HOOK — Frame 0 redesign:
- Frame 0 must be a frame that exists INSIDE the video (not a custom graphic).
- Choose the frame that has: highest visual contrast, subject making direct eye contact with camera, OR the moment of maximum action.
- The video must open on this frame — no black leader, no fade-in, no countdown.
- In the first 1.5 seconds, add a text hook overlay: one line, maximum 6 words, brand heading font, white text, black drop shadow (distance 2px, spread 0, blur 4px).
- The hook text must pose a question or make a bold claim. Generate 3 options and mark the strongest one.

LOOP — Final frame redesign:
- The final edit action must be a "pull back" — the camera (or graphic) appears to zoom out slightly (scale 100% → 97%) over the last 12 frames.
- On the very last frame, the scale must be 97% — this creates a visual "breath" that disguises the loop point.
- The audio must fade to -∞ dB over the last 8 frames using an exponential curve (not linear).
- If the video has background music, crossfade the music tail into the music head of the loop (12-frame crossfade).

VERIFICATION:
- Play frame (last-1) → frame 0 in a loop and confirm no visual jump.
- Confirm audio crossfade creates no pop or silence gap.

Output:
1. Recommended frame number for Hook frame 0 (with justification).
2. Three hook text options (mark best).
3. Exact keyframe values for the pull-back zoom.
4. Audio crossfade timecodes.
```

---

### 05 — Make a Moment Pop

```
You are a visual effects artist. The current video has one KEY moment that should be iconic.
Your job: identify that moment and make it visually unforgettable using three techniques.

STEP 1 — Identify the Moment:
- Watch (or read the transcript of) the video.
- The KEY moment is the one where: the subject makes their boldest statement, OR the most surprising visual occurs, OR the viewer is most likely to screenshot / share.
- State the timecode and a one-sentence justification.

STEP 2 — Cover Glance (Technique 1):
- At the KEY moment timecode, for exactly 12 frames:
  - Desaturate everything EXCEPT the subject by 80%.
  - Increase subject brightness by +15%.
  - Add a subtle vignette (radius 70%, feather 100%, opacity 40%) to the background.
- This is the "magazine cover" look. It should feel editorial, not cheap.

STEP 3 — Biggest Graphic (Technique 2):
- At the KEY moment timecode, animate in the most impactful text graphic in the video.
- This graphic should be 2× the size of any other text element.
- Entry: 6-frame scale from 120% → 100% with ease-out-back (slight overshoot).
- Color: brand primary color background, white text.
- Hold for exactly the duration of the key spoken phrase, then exit in 4 frames (fade).

STEP 4 — Subject Cutout (Technique 3):
- If the subject is clearly separated from the background (or can be rotoscoped in ≤ 10 minutes):
  - Cut out the subject on the KEY moment frame.
  - Place the biggest graphic BEHIND the subject.
  - This creates a "subject in front of title" depth effect.
  - If rotoscoping is not feasible, skip this step and note why.

Output:
1. KEY moment timecode + justification.
2. Exact parameter values for Cover Glance effect.
3. Graphic text content, size, position, timing.
4. Feasibility assessment for Subject Cutout + steps if feasible.
```

---

### 06 — Use Real Footage

```
You are a video editor creating a "before and after" social-media post.
The brief: show the same raw footage edited BEFORE (generic/default) and AFTER (your AI-assisted edit).
Your job: structure the video as a before/after reveal using real footage from the project.

STEP 1 — Pick the Old Video:
- Select a clip from the project's raw footage bin that is clearly unedited: no color grade, no music, no graphics.
- This clip should be 5–10 seconds long.
- Trim it to the most representative / interesting section.

STEP 2 — BEFORE Card:
- Cut to a full-screen card: black background, white text "BEFORE", brand body font, centered.
- Duration: 18 frames (0.75 seconds at 24fps).
- Add a subtle VHS / film grain texture overlay (opacity 20%) to signal "old / raw".
- Play the unedited clip immediately after the BEFORE card. Add no effects — raw only.

STEP 3 — Transition:
- Use a 6-frame horizontal wipe (left to right) to transition from the BEFORE clip to the AFTER clip.
- During the wipe, play a single "whoosh" SFX at -12 dB.

STEP 4 — CLAUDE RE-EDIT Card:
- Cut to a full-screen card: brand primary color background, white text "CLAUDE RE-EDIT", brand heading font, centered.
- Below the main text, in brand body font at 60% size: the date of the re-edit (auto-populated).
- Duration: 18 frames.
- Add a 3-frame white flash at the start of this card.

STEP 5 — AFTER clip:
- Play the same clip, now fully edited (color grade, music, graphics, captions all applied).
- At the end of the AFTER clip, freeze on the final frame for 12 frames.
- Fade to black over 8 frames.

Output: EDL (edit decision list) for the complete BEFORE/AFTER sequence with all timecodes.
```

---

### 07 — Tease the Prompt

```
You are creating a "prompt reveal" video that teases the AI prompt used to make this video.
The goal: show a terminal-style scroll of the prompt text in a way that is visually compelling and builds curiosity.

STEP 1 — Blur Strategy:
- The prompt text will be shown in a fake terminal window (dark background, monospace font, green or brand-accent text).
- Show the FIRST LINE of the prompt in full, unblurred.
- Show lines 2–5 with a progressive gaussian blur: line 2 = blur 2px, line 3 = blur 4px, line 4 = blur 8px, line 5 = blur 16px.
- Lines 6+ are fully blurred (blur 32px) and show only as visual "noise" rows.
- This communicates "there is more" without giving it all away.

STEP 2 — Scroll Animation:
- The terminal window scrolls upward at 40px/second (constant speed, no easing — terminals don't ease).
- Total scroll duration: (number of prompt lines × 0.4) seconds, minimum 3 seconds, maximum 8 seconds.
- The scroll starts 0.5 seconds after the terminal window fades in.

STEP 3 — STEP Headings Visible:
- Any line in the prompt that starts with "STEP" (e.g., "STEP 1 — ", "STEP 2 — ") must always remain UNBLURRED, regardless of position in the scroll.
- These act as "anchors" that reveal the structure of the prompt.
- Highlight STEP lines with a slightly brighter color (brand accent or pure white).

STEP 4 — Call to Action:
- After the scroll ends, the terminal window fades to 30% opacity.
- Over the top (on a transparent layer), display: "Full prompt in description ↓" in brand heading font, white, with a subtle pulsing opacity animation (90% → 100% → 90%, 1-second cycle).

STEP 5 — Audio:
- Under the entire sequence: a subtle keyboard typing SFX loop at -24 dB.
- On each STEP heading appearing in the scroll: a single soft keyboard "click" SFX at -18 dB.

Output: full animation spec with timecodes, blur values per line, scroll speed calculation, and SFX cue list.
```

---

### 08 — Fix a Take

```
You are a dialogue editor. One or more takes in this video are imperfect.
Your job: identify the best complete take for each line of dialogue and re-time the edit around it.

STEP 1 — List All Takes:
- Review all footage in the bin for this project.
- For each line of dialogue, list every recorded take:

| Line # | Line Text (first 5 words) | Take # | Timecode In | Timecode Out | Notes |
|---|---|---|---|---|---|
| 1 | "Hey welcome back to..." | Take 1 | 00:00:05:00 | 00:00:07:12 | Flubbed "welcome" |
| 1 | "Hey welcome back to..." | Take 2 | 00:01:02:00 | 00:01:04:08 | Clean |
| 1 | "Hey welcome back to..." | Take 3 | 00:02:30:00 | 00:02:32:15 | Best energy |

STEP 2 — Pick the Best Take:
- For each line, select the LAST COMPLETE take that has:
  1. No flubbed words or restarts.
  2. Best vocal energy (loudest, most confident).
  3. Most natural pacing (not rushed, not draggy).
- If two takes are equal on all criteria, pick the shorter one.
- Mark selected takes in the table with ✓.

STEP 3 — Re-time Everything:
- Build a new assembly cut using only the selected takes, in story order.
- For each cut point:
  - Trim 3 frames of handle from the outgoing clip tail.
  - Trim 3 frames of handle from the incoming clip head.
  - Apply a 3-frame audio crossfade at each edit point to prevent pops.
- After assembly, recalculate total video duration and note if it changed.

STEP 4 — Sync All Graphics:
- Every title card, caption, and graphic that was tied to a specific take must now be retimed to match the new edit.
- Caption SRT file must be regenerated from the new assembly (provide updated SRT).
- Every SFX cue must be re-synced to the new cut (provide updated SFX cue list).

Output:
1. Completed take selection table with ✓ marks.
2. New assembly EDL.
3. Updated SRT file.
4. Updated SFX cue list.
5. New total video duration.
```

---

## QC Prompt (Final, Copy-Paste Ready)

এটি সম্পূর্ণ QC প্রম্পট — রেন্ডারের আগে একবার পাঠাও:

```
You are a professional video quality-control (QC) engineer.
Review the following video project and run all 8 QC checks. Report results in the JSON log format specified for each check.

VIDEO PROJECT INFO:
- File: [INSERT FILENAME]
- Duration: [INSERT DURATION]
- Frame Rate: [INSERT FPS]
- Resolution: [INSERT RESOLUTION]
- Target Platform: [YouTube / Instagram / TikTok / Other]
- Has SRT Caption File: [Yes / No]
- Has Loop Requirement: [Yes / No]

RUN THE FOLLOWING 8 CHECKS IN ORDER:

CHECK 1 — SYNC
Verify every SFX / sound event is within ±2 frames of its corresponding visual event.
Tool: ffmpeg silence/transient detection. Report all violations.

CHECK 2 — FACE COVERAGE
Verify no text, graphic, or shadow overlaps the subject's face/eyes/mouth in any frame.
Tool: face detection + bbox comparison. Report all violations with timecodes.

CHECK 3 — SAFE ZONE
Verify all text, logos, and critical graphics are inside title safe zone (10% inset from all edges).
Tool: ffmpeg drawbox preview + manual verify. Report all violations.

CHECK 4 — CUT QUALITY
Verify no mid-word cuts, no audio pops, and no clipped phonemes at any edit point.
Tool: waveform + silence detection. Report all violations with timecodes.

CHECK 5 — CAPTION SPELLING
Load brand.json. Verify all brand names, product names, and proper nouns in the SRT match brand.json exactly (case-sensitive). Report all mismatches.

CHECK 6 — AUDIO LEVELS
Measure: Integrated LUFS, True Peak dBTP, LRA. Measure music/voice level ratio during voice sections.
Tool: ffmpeg ebur128 + loudnorm. Report all out-of-spec measurements.
Targets: -14 LUFS ±1, ≤ -1 dBTP, music ≥ -18 dB below voice.

CHECK 7 — LOOP (only if loop requirement = Yes)
Extract first and last frame. Compare pixel-perfect. Check audio loop point for pop.
Tool: ImageMagick compare + wave file inspect. Report any mismatch.

CHECK 8 — ARTIFACTS
Run black frame, flash, frozen frame, and encoding artifact detection.
Tool: ffmpeg blackdetect, flashdetect, mpdecimate. Report all findings.

OUTPUT FORMAT:
- One JSON object per check (8 total).
- After all 8: a master PASS/FAIL table in markdown format.
- If any check is FAIL: recommend which Revision Prompt (01–08) to run first.
- Sign off with: "QC COMPLETE — [X]/8 PASS — Ready for render: [YES / NO]"
```

---

## One-at-a-Time Rule

**কখনো একসাথে একাধিক রিভিশন প্রম্পট পাঠাবে না।**

সঠিক ওয়ার্কফ্লো:
1. QC prompt পাঠাও → JSON log দেখো
2. প্রথম FAIL পয়েন্টের রিভিশন প্রম্পট পাঠাও
3. পরিবর্তন apply করো, প্রিভিউ render করো
4. সেই নির্দিষ্ট QC check আবার রান করো
5. PASS হলে পরের FAIL পয়েন্টে যাও
6. সব PASS → ফাইনাল রেন্ডার

কেন? কারণ একটি রিভিশন (যেমন energy push) আরেকটি QC check (যেমন audio levels) পরিবর্তন করে দিতে পারে। তাই প্রতিটি রিভিশনের পরে QC re-run করা দরকার।

---

## Bengali/Multilingual Caption QC

SRT / VTT ফাইলে বাংলা বা মাল্টিলিঙ্গুয়াল টেক্সট থাকলে নিচের বিশেষ চেকগুলো চালাও:

### Bengali Unicode Encoding যাচাই

```bash
# SRT ফাইলটি UTF-8 BOM-less এনকোডিং-এ আছে কিনা নিশ্চিত করো
file -i captions.srt
# Output: text/plain; charset=utf-8

# BOM আছে কিনা চেক করো (BOM থাকলে কিছু player ভেঙে যায়)
python3 -c "
with open('captions.srt', 'rb') as f:
    first3 = f.read(3)
    if first3 == b'\xef\xbb\xbf':
        print('BOM FOUND — remove it')
    else:
        print('No BOM — OK')
"

# সমস্ত বাংলা character Unicode block U+0980–U+09FF-এর মধ্যে আছে কিনা
python3 -c "
with open('captions.srt', encoding='utf-8') as f:
    text = f.read()
for i, ch in enumerate(text):
    cp = ord(ch)
    if 0x0980 <= cp <= 0x09FF:
        pass  # Bengali block — OK
    elif cp > 127 and not (0x0980 <= cp <= 0x09FF):
        print(f'Non-Bengali non-ASCII char at position {i}: U+{cp:04X} ({ch!r})')
"
```

### হসন্ত ও অনুস্বার রেন্ডার চেক

বাংলা টাইপোগ্রাফিতে সবচেয়ে বেশি সমস্যা হয় এই character-গুলো নিয়ে:
- **হসন্ত (্):** U+09CD — ব্যঞ্জন সংযুক্তি তৈরি করে। ভুল font-এ "্" আলাদাভাবে দেখায়।
- **অনুস্বার (ং):** U+0982 — ঠিকমতো render না হলে "ং" missing বা displaced।
- **বিসর্গ (ঃ):** U+0983 — similar issue।
- **যুক্তাক্ষর:** যেমন ক্ষ, জ্ঞ — font-এ গ্লিফ না থাকলে ভেঙে দেখায়।

```bash
# Hasanta instances count করো
python3 -c "
with open('captions.srt', encoding='utf-8') as f:
    text = f.read()
hasanta = text.count('\u09CD')
anuswara = text.count('\u0982')
print(f'Hasanta (্): {hasanta}')
print(f'Anuswara (ং): {anuswara}')
# এই সংখ্যাগুলো content-এ expected count-এর সাথে মিলিয়ে নাও
"

# Font support যাচাই (HarfBuzz / fonttools দিয়ে)
python3 -c "
from fonttools.ttLib import TTFont
font = TTFont('path/to/your/font.ttf')
cmap = font.getBestCmap()
test_chars = [0x09CD, 0x0982, 0x0983, 0x0995, 0x09B7]  # ্ ং ঃ ক ষ
for cp in test_chars:
    status = 'OK' if cp in cmap else 'MISSING'
    print(f'U+{cp:04X}: {status}')
"
```

### SRT Timing Bengali Word Boundary Match

বাংলায় শব্দ সীমানা (word boundary) ইংরেজির মতো space-based নয় — বিশেষত যুক্তাক্ষর ও সন্ধির ক্ষেত্রে। SRT timing যেন বাংলা phoneme boundary-তে কাটে:

```python
# SRT timing check — প্রতিটি subtitle block-এর শেষ শব্দ সম্পূর্ণ কিনা
import re

def parse_srt(filepath):
    with open(filepath, encoding='utf-8') as f:
        content = f.read()
    blocks = re.split(r'\n\n+', content.strip())
    entries = []
    for block in blocks:
        lines = block.split('\n')
        if len(lines) >= 3:
            idx = lines[0].strip()
            time_match = re.match(
                r'(\d{2}:\d{2}:\d{2},\d{3}) --> (\d{2}:\d{2}:\d{2},\d{3})', lines[1])
            if time_match:
                text = ' '.join(lines[2:])
                entries.append({
                    'index': idx,
                    'start': time_match.group(1),
                    'end': time_match.group(2),
                    'text': text
                })
    return entries

entries = parse_srt('captions.srt')
issues = []
for e in entries:
    # শেষ character যদি হসন্ত (্) হয় তাহলে word cut হয়েছে
    if e['text'].rstrip().endswith('\u09CD'):
        issues.append({
            'index': e['index'],
            'timecode': e['end'],
            'issue': 'Subtitle ends on Hasanta — word boundary broken'
        })
    # শেষ character যদি vowel sign হয় (আ ই ঈ উ ঊ...)
    vowel_signs = range(0x09BE, 0x09CC+1)
    last_char = e['text'].rstrip()[-1] if e['text'].rstrip() else ''
    if last_char and ord(last_char) in vowel_signs:
        issues.append({
            'index': e['index'],
            'timecode': e['end'],
            'issue': f'Subtitle ends on vowel sign U+{ord(last_char):04X} — possible word split'
        })

import json
print(json.dumps(issues, ensure_ascii=False, indent=2))
```

### Bengali QC Checklist Summary

| চেক | টুল | Pass Criteria |
|---|---|---|
| UTF-8 encoding (no BOM) | `file -i`, Python | charset=utf-8, no BOM |
| Hasanta render | fonttools cmap | U+09CD in font cmap |
| Anuswar render | fonttools cmap | U+0982 in font cmap |
| Juktakkhor glyphs | fonttools, HarfBuzz | All conjuncts have glyphs |
| Word boundary timing | Python SRT parser | No subtitle ends on Hasanta or vowel sign |
| Brand name Bengali spelling | brand.json comparison | Exact Unicode match |
