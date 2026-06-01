---
name: lumina-consult-video-prompt
description: Supplemental prompting for consult.video_prompt; used for video prompt writing and shot-language guidance
---

# consult.video_prompt

## Applicable Scenarios

- The user wants to write, revise, or optimize a video prompt, but is not ready to submit a task yet.

## What This Handles

- Turn video ideas into directly usable prompt suggestions.
- Fill in subject action, camera position, camera movement, pacing, and scene changes.
- Provide concise shot breakdown suggestions when needed.
- Evaluate multiple prompts provided by the user, identify which are suitable for direct model input, and which are closer to a director’s treatment, shot list, asset notes, or internal analysis.
- When explaining optimization direction, make it concrete in terms of structure, duration, action density, reference materials, sound, and stability constraints; do not just say “make it more concise.”

## What This Does Not Handle

- Does not create video tasks.
- Does not read project-maintained assets.
- Does not present consultation output as formal workflow planning.

## Decision Principles

- Video prompts should prioritize clear action and clear camera language.
- Multi-shot requests must preserve sequence order and the goal of each segment.
- Use fewer vague adjectives; give real, executable information.
- For Seedance 2.0-style prompts, prioritize judging by "fixed parameters + anchor catalog + natural beat storyboard (SHOT N) + sound design + stability constraints"; when evaluating, watch for information overload caused by single-paragraph prose.
- A good prompt is not simply “the more detailed the better”; the version sent directly to the model should remove internal director-table fields such as narrative intent, key confirmations, focal length/aperture, audience understanding, golden first three seconds, etc.
- Reference image guidance should be split into two layers: image-generation prompts are for generating assets, while video prompts should list them only once in the anchor catalog; each SHOT should reference only the subset needed for that shot, not reuse the full image set across all shots.
- For high-risk shots (displacement / collision / traversal / crossing light-shadow zones), evaluate using the HDVP three-layer check: whether spatial anchors are fixed, whether motion vectors are clear, and whether physical feedback is visible.
- **Subject-First**: The visual description of every SHOT must begin with "camera + main person/subject," then follow with scene / lighting / set dressing; "environment first, person later" is not allowed, otherwise the model may treat the person as a delayed reveal and produce "sudden character pop-in" or a hard-cut feeling disconnected from the scene. See §Subject-First Rules.
- **Shot-Size Precision**: The `shot` line of every SHOT must use a shot-size term from the dictionary (extreme wide shot/long shot/full shot/medium shot/medium close-up/close-up/extreme close-up/reaction shot/insert shot/POV shot) + one primary camera movement + a pacing adverb; technical jargon such as focal length, aperture, depth of field, 85mm/f/1.4 is forbidden (Seedance does not respond to it and may even degrade). If you need "compression" or "spatial feel," write it in natural language: wide-angle spatial feel / natural perspective / portrait compression / macro texture; if you need frame occupancy, write it directly: "subject occupies 60% of frame," "face occupies 40%," "foreground 1/3 + subject in medium shot." See §Shot Size and Lens Feel Rules.

## Subject-First Rules

Seedance 2.0’s attention allocation depends heavily on "what appears first = what gets established first." If the key character in a shot is placed after the environment in the description, the model often treats that character as an element that appears only later in the shot, causing two common failure modes:

1. **Sudden character pop-in**: after the environment is established first, the character gets "inserted" into the frame a few frames later, creating a collage-like or hard-cut feel.
2. **Misaligned frame fusion**: visual memory from the previous shot and the current environment get established first, while the subject appears only at the end and fuses incorrectly with the previous shot’s subject.

Comparison from practice (user-tested):

```text
# Counterexample A (environment first, character may pop in)
The bright living room of the forest cabin has been completely tidied up, sunlight spreads across the wooden floor, storage baskets are neatly put in place,
the camera slowly pushes in at eye level toward Dudu standing in the center with family gathered around.

# Positive example B (subject first, character established stably)
The camera slowly pushes in at eye level toward Dudu standing in the center with family gathered around. The bright living room of the forest cabin has been completely
tidied up, sunlight spreads across the wooden floor, and the storage baskets are filled with toys that have been neatly put away.
```

