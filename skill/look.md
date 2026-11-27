# Look and shot rules

Measured on two viral shorts (a 4.4-million-view one with 75 shots in 100 seconds, and one with 72 shots in 114 seconds). These numbers are what make a video feel like the real thing.

## Pacing
- **About one shot a second.** Aim for 25–32 shots in every 35 seconds; the typical shot is 1.0–1.4 seconds.
- **Cut on phrases, not sentences.** One spoken line gets 2–3 sub-shots, each starting on a spoken word: a wide, a close-up, then an insert of the thing or a reaction.
- **Never hold one framing longer than 2.5 seconds,** except the last shot of the video.
- **Change the place often.** A 60-second story visits 4–6 places; the rule, the town's habits, "me", the loved person and the crisis each get their own.

## Shots
- **Vary the camera on every cut:** distance (wide, medium, close), height (low angle, eye level, top-down) and side. Never two wides in a row.
- **Close-ups:** the head fills about half the frame width, with room above the hair. A close-up that cuts off the chin or the hair is too tight. A face's centre is 4.45 studs up (times the character's scale). Put the camera about 7.2 studs away (times the scale) with a 38–40 lens, and aim it a little below the face, so the face sits in the upper half and clear of the captions.
- **The thing the line names is in the picture** when the word is said: the balloon, the hat, the tower, the number.
- **Low angles** for big or scary things (looking up a tower), **top-down** for small or lost things.
- **A slow push-in** (move the camera 0.3–0.8 studs over the shot) on about half the shots, so nothing is frozen.
- **The vertical frame is narrow.** At 8 studs away with a 40-degree lens you see about 3.3 studs across. Two people side by side need 4.4 studs between their centres, so frame two people from a little further back or at an angle.
- **Blur the background on close-ups** (pass the focus distance to `renderShot`); keep wides sharp.

## Acting
A character who doesn't move looks broken. Every visible character does at least one of these in every shot:
- **Breathes and looks around** (`idle(t)`), the minimum for anyone in the background.
- **Hops** when laughing, excited or angry; **kicks their legs** when floating or falling.
- **Moves their hands while talking,** and their mouth follows the voice (`speak`).
- **Walks or runs** in the direction they face. Check the stills: someone moving away from where they look is walking backward.
- **Pops** when a shot cuts to them: a quick squash and stretch in the first 0.2 seconds.
- **Reacts:** the head turns to who is talking or what just happened; the face changes on the key word.
- **Falls, jumps, flights and anything thrown are one continuous motion.** Write the path as a function of `globalT` across every line it spans (start, end, an easing), never a pose per shot: someone who slips off a roof keeps falling through every cut until they land. Nobody hangs or floats in one spot for a whole line ("stuck").
- **While someone falls or flies, the camera holds still and wide** so they cross the frame. A camera that follows them keeps them in the same place on screen and the fall looks frozen. Landings get a squash, one or two quick bounces and a reaction face.
- **One character may stay still on purpose** when stillness is their joke (the serious kid). Everyone around them moves more.
- **The street has people in it:** extras walking the pavements and cars driving past, from `life()` (api.md, "A living world"), never hand-placed.

## Characters
- **Give every main character an outfit with detail and one thing to wear:** a suit and a fedora, a zip hoodie and a hard hat, a denim jacket and a bow. Plain tee and jeans is for extras.
- **Skin can be a toy colour** (yellow, white, grey, green, blue) as well as a natural tone; mix them in one cast.
- **Extras come from `life()`** (which uses `someone(seed)`): 4–8 walkers and 2–4 cars per outdoor world, `stage` around the story's spots.

## Faces
- A new face on almost every sub-shot, changed on the word that causes it.
- Build up: `neutral` → `oh` → `scared` → `scream`. Don't open on the biggest face.
- The yelling and crying faces keep their own mouths; all others lip-sync when that character is the one speaking.

## Places and light
- **The world is never the same twice:** pick a kind of place (suburb, meadow, town square, shop street), a theme and a new seed for every video, and never the kind the last video used. The viral shorts each live somewhere different: a suburb, a grass field with a hill, a grey winter town.
- **Move between places inside the video too:** a 60-second story uses the outdoor world plus 2–3 rooms (a classroom, a diner, a bedroom) or built places.
- **Colour moods carry the story:** `night` when it gets dark, `danger` (all red) for the monster or the alarm, `memory` (black and white) for a flashback or "as it turns out", `cold` for winter. One mood per shot, set with `state.tone`.
- **Camera close and low:** the references put the character at half the frame height or more, shot from chest height looking slightly up. A small person in a big empty frame reads as a toy set.
- **Dress each indoor place** with 6–10 things that belong there (a board, desks, a poster with words on it, a clock, windows) and one thing only this story has.
- **Signs and posters carry facts.** Put the rule or a joke on a sign the camera can read. Make the words fit: text wider than the sign is cut off without a warning, so use a smaller font or shorter words.
- **Light follows the story:** bright day for normal life, `dusk` from the moment things go wrong, the `night` theme for a story that happens at night, and night fog for one scene that goes dark (see api.md).
- **Short lines still get cuts:** aim for about one shot a second, so a line under 1.2 s gets one shot and a longer one gets 2–3.
- Nothing floats, slides or passes through anything unless the rule makes it.

