---
name: audio-sound-designer
description: |
  ffmpeg ব্যবহার করে professional audio processing:
  ভয়েস ক্লিনআপ, মিউজিক ডাকিং, SFX sync, এবং -14 LUFS মাস্টারিং।
  SFX কখনো assets/sfx/ ফোল্ডারের বাইরে থেকে নেবে না।
version: "2.0"
author: AI Systems Lab
---

# audio-sound-designer

## Overview

এই স্কিলটি একটি নির্দিষ্ট সমস্যার সমাধান করে: অধিকাংশ AI-generated বা amateur video-তে যে chaotic, over-saturated audio পাওয়া যায় তা এড়িয়ে **restrained, cinematic sound design** তৈরি করা।

### Meme Edit বনাম Cinematic Sound Design

| বৈশিষ্ট্য | Meme Edit | Cinematic Sound Design |
|---|---|---|
| SFX ব্যবহার | প্রতিটি কাটে, প্রতিটি শব্দে | শুধু নির্দিষ্ট beat-এ, purposefully |
| Music volume | সর্বদা উচ্চ (-6 dBFS) | ভয়েসের উপর ভিত্তি করে dynamically ducked |
| Voice processing | Raw বা over-compressed | EBU R128 compliant, তিন-স্তরের cleanup |
| SFX layering | একসাথে একাধিক | কখনো দুটি SFX simultaneously নয় |
| Mastering | নেই বা clip করা | -14 LUFS, -1 dBTP true peak |
| অনুভূতি | হাইপার, chaotic | পরিচ্ছন্ন, focused, দর্শকের মনোযোগ ধরে রাখে |

এই স্কিল নিশ্চিত করে যে audio mix সবসময় **content-first**: ভয়েস সর্বোচ্চ অগ্রাধিকার পায়, মিউজিক emotional support দেয়, এবং SFX শুধু সুনির্দিষ্ট narrative moment-কে accent করে। কোনো element অন্যটিকে overwhelm করে না।

---

## Voice Processing Chain

ভয়েস হলো ভিডিওর মূল বার্তাবাহক। এই chain-টি তিনটি স্তরে কাজ করে: **cleanup → dynamics → loudness normalization**।

```bash
ffmpeg -i assembled.mp4 \
  -af "highpass=f=80,\
       anlmdn=s=0.0001,\
       acompressor=threshold=-18dB:ratio=3:attack=5:release=50,\
       loudnorm=I=-14:TP=-1:LRA=11" \
  voice_processed.mp4
```

### প্রতিটি ফিল্টারের বিস্তারিত ব্যাখ্যা

#### ১. `highpass=f=80`
**কী করে:** 80 Hz-এর নিচের সমস্ত frequency কেটে দেয়।

**কেন দরকার:**
- রেকর্ডিং সময় মাইক্রোফোন হাতে ধরলে যে low-frequency rumble তৈরি হয় (handling noise) সেটি সরায়
- ঘরের air conditioning, ফ্যান, ট্রাফিক — এই সব sub-80Hz ambient noise সরায়
- মানব কণ্ঠস্বরের মৌলিক frequency সাধারণত পুরুষের জন্য ~85 Hz এবং মহিলার জন্য ~165 Hz; তাই 80 Hz cut করলে voice intelligibility-তে কোনো ক্ষতি হয় না
- Result: কণ্ঠ পরিষ্কার, "boomy" মনে হয় না

#### ২. `anlmdn=s=0.0001`
**কী করে:** Non-Local Means Denoising algorithm ব্যবহার করে background noise কমায়।

**কেন দরকার:**
- `s=0.0001` একটি খুব gentle noise reduction strength — এটি voice artifact তৈরি করে না
- Traditional noise reduction (যেমন `afftdn`) অনেক সময় voice-কে "watery" বা robotic শোনায়; `anlmdn` non-destructive থাকে
- ঘরের constant hiss, হালকা হাওয়ার শব্দ, microphone self-noise সরায়
- **সতর্কতা:** `s` value 0.001-এর উপরে গেলে voice artifact শুরু হয়; সর্বদা 0.0001 থেকে শুরু করো

