# 🎬 video-project-scaffolder

> **AI ভিডিও এডিটিং প্রজেক্টের জন্য সম্পূর্ণ স্ক্যাফোল্ডিং স্কিল**
>
> Claude Code + HyperFrames + ffmpeg পাইপলাইনের জন্য প্রজেক্ট ফোল্ডার, কনফিগারেশন ফাইল ও গাইড স্বয়ংক্রিয়ভাবে তৈরি করে।

---

## 📖 স্কিলের পরিচয়

`video-project-scaffolder` হলো একটি Claude Code স্কিল যা একটি নতুন AI ভিডিও এডিটিং প্রজেক্ট শুরু করার জন্য প্রয়োজনীয় সমস্ত ফোল্ডার, কনফিগারেশন ফাইল এবং গাইড ডকুমেন্ট স্বয়ংক্রিয়ভাবে জেনারেট করে।

### এটি কী কী করে?

| ধাপ | কাজ |
|-----|-----|
| **১** | Node.js, ffmpeg ও HyperFrames-এর উপস্থিতি ও ভার্সন যাচাই করে |
| **২** | প্রজেক্টের সম্পূর্ণ ফোল্ডার স্ট্রাকচার তৈরি করে |
| **৩** | `config.json` — প্রজেক্ট কনফিগারেশন ফাইল জেনারেট করে |
| **৪** | `script.txt` — স্ক্রিপ্ট টেমপ্লেট তৈরি করে |
| **৫** | `recording-guide.md` — রেকর্ডিং গোল্ডেন রুলস ডকুমেন্ট তৈরি করে |
| **৬** | `takes.md` — টেক সিলেকশন লগ টেমপ্লেট তৈরি করে |
| **৭** | সফলভাবে সম্পন্ন হলে পরবর্তী ধাপের নির্দেশনা দেয় |

### কাদের জন্য?

- **কন্টেন্ট ক্রিয়েটর** যারা Instagram Reel, TikTok, YouTube Shorts বানান
- **ব্র্যান্ড মার্কেটার** যারা ক্লায়েন্টের জন্য ভিডিও কন্টেন্ট তৈরি করেন
- **ফ্রিল্যান্স ভিডিও এডিটর** যারা প্রতি প্রজেক্টে পরিষ্কার স্ট্রাকচার চান
- **AI ভিডিও এডিটিং শিক্ষার্থী** যারা Claude + HyperFrames ওয়ার্কফ্লো শিখছেন

---

## ⚙️ ইনস্টলেশন গাইড (ধাপে ধাপে)

### ধাপ ১: স্কিল ফাইল ক্লোন বা ডাউনলোড করো

তোমার Claude Code `skills/` ডিরেক্টরিতে এই স্কিলটি রাখো:

```
skills/
└── video-project-scaffolder/
    ├── SKILL.md
    └── README.md
```

**Git দিয়ে (যদি রিপোজিটারিতে থাকে):**
```bash
git clone <repo-url> skills/video-project-scaffolder
```

**ম্যানুয়ালি:**
```
SKILL.md এবং README.md ফাইল দুটো
skills/video-project-scaffolder/ ফোল্ডারে রাখো।
```

---

### ধাপ ২: Node.js ≥22 ইনস্টল করো

**macOS:**
```bash
# Homebrew দিয়ে
brew install node@22

# nvm দিয়ে (প্রস্তাবিত)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
nvm install 22
nvm use 22
nvm alias default 22
```

**Windows:**
```powershell
# winget দিয়ে
winget install OpenJS.NodeJS.LTS

# nvm-windows দিয়ে
# প্রথমে: https://github.com/coreybutler/nvm-windows থেকে ইনস্টলার নামাও
nvm install 22
nvm use 22
```

**Linux (Ubuntu/Debian):**
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**যাচাই:**
```bash
node --version   # v22.x.x দেখাবে
npm --version    # 10.x.x দেখাবে
```

---

