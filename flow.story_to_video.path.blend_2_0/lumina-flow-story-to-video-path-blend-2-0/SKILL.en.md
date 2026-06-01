---
name: lumina-flow-story-to-video-path-blend-2-0
description: The 2.0 blend-image autonomous path for story_to_video; primarily uses first-frame and reference-image video generation, and completes image-to-image, text-to-image, and blending by shot
---

# story_to_video path blend 2.0

## Path Positioning

- This path is intended for Seedance 2.0 / 2.0-class video models.
- This path is not the entry point for the overall directing specification; story arc, subtext, intercut narration, dialogue rhythm, and shot function have already been determined by the upstream script, storyboard, and shot_split.
- This path is responsible for turning confirmed shot intent into controllable key shot images, necessary reference images, and short prompts that can directly generate video.
- The core strategy is to first create a reliable key image for the current shot, then let the video naturally animate from that image.
- Images are responsible for locking identity, wardrobe, space, props, composition, and state; text is responsible for motion onset, camera movement, temporal shot beats, emotional landing points, and sonic character.

## Video Mode Rules

- first_frame is the default main path: suitable when the key image for the current shot has already been blended and the video only needs to move naturally from that image.
- reference_images is suitable for shots that require continued constraints across multiple characters, scenes, props, compositions, or memory fragments.
- first_last_frame is used only when the required ending result cannot be split into separate shots and cannot naturally evolve from the first frame.
- text_to_video is only for pure empty shots, abstract transitions, or low-risk shots.
- Do not mechanically use a mode just because the general context supports it; choose based on the risk of the current shot.

## Shot Image Rules

- First determine what this shot most needs to lock: first-frame composition, character relationships, spatial consistency, prop evidence, state transformation, emotional close-up, or a hard ending result.
- When a new same-frame image is needed, you may blend character images, scene images, and prop images into one key shot image for the current shot.
- The key shot image only serves the starting frame of the current shot: who is present, how they are positioned, where the action starts, and what the lighting and composition are.
- You may add close-up, action-composition, or state reference images, but each must resolve a specific risk: identity, space, prop interaction, action composition, state transformation, emotional evidence, or payoff of foreshadowing.
- For flashback or montage shots, only add key anchor images: the main real-world anchor, the most important emotional memory, prop evidence, and necessary scene anchors; do not generate one image for every time slice.
- For shots within 15 seconds, normally use 1–4 images; complex montages can use 5–7; when approaching 9 images, you must prioritize splitting the shot or merging anchors.
- Do not add irrelevant reference images just to enrich the frame, and do not create posters, collages, multi-panel comics, or process frames.

## Blend Prompt Rules

- Image-to-image and blend prompts must begin by stating the role of each reference image, for example, `Image 1` locks the scene space, `Image 2` locks Zhou Xing's identity and costume, `Image 3` locks the amulet material.
- The final blend generates only one image for the current shot, not a collage, side-by-side multi-image layout, before/after comparison, or shot group.
- A normal key-shot blend may handle two-character same-frame composition, character-scene fusion, prop ownership, and starting-action composition.
- A close-up blend may output only a single local piece of evidence, such as reddened eyes, a hand gripping talisman paper, knife-edge reflections, or a document bag landing on a table; it must not simultaneously handle full body, wide framing, and multiple props.
- For state transformations, clearly specify what to preserve and what to overwrite: preserve face and clothing; overwrite injuries, battle damage, light/color, and prop state according to the plot, to prevent the model from reverting to a default polished state.

## Video Prompt Writing