#### ৩. `acompressor=threshold=-18dB:ratio=3:attack=5:release=50`
**কী করে:** Dynamic range compression প্রয়োগ করে — জোরে অংশগুলি কমায়, শান্ত অংশগুলি আপেক্ষিকভাবে উঁচু হয়।

**প্রতিটি প্যারামিটার:**
- `threshold=-18dB`: -18 dBFS-এর উপরে signal থাকলে তবেই compressor সক্রিয় হবে
- `ratio=3`: প্রতি 3 dB বেশি input-এর জন্য মাত্র 1 dB output বাড়বে (3:1 ratio)
- `attack=5`: 5ms-এ compressor সম্পূর্ণ active হয় — transient (প্রথম consonant) preserve করে
- `release=50`: 50ms-এ compressor release করে — natural decay বজায় রাখে

**কেন দরকার:**
- বক্তা যখন উৎসাহী হয়ে জোরে বলেন বা ফিসফিস করেন, volume অনেক পরিবর্তন হয়
- Compressor এই variation সমতা করে দেয় — শ্রোতাকে volume বাড়াতে-কমাতে হয় না
- 3:1 ratio musical এবং natural শোনায়; 10:1 বা তার বেশি হলে "squashed" মনে হয়

#### ৪. `loudnorm=I=-14:TP=-1:LRA=11`
**কী করে:** EBU R128 standard অনুযায়ী integrated loudness normalize করে।

**প্রতিটি প্যারামিটার:**
- `I=-14`: Target integrated loudness -14 LUFS — YouTube, Spotify, এবং broadcast সবার জন্য safe
- `TP=-1`: True Peak maximum -1 dBTP — clipping এবং inter-sample peak সম্পূর্ণ এড়ায়
- `LRA=11`: Loudness Range 11 LU — dynamic range যুক্তিসঙ্গত রাখে, না বেশি flat না বেশি dynamic

**কেন -14 LUFS:**
- YouTube নিজেই -14 LUFS-এ normalize করে; আগে থেকে এই level-এ থাকলে platform আর কিছু করে না
- Podcast standard হলো -16 LUFS; -14 LUFS ঐ মাঝামাঝি যা সব platform-এ ভালো কাজ করে

---

## Music Ducking

Music ducking মানে ভয়েস থাকলে music volume স্বয়ংক্রিয়ভাবে কমানো এবং ভয়েস না থাকলে বাড়ানো। এটি mix-এর সবচেয়ে গুরুত্বপূর্ণ অংশ।

### Target Volume Levels

| পরিস্থিতি | Music Level | কারণ |
|---|---|---|
| ভয়েস চলছে | -18 to -22 dBFS | Voice intelligibility সর্বোচ্চ রাখতে |
| ভয়েস নেই (বিরতি, b-roll) | -12 dBFS | Emotional atmosphere বজায় রাখতে |
| Intro/Outro (কোনো ভয়েস নেই) | -10 to -8 dBFS | Engaging opening/closing |

### Ducking Parameters
- **Attack: 10ms** — ভয়েস শুরু হওয়ার পর 10ms-এ music দ্রুত কমে যায়; প্রাকৃতিক শোনায়
- **Release: 200ms** — ভয়েস শেষ হওয়ার পর 200ms ধরে music আবার উঠে আসে; abrupt jump এড়ায়

### ffmpeg Sidechain Compression

```bash
# ভয়েস ট্র্যাক দিয়ে মিউজিক control করা
ffmpeg -i music.wav -i voice_processed.mp4 \
  -filter_complex \
  "[0:a][1:a]sidechaincompress=\
    threshold=0.015:\
    ratio=4:\
    attack=10:\
    release=200:\
    level_sc=0.9\
  [music_ducked]" \
  -map "[music_ducked]" music_ducked.wav
```

**প্যারামিটার ব্যাখ্যা:**
- `threshold=0.015`: ভয়েস signal এই amplitude ছাড়িয়ে গেলে ducking শুরু হয়
- `ratio=4`: প্রতি 4 dB ভয়েস-এর জন্য music 1 dB কমে (4:1 aggressive ducking)
- `level_sc=0.9`: Sidechain (ভয়েস) signal sensitivity — 90% sensitivity