Executable writing rules:

- **The first sentence of every SHOT’s visualBeat / visual description must start with "camera + subject position + subject action"**, then add environment / lighting / set dressing / props.
- **Carry-over characters (characters already present in the previous shot)**: still mention them explicitly at the start of the current shot (position + state). Do not assume the model remembers. Example: "Zhu Xi still stands in the warm-lit center of the courtyard; a student runs in from the left side of frame." Do not write: "A student runs in from the left side of frame, while ahead of him Zhu Xi’s gaze locks onto him" ("ahead of him Zhu Xi" is a delayed reveal and may pop in).
- **Newly entering characters**: must specify entry direction + starting position + speed. Example: "X enters from the shadowed area on the left side of frame and walks closer / enters frame." Do not use passive reveal phrasing such as "suddenly appears," "suddenly flashes into view," or "X is now in the frame."
- **Multiple characters in one frame**: within one sentence, list all characters in "primary → secondary" order first, then add the environment as a whole. Example: "Dudu stands at frame center, Mama Dumi on the left, Papa Dushan on the right, Lili and Grandpa Yantu split across the background; the bright living room’s wooden floor is lit by sunlight."
- **Off-screen characters / absent voices**: do not put them into the subject action of the visualBeat; carry them through audioDirectives.sfx + the current subject’s reaction.
- Letting the **camera itself "discover" the character** is an anti-pattern: forbid phrasing such as "the camera pushes in and reveals X standing in the center" or "the camera pans over and finally lands on X," which treats the subject as a "surprise reveal"; change it to "X stands in the center, the camera slowly pushes in at eye level."

Use this checklist when evaluating prompts:

- Does the first sentence start with "environment + sunlight + set dressing"? If yes → flag "missing subject-first; likely to cause character pop-in."
- Does a carry-over character appear only in the second sentence? If yes → flag "carry-over character is revealed too late."
- For a newly entering character, is the entry direction specified? If not → flag "missing character entry direction; may cause position drift or pop-in."
- Does it use passive reveal terms such as "reveals," "finally appears," or "turns out to be standing"? If yes → rewrite as subject-first + objective state description.

## Shot Size and Lens Feel Rules

The Seedance 2.0 official docs explicitly note that technical jargon such as focal length / aperture / depth of field / 85mm / f/1.4 can degrade the model. If you need lens feel, rewrite it using natural phrases:

| What you want to express | Do not write | Rewrite as |
|---|---|---|
| Large-scale scene, wide-angle | 24mm wide | extreme wide shot / wide-angle spatial feel / open field of view / subject occupies 1/4 of frame |
| Narrative medium shot | 35mm cinematic | medium shot / natural perspective / subject occupies 1/2 of frame |
| Natural perspective | 50mm normal | medium close-up / natural perspective / no distortion |
| Intimate portrait | 85mm portrait | close-up / portrait compression / softened background / face occupies 40% of frame |
| Evidence/detail close-up | 100mm macro / f/2.8 shallow depth of field | close-up / macro texture / subject occupies 60% of frame / blurred background |

Shot size dictionary (choose one): `extreme wide shot / full shot / long shot / medium shot / medium close-up / close-up / extreme close-up / reaction shot / insert shot / POV shot`

Format for each SHOT’s `shot` line: `<shot size> + <one primary camera movement> + <pacing adverb>`, for example:

- ✅ `medium close-up, slow push-in, main reference @image2`
- ✅ `locked close-up, subject occupies 60% of frame, blurred background`
- ✅ `extreme wide shot, light lateral move, smooth, subject occupies 1/4 of frame, foreground lawn 1/3`
- ❌ `medium close-up 85mm shallow depth of field f/1.4` (focal-length jargon; Seedance degrades)
- ❌ `push in first, then circle` (two camera moves in one shot; ambiguous movement)
- ❌ `fast push-in` (`fast` is explicitly named in official guidance as degrading; use `quick whip pan / sudden push-in` instead)

Shot size changes should be split across different SHOTs or different timestamps. Do not write "push from wide shot to close-up" within a single time segment. If you need an internal shot-size change, write it as: `[00:00-00:02] medium shot locked` → `[00:02-00:05] slow push-in to close-up, subject occupies 50% of frame`.

## Working Method

- First lock the subject, scene, and main action line.
- Then fill in camera position, camera movement, pacing, and end state.
- Finally provide one directly usable prompt version, with brief shot suggestions if needed.
- If the user asks “which ones are good and which ones are bad,” evaluate along these dimensions: whether the timeline is clear, whether each segment has one shot focus, whether the subject and reference material are clear, whether the action can be generated continuously, whether sound is deduplicated and matched to the visuals, and whether constraints are short and effective.
- If the user asks “how to optimize,” give principles first, then a reusable template or partial rewrite; do not output only abstract suggestions.
- For 4–8 second videos, recommend 2–3 time segments; for 9–12 second videos, recommend 2–3 time segments; for 13–15 second videos, recommend 3–4 time segments. If over that, prioritize cutting actions or splitting shots.
- Action scenes may include running, chasing, and impact, but reduce parallel actions within the same time segment; dialogue scenes should prioritize pauses, eyelines, breathing, and hand movement to carry the lines.
- When evaluating a finished prompt, check in order: whether reference material is duplicated, whether it says “read references in image order,” whether a long overview repeats the timeline, whether voice-over notes were put into quoted dialogue, whether sound is duplicated, whether too many assets overlap in responsibility, and whether action density exceeds the duration.

## Quick Reference for Strengths and Weaknesses

- Worth learning from: natural SHOT segmentation, clear shot focus in each beat, **subject-first in the opening sentence**, transitions between actions, dialogue anchored by lip-sync or reaction, sound with a source event, subset-based reference image use per SHOT, and standard shot-size terms such as extreme wide shot / medium shot / close-up / extreme close-up.
- Needs compression: single-paragraph prose + stacked time ranges, **environment-first opening sentence**, too many fields, too much explanation, repeated asset responsibilities per segment, repeated sound, too many actions in the same second, emotion written only as abstract terms, close-up shots referencing the full set of images.
- Not recommended for direct feeding: full director’s treatments, professional shot lists, camera parameter tables (focal length / aperture / depth of field), image-generation asset lists, plot synopses or stacked raw dialogue, long "visuals and action" prose paragraphs.
- Keep when optimizing: SHOT numbering, duration, natural camera prompts, subject, natural shot sentences, visible emotional signals, the subset of reference images activated in the current beat, key dialogue placement, sound events, and short stability constraints.
- Remove when optimizing: narrative intent, key confirmations, focal length/aperture, camera model names, audience psychology explanations, repeated BGM/sound effects, adjectives unrelated to the image, paragraph connectors ("then / immediately after / finally"), and passive reveal phrasing ("reveals / finally lands on / turns out to be").

## Natural Beat Storyboarding (Recommended Output Form)

Upgrade a single-paragraph prose prompt into a `SHOT N / start–end s` natural-beat format:

- Header: one line for `style | duration | aspect ratio | frame size`; an `anchors:` list in the form `- @imageN = type:identity`.
- `Overview: ≤60 characters` should state "what moves from where to where, and where the emotion lands."
- Within each SHOT, first write one natural shot sentence, then add `shot`, `reference`, visible signals, spatial constraints, or sound events as needed; shot size and camera movement are natural guidance inside the `shot` line, not enum headers.
- Time slicing should follow shot rhythm: ≤6s usually 1–3 segments; 7–10s usually 3–4 segments; 11–15s usually 4–5 segments.
- For high-risk displacement, add an SDV sentence pattern: `space: <start anchor> → <X+/X-/Y+/Y-/Z+/Z-> → <end anchor>`; direct Chinese direction words are also acceptable.
- End with a constraints line: `constraints: character identity/clothing consistent / do not pass through opaque objects / light-shadow boundary does not drift / no subtitles or watermark`.

