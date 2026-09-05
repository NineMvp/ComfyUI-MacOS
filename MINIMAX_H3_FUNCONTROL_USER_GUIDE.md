# Complete Guide: MiniMax H3 Fun ControlNet & Turbo LoRA on Mac M5 Max (128GB)

This document provides complete instructions on how to use **MiniMax H3 Fun ControlNet** and **MiniMax H3 Turbo LoRA** on your MacBook Pro M5 Max with ComfyUI.

---

## 1. What This System Does

- **MiniMax H3 (33B Model)**: Generates high-fidelity video with natively synchronized stereo audio.
- **Fun ControlNet (`ComfyUI-H3-FunControl`)**: Directs the video output frame-by-frame using a control video (Depth, OpenPose, Canny, HED, or MLSD).
- **Turbo / Acc LoRAs**: Compresses the required sampling steps from 20–30 steps down to **4 to 8 steps** (a 3x to 5x generation speedup).

---

## 2. Directory & Model Setup

The following models are configured in your ComfyUI workspace:

| Model Type | File Name | Destination Directory |
| :--- | :--- | :--- |
| **ControlNet** | `minimax_h3_fun_controlnet_union_pruned_bf16.safetensors` | `models/controlnet/` |
| **Acc LoRA (8-Step)** | `MiniMax-H3-FL2VA-Acc-8Step_pruned_comfy.safetensors` | `models/loras/` |
| **Turbo LoRA (4-Step)** | `minimax_h3_turbo_4step.safetensors` | `models/loras/` |
| **Base Model (6-bit)** | `MiniMax-H3-MLX-6bit` / `model-*.safetensors` | `.omlx/models/pipenetwork/MiniMax-H3-MLX-6bit/` |

---

## 3. Workflows Available in ComfyUI

Open **`http://127.0.0.1:8188`** and load any of the pre-configured workflows via **Workflow 📁 ➔ Open**:

### 🟢 1. Single Control Video (`MiniMax_H3_FunControl_01_single_control.json`)
Use this when you have **one** driving video (e.g., Depth video or Pose video):
1. **`LoadVideo`**: Upload your guide video (depth map, skeleton animation, or edge video).
2. **`H3FunControlLoader`**: Select `minimax_h3_fun_controlnet_union_pruned_bf16.safetensors`.
3. **`H3FunControlApply`**:
   - `strength`: `0.75` – `0.85`
   - `start_percent`: `0.0`
   - `end_percent`: `0.85` (allows natural organic finishing on the last 15% of frames).
4. **`MiniMaxH3ImageToVideo`**: Enter your text prompt and optional start frame image.
5. Click **Queue Prompt**.

---

### 🔵 2. Multi-Control (Depth + Pose Reference) (`MiniMax_H3_FunControl_02_depth_plus_pose_reference.json`)
Use this when combining **both** spatial depth and actor skeletal pose simultaneously:
1. First control stream feeds the Depth video into `H3FunControlApply` #1.
2. Second control stream feeds the OpenPose video into `H3FunControlApply` #2.
3. Both control constraints guide the MiniMax DiT blocks simultaneously without conflict.

---

## 4. How to Activate Turbo / 4–8 Step Acceleration

To accelerate generation from 20 steps down to 4–8 steps:
1. Add a **`LoraLoader`** between the base model loader and the Sampler.
2. Select:
   - `minimax_h3_turbo_4step.safetensors` (for 4 steps) OR
   - `MiniMax-H3-FL2VA-Acc-8Step_pruned_comfy.safetensors` (for 8 steps).
3. Set `strength_model`: `1.0`.
4. In `BasicScheduler`: Set `steps` to `4` (or `8`).
5. In `BasicGuider`: Set `cfg` to `1.0`.

---

## 5. Output Video & Audio

Outputs are generated at **768p / 24fps** with synchronized stereo audio and saved automatically into:
📁 **`output/video/`**
