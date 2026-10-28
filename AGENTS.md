# Prompt → GMI images → HyperFrames video

## Primary skill

For a user video idea, **first read and follow
`skills/japanese-aesthetic-video/SKILL.md`**. It is the primary reusable workflow,
including required GMI MCP installation/connection commands and the default
Japanese-animation-inspired art direction. The sections below are supplementary
technical reference. If they differ, the skill and the user's current request
take precedence. Do not merely produce a procedural drawing or a prompt pack:
generate image assets through GMI MCP, use them in HyperFrames, and deliver video.

## Mission

Turn the user's prompt into an art-directed, editable video. Use **GMI Cloud
MCP** to generate still-image assets with **Tencent Hunyuan Hy-Image-3.5-preview**,
then animate and render them with **HeyGen HyperFrames**.

These are separate products:

- Image model: `hy-image-v3.5-preview`, served by GMI Cloud.
- Video framework: npm package `hyperframes`.
- Python package `hyperframe` is unrelated. Do not install it for video work.

Do not call a video-generation or image-to-video model unless the user explicitly
changes this requirement. Generated images plus code-driven animation are the
intended workflow. Do not describe the result as “no generative AI”; it uses
image generation, but not a video-generation model.

## Defaults and scope

- **1920 × 1080 landscape, 30 fps, 10 seconds** unless the user specifies otherwise.
  Do not infer portrait output merely because reference images are portrait.
- Preserve requested subject, visual style, copy, references and aspect ratio.
- Use the smallest useful asset set, usually 1–3 shots for a ten-second video.
  Do not silently create batches or paid variants.
- No narration, captions, logos or paid audio unless requested or approved.
- Existing videos, article projects and unrelated user work are not your output
  directory. Create a named project under `projects/<slug>/` for each new film.
- This file is a workflow, not evidence that credentials or MCP are configured.

## 1. Connect to the actual GMI MCP tools

1. Inspect the current coding host's available tools and discover **GMI Cloud**.
   Tool prefixes differ by client; find tools by their documented capabilities.
2. Use the authenticated GMI tools provided by the host. Do not copy credentials
   from another project, print API keys, or embed keys in compositions.
3. Verify the connection with a read-only model search. Expected capabilities:
   `search_models`, `get_model`, `estimate_generation`, `submit_generation`,
   `get_generation`, and `upload_media` when references need uploading.
4. If GMI tools are missing, stop before generation. Ask the user to connect and
   authenticate the official GMI Cloud MCP integration in their coding client.
   Consult current official setup docs if needed. **Do not invent an MCP URL,
   transport, npm server package, OAuth flow or client configuration.**
5. If the host uses a tool-discovery/code-mode interface, discover exact tool
   signatures there before calling them. Do not assume another client's syntax.

Do not fall back to another image provider or direct HTTP silently. A text-model
connection to GMI does not imply that the media MCP is installed or authenticated.

## 2. Plan the film before spending

Translate the prompt into `SPEC.md` with:

- Output dimensions, frame rate, duration and delivery filename.
- One-sentence concept and a short shot table with start/end times.
- Palette, lighting, texture, lens/view angle, subject placement and safe areas.
- Asset list: background plates, foreground objects, subjects and any cutouts.
- Motion per shot: what actually moves, where it travels, and how shots connect.
- Intended image count and spending authorization/budget.

Make practical assumptions rather than asking for every aesthetic detail. Ask
only for missing essentials, a material ambiguity, sensitive-reference upload
consent, or spending approval. If this is just a setup/documentation request,
write the guide and stop—do not generate media as a “test.”

If separate text inference is needed, prefer the user's selected GMI model:
`openai/gpt-6.1-sol` or `google/gemini-4-argon`, after verifying it in the live
catalog. A coding agent already running on a suitable model does not need an
extra billed text request just to plan. The image-generation MCP tool is not a
Chat Completions API; do not submit an LLM to it.

## 3. Discover and inspect the image model

Search the live multimodal image catalog for `Hy-Image-3.5`, then inspect the
matching model with `get_model` before preparing a request.

The following was verified on **2026-10-28** and is a starting point, not a
substitute for runtime discovery:

| Field | Verified value |
| --- | --- |
| Display name | Hy-Image-3.5-preview |
| Provider | Tencent |
| Model ID | `hy-image-v3.5-preview` |
| Output | image |
| `size` | Includes `1920x1080`, `1080x1920`, and other enumerated dimensions |
| `image` | Optional reference image URLs, up to 5, each under 20 MB and accessible without authentication |
| `seed` | Integer, 0 means random; use a recorded nonzero seed when appropriate |
| `generate_max_pixels` | Auto-size pixel budget; use the returned enum values/types exactly if needed |

