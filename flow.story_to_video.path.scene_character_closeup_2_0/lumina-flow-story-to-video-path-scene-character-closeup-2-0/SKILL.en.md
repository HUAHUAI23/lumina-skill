---
name: lumina-flow-story-to-video-path-scene-character-closeup-2-0
description: Autonomous 2.0 scene-and-character reference-image path for story_to_video; primarily uses image-to-video from references, with clean character images, no-character scene images, and key prop anchors, rarely using evidence close-up images, combined with lightweight thematic motion prompts
---

# story_to_video path scene character closeup 2.0

## Path positioning

- This path targets Seedance 2.0 / 2.0-like multimodal video models.
- This path is not the entry point for overall directing standards; story arcs, subtext, intercut narrative, dialogue rhythm, and shot function have already been decided by upstream script, storyboard, and shot_split.
- This path is only responsible for turning confirmed shot intent into executable reference-image combinations, necessary evidence anchors, and short video prompts.
- The primary path is reference-image-to-video, not blending character and scene into a generic first frame before generating video.
- The difference between this path and blend_2_0: it does not aim to build a full master storyboard blend. It only uses character images, scene images, and key props to lock stable facts, letting the video model complete natural co-framing, camera movement, and performance.
- Images are responsible for locking stable facts: character identity, clothing, unoccupied scene space, key props, and irreplaceable evidence states.
- Text is responsible for lightweight control: who is in what scene, main action direction, primary camera movement, emotional landing point, and sonic tone.
- At runtime, after single-shot analysis and before prompt package generation, the Reference Binder runs: it only selects the best-matching anchor image for `asset=` within the same `identity=` candidate pool, and code forcibly binds it to `selectedImageRefs`.
- At the prompt package stage, you must trust the already bound `selectedImageRefs` order; do not renumber, and do not assume the image order differs from the text description.
- Prompts should be short, clear, and shootable, leaving room for the model to perform naturally; excessive precision restricts performance and creates noise, stiffness, and reference-image conflict.
- Lightweight does not mean flat narration; first use `shotSynopsis` to summarize the whole shot, then use a small number of `microShots` to write natural shot beats.

## Input responsibility model

- Text: defines theme, action direction, main shot, rhythm, lighting, sound, and emotional landing, without micromanaging frame-by-frame poses.
- Character images: lock character identity, face, hairstyle, age stage, body shape, and clothing; they do not carry the scene.
- Scene images: lock spatial layout, time, lighting, materials, and set dressing; they should be as unoccupied as possible.
- Close-up images: only lock irreplaceable local evidence or state evidence; performance signals such as gaze, mouth corners, lip sync, breathing, and line of sight are handled by the video prompt by default.
- Reference images are not a sequential storyboard playback order, but a constraint set for the same video shot.
- `@ImageN` in the video prompt only represents the final input list actually passed into the video node, not the source reference order of upstream image tasks.
- If a `storyboardFrame` must first go through image-to-image or blending, then at the video stage the generated frame can only be labeled as `@ImageN = generated reference frame / evidence close-up reference`; the source images’ “Image 1/Image 2” belong only to the image task internals and must not continue pretending to be video reference images in the video prompt.
- `activeReferenceRefs` may reference original `img_x` or `frame:start/ref_1`, but the compiler will map them to the final video manifest; do not invent new numbering when writing.

## Boundary between reference images and keyframes

- `reference_images` are not timeline keyframes; each image is a visual fact the model must satisfy simultaneously.
- Do not use multiple local-state images of the same subject to express temporal changes such as “smiling first, then afraid, then gasping”; temporal changes belong to `microShots`, shot boundaries, or post-editing.
- If you truly need to lock start and end states, evaluate `first_last_frame` first or split into two shots; do not simulate keyframes in this path using multiple close-up images.
- Zero close-up images is the default optimal solution for this path, not missing information; only irreplaceable evidence should be elevated to a single evidence close-up image.
- `[Close-up]`, `[Reaction shot]`, and `[Insert shot]` are shot language and do not automatically mean adding a new close-up image in storyboardFrames.
- Multiple state references for the same subject significantly increase hard-cut risk; if reference images express different time points, expressions, or spatial states, prefer deleting images or splitting the shot rather than forcing one video segment to satisfy all states at once.

## Asset metadata reading

- `identity=` in the project asset summary is the unique identity anchor string; use it first when judging character, scene, or prop identity.
- `asset=` in the project asset summary is the one-line usage description for that asset; use it first when deciding which state image, angle image, or prop state image should be selected for the current shot.
- Only `tier=anchor` can serve as the main reference for character identity, clean scene, or clean prop; `tier=auxiliary` may only assist in generating local images and cannot serve as a full character identity anchor.
- Do not rely on a fixed `variantKey` vocabulary for decisions; “outdoor three-view, home three-view, sleeping three-view, injured three-view,” etc. for the same character are all identified through `asset=` descriptions.
- If the shot text includes a clear character state, prefer selecting a three-view / main-view whose `asset=` matches that state; if not found, fall back to the character’s base identity three-view.
- If a character participates in the shot but there is no same-identity `tier=anchor` candidate for a character three-view / main-view, the shot should be treated as missing character identity assets; do not temporarily substitute with a partial face, hand, or text-to-image generation.
- The same applies to scenes and props: prefer selecting clean anchor images whose `asset=` matches shot time, weather, state, angle, or prop state; only irreplaceable evidence states should be derived via image-to-image inside this shot.

## Video mode rules

- By default, prioritize reference_images.
- Use first_frame only when reference-image-to-video is unavailable, or when there is truly only one image that can independently carry the action.
- Multi-image `first_frame img2img` is only for precise first-frame composition lock: for example, cases like a phone screen lighting half a face, a prop handoff, contact at the doorway, or an opening by the bedside, where the source images must first be composited into a single opening visual fact.
- When precise first-frame blending is valid, the generated first frame becomes the video node’s only `first_frame` input; character, scene, and prop source images serve only the image task and should not be redundantly passed down as video `reference_images`.
- `first_last_frame` is used very rarely; only when the ending hard result cannot be split and not locking the end frame would cause story misreading.
- `text_to_video` is only for pure empty shots, abstract transitions, or low-risk shots with no usable reference images.
- Do not plan “scene image + character image + character image” as a generic blended first frame for a normal two-character same-frame shot.

