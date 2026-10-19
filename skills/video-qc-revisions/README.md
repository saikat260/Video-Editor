# video-qc-revisions

> ফাইনাল রেন্ডারের আগে 9-দফা QC অডিট এবং 8টি স্পেশালাইজড রিভিশন প্রম্পট।
> প্রতিটি প্রম্পট copy-paste-ready। SRT থাকলে বাংলা/মাল্টিলিঙ্গুয়াল ও ভিজ্যুয়াল বৈচিত্র্য QC-ও করে।

---

## 1. স্কিলের পরিচয়

**video-qc-revisions** স্কিলটি দুটি কাজ একসাথে করে:

### QC (Quality Control)
ফাইনাল রেন্ডারের আগে ৯টি নির্দিষ্ট পয়েন্টে ভিডিও যাচাই করে। প্রতিটি চেকের জন্য ffmpeg কমান্ড, Python স্ক্রিপ্ট এবং JSON লগ ফর্ম্যাট দেওয়া আছে। ফলাফল একটি master Pass/Fail টেবিলে সংকলিত হয়।

### Revision (পরিমার্জন)
QC-তে কোনো পয়েন্ট FAIL হলে, সংশ্লিষ্ট রিভিশন প্রম্পটটি সরাসরি AI-তে paste করলেই নির্দিষ্ট সমস্যার সমাধান পাওয়া যায়। প্রতিটি প্রম্পট standalone — আলাদা ব্যাখ্যা ছাড়াই কাজ করে।

### এই স্কিল কার জন্য?
- ভিডিও এডিটর যারা AI দিয়ে এডিট করেন
- Social media content creator যারা প্রতিদিন ভিডিও বানান
- ভিডিও প্রোডাকশন টিম যাদের একটি standard QC workflow দরকার
- বাংলা বা মাল্টিলিঙ্গুয়াল caption ব্যবহার করেন যারা

### কী কী লাগবে?
- `ffmpeg` (CLI, PATH-এ থাকতে হবে)
- `Python 3.8+` + PIL/Pillow, opencv-python, fonttools
- `ImageMagick` (loop check-এর জন্য)
- `brand.json` (ব্র্যান্ড config ফাইল — নিচে format দেওয়া আছে)
- `captions.srt` (Bengali QC-এর জন্য, optional)

### brand.json ফর্ম্যাট

```json
{
  "brand_names": ["YourBrand", "ProductName"],
  "product_names": ["Suite Pro", "AI Engine"],
  "taglines": ["Your Tagline Here"],
  "colors": {
    "primary": "#FF5733",
    "secondary": "#333333",
    "accent": "#FFD700",
    "background": "#0A0A0A",
    "text_light": "#FFFFFF",
    "text_dark": "#111111"
  },
  "fonts": {
    "heading": "YourHeadingFont-Bold",
    "body": "YourBodyFont-Regular"
  },
  "assets": {
    "logo_path": "assets/logo.png",
    "outro_template": "assets/outro_template.aep",
    "logo_animated": false
  }
}
```

---

## 2. সমস্ত ৮টি Revision Prompt (Copy-Paste Format)

> **⚠️ One-at-a-Time Rule:** একটি প্রম্পট পাঠাও → প্রিভিউ দেখো → QC re-run করো → PASS হলে পরেরটা।

---

### Revision 01 — Push the Energy

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

### Revision 02 — Use Your Brand

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

### Revision 03 — Motion on Every Line

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

### Revision 04 — Better Hook & Loop

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

### Revision 05 — Make a Moment Pop

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

### Revision 06 — Use Real Footage

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

### Revision 07 — Tease the Prompt

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

### Revision 08 — Fix a Take

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

## 3. QC Prompt (Copy-Paste Format)

রেন্ডারের আগে একবার এই পুরো প্রম্পটটি paste করো:

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

## 4. কখন কোন Revision পাঠাতে হবে — ডিসিশন চার্ট