### ধাপ ৩: ffmpeg ইনস্টল করো

**macOS:**
```bash
brew install ffmpeg
```

**Windows:**
```powershell
winget install Gyan.FFmpeg
# PATH-এ যোগ করা হয়ে যাবে। যদি না হয়, ম্যানুয়ালি System Environment Variables-এ যোগ করো।
```

**Linux:**
```bash
sudo apt-get update && sudo apt-get install -y ffmpeg
```

**যাচাই:**
```bash
ffmpeg -version
# ffmpeg version 6.x.x ... দেখাবে
```

---

### ধাপ ৪: HyperFrames ইনস্টল করো

**গ্লোবাল ইনস্টল (একবার করলেই হয়):**
```bash
npm install -g hyperframes
```

**npx দিয়ে (ইনস্টল ছাড়া):**
```bash
# ইনস্টল না করেও সরাসরি রান করা যাবে
npx hyperframes --version
```

**যাচাই:**
```bash
npx hyperframes --version
# hyperframes/X.X.X দেখাবে
```

---

### ধাপ ৫: স্কিল লোড করো

Claude Code চালু করো এবং স্কিল লোড হয়েছে কিনা যাচাই করো:

```
Claude, তোমার স্কিল লিস্ট দেখাও।
```

`video-project-scaffolder` দেখা গেলে সেটআপ সম্পন্ন।

---

## 💬 Usage Prompts — ৫টি উদাহরণ

নিচের যেকোনো প্রম্পট দিয়ে স্কিলটি ট্রিগার করা যাবে:

### প্রম্পট ১ — বাংলা (সবচেয়ে সরল)

```
নতুন ভিডিও প্রজেক্ট শুরু করো।
প্রজেক্টের নাম: ramadan-campaign-2025
```

Claude এই নামে ফোল্ডার তৈরি করবে, পরিবেশ যাচাই করবে, এবং সম্পূর্ণ স্ট্রাকচার জেনারেট করবে।

---

### প্রম্পট ২ — ইংরেজি (বিস্তারিত)

```
scaffold video project named "brand-reel-oct" for Instagram Reels.
Platform: instagram, aspect ratio: 9:16, language: Bengali (bn).
```

Claude প্ল্যাটফর্ম ও ভাষা অনুযায়ী `config.json` কাস্টমাইজ করে তৈরি করবে।

---

### প্রম্পট ৩ — মিশ্র (বাংলা + ইংরেজি)

```
আমার একটা নতুন ভিডিও প্রজেক্ট লাগবে।
নাম দাও "product-launch-v2", platform হবে TikTok,
আর background music ব্যবহার করবো তাই assets/music ফোল্ডার দরকার।
```

---

### প্রম্পট ৪ — দ্রুত সেটআপ (এক লাইন)

```
create project structure for "my-reel"
```

ন্যূনতম ইনপুটে ডিফল্ট সেটিংস দিয়ে সম্পূর্ণ প্রজেক্ট তৈরি হবে।

---

### প্রম্পট ৫ — আগের ভিডিও রিফারেন্সসহ

```
প্রজেক্ট ফোল্ডার বানাও "tutorial-series-ep3" নামে।
আমার কাছে আগের একটা ভিডিও আছে (before/after দেখাবো),
সেটা old/ ফোল্ডারে রাখবো।
SRT ফাইলও আছে বাংলায়, সেটা srt/ ফোল্ডারে রাখবো।
```

Claude `old/` এবং `srt/` ফোল্ডার তৈরি করে এবং SRT ফোল্ডার পলিসি মনে করিয়ে দেবে।

---

## 📁 প্রজেক্ট ফোল্ডার স্ট্রাকচারের ব্যাখ্যা

স্কিল রান করলে নিচের সম্পূর্ণ স্ট্রাকচার তৈরি হয়:

