# AGENTS.md — Wisps

## Phase 0 — Visual Specification

### Goal

Translate the Wisp image into explicit design rules before implementation.

### Tasks

Document:

### Body silhouette

Define the canonical Wisp body.
Required characteristics:

- approximately circular lower body
- pointed upper-right projection
- notch/cutout on right side
- flat vector geometry
- no outline by default

Create a normalized coordinate system.
Example:

```text
canvas: 100 × 100
body bounds: 12..88
face safe area: x=30..70, y=40..72
```

Do not hardcode pixel coordinates throughout the renderer.
Create reusable normalized geometry.

### Body colors

Create a curated palette rather than generating arbitrary RGB values.
Initial palette should include families similar to:

- cyan
- pink
- lime
- orange
- deep blue
- yellow
- violet
- pale gray
- sky blue

Each color should have:

```ts
interface WispColor {
  id: string
  body: string
  preferredFaceColors: string[]
}
```

Face colors must maintain adequate visual contrast.

### Pixel grid

Define a facial grid.
For example:
1 face unit = 4% of canvas width

All facial features should snap to this grid where practical.
This is important for maintaining the pixel-character aesthetic.

### Deliverable

Create:

```text
docs/
  visual-spec.md
```

Include:

- silhouette rules
- coordinate system
- palette
- facial grid
- safe zones
- examples

## Phase 1 — Deterministic Identity

### Goal

Generate a stable Wisp genome from any string.

### Tasks

Create:

```ts
createWisp(input: string): WispGenome
```

### String normalization

Define canonical normalization.
Suggested rules:

```ts
input.trim().normalize("NFKC")
```

Do not lowercase unless identity semantics explicitly require it.
"user@example.com" and "USER@example.com" may legitimately be distinct identifiers.

### Hashing

Use a stable cross-platform hash.
Good options:

- SHA-256
- xxHash
- MurmurHash
- FNV-1a

For maximum implementation portability, SHA-256 is acceptable even though cryptographic strength is unnecessary.
Do not use:

```ts
Math.random()
```

Do not use runtime-dependent string hashing.

### Seeded PRNG

Convert hash output into deterministic random values.
Provide functions such as:

```ts
rng.float()
rng.int(min, max)
rng.pick(array)
rng.bool(probability)
```

Every generated trait must ultimately derive from the seed.

### Versioning

Genome generation MUST be versioned.
Example:

```ts
createWisp(input, {
  version: 1
})
```

This prevents future trait changes from silently changing every existing avatar.
The seed namespace should include the version:

```text
wisp:v1:<normalized-input>
```

### Tests

Verify:

- same string + same version = identical genome
- different strings usually produce different genomes
- different versions may produce different genomes

Create golden test vectors.
Example:

- "alice" -> expected genome
- "bob" -> expected genome
- "agent-01" -> expected genome

## Phase 2 — Body Renderer

### Goal

Render the canonical Wisp silhouette.
Do not implement expressions yet.

### Tasks

Implement:

```ts
renderBody(genome)
```

Prefer vector rendering.
Recommended targets:

- SVG first
- Canvas later if required
- native equivalents may follow

SVG is ideal because Wisps need to scale from:

- 16px
- 24px
- 32px
- 64px
- 128px
- 256px

without becoming blurry.

### Trait possibilities

Initially keep body variation limited.
Possible deterministic traits:

- body.color
- body.notchDepth
- body.tipAngle
- body.roundness

Variation should remain subtle.
Every generated character must still clearly belong to the Wisp species.
Avoid procedural geometry so extreme that some seeds stop looking like Wisps.

### Requirement

At 24×24 px:

- silhouette must still be recognizable
- eyes must remain readable
- accessories must not overwhelm the body

## Phase 3 — Eye System

### Goal

Build the first expressive facial system.
Eyes should carry most of the personality.

### Eye primitives

Support reusable shapes:

- square
- rectangle
- pixel-dot
- pixel-cross
- half-eye
- wide-eye
- narrow-eye

Initially prioritize the styles visible in the reference:
#### Standard eyes

```text
▮ ▮
```

#### Tiny eyes

```text
▪ ▪
```

#### Cross eyes

