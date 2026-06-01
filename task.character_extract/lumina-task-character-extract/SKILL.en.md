---
name: lumina-task-character-extract
description: Supplemental prompt for task.character_extract; used to extract stable character assets from text
---

# task.character_extract

## Applicable Scenarios

- Extract stable character assets from novels, scripts, short drama scene breakdowns, and story outlines.
- Build a character library for subsequent character three-view/main-view images, state three-view images, storyboard image selection, and video reference images.

## Responsibilities

- Prioritize mapping character mentions in the text to existing character assets, merging the same character’s name, nickname, pronouns, and relational titles.
- Character extraction must strictly ensure coverage: any character with a name, stable title, dialogue, narration reference, direct action, camera attention, or a later need for facial/clothing continuity must be included in `newEntities`, `existingEntityUpdates`, or `unresolvedMentions`. Do not omit them due to limited appearance details.
- Identify characters worth maintaining long-term: protagonists, key supporting roles, recurring characters, plot-driving characters, and characters that must be reused consistently in later visuals.
- Preserve consistency anchors that support image generation: age stage, species/identity, facial contour, facial feature proportions, hairstyle, body proportions, base clothing, stable markers, common clothing states, and visible historical traces.
- Record multiple stable states of the same character as reusable asset descriptions under a single identity, such as home, outdoor, sleeping, combat, injured, sickly, disguised, special forms, and different age stages.
- Identify identity assets that require long-term maintenance later, such as base three-view, outdoor-state three-view, home-state three-view, sleep-state three-view, and combat/injured/special-form three-view.

## Not Responsible For

- Do not extract scenes or standalone props; only record stable clothing worn by the character, personal signature items, or identity-related props.
- Do not directly write final image prompts or plan image task nodes.
- Do not turn one-off actions, single-shot expressions, temporary blocking, or crowd extras into character assets.
- Do not split every outfit of the same character into a new person; if the identity is the same, record it as clothing or state variants of the same character whenever possible.
- Do not invent face shapes, clothing, professional background, or relationships not supported by the text just to complete the profile.

## Decision Principles

- Identity name priority: explicit name > stable form of address/nickname > stable relational title with identity clarification. Do not use “I,” “he,” “she,” “man,” or “woman” directly as long-term asset names.
- Clearly identical references such as “father, dad, he” or “mother, mom, she” must be merged; if identity cannot be confirmed, keep the mention unresolved.
- Age stages like “childhood Shen Shen,” “teenage Shen Shen,” “young adult Shen Shen,” “adult Shen Shen,” and “elderly Shen Shen,” when they share the same name or can be confirmed as the same identity, must be merged into one character asset; record different age stages as age-stage/state assets under the same character rather than creating multiple Shen Shens.
- Characters are not limited to humans; demons, monsters, animal companions, spirit beasts, and anthropomorphic roles with names, titles, agency, or narrative continuity should all be treated as character assets.
- When the same character appears across outfits, states, ages, or time periods, treat them as the same identity by default; age span, species form, or identity disguise only affects stage asset naming under the same character, and the relationship to the original identity should be explained rather than split into multiple independent characters.
- Observer groups, passersby, villagers, doctors, classmates, etc. usually should not become new character assets unless they have a stable narrative role or continuity requirement.
- Temperament cannot replace appearance. Do not write only “beautiful,” “cold,” “powerful,” or “gloomy”; convert these into visible anchors where possible, such as restrained posture, neat hairstyle, cool-toned clothing, steady gaze, or upright shoulders and back.
- Do not default to beautifying, making younger, or making healthier. If sickness, poverty, battle damage, labor traces, or atypical appearance are narrative facts, they should be preserved as stable states.

## Visual Anchor Hierarchy

- Identity-invariant items have the highest priority: age stage, species/identity, face shape contour, eye spacing and brow-eye proportions, nose shape, lip features, jawline, hairstyle silhouette, skin tone/skin texture, height proportions, posture, and stable markers.
- Base clothing is the outfit most frequently reused or most representative of the identity, primarily serving later three-view images and standard video shots.
- Clothing variants should only record stable states that will be reused later, such as homewear, commute wear, school uniform, battle robe, armor, formal wear, disguise, or patient gown.
- For state variants, write visible surface details, not abstract experience. For example, “rough knuckles on the right hand with cracks and dried blood” is better than “worked very hard”; “charred and damaged edges of the battle robe” is better than “went through a great war.”
- Special forms should remain under the same character as the normal form, such as demonized form, spirit form, beast form, cultivation manifestation, or awakened form; only emphasize stage differences when species and silhouette change completely.
- Personal items should only be recorded if they have stable value for character identity or emotional continuity; if an item needs standalone close-ups, handoff, or cross-scene reuse, it should be handled by prop extraction.

## Asset Clue Consolidation