```
QC রান করো
      │
      ▼
┌─────────────────┐
│ সব ৮টি PASS?   │──── হ্যাঁ ──→  ✅ ফাইনাল রেন্ডার করো
└─────────────────┘
      │ না
      ▼
┌───────────────────────────────────────────────────────────────────┐
│  কোন QC পয়েন্ট FAIL?          → কোন Revision পাঠাবে?           │
├───────────────────────────────────────────────────────────────────┤
│  1. Sync (SFX timing ভুল)      → Revision 01 (Push the Energy)  │
│  2. Face Coverage               → Revision 03 (Motion on Lines)  │
│     (caption মুখে)                 (reposition graphics)          │
│  3. Safe Zone (element বাইরে)  → Revision 03 (Motion on Lines)  │
│                                    (reposition to safe zone)       │
│  4. Cut Quality (mid-word cut) → Revision 08 (Fix a Take)       │
│  5. Caption Spelling (brand)   → Revision 02 (Use Your Brand)   │
│  6. Audio Levels (LUFS ভুল)   → Revision 01 (Push the Energy)  │
│                                    (loudness section)              │
│  7. Loop Check (frame mismatch)→ Revision 04 (Better Hook/Loop) │
│  8. Artifacts (black frames)   → Revision 06 (Use Real Footage) │
│     or (glitches, flash)         or Revision 08 (Fix a Take)    │
└───────────────────────────────────────────────────────────────────┘
      │
      ▼
Revision apply করো
      │
      ▼
সেই নির্দিষ্ট QC check আবার রান করো
      │
      ▼
PASS হলে → পরের FAIL পয়েন্টে যাও
      │
      ▼
সব PASS → ✅ ফাইনাল রেন্ডার
```

### বিস্তারিত ডিসিশন গাইড

| QC Fail পয়েন্ট | প্রথম Revision | কারণ | দ্বিতীয় Revision (যদি দরকার) |
|---|---|---|---|
| Sync | 01 — Push the Energy | Energy রিভিশন SFX re-sync করে | 08 — Fix a Take (যদি cut problem) |
| Face Coverage | 03 — Motion on Every Line | সব element reposition করে | 04 — Better Hook & Loop (যদি frame 0 সমস্যা) |
| Safe Zone | 03 — Motion on Every Line | Safe zone-এর মধ্যে রিপজিশন | 02 — Use Your Brand (যদি logo সমস্যা) |
| Cut Quality | 08 — Fix a Take | Best take বেছে re-time করে | 01 — Push the Energy (pause trimming) |
| Caption Spelling | 02 — Use Your Brand | Brand config থেকে সঠিক spelling টানে | নেই |
| Audio Levels | 01 — Push the Energy | LUFS/ducking সংশোধন করে | নেই |
| Loop Check | 04 — Better Hook & Loop | Frame 0 ও last frame redesign | নেই |
| Artifacts | 08 — Fix a Take | Clean take দিয়ে replace করে | 06 — Use Real Footage |

---

## 5. Pass/Fail টেবিল ফরম্যাট রেফারেন্স

প্রতিটি QC রানের পরে এই টেবিলটি কপি করে পূরণ করো এবং প্রজেক্ট ফোল্ডারে `qc_log_YYYYMMDD.md` নামে সেভ করো।

### Master QC Table (blank template)

```markdown
## QC Log — [PROJECT NAME] — [DATE]

| # | পয়েন্ট | স্ট্যাটাস | সমস্যার বিবরণ | টাইমকোড | Revision প্রম্পট |
|---|---|---|---|---|---|
| 1 | Sync | ⬜ PENDING | — | — | — |
| 2 | Face Coverage | ⬜ PENDING | — | — | — |
| 3 | Safe Zone | ⬜ PENDING | — | — | — |
| 4 | Cut Quality | ⬜ PENDING | — | — | — |
| 5 | Caption Spelling | ⬜ PENDING | — | — | — |
| 6 | Audio Levels | ⬜ PENDING | — | — | — |
| 7 | Loop Check | ⬜ PENDING | — | — | — |
| 8 | Artifact Check | ⬜ PENDING | — | — | — |

**Result: _/8 PASS — Ready for render: YES / NO**
```

### ভরা উদাহরণ (Filled Example)

```markdown
## QC Log — MyProject_v3 — 2024-03-15

| # | পয়েন্ট | স্ট্যাটাস | সমস্যার বিবরণ | টাইমকোড | Revision প্রম্পট |
|---|---|---|---|---|---|
| 1 | Sync | ✅ PASS | — | — | — |
| 2 | Face Coverage | ❌ FAIL | Caption "Subscribe" মুখের চিবুকে | 0:12 | 03 |
| 3 | Safe Zone | ✅ PASS | — | — | — |
| 4 | Cut Quality | ❌ FAIL | "import-ant" mid-word cut | 1:03 | 08 |
| 5 | Caption Spelling | ❌ FAIL | "Youtube" → "YouTube" x3 | 0:34, 1:22, 2:05 | 02 |
| 6 | Audio Levels | ✅ PASS | — | — | — |
| 7 | Loop Check | ✅ PASS | — | — | — |
| 8 | Artifact Check | ✅ PASS | — | — | — |
| 9 | Motion Diversity | ❌ FAIL | Kinetic typography কার্ডে আবদ্ধ | 1:45 | 03 |

**Result: 6/9 PASS — Ready for render: NO**
**Priority order: Fix #4 (08) → Fix #9 (03) → Fix #2 (03) → Fix #5 (02)**
```

