# audio-sound-designer — বিস্তারিত গাইড

## ১. স্কিলের পরিচয়

`audio-sound-designer` স্কিলটি AI-assisted video production-এর সবচেয়ে সূক্ষ্ম অংশটি সামলায়: **professional-grade audio post-production**। এটি নিছক volume balancing নয় — এটি একটি পূর্ণাঙ্গ audio pipeline যা নিম্নলিখিত কাজগুলি করে:

1. **Voice Cleanup:** Raw microphone recording থেকে noise, rumble, এবং dynamic variation সরায়
2. **Music Ducking:** ভয়েসের উপস্থিতি অনুযায়ী background music intelligently control করে
3. **SFX Synchronization:** Visual beat-এর সাথে sound effects 2-frame precision-এ sync করে
4. **Mastering:** সমগ্র mix-কে EBU R128 standard অনুযায়ী -14 LUFS-এ deliver করে

### এই স্কিল কার জন্য

- যে কেউ `assembled.mp4` থেকে broadcast-ready audio চায়
- যারা YouTube, podcast, বা social media-র জন্য consistent loudness চায়
- যারা meme-style chaotic sound এড়িয়ে cinematic, restrained audio চায়

### স্কিলের দর্শন

> **"Audio-র কাজ হলো story বলা, নিজেকে জাহির করা নয়।"**

ভালো sound design এমন হওয়া উচিত যা দর্শক সচেতনভাবে notice করে না — কিন্তু না থাকলে miss করে।

---

## ২. Audio Revision Prompts

নিচে ৪টি ready-to-use revision prompt দেওয়া হলো। এগুলো সরাসরি কপি করে ব্যবহার করা যাবে, অথবা নিজের project অনুযায়ী customize করা যাবে।

---

### Prompt ১: মিউজিক বদলানো

**পরিস্থিতি:** বর্তমান background music-টি ভিডিওর tone-এর সাথে মেলে না, বা নতুন কোনো track ব্যবহার করতে চাও।

```
আমার ভিডিওর background music বদলাতে চাই।

বর্তমান music file: assets/music/current_track.mp3
নতুন music file: assets/music/new_track.wav

নিম্নলিখিত নিয়ম মেনে নতুন track দিয়ে mix করো:
- ভয়েস থাকাকালীন music level: -20 dBFS
- ভয়েস না থাকলে (pause/b-roll): -12 dBFS
- Ducking attack: 10ms, release: 200ms
- ভিডিওর শেষ 3 সেকেন্ডে music fade out করো
- Final output -14 LUFS-এ master করো

SFX placement এবং voice processing অপরিবর্তিত রাখো।
```

---

### Prompt ২: SFX কমানো বা বাড়ানো

**পরিস্থিতি:** Current mix-এ SFX খুব বেশি বা খুব কম মনে হচ্ছে।

**SFX কমানোর প্রম্পট:**
```
এই ভিডিওতে SFX অনেক বেশি "meme-like" মনে হচ্ছে। Restraint rule apply করো:

1. সমস্ত SFX-এর list তৈরি করো timestamp সহ
2. যে SFX-গুলো clear narrative purpose নেই সেগুলো বাদ দাও
3. কোনো দুটি SFX 500ms-এর মধ্যে থাকলে একটি বাদ দাও
4. বাকি SFX-গুলোর volume -2 dB কমাও
5. Revised SFX list আমাকে show করো বাদ দেওয়ার কারণ সহ
```

**SFX বাড়ানোর প্রম্পট:**
```
এই ভিডিওতে SFX যোগ করতে চাই নির্দিষ্ট beats-এ।

নিম্নলিখিত timestamps-এ assets/sfx/ থেকে উপযুক্ত SFX place করো:
- [0:05.2] — zoom-in effect আছে
- [0:12.8] — graphic slide-in আছে
- [0:28.5] — reveal moment ("what it gave me back")

নিয়ম:
- SFX শুধু assets/sfx/ থেকে নেবে
- Voice থেকে -15 dBFS নিচে রাখবে
- Visual beat থেকে 2 frame-এর বেশি দেরি হবে না
- ফাইল না পাওয়া গেলে সেই beat silence রাখবে
```

