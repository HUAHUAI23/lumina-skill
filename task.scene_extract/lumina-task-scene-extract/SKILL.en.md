---
name: lumina-task-scene-extract
description: Supplemental prompt for task.scene_extract; used to extract stable scene assets from text
---

# task.scene_extract

## Applicable Scenarios

- Extract stable scene assets from text, scripts, and story passages.
- Build a scene library for subsequent empty-scene reference images, image-to-image spatial locking, storyboard continuity, and video shot reuse.

## What It Is Responsible For

- Prioritize mapping spatial mentions in the current text to existing scene assets, and merge different references to the same location.
- Identify locations and spaces worth maintaining long-term: scenes that recur, carry key plot events, affect visual continuity, or require reference image generation later.
- Preserve image-generable spatial anchors for each scene: space type, layout structure, scale relationships, entrances/paths, vertical hierarchy, key furnishings/natural elements, materials, lighting, color palette, weather/season, and default missing elements.
- Record key state changes within the same space, such as day/night, sunny/rainy-snowy, abandoned/revived, damaged/repaired, polluted/purified, poor/lavish, modern/aged.
- Leave enough basis for downstream empty-scene prompts so the scene can serve as a spatial locking reference, rather than only preserving an abstract mood.

## What It Is Not Responsible For

- Do not extract characters or standalone props.
- Do not treat one-off shot backgrounds, pure time, pure weather, or pure emotion as stable scenes.
- Do not directly write final image prompts or image task structures.
- Do not extract monsters, beasts, animal companions, or spirit beasts with agency and recurring presence as scene assets; they should go into character assets.
- Do not invent new locations, map relationships, or architectural structures that are not supported by the text just to enrich the image.

## Judgment Principles

- For identity names, prefer stable place names or space names used in the text, such as “rocky hill,” “ward,” “snowfield at the village entrance,” or “Xuanqing Sect sword array”; do not use generic labels like “on the mountain,” “over there,” or “inside the house” unless the context contains only that one stable space.
- Do not split the same location into multiple scenes because of slight angle, weather, time, or plot-action changes; these are usually states or shot tendencies of the same space.
- Scene descriptions should focus on drawable facts: spatial boundaries, front/back/left/right relationships, vertical hierarchy, entrances and exits, main materials, main light source, main color palette, typical furnishings, and missing elements.
- Atmosphere cannot substitute for space. Do not write only “oppressive,” “cozy,” “mysterious,” or “horrifying”; convert them into visible information such as lighting, color palette, materials, crowdedness/openness, decay/neatness, warm/cool tone, humidity, dust, sound sources, and so on.
- By default, interpret scenes as empty-scene assets; unless the scene itself depends on crowd activity, do not write character actions into the core scene description.
- Background animal groups, environmental creatures, or distant ecology should be recorded only when they serve the spatial atmosphere; creatures with independent identity and action continuity should not be placed in scenes.
- Generic locations such as hospitals, streets, construction sites, and villages should be added only when they carry important plot or are likely to be reused later; locations mentioned in passing should be left pending parsing or omitted.

## Spatial Anchors

- Basic spatial identity: indoor/outdoor, modern/ancient/xianxia/sci-fi, residence/sect/street/battlefield/ward/construction site/natural landform, etc.
- Layout structure: room orientation, doors and windows, corridors, bedside/under-bed, tabletop/steps, courtyards, mountain paths, streets and alleys, building clusters, formation center, etc.
- Scale relationships: narrow, open, depth, oppressive, low-ceilinged, high-ceilinged, layered, sloped, characters appearing tiny in long shots, etc.
- Key elements: architecture, furnishings, natural objects, terrain, entrances, paths, obstructions, recurring spatial markers.
- Materials and surfaces: wood, stone, metal, glass, concrete, soil, snowfield, water surface, broken walls, damp ground, scorch marks, mold stains, dust.
- Lighting and color palette: main light source, light direction, warm/cool, brightness/darkness, lightning flash, golden light, ordinary household lighting, morning light, night lamps, volumetric light, shadow density.
- Missing elements: no crowd, no modern decor, no green leaves or fruit, no clean streets, no warm lighting, etc.; choose according to plot facts.

## State Phases and Nonstandard Environments

- Scene assets should record stable structure, but also a set of key states that affect later storyboards: season, weather, destruction/repair, abandonment/prosperity, pollution/purification, frozen death/revival, day/night, poverty/luxury.
- When the same space undergoes major state changes, it should usually still remain one scene asset, with differences recorded in state notes rather than split into multiple scenes by default.
- When core elements in a scene are in a nonstandard state, the absence of default features must be explicitly recorded. For example, a frozen-dead orchard should state bare cracked trunks, no leaves, no fruit; an abandoned house should state broken windows, peeling walls, no maintenance, no clean new renovation.
- If a scene’s emotion comes from scale and oppressiveness, record generatable scale cues, such as a vast scree slope, terraced layers, giant building shadows, narrow corridors, low-ceilinged spaces, tiny figures in the distance.
- Plants, crops, building clusters, bloodstains on snow, muddy wooden signs, and similar elements may be included as scene elements if they primarily serve spatial atmosphere and continuity; if they require standalone close-ups, interaction, or cross-scene reuse, hand them off to prop extraction.

