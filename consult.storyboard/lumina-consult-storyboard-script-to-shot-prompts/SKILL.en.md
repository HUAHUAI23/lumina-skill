---
name: lumina-consult-storyboard-script-to-shot-prompts
description: Supplemental prompting for consult.storyboard; converts Chinese scripts/novels/dialogue/prose into compact storyboard text with a two-level "scene + shot" structure, where each scene is 12–15 seconds and consists of several one-line shots that can be sent directly to story_to_video.
---

# consult.storyboard / script-to-shot-prompts

Convert Chinese scripts/novels/dialogue/prose into a compact storyboard **grouped by scene**: each scene is 12–15 seconds and contains several one-line shots that can be executed directly by video/comic generation models.

---

## ⚠ Output Contract (overrides upstream prompt)

**This skill's output format takes priority over any wording in the system prompt such as "Markdown storyboard text," "project metadata," or "fielded storyboard subsections."** If there is any conflict, always follow this skill.

### Forbidden — if any appear, rewrite the entire output immediately

- Any project-level metadata block, for example:
  `## Project`, `**Title:**`, `**Clip Range:**`, `**Total Duration:**`, `**Style Tone:**`, `**Shot Rhythm:**`, `**Performance Principles:**`, `**Storyboard Strategy:**`, `## Spatial Anchors`
- Any fielded subsection, including but not limited to:
  `**Narrative Goal:** / **Emotional Intensity:** / **Scene/Atmosphere:** / **Subtext/Foreshadowing:** / **Visual Narrative:** / **Action & Blocking:** / **Shot Size/Angle/Camera Movement:** / **Eyeline & Space:** / **Dialogue/Voiceover:** / **Sound & Silence:** / **Editing Continuity:**`
- Shot subheadings like `### Shot 01: xxx (about X seconds)`
- Multi-action blocks like `m1: ... / m2: ...`
- Using bullet lists (`- xxx`) or multiple paragraphs to describe a single shot
- Any meta-commentary paragraphs (sound design, lighting notes, edit continuity, foreshadowing, unified markers, style tone, etc.)
- `---` horizontal separators
- Ending summary / next-episode teaser / production notes
- Using `says:` as a narrative lead-in for dialogue inside a shot — Seedance treats it as noise rather than a lip-sync trigger; quotation marks are the trigger (see dialogue rules below)

### Required — must be followed

A valid output may contain **only** these six line types:

1. (Optional) A single top title line `# Storyboard Text`, omitted by default unless the user explicitly requests it.

2. (Optional) A top `Character Anchors` block: **write only once**, before all scenes, one character per line, in this format:
   `Character Anchors: Xiaomei=pink puff-sleeve knit top/black short skirt/black short straight hair at jaw length/small mole on right brow bone; Jiang Ye=black security uniform/buzz cut/helix stud on left ear/thin black cord bracelet on right wrist`
   When that character appears in later shots, **reuse the existing appearance tokens from the anchor directly**; do not expand with new appearance descriptions in shot lines. Downstream AI video models rely on token-level repetition for cross-shot consistency. Repetition is the protocol.

3. (Optional) A top `Reference Pool` single-line block: place after `Character Anchors`, declaring the 12 slots of Seedance Universal Reference:
   `Reference Pool: @char_main_xiaomei=Xiaomei three-view sheet;@char_main_jiangye=Jiang Ye three-view sheet;@scene_securityroom=security room key image;@lighting_securityroom=cool white + red strobe lighting field;@prop_keycard=keycard close-up;@ambient_rain=indoor rain ambience 5s loop;@bgm_tension=tension BGM 8s`
   - Slot naming: `@<purpose>_<name>`, where purpose is one of `char_main` / `char_outfit` / `char_face` / `scene` / `lighting` / `prop` / `style_palette` / `motion` / `transition` / `ambient` / `bgm` / `voice`
   - Limits: 9 image + 3 video + 3 audio, 12 total; total video/audio duration each ≤15s
   - **Reference Pool is recommended for ≥4 shots; mandatory for long narratives of 10+ shots**; every shot line containing a mainline character/scene must explicitly reference it in the description with `(@char_main_xxx)`
   - If the project has no matching asset, omit that slot line; the downstream flow will flag it

4. **Scene title line** (one line per scene, with a blank line before it):
   `Scene X Scene Name Xs`
   - `Xs` is the sum of all shot durations in that scene, and must be within **12–15 seconds** (unless the user specifies otherwise)
   - Keep scene names short, e.g. `Forest Park Kindergarten - Outdoor Activity`, `Outside the Security Room`, `Bedroom - Phone Lights Up`
   - You may optionally append transition info from the previous scene after the title: `Scene X Name Xs Transition:type(token1+token2)`, for example `Scene 2 Security Room - Identity Reversal 13s Transition:sound_bridge(rain+@ambient_rain)+match_on_action(Jiang Ye's hand still clamps Xiaomei's throat)`
   - Transition type enum: `hard_cut / match_on_action / eyeline_match / sound_bridge / j_cut / l_cut / graphic_match / concept_cut`
   - The first scene has no previous scene, so do not write a transition; from the second scene onward it is **strongly recommended**, and mandatory for long narratives of 10+ shots

