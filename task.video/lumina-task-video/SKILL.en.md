---
name: lumina-task-video
description: Supplementary prompt for task.video; used to organize the current turn's requirements into an executable video task
---

# task.video
## Applicable Scenarios

- The user wants to turn the current turn’s requirements into a video generation task that can be issued directly.
- The task depends only on the current message text, the assets uploaded in the current message, and the injected project style, and does not read character, scene, or prop assets maintained by the project.
- Suitable for single-shot videos, short drama clips, product ads, talking-head videos, action showcases, atmosphere shots, reference-image-to-video, and first-frame-image-to-video.
## Responsibilities

- Organize the current turn's video request into a clear, generatable video task description.
- Determine whether the current turn is a text-to-video request, first-frame image-to-video, first-and-last-frame video generation, reference-image-based video generation, reference-video style/camera movement recreation, product showcase, short drama dialogue, music beat-sync, or video editing request.
- Determine solely based on the materials uploaded in the current turn whether reference materials are involved, and what role each material serves.
- Firmly specify constraints for the subject, scene, action, camera movement, pacing, lighting and shadow, sound, dialogue, and stability.
- Rewrite scripts, dialogue, dramatic text, or scene-by-scene descriptions into shot-based language rather than copying the original text verbatim.
## What It Does Not Handle

- Does not read character, scene, prop, or historical assets maintained in the project.
- Does not perform asset binding.
- Does not force multi-stage story-to-video workflow planning into the current round of video tasks.
- Does not promise to automatically handle long-video extension, cross-segment stitching, or continuation of existing project videos; the current task only organizes the video generation requirements that can be issued for this round.
- Does not output SD tag strings, weight parentheses, command-line parameters, or platform-specific private syntax.
## Decision Principles

- Seedance 2.0-style video models are better suited to natural language, clear temporal sequencing, explicit shots, and specific actions; do not just pile on style keywords.
- Reference assets come only from uploads in the current turn; the role of each reference asset must be specified, such as first frame, last frame, character appearance, product appearance, scene, composition, pose, action, camera movement, pacing, timbre, or sound atmosphere.
- For 4-8 second short videos, focus on 1-2 core actions or product showcase points; for 9-12 second short scenes, split into 2-3 time segments; for 13-15 second complete narratives, prioritize splitting into 3-4 time segments.
- For a single video segment, 4-15 seconds is the main controllable range. If the user requests a longer duration, the current pipeline can split it into multiple independent clip tasks, but do not disguise this as an automatic video extension pipeline.
- Within a single shot, specify only one primary camera movement whenever possible, such as a locked-off shot, slow push-in, smooth lateral move, tracking shot, top-down shot, over-the-shoulder shot, or cut to close-up; do not simultaneously require push, pull, pan, and move.
- Prioritize continuous, connectable small actions, and reduce actions like sprinting, big jumps, and violent rolling that can easily cause frame drift, unless the user explicitly requests action-blockbuster-style motion.
- For high-risk movements such as entering or exiting doors, approaching or moving away from the camera, shoving, chasing, falling, or circling around, prioritize rewriting literary action into spatial anchors, screen direction, depth changes, and physical feedback.
- Emotions should be externalized into visible signals, such as pauses, eye focus, the corners of the mouth, eye rims, shoulders, fingers, breathing, and gait, rather than expressed only as abstract emotions.
## Core Writing Style

- Consolidate fixed parameters separately: duration, aspect ratio, frame size, visual style, pacing, mood, lighting, and audio direction.
- Organize the main body according to a natural shot rhythm: for each segment, first write a filmable shot sentence, then add spatial state, movement direction, and stability constraints for high-risk actions.
- If the user provides an original script or dialogue, first extract the filmable content, then convert the lines into body movements, eye lines, pauses, expressions, and shot relationships.
- Dialogue should indicate the speaker and tone, and be placed within the corresponding visual segment; do not let the entire video become a spoken monologue.
- Sound design should stay synchronized with the visuals, considering ambient sound, action sound effects, dialogue/voice-over, and BGM emotional intensity separately.
- Add stability constraints at the end: consistent character identity, stable facial features, natural motion, continuous lighting and shadows, no material flicker, and no stutter or watermark in the image.
## Spatial Trajectory Writing