Prefer explicit `size` matching the video's frame. Do not send unsupported
`transparent_background`, motion, strength, duration, negative-prompt or batch
parameters. Use only the live model's schema. A listed preview model can change
or become unavailable; explain the blocker and ask before substituting it.

## 4. Prompt images for animation, not just beauty

Write the prompt for each required layer or shot. Include subject, composition,
palette, light direction, line/paint texture, camera angle and exclusions in the
text prompt when there is no supported dedicated exclusion parameter.

For example:

> Wide aerial illustrated meadow, 16:9 composition, soft sage greens, warm
> cream clouds, delicate olive ink hatching, afternoon light from upper left.
> Open grass corridor through the middle for a separately animated cyclist.
> Background plate only: no cyclist, no text, no watermark. Maintain generous
> edge detail for a small camera move. Match the approved reference palette.

- Keep a stable art bible across prompts. Reuse an approved reference when the
  model supports it and the user permits upload. Seeds alone do not guarantee
  character consistency across different prompts.
- For moving characters, prefer clean background plates plus separate subjects.
  Do not leave a baked-in cyclist/plane/bird beneath the animated one.
- Native alpha output is not promised. If cutouts are needed, use a documented
  background-removal workflow or reviewed masks; inspect matte edges and halos.
- A flattened painting supports camera motion, not independent wingbeats. For
  articulated subjects, create separate parts or a drawn rig, or use approved
  pose sprites. Do not rubber-warp the entire image and call it natural motion.
- Generated typography is optional reference material, not the source of truth.
  Render exact requested text as HTML/SVG with local fonts.

## 5. Estimate, authorize, submit, retrieve

Use the actual MCP signatures. This is an illustrative tool input, **not a shell
command** or a request to execute automatically:

```json
{
  "model_id": "hy-image-v3.5-preview",
  "prompt": "The approved image prompt goes here.",
  "parameters": {
    "size": "1920x1080",
    "seed": 17421
  }
}
```

1. Call `estimate_generation` with the same prompt and parameters you intend
   to submit. Read any size-dependent pricing notes.
2. Report its dollar estimate as **list price** until the organization discount
   is known from `get_model`. Charged estimate = list estimate × (1 − discount).
   Do not hardcode a price from this document or treat a raw internal amount as USD.
3. If the user explicitly requested these generated images or authorized an
   image-backed video, proceed within that scope and any stated budget. If
   image generation is your proposed paid addition, obtain approval first.
   Rejected variants, extra layers and higher-resolution upgrades need budget
   coverage; an estimate is not permission to spend.
4. Call `submit_generation` once. Save its `request_id` immediately.
5. If pending, call `get_generation` using that ID at a reasonable cadence,
   respecting tool guidance/rate limits. Do not busy-loop or submit again to
   check progress. Wait for success before using media.
6. On ambiguous submission/timeout, recover the existing request before retrying;
   do not force a new nonce to bypass deduplication. On failure, report the real
   reason/support reference without asserting that it was free or refunded.
7. Download successful output to `assets/generated/` with its real file extension.
   Verify MIME, nonzero size, decode success and dimensions. Inspect the image.
8. Never claim success based only on a request ID, poster, or queued status.

### Reference uploads

Use only references the user is entitled to share. Explain external upload if
the input is sensitive/private or authorization is unclear. For a local file:

1. `upload_media` obtains a signed upload URL and exact `content_type`.
2. PUT the actual bytes to that URL with **exactly** that Content-Type.
3. After successful PUT, pass `public_url` to the model's verified `image`
   reference parameter in the shape its current docs require. The generic
   `type: image` label alone is not enough to infer string vs array serialization;
   consult `search_docs` if the tool schema leaves it unclear.

Merely calling `upload_media` does not upload anything. Do not fabricate its
client-supplied `file` object. Keep signed upload URLs and credentials out of
source control, logs, public manifests and rendered screens.

## 6. Save provenance and inspect before animating

Keep `assets/manifest.json` recording each asset's local path, model ID, prompt,
supported parameters, seed, request ID, returned MIME/dimensions, reference
provenance (sanitized), and approved estimate. Distinguish downloaded originals
from derivatives/cutouts. Do not save secrets, private reference URLs or signed URLs.

Review the first useful asset before generating dependent layers. Check style
consistency, anatomy, unwanted text/subjects, framing and cutout quality. Reuse
approved assets; don't regenerate everything for a timing change. Explain a
generation failure instead of silently drawing substitutes and claiming they
were generated by GMI.

## 7. Build the HyperFrames project