5. **Shot line** (under each scene title, one shot per line, with a blank line between lines):
   `**Shot X X.Xs Time: ... [Shot Size/Camera Movement]** Description`
   - Complete everything on one line: visible action + emotion + prop/environment
   - When a mainline character/main scene appears in the description, **inline-reference the Reference Pool slot directly**: `Xiaomei(@char_main_xiaomei) pushes open the door of the security room(@scene_securityroom)`; the downstream flow will extract these `@xxx` into `reference_refs[]`
   - If there is dialogue, append directly as `Character (emotion, action) "line"` (the native 2026 audio model recognizes quotation marks as lip-sync triggers; see dialogue rules below)
   - **Shot numbering must increase continuously throughout the whole piece** (do not restart from 1 in each scene)
   - Each shot may have **only one primary camera movement**, and the keyword **`fast` is forbidden only in the `[Shot Size/Camera Movement]` field and in camera-movement phrasing** (Seedance 2.0 official docs specifically note that it causes artifacts); for faster pacing, use `quick whip pan / sharp turn / sudden cut` and move "fast" to the editing layer
   - Descriptions like `heartbeat quickens` / `he runs faster than last time` are allowed uses of "fast" in story/physiology contexts (the downstream flag only applies to camera-movement fields)
   - **Do not write technical jargon like focal length/aperture/depth of field/85mm/f/1.4** (Seedance does not respond to these parameters); for texture, use natural phrasing like `medium close-up / practical tungsten overhead`
   - You may optionally append negative constraints at the end of the line in this format: `｜Avoid:no subtitle bars/no background extras/no brand logo` (omit if there is no boundary risk; both full-width `｜` and half-width `|` are accepted, and downstream will normalize them)

6. (Optional) **Transition line below a shot line** (only when image_to_video / first_last_frame mode is needed):
   `Transition:type Mode:seedance_input_mode FirstFrame:@slot Repeat:token1+token2`
   - Example: `Transition:match_on_action Mode:image_to_video_first_frame FirstFrame:@last_frame_of_shot07 Repeat:@char_main_xiaomei+@scene_securityroom`
   - If omitted, default is `hard_cut + text_to_video` (downstream parses it this way); **strongly recommended for every shot in Tier 2/3 long narratives**

Nothing else may appear — no paragraphing, no extra headings, no bullets, no ending summary, no `---`.

---

## Core Output Format

```text
Character Anchors: Tuantuan=white fluffy rabbit/yellow overalls/white little boots/red nose round eyes; Lulu=pink rabbit/purple dress/pink bow on long ears; Doudou=brown squirrel/green vest/fluffy large tail
Reference Pool: @char_main_tuantuan=Tuantuan three-view sheet;@char_main_lulu=Lulu three-view sheet;@char_main_doudou=Doudou three-view sheet;@scene_playground=kindergarten playground key image;@lighting_daylight=5600K high-key daylight overhead;@ambient_kids=children playing ambience 5s loop

Scene 1 Forest Park Kindergarten - Outdoor Activity 14s

**Shot 1 3.0s Time: Daytime. [Wide Shot/slow pan]** Sunlight fills the kindergarten playground(@scene_playground), with 5600K high-key daylight overhead(@lighting_daylight); the green lawn, colorful small slide, flowerbeds, and bulletin board unfold in sequence, children play freely on the grass, and dandelion seeds drift gently past.

**Shot 2 2.5s Time: Daytime. [Medium Shot/static]** Lulu(@char_main_lulu) crouches by the flowerbed and picks a dandelion, her pink bow swaying lightly; she puffs her cheeks and blows, and the white fluff drifts apart slowly.
Transition:eyeline_match Mode:text_to_video Repeat:@scene_playground+@ambient_kids

**Shot 3 2.5s Time: Daytime. [Medium Shot/push-in slow]** Doudou(@char_main_doudou) climbs to the top of the small slide, his green vest bouncing, tail raised; once seated, he puffs out his chest like a little hero fan ready to speak.
Transition:hard_cut Mode:text_to_video Repeat:@scene_playground+@ambient_kids

**Shot 4 3.0s Time: Daytime. [Medium Shot/tracking smooth]** Tuantuan(@char_main_tuantuan) spins in circles chasing his own ears, slips and lands on his bottom, then immediately gets back up rubbing it with a goofy grin.
Transition:hard_cut Mode:text_to_video Repeat:@scene_playground+@ambient_kids

**Shot 5 3.0s Time: Daytime. [Insert Shot/static]** The bulletin board reads "Today's Snack: Carrot Cookies"; beside it is a childish hand-drawn "Lightning Rabbit" hero poster, with crooked little stars sketched in the corner.
Transition:hard_cut Mode:text_to_video Repeat:@scene_playground

Scene 2 Playground Corner - Alien Spaceship Arrival 13s Transition:sound_bridge(@ambient_kids continues+purple light surge SFX enters 0.4s early)

**Shot 6 2.0s Time: Daytime. [Long Shot/tilt up]** A streak of purple light suddenly flashes across the clear sky, leaving a brief purple edge on the clouds.

**Shot 7 2.5s Time: Daytime. [Close-up/static]** All three look up at once; Tuantuan blinks once → parts his lips slightly → locks his gaze on the purple light in the sky; Lulu instinctively hugs the dandelion, and Doudou's mouth curves upward.

**Shot 8 3.0s Time: Daytime. [Medium Long Shot/tracking smooth]** A silver mini spaceship flies in low, toy-like and rounded, gliding slowly above the playground, circling in the corner with a faint purple glow trailing from the rear.

**Shot 9 3.0s Time: Daytime. [Medium Close-up/static]** Doudou tilts his head up at the spaceship and bounces twice in place with excitement. Doudou (excited, hopping) "It's a spaceship, real aliens"

**Shot 10 2.5s Time: Daytime. [Reaction Shot/static]** Tuantuan hears "Bunny Bolt," blinks once → swallows lightly → presses his lips together awkwardly for 0.5s, briefly avoids eye contact, then quickly returns to normal.｜Avoid:no real brand logos
```

