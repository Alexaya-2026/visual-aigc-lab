# Stage 3 — Precision Control, Detailed Breakdown
### Companion to comfyui-mastery-roadmap.md — this expands Stage 3 into daily sessions

Same format as the Stage 1 doc: each session is ~20–40 minutes of hands-on work (a few run longer once ControlNet and full-pipeline days show up — flagged below), one variable at a time. Do them roughly in order — later sessions assume earlier intuition. Report back what you observe and I'll react before we move on.

**Assumed starting point:** you've cleared the Stage 2 graduation check — a background swap and a white cutout on a new product photo, first or second try, no hand-holding. If that's not solid yet, it's worth another few reps there before this stage; masking and segmentation both get harder to learn if you're still unsure whether a bad result is a prompting problem or a masking problem.

**One habit to start now, not later:** keep a running note — settings, before/after, what worked — as you go through these sessions. This is the "swipe file" from the main roadmap's methodology section, and Stage 3 is where it starts paying off, because "which grow/feather value worked for glass vs. fabric" and "which segmentation model won on my product type" are exactly the things you won't remember in three weeks without writing them down.

**One caveat specific to this stage:** custom-node names and which segmentation/upscale model is "best right now" shift faster than core sampler concepts do — this is genuinely one of the fastest-moving corners of the ComfyUI ecosystem. The node names below are accurate as of now, but if Manager surfaces something with a slightly different name, search by *capability* ("background removal," "controlnet preprocessor," "upscale model") rather than hunting for the exact string.

**Baseline to pin before Day 1:** masking, segmentation, and ControlNet are all less sensitive to steps/cfg than a pure edit is — but Days 1–3 are only a fair comparison against *each other* if you're not also quietly drifting model version, cfg, or LoRA state between them. Use whatever you landed on as your working baseline back in Stage 2 (model, steps, cfg, LoRA on/off) and hold it fixed across Days 1–3 unless a day specifically tells you to change it.

---

### Day 1 — Manual masking, the core loop
**Goal:** edit only a selected region while leaving every other pixel untouched — the thing Qwen's whole-image regeneration can't guarantee.

- Load a product photo you already know well (reuse one from Stage 1/2 if you can — fewer new variables).
- Right-click the loaded image → **Open in MaskEditor**, paint over one *small, low-stakes* region only (a shadow patch, a bit of background beside the product — not the product itself yet). Save to node.
- Build the minimal masked-edit path: `Load Image` → `VAE Encode (for Inpainting)` (takes both the image and your mask) → `KSampler` → `VAE Decode`. This is the standard route for masked edits driven by a text instruction, which is your situation.
- Prompt only for what you want *inside* the mask (e.g., "remove the dust").
- **Report back:** did the pixels outside your painted mask stay identical, or did you see any bleed at the edge?

---

### Day 2 — Mask geometry: grow and feather
**Goal:** fix the hard-edge seam problem before it shows up on a real product.

- Same mask as Day 1. On `VAE Encode (for Inpainting)`, compare `grow_mask_by = 0` against a small positive value — this pads the mask so the model has a bit of context past your exact paint line instead of stopping dead at it.
- Add a `Feather Mask` (or a mask-blur step) between your mask and the encode node. Compare a hard-edge result to a feathered one.
- **Report back:** at what grow/feather values does the seam disappear, without the edit visibly creeping into pixels you meant to leave alone?

---

### Day 3 — Two masking paths, and when to use each
**Goal:** know which masking route to reach for, since tutorials mix these without saying why.

- Same photo and mask. Run it two ways: **(a)** `VAE Encode (for Inpainting)` + standard model + high denoise (~1.0) + your text instruction. **(b)** `VAE Encode` (plain) → `Set Latent Noise Mask` → same standard model, but a lower denoise (try 0.4–0.6).
- Path (a) is built for inpainting-specific checkpoints but works reasonably with a standard model plus a clear instruction — which is closer to your Qwen setup. Path (b) treats the masked area as noise for the sampler and tends to want a lower denoise to look finished.
- **Report back:** which gave a cleaner blend for an *instruction-driven* edit, and which needed less denoise before it stopped looking unfinished?