## Asset Clue Consolidation

- During extraction, clues needed for later image-to-image spatial state assets must be consolidated into summary or attributeEntries; do not directly write image prompts.
- By default, leave only one most reusable empty-scene baseline requirement for the same scene; do not tend toward generating multiple scene images because of slight angles, one-off weather, temporary lighting, or character actions.
- attributeEntries do not require fixed keys; prefer a small amount of natural-language explanation of the asset requirement, such as “need a clean unoccupied baseline empty scene.” Only when there is a real need to reuse different form states, time settings, or spatial viewpoints long-term should you record derived requirements such as “night lit-state image,” “dilapidated-form image,” or “entrance-to-corridor depth-view image.”
- Every asset clue must answer: what is the unique scene identity, what form/time/spatial viewpoint this asset is for, which spatial anchors remain unchanged, what only changes in time/weather/season/form/state/viewing axis, and when it will be reused.
- Local details such as entrances, paths, bedside, walls, plaques, formation textures, and material wear should be recorded only as shot-level requirements, not maintained long-term as asset-library images; if multiple spatial angles are truly needed, output viewpoint images that clearly show scene identity and layout relationships, not fragmented local pieces.
- In new or incremental extraction, afternoon, night, rain, dilapidated, repaired, and similar variations of the same location should be merged into the same scene asset first; only if these states will be stably reused across multiple later shots should they become image-to-image derived assets of the same scene.

## Multi-Image Restraint Rules

- First judge whether multiple images are needed, then record derived requirements; scene extraction is not for generating one image per time point by default.
- One baseline empty scene is sufficient to cover: ordinary dialogue, minor movement blocking, one-off weather, temporary lighting, and local single-shot entrance/bedside/wall details.
- Multiple images are recommended only in three cases: the same space undergoes reusable form changes, such as damaged/repaired, abandoned/revived, polluted/purified, formation activated/deactivated; the same space needs multiple reusable time settings or weather/season states, such as normal afternoon and lit night; or the spatial structure is complex enough to require additional entrance, depth, reverse-angle, or upper/lower-level views for stable storyboarding.
- Ordinary composition angles, partial walls, bedside, window side, table corners, and one-off shot camera positions should not be consolidated into long-term scene images; leave these to story_to_video using the baseline empty scene plus text control.
- If a scene appears only at night, write night directly as the baseline empty scene and do not generate an additional daytime version; if it appears only in rain, do not add a sunny version either.
- When incremental extraction encounters a new time/weather/state for an existing scene, update existingEntityUpdates and record the requirement as “create an image-to-image derivative based on the existing baseline empty scene”; do not create a new same-named scene, and do not regenerate a new baseline image.

## Working Method

- First determine whether the spaces in the text can already be reused with existing scenes.
- Before deciding to add a new one, ask: will this place be reused? Does it carry key plot? Will a later empty-scene image be needed to lock the space? Does it have enough visible anchors?
- During extraction, check against “spatial identity + layout structure + scale relationships + key elements + material surfaces + light/shadow color palette + weather/season + state changes + missing elements.”
- If the source text contains reusable angles or later visual requirements, record them only as composition tendencies, such as wide shot establishing the space, entrance path, under-bed close shot, wall close-up; do not write them as full image prompts during extraction.
- For locations that the model may beautify by default, clearly state “what is not there”: no crowd, no modern decor, no green leaves or fruit, no clean streets, no warm lighting, etc., according to plot facts.
- Final scene descriptions should be short and precise: state what kind of space it is, the core layout/scale, key elements, lighting/color palette, current state, and which default elements must not appear.

## Quality Standards

- The scene list is stable, non-duplicative, and location naming is clear.
- Each scene asset can support an empty-scene reference image, not just abstract mood words.
- A scene asset must answer: from what angle is this place instantly recognizable? Which elements must appear? Which default beautifications must be excluded? Which state changes will affect later shots?
- Key scene states must not be washed out by broad naming; if the story requires desolation, withering, snow disaster, decay, or revival, there should be visible evidence and exclusive absences.
- Do not mistakenly write character actions, prop close-ups, or plot conclusions as the main subject of the scene.

## Failure Fallback

- When boundaries are unclear, it is better to capture too little than to mistakenly capture a one-off background.
- When information is insufficient, prioritize preserving space type, core layout, and main atmosphere; do not invent map relationships, architectural structures, or specific furnishings.
- If space is mixed together with props, plants, or character states, prioritize retaining only the space itself, and hand object states to the corresponding asset pipeline.