---

## Scene Splitting Rules / Scene Splitting Rules

### When to start a new scene

Start a new scene if any of the following is true:

- **Space changes**: playground → classroom → behind the tree → inside the spaceship
- **Time jumps**: daytime → night, present → flashback
- **Strong narrative beat switch**: everyday → anomaly appears, calm → crisis, reveal reversal, hero entrance

### Single scene duration

**A "scene" = 1 Seedance call = what the user calls "1 video"**, internally composed of multiple shots.

- **Default is 12–15s** (per scene), carried by 4–7 shots; stay near the 15s upper bound to maximize output per call
- **If the user does not explicitly specify duration, always use the default 12–15s; do not ask "how long?"**
- If the user explicitly specifies duration (e.g. "one video should be 8–10s" / "each segment 6 seconds" / "each scene 12s"), follow the user's value and recalculate shot count and scene splits accordingly
- If duration exceeds the single-scene limit, split it into two scenes, e.g. `Scene 3a Hiding Behind the Tree 7s` + `Scene 3b Lightning Transformation 8s`, rather than cramming into one scene

| User prompt | Scene duration |
|---|---|
| No duration mentioned | **Default 12–15s** (near upper bound) |
| "One video 8–10 seconds" | 8–10s |
| "Each segment 6 seconds" | 6s |
| "Each video no more than 10s" | ≤10s (use 8–10s) |
| "Each scene 12s" | 12s |

### Scene naming

Use `Location-Action` or `Location-Emotion Anchor`:

- ✅ `Forest Park Kindergarten - Outdoor Activity`
- ✅ `Security Room - Crisis Erupts`
- ✅ `Bedroom - Phone Lights Up`
- ❌ `Opening`, `Scene One`, `First Segment` (too vague)
- ❌ `Tuantuan Lulu and Doudou play on the sunny kindergarten playground` (too long)

---

## Shot Line Rules / Shot Line Rules

Each line must contain the following, and **must use only one line**:

1. `**Shot X` (continuous numbering across the whole piece, not reset per scene)
2. ` X.Xs ` (duration)
3. `Time: XX.` (daytime/night/dusk/pre-dawn, etc.)
4. ` [Shot Size/Camera Movement]**` (bold closes after the shot tag)
5. A space + visible action description / emotion / props
6. (Optional) dialogue: `Character (emotion, action) says: "line"`

### Suggested durations (single shot)

- Establishing/environment shot: 1.5–2.5s
- Expression close-up: 1.0–1.5s
- Simple insert shot: 0.8–1.2s
- Dialogue line: 2.0–3.5s
- Sudden action: 0.5–1.2s
- Reversal/closing beat: 1.5–2.5s

### User-specified duration overrides

If the user says "each shot 5 seconds" or "each scene 30 seconds," **use the user's values directly and ignore the defaults**, then recalculate shot count and scene splitting accordingly; the line format stays the same, and long shots should carry more action/reaction/dialogue compressed into the same line.

---

## Conversion Formula

*(internal heuristic — guides reasoning; never appears in output)*

Source text → output:

> Scene + character setup + appearance + action + expression + tone + dialogue + reversal
> ↓
> Scene title (space + emotional anchor, 12–15s)
>   └ Shot number + duration + time + shot size/camera movement + visible action + emotion + dialogue + reaction/anomalous detail

Practical workflow:

1. **Rough-cut scenes**: split the source text into multiple 12–15 second segments by space/time/beat
2. **Extract scene info**: time, place, interior/exterior, lighting/weather
3. **Extract characters**: names, appearance, emotion
4. **Extract visible action**: posture, hands, movement path, turning, gripping
5. **Extract dialogue**: split long dialogue into multiple dialogue shots when necessary
6. **Add insert shots**: hands, locks, surveillance, phones, shadows, reflections, props
7. **Add reaction shots**: after every key line or action, give the other side a facial/body reaction
8. **Check total duration**: the sum of all shot durations in each scene must be 12–15s (or user value)

---

## Shot Ordering Pattern (within a scene)

*(internal heuristic)*

Short suspense/reversal rhythm:
Establish → character enters → expression close-up → dialogue → reaction → insert → action → reversal close-up → decisive line → close

---

## Camera Vocabulary