## General rules for storyboard image planning

- For each shot, first decide which reference responsibilities are needed, rather than generating one large all-in-one same-frame image.
- Participating entities must be fully listed in shotSpec / requiredRefs first: do not omit characters who appear directly, speak, are referred to by narration, are being watched, or perform actions.
- Character anchor image selection is handled by the Reference Binder within the same-identity candidate pool; storyboard planning only decides whether extra img2img evidence images, temporary anchor images, or direct reuse are needed.
- The standard combination is: no-character scene image + primary character clean image + secondary character clean image + optional key prop; if there is no irreplaceable evidence, do not force in a close-up image.
- Ordinary dialogue, same-frame staging, slight blocking changes, expression changes, push-ins, turns, sitting down, and handing over objects should be controlled by video prompt language.
- Only plan an evidence close-up image when the audience must clearly see a specific piece of evidence and natural generation in the main frame is unreliable; emotional/relationship turns should by default be completed through shot language and visible performance.
- Each image should solve only one major risk: identity, clothing, space, prop, or state evidence; do not use images to lock action process, expression process, or time slices.
- Do not plan action-process images or turn every time slice into an image; for complex montage, retain only real scene, key characters, key props, and the strongest emotional memory anchors.

## Character image standards

- Character images must be clean base images: single person, no background or an extremely simple neutral background, no other characters, no complex scene dressing.
- Character images may be generated through image-to-image or blending to update clothing, hairstyle, age stage, injuries, or fantasy/modern state.
- Even if the character image needs to reference scene lighting, era, or costume, it may only output a clean character base image and must not place the character into a specific living room, heavenly gate, street, or other scene.
- In multi-character shots, each key character should have an independent clean character image to avoid identity crossover.
- Front/side/back three-view character images are used only for identity, clothing, and silhouette, and do not require the video to reproduce the three-view layout.
- Do not use text-to-image to generate a new face for an already bound character; existing characters must start from the bound character image for image-to-image or blending.

## Scene image standards

- Scene images must prioritize no-character scene images.
- Scene images are responsible for spatial layout, shot-scale base, time, lighting, furniture/building/prop dressing, and not for character actions.
- If a bound scene image already exists, do not regenerate the scene with text-to-image; for state changes such as nighttime, dilapidation, cold light, or battle damage, use image-to-image on the bound scene image to overwrite it.
- If the scene image originally contains people, the plan should require de-personing or regeneration of a no-character version.
- Do not blend scene and character images into a generic same-frame image; at the video stage, use reference images and language to put characters back into the scene.
- Scene consistency should primarily come from existing scene images; do not generate another similar space out of thin air with text alone.
- A single video shot should by default only pass down 1 main scene image; a base image of the same location, a broken-window state image, and an exterior city image should not all compete as `reference_images` for spatial facts.
- Events at the same location such as shattered glass, power outage, cold wind rushing in, white light outside the window, or a disaster night sky should be written first into `microShots.visualBeat`, `spatialHint`, `physicalHint`, `audioDirectives`, and lighting description, without adding a second scene image.
- Only when spatial identity truly changes, time/weather/form state needs to be reused across multiple shots, or a complex space needs reusable multi-angle axis references, should derivative images be generated in the scene asset pipeline; within the same video segment, still select only one main scene reference for the current shot.
- Exterior city views, crowds downstairs, or distant disasters should by default be carried by text, sound, window light, and subject reaction if they are not the main space of the current frame, and should not be passed down as a second scene image.

## Close-up image standards (default: zero close-ups)

- By default, do not plan any close-up images; ordinary expressions, lip sync, gaze, line of sight, breathing, pauses, fading smiles, pupil contraction, etc. should be written into `microShots.cameraHint`, `visualBeat`, and `emotionHint`.
- In a single-shot `storyboardFrames`, keep at most 1 evidence close-up image with `mode=img2img/txt2img`; multiple close-up images of the same subject are not allowed to compete with each other.
- Only when all 4 conditions below are met may 1 close-up image be planned:
  1. This is a turning point where relationship or emotion changes irreversibly;
  2. The turning point corresponds to a specific visible piece of evidence or state evidence, such as a blood drop, a crack in a jade pendant, ownership of a document pouch, a talisman mark, a chipped sword blade, or wound state;
  3. Existing character anchors, clean scene images, or prop anchors cannot directly express this evidence;
  4. If the video model only sees anchor + scene + text, it will almost certainly miss or misread this evidence.
- Expression close-ups must never be generated as images: eyes, lips, lip sync, mouth corners, pupils, brows, breathing, line of sight, tears, smile, and fear reactions must all be handled through textual performance and camera movement.
- Hand close-ups should be kept only when the hand carries concrete evidence, such as a shattered jade pendant in hand, blood dripping into the palm, or a document pouch being snatched away; a simple “finger stops, pinches sleeve, grips wrist” should by default be rewritten as `emotionHint`.
- If candidate close-ups contain state conflicts, such as a smiling close-up and a fearful pupil close-up appearing together, delete them all and let the video model complete the performance transition naturally.
- Evidence close-up images serve only the current shot and are not deposited into the character/scene/prop asset library.
- `selectedImageRefs` for an evidence close-up image should contain only the single most relevant seed image; do not stack character three-views, full scene images, and multiple prop images.
- A close-up image must not simultaneously carry a full character body, the whole scene, and multiple props; the output must still be a single local piece of evidence.
- Accurate Chinese text is not a success condition; documents, paper, and plaques should only generate material, blank space, blurred traces, or an area for post-added text.

## Close-up self-check

- Is it only gaze, mouth corner, lip sync, breathing, line of sight, or expression change? If yes, delete the close-up image and move it into `microShots`.
- Does the same subject have 2 or more close-up images? If yes, keep only the strongest evidence image; if they conflict, delete them all.
- Is the evidence already covered by a prop anchor or character state anchor? If yes, delete the close-up image.
- Does the close-up require multiple seed images just to barely work? If yes, prefer splitting the shot or using text instead; do not force it through multi-image blending.
- After self-check, the number of close-up images per shot must be 0 or 1.

