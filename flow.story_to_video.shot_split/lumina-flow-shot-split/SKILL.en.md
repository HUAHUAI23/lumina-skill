---
name: lumina-flow-shot-split
description: Supplemental prompt guidance for flow.story_to_video.shot_split; used to faithfully structure confirmed storyboard text into a continuous shot list, with necessary cleanup of shot boundaries, capacity, and continuity
---

# flow.story_to_video.shot_split

## Stage positioning

This is the structured compilation stage of story_to_video, not the directing/creative stage.

Upstream, `consult.screenplay` handles story structure, character motivation, setup, information asymmetry, and dialogue; `consult.storyboard` handles director-level storyboarding, shot language, editing, sound, and rhythm. The current stage only organizes storyboard text already confirmed by the user into a continuous shot list consumable by downstream workflows.

## Applicable scenarios

- The user has already confirmed a storyboard text through `consult.storyboard` and needs to enter story_to_video.
- The input may include: shot numbering (`### Shot NN: Title (about X seconds)`), dialogue blocks (`### Dialogue Block NN: Title (about X seconds)`), hard cuts with `[00:00-00:05]` timestamps inside shot blocks, a top-level three-part cross-shot manual of `## Character Anchors` + `## Continuity Ledger` + `## Visual Reference Pool`, and fielded sections (`Narrative Objective / Scene/Mood / Visual Narrative / Action & Blocking / Shot Size/Angle/Camera Movement / Eyeline & Space / Dialogue/lip-sync / Sound Three Sublayers / Causality/Physics / Reference Pool References / Handoff / Negative Constraints / Editorial Handoff`, etc.).
- The shot order, dialogue, sound, characters, scenes, props, duration tendencies, negative constraints, SDV vectors, named lighting fields, reference pool slots, handoff 5W, and cross-shot manuals in the text need to be preserved stably.
- Also compatible with users directly sending scripts or stories, but only perform conservative splitting; do not take on full storyboard creation here.

## What it is responsible for

- Faithfully structure the confirmed storyboard. Do not rewrite the story, reorder shots, or proactively alter dialogue.
- Preserve original shot order, titles, episode attribution, scenes, characters, props, dialogue, voiceover, sound, duration, and editorial handoff.
- Organize each shot into clear visual content, narrative objective, action starting point, direct result, sound text, ambient sound, and asset references.
- Perform continuity cleanup across adjacent shots: previous shot end state, current shot start state, action continuation, eyeline continuation, sound continuation, and next-shot entry.
- Mechanically split obviously overloaded shots: multiple spaces, multiple plot objectives, severely overloaded dialogue capacity, or cases like "start wide then cut to close-up" or "many years later cut to the present" inside one shot.
- Mark whether dialogue capacity is natural, tight, or overloaded. In confirmed storyboard text, do not delete key dialogue on your own just for capacity.
- Match character, scene, and prop names from current project assets. If uncertain, leave blank or place them in the visual description; do not fabricate stable assets.
- Character references must be checked strictly for omissions: any character who appears directly, speaks, is referred to by narration, is being looked at, performs an action, or serves the primary functional role of the current shot must be included in that shot's character references, so the downstream Reference Binder can force-bind turnaround/main-view anchors.

## What it is not responsible for

- Do not create new character, scene, or prop assets.
- Do not invent new plotlines, character motivations, twists, setups, end hooks, or thematic imagery.
- Do not change the original storyboard to make it "look better"; if creative problems are found, only perform minimal cleanup, and when necessary note that the user should return to storyboard consultation for revisions.
- Do not decide storyboard images, first frames, multi-reference, first/last frames, image blending, video mode, or final prompt strategy.
- Do not output platform-specific parameters, weighting syntax, coordinate values, CSV, or backend protocols.

## Input modes

### Confirmed storyboard text

This is the main path.

- Input conservation takes priority: process in original order, do not add shots, reorder shots, change plot, or change dialogue.
- Preserve existing shot numbers, titles, durations, shot sizes, camera movement, sound, dialogue, locations, characters, and props from the original text as much as possible.
- If a shot text is already clear and reasonably sized, do not break it down further just to make it "more cinematic."
- If a shot needs splitting, the resulting continuous shots must preserve the original intent and explain that they come from different actions, reactions, results, or sound segments of the same original shot.
- If the original text contains "Episode 1 / Episode 2", still keep a continuous shot list, but preserve episode attribution in the title or description.

### Raw story or script text

This is a compatibility path, not the recommended path.

- Only perform conservative shot-structure splitting: establish space, main action, key reaction, evidence close-up, and result landing point.
- Do not proactively apply short-drama formulas, rewrite the large structure, or invent new twists.
- If the text clearly has not gone through storyboard creation yet, provide an executable coarse-grained split and note that the user can return to storyboard consultation for further directing.

## Cross-shot manual consumption (top manifest)

Upstream `consult.storyboard` places two "cross-shot manual" sections at the top of the storyboard text first, **written only once and not repeated inside each shot block**. shot_split must read these two sections first, otherwise character consistency and cross-shot context will be lost:

### `## Character Anchors`

The format is like:
```
Xiaomei: pink puff-sleeve knit top / black short skirt / short straight black hair at jaw length / small mole on right brow bone / thin silver necklace
Jiang Ye: black security uniform / buzz cut / cartilage stud in left ear / thin black cord bracelet on right wrist
```

Consumption rules:

- Extract each character's `character name` and `visual anchor string`, and store them in the `anchor_descriptor` field of that shot's character reference object.
- If the character has a stable asset in the project, use the stable asset name under current rules; **in addition**, also carry `anchor_descriptor` for downstream Seedance prompt assembly of character tokens.
- If the project **does not** have a corresponding asset (under old rules this kind of character would be dropped into description text instead of asset references), **use anchor_descriptor as a temporary placeholder** in character references so downstream can consume it; also flag in quality notes: "character X lacks a project asset; using temporary anchor placeholder".
- Validation: every character appearing in each shot must be able to find a corresponding string in the anchor manifest; if missing, flag it in quality notes.

