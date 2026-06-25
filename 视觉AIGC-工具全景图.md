# 视觉 AIGC 工具全景图

> 截至 **2026 年 6 月**。下面的**工具名会过期**，但第 0 节的「四把尺子」不会——记住尺子，新工具自己会归位。
> 这是**弹药库地图，不是学习清单**。别想着学完它。

---

## 0. 先装四把尺子（你那个"无序集合"的解药）

你觉得这些东西像一团没顺序的集合，是因为你把**四种不同维度的东西**塞进了同一个袋子。
任何一个工具（包括明天新出的那个），拿这四把尺子各量一下，它就自动落位了：

| 尺子 | 在问什么 | 例子（**同一行的不是同类**！） |
|---|---|---|
| **A 它是哪一类东西** | 底座模型 / 操作界面 / 控制插件 / 加速器 / 放大器 / 托管成品 | Stable Diffusion=底座，ComfyUI=界面，ControlNet=插件，Midjourney=成品 |
| **B 它在管线哪一步动手** | 文生图 / 图生图·编辑 / 图生视频 / 视频转视频 / 放大补帧 / 成片 | 同是"视频"：Wan=生成，LivePortrait=驱动表情，RIFE=补帧 |
| **C 开源本地 还是 闭源托管** | 自己组装(控制力) vs 别人厨房(省心) | Flux/Wan=本地，MJ/Gemini/Seedance=托管 |
| **D 第几代** | 当下主力 / 上一代 / 已淘汰 | Wan2.2=主力，SVD/AnimateDiff=上一代 |

> **A 是最大的那把解药。** 你脑子乱，八成是把「界面 / 底座 / 插件 / 成品」当成了一类东西。它们是**四类**，根本不在一个层面上。

---

## 1. 一张总览：管线（横）× 角色（纵）

**管线**（从左到右，也是你的爬楼方向）：

```
文字 ──▶ 图 ──▶ (控制/编辑图) ──▶ 视频 ──▶ 放大/补帧 ──▶ 成片
 │        │          │              │          │           │
 └────────┴──────────┴──── 每一步背后都站着同样几"层"角色 ────┴───────┘
```

**角色层**（每一步都靠这几层协作完成）：

| 层 | 干什么 | 你扎根的那个 |
|---|---|---|
| 界面层 | 你在哪儿操作 | **ComfyUI** |
| 底座层 | 真正生成的发动机 | Flux / Qwen-Image（图）→ Wan（视频） |
| 控制层 | 让它听话 | ControlNet / IP-Adapter / 编辑模型 |
| 加速·放大层 | 更快 / 更清 | 按需取 |
| 训练层 | 把模型改成"你的" | LoRA |
| 成片层 | 拼成能发的东西 | 剪辑 + 文案 + 配音 |

---

## 2. 先把你用过的三个归位（拿已知锚定未知）

| 你用的 | 归类(A) | 管线位置(B) | 性质(C) | 当前版本(D) | 一句话 |
|---|---|---|---|---|---|
| **Midjourney** | 成品(界面+底座合一) | 文生图(+短视频) | 闭源托管 | **V8.1**（2026-06-11 起默认） | 出图最美、像有审美的人调过；但控制力弱，进不了精确管线 |
| **Gemini 画图** | 成品 | 文生图 + 对话编辑 | 闭源托管 | **Nano Banana Pro** = Gemini 3 Pro Image | 强在图里文字准、对话式改图、世界知识 / 品牌一致性 |
| **Seedance 2.0** | 成品 | 文/图生视频 | 闭源托管 | **2.0**（2.5 约 7 月初公开） | 当前视频天梯第一，音画一次成；即梦 / CapCut 背后就是它 |

> 看出来没——**你用过的三个，全是"闭源托管成品店"**：你在别人的厨房点菜，出餐快、卖相好，但你**碰不到灶台**。
> 你现在要进的 ComfyUI 世界，是**"开源本地自己组装"**：灶台、火候、刀全归你——这就是它一开始让你觉得乱的原因，你从「点菜」换到了「掌勺」。
> 这两边不是替代关系，是分工：**托管 = 快速出活 / 找参照组**，**本地 = 你要练的真功夫（控制力）**。

---

## 3. 分层详表（弹药库的货架）

> 看法：**找到你这一格 → 取一两个 → 其余知道存在就行**。

### 3.1 界面 / 框架层（你在哪操作）