Pixelated:

```text
+ +
```

#### Heart eyes

Pixelated hearts.

#### Sunglasses base

Technically an accessory but intersects the eye region.

#### Uneven eyes

Useful for goofy expressions.
Example:

```text
■ ▪
```

### Eye traits

Genome-controlled:

```ts
interface EyeTraits {
  style: EyeStyle
  spacing: number
  width: number
  height: number
  color: string
}
```

Expression-controlled:

```ts
interface EyeExpression {
  openness: number
  offsetY: number
  leftVariant?: EyeVariant
  rightVariant?: EyeVariant
}
```

Keep identity and expression separate.

## Phase 4 — Eyebrows

### Goal

Add a separate eyebrow layer.
Eyebrows should be optional.
Do not merge brows into eye geometry.

### Brow styles

Create primitives:

- none
- flat
- raised
- angled-in
- angled-out
- asymmetric
- single-brow
- pixel-block

Examples:

```text
Neutral:
—   —
▮   ▮
```

```text
Angry:
\   /
▮   ▮
```

```text
Worried:
/   \
▮   ▮
```

```text
Skeptical:
—   /
▮   ▮
```

```text
Surprised:
—   —

▮   ▮
```

The implementation should manipulate:

- rotation
- offsetX
- offsetY
- length
- thickness

rather than drawing unique geometry for every expression.

## Phase 5 — Mouth Expressions

### Goal

Create a reusable mouth grammar.
The mouth should be assembled from simple pixel/vector primitives.

### Initial mouth set

Implement:

```ts
enum MouthStyle {
  None,
  Flat,
  Smile,
  Frown,
  Open,
  TinyOpen,
  Grin,
  ToothGap,
  SideSmile,
  O,
  Zigzag,
  Tongue,
}
```

Suggested expressions:

| Expression | Geometry | Notes |
| --- | --- | --- |
| Neutral | — | Flat mouth. |
| Happy | ⌣ | Pixel approximation. |
| Sad | ⌢ | Frown. |
| Shocked | □ | Open mouth. |
| Tiny shocked | ▪ | Tiny open mouth. |
| Grin | ▭ | Potential internal teeth blocks. |
| Side smirk | Asymmetrical horizontal geometry | Side smile. |
| No mouth | None | Important: some Wisps should communicate entirely through their eyes. |

### Mouth traits

Identity:

```ts
interface MouthTraits {
  defaultStyle: MouthStyle
  width: number
  color: string
}
```

Runtime expression may override the style.

## Phase 6 — Expression Composer

### Goal

Stop thinking in isolated eyes/mouths and introduce semantic expressions.
Create:

```ts
enum Expression {
  Neutral,
  Happy,
  Excited,
  Angry,
  Annoyed,
  Sad,
  Worried,
  Surprised,
  Confused,
  Sleepy,
  Focused,
  Smug,
  Silly,
  Love,
}
```

Each expression should map to:

```ts
{
  eyes,
  brows,
  mouth
}
```

Example:

```ts
Focused = {
  eyes: {
    openness: 0.7
  },
  brows: {
    style: "angled-in"
  },
  mouth: {
    style: "flat"
  }
}
```

A Wisp's genome should slightly influence how expressions appear.
For example:

- Happy Wisp A -> tiny smile
- Happy Wisp B -> wide grin
- Happy Wisp C -> no mouth + raised eyes

This prevents every Wisp from becoming the same emoji with different colors.

## Phase 7 — Accessories

### Goal

Introduce recognizable accessories without damaging readability.
Accessories should be their own layer.
Render order:

- back accessories
- body
- face
- front accessories

### Initial accessories

Implement:

- none
- dark shades
- round glasses
- square glasses
- tiny cap
- headband
- antenna
- earpiece
- bandage
- bow
- visor

Start small.
The reference image's dark shades should be one of the first.

### Dark shades

Should support:

- frameWidth
- lensWidth
- bridgeWidth
- verticalOffset

Avoid drawing them as an arbitrary image asset.
Make them geometry-based.

### Accessory probability

Most Wisps should NOT have accessories.
Suggested initial weighting:

```text
none:      65%
glasses:   10%
shades:     8%
headband:   5%
cap:        4%
antenna:    4%
other:      4%
```