## Handling already split close-up shots

- When the upstream storyboard already marks a shot as `[Close-up]`, `[Insert shot]`, or `[Reaction shot]`, that does not mean this shot must generate a close-up reference frame; first treat it as shot language for cameraHint and visualBeat.
- Even when the close-up shot is already an independent shot, `reference_images` should still contain only stable anchors: character state anchor, clean scene anchor, key prop anchor; do not add another expression/eye/lip image of the same subject for this shot.
- The same subject with “medium-shot injured-state image + fearful eye close-up + pale-lip close-up” creates multi-state constraints and easily causes hard cuts; prefer keeping only the injured-state character anchor and writing fear, gasping, and pale lips into `visualBeat` and `emotionHint`.
- If the upstream has already split an INSERT into a separate shot, the post-editing transition is handled by the shot boundary; do not simulate editing transitions within the same `reference_images`.
- Only when the core of the INSERT is an independent piece of evidence, such as a black hard drive, bloodied gauze, a countdown screen, or a shattered jade pendant, and existing prop/character state anchors cannot express it, may the shot retain 1 evidence close-up image.

## Blending rules

- Blending is allowed to generate clean character images: for example, using a bound character image + clothing/state reference to output a scene-free current-stage character base image.
- Blending is allowed to generate necessary evidence close-ups: for example, key prop + necessary lighting to output a shattered jade pendant in hand or dripping blood; do not generate close-up images for ordinary expressions and lip sync.
- When multiple images are blended into an evidence close-up, the prompt must begin by clearly stating each image’s responsibility; however, this path prefers using only the single most relevant seed image, adding a second only when lighting or material is truly irreplaceable.
- The output target of close-up blending must be a single local piece of evidence, such as a hand gripping a jade pendant, a hard drive pressed into a palm, blood dripping, or gauze soaked through with blood; do not generate close-up images for lips speaking, gaze dropping, or fear reaction.
- It is allowed to use a bound scene image in image-to-image mode to generate a no-character scene state image.
- It is forbidden to blend images to generate a generic two-person medium shot, generic same-frame opening frame, generic staging setup, or generic dialogue opening frame.
- It is forbidden to blend character and scene images into “one image that looks complete” and then generate video from it; this causes the reference-image path to degrade back into the old first-frame path.
- Exception: if a precise opening composition is truly needed, one opening frame may be generated, but it must be a “single opening visual fact.” Subsequent video input should reference only this generated opening frame, without redundantly listing the source character, scene, and prop images as video anchors.
- If usable bound images exist, do not replace them with text-to-image; use text-to-image only as a fallback when assets are completely missing and the shot cannot be interrupted.

## How to write image prompts

- Use full natural language in image prompts; do not write keyword tag strings, weight brackets, SD-style parameters, or provider-specific syntax.
- First state the image’s purpose, then describe the visual content; each image should serve only one primary responsibility: clean character, no-character scene, key prop, local evidence, state variant, or missing-scene fallback.
- Text-to-image formula: subject + action or static state + environment + composition/shot scale + light/shadow/color tone + material details + style/aspect ratio.
- Image-to-image and blending formula: reference image responsibility + what to preserve + what to change + output target + composition/light/material + prohibitions.
- For multi-image input, clearly state responsibilities at the start: `Image 1` handles identity or scene, `Image 2` handles clothing/prop/posture, `Image 3` handles lighting/material/local evidence; do not write “refer to the image above” or “keep the person in the image.”
- Character consistency must be specific: preserve the same face shape, facial proportions, eye spacing, nose shape, jawline, age stage, hairstyle, and stable identifiers; only change the clothing, injury, expression, or state needed for this shot.
- Character state editing should isolate what is “unchanged” and what is “variable”: identity, face, and body remain unchanged; clothing, battle damage, glow, tear tracks, red eyes, messy hair strands, etc. change according to the story.
- Character state images may describe visible states: pale complexion, bloodied gauze, injuries, damaged clothing, messy hair; but do not separately generate close-up images of gaze, mouth corners, pupils, or lip shape.
- No-character scene images must clearly state spatial layout, time, light source, color tone, materials, dressing, and empty-scene atmosphere; no people, human silhouettes, or unrelated characters may appear.
- Missing-scene fallback images serve only as shot-level temporary spatial anchors and are not deposited as project assets; the prompt must explicitly state “no-character scene” or “temporary scene anchor,” and must not treat it as a timeline keyframe.
- Close-up images must be written as a single local area: macro lens shot or close-up composition, with the subject occupying most of the frame, and the background retaining only necessary light and material; do not generate full body, wide shot, and multiple props at the same time.
- For props, paper, plaques, documents, etc., do not rely on accurate small Chinese text; generate material, blank space, blurred traces, or areas for post-added text instead.
- A short constraint may be added at the end of an image prompt: consistent character identity, no extra people, no collage, no text watermark, clean composition, clear subject.
- Self-check: does the image solve only one risk? Is the reference responsibility clear? Are preservation and change separated? Have same-frame master images, process frames, collages, and irrelevant decoration been avoided?

## Image count and order

