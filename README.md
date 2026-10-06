# Prompts behind the FLUX 3 [large] (Black Forest Labs) Physics-IQ Verified entries

Companion to the FLUX 3 [large] entries on the [Physics-IQ Verified leaderboard](https://physics-iq-verified.anates.ai) and in the [Physics-IQ benchmark README](https://github.com/google-deepmind/physics-IQ-benchmark). `descriptions/best_practice/descriptions_base.csv` below refers to that repository.

Every prompt we used to generate our submitted videos, plus the exact instructions that produced them.
None of these prompts, nor the models that wrote them, ever saw ground-truth frames, masks or scores.

| file | used by | written by |
|---|---|---|
| `i2v_frontier_opus55.csv` | i2v entries with frontier-LLM prompts (single generation and best-of-8) | Claude Opus 5.5 |
| `v2v_frontier_opus55.csv` | v2v entries with frontier-LLM prompts (single generation and best-of-8) | Claude Opus 5.5 |
| `i2v_aligned_qwen3vl.csv` | earlier i2v entries (single generation 43.39, best-of-8 46.95) | Qwen3-VL-32B-Instruct |
| `v2v_aligned_qwen3vl.csv` | earlier v2v single-generation entry (53.89) | Qwen3-VL-32B-Instruct |

The earlier v2v best-of-8 entry (58.43) used the unmodified base best-practice descriptions
(`descriptions/best_practice/descriptions_base.csv`).

Columns: `scenario` and `generated_video_name` as in the benchmark, `bpp_description` (the base best-practice
description the model was given), `prompt` (the text prompt actually used for every seed of that video) and, for the
frontier files, `negative_prompt` (the per-scenario negative prompt used as the classifier-free-guidance negative).

## Frontier-LLM prompts (Claude Opus 5.5)

- **Model:** `anthropic/claude-opus-5.5` via the OpenRouter chat-completions API, temperature 0, max 1500 output tokens,
  one call per scenario and input type (198 + 198 calls, about $9.40 in total).
- **Inputs:** only the base best-practice description plus the video model's own conditioning input:
  - i2v: the switch frame, exactly as fed to the model (832x480);
  - v2v: 6 frames (indices 0, 8, 17, 26, 35, 44) of the 45-frame, 24 fps conditioning clip, whose last frame is the switch frame.
- **No tools:** no web search, retrieval, browsing or tool use of any kind; the request contains only the
  images and the instruction below. The instruction does not name the benchmark.
- **Fixed instruction:** written once, before any frontier-prompt video was generated or scored, and never edited.
- **Output:** one JSON object with `prompt` and `negative_prompt`, used unchanged for all seeds.
- **Selection disclosure:** we generated one seed of two variants (prompt only, and prompt + negative prompt) and
  kept prompt + negative prompt for both input types based on that run's Physics-IQ Verified score. Every other setting
  of these entries was fixed before that comparison.

Instruction (`{condition}` and `{description}` filled per scenario; `{last}` is " in the LAST frame" for v2v, empty for i2v):

```text
{condition}

Short description of the demonstration (written as if from its very start):
{description}

Write the text prompt for the video model so that its 5-second video is as physically accurate as
possible, and a negative prompt.

Prompt:
- Start from EXACTLY the state shown{last}: name the relevant objects, materials, positions and contacts, and
  whether each is at rest or already moving (and in which direction). Never describe an action that has
  already happened as upcoming, and never let anything return to an earlier state.
- Reason about the physics (gravity, support and contact, collisions and momentum transfer, friction,
  rolling, elasticity, fluid flow, buoyancy, etc.) and describe what will actually happen next.
- Narrate the next 5 seconds as a timeline with approximate times (for example "In the first second ...",
  "by about 3 s ...", "at 5 s ..."), including where things come to rest.
- Keep the camera completely static, real-time speed, the same lighting and background. Do not add
  people, hands, text or objects that are not visible; if a hand or a device is visible, describe only the
  motion it is clearly performing.
- 120-220 words of plain prose.

Negative prompt: a comma-separated list (at most 50 words) of the specific visual failure modes to avoid
in THIS scene, e.g. objects moving backwards or repeating an action, objects passing through each other,
floating or defying gravity, appearing or disappearing, morphing, liquids moving unnaturally, motion
freezing, plus camera motion, cuts and zoom.

Answer with JSON only: {{"prompt": "...", "negative_prompt": "..."}}
```

`{condition}` for i2v:

```text
The image is the first frame of a real video of a physics demonstration, filmed by a static, locked-off camera. A video model will animate the next 5 seconds from exactly this frame.
```

`{condition}` for v2v:

```text
The images are frames from the END of a real video of a physics demonstration, filmed by a static, locked-off camera, in time order (oldest first, about 0.36 s apart; the last image is the most recent frame). A video model will continue this video from the last frame for 5 seconds.
```

## Start-state-aligned prompts (Qwen3-VL-32B-Instruct), earlier entries

Self-hosted Qwen3-VL-32B-Instruct, temperature 0, max 400 tokens. Same inputs as above: the base description plus the
switch frame (i2v), or 4 frames (indices 8, 20, 32, 44) from the last 1.5 s of the conditioning clip (v2v).

i2v instruction:

```text
The image is the FIRST frame of a video a model must generate. It is a real, static-camera frame from a physics demonstration, and it may be taken partway through the event: objects may already be released, moving, falling, rolling, tipping, flowing or burning.

Original description of the whole event (written as if from its very start):
{description}

Rewrite the description so it starts from EXACTLY the state visible in this frame and describes only what happens AFTER it. Rules:
1. Begin with one sentence stating the current state in the frame: where each relevant object is, and whether it is at rest or already in motion (and in which direction). Use visual cues such as motion blur, open grippers, an object in mid-air, tilted or splashing liquid.
2. Never describe as upcoming an action that the frame shows has already happened (e.g. if the gripper is already open and the object is in mid-air, do not say the gripper releases it; say it continues to fall). Nothing returns to an earlier state.
3. Then describe how the motion continues and ends, consistent with the original description and physics.
4. Keep the original's style sentence about the static locked-off camera and real-time framerate. Do not add objects, people, text or camera motion. 60-130 words.
Output only the rewritten description.
```

v2v instruction:

```text
The four images are frames from the END of a real, static-camera video of a physics demonstration, in time order (oldest first; the last image is the most recent frame, 0.5 s apart). A video model will continue this video from the last frame for 5 seconds.

Original description of the whole event (written as if from its very start):
{description}

Rewrite the description so it starts from EXACTLY the state in the LAST frame and describes only what happens AFTER it. Rules:
1. Begin with one or two sentences on the current state: where each relevant object is, and whether it is at rest or moving -- compare the frames to state the direction and how fast (e.g. falling, rolling left, tipping, flowing, already settled).
2. Never describe as upcoming an action that has already happened in these frames (e.g. if the object was already released, do not say it is released; say it continues to fall). Nothing returns to an earlier state.
3. Then describe how the motion continues over the next 5 seconds and ends, consistent with the original description and physics.
4. Keep the original's style sentence about the static locked-off camera and real-time framerate. Do not add objects, people, text or camera motion. 60-130 words.
Output only the rewritten description.
```