### `## Continuity Ledger`

The format is like:
```
Timeline: that night 22:00 → 22:15 (continuous 15 minutes, no cross-day jump)
Weather baseline: light rain → moderate rain, wind from south-southeast
Active props: keycard (in Jiang Ye's pocket), monitor screen (upper right on wall)
Lighting baseline: security room cool white fluorescent 4000K + red exterior sign light through glass door 3200K (flicker 1.5Hz)
Emotional trajectory: Xiaomei probing → alert → fearful; Jiang Ye lazy → suppressive → indifferent
Prop state machine: keycard intact (shot1) → rubbed by Jiang Ye's fingers (shot4) → slapped onto the desk (shot7) → slips to the floor (shot12)
Lighting continuity matrix: shot1-6 4000K cool white + red 3200K flicker; shot7 add monitor 6500K blue; shot8-12 red flicker frequency 1.5Hz→3Hz
Sound bridge plan: shot3→4 J-cut keycard click enters 0.4s early; shot7→8 L-cut monitor beep extends 1.2s; shot11→12 sound_bridge rain intensifies
```

Consumption rules:

- Store as-is in shot list global metadata `continuity_ledger` (not a per-shot field) for downstream cross-shot verification.
- `Prop state machine` / `Lighting continuity matrix` / `Sound bridge plan` are new fields; store them respectively in `continuity_ledger.prop_state_machine` / `continuity_ledger.lighting_matrix` / `continuity_ledger.sound_bridge_plan`, preserving the original text of each section.
- **Do not modify the ledger** when splitting shots; the ledger changes only when the user explicitly changes the plot back in `consult.storyboard`.
- Validation: each shot's time, lighting, and emotional progression must not conflict with the ledger; if they do, flag it in quality notes and do not alter the shot on your own.
- **Prop state validation**: when a shot description includes a prop from the state machine, its state description should match that shot's state-machine stage; if not, flag "shot N prop X state does not match continuity ledger".
- **Sound bridge validation**: for shot pairs mentioned in the ledger sound bridge plan (such as shot3→4), the two shots' `audio.sfx[]` / `audio.ambient` should correspond; if missing, flag it.

### `## Visual Reference Pool` (new manifest section, must be parsed)

The format is like:
```
Image slots (≤9):
  @char_main_xiaomei      Xiaomei turnaround sheet
  @char_main_jiangye      Jiang Ye turnaround sheet
  @scene_baoanshi         Security room main image
  @lighting_baoanshi      Named lighting field reference
  @prop_keycard           Keycard close-up
Video slots (≤3):
  @motion_pushin          Standard push-in rhythm 2s
Audio slots (≤3):
  @ambient_rain_indoor    Indoor rain ambience 5s loop
  @bgm_tension            Tension BGM 8s
```

Consumption rules:

- Parse each `@slot_name` line, extract `slot` (including the `@` prefix), `type` (image / video / audio, based on subsection), and `description`, then store in global metadata `reference_pack[]`.
- Validate slot limits: `image` ≤ 9 / `video` ≤ 3 / `audio` ≤ 3; if exceeded, flag "visual reference pool exceeds Seedance Universal Reference limit".
- Validate total video / audio duration ≤ 15s (if duration is declared in the description, e.g. `5s loop` / `8s`); if exceeded, flag it.
- Parse `slot` naming: `@<purpose>_<name>`, where purpose must be one of `char_main / char_outfit / char_face / scene / lighting / prop / style_palette / motion / transition / ambient / bgm / voice`; non-enum values should be flagged as "reference pool slot purpose is non-standard" but still stored.
- Every `@slot` parsed from a shot's "Reference Pool References" field (see § downstream constraints and meta field passthrough below) must be found in the global `reference_pack[]`; if not found, flag "shot N references undeclared reference pool slot @xxx".
- When the **top reference pool is missing but stable project assets exist**, continue under the current conservation mode and flag: "missing visual reference pool manifest; cross-shot consistency falls back to pure token mode; recommend adding `## Visual Reference Pool` at the top in `consult.storyboard`."

### Manifest format mutual-exclusion validation

The `## Character Anchors` at the top from `consult.storyboard` uses a **multi-line format** (one character per line, fields separated by ` / `); sibling skill `consult.storyboard-script-to-shot-prompts` uses a **single-line `Character Anchors: A=...;B=...` format**. This skill only handles the multi-line format.

If the input top matches a single-line `Character Anchors: A=...;B=...` or single-line `Reference Pool: @xxx=...;@xxx=...` format, **immediately prompt**: "The input uses compact manifest format and should be handled by sibling skill `lumina-flow-shot-split-scene-as-shot`" and stop structuring. Do not force-parse it.

## Video segment aggregation consumption (video segments)

Upstream `consult.storyboard` wraps shot blocks under **video segment** heading lines — **1 video segment = 1 Seedance call**.
Multiple internal shots are combined into one continuous generation via `[00:00-00:05]` timestamps. A video segment is the
aggregation layer between shots and the full piece, and shot_split must parse this grouping structure and write it into global metadata.

### Input format

```text
## Video Segment 1/8: Xiaomei visits the apartment complex gate at night｜Total duration about 14.5s｜Ending handoff: Jiang Ye sits behind the duty desk, corners of his mouth lifted, right hand paused beside the access-control button

### Shot 01: ...
### Shot 02: ...
...

## Video Segment 2/8: ...
```