- Seedance 2.0 / 2.0-like models can accept multiple reference images, but the quality budget should be independent of the hard limit; this path commonly uses 2–4 images.
- Default combination: no-character scene image + primary character anchor + optional key prop anchor + at most 1 evidence close-up.
- 5 or more images are considered high-risk; first check whether the shot should be split, reference responsibilities merged, or a more keyframe-suited path used instead; do not turn `reference_images` into an asset inventory.
- The number of images is determined by responsibility, not by padding the count or by dropping key characters, key props, or truly irreplaceable evidence just to use fewer images.
- Recommended order: no-character scene image first, then primary character clean image, then secondary character clean image, and finally key prop or the only evidence close-up.
- `@ImageN` numbering comes from the final video input manifest; if the first frame is generated by img2img, then `@Image1` should be the “generated first frame,” not any of the source scene / source character / source prop images.
- “Image 1/Image 2/Image 3” inside image tasks are only used to explain source-image responsibilities for image-to-image; the final video body must rewrite anchors according to the video manifest and must not reuse image-task numbering.
- Each image description should contain only one short responsibility, such as “Image 1 locks the cold-lit empty living room space,” “Image 2 locks Lin Zhaoyue’s identity as a white-robed female cultivator,” or “Image 4 locks the document pouch close-up.”
- Do not use confusing terms such as “the image above,” “image 1,” “reference image 1,” or “Reference image 1”; use only “Image 1, Image 2, Image 3”.
- Keep image-generation instructions and video body layered: the image stage may contain full generation prompts; the video stage should only contain short responsibility bindings such as “Reference: Image 1=empty street space, Image 2=Zhou Xing pajama identity, Image 3=silver-white flying sword form, Image 4=trash can hit evidence.”
- Do not write phrases like “read references in the order of Image 1 to Image 4,” which are easy to misread as temporal playback order; rewrite as “image numbering only represents responsibility, not visual appearance order.”
- If the image list already explains responsibilities image by image, do not restate them in full again in the final video body; keep only one short responsibility-binding line to avoid double repetition of “reference material section + Reference: Image N=...”.
- Identity images and current-state images may coexist, but the division of labor must be clear: the identity image locks face and body shape, while the state image locks clothing, hairstyle change, handheld object, or injury. If the current-state image already stably includes the handheld object, retain an extra prop close-up only for key punchlines, proposition evidence, or when generation easily deforms it.

## Storyboard planning workflow

1. First determine the shot’s theme and motion: who, where, where the action starts, how the camera moves, and where the emotion lands.
2. For shots prone to spatial inversion, clipping, or reversed motion, first establish static anchors: doorway, wall surface, bed edge, table corner, pillar, light-shadow boundary, or a central frame reference object.
3. Then list reference responsibilities: which clean character images are needed, which no-character scene image, and which key props or irreplaceable evidence.
4. For each already bound character, prioritize reuse or image-to-image generation of a clean character base image.
5. For each already bound scene, prioritize reuse or image-to-image generation of a no-character scene image.
6. Only add 1 evidence close-up image when key evidence cannot be seen clearly and cannot be expressed through anchor + text; emotional/relationship turns should be written into performance and camera movement.
7. Select reference_images; fall back to first_frame only if the capability is unavailable or the shot is extremely simple.
8. Finally check whether you mistakenly generated a generic same-frame blend or multiple close-ups of the same subject; if so, revert to character image + no-character scene image + key prop.

## Example storyboard implementation: bloodied gauze and black hard drive

The following 7 short shots have already been split into independent shots upstream, so `[Close-up]` only indicates shot scale and does not mean an extra close-up reference frame should be generated.

```text
Shot 1: The door opens and someone rushes into the room, right eye covered with bloodied gauze.
Shot 2: Chen Mian looks at the bloodied gauze and presses for answers.
Shot 3: Xu Ran leans against the wall, gasping, lips pale.
Shot 4: Chen Mian freezes and asks what he saw.
Shot 5: Xu Ran’s remaining eye is full of fear.
Shot 6: Xu Ran pulls out a black hard drive and stuffs it into Chen Mian’s palm.
Shot 7: Xu Ran stares at her and explains what is on the hard drive.
```

Recommended reference responsibilities:

- Shared across the whole sequence: nighttime indoor no-character scene anchor.
- Xu Ran: prioritize a character state anchor for “right eye with bloodied gauze, face pale.” If only a base character image exists, first generate a clean injured character state image; do not generate an eye close-up image.
- Chen Mian: use the character identity anchor; terror, tightening voice, and rigid back should be written into `visualBeat` / `emotionHint`.
- Black hard drive: use a prop anchor; shot 6 may activate the hard drive prop anchor, but by default do not generate an extra palm hard-drive close-up unless its shape or ownership cannot be seen clearly.

Per-shot recommendations:

| Shot | Reference image strategy | Close-up handling |
|---|---|---|
| 1 | Scene + Xu Ran injured-state character anchor | Bloodied gauze is a character state; do not generate a separate eye close-up |
| 2 | Chen Mian anchor + Xu Ran injured-state anchor, plus scene if needed | `[Close-up]` is handled by cameraHint locking onto gaze / bloodied gauze; no close-up image |
| 3 | Xu Ran injured-state anchor + scene | Pale lips and gasping go into emotionHint |
| 4 | Chen Mian anchor + scene | Reaction shot only writes rigid back and voice catching |
| 5 | Xu Ran injured-state anchor | Fearful gaze is a performance signal; do not generate an eye close-up image |
| 6 | Xu Ran anchor + Chen Mian anchor + black hard drive anchor | The hard drive is a key prop; only allow 1 evidence close-up if ownership / form cannot be seen clearly |
| 7 | Xu Ran injured-state anchor + black hard drive anchor | Dialogue information should not rely on on-screen text; do not generate a countdown / video-screen close-up |

## Spatial trajectory control

- Prompts in this path should convert literary actions into geometric actions: write less “enter through the door, dodge, turn and leave, rush over,” and write more relative anchors, screen direction, depth changes, and visible feedback.
- For complex motion, first define a fixed spatial anchor, such as “using the doorframe in Image 1 as the central reference,” “using the wall on the right side of Image 1 as the destination plane,” or “using the bed edge and the window-light boundary to lock front-back relationships.”
- Light and shadow can serve as spatial boundaries: a cool dark area as the start point, a warm bright area as the end point, and the character must cross the light-shadow boundary; do not let the brightness relationship reverse or drift during the action.
- To express inward movement, write it as “the center of mass shifts toward the depth of frame, the body gradually becomes smaller within the doorframe, and is occluded by the doorframe or interior dressing”; to express moving outward toward the viewer, write it as “approaches the camera along frame depth, the body blocks the background, and occupies an increasingly larger share of the frame.”
- For left-right movement, use screen coordinates clearly: enters from the shadow on the left side of frame, is pushed toward the wall on the right side of frame, slides horizontally from right to left along the table edge. Do not only write “go over,” “dodge,” or “step forward.”
- Write up-down movement only when necessary, using Y-axis or height level, such as raising a hand to chest height, sinking below tabletop level, or stepping half a pace down from the stairs; avoid writing complex rotation and depth displacement at the same time.
- In short shots, organize actions as “preparation -> burst -> damping”: first give the initiating cue in toes, shoulders, gaze, or center of mass; then the main displacement; finally inertia feedback in the wall, clothing, hair, paper, prop, or light/shadow.
- In the same 3-second segment, do not simultaneously require turning around, walking away, waving, looking back, and speaking; first rotate or react in place, then enter Z-axis displacement.
- High-risk actions such as collision, shoving, pinning against a wall, falling, tackling, or passing through a doorway must clearly state opaque object boundaries: the character must not pass through a door panel, wall, table, or another body, and may only go around, push aside, hit, or be occluded.
- Use reverse constraints only in high-risk actions, short and specific, such as “forbid reverse movement toward the camera,” “forbid the character from passing through the opaque door panel,” or “keep the wall and doorframe fixed with no drift.”

