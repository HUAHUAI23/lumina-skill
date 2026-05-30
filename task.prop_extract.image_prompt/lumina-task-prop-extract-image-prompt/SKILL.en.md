---
name: lumina-task-prop-extract-image-prompt
description: Supplemental prompt guidance for task.prop_extract.image_prompt; used to turn a single prop asset into an executable image task
---

# task.prop_extract.image_prompt

## Applicable Scenarios

- After prop extraction is complete, prepare a directly usable prop reference image task for a single prop.
- Ensure that subsequent storyboarding, image-to-image generation, and video shots can consistently recognize the same object.

## Responsibilities

- Compress the prop name, category, silhouette, material, color, size ratio, and key identifying features into image-generation-ready prompts.
- Generate clean, background-free single-prop reference images suitable for later reuse; by default, one baseline image is enough, and derived images should be added only when the same prop truly has reusable form/state variations.
- Keep the prop’s shape, material, color, texture, wear, glow effects, markings, and proportional relationships stable.
- Distinguish baseline and derived images: the baseline locks the object identity; derived images only change reusable forms or surface states and must not create a new prop.
- Use restrained negative constraints to reduce interference from people, hands, tabletops, rooms, clutter, and scene backgrounds.

## Not Responsible For

- Do not add new props or rewrite a prop identity already confirmed upstream.
- Do not split one prop into multiple entity tasks.
- Do not generate narrative storyboard images or let character actions, hand-held poses, or scene storytelling overpower the subject.
- Do not require the model to generate accurate long Chinese text, tiny document text, or complex readable text.

## Decision Principles

- Use complete natural-language descriptions. Do not write keyword tag streams, weight brackets, command-line parameters, or provider-specific syntax.
- The subject must remain stably recognizable, and the image must center on a single core prop.
- Default to a clean, background-free single-object reference image: centered subject, fully visible, clear edges, no occlusion, pure-color/transparent/minimal neutral background; no tabletop, floor, room, scene lighting context, or hand-held context.
- Restrained light effects may be added for spirits, ritual tools, energy devices, and similar objects, but the light effects must not obscure the main silhouette, material, or markings.
- Wear, mud, bloodstains, charring, cracks, fading, water damage, blurred writing, and long-term handling marks must be treated as core subject features; do not generate a brand-new product image.
- For prop sets, emphasize unified form, quantity relationships, arrangement relationships, and material consistency; do not turn a set into a random pile of miscellaneous items.
- Any later derived image must explicitly state: keep the same prop’s silhouette, material, color, size ratio, and stable markings; only change the reusable form, structural state, or surface state for this version.

## Asset Variant Decision Protocol

- For each prop image task, first determine whether a multi-image asset chain is needed; do not include the reasoning process in the output, only implement the result in `prompt` and `variants`.
- By default, output only one baseline single-prop image; output additional variants only when the evidence is clear and the downstream reuse value is sufficient.
- Different viewing angles of the same prop are not grounds for long-term variants; do not generate multiple images just for “front/back/three-quarter/top-down/on a table.”
- Multiple images must satisfy at least one condition: there are reusable different forms or structural states, such as open/closed, intact/broken, folded/unfolded, inactive/activated, before repair/after repair; or there is damage, dirt, bloodstains, charring, fading, or stage-based state that will remain consistent across shots.
- Do not split the same prop into multiple props because of intact/damaged, clean/dirty, inactive/activated, open/closed, or new/worn differences; these should be handled as variants of the same prop.
- If damage, dirt, bloodstains, activation, opening/closing, or repair is only a one-off presentation in the current shot, or can be handled by the baseline image plus shot-level img2img in story_to_video, do not output it as a long-term variant.
- If a given state is the prop’s default primary state, such as a “worn bloodstained amulet” that remains that way after first appearance, then the baseline image should use that state directly and should not generate an extra clean version.
- Each derived variant must clearly state: “Keep the same prop’s silhouette, structure, material, color, size ratio, and stable markings; only change this version’s form, surface state, activation state, or open/closed state.”
- The first item in `variants` is the text-to-image baseline single-prop image; subsequent variants will by default use the first baseline image as the img2img object reference.
- When doing incremental extraction and a baseline image already exists, newly added state variants should by default use the existing baseline image as the img2img reference; do not output a new baseline single-prop image unless the original baseline image is clearly unusable in quality or identity anchoring.
- Every variant must fill in `assetDescription`, which is a required field in the structured schema: use one sentence to explain the unique prop identity, whether this asset is the baseline single-prop image or which form/state image it is, and what shot decision in story_to_video it is used for.
- `variantKey` is only a technical deduplication identifier and does not use a fixed vocabulary; short snake_case is allowed, but the decision information must be written into `assetDescription`.