- Establish a fixed anchor first, then describe the subject’s movement: doorframes, walls, table edges, bedside edges, car doors, steps, lighting boundaries, or a central object in the frame can all serve as anchors.
- When using multiple reference images, clearly specify which image locks the spatial anchor, for example: “`@Image1` locks the empty corridor and doorframe, `@Image2` locks the character identity”; image numbers indicate responsibilities only, not temporal order.
- Use screen coordinates and depth instead of vague verbs: enters from the left side of the frame, pushes toward the wall on the right side of the frame, steps back into the depth of the frame, moves along the depth axis toward the camera, becomes occluded by the doorframe.
- Inward movement should have visible evidence such as the body appearing smaller, being occluded by foreground elements or the doorframe, or moving toward a lit area in the distance; outward approach should have visible evidence such as the body appearing larger, occluding the background, or footsteps and breathing sounding closer.
- Light and shadow can serve as spatial boundaries: cool-toned shadow is the starting point, warm lit area is the endpoint; once the character crosses the light-shadow boundary, do not let them drift back in the opposite direction.
- For action scenes, write in the order “preparation -> burst -> damping”: weight shift, toe pivot, or shoulder drop as the wind-up; keep only one primary displacement; end with hair, sleeves, paper, or prop sway, or collision rebound to demonstrate inertia.
- In the same 3-second clip, do not write “turn around + walk away + look back + speak” at the same time; split it into turning in place or pausing first, then moving along the X/Y/Z axis.
- Actions that pass through opaque objects such as doors, walls, tables, or bodies should be changed to pushing open, going around, bumping into, stopping, or becoming occluded, and add a short constraint.
- Reverse constraints for high-risk actions should be short and specific, for example: “Do not move back toward the front of the camera,” “Do not pass through the opaque door panel,” “Keep the doorframe and wall fixed without drifting.”
## Natural Beat Storyboarding (Atomic Beat, Recommended)

Even in direct-to-platform mode without structured schema constraints, it is still recommended to organize prompts for Seedance 2.0 / multimodal video models by "natural beats" to avoid attention drift caused by a single block of prose:

- A single shot can be organized as `SHOT N (start–end s)`. For each SHOT, first write one natural shot sentence, then add natural shot cues using `Shot: ...`.
- If you enter Lumina pipeline structured mode, shot size/camera movement should be written as natural phrases in `cameraHint`; do not treat them as fixed enum fields.
- Time slicing should follow shot rhythm: ≤6s usually 1–3 segments; 7–10s usually 3–4 segments; 11–15s usually 4–5 segments.
- Each SHOT should reference only the `@ImageN` anchors that are truly helpful for that segment; close-ups should prioritize characters/props, wide shots should prioritize the scene, with exceptions when lighting or material is needed.
- Put style adjectives in the fixed parameter section; within SHOTs, keep the subject, action, gaze, lip movement, emotional landing point, and necessary spatial information.
- Do not forcefully split actions just to satisfy the format; only split into two consecutive SHOTs when multiple actions compete for weight.

Direct input example (8s gate scene):

```text
Visual style: Chinese-style 3D anime urban supernatural thriller | Duration: 8s | Aspect ratio: 16:9 | Frame size: 720p

Anchors:
- @图片1 = Scene: Xingfu Community gate
- @图片2 = Character: Xiaomei identity/outfit
- @图片3 = Prop: access control red light

Overview: Push in from the cold-toned exterior of the gate to an extreme close-up of the lips, waiting → probing →诱导.

SHOT 1 (0.0–2.0s)
Shot: Wide shot, slow push toward the iron gate.
Reference: @图片1 + @图片3. At the quiet, empty entrance of the residential complex under cold blue night lighting, the half-closed iron gate and the blinking access control become the center of the frame.

SHOT 2 (2.0–5.5s)
Shot: Locked medium close-up.
Reference: @图片2 + @图片1 (primary: @图片2). Xiaomei stops outside the iron gate, arms folded and leaning forward slightly, first looking at the security booth glass, then hooking her gaze toward the figure behind the glass.
Line 1: Xiaomei (cloying): "Hey handsome, I live in Building 3, Room 404. I forgot my access card."

SHOT 3 (5.5–8.0s)
Shot: Extreme close-up, slow push toward the lips and fingertips.
Reference: @图片2. The camera pushes in to the lips and fingertips. Xiaomei lightly taps her red lips with her index finger, her smile deepening before leaving a half-beat pause.
Line 2: Xiaomei (seductive): "You can do whatever you want."

Sound: ambience=far wind + access control current; actions=high heels + breathing behind glass; BGM=light suspense low
Constraints: character identity/outfit consistent / face stable / terminator line does not drift / no subtitles or watermark
```

Dialogue tag conventions (runtime compilation output; in pipeline structured mode the LLM should not assemble these itself):