This keeps accessories special.
Do not rely on equal probability.

## Phase 8 — Trait Constraints

### Goal

Prevent ugly or impossible combinations.
Pure random trait selection will produce bad avatars.
Implement compatibility rules.
Examples:

- heart eyes + sunglasses -> invalid
- cross eyes + visor -> invalid
- very tall eyes + low cap -> potentially overlapping
- yellow body + white eyes -> insufficient contrast
- large accessory + oversized brows -> visual collision

Create:

```ts
validateGenome(genome)
repairGenome(genome)
```

Prefer deterministic repair.
Never reroll with uncontrolled randomness.
For example:

```text
if incompatible(accessory, eyeStyle):
    accessory = deterministicFallback(...)
```

## Phase 9 — Rarity and Weighted Traits

### Goal

Make generated populations visually interesting.
Not every trait should be equally common.
Example:

```ts
weightedPick([
  ["standard", 40],
  ["tiny", 20],
  ["wide", 15],
  ["cross", 8],
  ["heart", 5],
  ["unusual", 2],
])
```

Think in terms of:

- common
- uncommon
- rare

but do not expose NFT-like rarity mechanics unless there is a product reason.
The objective is visual diversity, not artificial scarcity.

## Phase 10 — Semantic Seed Streams

### Goal

Prevent future additions from changing existing traits.
DO NOT generate everything from one sequential RNG stream:

```ts
body = rng.pick(...)
eyes = rng.pick(...)
mouth = rng.pick(...)
```

because inserting a new trait earlier changes everything afterward.
Instead derive independent namespaced streams:

```ts
hash(input + ":body")
hash(input + ":eyes")
hash(input + ":mouth")
hash(input + ":brows")
hash(input + ":accessory")
```

Better:

```ts
rng.for("body")
rng.for("eyes")
rng.for("mouth")
rng.for("brows")
rng.for("accessory")
```

Example:

```ts
SHA256("wisp:v1:bob:eyes")
```

This is a major architectural requirement.
Adding `horns` later should not unexpectedly change Bob's eye style.

## Phase 11 — Public API

### Goal

Make the avatar system pleasant to integrate.
Example API:

```ts
const wisp = createWisp("agent-01")
```

Rendering:

```ts
renderWisp(wisp)
```

Convenience:

```ts
renderWispFromString("agent-01")
```

Expressions:

```ts
renderWisp(wisp, {
  expression: "focused"
})
```

Sizing:

```ts
renderWisp(wisp, {
  size: 64
})
```

Potential component API:

```tsx
<Wisp
  seed="agent-01"
  size={48}
  expression="focused"
/>
```

Override traits:

```tsx
<Wisp
  seed="agent-01"
  accessory="shades"
/>
```

Overrides must not mutate the genome.

## Phase 12 — Serialization

### Goal

Allow Wisps to be persisted or transferred.
Provide:

```ts
serializeWisp()
deserializeWisp()
```

Example:

```json
{
  "schema": "wisp",
  "version": 1,
  "seed": "agent-01"
}
```

Avoid serializing every generated trait unless required.
Ideally the seed and version are sufficient to reconstruct identity.
Optionally expose:

```ts
getWispGenome(seed)
```

for debugging.

## Phase 13 — Avatar Collections

### Goal

Support swarms.
Example:

```ts
createWisps([
  "planner",
  "researcher",
  "coder",
  "reviewer"
])
```

Make sure visually similar neighboring seeds do not obviously correlate.
Do not generate swarm members using:

```ts
seed + 1
seed + 2
```

unless each goes through full hashing.
A swarm UI should remain distinguishable at:

- 24px
- 32px
- 40px

## Phase 14 — Agent Roles

### Goal

Optionally allow external systems to communicate agent role without changing identity.
Identity:

```tsx
seed="agent-21"
```

Role:

- coder
- researcher
- planner
- reviewer

Do NOT permanently derive role from seed.
Instead role may influence temporary presentation:

```tsx
<Wisp
  seed="agent-21"
  role="coder"
/>
```

Potential subtle role indicators:

- tiny badge
- accessory
- background dot
- status ring

