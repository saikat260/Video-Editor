---
name: remotion-motion-engine
description: |
  Remotion (React) ভিত্তিক হাইপার-লেভেল 3D মোশন গ্রাফিক্স, থ্রি-ডি স্প্রিং ফিজিক্স,
  Three.js সিনারিও, এবং Johnny Harris স্টাইল 3D জিওস্প্যাশিয়াল ম্যাপ এনিমেশন ইঞ্জিন।
  WebGL, @remotion/three, MapLibre GL 3D এবং CesiumJS ব্যবহার করে সিনেমাটিক
  ভিজ্যুয়াল আর্ট তৈরি করে। লাইভ রিমোশন স্টুডিও এবং হেডলেস অ্যাঙ্গেল রেন্ডারিং সাপোর্ট করে।
version: "2.0"
author: AI Systems Lab
triggers:
  - "3D motion graphics"
  - "hyper 3d graphics"
  - "remotion video"
  - "3D map animation"
  - "johnny harris map"
  - "threejs animation"
  - "spring physics typography"
  - "রিমোশন থ্রিডি মোশন গ্রাফিক্স"
  - "থ্রিডি ম্যাপ বানাও"
requires:
  - Node.js >= 20
  - remotion >= 4.0
  - "@remotion/three"
  - three
  - "@react-three/fiber"
  - maplibre-gl
  - "@turf/turf"
tags:
  - 3d-graphics
  - remotion
  - threejs
  - webgl
  - maps
  - spring-physics
  - motion-graphics
---

# Remotion 3D Motion Graphics & Geospatial Engine

## ১. ওভারভিউ (Overview)
`remotion-motion-engine` হলো আমাদের ভিডিও প্রোডাকশন স্যুটের প্রিমিয়াম **হাইভোল্টেজ 3D মোশন গ্রাফিক্স ইঞ্জিন**। সাধারণ 2D ফ্ল্যাট কার্ড কিংবা স্ট্যাটিক ইমেজ ড্রয়িংয়ের সীমাবদ্ধতা চিরতরে ভেঙে দিয়ে, এটি React ও WebGL-এর পূর্ণ শক্তিতে হাইপার-লেভেল 3D সিনারিও, স্প্রিং ফিজিক্স এবং জিওস্প্যাশিয়াল ফ্লাইট ম্যাপ তৈরি করে।

### এই ইঞ্জিনের মূল স্তম্ভসমূহ:
1. **Three.js & React Three Fiber (`@remotion/three`):** সিনেমাটিক লাইটিং, শ্যাডো, 3D অবজেক্ট, কাস্টম পার্টিকল ফিল্ড এবং ডায়নামিক ক্যামেরা মুভমেন্ট।
2. **CSS 3D Perspective & Isometric Rig:** ফিজিক্যাল স্প্রিং রিঅ্যাকশন সহ 3D স্পেসে ভাসমান গ্লাস মরফিজম ইন্টারফেস কার্ড (`rotateX`, `rotateY`, `translateZ`, `perspective`)।
3. **Johnny Harris Geospatial 3D Maps:** MapLibre GL 3D ভেক্টর ম্যাপ এবং CesiumJS ফটোরিয়ালিস্টিক ৩D টেরেইন/সিটি ফ্লাইওভার।
4. **Spring Physics Typography:** লিনিয়ার এনিমেশনের বদলে ম্যাস, ড্যাম্পিং ও স্টিফনেস নিয়ন্ত্রিত কাইনেটিক স্প্রিং টেক্সট।
5. **Remotion Studio (`npx remotion studio`):** ব্রাউজারে মিলিসেকেন্ড নিখুঁত লাইভ টাইমলাইন স্ক্রাবিং।

---

## ২. প্রি-রিক্যুইজিট ও ইনস্টলেশন (Prerequisites)

যেকোনো রিমোশন 3D সিন রান করার জন্য প্রজেক্টে নিচের প্যাকেজগুলো নিশ্চিত করতে হবে:

```bash
# Remotion কোর ও 3D এক্সটেনশন ইনস্টলেশন
npm install remotion @remotion/cli @remotion/three three @react-three/fiber

# 3D ম্যাপ ও জিওস্প্যাশিয়াল লাইব্রেরি
npm install maplibre-gl @turf/turf

# যদি CesiumJS ৩D সিটি/ল্যান্ডস্কেপ ফ্লাইওভার প্রয়োজন হয়
npm install resium cesium
```