### JSON লগ Master Format

সব ৯টি চেকের JSON একটি ফাইলে:

```json
{
  "project": "MyProject_v3",
  "date": "2024-03-15",
  "qc_version": "2.0",
  "overall": {
    "pass_count": 5,
    "fail_count": 3,
    "ready_for_render": false
  },
  "checks": [
    { "check": "sync", "status": "PASS", "issues": [] },
    {
      "check": "face_coverage",
      "status": "FAIL",
      "issues": [
        {
          "timecode": "00:00:12:00",
          "description": "Caption overlaps subject chin",
          "severity": "MEDIUM"
        }
      ]
    },
    { "check": "safe_zone", "status": "PASS", "issues": [] },
    {
      "check": "cut_quality",
      "status": "FAIL",
      "issues": [
        {
          "timecode": "00:01:03:08",
          "description": "Mid-word cut on 'import-ant'",
          "type": "mid_word_cut",
          "severity": "HIGH"
        }
      ]
    },
    {
      "check": "caption_spelling",
      "status": "FAIL",
      "issues": [
        { "timecode": "00:00:34:00", "found": "Youtube", "expected": "YouTube", "severity": "HIGH" },
        { "timecode": "00:01:22:00", "found": "Youtube", "expected": "YouTube", "severity": "HIGH" },
        { "timecode": "00:02:05:00", "found": "Youtube", "expected": "YouTube", "severity": "HIGH" }
      ]
    },
    { "check": "audio_levels", "status": "PASS", "issues": [] },
    { "check": "loop", "status": "PASS", "issues": [] },
    { "check": "artifacts", "status": "PASS", "issues": [] }
  ]
}
```

---

## 6. স্পেলিং ও Brand Name যাচাইয়ের গাইড

### কেন এটা গুরুত্বপূর্ণ?
ভুল brand name spelling ভিডিওকে unprofessional দেখায় এবং কোনো কোনো ক্ষেত্রে copyright/trademark সমস্যা তৈরি করতে পারে।

### সাধারণ ভুলের তালিকা

| ভুল | সঠিক | নোট |
|---|---|---|
| Youtube | YouTube | capital T অবশ্যই |
| Tiktok | TikTok | capital T ও K |
| Instagram | Instagram | সঠিক — lowercase g |
| adobe premiere | Adobe Premiere | উভয় শব্দ Capitalize |
| after effects | After Effects | উভয় শব্দ Capitalize |
| photoshop | Photoshop | capital P |
| davinci resolve | DaVinci Resolve | capital D, V, R |
| iphone | iPhone | lowercase i, capital P |
| macbook | MacBook | capital M, B |
| chatgpt | ChatGPT | capital C, G, P, T |
| openai | OpenAI | capital O, A, I |

### Brand Name Extraction Script

```python
#!/usr/bin/env python3
"""
brand_spell_check.py
SRT ফাইলের সমস্ত brand name spelling যাচাই করে।
Usage: python brand_spell_check.py captions.srt brand.json
"""
import json
import re
import sys
from pathlib import Path

def load_brand(brand_path):
    with open(brand_path, encoding='utf-8') as f:
        return json.load(f)

def extract_srt_text_with_timecodes(srt_path):
    """SRT থেকে (timecode, text) tuple list বের করো"""
    with open(srt_path, encoding='utf-8') as f:
        content = f.read()
    blocks = re.split(r'\n\n+', content.strip())
    entries = []
    for block in blocks:
        lines = block.split('\n')
        if len(lines) >= 3:
            tc_match = re.match(
                r'(\d{2}:\d{2}:\d{2}[,\.]\d{3})\s*-->\s*(\d{2}:\d{2}:\d{2}[,\.]\d{3})',
                lines[1]
            )
            if tc_match:
                text = ' '.join(lines[2:])
                entries.append((tc_match.group(1), text))
    return entries

def check_brand_spelling(entries, brand):
    issues = []
    all_names = (
        brand.get('brand_names', []) +
        brand.get('product_names', []) +
        brand.get('taglines', [])
    )
    for timecode, text in entries:
        for correct_name in all_names:
            # Case-insensitive search
            pattern = re.compile(re.escape(correct_name), re.IGNORECASE)
            for match in pattern.finditer(text):
                if match.group() != correct_name:
                    issues.append({
                        'timecode': timecode,
                        'found': match.group(),
                        'expected': correct_name,
                        'context': text[:60] + ('...' if len(text) > 60 else ''),
                        'severity': 'HIGH'
                    })
    return issues

def main():
    srt_path = sys.argv[1] if len(sys.argv) > 1 else 'captions.srt'
    brand_path = sys.argv[2] if len(sys.argv) > 2 else 'brand.json'

    brand = load_brand(brand_path)
    entries = extract_srt_text_with_timecodes(srt_path)
    issues = check_brand_spelling(entries, brand)

    if issues:
        print(f"❌ FAIL — {len(issues)} spelling issue(s) found:\n")
        print(json.dumps(issues, ensure_ascii=False, indent=2))
    else:
        print("✅ PASS — All brand names spelled correctly.")

if __name__ == '__main__':
    main()
```

