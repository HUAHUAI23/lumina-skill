---
name: lumina-consult-image-prompt
description: Supplemental prompting for consult.image_prompt; used for image prompt guidance and optimization
---

# consult.image_prompt

## Applicable Scenarios

- The user wants to write, revise, or optimize an image prompt, but is not ready to submit a task yet.

## What It Does

- Organize image ideas into prompt suggestions that can be used directly.
- Fill in the subject, action/state, environment, composition, style, lighting, color palette, materials, aspect ratio, and constraints.
- Help the user lock down the visual goal and reduce description conflicts.
- Adjust wording based on the use case, such as character reference images, empty scene shots, isolated prop images, posters, thumbnails, product images, or style transfer images.
- Choose the appropriate prompt pattern based on the user's goal, such as text-to-image, image editing, multi-image fusion, style transfer, text poster, or character consistency.

## What It Does Not Do

- Does not create image tasks.
- Does not read project-maintained assets.
- Does not bind characters, scenes, or props.
- Does not output SD tag strings, weight parentheses, command-line parameters, or provider-specific private syntax.

## Decision Principles

- Modern image models respond better to complete natural language and logically clear constraints; prioritize understandable scene descriptions over keyword stuffing.
- When user information is incomplete, prioritize filling in the subject, use case, style/mood, composition/shot type, lighting/color palette, materials, and aspect ratio.
- Prompts should be directly usable, not left as creative prose or written as loose tags.
- Constraints must be realistic and effective, and should not conflict with each other.
- Professional photography terms, lens terms, or material terms may keep common English expressions, such as Macro lens shot, rim light, and soft lighting, but do not turn them into a stream of English tags.
- When editing an existing image, prioritize stating "what to change" and "what to keep"; do not rewrite it as a completely new text-to-image prompt.
- In multi-image workflows, you must clearly define the role of each `Image 1/Image 2/Image 3` to avoid vague references like "the above image" or "this image."
- In multi-image fusion, declare the **main image** (HDVP main image denoising): `Main: Image 1`, so the model knows identity/materials are inherited 100% from the main image, while other images are only auxiliary references (lighting/composition/pose).
- Character consistency must be specified down to face shape, facial feature proportions, eye spacing, nose shape, jawline, age range, hairstyle, and stable identifying markers.
- If the user needs strict consistency, separate fixed elements from variable elements before writing the final prompt; fixed elements take priority over style enhancement.
- A single image should follow the "single responsibility" principle: each image should solve only one primary risk (identity / space / prop / close-up detail / state variation / light-shadow boundary). Making one image handle "full identity + panoramic space + multiple props" at the same time is an antipattern and causes the model to conflict across multiple goals.

## Working Method

- First determine the request type: text-to-image, image editing, multi-image fusion, style transfer, text poster, character reference image, thumbnail, or product image.
- For text-to-image, organize as "subject + action/state + environment + composition/shot type + lighting/color palette + material details + style/aspect ratio."
- For image editing, organize as "change action + target of change + changed characteristics + unchanged elements."
- For multi-image fusion, first list image responsibilities: `Image 1` locks subject identity, `Image 2` locks scene/style, `Image 3` locks pose/props/composition; then write the final scene, and finally declare `Main: Image N`.
- For character expression changes, do not only write "happy/sad/shocked"; describe visible changes such as mouth corners, eye rims, brows, pupils, breathing, tear marks, and muscle tension (that is, the image version of "visible emotional signals"), and state that the face must not be changed, beautified, or aged up/down.
- For scene images, by default clearly state the space type, layout, entrance/path, key furnishings, lighting, season/weather, materials, and main color palette; if it is an asset reference image, default to an empty shot with no people.
- When a scene image involves a light-shadow boundary (cool blue → warm yellow, indoor/outdoor, bright/dark), explicitly state the "position of the light-dark dividing line" and the "distribution of warm and cool areas" so downstream video shots can lock the HDVP light-shadow depth wall.
- For prop images, by default clearly state the silhouette, material, size ratio, texture, wear, glow, or markings; unless the user explicitly asks, do not add a person holding it or a complex background.
- For text posters, put the generated text in quotation marks, and specify the font style, layout position, color scheme, and use case; also remind the user that complex small text may require post-edit proofreading.
- Finally, provide one copy-ready prompt version; for complex requests, you may first provide "asset/reference image responsibilities" and then the main prompt text.

## Output Format

- Simple text-to-image: directly output one natural-language prompt.
- Editing an existing image: output one short edit instruction that clearly states what is preserved and what is changed.
- Multi-element or multi-image: use bullet points to explain the subject, scene, composition, lighting, style, materials, and reference image responsibilities.
- Strict character consistency: write "fixed identity" and "variable state" separately, and add short negative constraints if necessary.

## Quality Standards

- Specific, actionable, and directly usable.
- Clear visual goal, with no conflict between style and subject.
- At minimum, cover the key elements among subject, composition, lighting, color palette, materials, and aspect ratio.
- Reference image responsibilities are clear, preserved vs. changed elements are separated, and there are no contradictory instructions; if there is a clear main reference in a multi-image setup, declare the main image.
- A single image solves only one primary risk; do not force one image to serve multiple purposes.
- Key text is already in quotation marks, and character consistency includes specific constraints.
- For scene images involving light-shadow boundaries, specify the warm/cool regions and the light-dark dividing line to help downstream video lock the HDVP space.

## Failure Fallback

- If the user provides too little information, first give a minimum usable version, then point out the missing dimensions.
- Do not use vague adjectives in place of specific visual information.
- When the user says "make it look better," convert that into actionable changes: increase contrast, adjust lighting, change the color palette, strengthen the subject, clean up the background, or improve materials.
- If editing multiple objects at once creates high risk, recommend step-by-step changes: first the subject, then the background, then lighting or text.
- If identity drifts after multi-image fusion, check whether a main image was declared; if not, add `Main: Image N, identity/materials inherited 100% from Image N`.
- If one image is being asked to handle "full character + panoramic space + multiple props" at the same time, split it into 2-3 single-responsibility images, then combine the reference images at the video stage.