### মিউজিকের Clean Ending

ভিডিওর শেষ শব্দের পর music একটি **fade out** দিয়ে শেষ করো। Abrupt cut কখনো নয়।

```bash
# শেষ 3 সেকেন্ডে fade out
ffmpeg -i music_ducked.wav \
  -af "afade=t=out:st=TOTAL_DURATION_MINUS_3:d=3" \
  music_final.wav
```

---

## SFX Placement System

### SFX Source Rule — অলঙ্ঘনীয়

> **SFX শুধুমাত্র `assets/sfx/` ফোল্ডার থেকে নেওয়া হবে।** কোনো SFX ফাইল অন্য কোথাও থেকে generate বা download করা যাবে না।

### Approved SFX Map

| ফাইল | Beat Type | ব্যবহারের সময় |
|---|---|---|
| `assets/sfx/whoosh.wav` | Zoom in/out | ভিডিওতে zoom effect বা camera push-in/pull-out বিটে |
| `assets/sfx/click.wav` | Hard cut | দুটি clip-এর মধ্যে sharp cut বিটে; attention snap |
| `assets/sfx/pop.wav` | Waveform beat | Background music-এর strong beat বা bass hit-এর সাথে sync |
| `assets/sfx/riser.wav` | Reveal moment | "What it gave me back" বা যেকোনো dramatic reveal-এর ঠিক আগে |
| `assets/sfx/swoosh.wav` | Graphic slide-in | Lower-third, title card, বা graphic element slide করে আসার সময় |

### Timing Precision

- SFX এবং visual beat-এর মধ্যে **সর্বোচ্চ 2 frame** পার্থক্য অনুমোদিত
- 24fps ভিডিওতে 2 frame = ~83ms
- 30fps ভিডিওতে 2 frame = ~67ms
- Sync map-এ প্রতিটি word-এর start timestamp ব্যবহার করো

```bash
# নির্দিষ্ট timestamp-এ SFX মেশানো (উদাহরণ: 5.234s থেকে)
ffmpeg -i assembled.mp4 -i assets/sfx/whoosh.wav \
  -filter_complex \
  "[1:a]adelay=5234|5234,volume=-15dB[sfx]; \
   [0:a][sfx]amix=inputs=2:duration=first" \
  output_with_sfx.mp4
```

### SFX Volume Guideline

- SFX volume সর্বদা voice থেকে **-12 to -18 dBFS নিচে** রাখো
- `whoosh.wav`, `swoosh.wav`: -15 dBFS (subtle)
- `click.wav`, `pop.wav`: -12 dBFS (punchy কিন্তু overwhelming নয়)
- `riser.wav`: -18 dBFS (undercurrent, atmospheric)

### SFX না পাওয়া গেলে

> **যদি কোনো SFX ফাইল `assets/sfx/`-তে না থাকে:**
> - সেই beat-এ silence রাখো
> - Placeholder বা generated SFX কখনো ব্যবহার করো না
> - Log-এ note করো: `[SFX MISSING: whoosh.wav at 5.234s — beat left silent]`

এটি একটি quality-over-quantity decision: ভুল SFX কোনো SFX-এর চেয়ে সবসময় খারাপ।

---

## Audio Click Prevention

Audio clicks এবং pops তৈরি হয় যখন audio signal হঠাৎ করে শুরু বা শেষ হয় — waveform "zero crossing" ছাড়াই কাটা হলে।

### তিন-স্তরের Prevention

#### স্তর ১: Clip জোড়ায় Crossfade
```bash
# প্রতিটি cut-এ 3ms crossfade
ffmpeg -i clip_a.wav -i clip_b.wav \
  -filter_complex "acrossfade=d=0.003:c1=tri:c2=tri" \
  joined.wav
```
- `d=0.003`: 3 millisecond duration
- `c1=tri`, `c2=tri`: Triangular fade curve — সবচেয়ে natural শোনায়

#### স্তর ২: Volume Ramp
প্রতিটি audio segment-এর শুরুতে 1ms fade-in এবং শেষে 1ms fade-out:
```bash
ffmpeg -i segment.wav \
  -af "afade=t=in:st=0:d=0.001,afade=t=out:st=SEGMENT_END_MINUS_0.001:d=0.001" \
  segment_ramped.wav
```