This is also a second look at something you already found in Stage 1 Day 5: Qwen conditions on the reference image through the text encoder, not through partial noise, which is exactly why path (a)'s high denoise suits an edit model while path (b)'s lower denoise is doing something conceptually closer to classic img2img. If path (b) looked worse here, treat that as confirmation of the same mechanism in a masking context — not a new, separate problem.

---

### Day 4 — Automatic segmentation
**Goal:** a pixel-precise cutout with no hand-painting at all.

- Via ComfyUI Manager, install a background-removal/segmentation node pack — search "RMBG" or "rembg." A currently well-maintained option bundles several model backends in one node (RMBG-2.0, BiRefNet, BEN2, SAM among them), so you can A/B backends without reinstalling anything.
- Run your *messiest-background* product photo through it with two different backends (e.g., a general-purpose model vs. one of the BiRefNet variants, which tends to hold up better on fine detail like mesh, fur, or hair-like edges).
- **Report back:** which backend handled your actual product category best — and did it beat what hand-masking got you in Day 1–3 in the same amount of time?

---

### Day 5 — Edge quality: killing the halo
**Goal:** understand why cutouts get a light or dark fringe, and how to fix it.

- Take your Day 4 cutout and zoom in ~400% on an edge, ideally a curved or fine-detail one.
- Look for a threshold/sensitivity parameter on the segmentation node, and — if the node exposes it — an edge-refinement or alpha-matting option (these extend the cut into a soft boundary zone instead of a hard binary line).
- Composite the result onto a new solid-color background and check for a bright or dark rim around the product.
- **Report back:** fringe gone, reduced, or unchanged — and which parameter actually moved it?

---

### Day 6 — ControlNet basics: which Qwen ControlNet did you actually download? *(runs longer — read before you download anything)*
**Goal:** guide structure without a mask or a full redraw — but first, avoid an hour of confused wiring, because Qwen doesn't have one ControlNet, it has three separate, incompatible ones.

- **Read this before installing anything.** Qwen ControlNet support currently comes in three families that are not interchangeable and wire completely differently:
  - **InstantX Union** (`Qwen-Image-InstantX-ControlNet-Union.safetensors`) — a true ControlNet. Wires the classic way: `Load ControlNet Model` → `Apply ControlNet`. One file covers canny, soft edge, depth, and pose — you just swap which preprocessor feeds it.
  - **DiffSynth model patches** (`qwen_image_canny_diffsynth_controlnet.safetensors`, plus separate `_depth_` and `_inpaint_` versions) — despite the name, these are *not* ControlNets, they're model patches. Different graph entirely: `ModelPatchLoader` → `QwenImageDiffsynthControlnet`, inserted into the model flow rather than the conditioning flow, and you need a separate file per control type. This is what ComfyUI's own official Qwen tutorial leads with, so it's an easy trap to land in if you just search "controlnet" in Manager and grab the first canny result.
  - **Union DiffSynth LoRA** (`qwen_image_union_diffsynth_lora.safetensors`) — wired differently again, as a LoRA via `LoraLoaderModelOnly`. Covers lineart/softedge/normal/openpose.
- **For this session, deliberately go get InstantX Union.** It matches a standard `Apply ControlNet` graph, it's one file instead of three, and it's the option multiple independent write-ups converge on as the simplest starting point. Search specifically for "InstantX," not just "controlnet," so you don't end up with the DiffSynth version by accident.
- Install ControlNet preprocessor nodes via Manager (search "controlnet aux") if you don't already have them.
- Build: `Load Image` → canny preprocessor → `Apply ControlNet` (feeds your conditioning) → `KSampler`.
- Fixed seed, sweep `strength` at 0.3, 0.6, 1.0.
- **Report back:** at what strength does the output start looking locked-in/rigid rather than just structurally guided? Also confirm which of the three families you actually ended up with — it matters for Day 7.

