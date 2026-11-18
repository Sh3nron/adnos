<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# Ni'ma: the AdnOS design language

**نعمة · grace** · Draft 0.1 · 2026-11-18

> [!NOTE]
> An early draft. The principles are meant to hold; the exact values will change as we build.

Ni'ma is designed the way a great building is: by someone who cares about every joint. Its lineage has three parts:

- **webOS on the Palm Pre** gives it its *behavior*: cards, the gesture area, the Wave, Just Type, notifications that never interrupt.
- **Modern iOS** gives it its *physics*: depth, translucent materials, springs that feel alive.
- **Islamic architecture of the Levant, the Middle East and West Asia** gives it its *fabric*: geometry, light, material and the passage through thresholds. Think of the Dome of the Rock's tilework, Iznik and Isfahan glazes, Jerusalem limestone, the marble and gold of the great courtyards.

---

## 1. Principles

### 1.1 Two readings

Every surface is read twice. **From a distance** it is calm: one color, one clear shape, generous space. **Up close**
the intricacy appears: hairline lattice in the header, a star where a divider begins, the shadow
falling the way the sun does right now. Detail never competes with content; it rewards the person who pauses.

*Rule:* ornament lives at **2–6% contrast** and at **thresholds** (headers, edges, empty states, the space between
cards). It never sits under text you're reading.

### 1.2 Geometry before decoration

Islamic pattern is not decoration added to a surface. It is a geometry the surface is built from. Ni'ma's layout,
spacing and ornament come from one construction: the **square and the square turned 45°**, which together make the
eight-pointed star (*khatam*). From that comes the **√2 scale**.

### 1.3 Light is always present

Great buildings are designed so light is never absent, not even at night, and so it is shaped rather than merely
admitted. In Ni'ma, light is the physics of the whole system, not a theme. The interface is lit by the real sun
(section 4) and obeys the laws in section 4.1. There is no pure black and no dead grey. Shadow is just light that has
traveled further.

### 1.4 Every transition is a threshold

Opening an app is passing under an arch into a room. Going back is stepping out into the courtyard. Motion explains
*where you are*.

### 1.5 Calm and instant

Inherited from webOS: nothing loads in front of you, nothing pops up uninvited, and notifications wait at the edge.

---

## 2. Geometry

### 2.1 The √2 scale

Spacing, radii and type sizes step by √2 (≈1.414), the ratio of a square's side to its diagonal: the same ratio
inside the eight-pointed star.

| Token | px | | Token | px |
|---|---|---|---|---|
| `space-1` | 4 | | `space-5` | 16 |
| `space-2` | 6 | | `space-6` | 23 |
| `space-3` | 8 | | `space-7` | 32 |
| `space-4` | 11 | | `space-8` | 45 |

### 2.2 Shapes

| Shape | Where | Built from |
|---|---|---|
| **Card** | Apps in Itlala, list groups | Rounded rectangle, radius `space-6` |
| **Icon** | App icons, the dock | webOS shapes: rounded squares, with circles for system actions. Icon shapes are not altered |
| **Star** (*khatam*) | Divider marks, selection, the lattice | Square + square rotated 45° |
| **Arcade** | Wallpapers, the card-view floor | A rhythm of pointed arches |

### 2.3 The lattice

A fine *khatam* lattice (square + rotated square per cell, joined to its neighbors) is drawn at hairline weight.
It appears in headers, the gesture area, wallpapers and the card-view sky, at 2–8% opacity depending on the light.

---

## 3. Materials

