---
name: lumina-asset-scene-from-image
description: Supplemental prompt guidance for asset.scene_from_image; used to bind uploaded scene images into stable scene assets
---

# asset.scene_from_image

## Applicable Scenarios

- The user uploads a scene image and wants to turn the location or space into a stable scene asset within the project.

## Responsibilities

- Determine which stable space or location the image represents.
- Provide a natural, clear, and reusable scene name.
- Extract the space type, structure, key furnishings, color palette, and atmosphere.

## Out of Scope

- Do not mistake a character subject for a scene asset.
- Do not split the same location into multiple scenes because of camera angle changes.
- Do not elaborate story events or spatial settings beyond what is shown in the image.

## Decision Principles

- If the user explicitly specifies the scene assignment, reuse that scene name first.
- Scene naming should be based primarily on the location or stable space name; do not aim for literary phrasing.
- Keep only information that helps with later visual reuse in the description.

## Working Method

- First determine whether this is a stable space suitable for long-term reuse.
- Then identify core anchors such as structure, furnishings, color palette, and atmosphere.
- Do not split out a new scene for minor lighting or angle changes within the same location.

## Quality Standards

- The scene name is stable and can continue to be reused in later turns.
- The description helps the model recreate the sense of space and environmental mood.

## Failure Fallback

- If the specific location name cannot be confirmed, use a conservative spatial name.
- Do not invent map relationships, historical background, or structural details beyond the frame.