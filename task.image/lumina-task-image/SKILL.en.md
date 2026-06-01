---
name: lumina-task-image
description: Supplemental prompting for task.image; used to turn the current turn’s request into an executable image task
---

# task.image

## Applicable Scenarios

- The user wants the current turn’s request organized into an image generation task that can be issued directly.
- The task depends only on the current message text, the assets uploaded in the current message, and any injected project style; it does not read project-maintained character, scene, or prop assets.

## What It Does

- Determine whether the current turn is asking for text-to-image, image editing, multi-image fusion, style transfer, a poster/thumbnail, product image, character reference image, environment establishing shot, or single-prop image.
- Turn the image goal into a complete natural-language prompt covering subject, action/state, environment, composition/shot size, lighting/color tone, materials, style, aspect ratio, and constraints.
- Decide which reference images are used based only on assets uploaded in the current turn, and define the role of each reference image.
- In image-to-image, editing, and multi-image fusion tasks, clearly define what must be preserved and what must change to reduce drift across character, space, material, and style.
- Choose suitable prompting strategies for character consistency, text posters, environment establishing shots, single props, and product presentation.
- For scene images, first frames, last frames, or action-pose images that may later be used for image-to-video, establish clear spatial anchors and occlusion boundaries.

## What It Does Not Do

- Does not read project-maintained character, scene, prop, or historical assets.
- Does not bind assets or convert the current image task into long-term character/scene/prop assets.
- Does not keep a consultative tone or explanatory filler.
- Does not output SD tag strings, weighted parentheses, command-line parameters, or platform-specific private syntax.
- Does not write image tasks as video storyboards or story-to-video workflows.

## Decision Principles

- Modern image models respond better to clear, specific, logical full natural language; do not write prompts as piles of keywords.
- When information is incomplete, prioritize filling in subject, purpose, composition/shot size, style/mood, lighting/color tone, material detail, and aspect ratio.
- Reference images come only from assets uploaded in the current turn; each image must have a stated role, such as subject identity, clothing, scene, composition, pose, prop, material, or style.
- When editing an existing image, prioritize stating “what changes” and “what stays the same”; do not rewrite the task as a completely new text-to-image request.
- Organize multi-image fusion by image role; do not use vague references like “the image above” or “this image.”
- Style terms should modify the visual treatment only and must not override subject identity; core identifying information for characters, products, scenes, or props takes priority over stylistic enhancement.
- If the image will be used as video reference, prioritize clear spatial relationships, subject orientation, entrances/routes, light-dark boundaries, and occlusion relationships rather than only visual atmosphere.
- Common professional photography and lens terms may remain in English, such as Macro lens shot, soft lighting, and rim light, but do not turn them into a stream of English tags.

## Text-to-Image Writing

- Organize as “subject + action/state + environment + composition/shot size + lighting/color tone + material details + style/aspect ratio.”
- The subject must be specific; avoid generic labels like “a woman,” “a car,” or “a room.” Include age range, identity and temperament, clothing, object category, or space type whenever possible.
- Composition must serve the use case: portraits use close-up or half-body framing, posters use key-visual composition, product images use clean commercial presentation, scene images use wide shots or establishing shots, and prop images use a centered single object.
- Lighting and color tone must be actionable, such as warm morning light, cool white overhead light, side backlight, soft light, low saturation, high contrast, or neon reflections on a rainy night.
- Materials must be specific to visible surfaces, such as cool metallic reflections, frosted glass, rough concrete, soft fabric, old paper creases, or worn wood grain.

## Spatial Anchors and Video Reference Images

- Environment establishing shots, first frames, last frames, and action-pose reference images must provide the video model with a stable coordinate system: entrances, exits, walls, floor, table edge, bedside, car door, steps, columns, window light, or light-dark dividing lines must be visible.
- Scene images should preferably be written as establishing shots or medium-wide shots so foreground, midground, and background relationships are clear; do not generate only localized atmosphere shots, blurred backgrounds, or decorative images with unreadable paths.
- For images that need to show “entering / leaving / approaching / moving away,” the composition must preserve cues that indicate depth, such as door frames, corridor perspective, floor texture, furniture occlusion, subject scale in frame, or foreground-background layering.
- Light-dark contrast and warm-cool lighting can serve as spatial boundaries: cool dark zones, warm bright zones, shadow edges inside vs. outside a doorway, and window-light cut lines should be clear to help the video stage lock start and end positions.
- Action-pose reference images should lock only one key state: wind-up, impact, collision result, fall result, or hand evidence; do not generate action collages, process sheets, or multi-stage duplicates.
- Action images involving collision, pushing against a wall, falling, passing through a door, moving around a table, etc. must show opaque boundaries and contact relationships, such as a back against the wall, a palm pushing a door, the body occluded by a table edge, or feet stopping before a threshold; do not let bodies and solid objects intersect.
- First and last frames must belong to the same space, the same subject, and the same continuous action chain. The last frame may change position, pose, lighting, or outcome, but it must not switch space, switch identity, or let object boundaries drift.
- If an image is only for character identity or single-prop reference, do not force in complex spatial anchors; keep the background clean and avoid accidentally binding a one-off scene to the subject identity.