Shot size dictionary (choose one): `Extreme Wide Shot / Wide Shot / Long Shot / Medium Shot / Medium Close-up / Close Shot / Close-up / Extreme Close-up / Reaction Shot / Insert Shot / POV Shot`

Primary camera movement dictionary (**choose only one**, do not mix): `static / push-in / pull-out / pan / tilt / tracking / orbit / handheld / low-angle rush / slow push-in`

Rhythm adverbs (optional, appended after primary movement): `slow / smooth / stable / gradual / gentle` (officially recommended by Seedance 2.0) — **`fast` is strictly forbidden** (officially noted to degrade output).

Shot texture phrases (optional, place in the description to replace focal-length jargon): `wide-angle sense of space / natural perspective / portrait compression / macro texture / background blur / subject occupies 60% of frame / face occupies 40% of frame / foreground 1/3 + subject in medium shot`. **Forbidden:** technical jargon such as `85mm / f/1.4 / shallow depth of field / focal plane / focal length` (Seedance does not respond).

Common shorthand combinations:

- `[Wide Shot/slow pan]` — establish scene
- `[Long Shot/static]` — isolate character in a larger environment
- `[Medium Shot/push-in slow]` — introduce character or add pressure
- `[Medium Close-up/static]` — dialogue with body language
- `[Close-up/static]` — face/eyes/mouth/hands/props
- `[Close-up/push-in gradual]` — emphasize key line or clue
- `[Reaction Shot/static]` — listener's immediate emotional reaction
- `[Insert Shot/static]` — button, door lock, phone, ID card, shadow, reflection
- `[Quick Cut/handheld shake]` — sudden action, attack, chaos (use `quick whip`, not `fast`)
- `[Silent Shot/static]` — emotional freeze or horror pause
- `[Effects Shot/effects presentation]` — supernatural eyes, distorted shadow, halo, glitch
- `[tracking smooth]` / `[reverse medium close-up/static]` / `[POV Shot/static]` / `[orbit slow]` / `[tilt up/static]` / `[low-angle rush]`

### Multi-shot timestamp syntax (Seedance 2.0 one-pass multi-shot output)

A single Seedance 2.0 generation can output a multi-shot edited sequence (total duration ≤15s), and it recognizes `[00:00-00:05]` as a hard-cut instruction. When a "shot line" is actually carrying a dialogue block or action sequence (with the model generating multiple sub-shots automatically), **embed timestamped hard cuts inside a single line**:

```
**Shot 7 12.0s Time: Night. [Dialogue Block/medium two-shot start]** A stands by the door, B sits by the table, rain taps the glass. [00:00-00:02] medium two-shot;[00:02-00:05] cut to A medium close-up, A blinks and swallows lightly, A (probing, low) "Did you go that day";[00:05-00:08] cut to B reverse medium close-up, B half-smiles with 1.2s silence, B (guarded, half-smiling) "Go where";[00:08-00:12] back to wide, A turns and walks away, B's gaze follows to frame-right exit.
```

Total timestamp duration must be ≤15s (single-generation limit of Seedance), with no gaps and no overlap, and each segment may have only one primary camera movement. Simple single shots do not need timestamps; just write them in one line as `**Shot X X.Xs Time: ... [Shot Size/Camera Movement]**`.

---

## Dialogue Rules

Every shot containing dialogue must also contain visible action or emotional buildup on the same line, and must use the **lip-sync three-part pattern** so Seedance 2.0 can produce phoneme-level aligned lip sync in a single forward pass (supports 8+ languages):

1. **Quotation marks**: wrap dialogue directly in `"..."` (do not use narrative lead-ins like "says:"; Seedance recognizes quotation marks as the lip-sync trigger).
2. **Emotion/tone prefix**: place it in parentheses before the quotation marks, e.g. `Jiang Ye (playful, soft) "Reward"` / `B (shaking, low) "Don't come closer"`. Atlabs testing shows dialogue with emotion prefixes has significantly better lip-sync accuracy than bare dialogue.
3. **Short lines**: each line of dialogue ≤ 12 Chinese characters / 8 English words. Split long lines into multiple dialogue shots; the simpler the syllables, the more accurate the lip sync.

Weak (gets everything wrong):
```text
**Shot 4 3s Time: Night. [Medium Close-up/static]** Jiang Ye says: "Reward?"
```
Problems: uses `says:` as a narrative lead-in (Seedance treats it as noise instead of a lip-sync trigger), lacks an emotion prefix, and the shot is too long.

Strong (all three parts present):
```text
**Shot 4 1.5s Time: Night. [Close-up/static]** Glass reflections obscure half of Jiang Ye's face; the corner of his mouth lifts slowly into an unreadable smile. Jiang Ye (playful, soft) "Reward"
```
Key points: no `says:` lead-in, has emotion prefix `(playful, soft)`, dialogue is ≤12 Chinese characters, and quotation marks are the lip-sync trigger.

After a key line, add a reaction shot:
```text
**Shot 5 1.5s Time: Night. [Reaction Shot/static]** Hearing those two words, Xiaomei's smile turns sweeter, with a hint of triumph in her eyes, as if she thinks Jiang Ye has already taken the bait.
```