---

## ৩. কোর আর্কিটেকচার রুলস (Golden Execution Rules)

Remotion-এ 3D মোশন গ্রাফিক্স তৈরির সময় নিচের নিয়মগুলো অলঙ্ঘনীয়:

1. **Deterministic Frame-Driven Execution:**
   - ব্রাউজারের কোনো `CSS transition`, `CSS keyframe animation` বা `setTimeout` ব্যবহার করা নিষিদ্ধ।
   - সব অ্যানিমেশন অবশ্যই `useCurrentFrame()` এবং `interpolate()` অথবা `spring()` দিয়ে চালিত হতে হবে।
   - Three.js-এর ভেতর `@react-three/fiber`-এর `useFrame()` ব্যবহার করা যাবে না; রিমোশনের `useCurrentFrame()` ব্যবহার করতে হবে, নতুবা হেডলেস রেন্ডারে ফ্রেম ড্রপ/ফ্লিকার করবে।

2. **Spring Physics Contract:**
   - লিনিয়ার গতি বর্জনীয়। টেক্সট বা কার্ড স্ক্রিনে এন্ট্রি নেয়ার সময় `spring()` ব্যবহার করতে হবে:
     ```tsx
     const progress = spring({
       frame,
       fps,
       config: { mass: 0.6, damping: 12, stiffness: 100 }
     });
     ```

3. **ThreeCanvas Setup:**
   - প্রতিটি 3D সিনে `<ThreeCanvas>` ব্যবহার করতে হবে যার সুস্পষ্ট `width` ও `height` প্রপস থাকবে।
   - থ্রি-ডি সিনে আবশ্যিকভাবে ডিরেকশনাল লাইট ও অ্যাম্বিয়েন্ট লাইট থাকতে হবে যাতে মেটেরিয়ালের 3D ডেপথ ফুটে ওঠে।

4. **Safe Zone Awareness:**
   - 9:16 ভার্টিক্যাল ভিডিওর জন্য টপ ২২০px, বটম ৪২০px এবং রাইট ১৪০px সেফ-জোন শতভাগ মেনে ৩D অবজেক্ট ফোকাস করতে হবে।

---

## ৪. হাইপার 3D টেকনিক্স (Hyper 3D Techniques)

### টেকনিক ১: ফ্লোটিং 3D ইন্টারফেস কার্ড (Isometric Tilt & Glassmorphism)
রবার্তো নিক্সন ও কালাওয়ে স্টাইলের ভাসমান প্রিমিয়াম ৩D টেকনোলজি কার্ড।

```tsx
import React from 'react';
import { interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';

export const Hyper3DCard: React.FC<{ title: string; subtitle: string; tag: string }> = ({
  title,
  subtitle,
  tag,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Spring entrance
  const entrance = spring({
    frame,
    fps,
    config: { mass: 0.8, damping: 14, stiffness: 120 },
  });

  // Continuous subtle 3D hover tilt
  const tiltX = interpolate(Math.sin(frame / 20), [-1, 1], [-6, 6]);
  const tiltY = interpolate(Math.cos(frame / 25), [-1, 1], [-8, 8]);
  const floatZ = interpolate(Math.sin(frame / 15), [-1, 1], [-15, 15]);

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        perspective: '1200px',
      }}
    >
      <div
        style={{
          width: '780px',
          padding: '48px',
          background: 'linear-gradient(135deg, rgba(255, 255, 255, 0.08), rgba(255, 255, 255, 0.02))',
          backdropFilter: 'blur(30px)',
          borderRadius: '28px',
          border: '1px solid rgba(255, 255, 255, 0.15)',
          boxShadow: '0 30px 80px rgba(0, 0, 0, 0.6), 0 0 40px rgba(217, 119, 87, 0.2)',
          transformStyle: 'preserve-3d',
          transform: `
            scale(${entrance})
            rotateX(${tiltX + (1 - entrance) * 25}deg)
            rotateY(${tiltY}deg)
            translateZ(${floatZ}px)
          `,
        }}
      >
        <div style={{ transform: 'translateZ(40px)', transformStyle: 'preserve-3d' }}>
          <span
            style={{
              padding: '6px 16px',
              borderRadius: '999px',
              background: '#D97757',
              color: '#FFFFFF',
              fontSize: '22px',
              fontWeight: 800,
              textTransform: 'uppercase',
              letterSpacing: '2px',
              display: 'inline-block',
              boxShadow: '0 8px 20px rgba(217, 119, 87, 0.4)',
            }}
          >
            {tag}
          </span>
          <h2
            style={{
              fontSize: '56px',
              fontWeight: 900,
              color: '#FFFFFF',
              margin: '24px 0 12px 0',
              lineHeight: 1.1,
              transform: 'translateZ(30px)',
            }}
          >
            {title}
          </h2>
          <p
            style={{
              fontSize: '28px',
              color: '#A0A0A0',
              margin: 0,
              transform: 'translateZ(20px)',
            }}
          >
            {subtitle}
          </p>
        </div>
      </div>
    </div>
  );
};
```