Field description (from upstream § video segment aggregation rules):

- `Video Segment X/N`: X is the current index, N is the total number of video segments in the full piece
- Anchor: a one-line summary of the scene/action mainline
- `Total duration about X.Xs`: the sum of all shot durations in the segment (**default 12-15s**, or a user-explicit value)
- `Ending handoff`: final-frame character position / eyeline / hand pose / sound tail, etc.

### Write into structured output

Parse each video segment as one item in global metadata `video_segments[]`:

```json
{
  "video_segments": [
    {
      "segment_index": 1,
      "segment_total": 8,
      "anchor": "Xiaomei visits the apartment complex gate at night",
      "declared_duration": 14.5,
      "actual_duration": 14.5,
      "ending_handoff": "Jiang Ye sits behind the duty desk, corners of his mouth lifted, right hand paused beside the access-control button",
      "shot_ids": ["shot01", "shot02", "shot03", "shot04", "shot05", "shot06", "shot07"]
    }
  ]
}
```

Add a top-level `video_segment_id` field to each shot, pointing to its segment index (`1` / `2` / ...), so the downstream router
can batch-generate Seedance calls by segment.

### Validation rules

- **Default duration range 12-15s**: each video segment's `actual_duration` should be within 12-15s; if outside, flag
  "video segment X duration Y.Ys deviates from default 12-15s; confirm whether the user explicitly specified a different duration". Still write output from the original text; do not merge/split on your own.
- **Use user-explicit override values**: if the user explicitly specifies another duration (such as "each video 8-10s"), validate against that range and no longer flag against 12-15s.
- **Seedance hard limit ≤15s**: if any video's `actual_duration` > 15s, flag "video segment X duration Y.Ys exceeds
  Seedance's hard single-generation limit of 15s; must return to consult.storyboard for segment splitting", and do not truncate on your own.
- **Segment numbering must increase continuously**: start from 1, with no skips or duplicates; if inconsistent, flag it.
- **Segment total N must match actual segment count**: if a title says `Video Segment 1/8` but there are actually only 7 segments, flag it.
- **`declared_duration` vs `actual_duration`**: if declared duration differs from the sum of shot durations, flag it and use actual as authoritative.
- **`ending_handoff` is required** (except for the final segment of the full piece): if missing, flag "video segment X is missing ending handoff; downstream `image_to_video_first_frame` cannot obtain first-frame context".

### When video segments are missing (compatibility for old storyboards)

If the upstream storyboard has no video segment title lines (old-version output / user handwritten not following the new spec), use the following fallback:

1. flag "input missing video segment aggregation; cross-segment consistency falls back to shot-level handoff only (see § cross-shot handoff fields)"
2. **Do not group shots into video segments on your own** — let downstream handle them as "one independent Seedance call per shot" (higher cost but safer)
3. In quality notes recommend: "return to consult.storyboard and add `## Video Segment X/N:` title lines at the top to wrap shot blocks"

## Working method

1. First determine whether the input is confirmed storyboard text. If it clearly contains shot numbering, shot blocks, dialogue blocks, or semantics such as "confirmed / execute this", enter conservation mode.
2. **Read the three top cross-shot manual sections first** (`## Character Anchors` + `## Continuity Ledger` + `## Visual Reference Pool`), and build global context according to the consumption rules above. If the top uses a single-line compact manifest format, instruct the user to switch to the sibling skill.
3. **Parse video segment title lines** (`## Video Segment X/N: ...｜Total duration about X.Xs｜Ending handoff: ...`), build `video_segments[]` global metadata according to § video segment aggregation consumption rules, and assign each shot to its `video_segment_id`. If the input lacks video segment titles, apply the fallback in § when video segments are missing (compatibility for old storyboards) + flag.
4. Build the shot list in original order, preserving episode, scene, and shot title.
5. For each shot/dialogue block, extract existing information: narrative objective, visual action, shot size/angle/camera movement, eyeline/spatial vectors, characters, scenes, props, dialogue/lip-sync, sound three sublayers, reference pool references, handoff 5W, negative constraints, duration, transitions, and creative meta (emotion intensity, subtext, causal/physical logic).
6. **Recognize dialogue blocks and timestamp hard cuts**, and expand them into continuous sub-shots according to the rules in § "Dialogue blocks and timestamp splitting"; shots without dialogue blocks or timestamps are converted directly under the current conservation mode.
7. Check shot boundaries: only further split into continuous shots when there is a change in space, subject, emotional reversal, new information reveal, action result landing point, or overloaded sound text.
8. Organize adjacent continuity: character position, orientation, eyeline, held props, action ending, ambient sound, and emotional aftertone must not jump without cause.
9. **Parse each shot's "Handoff" block** (5W + transition type + repeated token + Seedance input mode + first-frame reference), and write it as-is into structured output according to the rules in § "Cross-shot handoff fields". The first shot may omit handoff; from the second shot onward, missing handoff should be flagged but not backfilled.
10. Match project assets + reference pool slots: follow the character reference rules in § "Cross-shot manual consumption"; stable project asset names go into `characterRef`, anchor strings go into `anchor_descriptor`, reference pool slots go into `reference_refs[]`, and all three coexist.
11. Check for missed characters: for each shot, independently verify character references at the end. Do not miss a character just because they appear only in dialogue, narration, back view, reaction, or partial close-up; also validate against the anchor manifest.
12. Mark capacity: estimate dialogue/voiceover length, speakable time, pauses, and aftertone. When overloaded, prioritize splitting or marking overload rather than swallowing key lines; for lip-sync dialogue, additionally validate the three-part requirement (quotes + emotional prefix + ≤12 Chinese characters).
13. **Determine Seedance call mode**: each shot's top-level `seedance_call_mode` field is read from that shot's `Seedance Input Mode` inside the handoff block; if missing, default to `text_to_video` and flag "shot N missing Seedance input mode declaration; falling back to default text_to_video; cross-shot consistency may degrade".
14. Pass through downstream constraints: negative constraints, sound three sublayers, SDV vectors, causal/physical logic, reference pool references, handoff, and meta fields must be written as-is into structured output according to § "Downstream constraints and meta field passthrough", with nothing dropped or rewritten.
15. Finally perform a conservation check: whether shot-count changes were necessary, whether key original text was preserved, whether order is consistent, whether assets were not fabricated, whether the cross-shot manuals were not altered, and whether handoff 5W is complete; also whether video segment aggregation is complete (each segment 12-15s or user-specified value, `ending_handoff` exists, and `shot_ids` cover all shots).