## Captions
- **One or two words at a time,** lowercase, low in the frame (66 % of the height), white with a dark edge.
- **Key words in yellow:** the rule's thing every time it's said, names, numbers, the peak word of a line. List them in `story.json` under `keys`.
- **Counted numbers go big** at the top of the frame for the moment they're said (3, 6, 60, 200). The number covers the top 30 % of the frame, so frame the subject below it.
- Keep faces and the main thing out of the caption band (60–72 % of the height).

## Sound
The voice and one music track from the first frame, the music well under the voice (`musicVolume` 0.12).

**Sound effects: one sound per thing that appears, landing on its frame.** You don't place them by hand: the directing helpers log what happens on screen and the render turns that into sounds, mixed under the voice, then masters to -14 LUFS.
- **A new place** (the camera jumps somewhere else): a whoosh. A plain cut inside the same place is silent.
- **A big number or word** (`d.big`): a slam (camera, then pitched echoes, then a break, never the same twice in a row, each a little quieter).
- **A character popping in** (`d.pop`): a soft click, quieter than everything else.
- **A story moment** the helpers can't see (a door opens, a car honks, something lands): add `d.sfx(name, word)` in that shot. Sounds: `whoosh`, `whoosh2`, `swoosh`, `click`, `pop`, `ting`, `camera`, `snap`, `break`, `hit`, `drop`. One sound does one job; use them for things the viewer sees.
- **Keep air:** two sounds closer than 0.18 s keep only the more important one.
- **Never:** typing clicks, cartoon foley, a sound on every cut, or anything louder than the voice.
After the render, read `out/sfx-plan.json` (every sound with its time and reason) and remove `d.sfx` calls that feel like too much.

## What five viral shorts measured (pick the row closest to your story)
| Story | Length | Shots | Typical shot | Camera | Captions |
|---|---|---|---|---|---|
| Rescue-tool rule story | 100 s | 75 | 1.0–1.4 s | cuts, slow push-ins | 1–2 words, lowercase, yellow key words |
| Greedy narrator | 107 s | 72 (40 a minute) | 1.5 s | cuts only, a new angle on every word group | 1 word at a time, white |
| Town afraid for the wrong reason | 118 s | 49 cuts + 34 camera moves | 1.3 s between changes | glides and orbits inside a place, top-down for games and crowds | 2–3 words, one coloured key word, a different colour per line |
| Dark newcomer twist | 43 s | 20 | 1.7 s | handheld tilt, low angles, top-down for the lonely shot | 2 lines: white words over one key phrase in caps, colour changes per line |
| Cliffhanger | 30 s | 16 | 1.75 s | a crowd walking at the camera, big inserts of the rule object | a phrase, key word in caps, red |

- **Faster story, more shots:** a 250-words-a-minute story cuts every 1.0–1.5 s; a slow 170 one may hold 1.7 s. Never slower than that.
- **The first 3 seconds have 2–3 changes** and show the rule on a person, not an empty street.
- **The opening list:** each town habit gets its own place and its own 1–2 s shot (stilts, a rope bridge, a zipline, a dog on stilts).
- **Caption styles** are set in `story.json` under `"caption"` (see api.md). Keep one style for the whole video.

## Things these stories need (build them, don't avoid them)
- **A label over a head** (a button, a number, a name): a `sign()` parented above the head, turned to the camera.
- **A world with one colour rule** (everything grey until painted): recolour the town's materials once in `create`, and change single meshes when the story paints them.
- **Animals and creatures** (a dog, a mouse, a robot, a giant): build from blocks with a face, legs that swing, and a size the story needs. A monster is shown as a shape, a shadow or an eye, never hurting anyone.
- **Tall or strange ways to move** (stilts, ziplines, rope bridges, riding a creature): mark the rider `airborne` and move the prop with them.
- **Effects from blocks:** a puff of smoke (white balls that grow and fade), paint splats (flat coloured discs), a person dissolving (small cubes flying up while they shrink), a white-out (fog near 0 with a white colour).
- **Inside places with their own mood:** a warm living room, a red diner, a dark purple library. Change the light and fog per shot.