## How to write video prompts (natural beat atomization)

The output form is `shotSynopsis` + `microShots`, but the mental model is not form-filling. It is to break one shot into a few natural, shootable shot beats. The structure only guarantees compilability; actual shot understanding is still conveyed by natural language.

- `shotSynopsis` is a one-sentence overview of the whole shot, clearly stating where movement starts, where it ends, and where the emotion lands; it may include a small amount of narrative emotion words, but do not restate every beat.
- `microShots` is an array of shot beats. The most important field in each beat is `visualBeat`, a natural shot sentence that may simultaneously include subject, action, line of sight, lip sync, camera movement, and emotional landing.
- **`visualBeat` must be subject-first**: the first sentence of each beat must begin with “camera + subject position + subject action,” and only then describe scene / lighting / dressing / props. Seedance 2.0 attention depends on “what appears first = what gets established first”; “environment first + camera pushes in to X” makes the model treat the character as “an element that appears later in the frame,” triggering **sudden character popping** or **fusion with the previous beat’s subject**. See § Subject-First.
- `cameraHint` is an optional natural camera prompt, not an enum selector. You may write “fixed medium close-up,” “slow push in from outside the door to facial reaction,” or “low-angle light handheld follow.” For shot scale, use the 9 standard terms (extreme wide shot / wide shot / long shot / medium shot / medium close-up / close-up / close shot / extreme close-up / reaction shot / insert shot / POV shot); do not use focal-length jargon. For “compression” or “spatial feel,” use `wide-angle spatial feel / natural perspective / portrait compression / macro texture` plus frame-share anchors (`face occupies 40% of frame` / `subject occupies 60% of frame`).
- In each beat, only fill fields that add information; ordinary expressions, lip sync, pauses, and shot-reverse-shot do not need forced `spatialHint`, `physicalHint`, or `constraintHints`.
- Time windows should continuously cover the shot duration, but the number of segments should follow the shot’s natural rhythm: short shots can have 1–2 beats, 7–10 seconds usually 3–4 beats, and complex actions can have more.

## Subject-First

Empirical comparison (user-tested, same plot, same anchors, only the first-sentence order differs):

```text
# Counterexample (environment first, character suddenly appears)
visualBeat: The bright forest cabin living room has been fully restored to neatness, sunlight spreads across the wooden floor, storage baskets are neatly placed,
           and the camera slowly pushes in at eye level toward Dudu standing in the center with family gathered around.

# Correct example (subject first, character established stably)
visualBeat: Dudu stands in the center of the frame, with Mom, Dad, Lili, and Grandpa Yantu positioned around him, as the camera slowly pushes in at eye level;
           the bright forest cabin living room has been fully restored to neatness, sunlight spreads across the wooden floor, and the storage basket is filled with neatly put-away toys.
```

Writing rules:

1. **List the subject first in the first sentence of every `visualBeat`**: `<subject>(at X position in frame) + <action/state>`, then `<scene> + <lighting> + <props>`.
2. **Carry-over characters**: for characters carried across beats/shots, the first sentence of the current beat must still **explicitly restate** their position and state (“Dudu still stands in the center of frame, chest slightly lifted”); do not assume the model remembers. Seedance 2.0 often causes carry-over characters to merge with the scene during micro-shot switching due to “delayed reveal.”
3. **New entering characters**: write **entry direction + starting position + speed** (“Jiang Ye cuts in from the left-side shadow of frame”); forbid “suddenly appears,” “suddenly flashes in,” or “there is now X in the frame.”
4. **Multiple characters in the same frame**: in one sentence, list all appearing characters in “primary -> secondary” order first, then attach the environment uniformly.
5. **Offscreen characters / absent voices**: if they do not enter `visualBeat` as subject action, imply them through `audioDirectives.sfx` + the current subject’s gaze/reaction.
6. **Do not let the camera “discover” the subject**: “the camera pushes in and reveals X in the center / pans and finally lands on X / turns out X is standing there” are anti-patterns; rewrite as `X stands in the center of frame, the camera slowly pushes in at eye level`.
7. **Keep `activeReferenceRefs` order aligned with the subject mention order in `visualBeat`**: place the first named subject in `activeReferenceRefs[0]` and set it as `primaryReferenceRef`, so “reference weight” aligns with “narrative weight.”
- `activeReferenceRefs` is an attention-anchor subset, not renumbering, and does not represent appearance order in the frame. It references real `img_x`; the compiler will map it again to `@ImageN`.
- Each micro-shot should by default activate only 1–2 references that truly add information; a single-person reaction beat should activate only that character, a single-prop insert beat should activate only the prop or generated evidence frame, and only a two-person same-frame beat should activate both characters at once.
- Do not stuff all available anchors into every micro-shot by default; full activation causes characters, scene, props, and generated frames to fight for weight.
- For close-up shot scale, prioritize character or prop anchors; for extreme wide shots, prioritize scene anchors. Only when evidence material or lighting is truly irreplaceable should the scene image also be referenced; do not reference everything meaninglessly.
- `primaryReferenceRef` should be filled only when there is an obvious primary image among multiple references; leave it empty when there is no clear primary image.
- HDVP/SDV are only for high-risk spatial actions: entering/exiting doors, moving toward/away from camera, pushing/collision, pinning against a wall, falling, chasing, passing through, or circling around. Do not force vectors into ordinary expressions and dialogue.
- Only fill `spatialHint` for high-risk displacement: clearly write start point, direction, end point, and non-penetrable boundaries; add `physicalHint` only if there is collision or aftershock.
- `beatRole` is a narrative hint, not a required field. Use it only when induction, turning point, reveal, aftershock, or irreversible relationship change is obvious.
- Fill `emotionHint` only when it helps generation, such as frozen mouth corner, held breath, stopped fingers, whitening knuckles; these are only textual performance prompts and do not generate corresponding close-up images.
- Write `constraintHints` as natural phrases, such as “forbid passing through opaque wall,” “forbid displacement direction reversal,” or “keep the light-shadow boundary stable”; do not write enum keys.
- Bind dialogue, VO, and OS through `dialogueLineIndices` to beats that can carry lip sync, listener reaction, narration-supporting imagery, or emotional landing.
- Absent characters (silhouette behind glass, voice behind the door, footsteps upstairs) should be carried first through `audioDirectives.sfx` and the current subject’s line of sight, rather than forced into visual action.

