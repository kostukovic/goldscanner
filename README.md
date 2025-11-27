# Goldscanner

**Goldscanner** is an open-source vision toolkit for detecting **placer gold and gemstones** in concentrates (black sand, river gravel) using **RGB / multispectral cameras** and cheap hardware (e.g. Raspberry Pi, Jetson, or a standard PC).

The long-term goal:  
> A portable “vision box” that can watch a thin layer of concentrate and automatically highlight – and later pick – **gold, platinum and gemstones** such as **diamond, ruby, sapphire, garnet, zircon, tourmaline**.

---

## ✨ Key idea

Instead of relying only on:

- **industrial ore sorters** (huge, expensive, for big mines only), or  
- **metal detectors** (no vision, no gemstone detection),

Goldscanner focuses on:

- **small-scale, placer gold setups** (sluice, black sand cleanup),
- **inline vision** of the moving material (water flow or thin dry layer),
- **local, offline processing** (no cloud, no proprietary API),
- **open-source models and pipelines** that others can reuse and improve.

---

## 🧱 Project status

> **Idea / early design stage.**  
> No public prototype exists yet.

The initial focus is on:

- designing the **software pipeline** (camera → pre-processing → segmentation → features → classification),
- collecting example data (photos/video of black sand, gold flakes, placer gemstones),
- defining a **simple reference box**:  
  a small, portable box (~1.5 × 0.5 × 0.5 m) with camera + lighting watching a thin material layer.

---

## 🔍 What Goldscanner wants to detect

Typical valuables in river placer material that are suitable for **vision-based recognition**:

| #  | Material                | Typical occurrence in river/placer | Main value / use                   | Visual cues (for camera)                                                  | Suitability for vision |
|----|-------------------------|-------------------------------------|------------------------------------|---------------------------------------------------------------------------|------------------------|
| 1  | Gold                    | Flakes, fine grains, small nuggets | Precious metal, investment, jewelry | Strong **yellow metallic** color, high specular shine, very dense, moves slowly in water | **Very high** – color + metal shine + motion |
| 2  | Platinum                | Small grey nuggets in black sand   | Precious metal, catalyst, jewelry  | **Silvery-grey**, dense, often dull metallic                              | High – needs good separation from other grey metals |
| 3  | Native silver           | Very rare, small grains/threads    | Jewelry, electronics               | Initially bright silver, quickly tarnishes dark                           | Medium – easily confused once tarnished |
| 4  | Diamond                 | Small rolled crystals in gravel    | Jewelry, industry                  | Colorless to light yellow/brown, strong sparkle, high refraction (“fire”) | High – with controlled lighting (sparkle patterns) |
| 5  | Ruby (red corundum)     | Small rounded red crystals         | High-value gemstone                | Intense red, often slightly cloudy, very hard                             | High – color + typical habits; can be confused with garnet |
| 6  | Sapphire (blue corundum)| Blue to colorless pebbles          | Gemstone                           | Blue (sometimes green/yellow), very hard, glassy to greasy luster         | High – color; multispectral helps vs other blue stones |
| 7  | Garnet                  | Common in black sands              | Medium-value gem, indicator mineral| Red-brown, often dodecahedral, strong glassy luster                       | **Very high** – very characteristic shape + color |
| 8  | Zircon                  | Small, heavy crystals              | Gemstone, geochronology           | Brown/red/colorless, high density, very bright luster                     | High – especially with color + motion (density) |
| 9  | Topaz                   | Clear to slightly colored fragments| Gemstone                           | Colorless, yellow, blue, glassy, transparent                              | Medium – similar to quartz; spectral/UV cues helpful |
| 10 | Tourmaline              | Dark elongated grains              | Gemstone (some varieties very valuable) | Black/dark green/multicolored, strong luster, often prismatic           | High – shape + luster; needs training vs other dark minerals |
| 11 | Aquamarine / Beryl      | Blue-green crystal fragments       | Gemstone                           | Light blue/sea-green, transparent, glassy                                | High but rare – good as “bonus class” |
| 12 | Black sands (magnetite, ilmenite) | Concentrate base | Mostly low value, but **indicator** for heavies | Black, matte to metallic, dominates heavy concentrate                    | **Very high** – forms the background “mask” for detection |

**Important:**  
Most of these originate from hard rock deposits, weather out, and then get concentrated in placer settings.  
Goldscanner cares about the **final free grains/pebbles**, not the original bedrock.