| Material | Role | Inspired by |
|---|---|---|
| **Marble** | Main surfaces (warm white, very faint veining) | Courtyard floors and walls |
| **Limestone** | Secondary surfaces, dividers, quiet buttons | Jerusalem stone |
| **Glaze** | Interactive accents: toggles, primary buttons, app icons. A soft gloss, like glazed tile (webOS's glossy buttons, reinterpreted) | Iznik and Isfahan tilework, Dome of the Rock |
| **Gold leaf** | Hairlines, selection, the light of the gesture area. Used sparingly | Capitals and calligraphy bands |
| **Lapis night** | System chrome: gesture area, menus, notifications, Wave | The dusk sky over the domes |
| **Jali glass** | Translucent overlays (Just Type, Wave, quick launch): blurred glass carrying a faint lattice where light passes | Mashrabiya screens |
| **Water** | Reflections beneath cards in Itlala | Courtyard pools |

---

## 4. Light

### 4.1 The physics of light

These laws apply to every component. Each one comes from how the buildings that inspire Ni'ma actually handle light.

| Law | In the architecture | In the interface |
|---|---|---|
| **Light comes from above** | Domes, oculi and clerestory windows pour light down onto the floor | Every surface is lit from its top edge: a soft top-to-bottom falloff and a fine highlight along the top rim. Nothing is flat-lit |
| **Light gathers at the threshold** | The far end of an arcade is always the brightest point; you walk toward it | The destination glows. An opening card brightens from within; scroll edges dissolve into warm light, never into darkness; the focused card in Itlala carries the most light |
| **Light pools and reflects** | Polished marble floors and courtyard pools mirror what stands on them | Cards in Itlala reflect in a pool below them; elevated surfaces cast a faint pool of reflected light, not only a shadow |
| **Light is filtered** | Mashrabiya and jali screens break sunlight into patterned light on the floor | Translucent layers are jali glass: blur plus a faint lattice. Somewhere on the screen, patterned light falls across a surface at the sun's angle, visible only when you look |
| **Edges catch light** | Gold capitals and carved edges catch rim light and glint | Interactive edges (the selected item, a toggle's knob rim, the gesture-area lamp) carry a fine gold or warm highlight that brightens when touched |
| **Shadows are soft, graded and colored** | Niches and vaults fall off gently into warm, luminous shade; bounce light fills them | Two-layer shadows: a tight contact shadow plus a wide ambient one, tinted by the light (warm at Maghrib, cool at Fajr, lapis at Isha) and never neutral grey. Concave things (niches, pressed buttons, inset fields) get soft inner shading |
| **Shadows follow the sun** | The same courtyard reads differently at every hour | Shadow direction and length come from the current light: west at dawn, short at noon, east at dusk, soft and directionless at night |
| **Bounce light carries color** | Turquoise tile and gold leaf tint the stone around them | Surfaces next to strongly colored elements pick up a trace of that color |
| **Night is lit, not dark** | Lamps, moonlight and glowing windows keep the building alive | Isha is a lamp-lit theme: warm glow from gold elements and the gesture area, moonlight on top edges, lit windows in the arcade |
| **Depth is a procession of thresholds** | Arches within arches recede toward the light | Hierarchy is nested thresholds. Each level deeper is one more arch in, and stepping back moves you toward the courtyard |

### 4.2 The four lights: the interface follows the sun

AdnOS has four lights, named for the times of day. **Auto** follows the local sun.

| Light | When | Character |
|---|---|---|
| **Fajr** (dawn) | First light | Cool marble, pale blue to peach sky, shadows fall west |
| **Dhuhr** (day) | Midday | Bright warm marble, short shadows straight down |
| **Maghrib** (dusk) | Sunset | Rose-warm marble, indigo-to-rose sky, shadows fall east |
| **Isha** (night) | Night | The dark theme: lapis surfaces, gold detail, a crescent and stars over the arcade |

Shadow direction, wallpaper sky and chrome tone all follow the light. It's barely noticeable, until you notice it.
"Reduce motion" and a fixed light are always available.

---

## 5. Palette

Text values are chosen for at least 4.5:1 contrast on their surfaces.

| Token | Dhuhr | Isha | Source |
|---|---|---|---|
| `marble` (ground) | `#F5F1E8` | `#0D1630` | Courtyard marble / night sky |
| `alabaster` (surface) | `#FFFDF8` | `#15214A` | Polished marble |
| `limestone` (surface 2) | `#EFE6D6` | `#1C2A57` | Jerusalem stone |
| `seam` (line) | `#E3D6BE` | `#26355F` | Stone joints |
| `ink` (text) | `#1A2238` | `#EDE6D3` | Calligrapher's ink |
| `ink-soft` | `#5A6175` | `#A9B0C4` | |
| `lapis` (primary) | `#23458F` | `#8FB0F0` | Cobalt glaze, Isfahan |
| `turquoise` (on / active) | `#17767D` | `#5CC4C9` | Dome of the Rock tile |
| `gold` (detail) | `#B8892A` | `#E2B84A` | Gold leaf |
| `cypress` (affirm) | `#2F7D55` | `#5FBF8E` | Garden cypress, Iznik green |
| `iznik-red` (destructive) | `#B23A2A` | `#F08A78` | Iznik coral red |
| `night` (chrome) | `#0F1B3A` | `#070D1F` | Lapis dusk |

---

## 6. Type

- **Latin:** Source Sans 3, a humanist sans and the open-source heir to webOS's Prelude. Light weights for display, semibold for headings.
- **Arabic:** Noto Naskh Arabic for display accents (app names, wordmark); a UI Arabic face to be chosen.
- Sizes follow the √2 scale: 11 · 16 · 23 · 32 · 45.
- Arabic and Latin share a baseline rhythm; bilingual labels set the Arabic in gold at a smaller size, as a calligraphic echo.

---

## 7. Components (first set)

| Component | Ni'ma treatment |
|---|---|
| **Gesture area** | Lapis night strip with a faint lattice and a warm lamp-light that answers your touch |
| **Page header** | Marble with a gold inner hairline and a lattice band visible only up close; app icon; bilingual title |
| **Divider** | Lapis small caps, a gold star at the start of a hairline that fades away |
| **List group** | Alabaster card with stone-joint separators |
| **Toggle** | webOS ON/OFF pill: turquoise glaze when on, limestone when off, a marble knob with a gold rim |
| **Buttons** | Primary: lapis glaze. Affirm: cypress glaze. Quiet: limestone |
| **Cards (Itlala)** | Float above a courtyard pool and reflect in it, under a sky that follows the light |
| **The Wave** | A band of jali glass with a gold top edge that follows the finger and ripples |
| **Notifications** | Lapis night bar at the bottom edge with a gold hairline |

---

## 8. Motion

- **Springs, not curves.** Everything settles like a physical object (iOS heritage).
- **Procession.** Moving between layers feels like walking through a colonnade: near layers move more than far ones.
- **Thresholds.** Entering an app scales the card up through the frame; leaving reverses it.
- **Ripple.** The Wave's lift falls off smoothly with distance from the finger.

---

## 9. Accessibility

- Ornament never carries meaning and never reduces text contrast.
- "Reduce transparency" makes glass opaque; "Reduce motion" replaces springs with fades.
- Every light, Isha included, meets WCAG AA for text.

---

<sub>This document is licensed under CC BY 4.0.</sub>
