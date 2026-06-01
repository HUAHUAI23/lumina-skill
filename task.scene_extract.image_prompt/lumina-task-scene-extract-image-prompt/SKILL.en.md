---
name: lumina-task-scene-extract-image-prompt
description: Supplemental prompting for task.scene_extract.image_prompt; used to turn a single scene asset into an executable image task
---

# task.scene_extract.image_prompt

## Applicable scenarios

- After scene extraction is complete, prepare a directly usable scene reference image task for a single scene.
- Keep the same space reusable across downstream storyboards, image-to-image generation, and video shots.

## What this handles

- Compress the scene name, spatial attributes, layout structure, key elements, lighting palette, and current state into an image-ready prompt.
- Generate a scene reference image suitable for downstream space locking via image-to-image, rather than chasing a one-off mood image only.
- Default to an empty establishing shot; unless upstream explicitly requires people, do not include people, crowds, backs, silhouettes, faces, or animals.
- Distinguish the base image from derived variants: the base image establishes space identity, while derived variants change only reusable form/state, time of day, weather/season, or the spatial viewing axis, without inventing a new location.
- Use restrained negative constraints to reduce human contamination, incorrect beautification, unwanted modernization, and irrelevant decoration.

## What this does not handle

- Do not add new locations or rewrite a scene identity that upstream has already confirmed.
- Do not split one scene into multiple entity tasks.
- Do not generate storyboard frames, and do not treat character actions, dialogue outcomes, or voiceover emotion as the scene subject.
- Do not rely on precise small text, long sign text, or complex wall text as the sole identifying signal of the space.

## Decision principles

- Use complete natural-language descriptions; do not write keyword tag streams, weight parentheses, command-line parameters, or provider-specific private syntax.
- Scene asset images should primarily serve space locking: clear layout, clear scale relationships, clear key elements, and stable materials and main light source.
- Default to an empty establishing shot; do not include people, crowds, backs, human silhouettes, portraits, animals, or subjects unrelated to the scene identity.
- Emotion may only be translated into spatial atmosphere and lighting, such as cold white light, warm dim bedroom light, golden light, lightning, shadows, emptiness, oppression, dust, dampness, or decay.
- If a derived variant is used, explicitly state: keep the same space identity, structure, materials, primary palette, and key elements, and change only the form/state, time/weather/season, or reusable spatial viewpoint for this variant.
- Aspect ratio and composition should serve the purpose: space establishment should prefer horizontal wide frames or wide shots; entrances and paths should use medium-long shots; only anchors such as materials, walls, under-bed areas, or formation textures should use close-up or Macro lens shot.
- Unconventional scenes must state both the current state and the absence of default features, so the model does not turn an orchard into a harvest, a ward into a bright clean room, an old house into a cozy tidy interior, or ruins into a new building.

## Asset variant decision protocol

- For each scene image task, first determine whether a multi-image asset chain is needed; do not output the reasoning process, only reflect the result in the prompt and variants.
- By default, output only one clean empty base establishing shot; add more variants only when there is clear evidence and enough downstream reuse value.
- Multiple images must satisfy at least one condition: there are different time states that will be reused repeatedly, such as default afternoon versus lit-at-night; weather/season/damage/restoration/abandonment/revival/contamination/purification/formation activated states persist across shots; or the space is complex enough that an extra entrance, path, depth axis, reverse angle, or upper-lower-level view is needed for stable storyboarding.
- Do not split the same space into multiple scenes because of day/night, breakfast/evening, sun/rain/snow, season, or slight state changes; these should become variants of the same scene.
- If time, weather, season, state stage, or spatial viewpoint is only a one-off mood for the current shot, or can be handled by the base empty shot plus shot-level img2img in story_to_video, do not output long-term variants.
- If a time or weather state is already the default primary state of the scene, such as a convenience store that only appears at night with lights on, use that night state directly as the base establishing shot and do not generate an extra daytime version.
- Each derived variant must explicitly state: keep the same space identity, layout structure, era, materials, key elements, and spatial proportions; change only the form, time/weather/season/state, or reusable spatial viewpoint for this variant.
- The first variant is the text-to-image base establishing shot; later variants should by default use the first base image as the image-to-image space reference.
- When incrementally extracting from an existing base establishing shot, new state variants should by default use the existing base image as the image-to-image space reference; do not output a new base establishing shot unless the original base image quality or space anchors are clearly unusable.
- Every variant must fill `assetDescription`, which is required by the structured schema: use one sentence to state the unique scene identity, whether this asset is the base establishing shot or a state/view variant, and what kind of story_to_video shot decision it supports.
- `variantKey` is only a technical deduplication identifier and does not use a fixed vocabulary; a short snake_case key is fine, but the decision information must be written in `assetDescription`.

