---
name: lumina-project-bind-style
description: Supplemental prompt for project.bind_style; used to distill user-provided text or image/text style references into project-level style constraints
---

# project.bind_style

## Applicable Scenarios

- The user wants to bind a long-term, stable visual style to the entire project for subsequent character, scene, image generation, and video generation tasks.
- Input may be text only, reference images, reference images with text notes, or additions/replacements to an existing style.
- The user may be defining a style for the first time, or supplementing, correcting, or replacing an existing one.

## What This Handles

- Extract a project visual direction suitable for long-term reuse.
- Establish two sets of constraints at the same time: one for image generation style and one for video generation style.
- Condense medium, color, material, camera, lighting, composition, motion rhythm, and overall mood into style language that can be directly used in prompts.
- Infer style from reference images rather than copying the subject, scene, accidental composition, or narrative action in the image.
- When the project already has a style, merge and refine it rather than overwriting it mechanically.

## What This Does Not Handle

- Do not bind character, scene, or prop identity.
- Do not mistake the specific character, composition, or narrative action in a single reference image for the project’s long-term style.
- Do not create image tasks, video tasks, or other structured workflows.
- Do not expand worldbuilding, plot setup, or character relationships.
- Do not write platform-specific parameters, weight syntax, model names, or tool invocation methods.
- Do not use a specific artist, living creator, film/TV IP, brand visual language, or protected character as a shortcut for project style; rewrite it into executable, generic visual language.

## Decision Principles

- Prioritize abstracting stable style language rather than restating surface content from reference images.
- Text instructions take priority in setting direction; reference images are used to calibrate texture and visual landing points.
- Image generation style emphasizes single-frame image quality, composition, and material expression.
- Video generation style emphasizes cinematography, motion rhythm, temporal atmosphere, and dynamic consistency.
- The style must be reusable over the long term and should not depend excessively on accidental details from any single image.
- Style prompts should be specific but not bloated; prioritize keeping 4-8 anchors that most strongly affect the result.
- Prefer positive descriptions. Do not rely on large numbers of “no / do not / forbidden” statements to define style.

## Core Reverse-Inference Workflow

1. First determine the user’s goal: create a new style, strengthen an existing style, replace an existing style, or simply extract style from reference images.
2. Read the textual intent: subject direction, aesthetic keywords, platform use, and anything the user explicitly wants to keep or exclude.
3. Read the style of the reference images: medium/rendering approach, color system, light-shadow relationships, light sources, camera/shot scale, compositional order, material texture, line and edge quality, detail density, post-processing feel, and emotional atmosphere.
4. Strip out non-style content: specific character identity, objects, locations, narrative actions, pose in a single image, accidental occlusion, text watermarks, logos, and one-off compositions.
5. Synthesize a long-term style: use a small number of high-impact anchors to describe “how all future visuals should look,” not “what is in this image.”
6. Split into two expressions for image generation and video generation: image generation describes static-frame visuals; video generation adds motion, camera behavior, and rhythm on top of the same visual basis.

## Reference Image Reverse-Inference Rules

- A single reference image provides style evidence only; it does not automatically define the project subject. Extract “what it looks like,” not “who is depicted, where it is, or what is happening.”
- For multiple reference images, find the common denominator: only repeatedly appearing color, lighting, material, camera, linework, grain, composition, and mood are suitable for inclusion in the project style.
- If multiple references conflict, prioritize the user’s text instructions; if no clear instruction exists, keep the most stable primary direction and put conflicting points into reference notes.
- If the reference image is character-focused, style extraction should focus on art style, lighting, color, texture, depth of field, and camera; do not turn the face, clothing, pose, or identity into global style.
- If the reference image is scene-focused, style extraction should focus on spatial texture, lighting, color, perspective, materials, and atmosphere; do not turn the specific location or set dressing into global style.
- If the reference image contains obvious watermarks, subtitles, UI screenshots, social media borders, or compression noise, these are usually contamination and should not enter the style.

## Text Description Reverse-Inference Rules