| 工具 | 性质 | 说明 |
|---|---|---|
| **ComfyUI** | 开源 | 节点式，已是事实标准/基础设施，你的主场 |
| A1111 WebUI / Forge·Forge Neo / Fooocus / SwarmUI / InvokeAI | 开源 | 老牌网页式，更"傻瓜"但天花板低 |
| diffusers / SGLang-Diffusion | 开源 | 代码/后端层，写程序时才碰 |
| Midjourney / 即梦·Dreamina / Krea / CapCut | 托管 | 界面=产品本身，没有"搭工作流"概念 |

### 3.2 图像底座模型层（生图发动机）

**开源本地：**

| 模型 | 出品 | 特点 | 代际 |
|---|---|---|---|
| **Flux.1 [dev/schnell] / Flux.2** | Black Forest Labs | 一致性、物理感强，开源标杆 | 主力 |
| **Qwen-Image / -Edit-2511 / -Layered** | 阿里通义 | 中文文字渲染 + 精准编辑 + 分层，**电商友好★** | 主力 |
| Z-Image (Turbo) | | 快、低显存 | 主力(轻量) |
| SD 1.5 / SDXL | Stability | 老地基，LoRA/生态最全，但画质被超越 | 上一代(仍广用) |
| Krea / GLM-Image / Ideogram(开源) / Chroma | 各家 | 各有侧重 | 新晋 |

**闭源托管：**

| 模型 | 出品 | 强在 |
|---|---|---|
| Nano Banana Pro / Nano Banana 2 / Imagen 4 Ultra | Google | 文字渲染、对话编辑、grounding；Imagen=照片级 |
| Midjourney V8.1 | Midjourney | 审美天花板 |
| Seedream 5.0 | 字节 | 即梦图片 |
| DALL·E 3 / Firefly / Recraft / Ideogram / Reve | 各家 | Firefly/Recraft 版权干净、设计向 |

### 3.3 图像控制 + 编辑层（★你的主业核心就在这）

| 工具 | 干什么 |
|---|---|
| **ControlNet**（canny线稿 / depth深度 / openpose骨架 / seg语义 / scribble…） | 锁构图、姿势、结构（Block 3） |
| **IP-Adapter** | 用参考图锁"风格/长相" |
| T2I-Adapter / InstantID | 轻量控制 / 锁人脸 |
| LoRA | 把风格·角色·概念注入底座（既是控制也是训练产物） |
| Inpaint / Outpaint | 局部重绘 / 扩图 |
| **★ 原生编辑模型（新一代，"锁本体改局部"=你电商主业）** | **Qwen-Image-Edit、Flux Kontext/Fill/Redux、Nano Banana Pro edit、Qwen DiffSynth/InstantX 控制补丁** |

> 你最早那份课程大纲的"图生图/局部重绘"教的是 **SD 时代 inpaint**；2026 干你这活的真功夫是上面那行**原生编辑模型**。这一格要重点扎根。

### 3.4 视频底座模型层（图/文 → 视频）

**开源本地：**

| 模型 | 出品 | 特点 | 代际 |
|---|---|---|---|
| **Wan 2.2**（MoE 架构） | 阿里 | 本地开源最强，文/图/音生视频；2.5/2.6 是 API/更新版 | 主力★ |
| HunyuanVideo 1.5 | 腾讯 | | 主力 |
| LTX-2 | Lightricks | 快、视频+音频多模态 | 主力 |
| CogVideoX | 智谱 | | 偏上一代 |
| SVD / AnimateDiff | Stability / 社区 | 2023，短/糊/动画转绘老牌 | **上一代→淘汰** |

**闭源托管：**

| 模型 | 出品 | 备注 |
|---|---|---|
| **Seedance 2.0** | 字节 | 天梯#1，音画一次成（**你在用**） |
| Veo 3 / 3.1 | Google | |
| Sora 2 | OpenAI | |
| Kling 3.0 | 快手 | 运镜控制强 |
| Runway Gen-4.5 / Aleph | Runway | |
| Luma Ray 3.2 / Hailuo(MiniMax) / Pika / Vidu / PixVerse | 各家 | |

### 3.5 运动 / 角色 / 数字人层

| 工具 | 干什么 |
|---|---|
| LivePortrait | 表情/面部动作迁移（课里 47/48 节那种） |
| MimicMotion / Wan Fun Control / VACE | 用参考视频驱动运动（含 ControlNet 式控制） |
| Hallo / SadTalker / AniPortrait / InfiniteTalk / SoulX | 数字人对口型/说话 |