```
my-edit/
├── raw/                    # সব raw ভিডিও টেক
├── srt/                    # Pre-existing SRT ফাইল
├── script.txt              # ফাইনাল স্ক্রিপ্ট
├── old/                    # আগের ভিডিও রেফারেন্স
├── assets/
│   ├── music/              # ব্যাকগ্রাউন্ড মিউজিক
│   ├── sfx/                # সাউন্ড ইফেক্ট
│   ├── fonts/              # ক্যাপশন ফন্ট
│   ├── images/             # ব্র্যান্ড লোগো ও গ্রাফিক্স
│   └── broll/              # B-roll ভিডিও ফুটেজ
├── takes.md                # AI টেক সিলেকশন লগ
├── config.json             # প্রজেক্ট কনফিগারেশন
├── recording-guide.md      # রেকর্ডিং গাইড
└── output/
    ├── preview/            # প্রিভিউ ফাইল
    └── final.mp4           # ফাইনাল আউটপুট
```

### প্রতিটি ফোল্ডারে কী রাখবে?

#### `raw/` — Raw ভিডিও ফুটেজ

রেকর্ড করা সব ভিডিও ক্লিপ এখানে রাখো। নামকরণ রেকর্ডিং অর্ডার অনুযায়ী করা ভালো:

```
raw/
├── take_001.mp4    # প্রথম টেক
├── take_002.mp4    # দ্বিতীয় টেক
└── take_003.mp4    # তৃতীয় টেক
```

Claude `raw/` ফোল্ডার স্ক্যান করে সবচেয়ে ভালো টেকগুলো বেছে নেয় এবং `takes.md`-এ লগ করে।

#### `srt/` — সাবটাইটেল ফাইল

নিজে ট্রান্সক্রাইব করা SRT ফাইল এখানে রাখো। **SRT ফাইল থাকলে AI ট্রান্সক্রিপশন বন্ধ হয়ে যায়** (বিস্তারিত: নিচের SRT পলিসি দেখো)।

#### `script.txt` — ভিডিও স্ক্রিপ্ট

ভিডিওর পুরো স্ক্রিপ্ট এখানে লেখো। Claude এটিকে Sync Map হিসেবে ব্যবহার করে raw ফুটেজের সাথে মেলায়।

#### `old/` — রেফারেন্স ভিডিও

Before/After তুলনার জন্য আগের ভিডিও এখানে রাখো। Claude প্রয়োজনে এটি রেফারেন্স হিসেবে ব্যবহার করে।

#### `assets/` — সমস্ত মিডিয়া Assets

```
assets/
├── music/     # .mp3, .wav — ব্যাকগ্রাউন্ড মিউজিক
├── sfx/       # .mp3, .wav — সাউন্ড ইফেক্ট
├── fonts/     # .woff2, .ttf, .otf — ক্যাপশন ফন্ট
├── images/    # .png, .jpg, .svg — স্ট্যাটিক গ্রাফিক্স
└── broll/     # .mp4, .mov — B-roll ভিডিও
```

#### `output/` — ফাইনাল রেন্ডার

```
output/
├── preview/         # লো-রেজ প্রিভিউ, থাম্বনেইল, বিট স্টিলস
└── final.mp4        # ফাইনাল হাই-কোয়ালিটি ভিডিও
```

---

## 📝 SRT ফাইল ব্যবহারের নিয়ম

### সোনালি নিয়ম

> **`srt/` ফোল্ডারে একটিও `.srt` ফাইল থাকলে Claude কোনো ক্লিপ ট্রান্সক্রাইব করে না।**

### কেন এই নিয়ম আছে?

বাংলা, আরবি, হিন্দিসহ অনেক ভাষার ক্ষেত্রে Whisper বা অন্য AI ট্রান্সক্রিপশন টুল শতভাগ নির্ভুল নয়। ব্যবহারকারী নিজে হাতে লেখা বা সম্পাদনা করা SRT প্রদান করলে সেটি সবসময় AI ট্রান্সক্রিপশনের চেয়ে সঠিক। তাই Claude সেই SRT-কে সর্বোচ্চ অগ্রাধিকার দেয়।