---

### Prompt ৩: ভয়েস ক্লিনআপ সমস্যা সমাধান

**পরিস্থিতি:** ভয়েস processing-এর পর audio অস্বাভাবিক শোনাচ্ছে।

**সমস্যা: ভয়েস "watery" বা robotic শোনাচ্ছে**
```
Voice processing-এর পর ভয়েস robotic/watery শোনাচ্ছে।

সম্ভাব্য কারণ: anlmdn noise reduction strength বেশি।

দয়া করে:
1. anlmdn parameter s=0.0001 নিশ্চিত করো (0.0001-এর বেশি নয়)
2. যদি এখনও সমস্যা থাকে, anlmdn সম্পূর্ণ বাদ দিয়ে শুধু highpass দিয়ে process করো
3. Re-process করা file-টি deliver করো
4. আগের এবং পরের loudness measurement দেখাও
```

**সমস্যা: ভয়েস compressed/squashed মনে হচ্ছে**
```
Voice অনেক বেশি compressed মনে হচ্ছে — natural dynamics নেই।

দয়া করে acompressor settings adjust করো:
- ratio: 3 থেকে 2-এ কমাও
- threshold: -18dB থেকে -15dB-এ বাড়াও
- release: 50ms থেকে 80ms-এ বাড়াও

Re-process করো এবং নতুন settings-সহ file deliver করো।
আগের এবং পরের waveform compare করে দেখাও।
```

**সমস্যা: ভয়েস অনেক কম বা বেশি জোরে**
```
Final output-এ ভয়েস target loudness থেকে সরে গেছে।

দয়া করে:
1. এই command দিয়ে current loudness measure করো:
   ffmpeg -i voice_processed.mp4 -af ebur128 -f null - 2>&1
2. যদি integrated loudness -14 LUFS না হয়, loudnorm filter re-apply করো
3. Corrected file deliver করো এবং measurement output paste করো
```

---

### Prompt ৪: Audio Click/Pop ঠিক করা

**পরিস্থিতি:** Final output-এ clicks, pops, বা glitches শোনা যাচ্ছে।

```
Output audio-তে clicks/pops শোনা যাচ্ছে নির্দিষ্ট cut points-এ।

Click prevention measures apply করো:

১. প্রতিটি clip join point-এ 3ms crossfade:
   acrossfade=d=0.003:c1=tri:c2=tri

২. প্রতিটি segment-এ 1ms fade-in/out ramp:
   afade=t=in:st=0:d=0.001

৩. যদি কোনো specific timestamp-এ click আছে জানো:
   [timestamp এখানে দাও, উদাহরণ: 0:08.4, 0:15.2]
   সেই specific points-এ বিশেষ মনোযোগ দাও

৪. Re-render করো এবং deliver করো।
   Click-prone timestamps-এর list-ও জানাও।
```

---

## ৩. SFX File Naming Guide

`assets/sfx/` ফোল্ডারে ফাইল রাখার সময় নিচের naming convention মেনে চলো।

### Approved Filenames (পরিবর্তন করা যাবে না)

| Filename | Description | Usage Context |
|---|---|---|
| `whoosh.wav` | Quick air movement sound | Camera zoom, fast motion |
| `click.wav` | Sharp tap or snap | Hard cut, attention grab |
| `pop.wav` | Bubble or bass hit | Beat sync, waveform accent |
| `riser.wav` | Building tension sound | Pre-reveal, dramatic moment |
| `swoosh.wav` | Smooth slide sound | Graphic slide-in, transition |

### Naming Rules

1. **সর্বদা lowercase** — `Whoosh.wav` বা `WHOOSH.WAV` কাজ করবে না
2. **Extension সহ** — `.wav` extension বাদ দেওয়া যাবে না
3. **Space নেই** — `click sound.wav` নয়, `click.wav` হতে হবে
4. **নতুন SFX যোগ করতে হলে** — নতুন নাম SKILL.md-তে SFX Map-এ যোগ করো

