---
name: japanese-aesthetic-video
description: Turn a user's video idea into Japanese-animation-inspired illustrated images generated with Hy-Image-3.5-preview through GMI Cloud MCP, then animate those images with HyperFrames and render an editable MP4. Use for aesthetic anime-inspired nature films, illustrated worlds, cloudscapes, forest flights, and image-backed motion shorts. Includes required GMI MCP installation and authentication setup.
---

# Japanese Aesthetic Video

## What to do

When the user gives an idea for a video, make the video—not merely a prompt,
storyboard, or collection of images.

**Pipeline:** user idea → art direction → **GMI MCP image generation** → local
image assets → layered/rigged **HyperFrames animation** → reviewed MP4 + source.

Use **Tencent Hunyuan Hy-Image-3.5-preview**, model ID
`hy-image-v3.5-preview`, for the images. Use the **`hyperframes` npm framework**
for animation and rendering. Hy-Image is not a HeyGen model; HyperFrames is a
HeyGen project. Python's `hyperframe` package is unrelated.

**Do not skip image generation and replace it with procedural drawings.** The
generated artwork must visibly form the finished scenes. Code may add rigs,
cutouts, particles, water detail, shadows, or original subject animation.
Do not call video-generation/image-to-video models unless explicitly requested.

If the user is only asking to install/write this skill, stop after setup; do not
spend money making demonstration assets.

## Default delivery

- **1920×1080 landscape · 30 fps · 10 seconds**, unless the user says otherwise.
- A complete editable HyperFrames project, final MP4, asset manifest and preview.
- Usually 2–3 shots for ten seconds. A different idea may work best in one shot.
- No captions, narration, logos or paid music unless requested/approved.
- Portrait references inform the art; they do not change the video aspect ratio.
- New projects go in `projects/<descriptive-slug>/`; preserve existing user work.

Ask only for essential ambiguities, spending authorization or sensitive upload
consent. Otherwise make clear creative choices and proceed. When the user asks
for a finished video, checking and reviewing are followed by rendering.

## 1. REQUIRED: install/connect and use GMI Cloud MCP

**You must use GMI MCP to generate the images.** First inspect the host's tools.
If it is already connected, reuse that connection—do not install it twice.
If absent, register the official remote server in the user's current client,
then have the user complete browser authentication. This is a remote HTTP MCP
connection; there is no local GMI server package to invent or npm-install.

Official endpoint:

```text
https://mcp.gmicloud.ai/mcp
```

### Claude Code

Run in the video workspace for a project-local connection:

```bash
claude mcp add --transport http gmi https://mcp.gmicloud.ai/mcp
claude mcp list
```

Inside Claude Code, use `/mcp`, select `gmi`, and follow the browser sign-in.
Only use `--scope user` if the user wants the connection in every project.

### Codex CLI

```bash
codex mcp add gmi --url https://mcp.gmicloud.ai/mcp
codex mcp login gmi
```

### Cursor

Merge this entry into the existing project `.cursor/mcp.json` (or the user's
`~/.cursor/mcp.json` only if global installation is requested). Preserve all
other servers:

```json
{
  "mcpServers": {
    "gmicloud": {
      "url": "https://mcp.gmicloud.ai/mcp"
    }
  }
}
```

Open Cursor's MCP settings, connect, and complete GMI browser sign-in.

### Other clients

Add the same URL as a **remote HTTP MCP server with OAuth** in the current
client's MCP settings. Use that client's documented schema; do not paste a
Claude/Cursor configuration into a different client without translating it.
If you cannot edit/register it from the current host, give the matching setup
steps and pause for the user to connect. Never pretend registration succeeded.

### Authenticate and verify

- The user needs a GMI Cloud account, organization access and funding for images.
- OAuth/browser sign-in authorizes tools; an inference API key for Sol/Argon is
  **not** the MCP authorization. Do not ask for a key in chat or put it in code.
- Read-only access is insufficient to generate. Submission needs
  `generation:write` and `generation:read`; uploading also needs write access.
  If refused for scope, reconnect with the required access. Do not repeatedly submit.
- Reload/restart the agent's tool connection if the newly added server is not
  exposed. Verify with a **read-only** `search_models` for the image model.
- Discover the client's exact tool names/signatures; prefixes vary. Required
  capabilities: `search_models`, `get_model`, `estimate_generation`,
  `submit_generation`, `get_generation`, and `upload_media` for local references.
- If connection/authentication fails, report the blocker. Do not silently use a
  different provider, direct HTTP, placeholder images, or code drawings instead.

Setup source, verified 2026-10-28:
https://docs.gmicloud.ai/mcp/gmi-mcp-server
Consult this source again if setup behavior or tool signatures have changed.

## 2. REQUIRED: install and read HyperFrames tooling

Verify Node.js 22+, npm and FFmpeg. Install missing prerequisites with the user's
platform/package manager when authorized; do not claim installation from a command
you did not execute. Do not install video-generation models.

