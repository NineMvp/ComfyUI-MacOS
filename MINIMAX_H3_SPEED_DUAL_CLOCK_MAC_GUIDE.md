# MiniMax H3 Speed & Dual-Clock Audio Workflow (Mac Apple Silicon Guide)

This guide covers running **Prompt Mastery's "Best MiniMax H3 Speed Workflow"** ([YouTube eb4rfamdJT8](https://www.youtube.com/watch?v=eb4rfamdJT8)) on **MacBook Pro M5 Max (128 GB Unified Memory)**.

---

## 1. Why This Workflow Excels

1. **6-Step Single-Stage Speed**: Unlike traditional 12–20 step two-pass H3 workflows, this workflow uses step-distillation with a single 6-step pass at ~1 MP resolution.
2. **Dual-Clock Sampler (`MiniMaxH3DualClockSamplerT8`)**:
   - Audio and video denoise on independent schedules (`shift_video=12.0`, `shift_audio=3.0`).
   - Prevents word-dropping or robotic audio degradation even at ultra-low step counts (4 to 6 steps).
3. **Audio Lock (`lock_source`)**:
   - Preserves source audio track timing, rhythm, and phrasing while synchronizing character lip and body motion.

---

## 2. Mac M5 Max (128 GB) Compatibility & Optimizations

- **PyTorch Native SDPA**: PC tutorials use CUDA-exclusive SageAttention or NVFP4 Triton patches. On Apple Silicon MPS with 128 GB unified memory, these are bypassed. PyTorch SDPA runs out of the box with zero VRAM pressure.
- **Node Pack Installed**: `comfyui-minimax-h3-audio-T8` is cloned and active in `custom_nodes/comfyui-minimax-h3-audio-T8/`. All 150+ T8 audio, speed, and dual-clock nodes are registered in your running ComfyUI instance.

---

## 3. Required Models & Locations

| Component | Recommended File | Local Status / Path |
| :--- | :--- | :--- |
| **Base / Singularity Model** | `Minimax-h3_Singularity_ref2va_Pruned_v1.3_int8.safetensors` (21 GB) or `minimax_h3_fl2va_pruned_int8_convrot.safetensors` | `models/diffusion_models/` |
| **Text Encoder** | `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` or standard Qwen3-VL | `models/text_encoders/` |
| **Video VAE** | `minimax_h3_video_vae_fp16.safetensors` | ✅ Installed in `models/vae/` |
| **Audio VAE** | `minimax_h3_audio_vae_fp32.safetensors` | ✅ Installed in `models/vae/` |
| **Acceleration LoRA** | `minimax_h3_turbo_4step.safetensors` | ✅ Installed in `models/loras/` |

---

## 4. How to Load in ComfyUI

1. Open `http://127.0.0.1:8188`.
2. Click **Workflow** -> **Open** (or load from browser workspace).
3. Select **`MiniMax_H3_Speed_DualClock_Mac.json`** from `user/default/workflows/`.
4. Drop your reference audio in the **LoadAudio** node.
5. In **MiniMaxH3DualClockSamplerT8**, default steps is set to `6` (or `4` for maximum speed).
6. Click **Queue Prompt**.