- The final video prompt should follow Seedance style: put fixed parameters separately at the front, and let the body advance according to natural shot rhythm.
- Lumina pipeline fields use `shotSynopsis + microShots + dialogueLineIndices + audioDirectives + constraintHints`; do not output old-style long-form prose video prompts.
- If the output is a complete prompt sent directly to Seedance / Volcano Engine, you may use official references such as `@Image1`, `@Video1` and merge fixed parameters, audio, and constraints; if the output is for Lumina pipeline fields, then each micro-shot should use `visualBeat` to write one natural shot sentence.
- In the video body, prioritize writing the natural shot sentence first, then add natural shot guidance, reference anchors, spatial direction, visible emotional signals, and short constraints as needed.
- **For each SHOT / visualBeat, the subject must come first in the opening sentence**: first write "camera + subject position + subject action", then scene/lighting/set dressing. "Environment first + the camera pushes in to X" is a high-frequency anti-pattern that triggers **sudden character pop-in** / **cross-shot frame blending**. See §Subject-First for details.
- **Use the 9 standard shot-size terms** (extreme wide shot / wide shot / long shot / medium shot / medium close-up / close-up / tight close-up / extreme close-up / reaction shot / insert shot / POV shot). Do not write technical jargon such as focal length, aperture, depth of field, 85mm/f/1.4, etc. (Seedance does not respond to them and may even degrade); if you need lens feel, use `wide-angle spatial feel / natural perspective / portrait compression feel / macro texture` plus frame-coverage anchors (`face occupies 40% of the frame` / `subject occupies 60% of the frame`).
- A first_frame shot must move naturally from the first frame; in the first beat, clearly state the motion onset "starting from the first-frame image" and do not switch space or blocking out of nowhere.
- When a reference_images shot needs to refer to source assets, use the real `img_x` or the compiler-mapped `@ImageN`; numbering only indicates reference role, not time order.
- Separate image-generation prompts from the video body: in the image stage, you may write full generation or blend instructions; in the video stage, retain only short role bindings and do not stuff the full image instructions into every time segment.
- If per-image descriptions already exist, the final video body should retain only one shared reference-material segment and should not repeat the full responsibility list as “Reference: Image N=...”; do not write “read references in image order.”
- For durations above 9 seconds, usually split into 2–4 natural beats; for 13–15 second narrative shots, usually split into 3–5 beats; quick-cut flashbacks may be written as 2–3 very short shots within a single beat.
- Beats only control shot focus and movement; they do not lock pose frame by frame. Do not force vector fields into ordinary segments.
- Do not express emotion only with abstract words; land it in visible signals: eyes gradually redden, breathing stops, fingers tighten, mouth corners stiffen, shoulders and back stay still, gaze averts or settles.
- Dialogue will be appended by the compiler; the video body must leave space for speaking motion, listener reaction, pauses, or emotional handoff for each key line, and must not rewrite the original dialogue in the visual body.
- The compiler outputs `Dialogue N:` / `Voice-over VO` / `Off-screen voice` / `Inner monologue` according to `dialogue.lines[].kind`; PromptPackage must not assemble these prefixes itself, and must not repeat the dialogue text inside visualBeat.
- If off-screen voice content continues into the next shot, do not write the explanatory text as quoted dialogue; in the current shot, write only the off-screen voice direction, vocal pressure, and listener reaction.
- Audio will be appended by the compiler; audioDirectives should only supplement ambient sound, action sound, and BGM character and intensity, and should not stuff BGM notes into the visual body.
- Before outputting audioDirectives, deduplicate across the three categories of ambient sound, action sound, and BGM, and remove repeated variants of the same wording.
- For montage and flashbacks, clearly state the real-world anchor, transition motivation, memory anchor, and landing point back in reality; do not write them as a running list of reference images.
- For strong physical interaction, hugging, fighting, piercing, or complex hand actions, prioritize close-ups, silhouettes, props, reaction shots, and environmental consequences to avoid model weaknesses.
- In the final body given to the model, remove directing table fields and overly specific cinematography parameters, such as narrative purpose, key confirmation, golden three seconds, audience comprehension, focal range, aperture, and lens model.

## Seedance Structural Preferences

- A good short prompt is not simply the shorter the better; the information should be clearly partitioned: reference material roles are clear, visual action advances over time, and dialogue and sound have corresponding performance positions.
- You may preserve the texture of natural shot sentences, but do not output long stacks of sections; for the model, the focus is actionable movement, shots, and time order.
- For dialogue scenes, prioritize stable close-up or medium close-up, slow push-in, over-the-shoulder, and reaction close-ups; use pauses and eye-line continuity to carry dialogue so the whole scene does not become flat line-reading.
- For disaster, awakening, combat, and flashback sequences, quick cutting is allowed, but every short shot must have a clear function: destruction evidence, bodily reaction, prop payoff, emotional reversal, or real-world landing point.

## Subject-First

