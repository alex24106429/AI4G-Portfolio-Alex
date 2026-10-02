# Climate Action

**Pair Partners:** Alex Wróbel & Haki Abdulovski  
**Core Tool:** ComfyUI
**UN SDG:** SDG 13 - Climate Action  
**Video Output:** 35-Second Short Film
**Video Link:** [YouTube (Watch Here)](https://www.youtube.com/watch?v=MnGeSE4xGQE)

---

## 1. Problem Definition & Target Audience

### Problem Statement
The physical infrastructure supporting frontier artificial intelligence models has decoupled from sustainable grid planning. OpenAI’s "Stargate" AI supercomputing initiative relies on on-site natural gas combustion turbines and backup diesel generation permitted under industrial loopholes (Floodlight, 2026). In Texas alone, these facilities are permitted to emit over 1.6 million tons of greenhouse gases annually. 

According to the International Energy Agency (IEA), global data center electricity consumption will nearly double by 2030, with approximately 30% of global data center power still supplied by coal. Despite tech industry sustainability marketing, "behind-the-meter" fossil generation is expanding rapidly to bypass utility interconnection delays.

### Intended Audience
* **Primary Demographic:** Dutch university students and young adults (ages 18–25) who are frequent daily users of ChatGPT, Claude, and generative coding tools.
* **Psychographic Profile:** Tech-literate, climate-conscious individuals who experience a cognitive disconnect between a digital browser interface and the physical, fossil-fueled reality of AI data centers.
* **Who It Is Not For:** General climate deniers or purely non-technical demographics who lack awareness of current AI software.

---

## 2. Technical Architecture & ComfyUI Pipeline

The entire short film was designed, generated, upscaled, subbed, and assembled **100% inside ComfyUI** across modular pipelines. No external third-party editing suites (such as Premiere Pro, DaVinci Resolve, or After Effects) or external AI voice APIs were used.

### Pipeline Breakdown

#### A. Base Image Generation & Image Editing
* **Krea 2 Turbo (`image_krea2_turbo_t2i_int8.json`)**:
  * **UNet**: `krea2_turbo_int8_convrot.safetensors` (INT8 quantized diffusion backbone).
  * **Text Encoder**: `qwen3vl_4b_fp8_scaled.safetensors` (FP8 scaled).
  * **Sampling**: Euler sampler, `simple` scheduler, 8 steps, CFG 1.0.
  * **Automated Prompt Expander**: An integrated `TextGenerate` node utilizing Qwen3-VL 4B transforms short conceptual prompts into camera-authentic, 35mm optical descriptions (aperture, lighting physics, film grain, and anti-AI imperfection prompts).
* **Qwen-Image 2.1 T2I & Editing (`image_qwen_image_2_1_t2i.json`, `image_qwen_image_2_1_image_edit.json`)**:
  * **Model**: `qwen_image_2.1_int8_convrot.safetensors` with `QwenImage21Cache` enabled for memory efficiency.
  * **Text Encoder / VAE**: `qwen3vl_8b_int8_convrot.safetensors` & `qwen_image_2.1_vae_bf16.safetensors`.
  * **Inpainting & In-Context Modification**: Used to replace placeholder signage with crisp, legible typography (*"STARGATE AI DATACENTER"*) and modify company logos accurately.

#### B. Joint Video & Synchronized Audio Generation (MiniMax H3)
Unlike traditional pipelines requiring separate TTS and Foley generation, the film uses **MiniMax H3**, which natively synthesizes synchronized 24 FPS video and 32 kHz diegetic audio in a single latent diffusion pass.

* **Checkpoints & Precision**:
  * Diffusion: `minimax_h3_fl2va_pruned_int8_convrot.safetensors` (T2V/I2V) and `minimax_h3_ref2va_pruned_int8_convrot.safetensors` (Ref2VA).
  * Text Encoder: `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` (32B vision-language model in NVFP4 quantization).
  * Dual VAEs: `minimax_h3_video_vae_fp16.safetensors` (spatial video latent) and `minimax_h3_audio_vae_fp32.safetensors` (audio spectral latent).
* **Sampling Acceleration**:
  * Accelerated via the `taomate_h3_3step_comfy.safetensors` distilled LoRA.
  * Custom sampler: `SamplerCustomAdvanced` + `KSamplerSelect` (`res_multistep`) + `BasicScheduler` (3 steps, `simple`, denoise 1.0).
* **Temporal Dimension Formula**:
  MiniMax H3 requires temporal latent block alignment where frame count must satisfy $T \equiv 5 \pmod{17}$. Duration was calculated using an automated `ComfyMathExpression` node:
  $$\text{length} = \max(5, \text{round}(\text{duration} \times 24)) + \left(5 - (\max(5, \text{round}(\text{duration} \times 24)) \pmod{17})\right) \pmod{17}$$
* **Director-in-the-Loop Prompt Engineering (Ref2VA Schema)**:
  `video_minimax_h3_r2v.json` and `video_minimax_h3_i2v.json` implement an automated LLM cinematic director via Qwen-VL that parses reference images into a strict 6-part Context-IR schema:
  1. `subject_definitions:` Assigns dynamic tracking handles (`<Subject 1>`, `<Subject 2>`) to stitched image inputs.
  2. `summary:` Prefixed with `[reference generation]`.
  3. `retention_analysis:` Strict enforcement of `fully_preserved`, `partially_preserved`, or `weak_reference`.
  4. `detailed_description:` Natural-language camera kinematics (`[Camera] + [Motion] + [Amplitude] + [Speed] + [Framing]`), explicit millisecond cuts (`[Shot 2] At 00:03.500, the camera cuts to...`), dialogue blocks with `<d>[English] ...</d>` lip-sync tags, and explicit mouth-closure constraints.
  5. `overall_soundscape:` Foley, acoustic spatial reverberation, and background ambience.
  6. `non_diegetic_music:` Score and tempo progression.

#### C. Video Super-Resolution & Upscaling
Two upscaling pipelines were implemented to evaluate performance and quality:
1. **Diffusion-Based Upscaling (`video_upscale.json`)**:
   * **Model**: `seedvr2_7b_sharp_int8_convrot.safetensors` + `seedvr2_ema_vae_fp16.safetensors`.
   * **VRAM Management**: Uses `SeedVR2TemporalChunk` and `SeedVR2TemporalMerge` along with `VAEEncodeTiled` (tile size 512, overlap 32, temporal size 32, temporal overlap 4) to upscale long temporal sequences within GPU memory limits.
   * **Pre/Post Processing**: Lanczos 2x spatial resampling paired with `SeedVR2PostProcessing`.
2. **Hardware-Accelerated Super-Resolution (`video_upscale_rtx.json`)**:
   * **Node**: `RTXVideoSuperResolution` from `comfyui_nvidia_rtx_nodes`.
   * **Settings**: 2.8x scaling multiplier, `ULTRA` quality profile for fast high-resolution rendering.

#### D. In-Engine Compositing, Subtitling & Assembly (`compose.json`, `compose_4k.json`)
The final master was cut and finalized directly inside ComfyUI:
* **Frame-Accurate Subtitling Engine**: Modular subgraphs decompose input videos via `GetVideoComponents`, apply custom `ComfyMathExpression` logic to calculate starting, target, and trailing frame indices. Frames are partitioned via `ImageFromBatch`, burned with styled typographic text (`TextOverlay`), resized, and concatenated seamlessly with untouched frames via `BatchImagesNode`.
* **Audio Leveling & Trimming**: Trimming via `Video Slice`, audio level ducking using `AudioAdjustVolume`, and silent room-tone padding using `EmptyAudio` (32 kHz stereo).
* **Multi-Clip Concatenation**: Assembled sequentially through the `ConcatenateVideo` node.
* **Safety Watermarking**: Top-right persistent `"AI-Generated"` watermark burned via `TextOverlay` across all active video batches.
* **Master Encoding**: Exported via `SaveVideo` in MP4 container using the AV1 codec with auto-bitrate.

---

## 3. Shot List & Technical Storyboard

| Shot | Timecode | Visual Description | Pipeline & Models Used | Seed | Key Parameters / Directing Prompts |
|---|---|---|---|---|---|
| **1** | 0:00–0:02 | Establishing shot of tranquil pine forest. | `video_minimax_h3_t2v` (FL2VA INT8) | `992750253316` | **Steps:** 3, **FPS:** 24, **Length:** 49 frames.<br>**Prompt:** Live-action cinematic shot of a pristine, ancient pine forest at dawn. Sunbeams filter through morning mist. |
| **2** | 0:02–0:05 | 2D vector squirrel bounding; discovers acorn. | `video_minimax_h3_i2v` (FL2VA INT8) | `1` (Fixed) | **Steps:** 3, **FPS:** 24, **Length:** 73 frames.<br>**Input:** `edited_00003_.png`<br>**Dialogue / Audio:** Rustling leaves, footstep foley. |
| **3** | 0:05–0:11 | Sam Altman & Larry Ellison in high-tech server boardroom. | `video_minimax_h3_i2v` + `Add Subtitles` | `482094821` | **Steps:** 3, **FPS:** 24, **Trimming:** 6.0s.<br>**Subtitles:** `0.00-4.00s`: "The energy demands for Stargate are, frankly, astronomical, Sam" / `4.00-5.50s`: "The model requires it" / `5.50-8.00s`: "The grid will simply have to bend". |
| **4** | 0:11–0:16 | Drone timelapse of forest being cleared for datacenter. | `video_minimax_h3_t2v` | `102948192` | **Steps:** 3, **FPS:** 24, **Length:** 121 frames.<br>**Prompt:** High-altitude drone timelapse of lush green forest being aggressively excavated, replaced by mud, concrete slabs, and industrial foundations. |
| **5** | 0:16–0:19 | Gas/coal plant belching exhaust over arid land. | `image_krea2_turbo_t2i` ➔ `video_minimax_h3_i2v` | `645833452` | **Steps:** 3, **FPS:** 24, **Length:** 73 frames.<br>**Prompt:** Massive cooling towers and exhaust stacks venting dark combustion gases into a stagnant, hazy sky. |
| **6** | 0:19–0:24 | Squirrel confronts brutalist monolith and Stargate billboard. | `image_qwen_image_2_1_image_edit` ➔ `video_minimax_h3_i2v` | `902878438` | **Steps:** 3, **FPS:** 24, **Length:** 121 frames.<br>**Subtitles:** `0.00-2.50s`: "This is my forest" / `2.50-5.00s`: "I will not be ignored!". |
| **7** | 0:24–0:28 | Squirrel enters electrical substation and bites heavy cables. | `video_minimax_h3_r2v` (Ref2VA Mode) | `296745100` | **Steps:** 3, **FPS:** 24, **Length:** 97 frames.<br>**Inputs:** Squirrel (`ref_image_0`) + Substation (`ref_image_1`).<br>**Subtitles:** `1.00-4.00s`: "I'm going to take climate action!". |
| **8** | 0:28–0:32 | Datacenter power failure; server room blacks out; panic. | `video_minimax_h3_i2v` + `Add Subtitles` | `3829104` | **Steps:** 3, **FPS:** 24, **Length:** 97 frames.<br>**Subtitles:** `0.00-1.50s`: "Everything's going to plan, Sam" / `1.50-3.00s`: "The datacenter will be comple-" / `3.00-5.00s`: "Shit, they got us!". |
| **9** | 0:32–0:35 | Title card with skull/OpenAI logo graphic and impact stat. | `video_minimax_h3_t2v` (FL2VA INT8) | `7482910` | **Steps:** 3, **FPS:** 24, **Length:** 49 frames.<br>**Prompt:** Animated title card. Black background, stark white and red typography. Skull and crossbones with OpenAI logo as skull. Stat: "By 2030, AI infrastructure will consume as much power as entire industrialized nations." |

---

## 4. Hardware Requirements & Reproduction Guide

### Hardware Environment
* **GPU**: NVIDIA RTX 3090 (24 GB VRAM) or equivalent Ada Lovelace / Ampere GPU.
* **Host RAM**: 32 GB minimum (64 GB recommended for model caching).
* **Disk Space**: ~45 GB for diffusion models, text encoders, and VAE weights.

### Model Weight Directory Setup
Download and place weights into standard ComfyUI directories:

```
ComfyUI/models/
├── diffusion_models/
│   ├── krea2_turbo_int8_convrot.safetensors
│   ├── qwen_image_2.1_int8_convrot.safetensors
│   ├── minimax_h3_fl2va_pruned_int8_convrot.safetensors
│   ├── minimax_h3_ref2va_pruned_int8_convrot.safetensors
│   └── seedvr2_7b_sharp_int8_convrot.safetensors
├── text_encoders/
│   ├── qwen3vl_4b_fp8_scaled.safetensors
│   ├── qwen3vl_8b_int8_convrot.safetensors
│   └── qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors
├── vae/
│   ├── qwen_image_vae.safetensors
│   ├── qwen_image_2.1_vae_bf16.safetensors
│   ├── minimax_h3_video_vae_fp16.safetensors
│   ├── minimax_h3_audio_vae_fp32.safetensors
│   └── seedvr2_ema_vae_fp16.safetensors
└── loras/
    └── taomate_h3_3step_comfy.safetensors
```

### Required Custom Node Extensions
* `comfy-core` (v0.37.0+)
* `comfyui_nvidia_rtx_nodes` (RTX Video Super Resolution)
* `ComfyUI-SeedVR2` (Temporal latent upscaling & conditioning nodes)
* `ComfyUI-MiniMax-H3` (MiniMax Image-to-Video and Reference-to-Video support)

### How to Run the Workflows
1. **Asset Generation**: Open `image_krea2_turbo_t2i_int8.json` or `image_qwen_image_2_1_t2i.json` in ComfyUI. Generate keyframe assets. Run `image_qwen_image_2_1_image_edit.json` for logo and typography modifications.
2. **Video & Audio Synthesis**: Load the generated image into `video_minimax_h3_i2v.json` (or reference images into `video_minimax_h3_r2v.json`). Click **Queue Prompt**. The workflow evaluates duration via math nodes, prompts the internal LLM director, and generates synchronized video (`.mp4`/`.webm`) with diegetic audio.
3. **Upscaling**: Feed intermediate shot renders into `video_upscale.json` (for SeedVR2 diffusion enhancement) or `video_upscale_rtx.json` (for real-time tensor-accelerated 2.8x super-resolution).
4. **Master Assembly**: Load `compose.json` or `compose_4k.json`. Ensure shot clips are mapped into the respective `LoadVideo` input nodes. Queue the workflow. ComfyUI slices clips, dynamically overlays frame-calculated subtitles, normalizes audio tracks, appends watermarking, and renders the finished film to `output/video/composited.mp4`.

---

## 5. Team Contributions

* **Haki Abdulovski:** Factual background research, data sourcing (IEA, CBS, Floodlight), initial image prompt testing, script structure, and slide deck preparation.
* **Alex Wróbel:** ComfyUI workflow architecture design, INT8/NVFP4 model pipeline configuration, custom math node integration, video generation execution, upscaling pipelines, and final in-engine compositing.
* **Shared Effort:** Creative concept, storyboard development, narrative pacing, and presentation design.

---

## 6. Ethical Reflection

* **Informing vs. Misleading:**  
  The multi-gigawatt energy footprint, behind-the-meter gas turbine deployments, and extensive water consumption of hyperscale data centers are documented facts. The physical sabotage by a woodland creature and the exaggerated brutalist monolith are deliberate satire and artistic metaphors used to communicate scale and impact within a 35-second runtime.
* **Public Likenesses (Sam Altman & Larry Ellison):**  
  Portrayed strictly under political/corporate parody and fair-use critique regarding high-profile infrastructure decisions, avoiding malicious or defamatory personal context.
* **Provenance & Synthetic Labeling:**  
  A persistent, outline-rendered `"AI-Generated"` watermark is composited into the top-right corner of the video throughout its entire duration, preventing viewers from mistaking synthetic sequences for actual security or corporate footage.
* **Compute Footprint & Justification:**  
  Across the entire development cycle, 78 generation passes were logged across Krea 2 Turbo, Qwen-Image, and MiniMax H3. Total energy expenditure was approximately 2.1 kWh of local GPU compute (~0.85 kg $\text{CO}_2$). This small local footprint is offset by producing an educational asset that targets thousands of student AI power-users, promoting prompt discipline and query restraint.

---

## 7. Sources

1. Floodlight News (2026). *This permit, common for dry cleaners, is now being used to build AI power plants.*  
   https://floodlightnews.org/ai-data-centers-texas-pollution-loophole/
2. Centraal Bureau voor de Statistiek (CBS, 2026). *AI in de samenleving: ervaringen en opinies.*  
   https://www.cbs.nl/nl-nl/longread/diversen/2026/ai-in-de-samenleving-ervaringen-en-opinies/2-ervaring-met-ai
3. International Energy Agency (IEA). *Energy Demand from AI.*  
   https://www.iea.org/reports/energy-and-ai/energy-demand-from-ai
4. International Energy Agency (IEA). *Energy Supply for AI.*  
   https://www.iea.org/reports/energy-and-ai/energy-supply-for-ai