### SFX ফাইল অনুপস্থিত হলে কী করবে — ffmpeg Auto-Synthesis

যদি `assets/sfx/` ফোল্ডারে কোনো কাস্টম ফাইল না থাকে, তাহলে ম্যানুয়ালি ডাউনলোড করার কোনো দরকার নেই। নিচের কমান্ডগুলো চালিয়ে ffmpeg-এর `sine` ও `anoise` ফিল্টার দিয়ে ৫টি ক্লিন সাউন্ড এক ক্লিকে জেনারেট করে নেওয়া যায়:

```bash
# ৫টি মিনিমালিস্টিক সাউন্ড এক ক্লিকে তৈরি করুন:
ffmpeg -f lavfi -i "sine=frequency=1600:duration=0.015,afade=t=in:st=0:d=0.002,afade=t=out:st=0.004:d=0.011,volume=0.25" -c:a pcm_s16le assets/sfx/click.wav -y
ffmpeg -f lavfi -i "aeval='0.35*sin(2*PI*(850-9000*t)*t)':d=0.05,afade=t=out:st=0.015:d=0.035" -c:a pcm_s16le assets/sfx/pop.wav -y
ffmpeg -f lavfi -i "anoise=d=0.3:c=white,bandpass=f=650:width_type=h:w=500,afade=t=in:st=0:d=0.12,afade=t=out:st=0.12:d=0.18,volume=0.28" -c:a pcm_s16le assets/sfx/whoosh.wav -y
ffmpeg -f lavfi -i "anoise=d=0.22:c=pink,bandpass=f=950:width_type=h:w=600,afade=t=in:st=0:d=0.07,afade=t=out:st=0.07:d=0.15,volume=0.22" -c:a pcm_s16le assets/sfx/swoosh.wav -y
ffmpeg -f lavfi -i "aeval='0.2*sin(2*PI*(200+350*t)*t)':d=1.0,afade=t=in:st=0:d=0.25,afade=t=out:st=0.75:d=0.25" -c:a pcm_s16le assets/sfx/riser.wav -y
```

```
# Log format (assets/sfx/ ফোল্ডারে sfx_log.txt-এ লেখো)
[TIMESTAMP] [SFX_FILE] [STATUS]
0:05.234    whoosh.wav  SYNTHESIZED (via ffmpeg anoise)
0:12.800    click.wav   SYNTHESIZED (via ffmpeg sine)
0:28.500    pop.wav     CUSTOM (from assets/sfx/)
```

---

## ৪. মিউজিক ফাইল ফরম্যাট সাপোর্ট

এই স্কিল নিম্নলিখিত music file format support করে:

### Primary Formats

| Format | Extension | Sample Rate | Bit Depth | Notes |
|---|---|---|---|---|
| WAV (PCM) | `.wav` | 44.1–96 kHz | 16–32 bit | **Best quality**, লসলেস |
| FLAC | `.flac` | 44.1–96 kHz | 16–24 bit | লসলেস compression, ছোট file size |
| MP3 | `.mp3` | 44.1–48 kHz | N/A (lossy) | সবচেয়ে common, acceptable quality |

### Recommended Formats

```
সর্বোত্তম:  WAV 48kHz 24-bit (video production standard)
ভালো:      FLAC 44.1kHz 24-bit
গ্রহণযোগ্য: MP3 320kbps
এড়িয়ে চলো: MP3 128kbps বা কম (artifacts শোনা যায়)
```

### Format Conversion Commands

```bash
# MP3 থেকে WAV-এ convert (processing-এর আগে)
ffmpeg -i music.mp3 -acodec pcm_s24le -ar 48000 music_converted.wav

# FLAC থেকে WAV-এ convert
ffmpeg -i music.flac -acodec pcm_s24le -ar 48000 music_converted.wav

# Final output: video-র সাথে AAC 192kbps (YouTube-র জন্য optimal)
ffmpeg -i mastered_output.mp4 -c:v copy -c:a aac -b:a 192k final_youtube.mp4
```