### SRT ওয়ার্কফ্লো

```
① raw/ ফোল্ডারে ভিডিও রাখো
② নিজে SRT ফাইল তৈরি করো (বা রেডিমেড SRT নামাও)
③ srt/ ফোল্ডারে রাখো
④ Claude-কে বলো এডিটিং শুরু করতে
   → Claude SRT দেখবে → ট্রান্সক্রিপশন বাদ দেবে → SRT দিয়ে সরাসরি কাজ করবে
```

### SRT না থাকলে কী হয়?

```
① raw/ ফোল্ডারে ভিডিও রাখো
② srt/ ফোল্ডার খালি রাখো
③ Claude-কে বলো এডিটিং শুরু করতে
   → Claude raw ফুটেজ ট্রান্সক্রাইব করবে → SRT তৈরি করবে → এডিটিং করবে
```

### SRT ফাইলের ফরম্যাট উদাহরণ

```srt
1
00:00:00,000 --> 00:00:02,500
আমি আজকে তোমাদের একটা জিনিস দেখাবো

2
00:00:02,800 --> 00:00:05,200
যেটা তোমার জীবন বদলে দিতে পারে

3
00:00:05,500 --> 00:00:08,000
চলো শুরু করা যাক
```

---

## 🎨 Assets ফোল্ডারে কী কী রাখতে হয়

### `assets/music/` — ব্যাকগ্রাউন্ড মিউজিক

| ফরম্যাট | প্রস্তাবিত | বিবরণ |
|---------|-----------|-------|
| `.mp3` | ✅ হ্যাঁ | ছোট ফাইল সাইজ, ভালো কোয়ালিটি |
| `.wav` | ✅ হ্যাঁ | Lossless কোয়ালিটি |
| `.aac` | ✅ হ্যাঁ | iOS-ফ্রেন্ডলি |
| `.flac` | ⚠️ ঐচ্ছিক | খুব বড় ফাইল |

**টিপস:**
- Royalty-free মিউজিক ব্যবহার করো (Pixabay, Mixkit, Epidemic Sound)
- BPM (beats per minute) মিলিয়ে কাটিংয়ের সাথে সিঙ্ক হলে ভিডিও আরো প্রফেশনাল লাগে
- `config.json`-এ `audio.music_duck_db: -20` সেট আছে — কথার সময় মিউজিক -20dB কমবে

---

### `assets/sfx/` — সাউন্ড ইফেক্ট

নামকরণ কনভেনশন মেনে চললে Claude স্ক্রিপ্টের নির্দেশ অনুযায়ী স্বয়ংক্রিয়ভাবে সঠিক SFX বাছাই করতে পারে:

```
assets/sfx/
├── whoosh.mp3       # [sfx: whoosh] — ট্রানজিশন সুইশ সাউন্ড
├── pop.mp3          # [sfx: pop] — আইটেম আসার শব্দ
├── click.mp3        # [sfx: click] — UI ক্লিক
├── riser.mp3        # [sfx: riser] — বিল্ড-আপ সাউন্ড
├── swoosh.mp3       # [sfx: swoosh] — স্লাইড সাউন্ড
├── ding.mp3         # [sfx: ding] — নোটিফিকেশন
└── transition.mp3   # [sfx: transition] — সাধারণ ট্রানজিশন
```

---

### `assets/fonts/` — ক্যাপশন ফন্ট

ক্যাপশনের জন্য ফন্ট ফাইল এখানে রাখো। বাংলা ক্যাপশনের জন্য অবশ্যই বাংলা ইউনিকোড সাপোর্ট করে এমন ফন্ট ব্যবহার করো।

**প্রস্তাবিত বাংলা ফন্ট:**
- **Hind Siliguri** — ক্লিন, রিডেবল, স্ক্রিনের জন্য উপযুক্ত
- **Noto Sans Bengali** — Google-এর ফ্রি ফন্ট, সব ডিভাইসে কাজ করে
- **SolaimanLipi** — ক্লাসিক বাংলা ফন্ট