Voiceover / VO / inner monologue / off-screen voice should **not** use quotation marks or the lip-sync three-part pattern; use book-title brackets or narrative sentences instead, to avoid Seedance misreading it as spoken lip-sync dialogue and causing mismatched mouth movement:

```text
**Shot 2 2.0s Time: Night. [Medium Close-up/static]** Chen Mian stares at the phone screen, half her face lit by cold light, her breathing stopping for a beat. Voiceover (Chen Mian VO, low, as if dragged out by memory): Seven years ago, the last time she called me, she said this too.
```

---

## Subject-First Rule / Subject-First Rule

Each shot line description must **start with "shot + subject position + subject action"**, then add environment/lighting/set dressing/props. **Seedance 2.0 allocates attention heavily based on "appears first = established first."** "Environment first + camera pushes in to X" is a high-frequency anti-pattern that triggers "character suddenly appears / frame fusion mismatch."

Observed comparison (user-tested):

```text
# Bad example A (environment first, character may pop in)
**Shot 1 3.5s Time: Daytime. [Medium Wide Shot/push-in slow]** The bright forest cabin living room(@scene_xxx) has
returned completely to order, sunlight spills across the wooden floor, the storage basket(@prop_xxx) sits neatly in place, and the camera slowly pushes in at eye level toward Dudu(@char_main_dudu) standing
in the center with his family around him.

# Good example B (subject first, character established stably)
**Shot 1 3.5s Time: Daytime. [Medium Wide Shot/push-in slow]** Dudu(@char_main_dudu) stands in the center of the frame,
with Mom, Dad, Lili, and Grandpa Yantu positioned around him as the camera slowly pushes in at eye level; the bright forest cabin living room(@scene_xxx)
has been fully restored, sunlight spreads over the wooden floor, and the storage basket(@prop_xxx) is filled with toys neatly put away.
```

Writing rules:

1. **List the subject first in the first sentence of every shot**: `<subject>(at X position in frame) + <action/state>`, then write `<scene>` + `<lighting>` + `<props>`.
2. **Carry-over characters**: for characters carried across from the previous shot, the first sentence of the current shot must still **explicitly restate** position and state (`Zhu Xi still stands in the warm-lit center of the courtyard`); do not assume the model remembers the previous blocking.
3. **New entering characters**: must state **entry direction + starting position + speed** (`a student enters from the shadowed left side of frame at a run`); **forbidden**: passive reveal phrasing such as "suddenly appears," "suddenly flashes into frame," or "there is now X in the frame."
4. **Multiple characters in one frame**: in one sentence, list all characters in "primary → secondary" order first, then append the environment (`Dudu stands center frame, Mom on the left, Dad on the right, Lili and Grandpa Yantu separated in the background`).
5. **Do not let the camera "discover" a character**: phrases like `the camera pushes in and reveals X / pans over and finally lands on X / it turns out X is standing in the center` are anti-patterns; rewrite as `X stands in the center of frame, camera slowly pushes in at eye level` or `X enters from frame-left, camera tracks sideways with them`.
6. **Off-screen characters / absent sounds**: for subject actions not shown in frame, use SFX suggestion + current subject reaction to carry it (`(off-screen) X's footsteps echo from deep in the hallway, and the subject turns their head`).

Added Format Drift Self-Check item: if the first sentence starts with `Bright X / Sunlight / Cold light / The scene / ...` → flag as `subject-first missing, likely to cause character pop-in`, and rewrite the entire shot.

## Visual Specificity Rules

*(internal heuristic)*

- Abstract `he is very scared` → visual `his Adam's apple bobs, fingers dig into the table edge, and his eyes keep darting away`
- Abstract `she is very smug` → visual `the corners of her lips keep lifting, her fingertips slowly coil around a strand of hair, and her gaze waits like prey is about to approach`
- Abstract `the atmosphere is eerie` → visual `the cold white fluorescent tube flickers twice, and the shadow reflected in the glass lags half a beat behind the real person`

### No adjective-only expressions; use the three-part expression pattern instead

Expressions **must not be adjective-based** (`aggrieved / icy / flustered / resolute / complicated / meaningful`); write them as:

1. **Micro-expression chain**: 2–4 micro-actions linked into one line — `blinks once → swallows lightly → mouth corners press down for 0.5s → gaze drops to the tabletop`
2. **Eyeline anchor**: what they look at, and for how long — `gaze locks on B's left eye for 1.6s, then shifts to B's lips when B starts the second line`
3. **Push-in trigger** (optional; use framing to pull the expression into focus): `slow push-in from medium two-shot to close-up, stopping at 2.4s with the face occupying 60% of frame`

Seedance 2.0 is especially strong on the combination `locked medium shot + micro-action chain` (the tested sweet spot in Atlabs / Apiyi): a locked medium shot lets the model generate blinking/breathing/small head turns/subtle mouth-corner shifts automatically, and is usually more stable than an explicit push-in. Expression-heavy shots should be ≥1.5s, otherwise the micro-actions are cut away before they establish.

### No generic lighting words; use named lighting terms