---

### Day 7 — Depth, plus timing controls
**Goal:** same mechanism, a different signal, plus *when* during sampling it applies.

- If you're on InstantX Union from Day 6, this is simple: swap your canny preprocessor for a depth preprocessor (a Lotus Depth or DepthAnythingV2 node both work), same `Apply ControlNet` graph, same model file — you're just changing which map feeds it. If you ended up on DiffSynth instead, you'll need the separate depth patch file and the same `ModelPatchLoader` graph as Day 6, just swapped to depth — it's genuinely a different setup, not a settings change.
- Fixed seed and prompt. Now hold strength fixed and vary `start_percent`/`end_percent` instead — e.g., only apply control for the first half of the steps vs. the whole run. This lets structure lock in early while later steps stay free to refine texture and color.
- **Report back:** comparing canny vs. depth on the same photo, which felt more useful for "place the product in a specific composition"?

---

### Day 8 — Applying it: staged product composition *(longer session — this is a real mini-project)*
**Goal:** the actual e-commerce use case — putting your product into a reference layout.

- Take a reference "empty scene" photo (a shelf, a table setting) as your ControlNet guide image, plus your clean product cutout from Day 4–5.
- Try both approaches: **composite-first** (drop the cutout into the scene, then run a light edit pass to blend lighting/shadow) vs. **generate-with-guide** (feed the scene's depth/canny map as the ControlNet guide and edit-prompt the product into it).
- **Report back:** which got you a more convincing result with less fighting?

---

### Day 9 — Upscaling: model-based vs. plain resize
**Goal:** marketplace/zoom-ready output, without the soft look of a plain enlarge.

- Install an upscale model via Manager if you don't have one — a solid general-purpose starting point is 4x-UltraSharp for photorealistic products (RealESRGAN x4plus is a reasonable alternative).
- Build: `Load Upscale Model` → `Upscale Image (Using Model)`, and compare against a plain bicubic/Lanczos `Image Scale` node at the same target resolution.
- **Report back:** on a 100% crop, what's the actual visible difference between the model upscale and the plain resize?

---

### Day 10 — Upscaling at scale: tiling
**Goal:** upscale large or high-res images without running out of VRAM.

- If your GPU is modest or your target size is large, install **Ultimate SD Upscale** via Manager. It tiles the image, upscales each tile, and can optionally run a light img2img pass per tile at low denoise (0.3–0.5) to add real detail rather than just interpolate.
- Run it once with that img2img pass on, once off, same tile size.
- **Report back:** did the img2img pass add real detail, or mostly just add time? Note any visible seams between tiles.

---

### Day 11 — Full pipeline dry run *(this is your graduation check)*
**Goal:** chain everything into the actual Stage 3 deliverable.

- A new, never-before-seen product photo with a genuinely messy background.
- Pipeline: segment/cutout (Day 4–5 settings) → composite onto a new background → optional ControlNet-guided placement/lighting pass (Day 6–8) → upscale (Day 9–10) → save.
- Zoom to 100% on every edge: no halo, no fringing, product itself untouched.
- **Report back:** does it hold up unassisted, first or second try? That's the Stage 3 graduation signal from the main roadmap.

---

## Self-test before moving to Stage 4
Without running anything, predict:
1. Why does a masked edit sometimes still bleed slightly outside your painted line, and which two parameters fix that?
2. Why must a ControlNet model match your base model family, and what actually happens if it doesn't?
3. At the pixel level, why is a model-based upscale different from a plain resize?
4. When would you reach for automatic segmentation instead of hand-painting a mask, and when would you do the reverse?

If you can answer all four confidently, you're done with Stage 3 — genuinely, not just on paper.