- Dialogue `dialogue`: `台词N：${speaker}（${tone}）：「...」`, where N counts only spoken lines heard within the scene.
- Voiceover `voiceover`: `旁白VO（${tone}）：「...」`, with no `台词` prefix.
- Offscreen `offscreen`: `画外音（${speaker}，${tone}）：「...」`, with no `台词` prefix.
- Inner monologue `inner_monologue`: `内心独白（${speaker}，${tone}）：「...」`, with no `台词` prefix.
## HDVP Vectorized Writing (Recommended for High-Risk Shots)

For high-risk shots involving displacement, collision, passing through, crossing light-shadow zones, and other scenarios prone to "spatial inversion / clipping / direction reversal," even direct prompt input is recommended to use the HDVP three-layer structure:

- **Spatial anchors**: Use the doorframe, wall surface, table edge, or light-shadow boundary line in `@ImageN` as fixed points, and have all other subjects in the frame move relative to those anchors.
- **Motion vector (SDV)**: Use `<starting position> → <X+/X-/Y+/Y-/Z+/Z-> → <ending position>` instead of vague verbs such as "enter," "step back," or "dodge."
  - X+ = toward the right side of the frame; X- = toward the left side of the frame
  - Y+ = rising; Y- = descending
  - Z+ = toward the depth of the frame (away from the camera); Z- = toward the camera
- **Physical feedback**: Only for actions such as collision, slamming into a wall, tackling, or pinning should you add visible feedback (hair / fabric / paper / rebound / lighting changes) to prove the sense of motion.
- **Light-shadow depth wall**: When crossing between cool and warm zones, explicitly state "cool-toned start point → vector → warm-light end point; the light-shadow boundary line does not drift," and add "stable light-shadow boundary line" in the constraints.
- **Main image denoising**: When using multiple refs, declare "main reference @ImageN" so the model knows the face/clothing should be inherited 100% from that image, while the other images serve only as auxiliary references.
- **Reverse constraints**: For high-risk displacement, add short prohibitions such as "no reversal of movement direction / no passing through opaque objects."

HDVP sentence pattern example (wall-pin scene, 2.0–4.5s segment):

```text
SHOT 2 (2.0–4.5s)
Camera: medium shot, slight handheld.
Space: using the doorframe in @Image1 as the origin; Jiang Ye starts in the cool-toned shadow on the left side of the frame → X+ → ends against the right-side wall in @Image1, no penetration allowed.
Jiang Ye cuts in from the shadows, grips the throat, and uses his arm to push Xiao Mei against the wall on the right.
Constraints: no reversal of movement direction; the light-shadow boundary line does not drift.
```
## Output Modes

- Direct-to-platform mode: when the user wants a final prompt that can be "directly copied for use in Seedance / Volcano Engine / the platform," fixed parameters, shared reference assets, timeline body, sound design, and constraints can be combined into one complete prompt.
- Pipeline-structured mode: when the user wants to enter the Lumina workflow, story_to_video, task JSON, or a compiler chain, use `shotSynopsis + microShots + audioDirectives + constraintHints`; within microShots, center on `visualBeat`, and add `cameraHint/spatialHint/emotionHint/physicalHint/beatRole` as needed. Do not repeatedly write sections like "Dialogue:", "Sound:", or "BGM:".
- By default, organize the current turn's single video task in direct-to-platform mode; if upstream has already specified field names or the compiler will append dialogue/sound, switch to pipeline-structured mode.
- Both modes must preserve the same shot logic: reference assets have clearly defined roles, actions are continuous across time segments, dialogue has visual anchors, sound has visible sources, and ending constraints are brief but effective.
- Even in direct-to-platform mode, do not repeat the same information: list shared reference assets only once; keep location/emotion/lighting in only one place, either fixed parameters or the opening body section; use any long scene overview only as a brief lead-in, without repeating every action already covered in the timeline.
## Reference Material Conventions

