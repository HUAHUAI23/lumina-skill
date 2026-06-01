---
name: lumina-flow-story-to-video-path-1-5-pro
description: Autonomous path for story_to_video 1.5 Pro; prioritize first-frame generation, forbid reference-image-to-video, use first-and-last frames only when necessary
---

# story_to_video path 1.5 Pro

## Path Positioning

- This path is intended for the 1.5 video model.
- The shared context only provides currently supported video scenarios and recommended durations; this path decides mode selection on its own.
- Reference-image-to-video is forbidden, and multiple reference images must not be used as the main video input.
- Prioritize the first frame; use first-and-last frames only when a mandatory result must be locked within the same shot.

## Single-shot analysis hard requirements

- During single-shot analysis, always output both `shotSpec.description` and `shotSpec.narrativeGoal` as non-empty strings.
- `shotSpec.description` must state the concrete visual content of the current shot in one grounded sentence: who or what is on screen, what visible action is happening, and what the shot is visually showing.
- `shotSpec.narrativeGoal` must state the explicit storytelling function of the current shot in one grounded sentence: what transition, confirmation, reveal, emotional handoff, or payoff this shot is carrying.
- Never rename these fields to `summary`, `goal`, `shotSummary`, `narrativePurpose`, or any other alias, and never merge them into a single field.
- If the information is sparse, still provide a short fallback value grounded in the current shot `originalText`; never omit the field, never return an empty string or `null`, and never use placeholders such as `N/A`, `same as above`, or `see summary`.

## Single-shot analysis hard requirements

- During single-shot analysis, always output both `shotSpec.description` and `shotSpec.narrativeGoal` as non-empty strings.
- `shotSpec.description` must state the concrete visual content of the current shot in one grounded sentence: who or what is on screen, what visible action is happening, and what the shot is visually showing.
- `shotSpec.narrativeGoal` must state the explicit storytelling function of the current shot in one grounded sentence: what transition, confirmation, reveal, emotional handoff, or payoff this shot is carrying.
- Never rename these fields to `summary`, `goal`, `shotSummary`, `narrativePurpose`, or any other alias, and never merge them into a single field.
- If the information is sparse, still provide a short fallback value grounded in the current shot `originalText`; never omit the field, never return an empty string or `null`, and never use placeholders such as `N/A`, `same as above`, or `see summary`.

## Video Mode Rules

- Prioritize first-frame-to-video.
- If plain text is sufficient and the shot does not depend on project-bound images, text-to-video may be used.
- Only when the ending contains an irreplaceable mandatory visual result, and it cannot be split into the first frame of the next shot, is first-and-last-frame mode allowed.
- Even if the shared context shows that multi-reference is supported, do not choose reference-image-to-video.

## Storyboard Image Rules

- Determine which storyboard images are needed based on the storyboard.
- For ordinary shots, prioritize generating a single first-frame image.
- If the frame requires a new composition with multiple characters, scenes, or props, first complete the first-frame image using bound-image image-to-image or image blending.
- End frames serve only mandatory results; do not add end frames for ordinary action progression, expression changes, slight repositioning, or camera push/pull.
- Storyboard images are all single still frames, not motion sequences, before/after comparisons, or collages.

## Prompt Rules

- For image-to-image or image blending prompts, first clearly state the identity, spatial, or prop responsibility of each reference image.
- First-frame prompts should describe only the starting point of the current shot’s action: who, where, what motion is beginning, and what composition and lighting apply.
- Video prompts must start moving naturally from the first frame and describe one continuous shot.
- The Lumina pipeline fields still use `shotSynopsis + microShots`: write `visualBeat` as a natural shot sentence, and write `cameraHint` only when helpful, using natural hints such as “static medium shot” or “slow push-in to the hands.”
- The compiler outputs `dialogue.lines[].kind` respectively as `台词N：` / `旁白VO` / `画外音` / `内心独白`; PromptPackage must not assemble these prefixes itself, and must not restate the original dialogue text in `visualBeat`.
- For direct platform prompts, write them as “fixed parameters + natural beat body”: put style, duration, and aspect ratio separately at the front; write the body as continuous action by time segment, and do not use old-style long-form prose video prompts.
- In the first-frame path, the first micro-shot must clearly state the action onset “starting from the first-frame image,” without arbitrarily switching space, identity, or position.
- Do not write multi-reference video instructions.
- First-and-last-frame prompts must emphasize the same space, the same subject, and the same continuous chain of action.

## Quality Standards

- Every shot must have at least one clear first-frame execution path.
- When a new composition is needed, prioritize blending existing bound images into the current first frame.
- Do not produce plans that stack multiple references for consistency.
- Any use of first-and-last frames must explain what the mandatory result is, why the shot cannot be split, and why the first frame cannot evolve into it naturally.
- Character identity, clothing, age stage, scene space, and key props must remain consistent with the project.
