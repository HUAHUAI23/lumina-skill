---
name: lumina-asset-character-from-image
description: Supplemental prompts for asset.character_from_image; used to bind uploaded character images into stable character assets
---

# asset.character_from_image

## Applicable Scenarios

- The user uploads character images and wants to bind the images to project characters, or use new images to update the current character reference.

## Responsibilities

- Determine who the character in each uploaded image is, one image at a time.
- Provide stable, natural character names that can be reused long term.
- Extract enough appearance detail to support continuity in later use.

## Out of Scope

- Do not forcibly merge multiple images of different characters into a single character.
- Do not expand the story, relationship network, or character backstory.
- Do not decide downstream steps outside the project on the user's behalf.

## Decision Principles

- If the user explicitly specifies "who is in image 1" and "who is in image 2," follow the user's binding semantics exactly.
- Prefer reusing existing stable character names; only provide a new natural name if needed.
- Focus descriptions on appearance, apparent age, temperament, clothing, and strongly distinctive features.

## Working Method

- Review uploaded images one by one in upload order and assess them separately.
- Confirm character identity first, then add a short and precise stable description.
- If the user says "this is a new image of a certain character," prioritize interpreting it as strengthening that character's current reference.

## Quality Standards

- Character names should remain stable and not change frequently due to camera or shot differences.
- Descriptions should be reusable and sufficient to support later visual continuity.

## Failure Fallback

- If a specific identity cannot be confirmed, use a conservative, natural temporary character name.
- Do not invent complex settings, and do not present uncertain content as established fact.