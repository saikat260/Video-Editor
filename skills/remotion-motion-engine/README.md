# Remotion 3D Motion Engine (হাইপার-লেভেল 3D মোশন গ্রাফিক্স)

> **React, Three.js, এবং WebGL চালিত নেক্সট-জেনারেশন ৩D মোশন গ্রাফিক্স, স্প্রিং ফিজিক্স ও সিনেমাটিক ম্যাপ এনিমেশন ইঞ্জিন।**

---

## ১. স্কিলের পরিচিতি

`remotion-motion-engine` হলো ভিডিও এডিটিং পাইপলাইনের সবচেয়ে শক্তিশালী ভিজ্যুয়াল কম্পোনেন্ট। সাধারণ 2D ক্যানভাস বা স্ট্যাটিক ইমেজ রেন্ডারিংয়ের সীমাবদ্ধতা দূর করে এটি সম্পূর্ণ ফ্রেম-অ্যাকিউরেট **হাইভোল্টেজ 3D ভিজ্যুয়াল এফেক্ট** তৈরি করে।

### প্রধান ফিচারসমূহ:
- **Three.js & React Three Fiber (`@remotion/three`):** 3D পার্টিকল, মেটাবিক গ্লোব, শ্যাডো, ডিরেকশনাল লাইট ও কাস্টম ক্যামেরা প্যান।
- **Isometric 3D Glassmorphism Cards:** বাস্তবসম্মত স্প্রিং ফিজিক্স, গ্লাস ব্লার, পার্সপেক্টিভ টিল্ট (`rotateX`, `rotateY`, `translateZ`)।
- **Johnny Harris 3D Maps:** MapLibre GL 3D ও `@turf/turf` দিয়ে গ্রেট-সার্কেল ফ্লাইট রুট এবং ৩D সিনেমাটিক টেরেইন ট্র্যাকিং।
- **Photorealistic CesiumJS Flyovers:** গুগল ৩D ফটোরিয়ালিস্টিক টাইলস দিয়ে রিয়েল ৩D শহরের ওপর দিয়ে ড্রোন-স্টাইল ফ্লাইট।
- **Remotion Studio (`npx remotion studio`):** কোড পরিবর্তন করার সাথে সাথে ব্রাউজারে লাইভ টাইমলাইন স্ক্রাবিং প্রিভিউ।

---

## ২. ইনস্টলেশন ও এনভায়রনমেন্ট সেটআপ

প্রজেক্টে রিমোশন থ্রি-ডি সাপোর্ট চালু করতে টার্মিনালে নিচের কমান্ডগুলো রান করুন:

```bash
# ১. রিমোশন ও থ্রি-ডি লাইব্রেরি
npm install remotion @remotion/cli @remotion/three three @react-three/fiber

# ২. জিওস্প্যাশিয়াল ম্যাপ ইঞ্জিন
npm install maplibre-gl @turf/turf

# ৩. ভেরিফাই ইনস্টলেশন
npx remotion --version
```

---

## ৩. মাস্টার ৩D মোশন গ্রাফিক্স প্রম্পটস (Copy-Paste Ready)

আপনার ভিডিওর স্ক্রিপ্ট বা সিন অনুযায়ী নিচের যেকোনো প্রম্পট ব্যবহার করে সরাসরি হাইপার-লেভেল 3D ভিজ্যুয়াল তৈরি করতে পারবেন:

### প্রম্পট ১: Hyper 3D Floating Glassmorphism Card (রবার্তো নিক্সন স্টাইল)
```text
Create a hyper-level 3D floating interface card using Remotion and CSS 3D perspective.
- Style: Dark futuristic glassmorphism with 30px backdrop blur, subtle 1px border (#ffffff15), and Claude orange (#D97757) neon drop-shadow.
- Physics: Spring entrance using Remotion spring() with mass: 0.8, damping: 14, stiffness: 120.
- 3D Motion: Subtle continuous 3D hover rotation (rotateX: -6deg to +6deg, rotateY: -8deg to +8deg, translateZ: 20px).
- Content: Layered Z-depth elements. Floating tag with neon pill at translateZ(40px), bold 56px title at translateZ(30px), and subtitle at translateZ(20px).
- Output: Remotion Composition rendered at 1080x1920 (9:16 vertical) with transparent alpha background.
```

---