```
assets/fonts/
├── caption-regular.woff2   # প্রধান ফন্ট (হালকা ওজন)
├── caption-bold.woff2      # হাইলাইট শব্দের জন্য
└── display.ttf             # শিরোনাম / বড় টেক্সটের জন্য
```

---

### `assets/images/` — স্ট্যাটিক গ্রাফিক্স

```
assets/images/
├── logo.png              # ব্র্যান্ড লোগো (transparent background)
├── logo-dark.png         # ডার্ক ব্যাকগ্রাউন্ডের জন্য লোগো
├── business-card.jpg     # বিজনেস কার্ড / কন্ট্যাক্ট ইনফো
├── product-001.png       # পণ্যের ছবি
├── screenshot-app.png    # অ্যাপ স্ক্রিনশট
└── intro-card.png        # ইন্ট্রো ব্যানার গ্রাফিক
```

**গুরুত্বপূর্ণ:**
- লোগো সবসময় PNG ফরম্যাটে ট্রান্সপারেন্ট ব্যাকগ্রাউন্ডসহ রাখো
- ইমেজের রেজোলিউশন কমপক্ষে `1080×1080px` রাখো
- `logo.png` বা `logo-dark.png` নাম দিলে Claude স্বয়ংক্রিয়ভাবে ব্র্যান্ড লোগো হিসেবে চেনে

---

### `assets/broll/` — B-roll ফুটেজ

```
assets/broll/
├── city-timelapse.mp4        # শহরের টাইমল্যাপস
├── product-closeup.mp4       # পণ্যের ক্লোজ-আপ শট
├── office-ambience.mp4       # অফিস পরিবেশের শট
├── screen-recording-demo.mp4 # অ্যাপ ডেমো স্ক্রিন রেকর্ডিং
└── nature-background.mp4     # প্রকৃতির ব্যাকগ্রাউন্ড শট
```

স্ক্রিপ্টে `[b-roll: শহর]` লিখলে Claude `broll/` ফোল্ডারে "city" বা "শহর" নামের ফাইল খুঁজবে।

---

## 🔧 config.json কাস্টমাইজেশন গাইড

`config.json` ফাইলটি প্রজেক্টের কন্ট্রোল প্যানেল। এই ফাইলের মাধ্যমে সব সেটিংস নিয়ন্ত্রণ করা যায়।

### প্ল্যাটফর্ম অনুযায়ী সেটিংস

**Instagram Reels / TikTok (ডিফল্ট):**
```json
{
  "project": {
    "platform": "instagram_tiktok",
    "aspect_ratio": "9:16",
    "resolution": "1080x1920"
  }
}
```

**YouTube Landscape:**
```json
{
  "project": {
    "platform": "youtube",
    "aspect_ratio": "16:9",
    "resolution": "1920x1080"
  }
}
```

**YouTube Shorts:**
```json
{
  "project": {
    "platform": "youtube_shorts",
    "aspect_ratio": "9:16",
    "resolution": "1080x1920"
  }
}
```

**Square (Facebook / LinkedIn):**
```json
{
  "project": {
    "platform": "facebook_linkedin",
    "aspect_ratio": "1:1",
    "resolution": "1080x1080"
  }
}
```

---

### কালার প্যালেট পরিবর্তন করতে

```json
{
  "palette": {
    "primary": "#FFFFFF",   ← ক্যাপশন টেক্সট রঙ
    "dark": "#141413",      ← ব্যাকগ্রাউন্ড রঙ
    "accent": "#D97757",    ← হাইলাইট রঙ (শব্দ হাইলাইট, আন্ডারলাইন)
    "easing": "expo-out"    ← অ্যানিমেশন কার্ভ
  }
}
```

**easing বিকল্পসমূহ:**
- `expo-out` — দ্রুত শুরু, ধীর শেষ (ডিফল্ট, স্মুদ)
- `linear` — সমান গতি
- `ease-in-out` — ধীর শুরু, ধীর শেষ
- `bounce` — বাউন্স ইফেক্ট