## Common Fixes

- Misread reference image order: delete "read references in order from `@image1` to `@imageN`"; change to "image numbering only indicates reference responsibility; each SHOT should cite a subset."
- Repeated reference responsibilities: merge into the top-level "anchor catalog"; each SHOT should only write `anchors: @imageN + @imageM (main: @imageN)`.
- Single-paragraph prose + stacked time ranges: split into `SHOT 1/2/3...` multi-shot format, with one natural shot sentence at the start of each segment.
- **Environment-first causing character pop-in**: reverse the opening sentence from "bright living room + sunlight + storage basket + camera pushes in to X" to "camera pushes in to X standing in the center + followed by living room + sunlight + storage basket." See §Subject-First Rules.
- **Character being "revealed"**: change passive reveal lines such as "the camera pushes in and reveals X" or "finally lands on X" into "X stands at frame center / X enters from the left side of frame, the camera slowly pushes in."
- **Carry-over character drift**: when the same character continues across SHOTs, the first sentence of the next SHOT must still state that character’s position and state explicitly; do not rely on the model to "remember."
- **Focal-length jargon**: change `85mm/f/1.4/shallow depth of field/focal plane` into natural phrases such as `close-up + portrait compression + softened background + face occupies 40% of frame`. See §Shot Size and Lens Feel Rules.
- **Two camera moves in one shot**: split `push in first, then circle / tracking while panning` into timestamp hard cuts like `[00:00-00:02] push-in / [00:02-00:05] circle`, or split into two SHOTs.
- Multiple actions in parallel ("walking while talking while flirting with the eyes"): split into two consecutive SHOT segments; the first handles the action, the second binds the dialogue.
- Close-ups or partial shots citing full scene-image sets: prioritize reducing scene-image references, using person or prop anchors instead; if lighting/material is truly needed, keep only a short responsibility note.
- Multiple refs with no primary image: if one beat has a clear primary face, outfit, prop, or space, declare `main: @imageN`.
- Drift across light-shadow zones: in the `space` field, specify the start and end of the cool/warm zones + add "light-shadow boundary does not drift" to the constraints line.
- Mesh/intersection risk: add "do not pass through opaque objects" to the constraints line; in action description, use "collides with / gets occluded by / rebounds off" instead of "passes through."
- Voice-over mistakenly written as dialogue: if the content belongs to the next shot, do not write quoted dialogue; change it to "an icy female voice comes from upstairs, the exact words unclear, the character freezes / looks up."
- Repeated sound: keep only one instance of repeated footsteps, plastic bag rustle, air-cutting whoosh, metal sparks, or BGM; merge by ambient sound, action sound, and BGM, with ≤3 items per category.
- Too many reference images: first judge whether responsibilities overlap; identity images and state images can coexist, but prop images should only be kept when they are key to a punchline, product, or easily deform.
- Atmosphere words taking over: `cinematic / epic / oppressive / restrained / premium` should be confined to the fixed-parameter section whenever possible; inside SHOTs, keep only generatable actions, eyelines, lighting, and sound.

## Quality Standard

- The prompt has clear action and camera direction.
- The content can be fed directly to the model without requiring the user to translate it again.
- Evaluation conclusions must guide the next rewrite: clearly state what to keep, what to delete, what to merge, and how many segments to split into.

## Failure Fallback

- When materials are insufficient, provide the minimum usable version first, then point out the gaps.
- Do not bluff shot language with empty terms like “cinematic” or “premium feel.”