### Sample Rate Matching

Voice processing এবং music সবসময় একই sample rate-এ থাকা উচিত। সমস্যা এড়াতে:

```bash
# সব audio 48kHz-এ resample করো processing-এর আগে
ffmpeg -i input.wav -ar 48000 output_48k.wav
```

---

## ৫. LUFS Checking Commands

### Basic Loudness Check

```bash
# Integrated loudness, True Peak, এবং LRA একসাথে দেখো
ffmpeg -i output.mp4 -af ebur128=peak=true -f null - 2>&1 | tail -20
```

**Output example:**
```
[Parsed_ebur128_0 @ ...] Summary:

  Integrated loudness:
    I:         -14.0 LUFS
    Threshold: -24.1 LUFS

  Loudness range:
    LRA:         8.2 LU
    Threshold:  -24.1 LUFS
    LRA low:   -20.3 LUFS
    LRA high:  -12.1 LUFS

  True peak:
    Peak:        -1.0 dBFS
```

### Real-Time Loudness Monitoring

```bash
# Processing চলার সময় loudness দেখো (verbose mode)
ffmpeg -i input.mp4 \
  -af "ebur128=peak=true:framelog=verbose" \
  -f null - 2>&1 | grep -E "M:|S:|I:|Peak:"
```

### Batch Check (একাধিক ফাইল)

```bash
# সব wav ফাইলের loudness check করো
for file in *.wav; do
  echo "=== $file ==="
  ffmpeg -i "$file" -af ebur128 -f null - 2>&1 | grep "I:"
done
```

### Quick One-Line Check

```bash
# শুধু Integrated Loudness number
ffmpeg -i output.mp4 -af "loudnorm=print_format=summary" -f null - 2>&1 | grep "Input Integrated"
```

### Mastering Target চেক করার Script

```bash
#!/bin/bash
# check_lufs.sh — Final output verify করার জন্য

FILE=$1
echo "Checking: $FILE"

RESULT=$(ffmpeg -i "$FILE" -af ebur128=peak=true -f null - 2>&1)

LUFS=$(echo "$RESULT" | grep "I:" | tail -1 | awk '{print $2}')
PEAK=$(echo "$RESULT" | grep "Peak:" | tail -1 | awk '{print $2}')

echo "Integrated Loudness: $LUFS LUFS (target: -14.0)"
echo "True Peak: $PEAK dBFS (target: -1.0)"

# Pass/Fail check
if (( $(echo "$LUFS >= -15.0 && $LUFS <= -13.0" | bc -l) )); then
  echo "✓ LUFS: PASS"
else
  echo "✗ LUFS: FAIL — re-master required"
fi
```

---

## ৬. Troubleshooting: Common Audio Issues

### সমস্যা ১: Output-এ hiss বা background noise আছে

**কারণ:** `anlmdn` filter কাজ করছে না বা strength যথেষ্ট নয়।

**সমাধান:**
```bash
# anlmdn strength সামান্য বাড়াও (কিন্তু 0.0005-এর বেশি নয়)
ffmpeg -i input.mp4 \
  -af "highpass=f=80,anlmdn=s=0.0003" \
  output_denoised.mp4

# যদি noise room tone থেকে হয়, highpass frequency বাড়াও
ffmpeg -i input.mp4 \
  -af "highpass=f=100" \
  output_filtered.mp4
```

---

### সমস্যা ২: Music এবং Voice একই loudness-এ আছে

**কারণ:** Sidechain compression কাজ করছে না অথবা threshold ভুল।

**সমাধান:**
```bash
# threshold কমাও (ভয়েস আরও আগে ducking trigger করবে)
[0:a][1:a]sidechaincompress=threshold=0.008:ratio=6:attack=10:release=200[ducked]

# Manual volume automation (sidechain না থাকলে)
# ভয়েস segment-এ music -10 dB কমাও
ffmpeg -i music.wav \
  -af "volume=enable='between(t,5,15)':volume=0.316" \
  music_manual_duck.wav
  # (0.316 ≈ -10 dB)
```