Load the `hyperframes`, `hyperframes-cli`, and `media-use` authoring skills.
If unavailable, install the official core set from the workspace:

```bash
npx --yes hyperframes skills update
```

Read the installed skill files; restart the agent session if its skill discovery
requires it. Search the HyperFrames registry before hand-building named effects.

Scaffold a new project without overwriting existing files:

```bash
npx --yes hyperframes init projects/my-film --resolution landscape --non-interactive
cd projects/my-film
npm install
npx hyperframes browser ensure
```

Use the actual chosen slug and resolution. Keep the scaffold's pinned CLI version
and lockfile. Use CLI `--help` for current options. Work within this project for
the remaining production steps.

## 3. Translate the idea into the reference look

The default is an **original Japanese-animation-inspired illustrated world**,
not photorealism, glossy 3D, or generic neon anime key art. Use these art cues:

- Cream/off-white cumulus clouds with thin olive/graphite contours and curved
  contour hatching, rather than airbrushed realistic vapor.
- Pale sage and sun-washed yellow-green meadows, deep emerald/teal forests,
  occasional cobalt-blue water. Preserve a restrained, coherent palette.
- Painted watercolor/gouache-like color variation with fine hand-inked marks:
  directional grass strokes, dense small leaf shapes, restrained water lines.
- Elevated/aerial compositions, large quiet areas, tiny human/vehicle subjects,
  elongated afternoon shadows and a clear, shared light direction.
- Calm, airy, contemplative staging; stylized hand-drawn atmosphere without
  copied characters, logos, signatures or watermarks.

Adapt these cues to **the user's idea**. A train idea becomes a painted train
landscape, not the cyclist/plane/birds story repeated automatically. New explicit
user references or style changes override these defaults. Do not require a named
artist/studio imitation to obtain this look.

Write a short `SPEC.md`: concept, palette, light, camera angle, dimensions,
duration, timed shots, image/layer list, motion intent and intended image spend.
Keep the same art bible across all generated shots.

## 4. Generate images designed to become animation

1. Search GMI's live **multimodal image** catalog for `Hy-Image-3.5`.
2. Call `get_model` on `hy-image-v3.5-preview` before every prepared submission.
   Verify availability and copy parameter names, types and case-sensitive enums.
3. Explicit `size: "1920x1080"` was supported when verified; recheck it live.
   Use the chosen frame's supported dimensions, not an unsupported custom size.
4. Plan background/foreground separation **before** requesting art. For example:
   clean meadow plate, foreground clouds, cyclist/vehicle layer or drawn rig.
   Do not bake in the same moving subject behind an animated duplicate.
5. Generate the first useful plate and inspect it before buying dependent shots.
   Usually begin with 1–3 plates, then only the layers the movement needs.

### Image-prompt template

> Original Japanese-animation-inspired hand-painted illustration of [USER
> IDEA / SHOT]. [VIEWPOINT AND SUBJECT PLACEMENT]. [PALETTE AND SHARED LIGHT].
> Fine olive ink contours, curved cloud hatching, textured gouache-like color,
> delicate grass/leaf marks, quiet atmospheric composition. [SPACE RESERVED FOR
> ANIMATED SUBJECT]. [LAYER REQUIREMENT]. Wide 16:9 framing with detailed margins
> for gentle camera travel. No text, watermark, logo, photorealistic rendering,
> glossy 3D surfaces, or unintended duplicate subjects.

Example background plate:

> An aerial meadow crossed by a winding narrow path, pale sage and yellow-green
> painted grass, giant cream clouds with delicate curved olive hatching, sun
> from upper left and long soft shadows. Leave an open corridor in the lower
> middle for a separately animated cyclist. Landscape background only, no rider
> or bicycle. Original Japanese-animation-inspired ink-and-gouache illustration,
> generous edges, 16:9. No text, logos or watermarks.

A scene still for approval may include the subject, but a production plate for
independent movement should not. Native transparency is **not assumed**. For
cutouts use a documented removal/masking workflow, inspect halos and preserve
originals. Separate occluders and moving parts where practical.

### Reference images

If the user provides accessible references, inspect them and extract palette,
composition and texture. Do not pretend to have access to images from a previous
session. This skill's written art bible is the fallback when references are absent.

For reference-guided generation, verify the current `image` input format. Up to
five reference URLs under 20 MB each were supported when checked; revalidate.
If local inputs must be uploaded: obtain `upload_media`, PUT actual file bytes
to the returned URL using the **exact** returned Content-Type, then pass its
public URL to the model's documented reference field. Do not fabricate the
client's attachment object or treat getting an upload URL as uploading bytes.
Ask before sharing sensitive material; keep signed/private URLs out of manifests.

### Spending and lifecycle

- `estimate_generation` first, with exactly the prompt and parameters to submit.
  An estimate is a price, not schema validation. Inspect the schema separately.
- State image count, size and estimated cost. Estimates are **list prices** until
  an organization discount is known; charged estimate = list × (1 − discount).
