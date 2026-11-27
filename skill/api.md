# What a scene can call

A scene file is `scenes/<scene>.js` in the video project. Every beat in `story.json` names it (`"style": "town", "scene": "<scene>"`). Import everything from `'../lib/blocky/index.js'`. Units are studs: a character is 5.2 tall, x is left-right, y is up, z is toward the camera, and characters face +z.

```js
export default {
  async create({ THREE, scene }) { setThree(THREE); /* build the world and the cast once */ return state },
  update({ camera, state, globalT, index, raw, dur, words }) { /* place everything for this moment */ },
  render({ renderer, scene, camera, state }) { renderer.toneMapping = T.NoToneMapping; renderShot(renderer, scene, camera, { dof: state.dof }) },
  overlay({ ctx, W, H, raw, words, story, state }) { /* captions and big numbers */ },
}
```

`update` gets: `index` (which line), `raw` (seconds into the line), `dur` (the line's length), `globalT` (seconds into the video), `words` (the line's words: `{ word, start, end }`, seconds from the start of the line). Everything must depend only on these, never on what the last frame did, so any frame can be drawn alone. (`voice/timing.json` has the same words with times from the start of the video; use it to pick still times.)



## A scene in a few lines
```js
import { setThree, T, D, cl, Classic, someone, world, indoor, grade, loadMouth, renderShot, loadFonts, mat, box, sign, texturize, direct, captions } from '../lib/blocky/index.js'
export default {
  async create({ THREE, scene }) {
    setThree(THREE); await loadFonts()                            // fonts and the surface textures
    const w = world({ kind: 'suburb', theme: 'autumn', seed: 47 }); scene.add(w.root); scene.fog = w.fog
    const kid = Classic({ name: 'kid', skin: '#d3ab88', top: { kind: 'hoodie', color: '#e2574c' }, pants: { kind: 'jeans' }, hair: { style: 'messy', color: '#231d1c' } })
    scene.add(kid.root)
    return { kid, all: [kid], mood: w.mood, mouth: await loadMouth() }
  },
  update(env) {
    const { state: S, index: k } = env, d = direct(env, S.all)   // hides and resets everyone, returns the helpers
    if (k === 0) {
      const n = d.cut('town')                                     // sub-shot 0 until "town", then 1
      d.show(S.kid, 0, 0, 0, 0, n ? 'smile' : 'neutral'); d.talk(S.kid); d.pop(S.kid)
      if (n === 0) d.cam([2, 3, 14], [0, 4, 0], 52); else d.closeUp(S.kid, { side: 20 })
    }
    S.mood('day', [0, 0, 0]); d.done()                           // the light, then the frame check
  },
  render({ renderer, scene, camera, state }) { renderer.toneMapping = T.NoToneMapping; renderShot(renderer, scene, camera, { dof: state.dof }) },
  overlay(env) { grade(env.ctx, env.W, env.H, { look: env.state.tone ?? 'none' }); captions(env) },   // the colour finish first, then the captions
}
```

