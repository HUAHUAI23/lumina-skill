---
name: lumina-asset-prop-from-image
description: Supplemental prompting for asset.prop_from_image; used to bind uploaded prop images into stable prop assets
---

# asset.prop_from_image

## Applicable scenarios

- The user uploads a prop image and wants to turn it into a stable prop asset in the project.

## Responsibilities

- Identify the core prop in the image.
- Provide a natural, stable, and reusable prop name.
- Extract the material, shape, color, age/wear, and distinguishing features.

## Out of scope

- Do not misidentify a character's clothing or the scene background as the main prop.
- Do not elaborate on the prop's origin, usage story, or narrative significance.
- Do not invent supporting items that do not appear in the image.

## Evaluation principles

- Prioritize retaining key props that will affect continuity in later shots.
- Names must be stable and natural; avoid temporary or ad hoc labels.
- Keep descriptions only to content that supports later visual reuse.

## Working method

- Determine the most important prop based on the image's primary-secondary relationship.
- Finalize the name first, then add identifying description.
- When the user explicitly specifies the prop's identity, prioritize the user's wording.

## Quality standards

- The prop name should be reusable long term and should not be renamed frequently due to angle changes.
- The description should be brief and precise, enough to help maintain continuity in later shots.

## Failure fallback

- If a specific model or detail cannot be determined, use a conservative but clear generic name.
- Do not fabricate brands, time periods, or hidden structures.