- For prompts intended for direct use on the Volcano Engine / Seedance platform, consistently use the official reference naming: `@图片1`, `@图片2`, `@视频1`, `@音频1`. Do not mix in unofficial variants such as `@图1`, `@img1`, `参考图1`, `视频一`, etc.
- If the runtime only provides "current uploaded image numbers," you should still convert them in the output into platform-readable `@图片N` / `@视频N` / `@音频N`, and explain each one's role one by one.
- Text-only generation: directly write the subject, action sequence, environment and lighting, camera language, sound, and style.
- First-frame image-to-video: explicitly state that the currently uploaded image is the starting frame, that the video naturally animates from this image, and that subsequent actions and camera extensions must remain consistent with the first frame's composition.
- First-and-last-frame: explicitly define the starting frame and ending frame; the middle actions must transition naturally, without suddenly changing identity, space, or visual style.
- Multiple reference images: specify each image's role one by one, for example, `@图片1` locks character identity, `@图片2` locks scene space, `@图片3` locks product material, `@图片4` locks pose or close-up; do not treat reference images as frame-by-frame action.
- If a reference image serves as a spatial anchor, clearly state that the anchor must not drift, for example: doorframe, wall corner, table edge, product tabletop, car door edge, and the light-shadow boundary must remain fixed.
- Separate image-generation prompts from video prompts by layer: during the image-generation stage, you may fully describe "what image to generate"; during the video stage, keep only short role bindings such as "reference the empty street space in `@图片1`, the character identity in `@图片2`, and the prop form in `@图片3`." Do not stuff the full image-generation instructions for each image into the video body.
- For multi-image role binding, you can write: "Reference materials only lock identity, scene, props, and key evidence, and do not represent the order in which elements appear on screen"; avoid expressions like "read references in order from `@图片1` to `@图片4`," which can easily be misunderstood as frame-by-frame playback.
- If the original text contains both "`@图片N`: generate/lock ..." and "reference: 图片N=...", the final direct-input prompt should keep only one "Shared Reference Materials" section; image-generation instructions should remain in the material stage and not be repeated in the video body.
- Audit role overlap when there are many materials: an identity image and a current-state image can coexist; if the current-state image already clearly includes a handheld object, an extra prop image should only be kept when that prop is a key joke, product, or easily deformed. When approaching 6 images or more, explain why each one cannot be merged.
- Reference videos: use them only when the user has actually uploaded a video or explicitly requested it, and clearly specify whether the reference is for camera movement, action, pacing, effects, editing, or timbre; do not vaguely write "reference the overall feel of the video," for example: "Reference the handheld follow-shot pacing of `@视频1`."
- Audio reference: clearly specify whether the reference is for BGM rhythm, timbre, ambient sound, narration texture, or beat timing; for example: "The BGM mood references `@音频1`, but action sound effects are regenerated based on the visuals."
- When there are many materials, list the roles under "Shared Reference Materials" first, then write the timeline body, to avoid repeatedly explaining the same reference image in the main text.
## Scene-Type Strategies

- Short drama/dialogue: Bind visuals and lines together; dialogue is triggered by actions and emotions. Prioritize stable shots such as medium shots, close-ups, over-the-shoulder shots, and subtle push-ins.
- Xianxia/fantasy: Clearly describe the motion logic of magic circles, energy, particles, weapons, robes, and lighting effects. Visual effects should support the action and must not obscure the character’s face.
- Product/advertising: Clearly describe the product’s appearance, materials, display angle, key light, reflections, handling actions, selling points, and audio rhythm. Avoid letting people or the background distract from the main subject.
- Live-action talking-head/product selling: Prioritize close-up or medium shots, and clearly specify the person holding the product, eye line, facial expression, Chinese spoken delivery, background, and stable facial consistency constraints.
- MV/music beat sync: Clearly specify the beat, cut points, action sync points, lighting changes, and the rhythm of the reference audio/video. Do not just write “sync to the music.”
- One-take shot: Clearly specify that there are no cuts for the entire sequence, and describe the spatial path, occlusion-based transitions, subject movement, and how the camera follows.
## Duration and Pacing

- 4–8 seconds: suitable for one action, one product showcase, one atmosphere shot, or one short transition; avoid cramming in complex plot.
- 9–12 seconds: suitable for one complete short scene, which can be split into beginning, development, and payoff.
- 13–15 seconds: suitable for a complete micro-narrative; it is recommended to divide it into 3–4 time segments, with each segment carrying only one main information point.
- There should be clear connecting shots between multi-part tasks, but the current task does not automatically upload the previous segment as extension input for the next segment.
- Pacing can be written as slow, medium, or fast, but it must be reflected in shot length, action density, cut frequency, and sound intensity.
- Action density should scale with duration: within 6 seconds, use at most 2–3 main shot focal points; within 15 seconds, use at most 3–4 main information points. Action scenes may include running, chasing, and impacts, but do not stack multiple rapid cuts, complex blocking, multiple hit outcomes, and additional reactions all at once.
## Visuals and Sound