---

### টেকনিক ২: Three.js 3D ক্যামেরা ও পার্টিকল মেশ (`@remotion/three`)
আসল থ্রি-ডি মেটাবিক লাইটিং এবং অডিও-সিনথেটিক 3D অবজেক্ট।

```tsx
import React, { useRef } from 'react';
import { ThreeCanvas } from '@remotion/three';
import { useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import * as THREE from 'three';

const ParticleGlobe: React.FC = () => {
  const frame = useCurrentFrame();
  const meshRef = useRef<THREE.Points>(null);

  const rotY = frame * 0.015;
  const rotX = Math.sin(frame * 0.02) * 0.2;

  return (
    <points ref={meshRef} rotation={[rotX, rotY, 0]}>
      <sphereGeometry args={[2.5, 64, 64]} />
      <pointsMaterial
        size={0.035}
        color="#D97757"
        transparent
        opacity={0.85}
        blending={THREE.AdditiveBlending}
      />
    </points>
  );
};

export const Scene3DThree: React.FC = () => {
  const { width, height } = useVideoConfig();
  const frame = useCurrentFrame();

  const camZ = interpolate(frame, [0, 90], [7, 4.8], {
    extrapolateRight: 'clamp',
  });

  return (
    <ThreeCanvas
      width={width}
      height={height}
      camera={{ position: [0, 0, camZ], fov: 45 }}
    >
      <ambientLight intensity={0.4} />
      <pointLight position={[10, 10, 10]} intensity={1.5} color="#FFFFFF" />
      <directionalLight position={[-5, 5, 2]} intensity={0.8} color="#D97757" />
      <ParticleGlobe />
    </ThreeCanvas>
  );
};
```

---

### টেকনিক ৩: Johnny Harris 3D মানচিত্র ও ফ্লাইট রুট (MapLibre GL 3D & Turf)
ভৌগোলিক তথ্যচিত্রের জন্য কার্ভড গ্রেট-সার্কেল ফ্লাইট রুট এবং ৩D পিচ/ক্যামেরা ট্র্যাকিং।

