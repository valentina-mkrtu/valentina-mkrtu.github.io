---
layout: post
title: "An export bug needs a reproducible asset, not another screenshot"
date: 2026-10-13
permalink: /2026/10/13/art-export-bug-acceptance-evidence/
description: "A reproducible fixture connects an art export defect to technical checks and visual acceptance."
---

An art export bug often arrives as an image and a short comment: “The outline is wrong in the game.” That is enough to begin a conversation, but it is not enough to reproduce the defect. The exporter, source asset, import settings, camera, and display scale can all change what the reviewer sees.

The useful unit of work is a small reproduction package. It should let someone else produce the same output, observe the same defect, and judge the proposed correction without reconstructing the original conversation. Here is a worked example for a hypothetical reward icon with a dark fringe around its transparent edge.

## Define the fixture before changing the exporter

Use a dedicated test icon rather than a production artwork whose dimensions and layers are still changing. For this example, the source is a 256 by 256 image with a bright shape, transparent corners, and a narrow edge transition. Keep an unmodified copy in the reproduction folder.

Write the expected constraints next to it: the output remains 256 by 256; transparency survives; the importer uses the agreed alpha convention; and the icon occupies the same logical rectangle before and after the fix. If the pipeline trims transparent margins, record the trim rectangle and pivot as additional outputs.

These are example constraints, not a universal asset standard. A different game may need a compressed texture, a packed atlas, or a non-square source. What matters is that the reviewer can tell which properties are intentional.

## Capture the export as an operation

Record the source revision, exporter version, selected preset, output path, and import configuration. Keep the preset with the fixture where possible. Avoid instructions such as “export normally,” because a normal export may mean a different saved setting on another workstation.

An evidence directory could look like this:

```text
reproduction/
  source/reward-icon.png
  settings/export-preset.json
  before/reward-icon.png
  after/reward-icon.png
  evidence/checks.txt
  evidence/before-light.png
  evidence/before-dark.png
  evidence/after-light.png
  evidence/after-dark.png
```

The filenames describe the evidence to collect; they are not claims that these files already exist. The executor should create the captures from the actual tested build and record the viewing conditions in `checks.txt`.

## Assign preparation, execution, and acceptance

The artist prepares the visual reference and identifies the defect. The engineer prepares the reproduction steps and limits the permissible changes. A named executor repeats the export using their own authorized tools and access, then returns the generated files and a focused diff.

The artist judges whether the visual result satisfies the reference. The engineer checks the technical consequences, including dimensions, import compatibility, and unchanged unrelated outputs. The producer can coordinate the handoff and track completion, but should not be asked to infer alpha correctness from a success message.

[Wagglet's discussion of a cross-functional studio stack](https://wagglet.com/blog/ai-native-mobile-game-studio-stack) is useful context for keeping these responsibilities connected. For this task, the important practice is to carry the reproduction and acceptance evidence with the work rather than scattering them across messages.

## Change one variable in the worked example

Suppose the investigation points to a mismatch between the exported alpha convention and the importer. Keep the source image and camera fixed. Change only the relevant export or import setting, then regenerate the output. Do not also resize the image, replace the artwork, and retune the scene lighting.

View both versions over light and dark backgrounds at the same scale. A fringe may disappear on one background while remaining obvious on the other. Include a checkerboard if it helps inspect transparency, but also use the backgrounds the game actually shows.

The capture should include the same UI region, display scale, and build identifier. A crop can make an edge easier to compare, but retain a full-context capture so the reviewer can see whether the repair introduced a layout change.

## Turn acceptance into observable checks

A review checklist for this fixture can be specific:

- The exported width and height match the agreed dimensions.
- Transparent corners remain transparent in the imported asset.
- The outlined edge matches the approved reference on both backgrounds.
- The pivot and logical display rectangle remain unchanged.
- Repeating the export with the recorded preset reproduces the candidate result.
- No unrelated assets or global import settings change in the patch.

“Looks cleaner” is feedback, not an acceptance condition. If the reference permits a tolerance, name it. If the result still needs artistic judgment, name the person who will supply that judgment rather than disguising it as a numerical test.

## Preserve the failed case after acceptance

Keep the fixture and reproduction instructions after the bug is closed. They make the next exporter upgrade easier to evaluate. A future change can rerun the same case before it reaches a production asset pack.

The fixture does have limits. It does not prove that every texture format, atlas layout, or mobile device behaves correctly. A passing result establishes that the named defect is resolved under the recorded conditions. Additional representative cases are needed before making a broader compatibility claim.

The result of the task is therefore more than a corrected image. It is a small, reviewable record of what was wrong, what changed, and why the people responsible accepted it.