Empirical comparison (user-tested; same plot and anchors, only the opening sentence order differs):

```text
# Counterexample (environment first; characters easily pop in / blend across shots)
SHOT 1: The bright forest cabin living room has been fully restored to order, sunlight spreads across the wooden floor, storage baskets are neatly in place,
       the camera slowly pushes in at eye level toward Dudu standing in the center and family gathered around.

# Positive example (subject first; characters are established stably)
SHOT 1: Dudu stands in the center of the frame, with Mom, Dad, Lili, and Grandpa Yantu positioned around him, the camera slowly pushes in at eye level;
       the bright forest cabin living room is fully restored to order, sunlight spreads across the wooden floor, and the storage baskets are filled with toys put away securely.
```

Writing rules:

1. **In the first sentence of every SHOT / `visualBeat`, list the subject first**: `<subject>(at X position in the frame) + <action/state>`, then write `<scene> + <lighting> + <set dressing/props>`.
2. **carry-over characters**: when the same character continues across SHOTs, the first sentence of this SHOT must still explicitly restate position/state ("Dudu still stands in the center of the frame, chest slightly lifted"); do not assume the model remembers the previous beat.
3. **newly entering characters**: write the direction of entry into frame + starting position + speed ("X runs in from the shadowed area on the left side of the frame"); prohibited are passive reveal phrases such as "suddenly appears", "suddenly flashes into view", or "there is now X in the frame".
4. **multiple characters in the same frame**: within one sentence, list all appearing characters in the order of "primary → secondary", then attach the environment afterward.
5. **`first_frame` shots**: the first sentence must return to the visual facts of the key shot's first frame, and the subject must still appear in the first sentence; do not let "environment change" crowd out the subject.
6. **Do not have the camera "discover" the subject**: `the camera pushes in and reveals X / pans over and finally lands on X` is an anti-pattern; replace it with the subject's objective state + camera movement.

## Quality Standards

- Shot images serve execution of the current shot; do not make posters, character illustrations, or irrelevant atmosphere images.
- The first-frame key shot image can independently support the starting frame of the video.
- Additional shot images have clear responsibilities, do not duplicate, and are not filler.
- Video prompts stay consistent with the first frame or reference images and do not switch space or characters out of nowhere.
- Use first-and-last-frame mode sparingly and only for hard results.
- Video prompts do not repeat fixed parameters such as visual style, duration, or aspect ratio.
- Every beat has a natural shot sentence and a clear shot focus; it is not a prose retelling of the plot.
- Dialogue has visible performance anchors, and sound has corresponding on-screen events or atmospheric sources.
- For multiple reference materials, use `@ImageN` with clear role binding; do not treat reference images as frame-by-frame playback order.
- If a 15-second shot is packed with too many spaces, too many characters, or too many memory anchors, split the shot or reduce it to fewer representative anchors.

## Failure Fallback

- Prompt reads like an asset list: remove low-value reference images and keep only the main real-world anchor, key characters, key props, and the strongest emotional memory.
- Dialogue is floating: add the speaker's mouth movement, breathing, eye focus, and listener reaction in the corresponding time segment without rewriting the dialogue.
- Weak audio: add ambient sound, action sound, and BGM emotion and intensity in audioDirectives, and preserve visual events that can produce those sounds.
- Action distortion: split complex interaction into hand close-ups, prop results, reaction shots, and environmental consequences.
- Frame drift: return to the first-frame key shot image or add one clearly scoped reference image; do not keep piling on scattered text.
- **Character pop-in / cross-shot blending**: check whether the first sentence of visualBeat is environment-first; change it to subject-first. If a carry-over character's position is not restated in this beat, add a line such as "X remains in the center of the frame." For newly entering characters, add entry direction + starting position + speed.
- **Passive reveal phrasing**: if visualBeat contains "reveals X / finally lands on X / it turns out X is standing there / X suddenly appears / X suddenly flashes into view / there is now X in the frame", rewrite it as the subject's objective state + camera movement.
- **Ambiguous shot size or focal-length jargon**: if you wrote "slightly wide medium-long shot" or "85mm shallow depth of field", change it to a 9-tier standard shot size + natural phrase + frame coverage, for example, "medium close-up + portrait compression feel + face occupies 40%".