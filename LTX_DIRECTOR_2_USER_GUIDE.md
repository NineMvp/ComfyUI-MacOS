# Complete User Guide: LTX Director 2.0 with LTX-2.5 Distilled Q8_0 on Mac

Welcome to your comprehensive guide for using **LTX Director 2.0** integrated with **LTX-2.5 Distilled Q8_0**, **Gemma-4 12B Clip**, and **Apple Silicon Metal acceleration**.

---

## 1. What is LTX Director 2.0?

**LTX Director 2.0** turns ComfyUI into a full **non-linear timeline video editor** and director console for AI video generation. Instead of running separate workflows for every technique, you can do all of the following inside a single visual timeline:
1. **Text-to-Video (T2V)** with timed prompt changes along the timeline.
2. **Image-to-Video (I2V)** from a starting image.
3. **First-Frame + Last-Frame (FLF)** smooth scene morphing / camera travel.
4. **Multi-Image Sequencing** (chaining multiple shots seamlessly).
5. **Video-to-Video & Retake Mode** (re-generate only a specific 1–2 second segment of an existing video).
6. **Audio Inpainting & Sync** (matching video motion to sound effects or spoken dialogue).

---

## 2. How to Open the Workflow in ComfyUI

1. Open your browser and navigate to: **`http://127.0.0.1:8188`**.
2. Click **Workflow 📁 ➔ Open ➔ `LTX2.5_Director_2.0_GGUF_Mac`**.
3. You will see the canvas arranged with:
   - **Left Side**: Model Loaders (`LTX-2.5-Distilled-Q8_0.gguf`, `gemma4-12b-with-proj`, `ltx-2.5-video-vae-bf16`, `ltx-2.5-audio-vae-bf16`).
   - **Center**: The interactive **LTX Director 2.0 Console** (timeline UI with track layers).
   - **Right Side**: Sampler & Output Video Player (`SaveVideo`).

---

## 3. The Director 2.0 Timeline Interface Explained

```
+-------------------------------------------------------------------------+
| [Global Prompt] "Cinematic night street, neon reflections, rain..."     |
+-------------------------------------------------------------------------+
| [Track 1: Visual Keyframes]                                             |
| [Image 1 (Start)] ---------------------> [Image 2 (End)]               |
+-------------------------------------------------------------------------+
| [Track 2: Prompt Segments]                                              |
| "Camera pans up" ----> "Man looks at screen" ----> "Coffee steam rises" |
+-------------------------------------------------------------------------+
| [Track 3: Audio Track]                                                  |
| [Rain Sound / Speech Audio .wav]                                        |
+-------------------------------------------------------------------------+
| Duration: [5.0s] | FPS: [30] | Resolution: [1.0 MP / 1280x736]         |
+-------------------------------------------------------------------------+
```

### Key Controls in the Director Node:
- **Global Prompt**: The overarching artistic style, lighting, and cinematic quality.
- **Timeline Duration**: Set your target video length (default: `5` seconds).
- **FPS**: Set to `30` or `24` fps.
- **Main Track (Visuals)**: Add reference images to define starting, intermediate, or ending keyframes.
- **Prompt Track**: Add prompt markers that trigger at specific timestamps.
- **Audio Track**: Upload audio to synchronize audio-visual motion.

---

## 4. Step-by-Step Practical Modes

### Mode A: Standard Image-to-Video (I2V)
1. In the **Director 2.0** timeline, click **Add Keyframe** on Track 1 at timestamp `0.0s`.
2. Drag and drop your character or scene photo (e.g. `restored_cf_in_ChatGPT Image...png`).
3. In **Global Prompt**, describe the motion (e.g., *"Gentle camera push-in, soft natural breathing, rain falling through neon light beams"*).
4. Click **Queue Prompt**.

---

### Mode B: First-Frame + Last-Frame Morphing (FLF)
1. At timestamp `0.0s`, place your **Start Photo** (e.g. Character looking down).
2. At timestamp `5.0s` (the end of the timeline), place your **End Photo** (e.g. Character looking up at the sky).
3. In the prompt, describe the transition: *"The camera pans smoothly upward as the character slowly tilts their head toward the sky."*
4. Click **Queue Prompt**. The DiT engine smoothly interpolates all frames between the two keypoints.

---

### Mode C: Retake / Fix a Video Segment (Video-to-Video)
If you generated a video and want to change just seconds `2.0s` to `3.5s` without re-rendering everything from scratch:
1. In the Director node, toggle **Retake Mode: ON**.
2. Set **Retake Start**: `2.0s` and **Retake Length**: `1.5s`.
3. Enter your **Retake Prompt** (e.g., *"The character smiles and waves"*).
4. Set **Retake Strength** (e.g. `0.75` for moderate change, `1.0` for full replacement).
5. Click **Queue Prompt** — only that segment will be regenerated and blended into the video.

---

### Mode D: Audio-Driven Video
1. Enable the **Audio Track** on the Director timeline.
2. Upload your `.wav` or `.mp3` file (dialogue voiceover, rain sound, ambient music).
3. The workflow passes the audio latent directly through `LTXVConcatAVLatent` + `LTXVAudioVAEDecode` to output synchronized sound with video.

---

## 5. Recommended Settings for Mac M5 Max

| Setting | Recommended Value | Reason |
| :--- | :--- | :--- |
| **Model** | `LTX-2.5-Distilled-Q8_0.gguf` | 22B distilled model, highest visual fidelity with 8-step speed. |
| **Text Encoder** | `gemma4-12b-with-proj...safetensors` | Gemma-4 12B handles complex descriptive cinematic prompts. |
| **Resolution** | `1.0 MP` / `0.9 MP` (`1280x736`) | Standard 720p 16:9 widescreen (clean multiples of 32 for DiT). |
| **Steps & Sigmas** | 8 Steps (`ManualSigmas` `1.0 -> 0.0`) | Instant distilled convergence (~1 minute on Metal shaders). |
| **Sampler** | `euler_ancestral` or `euler` | Stable motion trajectory without artifact accumulation. |
| **Video CFG / Audio CFG** | `1.0` | Distilled LTX-2.5 is trained for CFG = 1.0 (do not set above 1.5). |

---

## 6. Troubleshooting & FAQs

- **Q: Why did my video have solid blue/purple frames before?**
  - **A**: The official template previously divided resolution in half and reinjected 100% noise before a 3-step Stage 2. We restructured this workflow to **Direct Native High-Resolution Sampling**, which denoises all frames across the full schedule directly into the VAE.
- **Q: What was the `avcodec_send_frame() returned 22` error?**
  - **A**: The raw audio waveform had out-of-bounds float values. We added waveform sanitization (`nan_to_num().clamp(-1.0, 1.0)`) and guarded muxing in `video_types.py` so exports always complete cleanly.
- **Q: Where are my finished videos saved?**
  - **A**: All outputs are saved in `output/video/` (e.g., `output/video/LTX-2.5_fixed_final_00001_.mp4`).
