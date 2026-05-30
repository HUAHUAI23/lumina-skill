---
name: lumina-task-prop-extract
description: Supplementary prompt for task.prop_extract; used to extract stable prop assets from text
---

# task.prop_extract

## Applicable Scenarios

- Extract stable prop assets from text, scripts, and story passages.
- Build a prop library for later clean, background-free single-object reference images, necessary form/state images, storyboard continuity, and video shot reuse.

## What It Does

- Prefer mapping prop mentions in the current text to existing prop assets, merging aliases, abbreviations, and descriptive references for the same prop.
- Identify key props that affect later shot continuity, plot progression, emotional foreshadowing, or visual recognition.
- Preserve image-generation anchors for each prop: category, silhouette, structure, material, color, size ratio, wear condition, texture, glow effects, markings, and signs of use.
- Identify stable state changes of the same prop, such as brand-new/worn, intact/broken, clean/bloodstained, clear writing/blurred writing, inactive/glowing activated.
- Leave a basis for judging whether a standalone reference image or form/state image is needed later; ordinary angles, materials, handheld shots, and local close-ups should be deferred to the story_to_video shot stage.

## What It Does Not Do

- Do not extract characters or scenes.
- Do not treat one-off background clutter, consumables, or decorative noise as stable prop assets.
- Do not embellish a prop's origin, brand, mechanism, or hidden abilities unless explicitly supported by the source text.
- Do not extract sentient, repeatedly appearing yokai, monsters, animal companions, or spirit beasts as prop assets; they should go into character assets.
- Do not repeatedly split a character's clothing, jewelry, or weapon carrying style into separate props; only preserve them independently when they are separately handled, delivered, highlighted in close-up, or reused across scenes.

## Decision Principles

- Prioritize extracting spirit objects, magical artifacts, weapons, tokens, keys, documents, potions, relics, evidence, vehicles, mechanical devices, and repeatedly appearing objects that require visual continuity.
- Keep one stable name for the same prop; do not split it into multiple objects because of contextual variation such as "long sword," "flying sword," or "bonded flying sword." If a set relationship truly exists, record the set quantity, shared form, and points of difference.
- Focus descriptions on visible identifying features: shape, structure, material, color, size, sense of weight, texture, age, damage, stains, markings, glow, or energy effects.
- Ordinary background clutter usually should not be added; only make it a prop asset if it appears repeatedly, is handled by characters, carries emotional weight, or serves as key evidence.
- Plants, saplings, and crops should generally be treated as scene elements when they mainly establish location atmosphere; only treat them as props when they are separately handled, repeatedly shown in close-up, used as evidence, or carry plot clues.
- Corpse remains, specimens, statues, totems, portraits, and similar objectified presences can be props; living beings with agency cannot.
- When the text only gives abstract meaning, convert it into a visible carrier. For example, if "the amulet symbolizes love," record the talisman paper, material, creases, golden glow, and carry wear, rather than only writing "important token."

## States, Text Surfaces, and Signs of Use

- Prop assets must record both stable shape and surface states that affect later shots.
- If a prop's significance comes from being old, broken, dirty, bloodstained, muddy, cracked, charred, faded, water-damaged, curled at the edges, blurred in writing, or worn through years of use, those states must be preserved as primary features.
- When the same prop changes across stages, do not split it into multiple props by default; prefer recording it as a state change, such as "writing is clear when first hung, then blurred after wind, snow, and mud," "blade goes from intact to chipped," or "talisman paper goes from dim to golden activated."
- For props that models tend to beautify by default, explicitly record what must be absent: not brand-new, not smooth, not a clean product shot, not modern plastic-looking, not intact and undamaged.
- For text, handwriting, symbols, serial numbers, and markings, distinguish between "content intent" and the "renderable carrier surface." Accurate long Chinese text, tiny text, and complex document content should not rely on image models to generate stably; the prop asset only needs to record the paper, wooden tag, carved marks, ink traces, blank areas, and rough writing traces.
- If accurate readable text is required later, note that post-production texture replacement or subtitle-layer overlay is more appropriate; do not treat precise readable text as the only identifying feature of a prop reference image.

## Asset Clue Preservation

