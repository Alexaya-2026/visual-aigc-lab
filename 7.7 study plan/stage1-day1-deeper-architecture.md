# Day 1, Deeper Still — Conditioning Mechanics, LoRA Internals, Model Precision
### A real second layer on Day 1 — not a repeat of what's already in stage-1-qa-notes.md

This covers three things we haven't touched yet: what "conditioning" literally *is* and how positive+negative combine mathematically (this connects straight into Day 3's `cfg`), what a LoRA is actually doing to the model's weights, and what the fp8/precision suffixes in your model filenames mean.

---

## 1. What "conditioning" literally is, and how cfg uses both positive AND negative

`TextEncodeQwenImageEditPlus` outputs something called **conditioning** — not text, not an image, but a set of numeric embedding vectors representing your instruction (and reference image, via the vision-language encoder) in a form the diffusion model's internal attention layers can actually use to steer denoising.

Here's the part that ties Day 1 directly into Day 3's `cfg`, and explains *exactly* what broke your very first image:

At each KSampler step, when `cfg` isn't 1.0, the model doesn't produce one prediction — it produces **two**: one guided by your **positive** conditioning, one guided by your **negative** conditioning. Then it combines them with this formula:

```
final_prediction = negative_prediction + cfg × (positive_prediction − negative_prediction)
```

This is called **classifier-free guidance** — literally where the "cfg" name comes from. Read what it says: `cfg` is how far *past* the negative direction, *toward* the positive direction, to extrapolate.

- `cfg = 1.0` → no extrapolation at all, just use the positive prediction directly. This is why Lightning-distilled LoRAs want `cfg=1` — they were trained assuming no extrapolation.
- `cfg = 8.0` → extrapolate 8× past the gap between negative and positive. That's an aggressive push — and it's *precisely* the mechanism behind your original blown-out, oversaturated image. You weren't just "using a high number," you were mathematically over-extrapolating far past what the model's predictions actually supported.

This is worth sitting with: your very first debugging win (Lightning LoRA + wrong cfg) wasn't a lucky settings fix — it was you correctly diagnosing a classifier-free guidance mismatch, several sessions before you had the formula for it.

---

## 2. What a LoRA actually does to the model's weights

"LoRA" = **Low-Rank Adaptation**. Concretely: instead of storing a full retrained copy of a model's weights, a LoRA stores a much smaller pair of matrices that, multiplied together, produce a *correction* to specific weight matrices inside the model (typically in the attention layers).

At inference:
```
effective_weights = original_weights + strength_model × (LoRA correction)
```

That's what `strength_model` is — a direct multiplier on how much the correction gets applied. `1.00` = full effect, exactly as the LoRA was trained. If you ever want a partial blend between base-model behavior and Lightning-distilled behavior (rather than fully one or the other), dialing `strength_model` down to something like `0.5` is a real, meaningful knob — not just an on/off switch. Worth remembering for when you revisit the LoRA later.

---

## 3. What "fp8" / "scaled" / "mixed" mean in your model filenames

You're loading `qwen_image_edit_2511_fp8mixed.safetensors` and `qwen_2.5_vl_7b_fp8_scaled.safetensors` — these suffixes aren't decoration, they tell you exactly what trade-off was made:

- **fp8** — weights stored as 8-bit floating point instead of the usual 16- or 32-bit. Roughly half (or more) the file size and VRAM usage of a 16-bit version, with some precision cost. This is *why* you can run a multi-billion-parameter model on consumer hardware at all.
- **scaled** — fp8 has a narrow representable numeric range. "Scaled" means per-tensor (or per-channel) scaling factors are applied so values make full use of that narrow range instead of clustering imprecisely — a technique to claw back quality that plain fp8 would otherwise lose.
- **mixed** — not every layer is compressed equally. Layers most sensitive to precision loss are kept at higher precision (e.g. bf16), while the bulk of less-sensitive weights get compressed to fp8. It's a deliberate quality/efficiency balance, not uniform compression.

You'll see this same naming pattern (fp8, bf16, fp16, GGUF Q4/Q8, etc.) on nearly every model you download from here on — recognizing the pattern is the useful part, not memorizing quantization theory.

---

## Self-test

Explain in your own words: what does `cfg` mathematically do, using *both* positive and negative conditioning — not just "it steers toward the prompt." If you can state the extrapolation idea back without looking, this layer of Day 1 is genuinely done.