- Translate abstract terms into visible visual language:
  - “premium” -> restrained color, clean composition, low noise, precise materials, soft key light.
  - “cinematic” -> film-still composition, natural skin tones, directional lighting, shallow or layered depth of field, slight film grain.
  - “oppressive” -> low illumination, low saturation, weak warm-cool contrast, heavy shadow coverage, spatial pressure around characters.
  - “xianxia” -> Chinese fantasy visual language, flowing fabrics, volumetric light, layered clouds and mist, jade/metal/silk materials.
  - “short drama feel” -> realistic human proportions, clear facial detail, natural everyday indoor/outdoor lighting, dialogue-friendly close shots.
- If the user provides only style keywords, do not force in complex camera work or worldbuilding; first form a clear, concise, long-term reusable style outline.
- If the user provides highly detailed descriptions, compress them into stable anchors and remove anything that only serves a single image.

## How to Write Image Generation Style

- Image generation style should cover key items among these areas: visual medium, rendering/photography approach, color system, lighting, composition, camera/depth of field, material texture, detail density, and post-processing feel.
- The image generation prompt is suitable as a short phrase or a set of short clauses, for example:
  - `3D Chinese fantasy character and scene style, cool-warm contrast in moonlight and magical volumetric light, clear facial detail and clothing materials, rich layers of jade, metal, silk, and mist, film-still composition, low noise, high consistency.`
- Negative constraints for image generation should include only long-term general risks: low clarity, excessive noise, text watermarks, style confusion, cheap plastic feel, broken faces, etc.
- Do not write project image-generation style as “a certain character stands somewhere, a prop is in the center of the frame, the camera moves from left to right.”

## How to Write Video Generation Style

- Video generation style should inherit the visual anchors of image generation, then add camera and temporal qualities: primary camera movement tendency, motion speed, editing rhythm, motion blur, depth-of-field changes, lighting changes over time, and naturalness and continuity of action.
- Video prompts should lean toward positive visual description and use fewer negative sentences. Change “don’t move around randomly” to “locked camera, restrained subject motion, stable camera”; change “don’t flicker” to “continuous lighting, stable image, natural motion.”
- If the project mainly produces short drama/dialogue content, the video style should emphasize close-up/medium-shot friendliness, facial stability, natural dialogue rhythm, and slight push-ins or locked shots.
- If the project mainly produces fantasy/action content, the video style should emphasize slow forward motion, volumetric light and particles flowing with movement, natural movement in fabric and hair, and camera motion that is not overly complex.
- Do not write a full storyboard timeline into the project style; project style should define only the overall motion temperament and camera preferences.

## Merging Existing Style

- If the user says “switch to / change to / don’t use the old one,” treat it as a replacement; keep from the old style only non-conflicting general quality requirements.
- If the user says “add a bit / make it more like / lean toward,” treat it as a supplement; keep the existing main direction and absorb only the clearest differences from the new input.
- If the user uploads a new reference image without text instructions, treat it by default as a calibration reference, not a full override of the existing style.
- After merging, remove internal conflicts. For example, if the style simultaneously asks for “realistic cinematic photography” and “flat cel-shaded line art,” you must decide the primary style; the other can only remain as a minor trait or be discarded.

## Working Method

- First determine the core style direction the user wants to preserve.
- Then separate what belongs to the project’s long-term style from what is only local content in the reference image.
- For mixed text-image input, combine textual intent with the image mood and converge it into a unified style expression.
- For an existing project style, prioritize continuing the main direction and absorb only the differences the user explicitly adds this time.

## Quality Standards

- A one-sentence style summary should clearly describe the project’s overall visual direction.
- The style expression should be stable and should not drift frequently.
- The style constraints for image generation and video generation should be consistent while each keeps its own emphasis.
- Language should be direct, specific, and executable; avoid stacking vague aesthetic buzzwords.
- The image generation prompt should be directly reusable in character, scene, prop, and storyboard image tasks.
- The video generation prompt should be directly reusable in video tasks and should not turn ordinary shots into complex multi-shot scripts.
- Reference notes should clarify which inputs were treated as style sources and which content was stripped out as non-style.
- The final result should support continued reuse across subsequent character, scene, image, and video tasks, while making different outputs from the same project look like they belong to one visual system.

## Failure Fallback

- When user intent is unclear, prioritize the clearest style signals.
- When reference image information is complex, it is better to converge on a more stable main direction than to collage together conflicting styles.
- Do not force uncertain local details into long-term project constraints.