Keep this optional.
A Wisp should still be identifiable when its role changes.

## Phase 15 — Status States

### Goal

Represent agent activity independently from emotion.
Create:

```ts
enum ActivityState {
  Idle,
  Thinking,
  Working,
  Waiting,
  Success,
  Warning,
  Error,
  Offline,
}
```

Do not immediately animate these.
Initial states can modify:

- expression
- small status glyph
- opacity
- background marker

Example:

- Thinking -> focused expression
- Waiting -> neutral
- Success -> happy
- Error -> concerned

Expression and status should still be independently overrideable.

## Phase 16 — Accessibility

### Goal

Make Wisps usable beyond color alone.
Do not use body color as the only differentiator.
Differences should come from combinations of:

- eye style
- brows
- mouth
- accessory
- body geometry

SVG output should support:

```html
role="img"
aria-label="Wisp avatar for Research Agent"
```

Ensure sufficient face/body contrast.
Test common color-vision deficiencies.

## Phase 17 — Snapshot Testing

### Goal

Protect the visual system against accidental changes.
Create deterministic snapshot fixtures.
Render representative seeds:

- alice
- bob
- planner
- researcher
- coder
- reviewer
- wisp-test-001
- wisp-test-002

Generate:

```text
tests/fixtures/v1/
```

Store SVG snapshots or rasterized reference images.
Changes to deterministic output should require explicit review.

## Phase 18 — Gallery Playground

### Goal

Build a development interface before animation.
Include:

- seed input
- random seed button
- expression picker
- accessory picker
- size picker
- version selector
- grid of generated Wisps

Useful debug view:

- seed
- hash
- body
- eyes
- mouth
- brows
- accessory

Allow generating:

- 10
- 50
- 100
- 500

Wisps at once.
This is essential for spotting:

- collisions
- repetitive designs
- ugly combinations
- unreadable small avatars

## Phase 19 — Collision Analysis

### Goal

Measure visual diversity.
Generate at least:
10,000 seeds

Analyze trait distributions.
Check:

- body color frequency
- eye style frequency
- accessory frequency
- expression defaults
- identical genome frequency

You do not need every genome to be globally unique.
You DO want visually identical combinations to be reasonably uncommon.
Increase trait entropy only where necessary.
Do not add meaningless variation simply to increase combinations.

## Phase 20 — Animation Architecture Only

### Goal

Prepare for animation without implementing most animation yet.
Do NOT create a giant animation system.
Introduce:

```ts
interface AnimationState {
  elapsed: number
  blinkPhase?: number
  lookDirection?: Vector2
}
```

Renderer should be capable of receiving dynamic offsets.
For example:

```ts
renderEyes({
  openness,
  offsetX,
  offsetY
})
```

instead of rendering static eyes only.
This means blinking later does not require rewriting the eye system.

## Phase 21 — First Animation: Blink

Implement only after the static system is stable.
Blinking sequence:

- open
- open
- half
- closed
- half
- open

Blink timing should vary slightly per Wisp.
Derive blink personality from the seed:

- blink interval
- blink speed
- double-blink likelihood

Do not synchronize an entire swarm.
A swarm where every character blinks simultaneously will look artificial.

## Phase 22 — Eye Tracking

### Goal

Allow a Wisp to look toward:

- mouse cursor
- active task
- another Wisp
- notification

Implement normalized:

```ts
lookAt(x, y)
```

Clamp eye movement within the eye sockets.
Movement should be subtle.

## Phase 23 — Microanimations

Only after blinking works.
Candidate animations:

- idle bob
- tiny squash
- blink
- look-around
- brow twitch
- mouth twitch
- sunglasses adjustment
- sleep droop
- surprised recoil
- thinking glance

Keep motion restrained.
The Wisp should feel alive without becoming visually distracting.

## Phase 24 — Swarm Animation

### Goal

Animate groups without chaos.
Each Wisp should have independently seeded timing.
Example:

- agent A blink offset: 1.2s
- agent B blink offset: 4.8s
- agent C blink offset: 2.9s

Avoid:

- all agents bobbing simultaneously
- all agents blinking simultaneously
- all agents reacting identically

