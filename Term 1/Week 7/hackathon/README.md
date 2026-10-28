# Smart Litter Detection in Nature

**Course:** AI for Good - Hackathon 7: Eyes on Earth  
**Authors:** Allan Hassan & Alex Wrobel  
**Assigned SDG:** SDG 15 - Life on Land (Target 15.5: Halt biodiversity loss and protect natural habitats)  
**Required AI Tool:** Roboflow (Dataset management, annotation, model training, and Cloud/Local Inference API)  
**Working Prototype Repository / Model:** https://universe.roboflow.com/4llan4iags-workspace/trash-detector-8zprp

---

## 1. Problem Definition & Context
In terrestrial ecosystems across the Netherlands (such as the Haagse Bos and the dune reserves of Zuid-Holland), non-biodegradable litter-predominantly single-use plastics, beverage cans, and food wrappers-poses a severe ecological threat. According to *Rijkswaterstaat* and the *Plastic Soup Foundation*, millions of litter items enter Dutch natural habitats each year. Over time, plastic litter undergoes mechanical weathering and UV degradation, fragmenting into microplastics that penetrate forest soils, alter microbial balance, leach toxic plasticizers, and poison ground-dwelling fauna.

Currently, park clean-up operations are purely **reactive and labor-inefficient**:
- **Where & When:** Cleaning takes place post-weekend or after tourist peaks across vast natural parks.
- **The Core Bottleneck:** Litter in natural habitats is rarely piled up; it gets dispersed, trapped in rough terrain, and occluded by fallen leaves and vegetation. Maintenance crews spend up to **40% of their operational hours** aimlessly walking trails searching for waste rather than actively removing it.
- **Consequence:** Litter remains outdoors for weeks before detection, accelerating fragmentation into irreversible microplastic contamination.

---

## 2. Target User Group
* **Primary Intended Users:** Municipal green-space maintenance crews (e.g., *Gemeente Den Haag*) and forest rangers / conservation teams (*Staatsbosbeheer*).
  * **User Needs & Context:** These teams operate with tight budgets and limited personnel. They need an automated, high-precision visual triage tool that can be mounted on patrol vehicles, handheld devices, or low-altitude survey drones to flag litter coordinates before maintenance routes are planned.
* **Explicit Non-Users:** 
  * *Casual park visitors / citizens:* This is not a crowd-sourced public reporting app or gamified cleanup app.
  * *Autonomous physical collection robots:* The system does not mechanically sweep waste; it acts purely as an intelligent visual detection layer to guide human workers directly to target locations.

---

## 3. Solution Architecture & Roboflow Integration
The project delivers an automated computer vision pipeline that detects litter in complex natural ground environments from images or video frames.

```
[ Input Image / Video Frame ]
           │
           ▼
[ Preprocessing (640x640, Normalization) ]
           │
           ▼
[ Roboflow Inference API / Custom Model ] ──► Model: RF-DETR NAS (or YOLOv8)
           │                                 Trained on custom Dutch nature dataset
           ▼
[ Bounding Box & Confidence Output ]
           │
           ▼
[ Annotated Image Display + JSON Coordinates / Dispatch Payload ]
```

### Role of Roboflow (Required Tool)
Roboflow is the core backbone of this solution. The prototype **cannot function without it** because:
1. **Custom Natural-Terrain Dataset:** Off-the-shelf models (e.g., standard COCO) are trained on indoor/urban scenes and fail completely against foliage, leaves, and dirt. Roboflow allowed us to curate, annotate, and augment 300+ custom field images specifically captured in Dutch nature reserves.
2. **Inference Pipeline:** Roboflow hosts the trained `RF-DETR NAS` model checkpoint (`latency: 25.6 ms`) and delivers an API endpoint consumed by our Python deployment script for real-time bounding box prediction and spatial metadata extraction.

---

## 4. Problem–Solution Fit & Why Simpler Alternatives Fail
* **Direct Response to Problem:** Instead of manual walking surveys, field workers receive precise visual confirmations and spatial locations of detected waste, transforming reactive patrols into targeted cleanups.
* **Why Simpler Alternatives Fail:**
  * *Heuristic / Color-based thresholding:* Cannot distinguish a crushed white plastic cup from white stones, birch bark, or dried leaves under dynamic outdoor lighting.
  * *Fixed Patrol Schedules:* Waste is randomly distributed; scheduled sweeps waste fuel and labor on clean trails while neglected areas accumulate degraded plastic.
  * *Pretrained Generic Object Detectors:* Standard detectors flag clear objects on asphalt but miss weathered, bent, or partially occluded litter buried in forest mulch.

---

## 5. Working Prototype

### Model Specifications
* **Dataset:** 300+ custom images/video frames filmed in Dutch natural areas (split: 70% Train, 20% Valid, 10% Test).
* **Architecture:** `RF-DETR NAS` / YOLO-compatible single-class (`litter`) object detector.
* **Evaluation Metrics on Test Set:**
  * **mAP@50:** 89.3%
  * **Precision:** 97.0% (minimizes false alarms)
  * **Recall:** 84.2%
  * **F1 Score:** 90.1%
  * **Latency:** 25.6 ms per frame (enabling real-time edge/drone feeds)

## 6. Ethical Reflection & Data Card

### Data Collection & Provenance
* **Source:** All 300+ images were captured via high-definition phone video and photographs directly within public Dutch nature reserves (Haagse Bos and Scheveningen coastal dunes). Video streams were converted into individual frames sampled at variable intervals to prevent near-duplicate overfitting.
* **Ground Truth:** Bounding boxes were manually annotated under a unified single class (`litter`), encompassing cans, bottles, food wrappers, paper, and plastic bags.

### Limitations & Failure Modes
1. **Severe Occlusion (>75% leaf/soil cover):** If litter is buried under soil or deep under autumn foliage, the model fails (false negatives).
2. **Dense Stacks / Piles of Trash:** Because training samples were collected in Dutch reserves where single items dominate, overlapping piles of trash can merge into a single bounding box or confuse the edge detection.
3. **Lighting Extremes:** Low-light dusk conditions or harsh sunlight glare on wet surfaces reduce model recall.

### Real-World Consequences for Users
* **Risk of False Positives:** If natural elements (e.g., fungi, birch bark, unusual leaves) are misclassified as litter, cleanup crews are dispatched unnecessarily, wasting scarce municipal resources and increasing fuel emissions.
* **Risk of False Negatives:** Undetected plastic remains in the wild, fragmenting into microplastics and directly undermining SDG 15 goals.
* **Surveillance / Privacy Concerns:** Deploying vehicle- or drone-mounted cameras in public parks risks recording park visitors without consent.

### Mitigations (Implemented & Planned)
* **Implemented:**
  * We calibrated the detection confidence threshold at **0.65**. Combined with our **97.0% precision**, this virtually eliminates false positive dispatches.
  * Extensive dataset augmentations were applied (brightness adjustments between -25% and +25%, contrast adjustments) to handle changing weather conditions in Dutch forests.
* **Next Steps / Roadmap:**
  * Integrate an automatic upstream privacy blur filter (OpenCV Haar/YOLO face & license plate detector) to strip human features before frames are uploaded to the cloud.
  * Implement active learning: flag low-confidence predictions in the field for ranger re-verification and retrain specifically on clustered and partially buried litter.