## HDVP usage boundaries

- Static anchors are for high-risk space, not for all shots. Doorframes, walls, table edges, bedside, and light-shadow boundaries can all be anchors.
- Light-shadow depth walls should be used only when cool/warm zones or brightness lines participate in spatial direction; ordinary lighting atmosphere should be handled by project style and `visualBeat`.
- Spatial vectors are risk hints, not beat-level tags. You may write X+/X-/Y+/Y-/Z+/Z-, or directly write English direction words; fixed dialogue and expressions do not need them.
- `physicalHint` is used to prove force and inertia, such as hair drifting forward, sleeves lagging behind, paper shaking, or the body briefly rebounding; do not fabricate feedback when there is no force.
- Reverse constraints should be as short as possible and only when needed, written as natural phrases such as “forbid reverse movement toward the camera,” “forbid the character from passing through walls,” or “light-shadow line must not drift.”

## Natural beat examples

In the examples below, “close-up / close shot” only indicates shot scale in `cameraHint` and `visualBeat`; `activeReferenceRefs` still reference only stable anchors and do not add expression, eye, lip, or finger close-up images.

8s gate scene (reference images: img_1=scene: residential gate, img_2=character: Xiaomei, img_3=prop: access-control red light):

```text
shotSynopsis: Push in from the cool-toned gate exterior to a close-range inducement gesture, moving from waiting -> probing -> luring.

microShots:
  1. 0.0–2.0s
     cameraHint: wide shot, camera slowly pushes from outside the gate toward the iron gate
     visualBeat: Under the cool blue night, the residential gate entrance sits empty and quiet as the camera slowly pushes toward the half-open iron gate and flickering access-control light.
     activeReferenceRefs: ['img_1', 'img_3']
     beatRole: establish waiting
  2. 2.0–5.5s
     cameraHint: fixed medium close-up
     visualBeat: Xiaomei stops outside the iron gate, arms folded and body slightly leaning forward, first looking at the security-room glass, then hooking her gaze toward the silhouette behind it.
     activeReferenceRefs: ['img_2', 'img_1']
     primaryReferenceRef: 'img_2'
     dialogueLineIndices: [0]
     beatRole: inducement advances
  3. 5.5–8.0s
     cameraHint: close-up, slowly pushing into facial reaction and fingertip gesture
     visualBeat: The camera pushes in toward Xiaomei’s profile and fingertips as she lightly taps a finger to her lips, lets the smile deepen, then leaves a half-beat pause.
     activeReferenceRefs: ['img_2']
     dialogueLineIndices: [1]
     beatRole: reveal the bait

audioDirectives:
  ambience: ['distant wind', 'access-control current']
  sfx: ['high heels', 'breathing behind glass']    # The absent Jiang Ye is implied by sfx
  bgmMood: 'light comic suspense'
  bgmIntensity: 'low'

constraintHints: []
```

8s wall-pin scene (enable SDV only for high-risk displacement):

```text
shotSynopsis: A reverse burst inside the warm-lit doorway, where playful mood collapses instantly into fear.

microShots:
  1. 0.0–2.0s
     cameraHint: fixed medium shot
     visualBeat: Xiaomei steps over the threshold into the warm-lit zone, with a joking smile still resting on her face.
     spatialHint: cool-toned exterior → Z+ → warm-lit interior doorway
     activeReferenceRefs: ['img_1', 'img_3']
     primaryReferenceRef: 'img_1'
     beatRole: establish
  2. 2.0–4.5s
     cameraHint: medium shot, light handheld
     visualBeat: Jiang Ye suddenly cuts in from the shadow on the left side of frame, hooks an arm around Xiaomei’s throat, and shoves her toward the wall on the right.
     spatialHint: left-frame shadow → X+ → right-side wall, non-penetrable
     activeReferenceRefs: ['img_2', 'img_3']
     primaryReferenceRef: 'img_2'
     constraintHints: ['forbid displacement direction reversal', 'forbid passing through opaque wall']
     beatRole: turning point
  3. 4.5–6.5s
     cameraHint: fixed medium close-up
     visualBeat: Xiaomei’s back hits the wall, her smile snaps off, and her body is forced to stop.
     physicalHint: damping rebound 0.5s, hair drifting forward, duty paper shaking
     activeReferenceRefs: ['img_1', 'img_3']
     primaryReferenceRef: 'img_1'
     beatRole: aftershock
  4. 6.5–8.0s
     cameraHint: close-up slowly pressing into facial reaction and tightening fingers
     visualBeat: The camera presses in toward Xiaomei’s frozen face and the fingers clutching her wrist as her breath stops and her knuckles blanch.
     emotionHint: breath stops, knuckles blanch
     activeReferenceRefs: ['img_3']
     beatRole: fear landing

constraintHints: ['light-shadow boundary stable', 'face stable']
```

## Complex narrative scenes