---

## 📡 Why multispectral (and not hyperspectral) – focusing on a few key wavelengths?

**Short answer:**  
Multispectral is **much cheaper** and fast enough (20–120 fps) for real-time vision in a sluice box or cleanup table.

### Price comparison (very rough)

| System type                                   | Typical price range      |
|----------------------------------------------|--------------------------|
| DIY multispectral (mono camera + LEDs + filters) | **150–500 €**         |
| Industrial multispectral camera (3–6 bands, RGB+NIR) | **800–3,000 €**   |
| Pro multispectral (e.g. 8–12 bands snapshot) | **3,000–10,000 €**       |
| Hyperspectral line/snapshot cameras           | **10,000–100,000 €**+    |

Multispectral = a **small set of spectral bands** (e.g. 3–8 carefully chosen LEDs / filters)  
instead of 100+ narrow bands of a hyperspectral sensor.

For many tasks (gold vs sand, diamond UV-glow, shiny metal vs rock) you do **not** need a full hyperspectral cube.

---

## 💡 Typical spectral strategies

### Gold / metal separation

Gold often shows:

- strong specular reflections,
- characteristic red-yellow response,
- specific NIR behavior.

Example minimal set:

- Blue (~450 nm)  
- Red (~630 nm)  
- NIR 850 nm  
- optional UV for special effects

With a **mono camera + time-multiplexed LEDs (TDM)** you can measure reflectance at each wavelength.

### Diamond

Diamonds can **fluoresce under UV** (365/395 nm).

Useful bands:

- UV for excitation  
- Blue/green to observe fluorescence and transparency  
- IR for absorption patterns

Result:  
Good separation between **diamond vs glass vs quartz vs Moissanite** with a few LEDs.

### Rhodium / precious metal coatings

Rhodium and other metals often have distinct **NIR / visible reflectance signatures**.  
With **red + NIR** you can separate:

- silver  
- steel  
- aluminum  
- rhodium-coated surfaces

### Ore / rock types

Many minerals differ in:

- UV absorption,
- blue/green response,
- NIR reflectance.

A **5–8 LED multispectral set** (UV, blue, green, red, NIR) is industrial standard for basic sorting.

---

## 🧩 Minimal DIY multispectral setup

A powerful low-cost setup can look like this:

- **Mono global-shutter camera** (RAW10/RAW12)  
  - e.g. Arducam / Basler / Dahua MIPI/USB  
  - price: ~60–200 €

- **LED ring / light bars** with distinct wavelengths  
  - UV 365 nm  
  - Blue 450 nm  
  - Green 525 nm  
  - Red 630 nm  
  - NIR 850 / 940 nm  
  - price: ~5–30 € per LED cluster

- **LED controller for TDM**  
  - ESP32 or small MCU, synced to camera  
  - price: ~5–10 €

- **Optional filters**  
  - IR bandpass filters  
  - UV-pass / UV-cut filters  
  - price: ~10–40 €

**Operation:**

1. Turn on LED #1, grab a frame.  
2. Turn on LED #2, grab a frame.  
3. … repeat for all wavelengths in a cycle (e.g. 20–60 Hz).  
4. Software stacks frames → **multispectral tensor** per pixel/particle.  
5. ML/NN classifier learns spectral + geometric patterns.

This gives **real-time (20–120 fps)** material recognition for a tiny fraction of hyperspectral costs.

---

## 🧮 Multispectral vs. hyperspectral – when to use what?

| Application                                 | Recommendation          |
|---------------------------------------------|-------------------------|
| Gold vs brass / bronze / steel              | Multispectral           |
| Diamond identification / UV check           | Multispectral (UV key)  |
| Rhodium / precious metal coating detection  | Multispectral           |
| Basic rock / ore sorting                    | Multispectral or Hyper  |
| Precise lab-grade mineralogy                | Hyperspectral           |
| Inline control at 20–100 fps                | Multispectral           |
| Budget < 10,000 €                           | Multispectral           |
| DIY / hacker-friendly system                | Multispectral           |

For **Goldscanner** use-cases (placer gold + gemstones in a box):

> ✅ Multispectral is the **sweet spot**.  
> Hyperspectral only makes sense for very advanced mineralogy / lab work.

---

## 🧠 Software architecture (planned)

Goldscanner is planned as a **modular Python toolkit**:

1. **Camera module**  
   - Capture via OpenCV / GStreamer  
   - Supports RGB and mono cameras  
   - Optional LED sync/TDM API