Lighting **must not** be written only as `cold light` / `warm light` / `mood light`; instead use film/lighting-library terms such as `Rembrandt key / butterfly key / single practical tungsten 2700K / blue rim 5600K / practical sodium-vapor orange 2200K / moonlight substitute HMI 5600K + 1/4 CTB / surveillance-screen cold blue 6500K / emergency light 1Hz flicker`. Replace `cold blue tone, oppressive atmosphere` with `5600K cool white fluorescent overhead + red 3200K doorway sign flicker at 1.5Hz outside the glass, with the light-shadow boundary cutting diagonally across the face`; for Seedance 2.0, the prompt-adherence gain is dramatic.

---

## Suspense / Supernatural Texture

*(internal heuristic)*

For succubus / mimic / rule-horror material: add a small amount of anomalous visual cues without over-explaining.
Shadow too long / delayed reflection / pupils briefly turn red / lock glitch / surveillance static / fluorescent flicker / smile holds one beat too long / footsteps stop but the shadow keeps moving / room temperature so low that breath is visible.
Choose one or two per scene; do not pile them on.

---

## Sensitive / Sexual Source Handling

*(internal heuristic)*

If the source text contains explicit sexualization, preserve the narrative function but rewrite into safe visuals:
- `very little fabric, breasts about to spill out` → `bold styling, seductive presence, purple slip mini dress`
- `you can do whatever you want` → `I'll give you a special reward`
- Explicit body content → guiding gaze, softened voice, slight forward lean, faint lift of red lips

Keep the mood as `seductive, dangerous, manipulative, suspenseful`; do not write explicit erotic content.

---

## Violence Handling

*(internal heuristic)*

For sudden violence, prioritize cinematic impact over gore.
Better: `Jiang Ye suddenly rises, one hand shooting out to clamp Xiaomei's throat and pin her against the wall.`
Avoid blood-and-flesh detail; focus on action, shock, restraint, reaction, and power reversal.

---

## Full Example

Source text:

```text
The shot goes to inside the security room; through the glass, Jiang Ye shows a smile: Reward?
Jiang Ye chuckles: Then what are you waiting for? Come in!
Xiaomei quickly walks into the security room. The next second, Jiang Ye's hand clamps onto her throat. Fear on her face, Xiaomei struggles: Y-you, what are you trying to do! Jiang Ye: A reward doesn't have to be claimed at home. It works here too.
```

Valid output:

```text
Scene 1 Security Room - Smile Behind the Glass 12s

**Shot 1 2.0s Time: Night.** [Medium Shot/static] Inside the security room, cool white fluorescent tubes flicker faintly; Jiang Ye sits behind the duty desk, and outside the glass Xiaomei's silhouette is faintly reflected at the doorway.

**Shot 2 1.5s Time: Night. [Close-up/static]** Glass reflections obscure half of Jiang Ye's face; the corner of his mouth lifts slowly into an unreadable smile. Jiang Ye (playful, soft) "Reward"

**Shot 3 1.5s Time: Night. [Reaction Shot/static]** Outside the glass, Xiaomei hears those two words; her smile turns sweeter, she tilts her head slightly, and there is a triumphant hint in her eyes, as if she thinks Jiang Ye has already taken the bait.

**Shot 4 2.0s Time: Night. [Medium Close-up/push-in slow]** The camera slowly pushes toward Jiang Ye as he leans back in the chair and gives a low chuckle; his eyes shift from lazy to excited, and his hand reaches toward the door-release button at the desk edge. Jiang Ye (excited, urgent) "Then what are you waiting for"

**Shot 5 1.0s Time: Night. [Insert Shot/static]** Jiang Ye's finger presses the access button, a green light comes on, and the security room door lock clicks softly.

**Shot 6 2.0s Time: Night. [Medium Shot/tracking smooth]** Xiaomei pushes the door open and steps in, still wearing a syrup-sweet smile, her body leaning forward slightly as if she can hardly wait to get close to Jiang Ye.

**Shot 7 2.0s Time: Night. [Quick Cut/handheld]** Jiang Ye suddenly rises, motion leaving an afterimage; one hand lashes out and instantly clamps Xiaomei's throat, pinning her against the wall.

Scene 2 Security Room - Identity Reversal 13s Transition:sound_bridge(base layer of rain continues)+match_on_action(Jiang Ye's hand still clamps Xiaomei's throat)

**Shot 8 2.0s Time: Night. [Close-up/static]** Close-up on Xiaomei's face: breath coming fast, pupils tightening slightly, red lips trembling. Xiaomei (terrified, struggling) "What are you doing"

**Shot 9 3.0s Time: Night. [Close-up/push-in gradual]** Jiang Ye's face moves close to Xiaomei; his gaze is frighteningly calm, a smile still hanging at the corners of his mouth, and his low voice sounds like he is seriously explaining something trivial. Jiang Ye (cold smile, oppressive) "A reward doesn't have to be claimed at home"

**Shot 10 2.5s Time: Night. [Medium Close-up/static]** Jiang Ye's grip does not loosen, his body leans forward slightly, the doorway sign outside the glass flickers behind him, and as he looks at Xiaomei his smile turns completely cold. Jiang Ye (calm, dangerous) "It works here too"

**Shot 11 2.5s Time: Night.** [Reaction Shot/static] Xiaomei's pupils suddenly widen; her struggle lags by half a beat before catching up, and her expression shifts from fear to a stunned realization that something is wrong.

**Shot 12 3.0s Time: Night.** [Insert Shot/static] On the security room surveillance monitor, Jiang Ye's shadow reaches the wall half a beat slower than his body, a flicker of static flashes lightly, and the screen returns to normal.
```