Swarm behavior should look loosely coordinated rather than synchronized.

## Phase 25 — Version 1 Freeze

Before declaring v1 stable:
Freeze:

- hash strategy
- normalization
- trait namespaces
- body geometry
- palette IDs
- trait weights
- compatibility rules

Document all of them.
Any change capable of modifying deterministic identity requires:
Wisp schema v2

Do not silently alter v1.

## Project Goal

Build a deterministic avatar system called Wisps.

A Wisp is a small, stylized digital creature derived from a string such as:

- user ID
- email
- username
- agent name
- UUID
- repository name
- task ID

The same input string must always generate the same Wisp for the same avatar schema version.

Wisps should feel:

- simple
- expressive
- recognizable at very small sizes
- visually distinct from one another
- deterministic
- composable
- animation-ready

The initial visual language is based on:

- a rounded asymmetric body
- a pointed upper-right "flame/comet" silhouette
- pixel-like facial geometry
- high-contrast flat colors
- simple accessories
- expressive eyes, eyebrows, and mouths

The renderer should not depend on AI image generation.

---

## Core Architecture

Prefer a layered avatar model:

```text
Input String
↓
Normalization
↓
Stable Hash
↓
Seeded RNG
↓
WispGenome
↓
WispExpression / WispState
↓
Renderer
↓
Static Avatar
```

Later:

```text
WispGenome

- WispState
- AnimationState
  ↓
  Animated Renderer
```

Keep identity traits separate from temporary expressions.

Example:

```ts
interface WispGenome {
  version: number
  seed: string

  body: BodyTraits
  eyes: EyeTraits
  brows: BrowTraits
  mouth: MouthTraits
  accessory: AccessoryTraits | null
}
```

Temporary state:

```ts
interface WispState {
  expression: Expression
  activity: ActivityState
  animationPhase?: number
}
```

Do not bake temporary emotions into the deterministic genome unless explicitly requested.

## Initial Folder Structure

Suggested:

```text
wisp/
├── src/
│   ├── core/
│   │   ├── hash.ts
│   │   ├── rng.ts
│   │   ├── seed.ts
│   │   └── genome.ts
│   │
│   ├── traits/
│   │   ├── body.ts
│   │   ├── colors.ts
│   │   ├── eyes.ts
│   │   ├── brows.ts
│   │   ├── mouths.ts
│   │   ├── accessories.ts
│   │   └── weights.ts
│   │
│   ├── expressions/
│   │   ├── expression.ts
│   │   └── presets.ts
│   │
│   ├── render/
│   │   ├── body.ts
│   │   ├── eyes.ts
│   │   ├── brows.ts
│   │   ├── mouth.ts
│   │   ├── accessories.ts
│   │   └── svg.ts
│   │
│   ├── validation/
│   │   ├── constraints.ts
│   │   └── repair.ts
│   │
│   ├── animation/
│   │   ├── state.ts
│   │   ├── blink.ts
│   │   └── eyes.ts
│   │
│   └── index.ts
│
├── playground/
├── tests/
│   ├── determinism/
│   ├── snapshots/
│   └── distributions/
│
├── docs/
│   ├── visual-spec.md
│   ├── genome-v1.md
│   └── expressions.md
│
└── AGENTS.md
```

## Important Agent Rules

When modifying this repository:

1. Preserve deterministic output unless the task explicitly introduces a new schema version.
2. Never use uncontrolled randomness in avatar generation.
3. Never make an existing trait depend on the order in which unrelated traits are generated.
4. Prefer namespaced deterministic seed streams.
5. Keep identity traits separate from:
   - expression
   - status
   - animation
   - agent role
6. Keep facial geometry grid-aligned whenever practical.
7. Test avatars at 24px before approving new facial features.
8. Avoid arbitrary SVG paths where reusable geometric primitives are sufficient.
9. Do not add traits purely to increase combinatorial count.
10. Do not let accessories obscure the entire face.
11. Any deterministic-output change requires snapshot review.
12. Any breaking deterministic-output change requires a new Wisp schema version.
13. Static rendering must remain usable without the animation package.
14. Do not introduce animation until static expression composition is stable.
15. Prefer small composable renderers over one monolithic render function.