### Manual Checklist (ম্যানুয়ালি চেক করতে হলে)

প্রতিটি ক্যাপশন / title card-এ নিচের প্রশ্নগুলো করো:

- [ ] কি সব brand name-এর first letter capitalize?
- [ ] কি CamelCase brand (YouTube, TikTok, MacBook) সঠিকভাবে লেখা?
- [ ] কি product version number সঠিক (যেমন: "Pro 2024" না "pro 2024")?
- [ ] কি tagline brand.json-এর সাথে word-for-word match করছে?
- [ ] কি Bengali brand name-এর Unicode বানান brand.json-এর সাথে identical?

---

## 7. Bengali QC Special Rules

বাংলা ক্যাপশন বা SRT ফাইল থাকলে নিচের বিশেষ নিয়মগুলো অবশ্যই মানতে হবে।

### নিয়ম ১: UTF-8 BOM-less এনকোডিং

SRT ফাইল UTF-8 হতে হবে, কিন্তু **BOM (Byte Order Mark) ছাড়া**।
BOM থাকলে অনেক ভিডিও প্লেয়ার ও এডিটর প্রথম ক্যাপশন ভেঙে দেখায়।

```bash
# Windows-এ BOM remove করো (PowerShell)
$content = Get-Content captions.srt -Encoding UTF8
$content | Set-Content captions_nobom.srt -Encoding UTF8NoBOM

# Linux/Mac-এ
sed -i '1s/^\xEF\xBB\xBF//' captions.srt
```

### নিয়ম ২: হসন্ত (্) রেন্ডার চেক

হসন্ত (U+09CD) বাংলা ব্যঞ্জন সংযুক্তির জন্য ব্যবহৃত হয়। ভুল font বা rendering engine-এ এটি আলাদা character হিসেবে দেখায় বা missing হয়।

**যে সমস্যাগুলো হতে পারে:**
- `ক্ষ` → `ক্` + `ষ` (আলাদা দেখায়)
- `জ্ঞ` → `জ্` + `ঞ` (ভেঙে যায়)
- `ত্ব` → `ত` + `্` + `ব` (হসন্ত আলাদা দেখা যায়)

**Fix:** OpenType layout feature সমর্থন করে এমন font ব্যবহার করো:
- Noto Sans Bengali
- Hind Siliguri
- Mukti (open source)
- Shonar Bangla (Windows built-in)

```bash
# Font-এ হসন্ত glyph আছে কিনা চেক করো
python3 -c "
from fonttools.ttLib import TTFont
font = TTFont('NotoSansBengali-Regular.ttf')
cmap = font.getBestCmap()
critical = {
    0x09CD: 'Hasanta (্)',
    0x09CE: 'Khanda Ta (ৎ)',
    0x09DC: 'RRA (ড়)',
    0x09DD: 'RHA (ঢ়)',
    0x09DF: 'YYA (য়)',
}
for cp, name in critical.items():
    status = '✅' if cp in cmap else '❌ MISSING'
    print(f'{status} U+{cp:04X} {name}')
"
```

### নিয়ম ৩: অনুস্বার ও বিসর্গ

- **অনুস্বার (ং):** U+0982 — "রং", "ঢাকং" — ভুল font-এ circle missing হয়
- **বিসর্গ (ঃ):** U+0983 — "দুঃখ", "প্রতিঃ" — দুটো dot ঠিকমতো দেখা যায় কিনা

```bash
# Count ও verify করো
python3 -c "
with open('captions.srt', encoding='utf-8') as f:
    text = f.read()
checks = {
    'ং (Anusvara U+0982)': '\u0982',
    'ঃ (Visarga U+0983)': '\u0983',
    '্ (Hasanta U+09CD)': '\u09CD',
    'ৎ (Khanda Ta U+09CE)': '\u09CE',
    '় (Nukta U+09BC)': '\u09BC',
}
for name, char in checks.items():
    count = text.count(char)
    print(f'{name}: {count} occurrences')
"
```