---

## Long-Duration Example (user requires 12–15 seconds per shot)

If the user-given single-shot duration is already ≥ 12 seconds, **a single shot itself becomes a scene**, and the scene title should be the shot's core action:

```text
Scene 1 Bedroom - Message Lights Up 13s

**Shot 1 13.0s Time: Night.** [Extreme Close-up→Medium Close-up/static light push] In the dark bedroom, the phone screen in Chen Mian's hand suddenly lights up, filling the frame with a message popup that says only 「Don't look at the sky」; the screen slides down slightly and stops on sender number ending in 0719; the shot slowly widens to a medium close-up of half of Chen Mian's face cut by cold light, her thumb hovering above the screen, neither opening nor replying, her breathing stopping for a beat, while an old notebook on the desk barely peeks into the right foreground. Chen Mian (VO, low, as if dragged out by memory) says: 「Seven years ago, the last time she called me, she said this too.」

Scene 2 Bedroom - Mother's Notes and Flashback 14s

**Shot 2 14.0s Time: Night.** [Close Shot→Close-up/match-cut flashback] Chen Mian's hand holding the phone lowers slightly, and the shot follows to the yellowed old notebook page on the right side of the desk; the instant her mother's handwriting 「Don't look at the sky tonight」 becomes legible, a match cut drops into the night sky seven years earlier — three moons hanging above the city, a single glimpse of an upside-down suspended city, a large empty gap suddenly opening in the standing crowd on the street, the back of a figure in a white cold-weather jacket walking into harsh light and vanishing in the next frame, and all sound vacuuming out for half a beat. Chen Mian (VO, restrained to the point of coldness) says: 「That year, the moon went from one to three. Eleven seconds later, three million people vanished. My mother died in Antarctica that same year.」
```

---

## Format Drift Self-Check

Before submitting, scan the draft once. If **any one** of the following is hit, rewrite the whole output:

- **Any shot line whose first sentence starts with environment phrases such as "Bright X / Sunlight / Cold light / Warm light / A whole / The entire room / The whole wall / In the scene"** (subject-first missing; characters may pop in or fuse with the previous beat); change it to start with `subject + subject position + subject action`, then write environment/lighting.
- Shot descriptions contain passive reveal phrasing such as `the camera pushes in and reveals X / pans over and finally lands on X / it turns out X is standing in the center / X suddenly appears / X suddenly flashes into frame / there is now X in the frame` (subject is revealed too late and may pop in); rewrite as an objective subject-state description.
- A carry-over character (present in the previous shot) is not explicitly restated with position/state in the first sentence of the current shot; add position + state.
- A newly entering character does not specify entry direction + starting position + speed; add anchored wording such as `enters from the shadowed left side of frame at a run`.
- Technical jargon such as `85mm / f/1.4 / shallow DOF / focal plane / focal length / aperture / depth of field` appears (Seedance does not respond); change to natural phrases like `close shot + portrait compression + background blur + face occupies 40% of frame` with proportion anchors.

- Any metadata block such as `## Project` / `**Title:**` / `**Total Duration:**` / `**Style Tone:**` / `## Spatial Anchors` appears
- Any fielded subsection such as `**Narrative Goal:**` / `**Emotional Intensity:**` / `**Visual Narrative:**` / `**Editing Continuity:**` appears
- Any heading in the style of `### Shot X` / `### Scene X: xxx (about X seconds)` appears
- `m1:` / `m2:` action blocks appear
- Bullet lists (`- xxx`) are used to describe inside a shot
- `---` horizontal separators are used between scenes
- Shot numbering resets between scenes (it must increase continuously throughout)
- Any scene's total duration deviates from the default 12–15s without the user explicitly specifying another duration (if the user does not specify, always use default 12–15s; do not ask "how long"; only follow user values after explicit specification)
- A shot line does not start/end with `**Shot X X.Xs Time: ... [Shot Size/Camera Movement]**`
- An ending summary / production note / style-tone paragraph appears
- A single shot contains two or more primary camera movements (`push in then orbit`, etc.); if movement must change, rewrite as timestamps like `[00:00-00:02] push;[00:02-00:05] orbit` or split into two shots
- The keyword `fast` appears in the `[Shot Size/Camera Movement]` field or in the camera-movement portion of a shot description (officially a degrade word in Seedance; use `quick whip pan / sharp turn / sudden cut` instead); story/physiology uses like `heartbeat quickens` do not count
- Technical jargon such as `85mm / f/1.4 / shallow DOF / focal plane` appears (Seedance does not respond)
- Dialogue is not wrapped in quotation marks, lacks an emotion prefix, or any single line exceeds 12 Chinese characters (lip-sync three-part pattern incomplete)
- `says:` / `said:` is used as a narrative lead-in for dialogue (Seedance treats it as noise); quotation marks are the lip-sync trigger
- Expressions use empty adjectives like `aggrieved / icy / flustered / resolute / complicated` instead of being broken into a micro-expression chain
- Lighting only uses generic words like `cold light` / `warm light` / `mood light` instead of named lighting terminology
- Total timestamp duration exceeds 15 seconds (single-generation Seedance limit)
- There are ≥4 shots but no top `Reference Pool:` line; or `@xxx` is referenced in shots but that slot is not declared in the top reference pool
- From the second scene onward, scene titles do not include a `Transition:` suffix, and there are >2 places in the full piece with 0 shared tokens across scenes (effectively a narrative jump-cut)
- The `Transition:` line under a shot uses a non-enum transition type (only `hard_cut / match_on_action / eyeline_match / sound_bridge / j_cut / l_cut / graphic_match / concept_cut` allowed)
- The `Mode:` field in a transition line uses a non-enum value (only `text_to_video / image_to_video_first_frame / first_last_frame / ref_pack / nine_panel_grid_i2v / video_extend` allowed)
- The transition line mode is not `text_to_video` but `FirstFrame:` is left blank

