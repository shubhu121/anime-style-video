# Japanese Aesthetic Video — reusable agent skill

## The skill

**[skills/japanese-aesthetic-video/SKILL.md](skills/japanese-aesthetic-video/SKILL.md)**
is the self-contained skill: YAML metadata, explicit GMI MCP setup commands,
Japanese-animation art direction, image generation, and HyperFrames production.

Install it by copying the whole `japanese-aesthetic-video/` directory into your
coding client's documented skill directory. For a client that supports the
shared Agent Skills location, a project-local layout is:

```text
.agents/skills/japanese-aesthetic-video/SKILL.md
```

Reload skill discovery if needed. If your client uses a different skill path,
use that path; merely saving a Markdown file elsewhere does not register it.
You can also ask any coding agent to read the linked SKILL.md explicitly.

### Invoke it

> Use the japanese-aesthetic-video skill. Make a ten-second landscape video of
> a small red train winding through a green valley beneath huge cream clouds.
> Generate Japanese-animation-inspired artwork using Hy-Image-3.5-preview via
> GMI MCP, then animate those images with HyperFrames. Estimate images before
> spending and deliver the final 1920×1080 MP4.

The skill explicitly requires installing/connecting **GMI Cloud MCP** if missing:
`https://mcp.gmicloud.ai/mcp`, with browser sign-in. It contains verified Claude
Code, Codex CLI and Cursor setup instructions from the
[official guide](https://docs.gmicloud.ai/mcp/gmi-mcp-server).

The default style is defined concretely: cream ink-hatched clouds, sage meadows,
cobalt water, emerald forest canopies, painted texture, aerial framing and calm
natural motion. Agents must generate and **use** the images—not silently replace
them with procedural drawings.

## Supplementary workspace guide

Open this folder in your coding agent and have it read **[AGENTS.md](AGENTS.md)**.
The guide turns a natural-language prompt into generated still assets and an
editable HyperFrames video.

## Stack

| Job | Tool |
| --- | --- |
| Generate image assets | GMI Cloud MCP → Tencent Hunyuan `hy-image-v3.5-preview` |
| Plan/write animation | Your coding agent; GMI Sol/Argon if separately needed |
| Animate, preview, render | HeyGen `hyperframes` npm framework |

The image model was verified in GMI's catalog on 2026-10-28. Its spelling is
**Hy-Image-3.5-preview**, not “HeyGen Image 3.5.” Agents must recheck the live
schema before using it.

## Before you start

- Connect/authenticate the official GMI Cloud MCP integration in your agent's
  client. This folder does **not** install a server or configure credentials.
- Make the HyperFrames authoring skills available to the agent.
- Have Node.js 22+, npm, FFmpeg and a supported browser available for rendering.
- Approve an image-generation budget or explicitly request the required images.

No API key belongs in this folder. The guide deliberately does not guess an
MCP server URL or client-specific configuration.

## Example prompt

> Read AGENTS.md. Make a ten-second, 1920×1080 landscape animated film about a
> tiny cyclist crossing a sunlit meadow beneath cream clouds. Generate original
> background/layer artwork with Hy-Image-3.5-preview through GMI MCP, then animate
> it with HyperFrames. Use sage greens, delicate ink texture and calm camera
> motion. Keep the cyclist's feet on the pedals. No video-generation model,
> narration or captions. Estimate the required images and ask before spending.

New work goes under `projects/<slug>/`. Earlier `quiet-skies` and workflow-demo
projects remain untouched. No images, paid generations or videos were created
when setting up this guide.
