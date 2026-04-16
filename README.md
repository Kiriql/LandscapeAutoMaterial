# LandscapeAutoMaterial

A procedural landscape material and vegetation system for **Unreal Engine 5.7**. Automatically applies terrain textures based on slope, elevation, and snow coverage — no manual painting required.

---

## Overview

LandscapeAutoMaterial solves the tedium of manually painting terrain layers on large landscapes. The master material reads geometry properties (height, slope angle, normals) at runtime and blends the appropriate surface layers automatically. Drop it on a landscape, configure the texture parameters, and the terrain textures itself.

---

## Demo


---

## Features

- **Height-based blending** — different materials at different elevations (grass → rock → snow)
- **Slope detection** — vertical cliff faces automatically switch to triplanar-projected rock
- **Snow mask** — procedural snow accumulation on near-horizontal surfaces above a threshold
- **Triplanar mapping** — eliminates texture stretching on steep terrain
- **Distance-based tiling & variation** — detail at close range, macro variation at distance; no obvious tiling
- **Cell bombing** — breaks up texture repetition patterns
- **Runtime Virtual Texture (RVT) integration** — efficient blending of landscape with placed meshes
- **Automatic foliage placement** — grass, bushes, and ground cover placed per material layer (32 plant variations total)
- **Ambient occlusion** — baked into material via distance fields

---

## Material Architecture

```
MM_LandscapeAutoMaterial   ← Master Material
└── MI_LandscapeAutoMaterial  ← Material Instance (tweak parameters here)

Material Layers
├── ML_Layer01  — base terrain (grass / soil)
├── ML_Layer02  — secondary terrain
├── ML_Layer03  — rock / exposed stone
├── ML_Layer04  — cliff faces
├── ML_LayerSlopes  — auto-applied to steep surfaces
└── ML_LayerSnow    — auto-applied at high elevations

Material Functions (19)
├── MF_SlopeMask / MF_Slopes       — slope detection & processing
├── MF_SnowMask                    — elevation + angle snow logic
├── MF_HeightBlendAttribute        — height-driven layer transitions
├── MF_Triplanar                   — triplanar UV projection
├── MF_DistanceBasedTiling         — near/far tiling switch
├── MF_DistanceFade / MF_DistanceVariation
├── MF_CellBombing                 — repetition breaking
├── MF_ColorVariation              — per-instance color variation
├── MF_AmbientOcclusion
├── MF_LayersBlending              — multi-layer compositor
├── MF_RVTBlend / M_RVTBlend       — Runtime Virtual Texture blending
├── MF_AutoFoliages                — foliage placement per layer
└── MF_Normal / MF_Roughness / MF_Specular / MF_TextureObject
```

---

## Foliage System

Vegetation is placed automatically based on the active material layer. Each layer has its own foliage type configuration.

| Category | Variations | Placement |
|----------|-----------|-----------|
| Grass | 8 meshes | Layers 01–02 |
| Bushes | 7 meshes | Layers 01–03 |
| Ground Cover | 17 meshes | All layers |

`BP_GlobalFoliageActor` orchestrates all foliage across the landscape.

---

## Textures

- **Quixel Megascans** — 5 terrain layers × 5 PBR maps each (`_B`, `_H`, `_N`, `_ORD`, `_ORM`) + snow layer
- **Placeholder textures** — included for prototyping without Megascans
- **RVT targets** — `RVT_Base`, `RVT_Height` for mesh-landscape blending
- **Foliage atlases** — grass, bush, ground cover texture sets

---

## Rendering Stack

Configured for maximum quality with UE5's modern pipeline:

| Feature | Setting |
|---------|---------|
| Global Illumination | Lumen |
| Reflections | Lumen |
| Shadows | Virtual Shadow Maps |
| Materials | Substrate |
| Ray Tracing | Enabled |
| Shader Model | SM6 (DX12) |
| Mesh Distance Fields | Enabled |

---

## Project Structure

```
LandscapeAutoMaterial-main/
├── LSAutoMaterial.uproject
├── Config/
│   ├── DefaultEngine.ini     # Rendering settings
│   ├── DefaultGame.ini
│   └── DefaultInput.ini
└── Content/
    └── AutoMaterial/
        ├── MM_LandscapeAutoMaterial.uasset   # Master Material
        ├── MI_LandscapeAutoMaterial.uasset   # Material Instance
        ├── M_RVTBlend.uasset
        ├── Functions/          # 19 material functions
        ├── MaterialLayers/     # 6 terrain layers
        ├── LayerFoliageTypes/  # Foliage configs per layer
        ├── Textures/
        │   ├── Placeholders/   # Prototyping textures
        │   ├── QuixelTextures/ # Megascans PBR sets
        │   └── RVT/            # Runtime Virtual Textures
        └── Assets/
            ├── 3DAssets/       # Beach rock, forest rock meshes
            └── Plants/         # Grass, bush, ground cover meshes & foliage types
```

---

## Getting Started

### Requirements
- Unreal Engine 5.7
- DX12-capable GPU (Shader Model 6)
- Quixel Bridge (optional — placeholder textures are included for prototyping)

### Setup
1. Open `LSAutoMaterial.uproject` in Unreal Engine 5.7
2. Open `Content/ExampleLevel` to see the system in action
3. To use in your own project:
   - Migrate `Content/AutoMaterial/` to your project via **Content Browser → Migrate**
   - Apply `MI_LandscapeAutoMaterial` to your Landscape actor
   - Configure texture parameters in the Material Instance to your own PBR sets

### Replacing Textures
Each layer exposes texture parameters in `MI_LandscapeAutoMaterial`. Swap the Quixel textures for your own — the blending logic is texture-agnostic.
