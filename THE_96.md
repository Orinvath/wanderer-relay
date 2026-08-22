# The 96 — every world-change the Avatar can want

Directive 198, data retrieval only. **Read verbatim out of the running build.** Nothing here was
chosen, changed or proposed.

This is the list `effects.js` builds from the real panel: every control that has a nameable effect,
in each direction it can actually move, with the effect-kind the build assigns it and the words the
Avatar itself would use. It is what the capability system hands the goal-former — and what the
offers model scores at zero, because none of these are among 196's ten named acts.

**96 candidates.**

## How it was produced

The exact call `acceptance-effects.js` makes, so this is the same list the suite counts:

```
candidates(drive(new WorldState(), 'light', -1), midAspects,
           { frame: 'offered', licence: 'takeover', needs: { relatedness: -10, competence: -10, autonomy: -10 } })
```

The world is driven dark, and the takeover licence is used because 082 gates the panel and the
bottom is the honest way to open all of it. A different world or licence yields a different list —
this is the one the suite measures.

## The effect-kinds in use

adding · approaching · brightening · cooling · enclosing · filling silence · making quiet · opening · quickening · stilling · taking away · warming · withdrawing

**13 kinds across 96 candidates.** `effects.js` assigns these; a control whose change has no
name yet produces no candidate at all, which is why the count is not simply every control times two.

## What each candidate carries

`name` · `addr` (the control's address) · `from` and `to` (the values that would actually land,
after the control's own rounding and end-stops) · `effect` (the axis vector) · `words` (the
effect-kinds, in order) · and `promises`, which 197 declared dead.

---

## Worlds  (1)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| taking away | taking away | Stage Visible | turning off | on | off | `showStage` |

## Painted Sky  (1)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| adding | adding + opening | Painted Sky | turning on | off | on | `paintedSky.on` |

## Planes  (9)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| taking away | taking away | Plane | turning off | on | off | `plane:far.on` |
| taking away | taking away | Plane | turning off | on | off | `plane:mid.on` |
| taking away | taking away | Plane | turning off | on | off | `plane:near.on` |
| opening | opening | Depth | up | 18 | 33 | `plane:far.depthStrength` |
| enclosing | enclosing | Depth | down | 18 | 3.000 | `plane:far.depthStrength` |
| opening | opening | Depth | up | 18 | 33 | `plane:mid.depthStrength` |
| enclosing | enclosing | Depth | down | 18 | 3.000 | `plane:mid.depthStrength` |
| opening | opening | Depth | up | 18 | 33 | `plane:near.depthStrength` |
| enclosing | enclosing | Depth | down | 18 | 3.000 | `plane:near.depthStrength` |

## Music Score  (3)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| making quiet | making quiet | Music Score | turning off | on | off | `musicOn` |
| filling silence | filling silence | Volume | up | 70 | 95 | `musicVolume` |
| making quiet | making quiet | Volume | down | 70 | 45 | `musicVolume` |

## Space  (4)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| opening | opening | Amount | up | 25 | 50 | `sfxSpace` |
| enclosing | enclosing | Amount | down | 25 | 0 | `sfxSpace` |
| opening | opening | Size | up | 40 | 65 | `sfxSpaceSize` |
| enclosing | enclosing | Size | down | 40 | 15 | `sfxSpaceSize` |

## Wind  (3)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| filling silence | filling silence | Wind | turning on | off | on | `sfxWindOn` |
| filling silence | filling silence | Level | up | 40 | 65 | `sfxWind` |
| making quiet | making quiet | Level | down | 40 | 15 | `sfxWind` |

## Rain  (3)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| filling silence | filling silence | Rain | turning on | off | on | `sfxRainOn` |
| filling silence | filling silence | Level | up | 40 | 65 | `sfxRain` |
| making quiet | making quiet | Level | down | 40 | 15 | `sfxRain` |

## Thunder  (3)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| filling silence | filling silence | Thunder | turning on | off | on | `sfxThunderOn` |
| filling silence | filling silence | Level | up | 45 | 70 | `sfxThunder` |
| making quiet | making quiet | Level | down | 45 | 20 | `sfxThunder` |

## Water  (3)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| filling silence | filling silence | Water | turning on | off | on | `sfxWaterOn` |
| filling silence | filling silence | Level | up | 35 | 60 | `sfxWater` |
| making quiet | making quiet | Level | down | 35 | 10 | `sfxWater` |

## Ocean Waves  (3)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| filling silence | filling silence | Ocean Waves | turning on | off | on | `sfxWavesOn` |
| filling silence | filling silence | Level | up | 40 | 65 | `sfxWaves` |
| making quiet | making quiet | Level | down | 40 | 15 | `sfxWaves` |

## Birds  (3)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| filling silence | filling silence | Birds | turning on | off | on | `sfxBirdsOn` |
| filling silence | filling silence | Level | up | 30 | 55 | `sfxBirds` |
| making quiet | making quiet | Level | down | 30 | 5 | `sfxBirds` |

## Insects  (3)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| filling silence | filling silence | Insects | turning on | off | on | `sfxInsectsOn` |
| filling silence | filling silence | Level | up | 25 | 50 | `sfxInsects` |
| making quiet | making quiet | Level | down | 25 | 0 | `sfxInsects` |

## Fire  (3)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| filling silence | filling silence | Fire | turning on | off | on | `sfxFireOn` |
| filling silence | filling silence | Level | up | 35 | 60 | `sfxFire` |
| making quiet | making quiet | Level | down | 35 | 10 | `sfxFire` |

