# PhotoPhysics-Diffusion
We are teaching AI image generators to think like a cinematographer — not just paint like one. Is that Possible? Lets Try

## The Problem

Every AI image generator — Stable Diffusion, FLUX.1, Leonardo Lucid Realism, Midjourney — produces images with **TOO PERFECT? Too Clean? **
Real world surfaces have microscopic imperfections — skin pores, fabric weave, dust on a lens, slight film grain,fingerprints on glass. 
AI images are hyper-detailed at the macro level but unnaturally perfect at the micro level.
This is what makes them feel "rendered" rather than "photographed."

The model learned what photographs look like statistically — not how light, lenses,and physics actually produce photographs.

You cannot tell it:
- Skin tones should sit at **65 IRE**
- Highlights should roll off at **85 IRE**  
- The scene should read at **4200K**
- Shadows should hold detail at **8 IRE**
- Blur like a "Actual DoF"
- 
You can only write words like `"cinematic"` or `"warm"` and hope the model guesses correctly.

AND.....

## The LUT resolve the issue?

LUT (Look-Up Table) files are the universal colour language of professional post-production.
Every tool — DaVinci Resolve, Premiere Pro, Blender, After Effects — speaks `.cube`.

BUT...Every existing LUT node applies the LUT AFTER generation.
Not one uses a LUT as a conditioning input before or during the diffusion process.  
Wrong shadows get tinted — they are still wrong.

## The IDEA - Reverse Engineering? (or "Physics-based inverse rendering" as technical term?)

Think about how a professional photographer or cinematographer works. Before they call a shot "done," they check their waveform/histogram monitor. 
This is a tool that shows the brightness of every part of the image as a graph — they can see exactly where the shadows are sitting, where the skin tones land, where the highlights peak. 
They grade the image until those numbers are exactly right.

HERE example of scope reading
<img width="1262" height="911" alt="image" src="https://github.com/user-attachments/assets/0be1a189-e988-4192-b527-a35f1700cfe0" />

PhotoPhysics-Diffusion brings that same discipline to AI image generation. Instead of asking an AI to guess what "cinematic" means, we:

•	Measure what a real reference image actually looks like in precise numbers
•	Express those numbers as a structured colour recipe (using tools a professional already understands)
•	Feed that recipe to the AI generator so it creates the image with the right numbers from the start
•	Verify the output by measuring it — not just looking at it


## PQEM — Photometric Quality Evaluation Metric

Several research groups have already built impressive pieces of this puzzle. 
NVIDIA published DiffusionRenderer at CVPR 2025 — it can reverse-engineer the geometry, materials, and lighting of a scene from a photograph. 
BokehDiff (ICCV 2025) handles realistic lens blur. 
DiLightNet (SIGGRAPH 2024) handles lighting conditions.

so far still haven't see any colour science layer. 
They all output images using a simple one-line tone mapping operation — essentially the equivalent of slapping an "auto" filter on the result. 
The tonal structure, colour temperature, and broadcast accuracy of the output are afterthoughts.
We can/try(?) a new metric built from three guidance/measurements:

•	Luma Accuracy (LA) — how closely does the brightness distribution of the AI image match the target? Measured in IRE units — the same scale a waveform monitor uses. A score within 5 IRE means broadcast-legal.
•	Chroma Accuracy (CA) — how closely does the colour match the target? Measured on the vectorscope — the tool colourists use to check for colour casts and oversaturation.
•	Colour Temperature Error (CTE) — how far off is the colour temperature? Measured in Kelvin. A 500 Kelvin error is enormous to a trained eye — but completely invisible to every existing quality metric.

PhotoPhysics-Diffusion is the missing colour science layer that completes the stack. 
Stack all together and you have a system that generates images with correct geometry, correct lighting, correct lens physics **(this really no ideal how)**, and correct colour — the complete physical photography simulation.  

```
PQEM = w₁ × LA + w₂ × CA + w₃ × CTE

where:
LA = Luma Accuracy — waveform IRE error (0–100 IRE scale)
CA = Chroma Accuracy — vectorscope distance (hue° + saturation%)
CTE = Colour Temp Error — Kelvin difference from target
weights (suggested): w₁=0.5, w₂=0.3, w₃=0.2
score range: 0.0 (worst) → 1.0 (perfect match)
```

We can also build on PyMotion v1.5 — an open source Python library that already has a professional colour pipeline including ACES colour space, waveform monitor, vectorscope, and RGB parade built in. 
Rather than rebuilding those tools from scratch, IREScope uses PyMotion as its foundation and adds the AI generation bridge on top.

## Expected results

```
Real photograph
      ↓
IREScope extracts IRE profile (waveform + vectorscope + colour temperature)
      ↓
Structured numbers → .cube LUT → CtrLoRA conditioning
      ↓
New AI image — broadcast-accurate, metered, verified by PQEM
```
Run IREScope over 1,000 images covering the full tonal range — deep shadows to specular highlights, 2800K tungsten to 6500K daylight, under- to over-saturated. 
Store structured IRE metadata alongside each image. This dataset is the training foundation 
Build and train with HuggingFace?

| Method | SSIM↑ | NIQE↓ | BRISQUE↓ | **PQEM↑** |
|---|---|---|---|---|
| Leonardo baseline | 0.71 | 4.82 | 28.3 | 0.41 |
| + ApplyLUT (post-only) | 0.72 | 4.79 | 27.8 | 0.67 |
| + IREScope LUT | 0.72 | 4.81 | 28.1 | 0.84 |
| **+ CURRENT IDEA** | **0.74** | **4.77** | **27.4** | **0.93** |

## References

1. [DiffusionRenderer](https://arxiv.org/abs/2501.18590) — NVIDIA CVPR 2025 Oral
2. [SCEM](https://arxiv.org/abs/2603.00337) — Structured Control Embedding, March 2026
3. [Controllable Colour Editing](https://arxiv.org/abs/2509.13756) — confirms language insufficient for colour
4. [Colour Alignment in Diffusion](https://arxiv.org/abs/2503.06746) — colour statistics conditioning
5. [CtrLoRA](https://github.com/xyfJASON/ctrlora) — ICLR 2025, 1k images 1hr training
6. [BokehDiff](https://github.com/FreeButUselessSoul/bokehdiff) — ICCV 2025
7. [DiLightNet](https://github.com/iamNCJ/DiLightNet) — SIGGRAPH 2024
8. [Generative Photography](https://arxiv.org/abs/2412.02168) — CVPR 2025, proves focal length gap
9. [LLIE Taxonomy Survey](https://arxiv.org/abs/2510.05976) — maps entire field
10. [PyMotion](https://github.com/Ohswedd/pymotion) — colour science foundation

```
@misc {PhotoPhysics-Diffusion,
  name = {gogoustudio|khorfooweng}
  reason = {make my life abit tougher}
  Please join this in anyone is interested.
  Thanks
}