- Obtain approval before spending unless this batch is already explicitly
  authorized within the user's budget. Never use the balance as a budget limit.
- Submit only authorized images. Save `request_id`; if pending, retrieve with
  `get_generation` at a reasonable cadence. Do not submit again to check progress.
- Ambiguous timeout? Recover the existing request before any retry. Do not force
  a fresh nonce to bypass deduplication. Report actual failures; no refund claims.
- On success download files to `assets/generated/`, verify decode/dimensions/MIME,
  and **look at the actual images**. Do not animate a queued job or a placeholder.
- Record prompt, model, seed/parameters, request ID, approved estimate, local file
  and sanitized provenance in `assets/manifest.json`. No secrets or signed URLs.

## 5. Animate the generated artwork, not a replacement world

Make the generated imagery the visual foundation. Build scenes from the actual
local files, masks and subject parts. Use `assets/derived/` for cutouts and rig
pieces, retaining their generated originals.

Use small, motivated camera moves and depth-dependent parallax. Foreground clouds
travel differently from background terrain. Water/detail effects and shadows
support the art; they should not cover it with unrelated procedural scenery.

**Natural motion requirements:**

- Cycling: feet on opposing pedals; knees from fixed limb lengths; wheel angle
  from ground distance/radius; hands stay on bars; contact shadows stay grounded.
- Flight: nose follows the path tangent; restrained bank accompanies a turn;
  no arbitrary side-slipping or sinusoidal whole-object rocking.
- Birds: body stays stable; shoulder/wrist wings articulate, tips lag; powered
  stroke differs from recovery; individual phase/cadence and short glide holds.
- Match shadows to one light source. Avoid dark duplicates that read as subjects.

A flat image cannot expose unseen anatomy or truly articulate a wing. Use
reviewed masks/parts, pose sprites or a compatible drawn rig. If additional image
assets are needed, estimate them within the budget. **Never pass off a slow pan
over a still as the cycling/flying action the user requested.**

Do not stretch portrait art to landscape, reveal empty edges, or let a camera
move drag the subject out of frame. Recompose or generate the right aspect ratio.

## 6. HyperFrames production contract

Follow the installed HyperFrames skills; these are the essentials:

- Explicit root width/height/duration. One scene per sub-composition.
- Timed elements have stable IDs, `class="clip"`, `data-start`, `data-duration`.
  Paint order uses z-index, not just track index.
- Each composition registers its own paused GSAP timeline in `window.__timelines`
  after building tweens. Scene host ID, composition ID and timeline key agree.
- Sub-composition styles, markup and scripts belong inside `<template>`.
- Absolute-time motion: no incremental frame accumulation, unseeded randomness,
  wall-clock animation timers, or infinite repeats. Forward/backward seeks agree.
- Animate a clip's child rather than its display/visibility/autoAlpha. Do not
  combine CSS and GSAP transforms on the same element.
- All images/fonts/media local for delivery. Never call GMI from browser render
  code. No remote generated URLs, API keys or auth tokens in the composition.
- For grades/treatments on imagery use the documented HyperFrames media workflow,
  not ad-hoc CSS filters. Exact titles/captions are HTML/SVG, not generated text.
- Audio gets IDs and explicit timing; silent videos use `muted playsinline`.

## 7. Inspect, refine, render, deliver

```bash
npx hyperframes check --json
npx hyperframes snapshot --review
npx hyperframes preview --background
```

Fix errors; review every shot and seam, anatomy, matte edges, contact and motion
paths. Inspect images and decoded frames, not only the tool's success flag.
Record creative calls, deliberate warnings and review limits in `REVIEW.md`.

Verify the actual preview URL responds and hand over its `/#project/<slug>` URL.
When the user requests the finished file, render after review; otherwise seek
preview approval first. Keep the preview alive during review.

```bash
npx hyperframes render --quality delivery --fps 30 --output renders/final.mp4
ffprobe -v error -show_entries stream=width,height,r_frame_rate:format=duration -of json renders/final.mp4
```

Honor the agreed frame rate/dimensions instead of blindly copying defaults.
Confirm nonempty output, dimensions, duration, decoded frames and audio where
present. State what was measured/seen/heard; do not invent playback approval.

Deliver: **final MP4 + editable project + generated assets/manifest + preview**.
Briefly identify the style choices and any limitations. Be truthful: GMI made
the still artwork; HyperFrames animated/rendered it. No video model was used.

## Completion gate

- [ ] GMI MCP installed/connected and authenticated, not merely mentioned.
- [ ] Correct Hy-Image model inspected live; authorized images actually generated.
- [ ] Images match the Japanese-animation art bible and the user's idea.
- [ ] Actual generated images used visibly in the final video.
- [ ] Requested subject movement, not only a still-image slideshow.
- [ ] HyperFrames checks and visual review performed; final dimensions verified.
- [ ] No credentials exposed; deliverables and provenance saved.
