---
title: "Stylized Ocean Waves Shader (Mobile)"
date: 2026-03-04
---
# Overview

Over the last few days, I’ve been experimenting with a stylized ocean water shader for a mobile game, Amikin Village. I’m happy with the wave animation I achieved and think it has good potential (still WIP), so I wanted to share the result and the core idea behind the approach.

In-game view:
TODO: img

Top view:
TODO: img

- Waves start at the edge of the map and move toward the center.
- Waves are dampened when they hit obstacles (rocks, terrain, etc.).
- Wave height increases closer to the coast.

# How It Works

The shader uses the following data, baked in Houdini into the water surface mesh:

- Path distance - for wave phase
- Wave mask - for wave damping
- Vertical distance to the bottom - for coastal wave amplification

## Path Distance & Wave Phase

First, I define the region from which waves start - a circular boundary around the island:
TODO: img

Then, I temporarily exclude the water surface beneath the terrain (needed for the next step):
TODO: img

For each vertex, I compute the shortest path distance to the nearest wave start point. For example, this vertex:
TODO: img

This gives clean path distances:
TODO: img

Applying frac() to generate the wave phase. The distance is baked into UV1.x:
TODO: img

Adding a time offset produces the animated wave phase:
TODO: img

## Wave Mask & Damping

Instead of raycasting, I compare the path distance with the direct distance to the wave start point:
- If they are similar, the wave reaches the point without hitting obstacles → white mask
- If the path is longer, it means the wave bypasses geometry → black mask

Then I slightly blur the mask:
TODO: img

This mask will later be used to dampen waves.

## Wave Phase → Wave Form

I shape the wave with a ramp to fine-tune its form, then add small sub-waves between the main peaks. The wave phase is used as input:
TODO: img

Waves are dampened by the wave mask we computed earlier, and their height is scaled by the baked distance to the bottom, increasing amplitude near the shore. All together:
TODO: img

# Foam
I ramp the wave phase, then multiply it by the masks, and use the result as input for smoothstep() with a noise texture. That produces the foam mask. Even without a foam texture, it already looks like foam:
TODO: img

Finally, I use the foam mask to draw a distorted foam texture.

## Additional Features:
On top of that:
- Depth gradient
- Specular highlights
- Caustic
- Refraction
- Other typical water shading components, depending on performance budget
