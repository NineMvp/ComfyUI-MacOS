# Complete Detailed Guide: SmartVision LTX 2.5 Director Workflow on Mac

This guide explains in complete detail how to use the new **SmartVision LTX 2.5 Director Workflow** (`LTX2.5_SmartVision_Director_GGUF_Mac.json`) with **LTX-2.5 Distilled Q8_0** and **Gemma-4 12B Clip** on Apple Silicon macOS.

---

## 1. Overview: What Makes This Director Workflow Unique?

In traditional AI video workflows:
- A single prompt controls the entire 5 seconds.
- Camera movements often clash with subject motion.
- Describing action in the second half of the video bleeds backward into the start.

The **SmartVision LTX 2.5 Director** solves this with:
1. **Shot Timeline Segmentation**: Divide your video into distinct chronological shots (e.g. Shot 1 = 0s–2.5s, Shot 2 = 2.5s–5.0s).
2. **Cross-Attention Prompt Relay**: Actions described in Shot 2 *only* apply during Shot 2.
3. **Camera Director Presets**: Choose camera moves (*Push In, Pan, Tilt, Tracking, Orbit*) with Speed & Amplitude dials per shot.
4. **Per-Shot Image Anchors (`image_1` … `image_8`)**: Attach reference images directly to any shot as either a **Start Keyframe** or an **End Keyframe**.

---

## 2. Opening the Workflow in ComfyUI

1. Open your browser at: **`http://127.0.0.1:8188`**.
2. Click **Workflow 📁 ➔ Open ➔ `LTX2.5_SmartVision_Director_GGUF_Mac`**.

---

## 3. Node Canvas Architecture Explained

```
+-------------------------------------------------------------------------------------------------------+
|  [UNET Loader GGUF]  --> LTX-2.5-Distilled-Q8_0.gguf                                                   |
|  [CLIP Loader]       --> gemma4-12b-with-proj-ltx-2.5-comfy-int8-convrot.safetensors                  |
|  [Video VAE Loader]  --> ltx-2.5-video-vae-bf16.safetensors                                           |
|  [Audio VAE Loader]  --> ltx-2.5-audio-vae-bf16.safetensors                                           |
+-------------------------------------------------------------------------------------------------------+
                                                     │
                                                     ▼
                                    +----------------------------------+
                                    |     LTX 2.5 DIRECTOR CONSOLE     |
   [LoadImage: Shot 1] ──preprocess─┤ • Timeline / Shot Segments       |
   [LoadImage: Shot 2] ──preprocess─┤ • Camera Presets & Speed Dials   |
                                    | • Global Prompt vs Shot Prompts  |
                                    +----------------------------------+
                                                     │
                                                     ▼
                                    +----------------------------------+
                                    |     STAGE 1 SAMPLER (8 steps)    |
                                    | (DualCFG Guider: Video=1, Audio=1)
                                    +----------------------------------+
                                                     │
                                                     ▼
                                    +----------------------------------+
                                    |    LTX25DirectorCrop (Guides)    |
                                    |  Latent Spatial Upscaler ×2      |
                                    +----------------------------------+
                                                     │
                                                     ▼
                                    +----------------------------------+
                                    |     STAGE 2 SAMPLER (Refine)     |
                                    +----------------------------------+
                                                     │
                                                     ▼
                                    +----------------------------------+
                                    |   VAEDecodeTiled & SaveVideo     |
                                    +----------------------------------+
```

---

## 4. How to Use the Director Timeline UI Step-by-Step

### A. The Global Prompt vs. Shot Action Prompts

There are **two types of prompts** on the Director node:

1. **Global Prompt (Bottom Textbox)**:
   - Use this **ONLY** for world-building, environment, lighting, character appearance, and artistic medium.
   - *Example*:
     ```text
     Cinematic 35mm film, hyper-realistic, rainy Tokyo night street, wet asphalt glistening with neon magenta and cyan reflections, soft atmospheric mist, melancholic tranquil mood, high dynamic range.
     ```

2. **Shot Action Prompt (Inside each Shot Block)**:
   - Use this **ONLY** for the specific character/camera action happening during that shot window.
   - Do **NOT** repeat environment/lighting description here.
   - *Example for Shot 1*:
     ```text
     The man in the dark coat slowly walks along the wet crosswalk.
     ```
   - *Example for Shot 2*:
     ```text
     The man stops, looks up, and gazes at the glowing neon billboard above.
     ```

---

### B. Setting Camera Motion per Shot

Click on any Shot block on the timeline to select it. Directly beneath the timeline, configure the **Camera Controls**:

| Control | Options | What It Does |
| :--- | :--- | :--- |
| **Camera** | `none`, `push_in`, `pull_out`, `pan_left`, `pan_right`, `tilt_up`, `tilt_down`, `crane_up`, `orbit_left`, `orbit_right`, `tracking_shot` | Compiles a natural camera direction sentence into this specific shot window. |
| **Speed** | `slow`, `normal`, `fast` | Controls the pace and smoothness of the camera movement. |
| **Amp (Amplitude)** | `subtle`, `moderate`, `dramatic` | Controls the camera travel distance / angle intensity. |

---

### C. Attaching Reference Images to Shots

1. **Shot 1 Starting Image**:
   - In the **`Shot 1 image → image_1`** node (left side of canvas), load your starting photo.
   - On the timeline, click **Shot 1**.
   - Make sure **Image Slot: 1** is selected and `is_end_frame` is **OFF** (Start frame).
2. **Shot 2 Ending Image (First-Last Frame Morphing)**:
   - In the **`Shot 2 image → image_2`** node, load your target ending photo.
   - On the timeline, click **Shot 2**.
   - Select **Image Slot: 2** and toggle `is_end_frame` to **ON** (`Place at end of this shot`).
3. **Guide Strength**:
   - Set to `0.7` – `0.85` (higher = strictly matches image structure, lower = allows more free motion).

---

## 5. Summary of Recommended Settings

- **Duration**: `5.0` seconds (120 frames at 24fps or 150 frames at 30fps).
- **Resolution**: `768x512` in Stage 1, automatically upscaled to `1536x1024` in Stage 2 (or direct 1.0 MP mode).
- **DualCFGGuider**:
  - `video_cfg`: `1.0`
  - `audio_cfg`: `1.0`
- **Enable Relay**: `True` (Prevents prompt bleed between shots).

---

## 6. Where Are Output Videos Saved?

Finished videos are rendered and saved automatically in:
📁 **`output/video/`** (e.g., `output/video/LTX2.5_SmartVision_00001_.mp4`)