- Lighting and shadows should serve emotion and spatial continuity, such as typical home lighting, warm morning light, cool night light, lightning flashes, golden light, neon reflections, side backlighting, or soft fill light.
- When lighting and shadow define spatial boundaries, clearly specify the roles of warm/cool zones, light-dark dividing lines, or the start and end points of shadow edges to avoid the space collapsing during motion.
- Use motion blur, depth of field, and special effects sparingly to keep the subject clear, the action readable, and the face stable.
- Write ambient sound with real spatial texture, such as footsteps, clothing rustle, breathing, wind, water, indoor room tone, thunder, and metal impacts.
- For BGM, specify emotion and intensity, such as suspenseful light-comedy low, epic gentle high, tense drumbeats mid; do not just write “nice music.”
- When no subtitles are needed, explicitly state no subtitles, no text, no LOGO, and no watermark; when dialogue is needed, do not default to generating on-screen subtitles.
- Deduplicate audio before output: for similar ambient sounds, action sounds, and BGM, keep only one instance of each; sound design should be written concisely as “ambient sound + action sound + BGM transition,” and do not repeat the same sound effect with different wording.
- Only put voice-over into the dialogue field and quote the original text if there is explicit spoken content; if it is only “a cold female voice comes from upstairs, content revealed in the next shot,” do not write it as quoted dialogue—rewrite it as a sound event and character reaction.
- For postlap or sound carried into the next shot, clearly state “content unclear / specific dialogue deferred to the next shot,” and in the current shot arrange for the listener to pause, look up, freeze, or turn back, so the model does not read the explanatory text aloud.
## Compression and Counterexample Filtering

- In the final prompt, remove these director-table fields: narrative purpose, key confirmation, golden first three seconds, shot intent, emotional explanation, and what the audience will understand.
- In the final prompt, remove overly detailed cinematography parameters: focal length, aperture, lens model, f-stop, and millimeter values; unless the user explicitly asks to imitate a photography setup.
- Keep generatable information: time, natural camera prompts, subject, visible actions, scene, lighting, dialogue placement, sound events, and stability constraints.
- Keep spatial trajectory information: static anchor points, start point, end point, screen direction, depth changes, and occlusion/collision feedback; remove abstract psychological explanations and motion intent that cannot be filmed.
- If the original text looks like a shot list or director’s treatment, first distill it into a Seedance time-based storyboard instead of preserving the stacked sections as-is.
## Final Prompt Cleanup Order

1. First determine the output mode: direct-to-platform or pipeline-structured.
2. Merge reference materials: keep only one shared reference materials section, and delete "read references in image order".
3. Merge the visual body: short intro + timeline; if the long intro duplicates the timeline, keep the timeline.
4. Validate dialogue: only content actually spoken by the character goes in quotation marks; descriptive voice-over should be converted into sound events.
5. Deduplicate audio: keep ambient sound, action sound, and BGM only once each; remove repeated terms.
6. Reduce action density: based on duration, remove secondary actions or split shots to ensure each segment has one main shot focus.
7. Add a spatial audit for high-risk motion: whether the anchor point is fixed, whether the direction is unique, whether there is evidence of depth, and whether clipping and reverse motion are avoided.
8. Finally add short constraints: identity, face, action, material, no subtitles/text/LOGO/watermark.
## Quality Standards

- The video objective is clear and can be sent directly for generation.
- The subject, actions, scene, camera movement, sound, dialogue, and duration do not conflict with each other.
- The responsibilities of reference materials are clearly defined: do not use a character image as a scene image, a scene image as frame-by-frame action, or a style image as an identity image.
- Each time segment has a clear shot focus, with natural transitions between actions.
- Character identity, clothing, facial features, space, materials, lighting and shadow, and sound remain continuous and stable.
- High-risk motion has clear anchors, a start point, an end point, direction, depth changes, and inertia/collision feedback, without spatial inversion, clipping, or reversed motion logic.
## Failure Fallback

- When materials are insufficient, first organize a pure-text executable version, then point out the gaps.
- When the user provides only an abstract idea, first fill in the subject, scene, action, camera movement, lighting and shadow, sound, and duration.
- When the user's request is too long or too complex, split it into multiple independent short-video tasks, and explain the visual transition points between each segment.
- When movement direction is ambiguous, first add static anchor points and screen coordinates; if conflicts still remain, remove secondary actions or split them into two time segments.
- When there is a risk of mesh clipping, change “pass through” to “push open, go around, collide with, stop, be occluded,” and add opaque boundary constraints.
- Do not secretly introduce project assets, historical videos, or workflow-layer context.