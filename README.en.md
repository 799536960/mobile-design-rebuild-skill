# Mobile Design Rebuild Skill

Language / 语言: [中文](./README.zh-CN.md) | [English](./README.en.md)

`mobile-design-rebuild` is a Codex skill for rebuilding mobile UI screens from one or more design images. It is intended for high-fidelity rebuilds, redesigns, or refactors of specified iOS, Android, or H5/mobile web pages.

The workflow is spec-first: Codex turns the design images into a reviewable HTML/JSON/Markdown spec packet, waits for user approval, then proceeds with asset generation, implementation, and platform-specific visual verification.

## What It Does

- Supports single-platform or multi-platform rebuilds for iOS, Android, and H5/mobile web.
- Produces an HTML/JSON/Markdown design spec before implementation starts.
- Converts measurements per target platform: iOS pt, Android dp/sp, and H5 CSS px/rem.
- Requires missing images, icons, textures, and decorative assets to be generated as raster assets instead of approximated with SVG, system icons, or code drawing.
- Requires subagent-generated images to be saved into a main-thread-readable shared folder, with candidate paths, `handoff.json`, and a review preview returned to the main thread.
- Supports a main-thread orchestration and subagent execution workflow for image generation, cropping, masking, and scoped implementation tasks.
- Verifies the result with iOS Simulator, Android Emulator, or Browser/Playwright depending on the target platform.

## Repository Structure

```text
mobile-design-rebuild-skill/
  README.md
  README.zh-CN.md
  README.en.md
  LICENSE
  mobile-design-rebuild/
    SKILL.md
    agents/
      openai.yaml
```

## Install

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo 799536960/mobile-design-rebuild-skill \
  --path mobile-design-rebuild
```

Restart Codex after installation so the new skill can be discovered.

## Usage

Invoke it in Codex:

```text
Use $mobile-design-rebuild for these mobile design images and target page. First identify the target platform, create the HTML/JSON/Markdown spec packet for review, and do not generate assets or edit code until I approve the spec.
```

## Validate Locally

```bash
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py ./mobile-design-rebuild
```

## Before Publishing

- Make sure `mobile-design-rebuild/SKILL.md` contains no local paths, customer data, private project names, or secrets.
- Make sure `mobile-design-rebuild/agents/openai.yaml` matches the skill instructions.
- Make sure `quick_validate.py` passes.