---

### সমস্যা ৩: SFX সঠিক time-এ sync হচ্ছে না

**কারণ:** `adelay` value millisecond-এ দিতে হয়, seconds-এ নয়।

**সমাধান:**
```bash
# সঠিক: milliseconds ব্যবহার করো
# 5.234 seconds = 5234 milliseconds
ffmpeg -i assets/sfx/whoosh.wav \
  -af "adelay=5234|5234" \
  whoosh_delayed.wav

# Stereo file-এর জন্য উভয় channel delay দরকার: 5234|5234
```

---

### সমস্যা ৪: Final output clip করছে (distortion)

**কারণ:** Loudnorm-এর আগে signal already clipping বা True Peak limit অতিক্রম করছে।

**সমাধান:**
```bash
# Limiter যোগ করো loudnorm-এর আগে
ffmpeg -i input.mp4 \
  -af "alimiter=level_in=1:level_out=0.891:limit=0.891:attack=5:release=50,\
       loudnorm=I=-14:TP=-1:LRA=11" \
  output_limited.mp4
  # 0.891 ≈ -1 dBFS
```

---

### সমস্যা ৫: Music হঠাৎ শেষ হয়ে যাচ্ছে (abrupt cut)

**কারণ:** Fade out যোগ করা হয়নি।

**সমাধান:**
```bash
# আগে total duration জানো
DURATION=$(ffprobe -i music.wav -show_entries format=duration -v quiet -of csv="p=0")

# শেষ 3 সেকেন্ড fade out
FADE_START=$(echo "$DURATION - 3" | bc)
ffmpeg -i music.wav \
  -af "afade=t=out:st=${FADE_START}:d=3" \
  music_faded.wav
```

---

### সমস্যা ৬: Different sections-এ loudness অসামঞ্জস্যপূর্ণ

**কারণ:** Mastering individual clips-এ করা হয়েছে, final mix-এ নয়।

**সমাধান:**
> সর্বদা মনে রাখো: **প্রথমে সব join করো, তারপর একবার master করো।**

```bash
# সব clips join করো
ffmpeg -f concat -safe 0 -i file_list.txt -c copy joined_raw.mp4

# তারপর একবার loudnorm apply করো
ffmpeg -i joined_raw.mp4 \
  -af "loudnorm=I=-14:TP=-1:LRA=11" \
  final_mastered.mp4
```

---

### সমস্যা ৭: Voice processing-এর পর পিচ পরিবর্তন মনে হচ্ছে

**কারণ:** `loudnorm` filter-এর দুই-pass mode ব্যবহার না করা।

**সমাধান:**
```bash
# Pass 1: Measurement
ffmpeg -i input.mp4 \
  -af "loudnorm=I=-14:TP=-1:LRA=11:print_format=json" \
  -f null - 2>&1 | grep -E '"[a-z_]+"'

# Pass 2: Correction (measured_I, measured_TP, measured_LRA values ব্যবহার করো)
ffmpeg -i input.mp4 \
  -af "loudnorm=I=-14:TP=-1:LRA=11:\
       measured_I=-16.2:\
       measured_TP=-2.1:\
       measured_LRA=8.5:\
       linear=true" \
  output_corrected.mp4
```
`linear=true` pitch artifact সম্পূর্ণ এড়ায়।

---

## দ্রুত রেফারেন্স

| কাজ | Command snippet |
|---|---|
| LUFS check | `ffmpeg -i file -af ebur128 -f null -` |
| Noise remove | `anlmdn=s=0.0001` |
| 80Hz highpass | `highpass=f=80` |
| Compress voice | `acompressor=threshold=-18dB:ratio=3:attack=5:release=50` |
| Target loudness | `loudnorm=I=-14:TP=-1:LRA=11` |
| 3ms crossfade | `acrossfade=d=0.003:c1=tri:c2=tri` |
| Music duck | `sidechaincompress=threshold=0.015:ratio=4:attack=10:release=200` |
| Fade out 3s | `afade=t=out:st=START:d=3` |