- During extraction, clues needed for later image-to-image prop state assets must be preserved in summary or attributeEntries; do not write direct image prompts.
- By default, only preserve one clean, background-free standalone baseline asset requirement for the same prop; do not lean toward generating multiple images because the text includes slight angle changes, handheld use, stains, local close-ups, or one-off actions.
- attributeEntries do not require fixed keys; prefer using a small amount of natural language to describe asset needs, such as "needs a clean background-free standalone anchor." Only record derived needs such as "activated glowing state image," "opened form image," or "damaged bloodstained state image" when different forms or stable states will truly be reused later.
- Every asset clue must answer: what the unique prop identity is, what form/state this asset is for, which prop anchors stay unchanged, what visible content changes, and when it will be reused.
- Material macro shots, handheld views, carved marks, runes, bloodstains, ink traces, wear, cracks, or opening/closing details should only be recorded as shot-level requirements, not maintained long-term as asset library images.
- In new or incremental extraction, if a state change only appears briefly in the current shot, prefer updating the same prop's summary; only preserve it as a later image-to-image derived asset if it will be reused across shots, changes prop recognition, or must be generated stably in that same state.

## Multi-Image Restraint Rules

- First judge whether multiple images are needed, then decide attributeEntries; do not treat variants as a default output.
- One baseline standalone image is enough to cover: standard presentation, slight handheld use, ordinary placement, temporary reflections, one-off stains, and local close-ups within a shot.
- Different angles of the same prop do not require multiple images by default; video models need stable object identity more than competing front/back/left/right angle references.
- Only recommend multiple images in two cases: the same prop has reusable different form/structural states, such as closed/open, intact/broken, inactive/activated; or the same prop undergoes a stable stage change in the story that later shots need to maintain.
- If the goal is only to see the back, thickness, handle, texture, or local carved marks more clearly, default to using the same clean standalone baseline image together with story_to_video shot-level img2img, rather than preserving long-term angle images.
- If a state is the prop's first appearance and remains its main state afterward, such as "a worn bloodstained dagger" staying in that state throughout, write it directly into the baseline image instead of generating both "clean dagger" and "bloodstained dagger."
- When incremental extraction encounters a new state of an existing prop, update existingEntityUpdates and record the need for "image-to-image derivation based on the existing baseline image"; do not create a new prop with the same name, and do not generate another baseline image.

## Boundary Rules

- Clothing, shoes, and jewelry that a character wears long-term but that do not act independently should generally be treated as character costume or markers; only consider them props when they are removed, delivered, lost, shown in close-up, or become evidence.
- Fixed furnishings in a scene such as beds, doors, walls, plaques, and trees should generally be treated as scene elements; only consider them props when they are moved, hidden, opened, repeatedly shown in close-up, or independently carry clues.
- If a weapon is only part of a character's combat presentation, it can be treated as a character-carried marker; if the weapon itself has a name, attributes, inheritance, set relationship, or is separately handled, delivered, or reused as evidence, it should be treated as a prop.
- For prop sets, record the "set identity" and individual recognition method, such as thirty-six flying swords, identical metal blades, unified arrangement, and cold reflections, rather than generating thirty-six separate prop assets.

## Working Method

- First determine whether an object in the text can already be reused from an existing prop.
- Before deciding to add a new one, ask: will it be reused? Is it handled, delivered, hidden, damaged, shown in close-up, or used as evidence? Does it have sufficiently stable visible features?
- During extraction, check by "category + silhouette/structure + material + color + size ratio + surface state + markings/text surface + usage relationship + state change."
- Keep only stable visible features and narrative purpose; do not write one-off placement, temporary actions, or shot effects as stable attributes.
- If a prop is long used or inherited by a character, record the relationship to the character and tactile traces, such as a wooden handle polished smooth by long use, an amulet creased from being kept close to the body, or an old workbook stained with mud and water.
- Final prop descriptions must be short and precise: state what it is, why it needs reuse, what it looks like, what its surface state is, and whether it needs a clean standalone asset or a stable form/state asset.

## Quality Standards

- The prop list is clean, non-redundant, and stable in naming.
- Every key prop can independently generate a clear single-object reference image with no characters, no hands, and no scene background, and can be accurately referenced in later storyboards.
- For every key prop, it must be possible to answer: what its silhouette is, what its main material is, what its size ratio is, what it has been through, what traces remain on its surface, and which default clean/new features must not appear.
- For props involving text surfaces, accurate long text must not be treated as the only identification method.
- Prop sets must preserve their set relationship and not be split into unrelated objects.

## Failure Fallback

- When boundaries are unclear, prefer under-collecting rather than mistakenly collecting one-off clutter.
- When information is insufficient, prioritize preserving the name, category, silhouette, and core material, and do not impose complex patterns, brands, mechanisms, or fantasy effects.
- If an object is only part of a character's clothing or scene furnishing, prefer not creating a new prop asset.