## Image Editing and Multi-Image Fusion

- For image editing, organize as “change action + target of change + changed attributes + preserved elements.”
- Preserved elements must be specific, such as preserving the character’s face shape and age, preserving the original composition, preserving the background space, preserving font color, or preserving product packaging proportions.
- For multi-image fusion, first define reference roles: image 1 locks subject identity, image 2 locks scene or style, image 3 locks pose, composition, prop, or material, then write the final image.
- If multi-image fusion is for video reference, roles must be separated: one image locks character identity, one locks spatial anchors, and one locks props or key pose; the final image must not become a collage or a frame-by-frame process sheet.
- If the user requests style transfer, explicitly state which subject content is preserved, which image provides the style, and which elements must not change.
- When simultaneous editing of multiple objects is high risk, prefer splitting into step-by-step tasks: subject first, then background, then lighting, text, or materials.

## Character Consistency

- For character consistency, prioritize locking face contour, facial feature proportions, eye spacing, nose shape, jawline, age range, hairstyle, skin tone, body proportions, and stable identifying markers.
- Expression changes should modify only visible expression signals, such as the corners of the mouth, brows, eye socket tension, pupil focus, tear traces, or breathing tension; do not change the face, beautify it, or alter age.
- Character reference images should prefer clean backgrounds, a single person, and either full-body or clear close-up views; do not add complex narrative scenes, multi-person interaction, or heavy occluding lighting effects.
- If reference image order is controllable, the clearest face or subject image should serve as the identity anchor; other images should only provide clothing, pose, scene, composition, or style.

## Scenes, Props, Products, and Text

- Scene reference images should default to empty environment shots with no people; specify the space type, layout, entrance/path, key furnishings, materials, main light source, weather/season, and dominant color tone.
- Scene reference images should preserve reusable spatial anchors and scale relationships, such as door frame position, corner direction, table-chair relationships, window-light direction, foreground-midground-background layering, and walkable paths.
- Prop reference images should default to a centered single object; specify silhouette, material, size proportion, texture, wear, markings, and glow effects, without adding hand-held usage or cluttered backgrounds.
- Product images should prioritize clear presentation of brand, packaging, material, and selling points; if the user wants a commercial advertising look, specify the background, key light, reflections, display angle, and use scenario.
- For text posters, put the text to be generated in quotation marks, and specify font style, layout position, color scheme, and purpose; for complex small text, long Chinese passages, and exact data, note that post-edit proofreading or replacement is needed.
- For content the model tends to beautify by default, explicitly state missing elements, such as not brand new, not a clean catalog product shot, no modern decor, no crowd, and no green leaves or fruit.

## Quality Standards

- The task intent is clear, and the prompt can be used directly for generation without extra explanation.
- Subject, composition, lighting, color tone, material, style, and aspect ratio do not conflict.
- Reference image roles are clear, and preserved vs. changed elements are clear.
- Characters, scenes, props, or products do not lose their original identity anchors due to stylistic enhancement.
- Images used for video reference have clear spatial anchors, entrances/routes, light-dark boundaries, occlusion relationships, or a single key pose.
- Text requirements are clearly placed in quotation marks, and exact accuracy of complex small text is not treated as the sole success criterion.

## Failure Fallback

- If uploaded assets are insufficient, provide an executable text-only version first, then point out the gaps.
- If the user only says “make it look better,” convert that into specific changes: clean up the background, increase contrast, strengthen the key light, unify the color tone, emphasize the subject, or improve material definition.
- If the user’s requirements conflict, preserve subject identity and use case first, then narrow style, lighting, and composition.
- If the image will later be used for video but the space is unclear, prioritize changing it to an empty establishing shot or preserving anchors such as door frames, walls, floor paths, and light-dark boundaries.
- If an action image shows mesh intersection or mixed multi-stage motion, change it to a single key pose and explicitly define contact boundaries, occlusion relationships, and no-collage constraints.
- Do not secretly inject project assets or other contextual images.