#### স্তর ৩: Zero Crossing Check
FFmpeg-এ `aresample` filter zero-crossing-এ cut নিশ্চিত করে:
```bash
ffmpeg -i input.wav -af "aresample=async=1" output.wav
```

### কখন Click হওয়ার সম্ভাবনা বেশি
- দুটি clip join করার সময় যদি একটির ending এবং অন্যটির beginning-এ waveform magnitude বেশি থাকে
- SFX হঠাৎ শুরু হলে (fade-in ছাড়া)
- Volume automation sudden jump করলে

---

## Mastering

Mastering হলো final mix-এর পরে সমগ্র audio-কে একটি cohesive, platform-ready level-এ নিয়ে আসা।

### কখন Mastering করবে

> **সমস্ত ভয়েস, মিউজিক এবং SFX ট্র্যাক join এবং mix করার পরে।** কোনো individual track mastering করো না — শুধু final mix-এ।

### Mastering Chain

```bash
ffmpeg -i final_mix.mp4 \
  -af "loudnorm=I=-14:TP=-1:LRA=11:print_format=summary" \
  mastered_output.mp4
```

### Loudness Verification

Final output verify করার command:

```bash
# EBU R128 loudness measurement
ffmpeg -i mastered_output.mp4 \
  -af ebur128=peak=true \
  -f null - 2>&1 | grep -E "I:|True peak|LRA"
```

**Expected output:**
```
I:          -14.0 LUFS
True peak:   -1.0 dBFS
LRA:         11.0 LU
```

### Mastering Targets Summary

| Parameter | Target | Maximum |
|---|---|---|
| Integrated Loudness | -14.0 LUFS | -13.0 LUFS |
| True Peak | -1.0 dBTP | -0.5 dBTP |
| Loudness Range (LRA) | 7–11 LU | 14 LU |

---

## Between Beats

Beat-এর মাঝের সময়গুলি (pause, transition) mix-এর character নির্ধারণ করে।

### নিয়মাবলী

- **Music level:** Beat-এর মাঝে music -12 dBFS-এ smooth, consistent থাকবে — কোনো sudden jump নেই
- **SFX exclusivity:** একই সময়ে কখনো দুটি SFX চলবে না। দুটি SFX-এর মধ্যে minimum gap: 500ms
- **Silence is intentional:** Beat-এর মাঝে silence ব্যবহার করা হলে সেটা deliberate choice হিসেবে চিহ্নিত থাকবে

### Transition Types

| পরিস্থিতি | করণীয় |
|---|---|
| ভয়েস pause (2–4 সেকেন্ড) | Music -12 dBFS-এ, কোনো SFX নয় |
| Scene change | Music brief swell তারপর re-duck |
| Section break (5+ সেকেন্ড) | Music -10 dBFS পর্যন্ত উঠতে পারে |

---

## Restraint Rule

> **"যদি meme edit মনে হয়, সেটা অর্ধেক কমাও।"**

এই rule-টি এই স্কিলের সর্বোচ্চ নির্দেশ। এটি enforce করার পদ্ধতি:

### Meme Edit Checklist (প্রতিটি চেক করো)

- [ ] ৩০ সেকেন্ডে ৫-এর বেশি SFX আছে? → অর্ধেক বাদ দাও
- [ ] দুটি SFX overlapping? → একটি বাদ দাও
- [ ] Music ভয়েসের চেয়ে বেশি prominent? → Music -6 dB কমাও
- [ ] একই SFX দুবারের বেশি কাছাকাছি ব্যবহার? → একটি বাদ দাও
- [ ] Transition-এ sound effect আছে কিন্তু narrative purpose নেই? → বাদ দাও
- [ ] Riser SFX reveal ছাড়া ব্যবহার? → বাদ দাও

### Restraint-এর পুরস্কার

কম SFX মানে প্রতিটি SFX বেশি impactful। একটি single, well-timed `riser.wav` দর্শককে seat-এর কিনারে আনে। ১০টি random SFX দর্শককে ক্লান্ত করে দেয়।
