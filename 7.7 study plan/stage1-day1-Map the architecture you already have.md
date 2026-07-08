
| Node                            | What it actually does                                                                                                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Load CLIP**                   | Text → embeddings (meaning-vectors). In this Qwen setup it also reads images, because it's a vision-language model — but its core job is language understanding.                                                                                 |
| **Load VAE**                    | Pixels ↔ latent, both directions. Nothing to do with text, ever.                                                                                                                                                                                 |
| **KSampler**                    | The actual engine. Runs `steps` rounds of denoising, using `cfg` to control how hard each round gets pulled toward your positive prompt and away from your negative one. Everything else in the graph exists just to feed this node good inputs. |
| **EmptySD3LatentImage**         | A blank canvas of pure noise, sized W×H. "SD3" is a leftover name — no SD3 model involved.                                                                                                                                                       |
| **TextEncodeQwenImageEditPlus** | Where your photo actually enters the pipeline — through `image1` + `vae`, baked into the conditioning.                                                                                                                                           |
| **VAE Decode**                  | Latent tensor → real pixels.                                                                                                                                                                                                                     |

The one insight worth really sitting with: there are **two separate channels** feeding KSampler, and they carry completely different information —

- the **latent** channel carries _nothing but noise_ (no trace of your photo)
- the **conditioning** channel carries _everything_ — your prompt, and your photo's content, via the encode nodes

KSampler is where those two meet, and denoising is the process of gradually shaping the noise to match what the conditioning is describing. That's why `denoise = 1.0` is correct here — the sampler needs full freedom to build the image from scratch, because it's not starting from a blurred version of your photo, it's starting from nothing and being steered entirely by conditioning.

Quick check for yourself, no need to write it out unless you want to: if you set `denoise` to `0.5` instead of `1.0` in this specific graph, what would you expect to happen to the output — and why? (Hint: think about what `denoise` controls versus what actually contains your photo's information.)

---

**Solid, no changes needed:**

- **Load Diffusion Model** ✅ — correct, nothing to add.
- **Load LoRA** ✅ — good understanding, matches everything we've covered.
- **Save Image** ✅ — correct.
- **"Why three loaders instead of one checkpoint"** ✅ — exactly right. A checkpoint file bundles all three together; splitting them lets you mix and match (swap just the VAE, just the CLIP, etc.) instead of everything being locked together.

**One important mix-up to fix — CLIP and VAE have swapped roles in your notes:**

- **Load CLIP** is actually the **text** encoder. Its main job is converting your _prompt_ into vectors the model can use — not images. (In this specific Qwen setup it _can_ also read images, because `qwen_2.5_vl_7b_fp8_scaled` is a "vision-language" model — but its core, defining job is turning text into meaning-vectors.)
- **Load VAE** has nothing to do with text at all. A VAE converts between **real pixels** and **latent space**, in both directions — it compresses a real image down into latent space, and decompresses a latent tensor back into a real image. That's it. No decoder-of-prompts involved.

So swap your two descriptions and it's correct: CLIP reads text (and here, images too), VAE converts pixels ↔ latent. This distinction matters a lot going forward — it's exactly why VAE gets plugged into _both_ the text-encode nodes (to help encode your reference photo) _and_ the final decode step.

**KSampler — this one undersells it a lot:** "A basket that gathers parameters" makes it sound passive. It's the opposite — **KSampler is the engine that does the actual work.** It's what runs the `steps` iterations of denoising, using `cfg` to decide how hard to pull toward your positive conditioning and away from your negative conditioning at each step. Everything else in the graph exists to feed this node correctly; this node is where the image actually gets made.

**EmptySD3LatentImage — the "why SD3" reasoning is off:** There's no SD3 model running in the background. The name is a historical leftover — this node was originally built for Stable Diffusion 3's latent format, and later models (Flux, Qwen) reuse the same latent format, so ComfyUI reuses the same node instead of making a new one per model. It's really just "make a blank canvas of size W×H" — nothing SD3-specific is happening.

**One genuinely important nuance you haven't hit yet:** in your graph, this "blank canvas" is pure noise — it does **not** contain your input photo. Your product photo only enters the process through the two `TextEncodeQwenImageEditPlus` nodes (via their `image1` + `vae` inputs). This is _why_ `denoise` sits at 1.0 in edit workflows — the sampler is allowed full freedom to denoise from scratch, because your photo is steering it through conditioning, not by being partially blurred into this starting latent. Keep that in your back pocket — it'll make Day 5 click faster.

**VAE Decode** ✅ — right, just note it's decoding a _latent tensor_, not "computer language" broadly — same idea, just the precise term.

Fix those two (CLIP/VAE swap, KSampler's real job) in your own head, and you're genuinely ready for Day 2 — steps, isolated. Same drill: fixed seed, run at 4/10/20/40, tell me what you see.