---

### ক্যাপশন সেটিংস পরিবর্তন করতে

```json
{
  "caption": {
    "words_per_screen": "2-4",   ← প্রতি স্ক্রিনে ২-৪ শব্দ (viral style)
    "font_size_1080": 68,        ← 1080px-এ ফন্ট সাইজ
    "fade_ms": 80,               ← ক্যাপশন ফেড সময় (ms)
    "position": "lower_middle"   ← ক্যাপশনের অবস্থান
  }
}
```

**position বিকল্পসমূহ:**
- `lower_middle` — নিচে মাঝখানে (ডিফল্ট, সবচেয়ে দেখা হয়)
- `upper_middle` — উপরে মাঝখানে
- `center` — একদম মাঝখানে

---

### অডিও সেটিংস পরিবর্তন করতে

```json
{
  "audio": {
    "target_lufs": -14,     ← YouTube/Spotify স্ট্যান্ডার্ড
    "peak_dbtp": -1,        ← পিক লিমিটার
    "highpass_hz": 80,      ← 80Hz-এর নিচের ঘোঁৎ ঘোঁৎ সাউন্ড কাটে
    "music_duck_db": -20    ← কথার সময় মিউজিক কমানো
  }
}
```

**LUFS রেফারেন্স:**
| প্ল্যাটফর্ম | প্রস্তাবিত LUFS |
|-------------|----------------|
| YouTube | -14 LUFS |
| Spotify | -14 LUFS |
| Instagram | -16 LUFS |
| TikTok | -14 LUFS |
| Broadcast (TV) | -23 LUFS |

---

### এক্সপোর্ট সেটিংস পরিবর্তন করতে

```json
{
  "export": {
    "codec": "h264",          ← সবচেয়ে কম্প্যাটিবল
    "bitrate_mbps": 18,       ← Instagram/TikTok-এর জন্য 18Mbps যথেষ্ট
    "audio_codec": "aac",
    "audio_bitrate": "320k"
  }
}
```

**codec বিকল্পসমূহ:**
- `h264` — সর্বোচ্চ কম্প্যাটিবিলিটি (ডিফল্ট)
- `h265` / `hevc` — ছোট ফাইল সাইজ, কিন্তু পুরনো ডিভাইসে নাও চলতে পারে
- `vp9` — YouTube-এর জন্য ভালো

---

## 🚨 ট্রাবলশুটিং (সাধারণ সমস্যা ও সমাধান)

### সমস্যা ১: "node: command not found"

**কারণ:** Node.js ইনস্টলড নেই বা PATH-এ নেই।

**সমাধান:**
```bash
# macOS/Linux
which node          # কোথায় আছে দেখো
echo $PATH          # PATH চেক করো

# nvm ব্যবহারকারীদের জন্য
nvm install 22
nvm use 22

# Windows (PowerShell)
Get-Command node    # কোথায় আছে দেখো
# ইনস্টল না থাকলে: winget install OpenJS.NodeJS.LTS
```

---

### সমস্যা ২: "ffmpeg: command not found"

**কারণ:** ffmpeg ইনস্টলড নেই বা PATH-এ নেই।

**সমাধান:**
```bash
# macOS
brew install ffmpeg

# Windows — ম্যানুয়ালি PATH যোগ করো:
# ১. ffmpeg.exe ডাউনলোড করো: https://ffmpeg.org/download.html
# ২. C:\ffmpeg\bin ফোল্ডারে রাখো
# ৩. System Properties → Environment Variables → Path → C:\ffmpeg\bin যোগ করো
# ৪. PowerShell রিস্টার্ট করো

# Linux
sudo apt-get install ffmpeg
```

---

### সমস্যা ৩: HyperFrames ইনস্টল হচ্ছে না

**কারণ:** npm রেজিস্ট্রি সমস্যা বা নেটওয়ার্ক ইস্যু।