## Shot boundary cleanup

- One shot should maintain the same space, the same subject relationship, and the same narrative objective.
- You may preserve a shot's internal opening move, progression, result, and short reaction, but it must not span scenes, time periods, or major objectives.
- If the original text says "first go to location A, then to B", "cut back from childhood to the present", "start wide then go macro", "many years pass", or "sudden flashback", it should usually be split.
- If the original text says "after he finishes speaking, silence", "she freezes after hearing it", or "everyone stops and looks at her", then if the dialogue or information matters, prioritize splitting out the reaction or consequence.
- Strong editing intent already present in the original text must be preserved: J-Cut, L-Cut, sound bridge, match cut, jump cut, occlusion transition, concept transition.

## Dialogue blocks and timestamp splitting

When writing dialogue and multi-shot segments intended to render as one video, upstream `consult.storyboard` uses two new structures. shot_split must recognize and expand them by rule:

### `### Dialogue Block NN: Title (about X seconds)` (declarative shot/reverse-shot)

A dialogue block represents a dialogue segment where Seedance 2.0 automatically covers shot/reverse-shot. Typical internal fields include: both sides and axis / relational blocking / coverage expectation / emotional arc / dialogue sequence / reaction anchors / negative constraints.

- **Default split granularity**: split into continuous N sub-shots based on the number of dialogue entries in "Dialogue Sequence" plus the number of key reactions in "Reaction Anchors". Each sub-shot duration is the dialogue block total duration divided evenly (e.g. total 12s, 4 lines of dialogue + 1 reaction anchor → 5 sub-shots of 2.4s each); if "Coverage Expectation" gives explicit durations (such as "wide start 3s → A medium-close 3s → B reverse 3s → back to wide 3s"), prioritize the explicit durations.
- **Shared fields**: copy the dialogue block's top-level `Both Sides and Axis / Relational Blocking / Emotional Trajectory / Negative Constraints` into corresponding fields of every sub-shot; do not let this information exist only on the first sub-shot.
- **Source marking**: add `dialogue_block_origin: Dialogue Block NN` to each sub-shot. Downstream presenters may choose to "merge back into one Seedance multi-shot call + `[00:00-00:04]` timestamps" or "split into N single-shot calls", but shot_split itself does not make this decision.
- **Axis conservation**: axis declarations like `A on the left side of frame, B on the right side of frame` inside the dialogue block must be written into each sub-shot's eyeline/spatial fields; do not let the model silently cross the line mid-sequence.

### `[00:00-00:05]` timestamp hard cuts inside shot blocks

When a `### Shot NN` block contains a set of `[00:00-00:05]` timestamp blocks in its "Action & Blocking" field (the syntax for one Seedance render containing multiple shots):

- **Default behavior**: split into continuous N sub-shots by timestamp count, where each sub-shot's `duration` = the difference of that timestamp block, and description = the original text of that timestamp block.
- **Total duration limit validation**: the total duration of the N sub-shots must be ≤ 15s (Seedance hard limit for one generation); if exceeded, flag in quality notes: "timestamp total duration X.Xs exceeds 15s and needs to return to storyboard consultation for segment splitting", and do not truncate on your own.
- **Source marking**: add `multi_shot_origin: Shot NN` to each sub-shot, serving the same role as the dialogue block marker.
- **Shared fields**: copy the original shot block's `Narrative Objective / Scene/Mood / Subtext/Setup / Eyeline & Space / Negative Constraints / Causality/Physics` into every sub-shot; assign dialogue, SFX, and reaction anchors to the specific sub-shot according to timestamp ownership.
- **Camera movement constraint conservation**: each sub-shot must have only one primary camera movement; if a timestamp says something like `push in first, then orbit`, flag in quality notes: "sub-shot contains two primary camera moves; recommend splitting in storyboard consultation", but still write it into structured output as-is.

### Mode B manual shot/reverse-shot (still uses the traditional single-shot path)

If the user uses `### Shot NN` (not a dialogue block) and there are no `[00:00-00:05]` timestamps, then convert shot by shot under the current conservation mode without further splitting. Manual shot/reverse-shot and declarative dialogue blocks may coexist in the same storyboard, and shot_split should handle each block according to its actual syntax.

## Cross-shot handoff fields

Upstream `consult.storyboard` writes a "Handoff" block in each shot. This is the highest-leverage field for keeping long-form 10+ shot narratives from drifting across shots. shot_split must parse it and write it into structured output as-is.

### Input field format

```
Handoff:
  Outgoing frame state: (character position / eyeline / hand pose / action completion / sound residue at the end of the previous shot)
  Incoming frame state: (the above elements at the first frame of the current shot)
  Transition type: match_on_action / eyeline_match / sound_bridge / j_cut / l_cut / graphic_match / concept_cut / hard_cut
  Repeated token: @char_main_xxx + @scene_xxx + ... (shared character/space/sound tokens, at least 1)
  Seedance Input Mode: text_to_video / image_to_video_first_frame / first_last_frame / ref_pack / nine_panel_grid_i2v / video_extend
  First-frame reference: @last_frame_of_shotNN / @char_main_xxx / @grid_main / ... (required for non-text_to_video modes)
```