### 3.6 放大 / 补帧 / 修复层

| 工具 | 干什么 |
|---|---|
| SUPIR / StableSR / CCSR / APISR | 图像放大 + 修复 |
| SeedVR2 / FlashVSR / Topaz | 视频超分 + 修复 |
| RIFE / GIMM | 视频补帧（更流畅） |
| ESRGAN / 4x-UltraSharp | 经典放大模型 |

### 3.7 加速 / 量化层（让慢的变快、大的能跑）

| 工具 | 干什么 |
|---|---|
| TensorRT / LCM / Turbo / Hyper-SD | 压缩采样步数提速 |
| Lightning LoRA / Lightx2v(4 步) | 图/视频 4 步极速 |
| GGUF 量化 / Nunchaku / SageAttention | 省显存 / 提速（小显卡救星） |

### 3.8 训练层（把模型改成"你的"）

| 工具 | 干什么 |
|---|---|
| LoRA 训练：kohya_ss / 秋叶·Anima 训练器 / ai-toolkit | 小数据改风格/角色（Block 4） |
| DreamBooth / 全量微调 / Textual Inversion | 更重的训练 |
| 注意 | 训练流程**随底座变**：练 SDXL / Flux / Qwen / Wan 的 LoRA 流程各不相同 |

### 3.9 成片 / 编排层（Block 6 的地盘）

| 环节 | 工具 |
|---|---|
| 剪辑 | 剪映·CapCut / DaVinci Resolve / Premiere |
| 文案·分镜 | GPT / Claude / Gemini |
| 配音·音乐 | TTS(IndexTTS / Fish-Speech / QwenTTS)、Suno / Udio |
| 自动化·编排 | ComfyUI API / fal.ai / Replicate |

---

## 4. 当下主力 vs 上一代 vs 淘汰（2026-06 快照）

> 学**主力**的操作；**上一代**只懂原理就行；**淘汰**的别花时间。

| 管线步 | 当下主力 | 上一代(懂原理即可) | 已淘汰 |
|---|---|---|---|
| 文生图 | Flux.2 / Qwen-Image / MJ V8.1 / Nano Banana Pro | SDXL | SD1.5 / 初代 DALL·E |
| 图编辑(你主业) | Qwen-Edit / Flux Kontext / Nano Banana Pro | SD inpaint | — |
| 图生视频(本地) | Wan2.2 / Hunyuan1.5 / LTX-2 | CogVideoX | SVD / AnimateDiff / Deforum |
| 图生视频(托管) | Seedance2.0 / Veo3.1 / Kling3.0 / Sora2 | Gen-3 / Luma 初代 | — |
| 放大/修复 | SUPIR / SeedVR2 | StableSR / CCSR | 老 ESRGAN 单用 |

---

## 5. 你该站在地图的哪几格

你的**主业**（电商写实 = 锁产品本体、改背景/光）→ 重心在 **3.3 编辑层**：
本地 **Qwen-Image-Edit / Flux Kontext** 是真功夫，**Nano Banana Pro** 当快速对照组。

爬楼站位（其余货架"知道存在、需要再取"，**别学**）：

| 层 | 你扎根的 | 何时 |
|---|---|---|
| 界面 | ComfyUI | 现在（唯一要扎根的） |
| 图底座 | Flux / Qwen-Image | Block 1–2 |
| 控制 | ControlNet + IP-Adapter | Block 3 |
| 编辑 | Qwen-Edit / Flux Kontext | 贯穿始终（主业） |
| 训练 | LoRA | Block 4 |
| 视频 | 到时选当下最新（大概率不是今天这批） | Block 5 |

---

## 6. 怎么用这张表

1. **它是弹药库地图，不是清单。** 90% 的格子你这辈子可能都用不上，正常。
2. **遇到新工具，先过四把尺子**（A 它是哪类→B 管线哪步→C 本地还是托管→D 第几代），它会自己落位，不再是"又一个不知道放哪的名字"。
3. **托管当对照组，本地练真功夫。** 用 MJ/Gemini/Seedance 快速看"天花板长什么样"，回 ComfyUI 练"我能不能控到那个程度"。
4. **这张表半年后必过期**——Wan 会到 2.x、底座会换代。过期的是格子里的名字，**四把尺子不过期**。到时候只换名字，不换脑子。