- During extraction, clues needed for later image-to-image state assets must be consolidated into `summary` or `attributeEntries`; do not directly write image prompts.
- `attributeEntries` does not require fixed keys; prefer a small amount of natural language to describe asset needs, such as “needs base identity three-view,” “outdoor-state three-view appears,” or “Chapter 2 adds injured battle-damage state three-view.”
- Every asset clue must answer: who the unique identity is, what state this asset is for, which identity anchors must remain unchanged, what visible content changes, and when it will be reused.
- Clothing/state assets should record reusable states such as home, sleepwear, outdoor, commute, school uniform, combat, work, formal, disguise, injured, sickly, awakened, etc.; do not force them without textual basis.
- Age-stage assets should record reusable stages such as childhood, adolescence, youth, adulthood, and old age; for age changes of the same character, clearly state the inherited identity/bloodline/facial continuity anchors, and note that this stage changes only age, height proportions, facial maturity, hairstyle/clothing stage, and posture.
- Facial expressions, hands, wounds, masks, seals, tattoos, jewelry details, and similar local details should only be recorded as shot-level needs in `summary`, not maintained long-term as asset-library images.

## Age Stage Merge Rules

- For same-named characters or different age stages clearly identified by context as the same person, always merge them into the same entity; do not split them into multiple roles due to age changes.
- Age stages may produce different character state images, but they must belong to the same character asset within the same workflow task, such as “Shen Shen base young-adult three-view” and “Shen Shen childhood-stage three-view.”
- If the current text only shows one age stage, record only the current stage; do not fill in childhood/youth/old age by default just to complete a life arc.
- If later incremental text adds another age stage of the same character, use `existingEntityUpdates` to supplement the age-stage asset requirement; do not create a new character, and do not regenerate unrelated baseline images.
- For cross-age image-to-image generation, emphasize identity continuity: preserve core face-shape family resemblance, eye spacing/brow-eye relationship, nose shape, jaw trend, hair color/hairline clues, and stable markers; only change age maturity, body proportions, clothing, and stage state.

## Boundary Between State Assets and Shot-Level Performance

- During character extraction, only consolidate identities, clothing, age stages, special forms, and stable visible states that will be reused across shots; do not turn dialogue reactions, gaze changes, pale lips, breath pauses, sightline targets, or one-off hand actions into asset-variant requirements.
- Expressions, mouth shapes, gaze, breathing, sightlines, paused fingers, and relationship turns belong to `story_to_video`’s `visualBeat` / `emotionHint`, not the character asset library images.
- Only states that stably alter character recognition or will be continuously reused later should be recorded as state assets, such as bloody bandaging over the right eye, chronically sickly complexion, battle-damaged clothing, fixed scars, mask cracks, constantly glowing sigils, corrupted form, or beast form.
- If local evidence only needs to be seen clearly in the current shot, such as a wound on the back of the hand, blood at the throat, holding a hairpin, a ring brushing past, or a twitch at the corner of the mouth, record it as a plot clue by default rather than generating a long-term character asset.
- Separate expression changes from identity anchors: what can vary is the mouth corners, brows, eye rims, breathing, and gaze; what must remain unchanged is face shape, eye spacing, nose shape, jawline, age stage, hairstyle boundary, and stable markers.

## Working Method

- First scan for stable forms of address and narrative function, then determine whether they can be reused under an existing identity.
- After scanning the full text, perform a coverage audit: check names, titles, dialogue speakers, narration subjects, action subjects, observed subjects, and relational titles one by one to confirm that every character requiring stability has been extracted or marked unresolved.
- Before deciding to add a new one, ask: will this character need stable reuse later? Is there enough identity evidence? Is this only a one-off crowd role or functional extra?
- For protagonists and key supporting roles, first lock identity-invariant items, then organize base clothing, stable state assets, and special forms; shot-level expressions/local clues go only into `summary`, not long-term image requirements.
- For key characters appearing at multiple age stages, first confirm whether those stages belong to the same identity, then organize age-stage assets under the same character; do not split “childhood Shen Shen” and “young adult Shen Shen” into two independently bindable characters.
- For narratively important characters with insufficient appearance information, preserve reliable age stage, identity relationship, temperament, and clothing hierarchy without inventing specific facial features.
- For stable non-human identities, specify species/form, silhouette, body size, movable parts, material texture, and narrative identity.
- Do not write scenes into character attributes; from “her in the living room,” extract only the character state or clothing, and leave “living room” to scene extraction.
- Final character descriptions should be short and precise: state who they are, their narrative role, base appearance, base clothing, and whether important clothing, age stages, special forms, or stable state variants exist.

## Quality Standards

- The character list is clean, non-duplicative, and uses stable naming.
- Do not miss characters: anyone with stable presence, dialogue, narration reference, or shot-central responsibility must be traceable in the result; do not extract only the protagonist while missing key supporting roles or new states.
- For every protagonist or key supporting role, it should at least be possible to answer: who they are, how they relate to the story, what their base appearance is, and which base outfit is best suited as a reference image.
- If the text includes multiple stable outfits or states, the character asset should answer: what variants exist, when they are used, and which differences later image-to-image generation must preserve while keeping identity consistent.
- Key characters must support the chain of “baseline character three-view/main-view + necessary state three-view/main-view + shot-level performance hints,” and identity must not drift due to expression, clothing, or scene changes.
- The extraction result must not treat plot explanation, psychological commentary, or author evaluation as imageable appearance.

## Failure Fallback

- When boundaries are unclear, it is better to collect less than to mistakenly include one-off characters.
- When information is insufficient, keep it unresolved rather than forcing a long-term asset from pronouns.
- When there is no appearance basis, use conservative identity and state descriptions; do not fabricate face shape, hair color, clothing, or background.