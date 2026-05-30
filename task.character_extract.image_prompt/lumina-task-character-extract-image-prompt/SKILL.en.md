---
name: lumina-task-character-extract-image-prompt
description: Supplementary prompt guidance for task.character_extract.image_prompt; used to organize a single character asset into an executable image task
---

# task.character_extract.image_prompt

## Applicable scenarios

- After character extraction is complete, organize an executable character reference image task for a single character.
- Ensure that subsequent short drama, storyboard, image selection, image-to-image, and video reference images can consistently reuse the same character identity, clothing, and state.

## What this is responsible for

- Consolidate the character name, identity, base appearance, and key states into character asset prompts that can be used directly for image generation.
- Prioritize clean front/side/back turnaround baseline images for protagonists, key supporting characters, and characters with sufficiently complete appearance information.
- Based on clothing, age stages, stable states, and special forms recorded upstream, plan any necessary state turnarounds or primary-view assets; local close-ups of expressions, eyes, lips, hands, and wounds are not maintained long-term in the asset library.
- Separate identity-invariant elements from state-variable elements to avoid recreating a new face when changing clothing, expressions, or lighting.
- Control prompt density: focus on identity anchors, base clothing, state differences, composition, and negative constraints; do not pile on minor textures or complex plot.

## What this is not responsible for

- Do not add new characters or rewrite character identities already confirmed upstream.
- Do not split one character into multiple entity tasks.
- Do not generate plot posters or storyboard frames, and do not paint full specific scenes such as living rooms, streets, battlefields, or palaces into character asset images.
- Do not invent clothing or state variants out of thin air; without a baseline identity, do not force complex image-to-image changes.

## Core methodology

- Use complete natural-language descriptions. Do not write keyword tag streams, weight parentheses, command-line parameters, or provider-specific private syntax.
- Character asset images must primarily serve reuse: single character, clean background, clear silhouette, clear clothing structure, stable face.
- Baseline images lock identity; state assets only change clothing, state, or form; age-stage assets only change age maturity and stage-specific body posture. Face-family similarity, facial feature proportions, eye spacing, nose shape, jawline trend, hairstyle cues, and stable markers must remain consistent.
- Do not default to beautifying, making younger, slimming, changing hairstyle, changing skin tone, or changing species; sickness, poverty, battle damage, signs of labor, and non-typical appearance described in the text must be preserved.
- Character reference images do not carry full narrative responsibility. Minimal backgrounds and a small amount of lighting may suggest mood, but do not add extra characters, plot interaction, complex prop stacking, or environmental storytelling.

## Asset variant decision protocol

- For each character image task, first determine whether a multi-image asset chain is needed; do not output the reasoning process, only reflect the result in prompt and variants.
- For protagonists, key supporting characters, recurring characters, or characters with sufficiently detailed appearance information, output variants. The first image is an identity turnaround or primary-view asset; if there is no basis for stable state variants, output only this one.
- Only continue outputting corresponding state turnaround/primary-view variants if the character attributes or description include reusable clothing, states, seasonal outfits, professional/occasion wear, injury or frailty, special forms, or age stages.
- If there is no clear basis for variants, do not invent pajamas, battle outfits, winter wear, wounds, or special forms just to enrich the image.
- Each derived variant must separate identity-invariant elements from the changes for this specific variant, and must explicitly state: “keep the same character identity, only change the clothing/state/form/age stage for this variant.”
- The first item in `variants` is the text-to-image baseline image; subsequent variants default to using the first baseline image as the identity reference for image-to-image.
- When incrementally extracting from an existing baseline image, newly added clothing, state, special-form, or age-stage variants should by default use the existing character baseline image / closest stage image as the identity reference for image-to-image; do not output a new base identity image unless the original baseline image is clearly unusable in quality or identity anchors.
- Every variant must fill in `assetDescription`, which is a required field in the structured schema: in one sentence, state who the unique identity is, whether this asset is the base identity or which state turnaround it is, and what shot decision in story_to_video it is used for.
- `variantKey` is only a technical deduplication identifier and does not use a fixed vocabulary; short snake_case is acceptable, but the decision information must be written into `assetDescription`.

## Baseline turnarounds

- For protagonists, key supporting characters, recurring characters, or characters with sufficiently detailed appearance information, prioritize front/side/back turnaround baseline images.
- A turnaround must clearly specify: the same character, front/side/back views shown together, full body, neutral pose, consistent proportions, clear base clothing, clear hairstyle silhouette, clean light-colored background.
- A turnaround is not three different people in one frame. The prompt must clearly state that these are multiple views of the same character, not three different characters.
- A turnaround only locks identity, silhouette, base clothing, and hairstyle. It does not carry plot states such as battle, injury, crying, corruption, or confession.
- Base clothing should be the outfit most frequently reused or most representative of the identity; do not cram all clothing variants into one image.

## Clothing and state variants