## Baseline Single-Prop Image

- Organize the baseline image as “prop category + silhouette/structure + material + color + size ratio + texture/wear + stable markings + clean background + style/aspect ratio.”
- Composition should prioritize showing the full object. Do not crop key edges, do not occlude the subject, and do not add hands, people, scene clutter, or packaging/advertising.
- Use `transparent background`, `plain white background`, or `flat neutral background`; tabletops, floors, rooms, streets, display cases, packaging ads, and any specific scene context are prohibited.
- Express size ratio as relative scale, such as palm-sized, longsword scale, document-paper sized, pendant-sized, or trunk-sized; do not force numeric measurements without evidence.

## Form and State Images

- Add derived images only when the same prop has reusable forms, open/close structures, damage, dirt, or activation states.
- Do not add derived images for ordinary angle differences; the front/back/left/right/three-quarter views of the same prop should by default be handled by the video model using the single-object anchor and text.
- Material close-ups, hand-held views, engraved-detail close-ups, bloodstain close-ups, and similar cases should be handled by story_to_video with shot-level img2img based on the clean prop image, not as long-term images in the asset library.
- Derived images must preserve the baseline image’s subject identity, color, material, and stable markings, and only change form, structural state, or subject state.
- Keep the number of derived images restrained: for ordinary props, 1 baseline image; when truly needed, 1 baseline + 1–2 derived images; more than 3 images requires a clear explanation of why each one cannot be replaced by shot-level derivation or another state image.

## Text Surfaces and Markings

- If the prop contains long text, handwritten last words, documents, talisman paper, plaques, serial numbers, or complex symbols, do not require the model to generate accurate readable text.
- You may describe “blurred handwritten traces, ink marks, engraved marks, symbolic patterns, blank areas, worn paper surfaces” so that the carrier surface remains stable.
- Text that must be read accurately is better handled with post-production texture overlays or subtitle-layer replacement; the focus of the prop reference image is the paper/wood plaque/metal engraving surface, ink placement, and period texture.
- Iconic symbols, ritual patterns, and glowing lines may be described in form and placement, but do not write excessively long, overly dense, or mutually contradictory pattern requirements.

## Negative Constraints

- Standard exclusions: people、person、human、hand、hands、holding、tabletop、floor、room、interior、environment background、scene background、clutter、busy background、multiple objects、cropped、blocked、logo、watermark。
- For worn props, also exclude `pristine`, `brand new`, `glossy`, `plastic`, `clean surface`, and other mistakenly added commercial product textures.
- For prop sets, exclude random clutter, confused quantities, and mixed-in objects of different styles.

## Quality Standards

- It should be immediately obvious what the prop is.
- The silhouette, edges, primary materials, and size ratio must be clear and must not require textual explanation to understand.
- Derived forms or states must still be the same object and must not turn into another prop because of lighting effects, background, or partial presentation.
- Wear, mud, bloodstains, blurred writing, and a sense of age must not be automatically cleaned up or commercialized.
- Prompts must be directly usable for generation, without vague rhetoric or conflicting material requirements.

## Failure Fallback

- When information is insufficient, prioritize preserving the name, category, silhouette, and core material.
- When prop detail is insufficient, generate a plain single-prop reference and do not force complex patterns, brands, mechanisms, or fantasy effects.
- If text surfaces cannot be generated stably, prioritize blank space, blurred pen traces, and the material carrier surface; do not force accurate text.