### প্রম্পট ২: Johnny Harris 3D Geodesic Flight Route Map (ভক্স/জনি হ্যারিস স্টাইল)
```text
Build a Johnny Harris style 3D animated flight route map using Remotion, MapLibre GL, and @turf/turf.
- Coordinates: Animate flight from London [-0.1276, 51.5072] to New York [-74.006, 40.7128].
- 3D Camera: Map pitched at 50 degrees with a continuous dynamic camera following the animated airplane marker.
- Geodesic Arc: Calculate the true curved great-circle path using turf.greatCircle() and slice it frame-by-frame using turf.lineSliceAlong().
- Styling: Minimal dark monochromatic vector map, dashed orange line (#D97757, line-width 5px), glowing white pulse dot at route head.
- Frame Duration: 5 seconds (150 frames @ 30fps), easing out smoothly as it reaches destination.
- Audio Sync: Add whoosh/sub-bass marker sync on departure and arrival.
```

---

### প্রম্পট ৩: Three.js 3D Audio-Reactive Particle Sphere (ফিউচারিস্টিক টেক)
```text
Implement a futuristic 3D particle sphere using Remotion, Three.js, and @remotion/three.
- Canvas: Wrap in <ThreeCanvas> with explicit width and height.
- Mesh: 3D point cloud sphere with 4,000 particles, AdditiveBlending, glowing Claude orange (#D97757) color.
- Animation: Frame-driven via useCurrentFrame() (strictly no @react-three/fiber useFrame). Camera pushes in from Z: 7 to Z: 4.8 with exponential ease-out.
- Lighting: Multi-point directional lighting creating deep 3D specular highlights across particle vertices.
- Alpha Channel: Render with transparent background for direct video overlay.
```

---

### প্রম্পট ৪: 3D Kinetic Split Typography (বর্ডারলেস মাস্ক রিভিল)
```text
Generate a hyper-impact 3D kinetic typography sequence using Remotion spring physics.
- Concept: Bold 84px uppercase text ("THE UNSEEN CODE") split into two horizontal slices via clip-path.
- 3D Entrance: Top half descends from -120px while rotating 25 degrees on X-axis; bottom half rises from +120px rotating -25 degrees on X-axis.
- Timing: Spring physics with mass: 0.5, damping: 10, snapping together with zero bounce at frame 24.
- Accent: The active keyword glows in electric orange (#D97757) with 3D chromatic aberration split on impact.
```

---

### প্রম্পট ৫: Photorealistic 3D City Flythrough (CesiumJS / Google 3D Tiles)
```text
Generate a cinematic 3D drone flythrough over Manhattan using Remotion CesiumFlythrough component.
- Mode: Google Photorealistic 3D Tiles (city mode).
- Route: Chaikin-smoothed flight path gliding between skyscrapers, starting at altitude 650m and descending to 420m.
- Camera: Banking roll aligned with curve tangent so the camera naturally tilts into turns.
- Render Mode: Deterministic WebGL capture with delayRender and --gl=angle.
```

---

## ৪. লাইভ স্টুডিও ওয়ার্কফ্লো (Interactive Timeline Scrubbing)

ভিডিও রেন্ডার হওয়ার আগেই ব্রাউজারে রিয়েল-টাইম স্ক্রাব করতে:

```bash
# ব্রাউজারে স্টুডিও চালু করুন
npx remotion studio

# নির্দিষ্ট কম্পোজিশন দেখতে
# ব্রাউজারে খুলুন: http://localhost:3000/Hyper3DCard
```

### ফাইনাল হেডলেস রেন্ডার কমান্ড:
```bash
npx remotion render src/index.ts MainComposition output/scene_3d.mp4 --gl=angle --concurrency=4
```

---

## ৫. রুলস ও কোয়ালিটি স্ট্যান্ডার্ডস

1. **কোনো Pillow বা 2D লিনিয়ার বক্স নয়:** সর্বদা রিমোশনের ৩D স্পেস, Three.js বা ভেক্টর ম্যাপ ব্যবহার করতে হবে।
2. **ফ্রেম ডিটারমিনিজম:** ব্রাউজার রিকোয়েস্টঅ্যানিমেশনফ্রেম বা সিএসএস ট্রানজিশনের ওপর নির্ভর করা যাবে না।
3. **সেফ-জোন রুল:** 9:16 রেশিওতে উপরের ২২০px এবং নিচের ৪২০px ফাঁকা রেখে ৩D অবজেক্টের মূল ফোকাস স্থাপন করতে হবে।