2. **Pre-processing**  
   - Denoising, color space transforms, contrast  
   - Background subtraction / stabilization  
   - Flattening moving water artifacts (if used inline in a sluice)

3. **Segmentation**  
   - Separate individual particles/grains from background  
   - Connected components / superpixels / simple CNN masks

4. **Feature extraction**  
   - Geometry: area, roundness, aspect ratio  
   - Photometry: brightness, specular highlights, color channels  
   - Multispectral: per-band response, simple indices

5. **Classification**  
   - Traditional ML (Random Forest / Gradient Boosting) as a start  
   - Later: compact CNN / transformer models (ONNX Runtime)  
   - Output classes: `gold`, `platinum`, `metal`, `gem_candidate`, `rest`

6. **Visualization & API**  
   - Web UI (FastAPI/Gradio/Streamlit)  
   - Live overlay: bounding boxes, masks, class labels  
   - Data export: CSV, JSON, segmentation masks  
   - Optional: coordinates for an external XY-picking head (via WebSocket/serial)

---

## 🧰 Hardware targets

- **Compute**  
  - Raspberry Pi 5  
  - NVIDIA Jetson (Nano/Orin, etc.)  
  - Standard Linux PC (laptop/desktop)

- **UI / control**  
  - Simple **web frontend**  
  - Useable from **tablet / iPad / phone** via browser

- **Optional mechanics**  
  - Small **XY gantry** (similar to a 3D printer)  
  - Vacuum / suction head to pick particles or nuggets based on vision coordinates

(Important: Goldscanner itself remains a **software project**.  
The mechanical “Gold Box” is conceived as a reference implementation, not mandatory.)

---

## 🆚 Positioning vs. existing solutions

### Large industrial sorters (e.g. TOMRA)

- Huge, expensive, fixed installations  
- Process **tons per hour** of **dry, crushed ore** on conveyor belts  
- Target customers: **major mining companies**, not individuals

**Goldscanner’s difference:**

- Portable, small **box-scale**, not factory-scale  
- Works on **wet or dry concentrates** (sluice, cleanup table)  
- Runs on cheap hardware, designs are **open** (FOSS)  
- Target users: **small-scale placer miners, prospectors, researchers, makers**

### Metal detectors & ground scanners

- Detect **metal** in soil by conductivity/induction  
- No image, no shape analysis, no gemstone detection  
- Point-wise scanning, not full-field vision

**Goldscanner’s difference:**

- **Vision + ML**, not just “beep”:  
  uses color, shape, shine, motion  
- Can distinguish **gold vs pyrite**, garnet vs “red junk”, diamond vs quartz  
- Scans the **entire field of view** in real time, not spot-by-spot  
- Can also find **non-metallic gems** (diamonds, rubies, sapphires, etc.)

---

## 🚧 Roadmap (early draft)

- **Phase 0 – Data & design**
  - Collect sample images/videos of black sand + known gold/gem grains  
  - Define reference box geometry and lighting setups  
  - Draft core API & pipeline layout

- **Phase 1 – Basic RGB prototype**
  - Implement camera capture + simple pre-processing  
  - Segment bright metallic particles on dark sand  
  - Train initial classifier for `gold` vs `rest` on RGB only  
  - Simple web UI with overlays

- **Phase 2 – Multispectral prototype**
  - Integrate TDM LED control  
  - Add multi-band feature extraction  
  - Train improved classifiers, including basic gem classes (`garnet`, `gem_candidate`)

- **Phase 3 – Release & community**
  - Publish first open dataset (anonymized or synthetic)  
  - Improve documentation and examples  
  - Attract testers from small-scale mining / maker / research communities

---

## 🤝 Contributing

Goldscanner is intended as an **open, community-driven** project.

Planned contribution areas:

- Data collection & labeling (gold vs sand, gem vs rock)  
- New detection models (better classifiers, gem types)  
- Hardware reference designs (LED rings, boxes, XY gantries)  
- Documentation, tutorials, demos

---

## 👤 Author

**Dmitrij Kostukovic**  
- Background in **mechanical engineering & CNC**  
- Transitioning into **Computer Vision / Machine Vision Software Engineering**  
- Email: `kostukovic@googlemail.com`

---

> **Status:** Concept / early design.  
> If you are interested in collaborating (data, hardware, field tests), feel free to open an issue or reach out by email.