### Write into structured output

Each shot's `handoff` field:

```json
{
  "handoff": {
    "previous_shot_outframe": "end frame of shot07 — Xiaomei pushes the door open and walks into the security room, body leaning forward...",
    "current_shot_inframe":  "the doorknob is still gripped by Xiaomei's left hand, the door is already open 80°...",
    "transition_type": "match_on_action",
    "repeated_tokens": ["@char_main_xiaomei", "@char_main_jiangye", "@scene_baoanshi", "@ambient_rain_indoor"],
    "seedance_input_mode": "image_to_video_first_frame",
    "first_frame_ref": "@last_frame_of_shot07"
  }
}
```

Add a top-level `seedance_call_mode` field to each shot (lifted from `handoff.seedance_input_mode` so the downstream router can read it directly):

```json
{
  "seedance_call_mode": "image_to_video_first_frame"
}
```

### Validation rules

- **The 1st shot may omit handoff**; from the 2nd shot onward, if handoff is missing, flag "shot N missing handoff block; cross-shot consistency falls back to hard_cut + text_to_video", but still write structured output using default values (`transition_type=hard_cut, seedance_input_mode=text_to_video`) without backfilling 5W content.
- **`transition_type` must be one of 8 enum values**: `hard_cut / match_on_action / eyeline_match / sound_bridge / j_cut / l_cut / graphic_match / concept_cut`; non-enum values should be flagged as "transition type X is non-standard" and fall back to `hard_cut`.
- **`seedance_input_mode` must be one of 6 enum values**: `text_to_video / image_to_video_first_frame / first_last_frame / ref_pack / nine_panel_grid_i2v / video_extend`; non-enum values should be flagged and fall back to `text_to_video`.
- **When `seedance_input_mode != text_to_video`, `first_frame_ref` is required**; if missing, flag "shot N mode X missing first-frame reference slot ID".
- **`repeated_tokens` must contain at least 1 token** (cross-scene hard cuts may drop space but character tokens must remain). If empty, flag "shot N and shot N-1 share 0 tokens; possible narrative jump".
- **Full-piece count of `transition_type=concept_cut`**: if more than 2 occurrences, flag "full piece has N concept_cut transitions; narrative may fall apart".
- **The slot referenced by `first_frame_ref`** must resolve either in global `reference_pack[]` or as `@last_frame_of_shotNN` (pointing to an existing shot ID); if unresolved, flag "shot N first-frame reference X failed to resolve".

### Seedance call routing recommendations (notes only, not enforced behavior)

Downstream presenters choose API call form based on `seedance_call_mode`:

| Mode | API call | Required input | Purpose |
|---|---|---|---|
| `text_to_video` | T2V | prompt only | Shot 1 / stylized empty shot / acceptable weaker consistency |
| `image_to_video_first_frame` | I2V (first frame) | 1 image + prompt | Same scene same subject + match_on_action (use previous shot's last frame as first frame) |
| `first_last_frame` | I2V (first + last frame) | 2 images + prompt | **Explicit A→B state transition** (transformation / reveal / exit-state to entry-state transition) |
| `ref_pack` | T2V/I2V + reference pack | prompt + 1-9 image + 1-3 video + 1-3 audio | Multi-character / multi-scene / default hard cut across scenes; identity anchor depends on `@char_main_*` turnaround sheet |
| `nine_panel_grid_i2v` | I2V (grid as input) | 1 nine-panel storyboard grid image + motion prompt | **In-segment storyboard compression**: compress 9 dense beats into one 15s continuous cinematic shot (instead of 6-8 small calls). **Not** an identity anchor across segments |
| `video_extend` | Video extend | previous segment video + prompt | **Extend the same plot in the same scene beyond >15s** (the model analyzes the whole trajectory, more stable than first_last_frame) |

**Key distinctions**:

- `nine_panel_grid_i2v`'s `first_frame_ref` should be a **segment-level grid** slot like `@grid_<segment_name>`, not a `@char_main_*` turnaround sheet. The former is for single-segment storyboard compression; the latter is for identity anchoring across segments.
- Between `image_to_video_first_frame` and `video_extend` for same-scene continuation, prefer the latter (trajectory analysis is more comprehensive); only use `first_last_frame` for explicit keyframe→keyframe transitions.

shot_split does not make this call for downstream. It only passes through `seedance_call_mode` and `handoff`.

## Spatial and continuity cleanup

- Organize according to the spatial logic already written in the confirmed storyboard; do not redesign camera positions.
- Within the same scene, characters' real positions are unchanged by default unless the text states movement.
- When displacement exists, preserve the starting point, direction, sense of distance, and landing point, e.g. "approaches the hospital bed from the doorway", "backs up to the table", "looks toward the hallway outside frame".
- For two-person dialogue and confrontation, preserve the original storyboard's left-right relation, eyeline direction, and height difference; when not specified in the original text, make only conservative inferences and do not create complex line-crossing.
- If adjacent shots clearly have position jumps, eyeline breaks, or disconnected action, only add minimal handoff description without changing the plot.
- Group shots must preserve the core visual focus; other characters should be handled as peripheral, background, or reaction group.
- When the original "Eyeline & Space" field contains an SDV vector expression like `<start anchor> → <X+/X-/Y+/Y-/Z+/Z-> → <end anchor>`, **store it as-is in the shot's `motion_vector` field** (string; do not rewrite into free description). This vector is used downstream by HDVP to lock the depth wall; if dropped, the directional logic of high-risk displacement is lost.
- When the original "Scene/Mood" field includes **terminator line position** and **cool/warm color temperature distribution**, store it as-is in the shot's `lighting_geometry` field (string). Rewriting or deleting it will make downstream lighting locks unstable.

## Dialogue, voiceover, and sound

### Dialogue parsing (protect the lip-sync three-part package)

Upstream `consult.storyboard` writes dialogue in the following format:

```
Character (emotion/tone) "short line"
```

Quotation marks may be `”...”` / `「...」` / `『...』`; treat them equivalently. Parsing rules:

- `character name` → `dialogue.speakerRef`
- `emotion/tone` → `dialogue.tone` (new field; preserve as-is; downstream Seedance uses it as a lip-sync strengthening signal)
- Text inside quotes → `dialogue.text`, **preserve the original quotation marks as-is** (Seedance recognizes quotation marks as a lip-sync trigger; remove them and lip-sync is lost)
- `dialogue.kind` defaults to `dialogue`; only switch to `voiceover / inner_monologue / offscreen` when encountering keywords like `VO / narration / inner OS / offscreen / offscreen voice / OS`, and **quotes may be omitted for these non-dialogue kinds**.
- Dialogue, voiceover, rhetorical questions, vows, dying words, confessions, judgments, insults, promises, theme lines, and psychological turning points in the input should be preserved as much as possible.
- Do not collapse `voiceover`, `offscreen`, and `inner_monologue` into ordinary spoken dialogue; if the original text says "thinks to herself", "inner OS", "heard from offscreen", or "the narrator says", this sound-source distinction must be preserved.
- Only fill `speakerRef` when it can be stably mapped to a character; if narration has no explicit character source, it may be left blank or use narrator semantics, but do not fabricate it as a spoken line from some character.

### `source` and rewrite constraints

- `source` reflects origin: use `verbatim` for word-for-word preservation of original text, `adapted` for slight rewrites due to speakability capacity, and `new_bridge` for short bridging lines newly added between shots.
- **lip-sync dialogue (kind=dialogue and quoted) does not allow `adapted` rewrites**; it may only be `verbatim` or `new_bridge`, and `new_bridge` must satisfy the lip-sync three-part package (quotes + emotional prefix + ≤12 Chinese characters / 8 English words). If rewriting is needed, instead flag in quality notes: "dialogue X exceeds capacity; recommend shortening in storyboard consultation", rather than silently editing it.
- **Any output with `source=new_bridge` must be forcibly flagged in quality notes** as: "shot N adds bridging dialogue line '...' (`new_bridge`) without user confirmation; if it should be kept, explicitly confirm it in consult.storyboard". `new_bridge` is a low-frequency path and must not be silently attached; any dialogue already written by upstream `consult.storyboard` always uses `verbatim`.
- `voiceover / inner_monologue / offscreen` types may use `adapted`, but core rhetoric must not be swallowed.

### Capacity

- Long original passages may be split into continuous voiceover montage or multiple adjacent shots, but core rhetoric must not be summarized away.
- Ordinary Chinese is about 4-5 characters/second; heavy emotion, weakness, crying voice, or hesitation is about 3-4 characters/second; quarrels and rapid information delivery are at most about 5-6 characters/second; extremely fast delivery should be used only for special style.
- When estimating speaking time, deduct action, pauses, reactions, ambient sound, and emotional aftertone.
- If confirmed storyboard dialogue is overloaded, do not delete or rewrite it on your own; prioritize splitting, extending duration tendency, or marking overload.
- Additional lip-sync validation: text inside a single pair of quotes must be ≤ 12 Chinese characters / 8 English words; if exceeded, flag it in quality notes.

### Sound three-sublayer mapping

Upstream `consult.storyboard` writes sound in three sublayers:

```
Sound:
  SFX: (action-triggered sounds, object-related sounds, timestamp anchors)
  Ambient: (continuous scene ambient)
  Silence/Aftertone: (length and position of silent beats)
```

shot_split must map these three sublayers into separate structured fields:

- `SFX:` → `audio.sfx[]` (array, may carry timestamp anchors)
- `Ambient:` → `audio.ambient` (string, describes continuous scene ambience)
- `Silence/Aftertone:` → `audio.silence[]` (array, each item includes position + duration)

**Do not merge the three sublayers into a single field**. The downstream native Seedance audio path consumes them by channel. If the original text only vaguely says "sound" without sublayers, classify by content instead of stuffing everything into one field.

- Even in shots without dialogue, preserve ambient sound, action sound, or silence basis; do not default-add strong music.

## Duration and granularity

- Recommended single-shot duration is only a reference for split granularity, not a fixed length. Do not organize the whole piece into mechanically uniform durations.
- If the confirmed storyboard provides durations, preserve them first; if not, estimate based on visual action, sound text, and emotional aftertone.
- Establishing shots, evidence close-ups, ultra-short reactions, strong fast cuts, sound bridges, and transitional empty shots may be shorter than the recommended duration, but must have a clear function.
- If a plot beat can be clearly expressed within the recommended duration, do not split it just to increase shot count.
- If dialogue capacity, spatial change, emotional reversal, or result landing point exceeds what one shot can carry, split it into continuous shots.

## Asset references

- Only reference stable names for characters, scenes, and props that already exist in the current project.
- When a bound character or scene appears, prioritize the project's stable name instead of replacing it with free description.
- Character references are stricter than scene/prop references: any character participating in the current shot must be referenced, otherwise downstream video tasks cannot force retrieval of that character's turnaround/main view.
- Temporary objects required by the plot but not present in the project should remain only in the description and must not be disguised as assets.
- Unusual states must be preserved in the description: critically ill, battle-damaged, ruined, withered, burned, impoverished, severely injured, polluted, abandoned, etc. must not be automatically beautified.
- The top `## Character Anchors` manifest coexists with project assets: when the project has a stable asset for that character, put the asset name in `characterRef` and the anchor string in `anchor_descriptor`, and send both downstream together; when the project does not, **use `anchor_descriptor` as a temporary placeholder in `characterRef`** so downstream Seedance can still receive a consistency token, and flag in quality notes: "character X lacks project asset; using temporary anchor placeholder".
- Do not drop a character from the shot just because "the project has no asset for this character" — anchor placeholders take priority over "strict asset matching".

## Downstream constraints and meta field passthrough

The upstream `consult.storyboard` fields `Negative Constraints` / `Causality/Physics` / `Emotion Intensity` / `Subtext/Setup` previously had no corresponding handling in shot_split. New rules:

| Source field | Structured output field | Requiredness |
|---|---|---|
| `Negative Constraints` | `negative_constraints` (string array, split by `/` or comma; recognize both full-width `｜` and half-width `\|`) | **Must preserve**, downstream Seedance prompt will append it to the negative slot |
| `Causality/Physics` | `meta.causal_physics` (string, as-is) | Must be preserved for shots with high physical interaction; downstream Seedance uses it as a WHY signal |
| `Emotion Intensity` | `meta.emotion_intensity` (string, as-is) | Preserve; no strong downstream consumption |
| `Subtext/Setup` | `meta.subtext` (string, as-is) | Preserve; for later revision-stage reference |
| SDV vector in `Eyeline & Space` | `motion_vector` (string, as-is) | Must preserve for high-risk displacement fields |
| Terminator line + cool/warm color temperature distribution in `Scene/Mood` | `lighting_geometry` (string, as-is) | Must preserve for named lighting fields |
| `Reference Pool References` | `reference_refs[]` (array, each item `{slot, type, purpose}`) | Must preserve for characters/scenes with stable project assets; downstream Seedance routes into ref slots |
| Entire `Handoff` block | `handoff` (object, see § cross-shot handoff fields) | Must preserve from shot 2 onward; flag if missing but do not backfill |
| `Seedance Input Mode` inside `Handoff` | `seedance_call_mode` (string, top-level field so downstream router can read directly) | **Must preserve** |

**Do not rewrite, merge, or compress** these fields. If these fields are empty in the original text, leave the corresponding structured output fields empty as well; do not invent values.

### `fast` keyword flag scope (restricted)

When checking the `fast` keyword, shot_split should **only flag within the following two field scopes**:
- the full `Shot Size/Angle/Camera Movement` field
- the camera-movement portions of the `Action & Blocking` field (`camera fast push-in`, `fast tracking shot`, etc.)

**Do not flag within the following scopes** (these are valid plot/physiological descriptions and do not affect Seedance camera movement):
- character descriptions (`he runs faster than last time`, `heart rate accelerates`)
- voiceover / dialogue content (`"come in quickly"` as lip-sync dialogue)
- causal/physical logic (`fast-flowing water` describing the physical property of water flow)

When flagging, give the specific occurrence location rather than a whole-text flag.

### Subject-first flag

shot_split does not rewrite the original storyboard sentence order, but should flag phrasing in quality notes that commonly triggers Seedance 2.0 anti-patterns such as "character popping in / inter-shot fusion". Check the first sentence of the `Visual Narrative` / `Action & Blocking` fields:

- **Environment-first**: if the first sentence starts with an environmental phrase like "bright X / sunlight / cold light / warm light / the whole room / a whole stretch of" and then follows with "the camera pushes in toward X / the camera pans across to X", flag "shot N first sentence is environment-first; Seedance 2.0 is prone to character pop-in. Recommend returning to consult.storyboard and changing to subject-first (state subject position + action first, then environment)."
- **Passive reveal language**: if it uses "reveals X / finally lands on X / it turns out X is standing / X suddenly appears / X flashes into view / there is now X in frame", flag "shot N uses passive reveal language; subject may be revealed too late".
- **Carry-over character absent**: if the same character from the previous shot (already bound in `character_refs`) is not explicitly present in the first sentence of this shot (no position/state description), flag "shot N carry-over character X is not restated with position/state in the first sentence; inter-shot fusion is possible".
- **Newly entering character lacks entry direction**: if a character first appearing in this shot (not in previous shot `character_refs`) has no direction/starting position in description (`enters from the left side of frame` / `at the lower-right corner of frame`), flag "shot N new entering character X lacks entry direction/starting position".

shot_split **must not rewrite** these fields on its own. Write them into description as-is; only flag in quality notes so the user can decide whether to revise in consult.storyboard.

### Focal-length jargon flag

If `Visual Narrative` / `Action & Blocking` / `Shot Size/Angle/Camera Movement` contains technical jargon like `85mm / f/1.4 / 35mm focal length / shallow DOF / focal plane / shallow depth of field / focal length / aperture / depth of field`, flag "shot N contains focal-length jargon; Seedance does not respond to it and may even degrade. Recommend changing to `wide-angle spatial feel / natural perspective / portrait compression feel / macro texture + frame occupancy`." Write the original text into description without rewriting.

## Quality standards

- Do not miss user-confirmed shots, dialogue blocks, dialogue, voiceover, sound, scenes, characters, or key props.
- Shot order must match the confirmed text; any split must only break an overloaded shot/dialogue block/timestamp block into continuous sub-shots without changing the meaning.
- Every shot must have clear visual action, narrative objective, direct result, and next-shot handoff point.
- Adjacent shots must not have characters suddenly changing position, eyelines suddenly reversing, dialogue suddenly starting, emotions suddenly jumping, or space suddenly breaking.
- Dialogue/voiceover capacity must match duration tendency; overload must be clearly marked or reasonably split.
- For lip-sync dialogue (`kind=dialogue` + quoted): quotation marks are preserved as-is, `tone` is filled, each line is ≤ 12 Chinese characters / 8 English words, `source` is not `adapted`; `source=new_bridge` has been strongly flagged.
- Asset references must truly exist; do not fabricate stable names that do not exist in the project; characters lacking project assets use top-level `anchor_descriptor` placeholders and are flagged.
- For the three cross-shot manual sections: all characters in the top `Character Anchors` are referenced by at least one shot; `Continuity Ledger` is stored in global metadata `continuity_ledger` (including the three new fields `prop_state_machine / lighting_matrix / sound_bridge_plan`), and no shot conflicts with the ledger; `Visual Reference Pool` is stored in global metadata `reference_pack[]`, and slot limits of 9 image / 3 video / 3 audio are validated.
- **Video segment aggregation**: every `## Video Segment X/N: ...` title line has been parsed into `video_segments[]`; every shot has top-level `video_segment_id`; each segment's `actual_duration` ∈ [12, 15] by default or user-explicit range; any segment > 15s has been flagged; segment numbering increases continuously and matches N; every non-final segment of the full piece has `ending_handoff`. When video segment aggregation is missing, fallback flags have been applied and shot-level handoff preserved without inventing segments.
- If the input uses single-line compact manifest format (`Character Anchors: A=...;B=...` / `Reference Pool: @xxx=...`), it has **prompted switching to the sibling skill** rather than force-parsing.
- Dialogue blocks and timestamp hard cuts: dialogue blocks are split into sub-shots by dialogue sequence + reaction anchors, timestamps are split into sub-shots by timestamp count; sub-shots all carry `dialogue_block_origin` / `multi_shot_origin` markers; total timestamp duration is ≤ 15s.
- **Cross-shot handoff (`handoff`)**: from shot 2 onward every shot has a `handoff` block; `transition_type` / `seedance_input_mode` are both valid enum values; `repeated_tokens` contains at least 1 token; `first_frame_ref` is filled and resolvable for non-`text_to_video` modes; full-piece `concept_cut` occurrences are ≤ 2; top-level `seedance_call_mode` has been lifted from handoff.
- Passthrough fields: `negative_constraints` / `motion_vector` / `lighting_geometry` / `meta.causal_physics` / `meta.emotion_intensity` / `meta.subtext` / `reference_refs[]` / `handoff` / `seedance_call_mode` are all written into structured output from the original text, with no rewriting, merging, or compression.
- `fast` flag scope is restricted to the `Shot Size/Angle/Camera Movement` field and the camera-movement portions of the `Action & Blocking` field; uses of fast in plot/physiology/dialogue are not falsely flagged.
- Sound: the three sublayers SFX / ambient / silence are written separately into `audio.sfx[]` / `audio.ambient` / `audio.silence[]`, not merged into one field.
- The output can continue to be consumed by later storyboard-image, video-path, and prompt-package stages without needing to guess the meaning of the original storyboard.

## Failure fallback

- If the input does not look like confirmed storyboard text, provide a coarse-grained structured split and note that the user should first go to storyboard consultation to improve directorial expression.
- If original spatial relationships are unclear, keep it simple: establishing shot -> medium-shot action -> close-up evidence or reaction -> result landing point.
- If original dialogue is too long for the duration, split it into continuous voiceover or dialogue shots; do not swallow key lines.
- If asset matching is uncertain, leave it blank; do not invent asset names for completeness.
- If the top cross-shot manual is missing (any of character anchors / continuity ledger / visual reference pool), continue under the current conservation mode, but flag in quality notes: "missing cross-shot manual X; cross-shot consistency may degrade; recommend supplementing it in consult.storyboard". If all three are missing, flag them together and note: "this is a required installation item outside Tier 1 ≤3-shot short segments".
- If video segment aggregation is missing (the input has no `## Video Segment X/N:` title lines), apply the fallback in § when video segments are missing (compatibility for old storyboards): flag "input missing video segment aggregation; cross-segment consistency falls back to shot-level handoff" + do not group on your own + recommend "return to consult.storyboard and add `## Video Segment X/N:` title wrappers around shot blocks according to § video segment aggregation rules". If video segment duration deviates from the default 12-15s and the user did not explicitly specify a different duration, flag it but still write output from the original text without merging/splitting on your own.
- If the input top contains single-line compact manifest like `Character Anchors: A=...;B=...` or `Reference Pool: @xxx=...;@xxx=...`, **immediately stop** structuring and prompt: "The input uses compact manifest format and should be handled by sibling skill `lumina-flow-shot-split-scene-as-shot`."
- For cases such as timestamp total duration > 15s, unclear semantics in dialogue block "coverage expectation", two primary camera moves in one shot, or lip-sync dialogue exceeding 12 characters, **do not repair on your own**. Write structured output from the original text + flag in quality notes, letting the user decide in consult.storyboard.
- When the handoff block is missing (from shot 2 onward) / transition type is not an enum value / Seedance input mode is not an enum value / non-`text_to_video` mode lacks first-frame reference / repeated token is empty / `first_frame_ref` fails to resolve, **do not repair on your own**. Write structured output with default values (`hard_cut + text_to_video`) + flag in quality notes, letting the user decide in consult.storyboard.
- If a long narrative has ≥10 shots but the top reference pool lacks the two minimum slots `@char_main_*` + `@scene_*`, continue under the current Tier 1 conservation mode and flag: "10+ shot long-form narrative lacks the Tier 3 reference-pool baseline; expected cross-shot consistency is below 60%; strongly recommend returning to consult.storyboard to add a 9-panel grid plan and reference pool slots".