Load the installed `hyperframes`, `hyperframes-cli` and `media-use` skills.
Search the HyperFrames registry before hand-building a named effect. If skills
are absent, read the current official instructions; install only what's needed.

Requirements: Node.js 22+, npm, FFmpeg and a supported Chromium browser. Use
the CLI's `--help` for current options, pin the project's working CLI version,
and preserve its lockfile.

```bash
npx hyperframes init projects/my-film --resolution landscape --non-interactive
cd projects/my-film
npm install
npx hyperframes browser ensure
```

`init` may install missing core skills; use its documented `--skip-skills` option
only if you already have and have read them. Do not overwrite an existing project.

Project layout:

```text
projects/<slug>/
  SPEC.md
  index.html
  compositions/             # one scene per sub-composition
  assets/generated/        # verified GMI stills
  assets/derived/          # cutouts/masks/layer parts
  assets/manifest.json
  REVIEW.md
  renders/
```

Render contract:

- Root explicitly declares `data-width`, `data-height`, `data-duration`.
- Every timed element has a stable `id`, `class="clip"`, `data-start` and
  `data-duration`. Track index is an editor lane, not paint order; use z-index.
- Each composition registers one paused GSAP timeline on
  `window.__timelines[compositionId]`, after its tweens are built.
- Sub-composition styles, markup and scripts belong inside `<template>`;
  scene host ID, composition ID and timeline key agree.
- Motion derives from absolute time. No wall-clock timers, unseeded randomness,
  incremental per-frame accumulation or endless loops.
- Use local `assets/...` references in compositions. No generation requests,
  remote image dependencies or API keys inside the render.
- Do not tween display/visibility/autoAlpha of a clip; animate its inner child.
  Do not combine GSAP transforms and CSS transforms on the same element.
- Local fonts for non-bundled families. Audio has a stable ID and explicit
  timing; videos are muted unless intentionally keeping their sound.
- Footage grades/treatments use the documented HyperFrames media workflow,
  not improvised CSS filters. Keep original assets intact.

### Make images move believably

Use background/foreground parallax, subtle camera travel, motivated occlusion,
and deliberate shot timing. Do not stretch a still to change aspect ratio or
pan so far that image edges become visible. Fill missing frame area intentionally.

If the user wants character animation, provide character animation:

- Wheels rotate from ground distance; feet stay on opposing pedals; solve limbs.
- Planes point along path tangents and bank with turns, not random oscillation.
- Bird wings articulate at shoulder/wrist with delayed tips, distinct phases,
  powered strokes and short glides. Keep body scale/anatomy stable.
- Shadows follow the subject, ground plane and shared light direction.

Do not substitute a slow zoom on a flat still for a requested cycling/flying
action. If generated art cannot support the action, propose a layered asset or
rig approach within the budget, or explicitly describe the limitation.

## 8. Check, review, preview, deliver

```bash
npx hyperframes check --json
npx hyperframes snapshot --review
npx hyperframes preview --background
```

Fix errors and inspect every shot, cut, key pose, subject contact and frame edge.
Explain deliberate warnings in `REVIEW.md`. A passing layout check is not an
artistic review. Verify backward seeks when animation is procedural.

Open the preview and verify its HTTP endpoint before handing over its actual
`/#project/<slug>` URL. Keep it alive during review. If the user requested the
finished video, render after checking/reviewing; otherwise get preview approval.

```bash
npx hyperframes render --quality delivery --fps 30 --output renders/final.mp4
ffprobe -v error -show_entries stream=width,height,r_frame_rate:format=duration -of json renders/final.mp4
```

Use the agreed fps rather than blindly copying 30 if the user changed it. Verify
file size, dimensions, duration, decoded frames and audio where applicable.
State which checks were actually performed; do not claim to have heard or watched
normal-speed playback if you only inspected stills/signal measurements.

Deliver the MP4, editable source, asset manifest, preview URL and a concise
summary of creative choices. Keep a truthful separation between generated
images, hand-authored assets, text-model code, and code-driven animation.

## Done means

- [ ] User's dimensions, duration and prompt are respected.
- [ ] GMI MCP verified; exact Hy-Image model/schema checked live.
- [ ] Paid calls authorized; estimates, request IDs and assets tracked.
- [ ] Generated stills downloaded locally and visually inspected.
- [ ] Layers/rigs support the requested movement; no baked-in duplicate subjects.
- [ ] No video-generation model used unless separately authorized.
- [ ] HyperFrames checks and visual review pass; output dimensions verified.
- [ ] No credentials or signed URLs in source, artifacts or video.
- [ ] Source, MP4 and clear production/provenance notes delivered.