### নিয়ম ৪: SRT Timing — বাংলা Word Boundary

বাংলায় একটি subtitle block যেন কোনো শব্দের মাঝে কাটা না যায়। বিশেষ করে:
- হসন্ত (্) দিয়ে শেষ হলে পরের ব্যঞ্জন missing
- Vowel sign (া, ি, ী, ু, ূ, ৃ, ে, ৈ, ো, ৌ) দিয়ে শেষ হলে স্বর missing
- র-ফলা (্র) বা য-ফলা (্য) দিয়ে শেষ হলে conjunct ভেঙে যায়

**Safe timing নিয়ম:**
- প্রতিটি subtitle block শেষ হোক একটি পূর্ণ শব্দে (space বা punctuation-এ)
- ন্যূনতম display duration: 40 ms/character (বাংলার জন্য), ইংরেজির 33 ms/char-এর চেয়ে বেশি
- Maximum characters per line: বাংলায় 25 character (ইংরেজির 42-এর চেয়ে কম, কারণ বাংলা গ্লিফ বড়)

### নিয়ম ৫: Multilingual Mixed Caption

একই SRT-এ বাংলা ও ইংরেজি মিশ্রিত থাকলে:

```
1
00:00:05,000 --> 00:00:08,000
আমি ব্যবহার করেছি Adobe Premiere Pro
এবং After Effects দিয়ে edit করেছি।
```

এখানে "Adobe Premiere Pro" ও "After Effects" ইংরেজিতেই থাকবে — বাংলা অনুবাদ করা যাবে না (brand name)।

**Python check:**
```python
# Mixed line-এ brand name ইংরেজিতে আছে কিনা
import re, json

with open('brand.json', encoding='utf-8') as f:
    brand = json.load(f)

with open('captions.srt', encoding='utf-8') as f:
    text = f.read()

for name in brand['brand_names'] + brand['product_names']:
    # বাংলা transliteration detect করো (heuristic)
    bengali_trans_pattern = re.compile(r'[\u0980-\u09FF]+' + re.escape(name[0].lower()))
    if bengali_trans_pattern.search(text.lower()):
        print(f"⚠️  Possible Bengali transliteration of '{name}' found — verify manually")
```

### Bengali QC Checklist (সম্পূর্ণ)

| # | চেক | টুল | Pass Criteria |
|---|---|---|---|
| B1 | UTF-8 BOM-less | `file -i` / PowerShell | charset=utf-8, no BOM |
| B2 | Hasanta glyph | fonttools cmap | U+09CD in font |
| B3 | Anusvar glyph | fonttools cmap | U+0982 in font |
| B4 | Visarga glyph | fonttools cmap | U+0983 in font |
| B5 | Khanda Ta | fonttools cmap | U+09CE in font |
| B6 | Conjunct rendering | HarfBuzz shaping | ক্ষ, জ্ঞ, etc. single glyph |
| B7 | Word boundary timing | Python SRT parser | No block ends on Hasanta/vowel sign |
| B8 | Brand name (Bengali) | brand.json compare | Exact Unicode match |
| B9 | Mixed lang brand | Manual / Python | Brand names remain in English |
| B10 | Char/line limit | Python line length | ≤ 25 chars/line Bengali |

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────────┐
│              VIDEO QC REVISIONS — QUICK REFERENCE           │
├─────────────────────────────────────────────────────────────┤
│  QC TARGETS                                                 │
│  Audio: -14 LUFS ±1  |  True Peak: ≤ -1 dBTP               │
│  Music duck: ≥ -18 dB below voice                           │
│  Safe zone: 10% inset (title)  |  5% inset (action)         │
│  Sync: ± 2 frames max                                       │
│  Flash: < 3 Hz (WCAG 2.3.1)                                 │
├─────────────────────────────────────────────────────────────┤
│  REVISION MAP                                               │
│  01 Energy  02 Brand  03 Motion  04 Hook/Loop               │
│  05 Moment  06 Footage  07 Prompt  08 Take Fix              │
├─────────────────────────────────────────────────────────────┤
│  RULE: ONE REVISION AT A TIME                               │
│  Send → Preview → QC re-run → Next                         │
├─────────────────────────────────────────────────────────────┤
│  BENGALI                                                    │
│  UTF-8 no BOM  |  Font: Noto Sans Bengali / Hind Siliguri  │
│  Max 25 chars/line  |  No block ends on ্ or vowel sign    │
└─────────────────────────────────────────────────────────────┘
```

---

*video-qc-revisions v2.0 — AI Systems Lab*