Only six line types are valid:
1. (Optional) Top `# Storyboard Text`
2. (Optional) Top single line `Character Anchors:A=...;B=...`
3. (Optional) Top single line `Reference Pool:@slot=description;@slot=description...`
4. Scene title line `Scene X Name Xs` or `Scene X Name Xs Transition:type(token)`
5. Shot line `**Shot X X.Xs Time: ... [Shot Size/Camera Movement]** ...`
6. (Optional) The line below a shot line: `Transition:type Mode:xxx FirstFrame:@xxx Repeat:xxx`

Nothing else is allowed.

---

## Final Checklist Before Delivering

- [ ] Is it grouped by scene? Does each scene start with `Scene X Name Xs`?
- [ ] Is each scene's total duration within the **default 12–15s** (when the user did not specify, always use the default and do not ask)? If the user explicitly specified another duration, were shot count and scene splitting recalculated using that value?
- [ ] When multiple characters appear, is there a top `Character Anchors:...` block, and do shot lines reuse anchor tokens instead of re-expanding appearance?
- [ ] For ≥4 shots, is there a single-line `Reference Pool:@xxx=...;@xxx=...` manifest? For long narratives of 10+ shots, are at least 3 slot types bound among `@char_main_*` + `@scene_*` + `@lighting_*`?
- [ ] In shot-line descriptions containing mainline characters/main scenes, are inline references like `(@char_main_xxx)` / `(@scene_xxx)` used? Were all referenced slots declared in the top Reference Pool?
- [ ] From the second scene onward, does the scene title line include a `Transition:type(token)` suffix? Across scenes, is at least one of sound_bridge / J-cut / L-cut / match_on_action provided?
- [ ] For Tier 2/3 long narratives (≥4 shots), is there a `Transition:type Mode:xxx FirstFrame:@xxx Repeat:xxx` line below each shot line?
- [ ] Does shot numbering increase continuously throughout the piece, without resetting in each scene?
- [ ] Does every shot line start and end with `**Shot X X.Xs Time: ... [Shot Size/Camera Movement]**`?
- [ ] Is every shot one line per shot, with no shot split into multiple paragraphs?
- [ ] Are actions visible and specific, rather than abstract psychological description?
- [ ] Does every dialogue line have action or emotional buildup on the same line?
- [ ] Does dialogue follow the lip-sync three-part pattern (quotation marks + emotion prefix + ≤12 Chinese characters)?
- [ ] Do expressions use the three-part pattern (micro-expression chain + eyeline anchor + optional push-in trigger), without adjectives?
- [ ] Does lighting use named lighting terminology (color temperature / fixture type / position), rather than just "cold light" or "mood light"?
- [ ] Does each shot have only one primary camera movement? Is the keyword `fast` completely absent throughout?
- [ ] Is there absolutely no technical jargon like focal length/aperture/depth of field/85mm/f/1.4? Where shot texture is needed, are phrases like `wide-angle sense of space / natural perspective / portrait compression / macro texture + frame occupancy` used instead?
- [ ] **Is every shot's first sentence subject-first** (`subject + subject position + subject action`, then environment/lighting/set dressing)? Does no shot start with environment phrases like `Bright X / Sunlight / Cold light / Warm light`?
- [ ] Are **carry-over characters** explicitly restated with position/state in the first sentence of the current shot? Do **new entering characters** specify entry direction + starting position + speed?
- [ ] Are passive reveal phrases like `reveals / finally lands on / turns out to be / suddenly appears / suddenly flashes in / there is now` completely absent?
- [ ] Is there a reaction shot after key dialogue?
- [ ] Are props/environment used to add pressure?
- [ ] For boundary-risk shots (subtitle bars / brand logos / model clipping), is `｜Avoid:...` appended at line end?
- [ ] For dialogue blocks that must be generated as one continuous clip, are `[00:00-00:05]` timestamp hard cuts used, with total duration ≤15s?
- [ ] Has explicit sexual content been rewritten into safe visual language while preserving narrative function?
- [ ] Is violence cinematic rather than gore-stacked?
- [ ] Are there absolutely no project-level metadata blocks, fielded subsections, bullets, subheadings, `---` dividers, or ending summaries?