```tsx
import React, { useEffect, useRef, useState } from 'react';
import { useCurrentFrame, useDelayRender, useVideoConfig, interpolate } from 'remotion';
import * as maplibregl from 'maplibre-gl';
import * as turf from '@turf/turf';
import 'maplibre-gl/dist/maplibre-gl.css';

export const JohnnyHarris3DMap: React.FC<{
  origin: [number, number];       // [lon, lat] e.g. London [-0.1276, 51.5072]
  destination: [number, number];  // [lon, lat] e.g. New York [-74.006, 40.7128]
}> = ({ origin, destination }) => {
  const containerRef = useRef<HTMLDivElement>(null);
  const mapRef = useRef<maplibregl.Map | null>(null);
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  const { delayRender, continueRender } = useDelayRender();
  const [handle] = useState(() => delayRender('MapInit'));

  // Pre-calculate geodesic curved route
  const greatCircle = turf.greatCircle(origin, destination, { npoints: 150 });
  const totalDistance = turf.length(greatCircle);

  // Animation progress
  const progress = interpolate(frame, [15, durationInFrames - 20], [0, 1], {
    extrapolateLeft: 'clamp',
    extrapolateRight: 'clamp',
  });

  const currentDistance = progress * totalDistance;
  const currentLine = turf.lineSliceAlong(greatCircle, 0, Math.max(0.01, currentDistance));
  const currentPoint = turf.along(greatCircle, Math.max(0.01, currentDistance));

  useEffect(() => {
    if (!containerRef.current) return;

    const map = new maplibregl.Map({
      container: containerRef.current,
      style: 'https://demotiles.maplibre.org/style.json', // অথবা কাস্টম ডার্ক ভেক্টর স্টাইল
      center: origin,
      zoom: 3.5,
      pitch: 50, // 3D Cinematic Tilt
      bearing: -20,
      interactive: false,
      attributionControl: false,
      canvasContextAttributes: { preserveDrawingBuffer: true },
    });

    map.on('load', () => {
      // Add animated route layer
      map.addSource('route', {
        type: 'geojson',
        data: currentLine,
      });

      map.addLayer({
        id: 'route-line',
        type: 'line',
        source: 'route',
        paint: {
          'line-color': '#D97757',
          'line-width': 5,
          'line-dasharray': [2, 1],
        },
      });

      // Add pulsating head marker
      map.addSource('head-marker', {
        type: 'geojson',
        data: currentPoint,
      });

      map.addLayer({
        id: 'head-point',
        type: 'circle',
        source: 'head-marker',
        paint: {
          'circle-radius': 8,
          'circle-color': '#FFFFFF',
          'circle-stroke-width': 3,
          'circle-stroke-color': '#D97757',
        },
      });

      mapRef.current = map;
      continueRender(handle);
    });

    return () => {
      map.remove();
    };
  }, []);

  // Frame-by-frame camera and route update
  useEffect(() => {
    if (!mapRef.current || !mapRef.current.isStyleLoaded()) return;

    const routeSource = mapRef.current.getSource('route') as maplibregl.GeoJSONSource;
    const pointSource = mapRef.current.getSource('head-marker') as maplibregl.GeoJSONSource;

    if (routeSource) routeSource.setData(currentLine);
    if (pointSource) pointSource.setData(currentPoint);

    // Dynamic 3D Camera tracking the airplane/route
    const coords = currentPoint.geometry.coordinates as [number, number];
    mapRef.current.jumpTo({
      center: coords,
      pitch: 50 + progress * 10,
      zoom: 3.5 + Math.sin(progress * Math.PI) * 0.8,
    });
  }, [frame]);

  return (
    <div style={{ width: '100%', height: '100%', position: 'relative' }}>
      <div ref={containerRef} style={{ width: '100%', height: '100%' }} />
      {/* Cinematic Vignette Overlay */}
      <div
        style={{
          position: 'absolute',
          inset: 0,
          boxShadow: 'inset 0 0 120px rgba(0,0,0,0.8)',
          pointerEvents: 'none',
        }}
      />
    </div>
  );
};
```

---

## ৫. লাইভ স্টুডিও ও হেডলেস রেন্ডারিং কমান্ডস (Studio & Rendering)

### লাইভ স্টুডিও প্রিভিউ (Interactive Visual Scrubbing):
রেন্ডার দেওয়ার পূর্বে ব্রাউজারে রিয়েল-টাইম টাইমলাইন স্ক্রাব করতে:
```bash
# ব্রাউজারে সরাসরি স্টুডিও ওপেন
npx remotion studio

# হেডলেস বা ইন-অ্যাপ প্রিভিউয়ের জন্য
npx remotion studio --no-open
```

### হেডলেস WebGL হাইপার রেন্ডার (Angle Hardware Accelerated):
```bash
# ৩D WebGL এবং ম্যাপ রেন্ডারিংয়ের জন্য --gl=angle ব্যবহার বাধ্যতামূলক
npx remotion render src/index.ts MainComposition output/scene_3d.mp4 --gl=angle --concurrency=4
```

---

## ৬. কোয়ালিটি কন্ট্রোল ও অ্যান্টি-টেমপ্লেট রুলস (Strict QC Checklist)

1. **নো ফ্ল্যাট কার্ড:** যেকোনো কার্ডে ৩D লাইটিং, গ্লাস ড্রপ-শ্যাডো ও পার্সপেক্টিভ থাকতে হবে।
2. **নো স্ট্যাটিক লাইন:** ম্যাপের রুট অবশ্যই ফ্রেম ধরে অ্যানিমেট হয়ে ড্র হবে।
3. **ক্যামেরা মোশন:** প্রতিটি ৩D সিনে ক্যামেরা অন্তত ১০-১৫% ট্রাভেল বা অরবিট করবে।
4. **অডিও-সিঙ্ক:** ৩D কার্ডের ফ্লিপ বা জুমের সাথে ২ ফ্রেমের মধ্যে সাউন্ড ইফেক্ট (`whoosh`, `sub-bass hit`) সিঙ্ক হবে।