## Base establishing shot

- Organize the base image as: space type + use/state + layout structure + scale relationship + key furnishings/natural elements + light direction + weather/season + material texture + color range + style/aspect ratio.
- Prefer compositions such as wide shot, establishing shot, or documentary still that clearly show spatial relationships.
- The image must make the space legible: where the entrance is, where the primary area is, how key objects are distributed, and how foreground, midground, and background are arranged.
- If the scene's meaning comes from scale pressure or the feeling of human smallness, the base image should prioritize the complete spatial relationship rather than a local mood fragment.
- For indoor spaces, specify doors and windows, bed/table/cabinet/wall, corridor, occlusion, floor, and the main light source; for outdoor spaces, specify terrain, path, buildings/natural elements, sky/weather, and near-far depth layers.

## Derived form, time, and spatial-view images

- Add derived images only when the same scene truly needs form changes, time changes, weather/season changes, or reusable spatial viewpoints such as entrance/path/depth/reverse angle.
- A derived spatial viewpoint is used to supplement a reusable storyboard axis for a complex space and must preserve the same space identity without changing structure, era, materials, or key elements.
- Local close-ups such as under-bed areas, wall surfaces, plaques, formation textures, or material wear should be handled by story_to_video using scene anchor images for shot-level img2img, not stored as long-term asset-library images.
- A derived viewpoint must still make it obvious that this is the same space and must not crop so tightly that it becomes unrecognizable.
- Keep the number of derived images restrained: for a normal scene, 1 base establishing shot; when truly needed, 1 base image plus 1-2 derived images; more than 3 images must be justifiable because each one cannot be replaced by shot-level derivation or another same-state image.

## Text surfaces and default beautification

- If the scene includes plaques, signs, document walls, wall slogans, or formation text, then unless upstream explicitly requires accurate wording, only describe blank space, blurred writing, rough markings, or symbolic texture.
- Precise small text is better handled in post-production; the priority of the scene reference image is the placement surface, materials, lighting, and spatial relationships.
- Unconventional states must use exclusionary descriptions, such as completely no leaves or fruit, no traces of new renovation, no crowd, no tidy street, no modern light fixtures, and no warm soft decoration.

## Negative constraints

- Common exclusions: people, person, human, character, portrait, face, figure, crowd, group, silhouette, animal, logo, watermark, caption.
- For unconventional scenes, add state-specific exclusions, such as lush greenery, ripe fruit, clean modern interior, pristine building, crowded street, or warm cozy decoration.
- Indoor scenes should exclude unrelated people, passersby, pets, and cluttered ads; outdoor scenes should exclude modern vehicles, signs, or street elements inconsistent with the era or region.

## Quality standards

- The scene type and space identity should be obvious at a glance.
- The layout, scale, key elements, primary materials, and main light source should be clear enough to provide stable spatial relationships for downstream storyboards.
- The result should be suitable to preserve as a scene reference image rather than a close-up fragment or a pure mood color block.
- Unconventional states must stay locked and should not be beautified by default because of the scene name.
- No people, no crowds, no animal contamination, and the space identity must not be overridden by local close-ups or decorative styling.

## Failure fallback

- When information is insufficient, prioritize preserving the space type, core layout, and primary lighting rather than inventing specific architectural structure.
- If the text provides only abstract emotion, rewrite it as restrained lighting, color, material, and the space's emptiness or crowding level.
- If it is unclear whether derived images are needed, default to one base empty establishing shot to avoid meaningless multi-image output.