## Lights  (3)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| brightening | brightening | Intensity | up | 0 | 5 | `light:key.intensity` |
| brightening | brightening | Shadows | turning off | on | off | `light:key.shadows` |
| brightening | brightening | Shadow Softness | up | 0 | 2 | `light:key.shadowSoftness` |

## God Rays  (4)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| brightening | brightening | God Rays | turning on | off | on | `godRaysEnabled` |
| brightening | brightening | Strength | up | 0 | 0.750 | `godRaysStrength` |
| brightening | brightening | Density | up | 0.300 | 0.525 | `godRaysDensity` |
| taking away | taking away + brightening | Dust | down | 1 | 0.750 | `godRaysNoise` |

## Light in the Air  (5)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| brightening | brightening | Light in the Air | turning on | off | on | `volLight` |
| brightening | brightening | Strength | up | 0 | 1 | `volLightStrength` |
| brightening | brightening + enclosing | Density | up | 0 | 0.750 | `volLightDensity` |
| adding | adding | Stone Relief | up | 0.350 | 0.575 | `volLightRelief` |
| taking away | taking away | Stone Relief | down | 0.350 | 0.125 | `volLightRelief` |

## Wisps  (9)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| adding | adding + brightening | Wisps | turning on | off | on | `wispsEnabled` |
| adding | adding | Count | up | 220 | 330 | `wispCount` |
| taking away | taking away | Count | down | 220 | 110 | `wispCount` |
| opening | opening | Spread | up | 9 | 13.500 | `wispSpread` |
| enclosing | enclosing | Spread | down | 9 | 4.500 | `wispSpread` |
| opening | opening | Height | up | 5.500 | 8.250 | `wispHeight` |
| enclosing | enclosing | Height | down | 5.500 | 2.750 | `wispHeight` |
| adding | adding | Size | up | 0.050 | 0.549 | `wispSize` |
| brightening | brightening | Cast Shadows in Shafts | turning off | on | off | `wispsCastShadows` |

## Position  (5)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| withdrawing | withdrawing | Left / Right | up | 0 | 15 | `avatarOffsetX` |
| withdrawing | withdrawing + opening | Up / Down | up | 0 | 15 | `avatarOffsetY` |
| withdrawing | withdrawing | Near / Far | up | 0 | 60 | `avatarOffsetZ` |
| approaching | approaching | Near / Far | down | 0 | -60 | `avatarOffsetZ` |
| adding | adding | Scale | up | 2 | 6.500 | `avatarScale` |

## Colour  (6)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| warming | warming | Tendrils | down | #7fecd4 | #82ec7f | `avatarTendrilColor` |
| warming | warming | Head | down | #7fecd4 | #82ec7f | `avatarHeadColor` |
| warming | warming | Eye — speaking | down | #cc00ff | #ff0099 | `avatarEyeBusyColor` |
| cooling | cooling | Eye — speaking | down | #cc00ff | #5100ff | `avatarEyeBusyColor` |
| brightening | brightening | Eye Glow | up | 0 | 25 | `avatarEyeGlow` |
| brightening | brightening + adding | Eye Glow Size | up | 1 | 2.250 | `avatarEyeGlowSize` |

## Color Effects  (11)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| brightening | brightening | Glow | up | 0 | 25 | `avatarAdaptGlow` |
| adding | adding | Vividness | up | 45 | 70 | `avatarAdaptVividness` |
| taking away | taking away | Vividness | down | 45 | 20 | `avatarAdaptVividness` |
| adding | adding | Rainbow | turning on | off | on | `avatarRainbow` |
| quickening | quickening | Rainbow Speed | up | 0.150 | 0.398 | `avatarRainbowSpeed` |
| stilling | stilling | Rainbow Speed | down | 0.150 | 0.010 | `avatarRainbowSpeed` |
| adding | adding | Pulse | turning on | off | on | `avatarPulse` |
| quickening | quickening | Pulse Speed | up | 0.500 | 1.238 | `avatarPulseSpeed` |
| stilling | stilling | Pulse Speed | down | 0.500 | 0.050 | `avatarPulseSpeed` |
| adding | adding | Pulse Depth | up | 35 | 60 | `avatarPulseDepth` |
| taking away | taking away | Pulse Depth | down | 35 | 10 | `avatarPulseDepth` |

## In the World  (4)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| brightening | brightening | Blocks Light Shafts | turning off | on | off | `avatarBlocksShafts` |
| brightening | brightening | Casts Light | turning on | off | on | `avatarCastsLight` |
| brightening | brightening | Light Strength | up | 0 | 50 | `avatarLightStrength` |
| brightening | brightening + opening | Light Reach | up | 5 | 26 | `avatarLightReach` |

## Physics  (4)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| adding | adding | Tendril Flow | up | 0.500 | 1 | `avatarFlowAmp` |
| taking away | taking away | Tendril Flow | down | 0.500 | 0 | `avatarFlowAmp` |
| quickening | quickening | Flow Speed | up | 0.700 | 1.175 | `avatarFlowSpeed` |
| stilling | stilling | Flow Speed | down | 0.700 | 0.225 | `avatarFlowSpeed` |

## Adapting  (3)

| effect-kind | as the Avatar sees it | control | direction | from | to | address |
|---|---|---|---|---|---|---|
| withdrawing | withdrawing | Adapt | turning on | off | on | `avatarAdapt` |
| approaching | approaching | Invert | turning on | off | on | `avatarAdaptInvert` |
| approaching | approaching | Strength | down | 100 | 75 | `avatarAdaptStrength` |

