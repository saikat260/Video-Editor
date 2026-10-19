# 🎬 AI Video Editor: Full-Stack Automated Video Editing Suite

> **Edit 100% with AI Code Agents · HyperFrames · ffmpeg**  
> *Roberto Nickson (RPN) & Kallaway style: clean, cinematic, restrained, and fast-paced.*  
> No video editing software opened (Premiere Pro, DaVinci Resolve, or CapCut).  
> Every effect you name in the script happens on screen the exact moment you say it.

---

## 📑 সূচিপত্র (Table of Contents)
1. [প্রজেক্ট পরিচিতি (Project Overview)](#-প্রজেক্ট-পরিচিতি-project-overview)
2. [সিস্টেম রিকোয়ারমেন্টস ও ইনস্টলেশন গাইড (Setup & Installation)](#-সিস্টেম-রিকোয়ারমেন্টস-ও-ইনস্টলেশন-গাইড-setup--installation)
   - [Windows ইনস্টলেশন](#1-windows-ইনস্টলেশন)
   - [macOS ইনস্টলেশন](#2-macos-ইনস্টলেশন)
   - [Linux / WSL ইনস্টলেশন](#3-linux--wsl-ইনস্টলেশন)
3. [প্রজেক্ট ফোল্ডার স্ট্রাকচার (Project Structure)](#-প্রজেক্ট-ফোল্ডার-স্ট্রাকচার-project-structure)
4. [৬টি মডুলার স্কিল আর্কিটেকচার (The 6 Modular Skills)](#-৬টি-মডুলার-স্কিল-আর্কিটেকচার-the-6-modular-skills)
5. [রেকর্ডিংয়ের ৩টি গোল্ডেন রুলস (Recording Rules)](#-রেকর্ডিংয়ের-৩টি-গোল্ডেন-রুলস-recording-rules)
6. [মাস্টার প্রম্পট (The Master Prompt)](#-মাস্টার-প্রম্পট-the-master-prompt)
7. [SRT ফোল্ডার ও বাংলা ভাষা ব্যবহারের নিয়ম (Bengali & SRT Policy)](#-srt-ফোল্ডার-ও-বাংলা-ভাষা-ব্যবহারের-নিয়ম-bengali--srt-policy)
8. [ইমেজ এবং B-roll ফুটেজ ব্যবহারের নিয়ম (Images & B-roll Integration)](#-ইমেজ-এবং-b-roll-ফুটেজ-ব্যবহারের-নিয়ম-images--b-roll-integration)
9. [৮টি স্পেশালাইজড রিভিশন প্রম্পট (The 8 Revision Prompts)](#-৮টি-স্পেশালাইজড-রিভিশন-প্রম্পট-the-8-revision-prompts)
   - [01: Push the Energy](#revision-01-push-the-energy)
   - [02: Use Your Brand](#revision-02-use-your-brand)
   - [03: Motion on Every Line](#revision-03-motion-on-every-line)
   - [04: Better Hook & Seamless Loop](#revision-04-better-hook--seamless-loop)
   - [05: Make a Moment Pop (Glance Cover)](#revision-05-make-a-moment-pop-glance-cover)
   - [06: Use Real Footage (Before / After)](#revision-06-use-real-footage-before--after)
   - [07: Tease the Prompt (Terminal Scroll)](#revision-07-tease-the-prompt-terminal-scroll)
   - [08: Fix a Take](#revision-08-fix-a-take)
10. [ফাইনাল কোয়ালিটি চেক প্রম্পট (Final QC Pass)](#-ফাইনাল-কোয়ালিটি-চেক-প্রম্পট-final-qc-pass)
11. [সমস্যা ও সমাধান (Troubleshooting)](#-সমস্যা-ও-সমাধান-troubleshooting)

---

## 🌟 প্রজেক্ট পরিচিতি (Project Overview)

এই রিপোজিটরিটি হলো একটি সম্পূর্ণ কোড-বেসড এআই ভিডিও এডিটিং অটোমেশন স্যুট। প্রথাগত কোনো ভিডিও এডিটিং অ্যাপ না খুলে শুধু কাঁচা ফুটেজ (`raw/`) ফোল্ডারে রেখে এআই কোডিং এজেন্টের (Claude Code / Antigravity / Gemini CLI) মাধ্যমে স্ক্রিপ্ট অনুযায়ী মিলিসেকেন্ড নির্ভুলতায় ভাইরাল রিল তৈরি করা যায়।

### আসল এডিটের পরিসংখ্যান:
* **কাঁচা ফুটেজ:** ৩ মিনিট ১৭.৪ সেকেন্ড (৩টি ক্লিপ)
* **ফাইনাল রিল:** ৩৩.৬ সেকেন্ড (1080x1920, 9:16)
* **বাদ দেওয়া টেক:** ৩৭টি (২০টি অল্টারনেট টেক + ১৭টি রিস্টার্ট)
* **কেটে ফেলা বিরতি:** ০.১৫ সেকেন্ডের বেশি ৭৫টি নীরবতা
* **সিঙ্ক টাইমিং:** প্রতিটি ইফেক্ট উচ্চারিত শব্দের ১–২ ফ্রেমের মধ্যে কার্যকর

---

## ⚙️ সিস্টেম রিকোয়ারমেন্টস ও ইনস্টলেশন গাইড (Setup & Installation)

এই স্যুটটি সঠিকভাবে চালানোর জন্য নিচের ৩টি প্রধান টুলস সিস্টেমে ইনস্টল থাকতে হবে:
1. **Node.js (v22 বা তার বেশি)** — HyperFrames ও CLI টুলস রান করার জন্য।
2. **ffmpeg (সর্বশেষ ভার্সন)** — অডিও/ভিডিও প্রসেসিং, ফিল্টারিং ও রেন্ডারিংয়ের জন্য।
3. **Claude Code / AI Coding CLI** — পুরো কোডিং ও পাইপলাইন অর্কেস্ট্রেট করার জন্য।

---

### 1. Windows ইনস্টলেশন

Windows-এ PowerShell (Administrator) ওপেন করে নিচের কমান্ডগুলো চালান:

#### Step A: Node.js 22+ ইনস্টল করুন
```powershell
# winget ব্যবহার করে Node.js LTS/Current ইনস্টল:
winget install OpenJS.NodeJS -e

# ইনস্টলেশন যাচাই করুন (Node 22+ হওয়া আবশ্যক):
node --version
npm --version
```

#### Step B: ffmpeg ইনস্টল করুন
```powershell
# winget দিয়ে ffmpeg ইনস্টল:
winget install Gyan.FFmpeg -e

# অথবা Chocolatey দিয়ে:
# choco install ffmpeg -y

# নতুন উইন্ডো খুলে যাচাই করুন:
ffmpeg -version
```
> *নোট: যদি কমান্ড কাজ না করে, তবে নিশ্চিত করুন `C:\ffmpeg\bin` আপনার System PATH-এ যুক্ত আছে।*

#### Step C: HyperFrames ইনস্টল করুন
```powershell
npm install -g hyperframes
# যাচাই:
npx hyperframes --version
```

#### Step D: Claude Code ইনস্টল করুন
```powershell
irm https://claude.ai/install.ps1 | iex
# অথবা npm এর মাধ্যমে:
# npm install -g @anthropic-ai/claude-code
claude --version
```

---

### 2. macOS ইনস্টলেশন

macOS Terminal-এ Homebrew দিয়ে সবকিছু এক ক্লিকে ইনস্টল করা যায়:

```bash
# 1. Homebrew আপডেট করুন
brew update

# 2. Node.js এবং ffmpeg ইনস্টল করুন
brew install node@22 ffmpeg

# 3. Node PATH লিঙ্ক করুন
brew link node@22 --force --overwrite

# 4. HyperFrames ইনস্টল করুন
npm install -g hyperframes

# 5. Claude Code ইনস্টল করুন
curl -fsSL https://claude.ai/install.sh | bash

# 6. সবকিছু ঠিক আছে কিনা যাচাই করুন
node --version
ffmpeg -version
npx hyperframes --version
claude --version
```

---

### 3. Linux / WSL ইনস্টলেশন

Ubuntu/Debian বা WSL2-তে:

```bash
# 1. সিস্টেম আপডেট ও প্যাকেজ ইনস্টল
sudo apt update && sudo apt install -y curl git build-essential ffmpeg

# 2. Node.js 22 ইনস্টল (NodeSource Repo থেকে)
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# 3. HyperFrames ইনস্টল
sudo npm install -g hyperframes

# 4. Claude Code ইনস্টল
curl -fsSL https://claude.ai/install.sh | bash

# 5. ভেরিফিকেশন
node --version
ffmpeg -version
npx hyperframes --version
claude --version
```

---

## 📁 প্রজেক্ট ফোল্ডার স্ট্রাকচার (Project Structure)

ভিডিও এডিট শুরু করার আগে আপনার প্রজেক্ট ফোল্ডারটি নিচের মতো সাজিয়ে নিন (অথবা আমাদের `video-project-scaffolder` স্কিল দিয়ে স্বয়ংক্রিয়ভাবে বানিয়ে নিন):

```
my-video-project/
├── raw/                    # আপনার রেকর্ড করা সব কাঁচা টেক (.mp4/.mov)
├── srt/                    # আগে থেকে থাকা SRT সাবটাইটেল (বাংলা/যেকোনো ভাষা)
├── script.txt              # আপনার ফাইনাল স্ক্রিপ্ট (Sync Map এর ভিত্তি)
├── old/                    # পুরনো কোনো ভিডিও (Before/After রি-এডিটের জন্য - অপশনাল)
├── assets/
│   ├── music/              # ব্যাকগ্রাউন্ড মিউজিক ট্র্যাক (.mp3/.wav)
│   ├── sfx/                # সাউন্ড ইফেক্ট (whoosh, pop, click, riser, swoosh)
│   ├── fonts/              # কাস্টম ক্যাপশন ফন্ট (.woff2/.ttf/.otf)
│   ├── images/             # লোগো, প্রোডাক্ট ছবি, গ্রাফিক্স বা স্ক্রিনশট (.png/.jpg)
│   └── broll/              # সাপোর্টিং B-roll ভিডিও ক্লিপ (.mp4/.mov)
├── takes.md                # এআই নির্বাচিত সেরা টেক এবং টাইমস্ট্যাম্পের লগ
├── config.json             # কালার প্যালেট, সেফ জোন ও এক্সপোর্ট কনফিগ
├── recording-guide.md      # রেকর্ডিংয়ের গোল্ডেন রুলস
└── output/                 # ফাইনাল রেন্ডার আউটপুট
    ├── preview/            # ড্রাফট প্রিভিউ ও প্রতিটি বিটের স্টিল ফ্রেম
    └── final.mp4           # 1080x1920 রেজোলিউশনের ফাইনাল রিল
```

---

## 🧩 পূর্ণাঙ্গ স্কিল স্যুট ও মোশন গ্রাফিক্স আর্কিটেকচার (Complete Skills Suite)

এই রিপোজিটরিতে ৬টি কোর ভিডিও এডিটিং স্কিল এবং ১০টি স্পেশালাইজড মোশন গ্রাফিক্স/SVG স্কিল একীভূত (Integrated) করা হয়েছে:

### কোর ভিডিও এডিটিং পাইপলাইন (Core Pipeline):
| ক্রম | স্কিল ফোল্ডার | ভূমিকা ও দায়িত্ব |
| :--- | :--- | :--- |
| **১** | **[`video-project-scaffolder`](skills/video-project-scaffolder/)** | প্রজেক্ট শুরু করার সাথে সাথে প্রয়োজনীয় সব ফোল্ডার ও টেমপ্লেট ফাইল তৈরি করে এবং সিস্টেম চেক করে। |
| **২** | **[`ai-video-director`](skills/ai-video-director/)** | ৯-ধাপের মাস্টার পাইপলাইন পরিচালনা করে, সেরা টেক বাছাই করে এবং সিন-বাই-সিন মোশন রাউটিং করে। |
| **৩** | **[`ffmpeg-video-cutter`](skills/ffmpeg-video-cutter/)** | অডিও ওয়েভফর্ম দেখে ০.১৫s পজ ছাঁটাই, ফেস ট্র্যাকিং জুম (100% vs 120%) এবং S-curve গ্রেডিং করে। |
| **৪** | **[`hyperframes-graphics-engine`](skills/hyperframes-graphics-engine/)** | শব্দ ধরে হাইলাইট হওয়া ক্যাপশন, সাবজেক্ট কাটআউট (মাথার পেছনে গ্রাফিক্স) ও সেফ-জোন নিশ্চিত করে। |
| **৫** | **[`audio-sound-designer`](skills/audio-sound-designer/)** | ভয়েস ক্লিনআপ (80Hz হাইপাস), অটো-ডাকিং (-২০ dB), -14 LUFS মাস্টারিং এবং অটো SFX সিন্থেসিস করে। |
| **৬** | **[`video-qc-revisions`](skills/video-qc-revisions/)** | ফাইনাল রেন্ডারের পূর্বে ৯-দফা QC অডিট (ভিজ্যুয়াল বৈচিত্র্য সহ) এবং ৮টি স্পেশালাইজড রিভিশন প্রম্পট পরিচালনা করে। |

### ডায়নামিক মোশন গ্রাফিক্স ও SVG ইঞ্জিন (Integrated Motion Skills from iart-ai):
| ক্রম | স্কিল ফোল্ডার | ভূমিকা ও সিনের ব্যবহার |
| :--- | :--- | :--- |
| **৭** | **[`svg-animation`](skills/svg-animation/)** | পিওর ভেক্টর SVG হ্যান্ড-ড্রন দাগ, সার্কেল, অ্যারো ও পাথ ড্রয়িং (`stroke-dashoffset`) অ্যানিমেশন। |
| **৮** | **[`talking-head-b-roll`](skills/talking-head-b-roll/)** | টকিং-হেড ভিডিওর কথার ওপর ভিত্তি করে ক্যানভাসে স্বয়ংক্রিয় ফুল-ফ্রেম বা ওভারলে B-roll তৈরি। |
| **৯** | **[`chart-animation`](skills/chart-animation/)** | ডাটা-ড্রিভেন বার চার্ট রেস, লাইন গ্রাফ রিভিল, নাম্বার কাউন্টার ও টিকার অ্যানিমেশন। |
| **১০** | **[`animated-infographic`](skills/animated-infographic/)** | পরিসংখ্যান কার্ড, মাল্টি-পার্ট ইনফোগ্রাফিক ও তুলনামূলক ডেটা ভিজ্যুয়ালাইজেশন। |
| **১১** | **[`kinetic-typography`](skills/kinetic-typography/)** | কাইনেটিক টাইপোগ্রাফি, পাঞ্চলাইন রিভিল ও আধুনিক টেক্সট অ্যানিমেশন। |
| **১২** | **[`lower-thirds`](skills/lower-thirds/)** | বক্তার নাম, পদবী, সোশ্যাল হ্যান্ডেল ও সাবস্ক্রাইব রিমাইন্ডারের ট্রান্সপারেন্ট ওভারলে। |
| **১৩** | **[`diagram-animation`](skills/diagram-animation/)** | প্রসেস ফ্লোচার্ট, সিস্টেম আর্কিটেকচার ও স্টেপ-বাই-স্টেপ এক্সপ্লেইনার ডায়াগ্রাম। |
| **১৪** | **[`logo-animation`](skills/logo-animation/)** | ভেক্টর ব্র্যান্ড লোগো স্টিং, উন্মোচন ও অ্যানিমেটেড আইকন। |
| **১৫** | **[`lottie-animation`](skills/lottie-animation/)** | Lottie ভেক্টর অ্যানিমেশন ইন্টিগ্রেশন ও প্লেব্যাক। |
| **১৬** | **[`motion-background`](skills/motion-background/)** | ডায়নামিক অ্যাম্বিয়েন্ট ব্যাকগ্রাউন্ড ও সিনেমাটিক গ্রেডিয়েন্ট মোশন। |

---

## 🎨 ডায়নামিক সিন-ভিত্তিক মোশন গ্রাফিক্স রাউটিং (Dynamic Scene-to-Motion Skills Router)

এআই যখন আপনার স্ক্রিপ্ট ও অডিও এনালাইসিস করে ভিডিও এডিট করবে, তখন এটি **প্রতিটি সিনের অর্থ অনুযায়ী স্বয়ংক্রিয়ভাবে সঠিক মোশন স্কিলটি নির্বাচন করে কোড জেনারেট করবে**:

```
┌───────────────────────────────────────────────────────────────────────────────────────────────┐
│                       DYNAMIC SCENE-TO-MOTION-SKILL ROUTING ENGINE                            │
├────────────────────────────────┬──────────────────────────┬───────────────────────────────────┤
│ সিনের বিষয় ও কথা (Spoken Intent)│ যে মোশন স্কিলটি ট্রিগার হবে │ কী জেনারেট ও রেন্ডার করবে         │
├────────────────────────────────┼──────────────────────────┼───────────────────────────────────┤
│ ১. ভেক্টর ড্রয়িং, দাগ দেওয়া,     │ `skills/svg-animation`   │ • হ্যান্ড-ড্রন সার্কেল ও আন্ডারলাইন │
│    তীর চিহ্ন, শেপ মরফিং, আইকন   │                          │ • stroke-dashoffset ড্রয়িং অ্যানিমেশন│
│    বা লোগো অ্যানিমেশন          │                          │ • কাস্টম ভেক্টর আইকন ও পাথ মরফিং   │
├────────────────────────────────┼──────────────────────────┼───────────────────────────────────┤
│ ২. গল্প, কাজের দৃশ্য, দীর্ঘ     │ `skills/talking-head-b-  │ • ফুল-ফ্রেম বা ওভারলে B-roll দৃশ্য │
│    ব্যাখ্যা বা উদাহরণের সিন     │  roll`                   │ • ক্যানভাসে কোড-ড্রন মোশন দৃশ্য    │
│    (Talking Head B-roll)       │                          │ • ৩টি ট্রিটমেন্ট (Speaker/Overlay/Cover)│
├────────────────────────────────┼──────────────────────────┼───────────────────────────────────┤
│ ৩. সংখ্যা, পার্সেন্টেজ, ডাটা,    │ `skills/chart-animation` │ • অ্যানিমেটেড বার চার্ট রেস         │
│    গ্রোথ, তুলনা বা মেট্রিক্স     │                          │ • লাইন চার্ট রিভিল ও কাউন্টার     │
│                                │                          │ • নাম্বার টিকার ও মেট্রিক ট্র্যাকিং│
├────────────────────────────────┼──────────────────────────┼───────────────────────────────────┤
│ ৪. মাল্টি-পার্ট ডাটা, পয়েন্ট ও │ `skills/animated-        │ • ইনফোগ্রাফিক কার্ড ও তুলনা চার্ট   │
│    ইনফোগ্রাফিক সম্বলিত সিন      │  infographic`            │ • ৩-৪টি বিষয়ের ভিজ্যুয়াল ব্রেকডাউন │
├────────────────────────────────┼──────────────────────────┼───────────────────────────────────┤
│ ৫. পাঞ্চলাইন, বোল্ড স্টেটমেন্ট   │ `skills/kinetic-         │ • কাইনেটিক টাইপোগ্রাফি             │
│    বা বড় টেক্সট রিভিল          │  typography`             │ • কাস্টম ইজিং টেক্সট অ্যানিমেশন    │
├────────────────────────────────┼──────────────────────────┼───────────────────────────────────┤
│ ৬. বক্তার নাম, পদবী, সোশ্যাল    │ `skills/lower-thirds`    │ • ট্রান্সপারেন্ট ব্রডকাস্ট লোয়ার-থার্ড │
│    হ্যান্ডেল বা সাবস্ক্রাইব    │                          │ • নেম স্ট্র্যাপ ও চ্যানেল ওভারলে  │
├────────────────────────────────┼──────────────────────────┼───────────────────────────────────┤
│ ৭. কাজের প্রসেস, স্টেপস, ফ্লো  │ `skills/diagram-animation`│ • ইন্টারেক্টিভ সিস্টেম ফ্লোচার্ট   │
│    বা আর্কিটেকচার ডায়গ্রাম     │                          │ • তীর চিহ্ন দিয়ে স্টেপ বাই স্টেপ ফ্লো│
├────────────────────────────────┼──────────────────────────┼───────────────────────────────────┤
│ ৮. ব্র্যান্ড লোগো, চ্যানেল ইন্ট্রো│ `skills/logo-animation`  │ • ভেক্টর লোগো রিভিল ও স্টিং       │
└────────────────────────────────┴──────────────────────────┴───────────────────────────────────┘
```

---

## 🎙️ রেকর্ডিংয়ের ৩টি গোল্ডেন রুলস (Recording Rules for Real Videos)

বাস্তব ভিডিওর জন্য ক্যামেরা অন করে এই ৩টি সহজ নিয়ম মেনে রেকর্ড করুন:

1. **Repeat (পুনরাবৃত্তি):** কোনো লাইনে তোতলামি বা ভুল হলে ক্যামেরা বন্ধ করবেন না। ১ সেকেন্ড থেমে একই লাইন আবার স্বাভাবিকভাবে বলুন। এআই স্বয়ংক্রিয়ভাবে শেষ ভালো টেকটি নির্বাচন করবে।
2. **Speak Naturally (স্বাভাবিক কথন):** কোনো ইফেক্ট, কাট বা ট্রানজিশনের নাম মুখে বলার কোনো প্রয়োজন নেই! স্বাভাবিকভাবে আপনার বক্তব্য দিন। এআই স্ক্রিপ্ট ও অডিও অ্যানালাইসিস করে নিজেই বুঝে নেবে কোথায় কোন ইফেক্ট দিতে হবে।
3. **Glance (স্ক্রিনে তাকানো):** ল্যাপটপ বা প্রম্পটারের দিকে তাকালে কোনো সমস্যা নেই। এআই ওই মুহূর্তগুলোকে ফুল-স্ক্রিন মোশন গ্রাফিক্স বা B-roll দিয়ে ঢেকে দেবে।

---

## 🧠 এআই কীভাবে কোনো কথা ছাড়াই নিজে থেকে কাট ও ইফেক্ট বোঝে? (Semantic Intelligence)

বাস্তব ভিডিওতে এআই নিচের ৭টি অ্যালগরিদমিক নিয়মে স্বয়ংক্রিয়ভাবে কাটিং ও মোশন গ্রাফিক্স পরিচালনা করে:

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                    AUTONOMOUS SEMANTIC INTELLIGENCE                          │
├──────────────────────────┬───────────────────────────────────────────────────┤
│ যা আপনি বলছেন (Semantic)  │ এআই স্বয়ংক্রিয়ভাবে যা করবে (Visual & Audio)        │
├──────────────────────────┼───────────────────────────────────────────────────┤
│ বাক্য শেষ / নতুন কথা শুরু    │ জাম্প কাট + ফ্রেমিং অল্টারনেশন (Wide 100% ↔ Close 120%)│
│ গুরুত্বপূর্ণ কথা / পাঞ্চলাইন  │ ০.৪s পুশ-ইন জুম (115%) + সফট Whoosh SFX + হাইলাইট  │
│ সংখ্যা, পার্সেন্টেজ বা ডাটা │ বোল্ড কাইনেটিক কাউন্টার কার্ড (মাথার পেছনে) + Pop SFX│
│ টুল, অ্যাপ বা ব্র্যান্ডের নাম │ assets/images/ থেকে লোগো/স্ক্রিনশট গ্লাস-কার্ডে প্রদর্শন  │
│ কাজের দৃশ্য / পরিস্থিতির বর্ণনা│ assets/broll/ থেকে প্রাসঙ্গিক B-roll ক্লিপে কাট       │
│ একটানা ৩-৪ সেকেন্ড স্থির     │ দর্শক ধরে রাখতে স্বয়ংক্রিয় সূক্ষ্ম ফ্রেমিং শিফট       │
│ ক্যামেরার বাইরে তাকানো      │ ফুল-স্ক্রিন মোশন গ্রাফিক্স বা কার্ড দিয়ে গ্ল্যান্স ঢাকা│
└──────────────────────────┴───────────────────────────────────────────────────┘
```

---

## 🚀 মাস্টার প্রম্পট ১: বাস্তব কনটেন্টের জন্য (Autonomous Semantic Director — Recommended)

এটি হলো আপনার আসল ভিডিওর জন্য মাস্টার প্রম্পট। এখানে এআই নিজে থেকে আপনার বক্তব্যের অর্থ বুঝে সম্পূর্ণ এডিটিং করবে:

```markdown
Edit a 9:16 short-form video for Instagram, TikTok, and YouTube Shorts using HyperFrames and ffmpeg following the skills/ai-video-director orchestrator pipeline.
The bar is a premium creator edit in the style of Roberto Nickson (RPN) and Kallaway: clean, cinematic, fast, and restrained. It must NOT look like a generic AI template.
I am speaking NATURALLY about my topic — I do NOT say effect names out loud. You must act as an Intelligent Autonomous Video Director. Work through every step without stopping until the preview.

PROJECT FILES:
- raw/ every take I recorded, in recording order
- srt/ pre-transcribed SRT subtitles (CRITICAL: if present, bypass transcription entirely and preserve Bengali Unicode word timestamps)
- script.txt the spoken script (semantic narrative reference)
- assets/music/ background track
- assets/sfx/ whoosh, pop, click, riser, swoosh (if empty, synthesize soft sine clicks or keep silent)
- assets/fonts/ caption font (if empty, default to Montserrat ExtraBold or Noto Sans Bengali Bold)
- assets/images/ logos, screenshots, infographics to display when tools/topics are mentioned
- assets/broll/ supporting B-roll video clips to cut to during explanations
- output/ final export location

SKILLS SUITE INTEGRATION:
You have access to 16 specialized skills in `skills/`. `skills/ai-video-director` is your master orchestrator. During Step 6, you must dynamically read and invoke the specialized motion skills (`skills/chart-animation`, `skills/svg-animation`, `skills/diagram-animation`, `skills/lower-thirds`, `skills/talking-head-b-roll`, `skills/kinetic-typography`, `skills/animated-infographic`, `skills/logo-animation`, etc.) depending on the scene's semantic intent.

STEP 1: BEST TAKE SELECTION
- Check srt/ folder FIRST. If any .srt file exists, SKIP transcription and use word-level timestamps directly. If empty, run `npx hyperframes transcribe --word-level true`.
- Match every script line to its takes. Pick the LAST complete take of each line (it usually has the best energy), unless it has stumbles or clipped words.
- Write takes.md: each line, chosen take, timestamps, and reason.

STEP 2: TIGHT CINEMATIC CUT
- Assemble selected takes in script order.
- Remove all silent pauses > 0.15s, breaths, ums, and restarts.
- Never cut mid-word. Leave 2-frame handles on every cut with a 3ms audio crossfade to prevent clicks.

STEP 3: CINEMATIC GRADE
- Clean S-curve contrast, warm natural skin tones, slightly cool shadows, soft vignette, very subtle film grain.
- Match exposure and white balance across all takes so cuts don't flicker.

STEP 4: FACE-TRACKED DYNAMIC ZOOMS
- Track my face so every zoom centers on my eyes.
- Two framings: Wide at 100% and Close at 120%. Alternate framings on jump cuts (sentence/thought boundaries) so the cuts look deliberate and dynamic.
- If source is 1080p, cap zooms at 115% for sharpness.

STEP 5: KINETIC CAPTIONS
- 2 to 4 words per caption, word-level timing, bold sans typography, lower middle of the frame (safe zone).
- Active spoken word highlighted in accent orange #D97757 (80ms fade-in, no bouncing).
- Strict Safe Zone: nothing in the top 220px, bottom 420px, or right 140px.

STEP 6: AUTONOMOUS SEMANTIC EDITING & SCENE-BY-SCENE REASONING ENGINE
Analyze my spoken transcript semantics and execute the 5-Stage Cognitive Framework:
1. Break speech into scene units at sentence boundaries and natural pauses (> 0.15s).
2. For each scene, analyze intent and output the mandatory Scene Reasoning Log (CoT): Spoken text, intent classification, visual hypothesis, and restraint audit.
3. Autonomously load the matching specialized skill from skills/ (such as skills/svg-animation for hand-drawn highlights/arrows, skills/chart-animation for stats/numbers, skills/diagram-animation for workflows/architectures, skills/lower-thirds for speaker credentials, skills/talking-head-b-roll for narrative cutaways, etc.).
4. Generate isolated scene assets in build/scenes/scene_NN/ in 1080x1920 (9:16) adhering to safe zones (Top: 220px, Bottom: 420px, Right: 140px).
5. Composite scenes sequentially on the master timeline with 2-frame handle accuracy, Roberto Nickson palette (#FFFFFF, #141413, #D97757), and synchronized audio SFX (pop, click, whoosh, riser).
6. Enforce the 2-4s retention rhythm: never stay visually static for more than 4 seconds, and never stack two visual effects simultaneously.

STEP 7: AUDIO MASTERING
- Clean voice: 80Hz high-pass filter, gentle noise suppression, smooth compression.
- Duck background music 18–22 dB under voice; restore music cleanly on pauses and ending.
- Master to -14 LUFS, true peak at -1 dBTP.

STEP 8: PREVIEW & VERIFY
- Render preview (540x960) + stills of major visual beats.
- Run 9-point QC check (sync, safe zone, face clearance, mid-word cuts, audio peaks, seamless loop, motion diversity).
- ⛔ STOP AT PREVIEW. Show me the preview and takes.md. Wait for approval.
- Final export after approval: output/final.mp4, 1080x1920, H.264 ~18 Mbps, AAC 320k.
```

---

## 🎬 মাস্টার প্রম্পট ২: ডেমো রিলের জন্য (Meta / Self-Referential Mode)

যদি আপনি ইচ্ছাকৃতভাবে এডিটিং নিয়ে কোনো ভিডিও বানান—যেখানে আপনি নিজেই মুখে ইফেক্টের নাম বলেন (যেমন: *"the cuts, the colour grade, the zoom ins..."*)—তখন নিচের ডেমো মোড প্রম্পটটি ব্যবহার করুন:

```markdown
Edit a 9:16 short-form video for Instagram and TikTok using HyperFrames and ffmpeg.
The video is about its own edit: every effect I name in the script must happen on screen the moment I say it.
Sync map:
- "cuts": hard jump cut + sharp click SFX.
- "colour grading": drop to ungraded, then thin vertical line sweeps grade back in.
- "captions": captions switch on, top title fades out.
- "zoom ins": smooth push-in + whoosh. "zoom outs": smooth pull-out + whoosh.
- "motion graphics": MOTION GRAPHICS text builds from sliding orange bars (0.8s).
- "sound effects": minimal waveform pulses with whoosh and pop.
- "single editing app": card "Editing apps opened: 0".
- "one prompt": dark terminal card types first line of prompt with counter.
- "Comment": kinetic text + comment bubble. Last frame matches frame 0 (seamless loop).
Work through all steps and stop only at preview.
```

---

## 🇧🇩 SRT ফোল্ডার ও বাংলা ভাষা ব্যবহারের নিয়ম (Bengali & SRT Policy)

### কেন SRT ফোল্ডার ব্যবহার করবেন?
স্বয়ংক্রিয় এআই ট্রান্সক্রিপশন টুলস (যেমন Whisper) অনেক সময় বাংলা কথ্য ভাষা, আঞ্চলিক টান বা মিশ্র ইংরেজি-বাংলা শব্দ (Banglish) নিখুঁতভাবে ধরতে পারে না।
তাই আপনি যদি CapCut, Premiere Pro বা অন্য কোনো টুল দিয়ে তৈরি করা একটি সঠিক `.srt` ফাইল আপনার প্রজেক্টের `srt/` ফোল্ডারে রেখে দেন:

1. এআই **কোনো ট্রান্সক্রিপশন রান করবে না** (সময় ও কম্পিউটিং সাশ্রয় হবে)।
2. সরাসরি আপনার `.srt` ফাইলের শব্দ এবং মিলিসেকেন্ড টাইমস্ট্যাম্প ধরে কাট ও ক্যাপশন তৈরি হবে।
3. যুক্তাক্ষর, হসন্ত ও দাড়ি সহ বাংলা ফন্ট নিখুঁতভাবে রেন্ডার হবে।

### বাংলা ভিডিওর জন্য প্রম্পট সংযোজন:
যদি ভিডিও বাংলায় হয়, মাস্টার প্রম্পটের সাথে এই অংশটি যুক্ত করে দিন:

```markdown
LANGUAGE & FONT OVERRIDE:
- Language: Bengali (বাংলা).
- SRT Priority: Parse srt/my_subtitles.srt directly. Do not call any transcribe command.
- Caption Font: Use Noto Sans Bengali Bold or Hind Siliguri Bold from assets/fonts/.
- Word Boundaries: Handle Bengali conjuncts (যুক্তাক্ষর) and vowel signs properly without splitting mid-syllable.
- Size: 76px at 1080 wide, line height 1.4.
```

---

## 🖼️ ইমেজ এবং B-roll ফুটেজ ব্যবহারের নিয়ম (Images & B-roll Integration)

আপনার ভিডিওতে শুধু ফেস-ক্যামেরা ছাড়াও যদি কোনো প্রডাক্টের স্ক্রিনশট, লোগো বা প্রাসঙ্গিক ফুটেজ দেখাতে চান:

1. **ইমেজ (`assets/images/`):**
   * কোম্পানির লোগো, চার্ট, অ্যাপ স্ক্রিনশট বা প্রডাক্ট ছবি এখানে রাখুন।
   * সাবজেক্ট কাটআউট টেকনোলজির মাধ্যমে বক্তার মাথার পেছনে অথবা অ্যানিমেটেড ফ্লিপ কার্ডে ইমেজ প্রদর্শিত হবে।
2. **B-roll ফুটেজ (`assets/broll/`):**
   * সাপোর্টিং ভিডিও ক্লিপগুলো এখানে রাখুন (যেমন: `typing-code.mp4`, `analytics-dashboard.mp4`)।
   * স্ক্রিপ্টে ওই বিষয়টি উচ্চারিত হওয়ার সাথে সাথে জাম্প-কাটের বদলে এআই B-roll ফুটেজে কাট করবে এবং মূল ভিডিওর মতো একই সিনেমাটিক গ্রেড যুক্ত করবে।

### B-roll ও ইমেজের জন্য অতিরিক্ত প্রম্পট নির্দেশনা:
```markdown
ASSET INTEGRATION RULES:
- Scan assets/images/ and assets/broll/.
- When I mention a specific tool or topic that matches an asset filename, cut to that B-roll or show the image on a rounded corner card (16px border radius).
- Place large cards and infographics BEHIND my head using subject cutout.
- Grade all B-roll to match the main footage's S-curve contrast and warm tones.
```

---

## 🎵 ব্যাকগ্রাউন্ড মিউজিক (BGM) ও সাউন্ড ইফেক্টস (SFX) গাইড

### ১. এসএফএক্স (SFX) কি আপনাকে দিতে হবে?
**উত্তর: না, কোনো বাধ্যবাধকতা নেই।**
- **স্বয়ংক্রিয় সিন্থেসিস (ffmpeg Auto-Synthesis):** যদি `assets/sfx/` ফোল্ডারে আপনি কোনো ফাইল না রাখেন, তবে এআই আটকে যাবে না। এআই ffmpeg-এর `sine` এবং `anoise` অডিও ফিল্টার দিয়ে স্বয়ংক্রিয়ভাবে ৫টি অতি-মার্জিত, ক্লিক-ফ্রি ও মিনিমালিস্টিক সাউন্ড জেনারেট করে নেবে:
  - `click.wav` (1600Hz sine micro-burst) — জাম্প কাটের জন্য
  - `pop.wav` (850Hz → 300Hz pitch-sweep) — কার্ড ও ক্যাপশন পপ-আপের জন্য
  - `whoosh.wav` (Bandpassed white noise sweep) — পুশ-ইন জুমের জন্য
  - `swoosh.wav` (Airy pink noise sweep) — টেক্সট স্লাইড হওয়ার জন্য
  - `riser.wav` (200Hz → 550Hz rising tone) — ক্লাইম্যাক্স বা রিভিলের জন্য

#### এক ক্লিকে ৫টি মিনিমাল সাউন্ড তৈরি করার টার্মিনাল কমান্ড:
```bash
ffmpeg -f lavfi -i "sine=frequency=1600:duration=0.015,afade=t=in:st=0:d=0.002,afade=t=out:st=0.004:d=0.011,volume=0.25" -c:a pcm_s16le assets/sfx/click.wav -y
ffmpeg -f lavfi -i "aeval='0.35*sin(2*PI*(850-9000*t)*t)':d=0.05,afade=t=out:st=0.015:d=0.035" -c:a pcm_s16le assets/sfx/pop.wav -y
ffmpeg -f lavfi -i "anoise=d=0.3:c=white,bandpass=f=650:width_type=h:w=500,afade=t=in:st=0:d=0.12,afade=t=out:st=0.12:d=0.18,volume=0.28" -c:a pcm_s16le assets/sfx/whoosh.wav -y
ffmpeg -f lavfi -i "anoise=d=0.22:c=pink,bandpass=f=950:width_type=h:w=600,afade=t=in:st=0:d=0.07,afade=t=out:st=0.07:d=0.15,volume=0.22" -c:a pcm_s16le assets/sfx/swoosh.wav -y
ffmpeg -f lavfi -i "aeval='0.2*sin(2*PI*(200+350*t)*t)':d=1.0,afade=t=in:st=0:d=0.25,afade=t=out:st=0.75:d=0.25" -c:a pcm_s16le assets/sfx/riser.wav -y
```

---

### ২. বিজিএম (BGM) নিয়ে কী করবেন?
ব্যাকগ্রাউন্ড মিউজিকের জন্য আপনার সামনে **৩টি স্মার্ট বিকল্প** রয়েছে:

1. **লোকাল ট্র্যাক ব্যবহার (Recommended for YouTube Shorts):**
   - Pixabay Music বা YouTube Audio Library থেকে যেকোনো পছন্দের কপিরাইট-ফ্রি Lo-Fi বা অ্যাম্বিয়েন্ট ট্র্যাক এনে `assets/music/` ফোল্ডারে রাখুন।
   - এআই কথা বলার সময় স্বয়ংক্রিয়ভাবে মিউজিক **-২০ dB নিচে নামিয়ে (Duck)** দেবে এবং শেষ শব্দে নিখুঁতভাবে ফেড-আউট করবে।
2. **সোশ্যাল মিডিয়া ট্রেন্ডিং সাউন্ড হ্যাক (Top Creator Secret — Best for Instagram & TikTok 🚀):**
   - ভিডিও রেন্ডারের সময় `assets/music/` খালি রাখুন (মিউজিক ছাড়া রেন্ডার দিন)।
   - ইনস্টাগ্রাম বা টিকটকে আপলোড করার সময় প্ল্যাটফর্মের অডিও লাইব্রেরি থেকে সরাসরি **Trending Sound** সিলেক্ট করুন এবং ভলিউম ৩% থেকে ৫% সেট করুন। এতে অ্যালগরিদমে আপনার ভিডিও ভাইরাল হওয়ার সম্ভাবনা বহুগুণ বেড়ে যায়!
3. **বিজিএম ছাড়া ক্রিস্প ভয়েস মোড (Crisp Studio Voice):**
   - কোনো মিউজিক না থাকলে এআই আপনার ভয়েসের ৮০Hz হাই-পাস ফিল্টার, ডিনয়েজ ও -14 LUFS স্টুডিও মাস্টারিং করবে, ফলে মিউজিক ছাড়াও আপনার বক্তব্য স্পষ্ট ও প্রফেশনাল শোনাবে।

## 🛠️ ৮টি স্পেশালাইজড রিভিশন প্রম্পট (The 8 Revision Prompts)

প্রথম প্রিভিউ দেখার পর নির্দিষ্ট পরিবর্তন করতে এই প্রম্পটগুলো ব্যবহার করুন:

### Revision 01: Push the Energy
*কখন ব্যবহার করবেন: ভিডিওর শুরুটা যদি ধীরগতির বা ফ্ল্যাট মনে হয়।*

```markdown
Raise the energy of this edit without making it look like a template.

OPENING (0:00 to 0:03)
- Frame 0 must be a strong visual: a bold motion graphic or a tight punch-in on my face. No static wide shot.
- Something changes every 1 to 1.5 seconds until 0:03: a cut, a zoom, or a graphic.

THROUGHOUT
- Find every stretch longer than 4 seconds where nothing changes and give it a cut, zoom, b-roll, or graphic.
- Make the motion graphics bigger and bolder: fill more of the frame, stronger contrast, same easing (expo out).

KEEP
- Never stack two effects on the same beat.
- Every effect stays within 2 frames of its word.
- Nothing in the safe zone, no text over my face.
- Don't change the takes, the cuts, or the audio mix.

Then render a new preview and list every change with its timestamp. Rate the edit out of 10 as a senior short-form editor would, and name the three things still holding it back.
```

---

### Revision 02: Use Your Brand
*কখন ব্যবহার করবেন: Claude Orange এর বদলে নিজের ব্র্যান্ড কালার ও ফন্ট ব্যবহার করতে।*

```markdown
Rebrand this edit with my brand. Remove the Claude orange (#D97757) from every graphic, caption, and card.

1. Brand Details:
   - Primary Accent: [আপনার ব্র্যান্ড কালারের হেক্স কোড, e.g., #0066FF]
   - Background Card Color: [e.g., #0F172A]
   - Text Color: [#FFFFFF]
   - Font: Use the font located in assets/fonts/ [বা Google Font নাম]

2. Apply it:
   - Captions in the heading font. Active keyword in the accent colour.
   - Cards, terminal, and timeline in my background and text colours.
   - The accent on one word or element at a time, never as a background.
   - Check every caption is readable over every shot. Add a subtle shadow only where it isn't.

Keep all timings, cuts, motion, and sound exactly as they are. Render the preview and stills of the title and one card.
```

---

### Revision 03: Motion on Every Line
*কখন ব্যবহার করবেন: স্ক্রিপ্টের কিছু লাইনে কোনো ভিজ্যুয়াল মুভমেন্ট না থাকলে।*

```markdown
Go through the script line by line and give every line its own visual beat.

1. Make a table: line, start time, what's on screen now, what you'll add.
2. For any line with nothing happening, add one move that shows what I'm saying:
   - If I say a number, show the number in bold typography.
   - If I name a tool or a thing, show the image from assets/images/ or a card.
   - If I make a claim, show proof (a stat, graph, or B-roll from assets/broll/).
3. Vary the type of move: cut, zoom, B-roll, graphic, kinetic text. No two zooms in a row.

RULES
- One effect at a time. Every effect within 2 frames of its word.
- Nothing in the safe zone, no text over my face.
- If a move doesn't add meaning, leave the line clean. Restraint beats decoration.

Render the preview and show me the table.
```

---

### Revision 04: Better Hook & Seamless Loop
*কখন ব্যবহার করবেন: শুরুর হুক জোরালো করতে এবং ভিডিও শেষ হয়ে আবার প্রথম থেকে সিমলেসলি লুপ করতে।*

```markdown
Rebuild the first and last 2 seconds so the hook lands and the video loops seamlessly.

OPEN
- Frame 0: this video playing small inside a video editor mockup built in HyperFrames, with a preview window and a timeline underneath. Use the real clips and real cut points from this edit, with effect layers lighting up as they play.
- On the words "right now" [বা আপনার হুক ওয়ার্ড]: a fast push-in (0.4s, expo out) through the preview window into the full-screen video.
- The on-screen hook text must be readable on mute within the first second.

END
- On the last word: pull back out into the same editor mockup, landing on a frame IDENTICAL to frame 0.

CHECK
- Export frame 0 and the final frame side by side and confirm they match pixel for pixel.
- Ensure no black frame, flash, or audio pop at the loop point.

Render the preview and show me both frames.
```

---

### Revision 05: Make a Moment Pop (Glance Cover)
*কখন ব্যবহার করবেন: ক্যামেরার বাইরে তাকানো ঢাকতে বা কোনো লাইনে বড় মোশন গ্রাফিক্স তৈরি করতে।*

```markdown
At [0:09 - নির্দিষ্ট সময়], where I look away from the camera, build the biggest motion graphic in the video.

- Cover the glance completely: start on the frame I look away and end on the frame I look back.
- Make it about what I'm saying at that moment: [এখানে বলুন আপনি কী বলছেন].
- Graphics must sit BEHIND my head using the subject cutout.
- One SFX hit (whoosh or impact) on the graphic's main move. Keep the music ducked under my voice.
- Timing: 0.8 to 1.5 seconds, expo-out easing, no shake, no bounce.
- Don't change anything before or after this moment.

Show me 3 stills from the moment (start, peak, end) and the preview.
```

---

### Revision 06: Use Real Footage (Before / After)
*কখন ব্যবহার করবেন: পুরনো দুর্বল ভিডিওর সাথে বর্তমান এডিটের পার্থক্য দেখাতে।*

```markdown
Use my real old content for the before/after comparison beat. I've placed the old video in old/.

1. Pick 2 seconds of it on a single spoken line. Use it untouched in a rounded card with a grey "BEFORE" tag.
2. Re-edit that same moment with this video's grade, zooms, captions, and sound. Place it in a "CLAUDE RE-EDIT" card.
3. Both cards must show the exact same spoken sentence so the comparison is fair.
4. Keep both cards inside the platform safe zone and time the swipe transition to the word "re-edit".

Render the preview and stills of both cards.
```

---

### Revision 07: Tease the Prompt (Terminal Scroll)
*কখন ব্যবহার করবেন: প্রম্পট বা কোডটি স্ক্রিনে স্ক্রোল করিয়ে কিউরিওসিটি তৈরি করতে।*

```markdown
Rebuild the "one prompt" beat as a scrolling dark terminal.

- Dark terminal card (#141413) with rounded corners.
- Type the first line of the prompt in sharp, crisp, readable text.
- Then scroll smoothly through the rest of the prompt. Blur everything after the first 3 lines (Gaussian blur, about 8px) so the length and structure are obvious but the exact words can't be copied.
- Keep the STEP headings slightly less blurred so viewers can see how comprehensive it is.
- Time the scroll to my voice and end it on my call to action.
- Ensure the terminal card stays strictly inside the safe zone.

Render the preview and export a paused full-resolution frame of the blur.
```

---

### Revision 08: Fix a Take
*কখন ব্যবহার করবেন: এআই যদি কোনো লাইনে ভুল টেক বা দুর্বল এনার্জির টেক সিলেক্ট করে।*

```markdown
For the line "[যে লাইনটিতে সমস্যা]", you used the wrong take.

1. List every take of that line from the transcript / SRT with timestamps.
2. Swap to the [e.g., 3rd take / last complete take] which has better energy and no stumbled words.
3. Re-time everything that depends on it: captions, zooms, graphics, and SFX on that line and every line after it.
4. Update takes.md with the reason for this manual override.
5. Check the new cut points: 2-frame handles, no clipped words, 3ms audio crossfade.

Render the preview from 5 seconds before that line to 5 seconds after.
```

---

## ✅ ফাইনাল কোয়ালিটি চেক প্রম্পট (Final QC Pass)

ফাইনাল MP4 রেন্ডার করার ঠিক আগে এই প্রম্পটটি দিন। এআই প্রতিটি ফ্রেম নিজে পরীক্ষা করে পাস/ফেল রিপোর্ট দেবে:

```markdown
Before the final render, perform a full 8-point quality check. Do not change the creative design. Check every frame and cut for:

1. Sync: Every effect lands within 2 frames of the start of its word. List any that miss.
2. Faces: No text, caption, card, or graphic overlaps my face at any point during its animation (shadows included).
3. Safe Zone: Nothing rendered in the top 220px, bottom 420px, or right 140px.
4. Cuts: No clipped words, no mid-word cuts, no audio clicks or pops at edit points.
5. Captions: Spelling of every name, brand, and Bengali conjunct is 100% correct.
6. Audio: Voice mastered at -14 LUFS, true peaks at -1 dBTP, music ducked 20 dB under voice.
7. Loop: Final frame matches frame 0 pixel for pixel with no flash or pop.
8. Artifacts: No black frames, frozen frames, or pixelation glitches.

Fix anything that fails, then give me a Pass/Fail table. Only render output/final.mp4 when every single point is PASS.
```

---

## ❓ সমস্যা ও সমাধান (Troubleshooting)

| সমস্যা | কারণ | সমাধান |
| :--- | :--- | :--- |
| **`ffmpeg not found`** | সিস্টেম PATH-এ যুক্ত নেই | উইন্ডোজে PATH ভ্যারিয়েবলে `C:\ffmpeg\bin` যোগ করুন অথবা টার্মিনাল রিস্টার্ট দিন। |
| **ক্যাপশনে বাংলা যুক্তাক্ষর ভাঙা** | ফন্ট ইউনিকোড সাপোর্ট করে না | `assets/fonts/` ফোল্ডারে `Noto Sans Bengali` বা `Kalpurush` ফন্ট রাখুন। |
| **কাট পয়েন্টে 'খটখট' (Click) শব্দ** | অডিও জিরো-ক্রসিং মিসিং | কাটিং ফিল্টারে `acrossfade=d=0.003` (3ms crossfade) ব্যবহার করুন। |
| **1080p ভিডিও জুমে ঝাপসা লাগে** | অতিরিক্ত ডিজিটাল জুম | 1080p ফুটেজে সর্বোচ্চ ১১৫% জুম রাখুন (`zoom_cap_1080p_pct: 115`)। |
| **Instagram/TikTok এ টেক্সট ঢাকা পড়ে** | সেফ জোন অমান্য করা | টপ ২২০px এবং বটম ৪২০px ফাঁকা রাখার কনফিগ যাচাই করুন। |

---

## 📜 লাইসেন্স
MIT License. যে কেউ বাণিজ্যিক বা ব্যক্তিগত যেকোনো ভিডিও প্রজেক্টে স্বাধীনভাবে ব্যবহার ও কাস্টমাইজ করতে পারবেন।