| Scene | Implementation method |
|---|---|
| Shot-reverse-shot dialogue | Alternate two micro-shots as “over-shoulder on A / medium close-up on B”; switch activeReferenceRefs between A/B character anchors; bind each side’s dialogue with dialogueLineIndices |
| Flashback | Prefer splitting into an independent shot; if it must remain within the same shot, write only one short micro-shot, with activeReferenceRefs pointing to stable memory anchors, and do not treat reference images as temporal keyframes |
| Inner monologue | dialogue.lines[].kind = inner_monologue + corresponding micro-shot uses a natural close-up shot sentence; write breathing pause / moist eyes in emotionHint; do not write mouth movement |
| Offscreen voice first (audio-visual offset) | dialogue.lines[].kind = offscreen + attach dialogueLineIndices to an earlier micro-shot; the visualBeat of that micro-shot should write the listener’s reaction |
| Montage | Prefer splitting into multiple shots or post-editing; if short beats are chained within the same shot, each activeReferenceRefs should carry only 1 representative stable anchor and not add expression close-up images |
| Metaphor shot | Insert as a separate shot; write the actual action in visualBeat (raindrops sliding down glass), not “symbolism”; build the metaphor through beatRole + temporal adjacency |
| Absent character implication | Do not put them into visualBeat subject action; carry them through audioDirectives.sfx + the current subject’s gaze direction |

## Dialogue and sound anchors

- At runtime, dialogue and sound will be concatenated into the final prompt; this path must leave enough visual motivation for them in visualBeat and audioDirectives.
- The compiler outputs `Dialogue N:` / `Voiceover VO` / `Offscreen voice` / `Inner monologue` according to `dialogue.lines[].kind`; PromptPackage must not assemble these prefixes itself, and must not restate the dialogue verbatim in visualBeat.
- In dialogue scenes, arrange visible support in the frame: the speaker’s mouth opening, the listener’s reaction, pauses, gaze landing point, and the aftertaste after speaking; do not let final dialogue float outside the image.
- For VO narration, arrange imagery that can carry the narration: protagonist memory, object evidence, empty shot, back view, slow push-in, or a stare; do not let VO cover unrelated action.
- For offscreen voice, preserve source direction or spatial relation, such as outside the door, on the phone, at the end of the corridor, or behind the screen; visualBeat should only write the character’s reaction after hearing it and the environment change, and must not miswrite the offscreen voice as on-screen mouth movement.
- If the content of the offscreen voice should be held for the next shot, do not write “a cold female voice comes from upstairs, content revealed in the next shot” as quoted dialogue; in the current shot, only write the pressure of the sound entering, the source direction, and the character freezing / looking up / continuing to flee.
- The dialogue field should contain only words truly spoken by the character or truly narrated by voiceover; explanatory text does not go in quotation marks, does not generate subtitles, and does not require the model to read it aloud.
- Inner OS / inner_monologue must not appear as lip sync or speaking to the camera; use pauses, staring, hand movement, breathing, back view, or object close-ups to carry it.
- Shot-reverse-shot dialogue should include a visible reaction chain for speaker and listener: establishing relationship shot -> speaker medium close-up / over-shoulder -> listener reverse shot -> emotional landing or return to two-person same-frame shot.
- `audioDirectives` should only supplement sound motivation, action sounds, or BGM tone missing from ShotSpec.audio. Sound words should be specific, such as light bed-edge creak, fine metallic sword tremor, muffled thunder strike on barrier, cloth friction, or short breaths.
- BGM tone should serve the segment arc, not just say “gentle / tense”; prefer phrases like “light comic suspense,” “gentle recovery within epic scale,” “suppressed then breaking,” or “after disaster rapid cuts, resolving into heartbeat.”
- Sound changes must correspond to visual changes: entering flashback, evidence close-up, emotional climax, or returning to reality must all have an audible turning source.
- Deduplicate before outputting `audioDirectives`: retain each category of ambient base noise, footsteps, cloth, plastic bag, rushing air, metal collision, and BGM only once; repeated words waste prompt budget and weaken shot control.

## Video prompt constraints

- Write the core as theme + motion, not long lyrical explanation.
- **Every `visualBeat` must be subject-first**: write subject position and action first, then environment / lighting / dressing (to prevent Seedance 2.0 from treating characters as late-appearing elements, which triggers popping / inter-beat fusion).
- **Use the 9 standard shot-scale terms** (extreme wide shot / wide shot / long shot / medium shot / medium close-up / close-up / close shot / extreme close-up / reaction shot / insert shot / POV shot), and do not write technical jargon such as focal length, aperture, depth of field, 85mm/f/1.4, etc. (Seedance does not respond to them and may even degrade); for camera texture, use `wide-angle spatial feel / natural perspective / portrait compression / macro texture` + frame-share anchors.
- A shot segment should specify only one main camera movement: fixed, slow push-in, light follow, steady horizontal move, etc.
- Write actions with direction, anchor, and transition, not frame-by-frame poses; prioritize slow, continuous, connectable small actions and avoid locking performance too tightly.
- For actions with high risk of spatial inversion, you must clearly write “relative to which anchor, toward which side of screen or end of depth, and what occludes or blocks the endpoint.”
- Write emotion as visible physical signals, such as gaze avoidance, frozen mouth corner, held breath, or unmoving shoulders/back, not just “sad, angry, shocked.”
- Place dialogue only at key positions in the corresponding time window; do not turn the whole segment into recitation.
- Dialogue, VO, and OS remain dialogue information and do not generate subtitles or on-screen text.
- Sound may briefly describe ambience, action sound, and BGM mood; do not expand sound into long explanations.
- Keep ending stability constraints short: stable character identity, natural motion, no flicker, no subtitle watermark.
- In the final body given to the model, remove directing table fields and explanatory content such as narrative purpose, key confirmations, golden three seconds, audience understanding, focal segment, aperture, and lens model; these may exist in internal analysis but must not enter the video prompt.

## Seedance methodology implementation