## The directing helpers
`const d = direct(env, cast)` at the top of `update`. `cast` is every character the scene made. It hides them all, resets their poses (everyone breathes and looks around by default) and gives you:
- **`d.shot(kind, c, { side, at, with })` frames people for you, use it for most shots:** `'face'` close-up, `'medium'` waist up, `'wide'` whole body and the place, `'low'` hero angle from below, `'top'` looking down, `'ots'` over c's shoulder `{ at: other }`, `'two'` both in frame `{ with: other }`. `side` = degrees around them. Heads never get cut and the face sits in the upper half, clear of the captions. Use `d.cam` only for shots the presets can't make (a car arriving, a roof, a crowd from above).
- **The framing check:** the video refuses a shot where the person talking has their face out of frame, or where nobody is in the shot at all. For a shot of a thing (a frog, a sign, someone's shoes while they talk off-screen) call `d.allowEmpty()` in that shot.
- **Big numbers (`d.big`)** are for a counted number the line says, and never on two lines in a row (the check refuses both). Most lines have none.
- `d.show(c, x, y, z, turn, face)`: show a character. `turn` in degrees, 0 faces the camera side (+z).
- `d.cam(position, lookAt, lens, focus)`: place the camera. `focus` is the distance to the subject for a soft background, or leave it out for a sharp frame.
- `d.closeUp(c, { side, tight, low, lens })`: a close-up that keeps the face in the upper half, clear of the captions, with a soft background. `side` = degrees around them, `tight` 0.8 = closer and 1.4 = a medium shot, `low` = a negative number for a low angle. Use it for every close-up. When something sits above the head (a balloon, a lamp, a hat), use `tight: 1.5` or place the camera with `d.cam` so it fits.
- `d.cut('word', 'word', ...)`: which sub-shot of this line we are in (0, 1, 2…), switching just before each word is said. A word that appears twice: `['the', 2]` is its second use (counting starts at 1). Words are matched without punctuation, so "I'm" is `'im'`. Afterwards `d.u` is the seconds since that sub-shot began.
- `d.at('word')`: when a word is said in this line (seconds), for timing an action inside a shot.
- `d.walk(c, x, y, z, turn, face, speed)`: someone walking (speed 2.4) or running (5): shown at x, y, z when the sub-shot starts and moved the way they face, legs in step. **Never `c.walk()` alone**: the legs move but the body stays put, and the video refuses that (`walks on the spot`). Set `c.treadmill = true` only when they really walk in place (a treadmill).
- `d.walkTo(c, [x0, z0], [x1, z1], face, speed, y, delay)`: walk from one spot to another **around** walls, props, cars and houses (the engine finds the route, turns at corners, legs in step, stops on arrival). Use it whenever anything stands between the two spots; `d.walk` only goes straight.
- `d.fall(c, { x, y0, z, at, g, vx, vz, faces })`: a real fall with gravity, landing on whatever is really below (a roof, a car, a loaf) with a squash and bounces; arms up in the air like the game. `at` is a word, seconds into the line, or `{ global: seconds }` for a fall that spans lines (call it in every line it covers). `g` 60 = a quick fall, 25 = slow motion.
- `d.jump(c, [x, y, z], [x, null, z], at, { height, dur })`: a jump that arcs up and lands on whatever is at the target (`null` = find it: a car roof, a wall top). `dur` = land exactly that many seconds later (to hit a word).
- **One-shot effects on a word** (call them every frame of the shot, like everything else):
  - `d.burst(kind, at, [x, y, z], { n, size, color, spread, floor })`: `'sparkle'` (white 4-point stars: magic, something appearing), `'puff'` (dust or smoke: something vanishing, a landing), `'debris'` (blocks flying and landing: a crash), `'notes'` (music notes rising: humming, happy), `'feathers'` (drifting down: a bird), `'confetti'`.
  - `d.appear(obj, at, { size, pos })`: something drawn, wished or magic pops in from nothing with sparkles and a white glow. `d.vanish(obj, at, { size })`: it shrinks away in a puff.
  - `d.shake(at, strength)`: a quick camera shake on an impact (0.3 small, 0.8 big).
  - `d.knock(obj, at, { dir: [dx, dz], mode: 'tip' | 'fly', dist, height })`: a bin, cone, box or chair tips over, or flies with spin and lands.
  - `d.drive(car, [[x, z], ...], at, { speed, accel })`: a car drives a route (keep to the lanes), speeds up, turns smoothly at the corners and brakes to a stop at the end; wheels spin on models that have them.
- **Drawing on paper:** `const book = sketchbook({ w, h })` in `create()` (a spiral sketchbook with a pencil; put it on a desk), then `drawOn(book, strokes, k)` each frame with k going 0 to 1: the line draws itself under the pencil. Strokes are `[[x, y], ...]` lists in page units (0..1); `DOODLES.bird | flower | heart | star | sun | house` are ready-made. Film it top-down and close (the reference shot), then `d.appear` the real thing.
- **Insert cards:** `d.insert('clock', '11:59')` (big red digits on black), `'phone'` (a dark timer pill, "23:59:58"), `'note'` (paper with serif text, lines split by `\n`), `'sign'` (bold printed sign; `{ color: '#2e9c4a' }` for green). Use one for a shot of its own, cut on a word, then cut back to the world.
- **People never overlap:** bodies that overlap step aside a little each frame (crowd push-apart), so extras and cast don't stand inside each other.
- **The camera is never behind a wall:** if a wall or prop sits between `d.cam` and the subject, the engine swings the camera round (or a little higher) to a clear view. `d.camFree(...)` keeps it exactly where you put it (a deliberate shot through something).
- **People placed inside a prop are moved out:** `d.done()` nudges anyone standing inside a wall, hedge or prop to the nearest clear spot and logs a `MOVED:` line. Read those lines after `stills` and fix the placement in your scene (the nudge is a safety net, not a plan).
- `d.pop(c)` (a squash and stretch as the sub-shot starts), `d.hop(c)` (bouncing; marks them airborne), `d.gest(c)` (hands and head move while talking), `d.laugh(c)`, `d.talk(c)` (this character speaks the line; the mouth follows the voice).
- `d.sfx(name, at, { gain, pitch })`: a sound for a story moment the helpers can't see; `at` is a word of the line or seconds into it. Rules and the sound list are in look.md, Sound.
- `d.big(60)`: a big number at the top of the frame for this moment. It covers the top 30 %, so frame the subject below it.
- `d.done()`: call last. Runs the frame check on everyone visible and standing on the floor at height 0. If people stand on a raised floor (a roof, a stage), call `d.done(thatHeight)` in that shot instead of marking them airborne.

`captions(env)` in `overlay` draws the captions in this look, the big number and the frame-check banner.

Also from the index: `T` is three.js (use it anywhere, for example `new T.PointLight(...)`), `D(degrees)` turns degrees into radians, `cl(v)` clamps a number to 0..1.

For props: `box(parent, w, h, d, color, x, y, z)` makes a block; `sign(text, w, h, background, textColour)` makes a flat sign whose text always fits (split lines with `\n`). `mat(color)` is a plain material.

## Characters
`Classic({ name, skin, top, pants, hair, scale, seed })` returns a character. Add `c.root` to the scene.
- `skin`: a colour, e.g. `'#d3ab88'`, `'#e6c3a5'`, `'#b98b68'`, `'#8a6248'`.
- `skin` can be any colour: natural tones, or toy colours like yellow `'#f5cd30'`, white `'#f4f4f4'`, grey `'#b9bcc2'`, green `'#7fbf6a'`, blue `'#6fc7e8'`.
- `top`: `{ kind, color, inner?, ... }`. Kinds: `tee`, `hoodie`, `jacket` (`print: 'stars' | 'none'`), `plaid`, `suit` (shirt, tie, lapels, buttons; `tie: colour`), `zip` (zip hoodie over an `inner` shirt), `denim` (open jacket over an `inner` shirt; `stripes: true`), `vest` (work vest with reflective bands), `uniform` (collar, tie, badge), `letter` (`text: 'E'` in a box; `ink`, `box` colours), `stripes` (striped tee; `inner` = the stripe colour). Any top takes `crop: true` to show the waist and `sleeve: 0..1`.
- `pants`: `{ kind: 'jeans' | 'plain' | 'stripe' | 'shorts' | 'skirt', color, shoes?, sole? }`. Give shoes their own colour (white sneakers: `shoes: '#f2f2f2', sole: '#22242c'`).
- `hair`: `{ style: 'swept' | 'messy' | 'long' | 'bun' | 'none', color }`.
- `scale`: 1 for a teen or adult, 0.78 for a small child, 1.08–1.1 for a tall adult.

On a character `c`:
- `c.root.position.set(x, y, z)`, `c.root.rotation.y`, `c.root.visible`.
- `c.setFace(name)`. Names (`CLASSIC_FACES`): shock (dot eyes, big red O mouth), sweat (worried with a sweat drop), tear (one tear running down), shine (big glossy eyes with highlights, the cute look), mad (angry brows, toothy grin), neutral, smile, smirk, grin (wide open), happy, cute (big shiny eyes), wink, love (heart eyes), cool (half-closed eyes), frown, sad, worried, angry, teeth (gritted), determined, oh, talk, scream, shout, scared, dizzy, freezing, cry, glasses, sleep, epic (eyes rolled up, huge laugh), laugh (eyes shut, open laugh), squint (> <), bored (half-closed), cat (:3), tongue, stare (wide round eyes), star (star eyes), spiral (hypnotised), fang (sly, one fang), lash (lashes and blush), buck (buck teeth), shy, sharp (glaring, pointed teeth), fangs (eyes shut, open mouth with two fangs), xd (eyes squeezed shut, laughing), blank.
- `c.wear(kind, ...colours)` puts something on; a character can wear several. Colours are optional.
  - Hats: hardhat, cap, tophat, fedora, cowboy, beanie, slouchy, bucket, crown, halo, hood (covers the top half of the head; pair it with the `sharp` face).
  - Head: ears (cat), bunny, horns, antlers, mask (an animal head: `c.wear('mask', '#e8792c', '#f4f4f4')` is a fox, grey + white a wolf, black + white a panda), headphones, headband, bow, clips, flower.
  - Face: glasses (round), nerd (black frames), shades, goggles, moustache, beard.
  - Body: necklace, scarf, bowtie, bag (crossbody), backpack, cape, wings, badge, pet (a small animal on the shoulder), bracelet.
  - In the right hand: hammer, sword, wand, phone, book, balloon, lollipop. Raise the arm to show it: `c.pose({ armR: [70, 6] })`.
- **People nobody designed:** `someone(seed)` builds a whole random person (skin, outfit, colours, hair or hat, shoes, accessories, height) from one number. The same seed is always the same person. Use it for every extra and crowd member (`someone(seed, { scale: 0.8 })` to force an option). Design the main characters by hand; never reuse a seed from another video.
- `c.pose({ armL, armR, legL, legR, head, torso })`, in degrees. An arm or leg is `[swing, out]`: swing forward is positive (90 = straight ahead, 170 = up), out lifts it sideways and must be 0 or more (a negative value pushes the arm into the chest). `head` and `torso` are `[nod, turn, tilt]`.
- `c.walk(phase)` swings arms and legs (use `t * 9`); `c.idle(t)` breathes and looks around.
- `c.speak(mouthAt(state.mouth, globalT))` while that character is speaking, `c.speak(null)` otherwise.
- `c.airborne = true` when they hop, float or fall; the frame check then doesn't ask for feet on the floor.
- Extra blocks (a hat, a badge) can be added to `c.parts.head` or `c.parts.torso` as child meshes. The head is 1.24 wide and 1.2 tall, its centre is the head's origin, and its face looks along -z in its own space, so turn a hat by 180 degrees (`hat.rotation.y = Math.PI`). The torso is 2 × 2 × 1 with its origin at its centre.
- Parts you can attach to or hide: `head`, `torso`, `armL`, `armR`, `legL`, `legR`. On the torso, the front faces +z: a thing held in front goes at a positive z.
- Handy poses: hands over mouth `armL: [150, 8], armR: [150, 8], head: [-14, 0, 0]`; arms up `[170, 14]`; pointing `armR: [95, 0]`; shrug `armL: [30, 40], armR: [30, 40]`. A pose overrides only the parts it names; the rest keep breathing.

## Lip-sync
`const mouth = await loadMouth()` in `create` (reads `voice/mouth.json`, written by `node mouth.mjs` after the voice), then `mouthAt(mouth, globalT)` gives `{ open, wide, round }`.

## Places
`world({ kind, theme, seed })` returns `{ root, fog, mood, kind }`. Add `root` to the scene and set `scene.fog = fog`. **Every video picks a kind it has not just used**, so no two videos share a world.
- `kind: 'suburb'`: family homes with lawns, driveways, fences, trees. Same layout as the street below (pavement x -3.5..3.5, road x 3.5..17.5), but the homes sit 10 studs back and the lawns (x < -3.6) are walkable.
- `kind: 'meadow'`: open green land with real grass and flowers, hills, a dirt path along z at x -2..2, a pond at (17, -16), a fence at z -30. Flat for 45 studs around the origin. Stand people on the grass, not the path.
- `kind: 'plaza'`: the town square. Cobbles x -26..26, z -26..26; a fountain at (0, -8); the town hall with a clock at z -40; market stalls at x -15 and 15; shops around the edge.
- `kind: 'city'`: the shop street described below.
- **Building your own set inside the world** (a tower, a cinema, a field far down the street): call `w.clear(x, z, radius)` for that spot first, so no horizon hill, tree or far house stands inside it. Check the stills for anything poking through a set.
- Rooms: `indoor({ kind, at: [x, 0, z], seed, night?, wall?, floor? })` returns a furnished room to add to the scene. Kinds: `living`, `classroom`, `diner`, `library`, `bedroom`. It is 26 wide, 20 deep, 12 tall and open toward +z; people stand at z -6..8. Put each room far away (`at: [600, 0, 0]`, the next at 680) and pass that spot to `mood`.
- **The seed changes the place, not only its colours.** A suburb's houses, trees, street things (hydrant, bus stop, hoop, swing, bins) and parked vehicles; a meadow's path (straight, crossing, none), water (pond, stream, none), fence and its one landmark (windmill, barn, tent, well, rocks, orchard); a square's centrepiece (fountain, statue, big tree, clock, bandstand), hall (width, columns, tower or dome, name), paving and stalls all come from it. Render stills first to see what this seed gave you, then place the story around it.
- `vehicle(kind, colour)` builds one vehicle (front toward +z): `sedan`, `hatch`, `pickup`, `van`, `bus`, `taxi`, `police`, `icecream`, `truck`. The worlds park a seeded mix in the parking lanes (x 5.3 and 15.7, never near z 0); drive your own in the middle lanes (x 9 toward +z, x 12.2 toward -z; with Polyfork cars, one lane at x 10.5); a car pulling over stops at x 5.3 so nothing passes through a parked car. The frame check also sees vehicles: a car through a car, or a person inside a car, is a CLIP like any other (set `c.riding = true` for someone sitting in one). Sedans, hatches, taxis and police cars have a door: `car.userData.door(k)` (k 0 shut to 1 open).
- `home(r, CITY_THEMES.sunny, { w, h, wall, roof, tex: 'siding' | 'brick' | 'stone' })` is one house (front faces +z) for a place you build yourself; `pine(x, z, size, snowy)` and `grassField(root, r, colour, [x0, x1, z0, z1], skip)` add trees and real grass.
- **Texture on what you build:** call `texturize(group)` once after building props or a set from boxes, or they look flat next to the world. To choose a surface for one mesh set `mesh.userData.tex` first: `brick`, `stone`, `cobble`, `tiles`, `planks`, `wood`, `shingle`, `asphalt`, `concrete`, `grass`, `snow`, `sand`, `fabric`, `siding`.

### A living world: NPC people and cars (game logic)
`const L = life(w, { people: 5, cars: 3, seed, stage: [[x, z, radius], ...] }); scene.add(L.root)` in `create()`, then in `update()`: `const d = direct(env, [...cast, ...L.people])`, your shots, `L.step(env.globalT)`, `d.done()`.
- People walk the pavements on their own lanes: the world's solids (hedges, lamps, fences, mailboxes, houses) are measured, each lane keeps only its clear stretches, one NPC per lane, lanes 4.6 studs apart, and they turn round at the end of a stretch. They can never walk into each other or into the world.
- `stage` is where the story happens (the main characters' spots, with room around them): no lane enters it, so extras never walk into the shot's people.
- Cars drive one lane down the middle (x 10.5) in one direction at one speed, evenly spaced, wheels turning; `dir: 1` drives toward the camera, `-1` away; `cars: 0` when the story drives its own car down that lane. Cars are the world's `cars` models if it has them.
- Kinds: `suburb` and `city` (both pavements and the road), `plaza` and `meadow` (people only).
- This replaces hand-placed extras: don't `show()` extras on lawns, kerbs or by hedges; let `life()` place them.

### Real models for the story (Polyfork)
The look of the viral shorts comes from real modelled things, so every story gets its own buildings, vehicles and props from Polyfork (thousands of free models in themed kits, at real size). The sky and trees are already real (photo sky by day, photo firs and leafy trees); never build a house, car or big prop from boxes when a model exists.
1. **Find** what each place and line needs, in plain words: `node pf.mjs search school bus`, `node pf.mjs search haunted house --class building`, `--class vehicle | building | prop | character | animal`. `node pf.mjs kits [words]` lists themed kits (Neighborhood, School, Hospital, Restaurant Cafe, Office, Halloween Night, Christmas Village, Alpine and Snow, Campground & Lake, Construction Site, Coast and Boardwalk, Skatepark, New York City, Little Tokyo, Nature & Forest, Cozy Farm, Frontier Town...) and `node pf.mjs kit <kit-id>` shows every part of one. Things from one kit share colours and scale: build each place from one kit.
2. **Fetch** only what this story uses: `node pf.mjs get <id> <id> ...` (into `models/`, about 0.2 MB each). `node pf.mjs list` shows what the project has.
3. **Load and place** in `create()`: `await loadModels([...ids])`, then `const bus = model('yellow-school-bus-0fd3b6'); bus.position.set(10.5, -0.2, -20); scene.add(bus)`. Models stand on y 0, front toward +z, at real size (1 m = 2.9 studs; vehicles at 0.75 so they fit the street). `model(id, { height: 30 })` sets a height in studs instead; `{ night: true }` makes its windows and lamps glow.
4. **A whole street from them:** `world({ kind: 'suburb', theme, seed, houses: [3-5 house ids], cars: [3-5 car ids] })` builds the suburb's homes and parked cars from those models (seeded). Load them first.
5. **Moving parts:** `m.userData.part('door', 80)` opens a door (degrees), `part('wheel-fl', t * 400)` spins a wheel; `modelInfo(id).rigged` lists each model's parts and `modelInfo(id).describe` says what it is.
- **Lanes with models:** parked cars at x 5.3 and 15.7 (the world does this), a car you drive goes down the middle at x 10.5. The frame check sees model vehicles like any other.
- Every video picks new models: different houses, cars and props from the last video, chosen for this story's places.
- Never share the model files themselves; they are used to make the video.

### Props people hold, throw or land on; roofs; trees in the way
- **A prop someone holds, throws, catches or lands on** (a loaf, a ball, a mattress, a box they carry): set `prop.userData.carried = true` once in `create()`. The frame check then doesn't treat it as a wall. Everything else stays checked.
- **Hide a prop** with `prop.visible = false` on its group; hidden things never count for the frame check.
- **Hold something in a hand:** each frame, after `d.show(...)`, place it from the torso: `c.root.updateMatrixWorld(true); prop.position.copy(c.parts.torso.localToWorld(new T.Vector3(0, -0.3, 1.3)))` (in front of the chest; raise the arms with `c.pose({ armL: [70, 0], armR: [70, 0] })`). A pocket is `(0.6, -1.15, 0.6)`.
- **Someone on a roof or hanging from a gutter:** never guess the height. `roofAt(house, x, z)` gives the real height of the roof at that spot (a two-storey Polyfork house is ~19 studs at the eaves, a bungalow ~10). Pick a tall house for a big fall; you can swap a world's house for another model at the same spot (`m.position.copy(old.position); m.rotation.copy(old.rotation); old.visible = false`).
- **Trees between the camera and a set:** `w.clearTrees(x0, x1, z0, z1)` hides every tree inside that box. Check the stills for leaves across a face or a roof.
- **Small animals and props** come at real size (a pigeon is under 1 stud): give them a size with `model(id, { height: 2.6 })`.

### The shop street
`city({ theme, seed })` (or `world({ kind: 'city', ... })`, which also textures it) returns `{ root, fog, mood }`.
- Themes: `sunny`, `pastel`, `autumn`, `snow`, `night`. Any number as the seed gives a different street.
- People stand on the near pavement: x from -3.5 to 3.5, y = 0. The road is x 3.5 to 17.5 (y = -0.2); buildings start at x = -3.6 on the left and x = 24.6 across the road. The street runs from z = 40 down to z = -150.
- `mood('day' | 'dusk', [x, 0, z])` sets the light; pass where the shot looks so the shadows stay sharp there. Call it every frame.
- **Fog, night and glow inside a story.** The theme is set once, but you can change the light per shot after calling `mood`: for night fog, set `scene.fog.color`, `scene.fog.near` (8) and `scene.fog.far` (55) and lower `w.root`'s lights; set them back in shots that need day. A glowing thing is a mesh with `emissive` colour and `emissiveIntensity` 2–3, plus a `T.PointLight(colour, 40–120, distance 20–40)` next to it. Stronger lights wash out the whole frame.
- **What stands on the near pavement:** street lamps at x = 3 (every 26 studs) and planters at x = -2.4. Check stills for a lamp or planter in front of a face, and move the camera or the person.
- `snowfall()` returns flakes: add them, call `flakes.userData.update(globalT)` each frame.
- Other places (a classroom, a bedroom, a tower) are built in `create` from boxes at a spot far from the street (for example x = 200) and lit by the same light.

## Places from kits
Polyfork has ~55 themed kits, and each one is a whole place: floor tiles, wall modules, buildings, fences, furniture,
props, things that hang on walls or stand on tables, vehicles. `kitPlace()` lays a kit out from a seed, so the same
kit gives a different room or yard every time.

```
node pf.mjs kits school                 # find the kit
node pf.mjs place school-456eda         # fetch its parts, sorted: floor, ground, wall, window, building, edge,
                                         # inside, outside, onwall, small, vehicle  (prints each list)
```
```js
await loadKit('school-456eda')                                  // in create(), once per kit
const gym = kitPlace({ kit: 'school-456eda', mode: 'room', seed: 8, at: [300, 0, 0],
  must: ['vaulting-horse-…'],                                   // things the script names: always placed
  inside: ['gym-floor-mat-…', 'climbing-rope-…', 'ball-bin-…'] })// optional: only these props (ids from `place`)
scene.add(gym.root)
const yard = kitPlace({ kit: 'halloween-night-46ade4', mode: 'yard', theme: 'night', seed: 3 })
scene.add(yard.root); scene.fog = yard.fog; yard.mood('day', [0, 0, -6])
```
- **room:** 5–6 by 4 modules of 2 m (about 29–35 × 23 studs), open toward +z, 14 studs tall: the kit's floor tiles,
  its wall modules on three sides (some glazed), a painted band and a ceiling above, furniture along the walls,
  small things on tables, signs and clocks on the walls. Put rooms far from the outdoor place (x 300, 420, 540...).
  The camera can stand back past the open side: a shell hides the sky.
- **yard:** 9 × 7 modules of ground tiles, the kit's buildings along the back (one may turn in on a side), fences
  down the sides, props, maybe a parked vehicle, and the photo sky and forest of `world()`. It returns `fog`,
  `mood(name, at)`, `sun`, `hemi` like a world. Themes as for `world()`.
- **stage:** `[[x, z, r]]` in the place's own coordinates: no prop enters these circles. The default keeps the middle
  front clear (room: `[0, D * 0.12, …]`; yard: `[0, D * 0.1, 12]`), which is where the cast stands. Pass your own when
  the story happens elsewhere.
- **Replacing the kit's picks:** `floor`, `walls`, `windows`, `buildings`, `edge`, `inside` / `outside` (props),
  `onwall`, `small`, `vehicles`: lists of ids from `node pf.mjs place`. The sorting is by size and name, so a yard can
  get an indoor thing: if a prop is wrong for the place, pass the list you want.
- Returns `{ root, kind, kit, seed, size: [W, D], stage, plan, clear }`. `life(place, { people, seed })` walks extras
  through it (`plan` gives life its lanes). Every part is measured by the frame check like the rest of the world.
- `night: true` makes the models' lights glow (lamps, windows); a yard with `theme: 'night'` does it by itself.

## The place plan
story.json `"places"` says where every line happens and how it is built. The render checks it against what the scene
really built (`kitPlace`, `world`, `indoor` all report themselves) and against the person's recent videos
(`node render.mjs history`; the list is kept on this computer in `~/roblox-shorts/history.json`).
```json
"places": [
  { "name": "the ward", "lines": [1, "2-4"], "build": "kit:hospital-kit-2ebfcb", "models": ["hospital-bed-31132d"] },
  { "name": "the car park", "lines": [5, 6], "build": "world:city" },
  { "name": "her bedroom", "lines": [7], "build": "room:bedroom" },
  { "name": "the rocket", "lines": [8], "build": "hand", "why": "no kit or model has a rocket like the story's" }
]
```
- `lines`: line numbers (1 = the first line), or ranges like `"2-4"`. Every line needs a place.
- `build`: `kit:<kit-id>`, `world:<kind>`, `room:<kind>` or `hand` (with a `why`).
- `models`: the things the line names; each must really appear in the scene.
- The video refuses: a line with no place, a place the scene never built, a listed model never used, a `hand` place
  with no reason, the same place + theme + seed as a recent video, the same opening place (and theme) as one of the
  last two videos, or 60% of the models shared with the last video. `check` and `stills` print the same problems
  early. `"repeat": "<why>"` turns the history problems into notes, only for a follow-up the person asked for.

## Camera and finish
- Set `camera.fov`, `camera.near = 0.1`, `camera.far = 900`, call `camera.updateProjectionMatrix()`, then `camera.position.set(...)` and `camera.lookAt(...)`.
- Lenses: close-up 36–42, medium 44–50, wide 52–60.
- `renderShot(renderer, scene, camera, { dof })`: `dof` is the distance to the face for a soft background, or `null` for a sharp frame.
- `grade(ctx, W, H, { look, amount })` in `overlay`, before `captions`, gives the vivid game tone. `look` is a colour mood for the shot: `'none'`, `'night'` (blue-purple dark), `'danger'` (everything red on black: a monster, an alarm), `'memory'` (black and white: a flashback, "as it turns out"), `'grey'` (washed out), `'warm'` (golden hour), `'cold'` (winter blue). Set `state.tone` per shot in `update` (after `direct(...)`, which resets it); `amount` 0..1 fades it in. Use the `night` theme for a whole night story and the `night` look for one night scene.

## The frame check
`d.done()` checks every frame for body parts passing through each other, through another character or below the floor. Stills print problems as `CLIP:` lines, and `render.mjs video` refuses to finish while any frame has one. Two people side by side need their centres 4.4 studs apart or more; staggering them in depth helps.

## story.json
```json
{ "title": "In This Town, …", "format": "vertical", "fps": 30, "captions": false,
  "music": "audio/bg.mp3", "musicVolume": 0.12,
  "keys": ["balloon", "3", "pia"],
  "beats": [{ "line": "In this town, …", "style": "town", "scene": "<scene>" }] }
```
`"captions": false` turns off the engine's own captions, because the scene's overlay draws them in this look. A beat may also set `"pause"` (seconds of silence after it) and `"emotion"` (a delivery hint). The voice comes from step 4 only: the downloaded Kokoro Windows app, Google AI Studio in the browser, or the person's own audio file. `node voice.mjs import-audio <file>` fits one continuous narration file (WAV or MP3) to these beats and writes `voice/voice.mp3`, `voice/timing.json` and `voice/mouth.json`, which are what the renderer reads. Background music is fixed to `audio/bg.mp3` with the supplied track; the renderer loops it under the voice automatically. Keep `"music": "audio/bg.mp3"`, `"musicVolume": 0.12` and `"sound": { "bed": "none", "music": -14 }` in every story.

`"caption"` (optional) changes the caption style: `{ "words": 1 | 2 | 3, "keyColors": ["#ffd21f"], "keyCase": "lower" | "upper", "y": 0.66 }`. Several `keyColors` rotate from caption to caption. Three words only fit when the words are short: use 2 for long sentences. Examples: one word at a time `{ "words": 1 }`; coloured key word per line `{ "words": 3, "keyColors": ["#3fd04b", "#39a7ff", "#ff5fb0", "#ff4a3d", "#ffd21f"] }`; red caps key word `{ "words": 3, "keyColors": ["#ff4a3d"], "keyCase": "upper" }`.

## Story tricks
- **A label over a head:** `const tag = sign('DELETE', 2.2, 0.8, '#e0202a', '#fff'); tag.position.set(0, 1.5, 0); tag.rotation.y = Math.PI; c.parts.head.add(tag)`. It faces the way the character faces; for side shots add a second sign turned 180 degrees behind it.
- **One-colour world:** after `city(...)`, `const grey = (m) => { if (!m.color || m.isMeshBasicMaterial) return m; const c = m.clone(); c.map = null; c.color.set('#b9bcc2'); return c }; w.root.traverse((o) => { if (o.isMesh) o.material = Array.isArray(o.material) ? o.material.map(grey) : grey(o.material) })`. To "paint" one thing later, set that mesh's `material.color` (the sky keeps its own colour).
- **Creatures:** a `T.Group` of `box(...)` parts (body, head, four legs), eyes as small dark boxes or a `sign()` face. Swing legs with `Math.sin(globalT * 9)`. Scale the group for a giant version. Creatures are not in the frame check: check stills that they don't pass through people.
- **Riding, stilts, ziplines:** set `c.airborne = true`, set `c.root.position.y` to the height, and place the prop (two long thin boxes under the feet, a seat, a rope) at the same spot every frame.
- **Puffs, splats, dissolves:** make 10–20 small meshes once in `create`; in `update` place them from the time since the trigger word (`raw - d.at('word')`): grow and fade for smoke, fly up and shrink for a dissolve. Hide them outside that window.
- **A storyteller (he/she) video:** nobody calls `d.talk()` except on a quoted line.