- Clothing/state variants are still character reference images, preferably single-character full-body or mid-to-full-body, with a simple background.
- Every variant must clearly state: “keep the same character identity, only change the clothing, state, or age stage for this variant.”
- Home state, outdoor state, battle state, injured state, frail/sick state, formal state, disguised state, and special form should be carried separately; do not mix them in one image.
- Only change what this variant needs to change. For example, a battle state may change battle robes, armor, worn weapons, and battle damage; do not also change age, face shape, hair color, height, or species. An age-stage image should only change age maturity, height proportions, facial youthfulness/maturity, stage-specific clothing, and body posture.
- Injury, frailty, labor, poverty, and battle damage must be written as visible surface details: pale complexion, bandages, dried blood, mud stains, rough palms, torn sleeves, charred armor plates.
- If the state is counter to the default expectation, specify missing elements, such as: no healthy rosy complexion, no clean new clothes, no modern fashionable styling, no smooth uninjured skin.

## Age-stage variants

- Childhood, adolescence, youth, adulthood, old age, and similar cases are age-stage variants of the same character, not new character tasks.
- For example, “childhood Shen Shen” and “young adult Shen Shen” should be output as different age-stage images within the same Shen Shen image task; `assetDescription` must clearly state “Shen Shen childhood stage” or “Shen Shen youth stage” and explain that it is used by story_to_video to choose images along the timeline.
- Age-stage images must preserve identity continuity: core face-family similarity, eye spacing and brow-eye relationship, nose shape, jawline trend, hair color/hairline cues, stable birthmarks/scars/accessories, etc. should remain unchanged or evolve reasonably.
- Childhood stages may change height proportions, facial youthful softness, hairstyle arrangement, children’s clothing, and body posture; youth/adult stages may change maturity, clothing, build, and temperament, but cannot become a different face.
- If the text only mentions one age stage, do not fill in other stages by default for life completeness; only output age variants when multiple age stages are explicitly recorded upstream or will be reused later.

## Expressions and local close-ups

- Close-ups should only be derived in specific story_to_video shots and should not be output as variants at this stage.
- Expression changes should only serve as shot-level performance signals in text: corners of the mouth, furrowed brows, moist eyes, tense breathing, frozen gaze, restrained facial muscles; do not output local variants for eyes, lips, expressions, or fingers.
- States that will be reused across shots, such as a blood-stained bandage over the right eye, a fixed scar, a cracked mask, a constantly glowing sigil, or persistent battle damage, may be included in character state turnarounds or primary-view assets; one-off wounds, bloodstains, object holding, hand seals, mouth twitching, and pupil changes do not enter the asset library.
- If facial or local close-ups are truly needed later, story_to_video should derive shot-level img2img from the character turnaround/state images; at this stage, only ensure that identity anchors and stable states are clean and usable.
- Do not use abstract words like “happy,” “sad,” or “shocked” alone to drive asset images; when a state asset is needed, it must be a stable visible state, not a momentary expression.

## Composition and style

- For baseline turnarounds and full-body variants, prefer portrait or square compositions to ensure the full body is complete, with no cropping of head or feet and no obstruction of clothing structure.
- Shot-level facial close-ups and local close-ups should be produced by story_to_video via img2img from turnaround/state images; do not choose close-range composition at this stage.
- Style should inherit the project’s overall image-generation style; style should only refine rendering texture and must not override character identity.
- Character reference images may use restrained lighting and shadow, but do not use strong volumetric light, complex particles, battle effects, or depth-of-field that obscures identity details.

## Negative constraints

- Standard exclusions: different face, age changes not required by this variant, three different characters, multiple people in frame, complex background, plot scene, text, logo, watermark, facial distortion, limb errors, malformed hands. Non-age-stage images must also exclude different hairstyles and age drift; age-stage images only allow changes in age, body posture, and stage-specific clothing as required upstream.
- Turnaround exclusions should target “three different characters, different faces, inconsistent clothing,” and should not exclude the multi-view layout itself.
- Shot-level close-up derivations should exclude extra characters, obstructed faces, overly exaggerated expressions, severe deformation, text, and watermarks; these are not variants at this stage.

## Quality standards

- It should be immediately clear who the character is, and the result should be suitable for continued use as a character reference image.
- A baseline turnaround should look like the same person, not three different people; full-body proportions, hairstyle silhouette, and clothing structure should be clear.
- Identity must remain consistent across variants, with only the clothing, state, form, or age stage explicitly required upstream being changed.
- Persistent injury, illness, labor, poverty, battle damage, or special forms must not be smoothed away by default beautification.
- Prompts must be directly generatable and understandable without relying on explanatory notes.

## Failure fallback

- When information is insufficient, prioritize stable identity, base clothing, and a clean turnaround; do not force highly specific facial features.
- If the character is only mentioned lightly and lacks sufficient appearance detail, a plain single-character reference is acceptable; for protagonists and key supporting characters, do not skip the baseline turnaround just to save images.
- If there are no clear clothing or state variants, do not force them; add them later when needed by the plot.