- Separate fixed parameters from the body: style, duration, and aspect ratio are placed at the top by the compiler, and microShots do not repeat them.
- Prioritize visual storytelling: first use shotSynopsis to make the whole shot read smoothly, then use microShots to write natural beats.
- A natural shot sentence usually includes: time sense, spatial anchor, camera prompt, subject, direction of movement, scene/lighting, and visible emotional signal or physical feedback; not every beat needs all fields.
- Keep shot language restrained: one main shot scale or one main camera movement per time window; do not stack push, pull, pan, and move at the same time.
- Keep action shootable: use pauses, looking up, turning the head sideways, breathing, fingers stopping, objects slipping, and other small actions to express emotional turns.
- Keep space verifiable: anchors such as doorframe, wall, table edge, bedside, and light-shadow boundary stay fixed; character movement happens around anchors so space does not collapse around the subject.
- Keep motion verifiable: entering, exiting, approaching, leaving, shoving, falling, and circling should be readable from body-size proportion, occlusion relation, center-of-mass direction, and inertia feedback.
- Make reference-image responsibilities explicit: each image should lock only identity, space, prop, or irreplaceable evidence, and should not serve as frame-by-frame action or temporal keyframe.
- Keep language control restrained: lock only main action, direction, and emotional landing, without chasing one action per second or every micro-expression.
- Dialogue and sound cannot rely only on the final fields; the image must have corresponding lip sync, reactions, action-sound sources, and BGM turning points.
- Keep constraint words short: retain only high-yield constraints such as stable character identity, stable face, natural motion, and no subtitle watermark.

## Quality standards

- The plan result should make it clear that character images are clean base images, scene images are unoccupied, and key props are stable; if a close-up image exists, it must correspond only to a concrete piece of evidence.
- Ordinary same-frame relationships should not depend on blending, but be completed through reference images + language control.
- When bound character/scene assets exist, do not regenerate identity or space with text-to-image.
- If multiple images are truly blended into an evidence close-up, `Image 1` and `Image 2` must each have clear responsibilities, and the output must still be a single local piece of evidence; ordinary expressions and reactions must not use multi-image blending.
- `microShots` time windows must continuously cover the shot duration; each beat must have a clear shot center and natural visualBeat.
- `microShots` must not pile on directing psychology explanations; necessary emotion should preferably be written as visible action, gaze, lip sync, pause, or body signal.
- Close-up shot scale, local evidence, and insert beats should prioritize reducing scene-image interference; wide shots, long shots, and establishing shots should prioritize including scene anchors. Exceptions are allowed when lighting/material truly requires it.
- Fill `primaryReferenceRef` when `activeReferenceRefs` length is ≥2 and there is a clear main image; do not force-fill it when there is no clear main image.
- For high-risk actions, prioritize coverage with `spatialHint`, `physicalHint`, and `constraintHints`; use `beatRole` only when the narrative turn is obvious.
- For actions crossing light-shadow zones, `spatialHint` should state the cool/warm zone start and end, and `constraintHints` should include “light-shadow boundary stable.”
- Attach dialogue via `dialogueLineIndices` to micro-shots that can carry lip sync / reaction; attach offscreen lines to listener-reaction beats.
- Deduplicate all three `audioDirectives` categories (ambience/sfx/bgmMood); absent characters are implied by sfx.
- Each image must have a clear responsibility and must not be a poster, collage, process frame, or complete storyboard set.
- First/last frames should appear very rarely; most shots should be carried by `reference_images` or, when necessary, `first_frame`.

## Failure fallback

- The LLM still outputs old-style long prose video prompts: check whether the schema loaded correctly; rewrite into shotSynopsis + microShots, keeping natural shot sentences in visualBeat.
- visualBeat carries multiple actions competing for weight: retain the main shot center and split secondary actions into the next beat or delete them.
- **Characters suddenly pop into the generated video / fuse with the previous beat**: check whether the first sentence of visualBeat is environment-first (“bright living room... + camera pushes in to X”); change it to subject-first (“X stands in the center of frame, camera pushes in; bright living room...” ). The same issue also occurs when a carry-over character’s position is not explicitly restated in the first sentence of the current beat; add a sentence like “X is still in the center of frame.”
- **Passive reveal phrasing**: if visualBeat contains “reveals X / finally lands on X / it turns out X is standing there,” rewrite it as objective subject state + camera action.
- **New entering character “appears out of nowhere”**: if visualBeat contains “X suddenly appears / there is suddenly X / X appears in the frame,” add entry direction + starting position + speed (“X enters from the shadow area on the left side of frame and runs closer”).
- **Ambiguous shot scale or focal-length jargon**: if cameraHint says “somewhat wide medium-long shot” or “85mm shallow depth of field,” change it to a 9-standard-term shot scale + natural phrase + frame-share anchor, such as “medium close-up + portrait compression + face occupies 40% of frame.”
- A close-up or local shot includes a scene image and causes face drift: remove the scene image first and keep only the character anchor; if environmental lighting reference is truly needed, shorten the scene reference responsibility.
- There is a gap in micro-shot time windows: extend the nearest micro-shot or add a transition micro-shot; “blank space” is not allowed.
- Spatial inversion: add a spatialHint that clearly writes the SDV phrasing and screen direction; add “forbid displacement direction reversal” in constraintHints.
- Clipping: write “forbid passing through opaque objects” in constraintHints; for aftershock beats, write “hits / is occluded / rebounds” rather than “passes through.”
- Drift across light-shadow zones: write the cool/warm zone start and end in spatialHint, and “light-shadow boundary stable” in constraintHints.
- Face mixing from multi-image identity conflict: when `activeReferenceRefs ≥2` and there is a clear primary face / primary clothing, add `primaryReferenceRef`; for close-up beats, prefer referencing only 1 character anchor.
- Local evidence is unclear: add a short insert beat (≤1.2s) with activeReferenceRefs referencing only that evidence image; if the emotional turn is unclear, rewrite performance, camera movement, and emotionHint instead of adding an expression image.
- Offscreen voice is mistaken for spoken dialogue: set `dialogue.lines[].kind = offscreen` and attach it to the listener micro-shot; visualBeat should write the listener’s reaction.
- Reference image duplication: the anchor registry is the single source of truth, and micro-shots should reference only imageRef; inventing new numbering in spatialHint or visualBeat is not allowed.
- Repeated sound: deduplicate by ambience/sfx/bgmMood categories; each audioDirectives category should have ≤3 items.
- Atmosphere words erode the shot: if microShots contain words like “oppressive / restrained / atmosphere,” rewrite the whole beat as visible signals.
- The image is too stiff: reduce the number of constraintHints; let reference images lock identity and space, while micro-shots provide only movement direction without locking a micro-expression chain.