**সমাধান:**
```bash
# npm cache পরিষ্কার করো
npm cache clean --force

# আবার ইনস্টল করো
npm install -g hyperframes

# যদি তারপরও না হয়, npx দিয়ে ব্যবহার করো
npx hyperframes --version
```

---

### সমস্যা ৪: SRT ফাইল থাকা সত্ত্বেও AI ট্রান্সক্রাইব করছে

**কারণ:** SRT ফাইলের এক্সটেনশন ভুল (`.txt` বা `.vtt` হতে পারে)।

**সমাধান:**
```bash
# ফাইলের এক্সটেনশন চেক করো
ls srt/

# .vtt থেকে .srt-এ রূপান্তর করো
# ffmpeg দিয়ে:
ffmpeg -i subtitle.vtt subtitle.srt
```

---

### সমস্যা ৫: config.json এরর দেখাচ্ছে

**কারণ:** JSON ফরম্যাট ভুল (comma, quote বা bracket সমস্যা)।

**সমাধান:**
```bash
# JSON যাচাই করো (Node.js দিয়ে)
node -e "JSON.parse(require('fs').readFileSync('config.json','utf8'))"

# সঠিক হলে কোনো আউটপুট নেই
# ভুল থাকলে লাইন নম্বরসহ এরর দেখাবে
```

---

### সমস্যা ৬: বাংলা ক্যাপশন দেখাচ্ছে না বা ভেঙে যাচ্ছে

**কারণ:** বাংলা ফন্ট `assets/fonts/` ফোল্ডারে নেই।

**সমাধান:**
```bash
# Google Fonts থেকে Hind Siliguri নামাও
# https://fonts.google.com/specimen/Hind+Siliguri

# এরপর assets/fonts/ ফোল্ডারে রাখো:
# assets/fonts/caption-regular.woff2
# assets/fonts/caption-bold.woff2

# config.json চেক করো:
# "language": "bn"  ← এটা সেট আছে কিনা
```

---

### সমস্যা ৭: আউটপুট ভিডিও খুব বড় (file size বেশি)

**কারণ:** `bitrate_mbps` বেশি সেট আছে।

**সমাধান:**
```json
// config.json-এ পরিবর্তন করো:
{
  "export": {
    "bitrate_mbps": 8,     // 18 থেকে 8-এ নামাও
    "codec": "h265"        // h264 থেকে h265-এ পরিবর্তন করো (ছোট ফাইল)
  }
}
```

---

### সমস্যা ৮: Environment check পাস হচ্ছে না (সব ঠিক থাকলেও)

**কারণ:** Terminal/PowerShell PATH refresh হয়নি।

**সমাধান:**
```bash
# macOS/Linux — নতুন terminal খোলো অথবা:
source ~/.bashrc    # bash ব্যবহারকারীদের জন্য
source ~/.zshrc     # zsh ব্যবহারকারীদের জন্য

# Windows — PowerShell বন্ধ করে আবার খোলো
# অথবা:
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

---

## 📚 আরো সম্পদ

| রিসোর্স | লিংক |
|---------|-------|
| HyperFrames ডকুমেন্টেশন | `npx hyperframes --help` |
| ffmpeg ডকুমেন্টেশন | https://ffmpeg.org/documentation.html |
| Node.js LTS ডাউনলোড | https://nodejs.org/en/download |
| Royalty-free মিউজিক | https://pixabay.com/music/ |
| Royalty-free SFX | https://mixkit.co/free-sound-effects/ |
| Google Fonts (বাংলা) | https://fonts.google.com/?subset=bengali |

---

## 📄 লাইসেন্স

এই স্কিলটি AI Systems Lab কর্তৃক তৈরি। ব্যক্তিগত ও বাণিজ্যিক উভয় ব্যবহারের জন্য অনুমোদিত।

---

*স্কিল ভার্সন: 2.0 | লেখক: AI Systems Lab*
