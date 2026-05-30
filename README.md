# Makebsp

Makebsp is a high-performance idTech 3 BSP compiler modernization based on the original id Software `q3map` source code. This toolchain is designed for cinema-grade lighting, high-precision ray tracing, and deep integration of 3D models as first-class world geometry.

## Table of Contents
- [💡 Key Features](#-key-features)
    - [1. High-Performance Ray Tracing (Intel Embree)](#1-high-performance-ray-tracing-intel-embree)
    - [2. 32-bit Floating Point Pipeline](#2-32-bit-floating-point-pipeline)
    - [3. First-Class Model Support (misc_model)](#3-first-class-model-support-misc_model)
    - [4. Brush-to-Light Generation (func_light)](#4-brush-to-light-generation-func_light)
    - [5. In-Editor Configurable Surface Colors](#5-in-editor-configurable-surface-colors)
    - [6. Automatic Light Halos](#6-automatic-light-halos)
    - [7. Modernized Color & Lighting Models](#7-modernized-color--lighting-models)
    - [8. Macro Ambient Occlusion (MAO)](#8-macro-ambient-occlusion-mao)
    - [9. High-Quality 2D Filtering](#9-high-quality-2d-filtering)
    - [10. Deluxe Mapping V2](#10-deluxe-mapping-v2)
    - [11. Customizable Game Profiles (JSON)](#11-customizable-game-profiles-json)
- [🛠 Recommended Workflow](#-recommended-workflow)
- [🎨 Shader Modifications](#-shader-modifications)
- [📦 Entity Keys Reference](#-entity-keys-reference)
- [💻 CLI Command Reference](#-cli-command-reference)

---

## 💡 Key Features

### 1. High-Performance Ray Tracing (Intel Embree)
The legacy BSP-traversal ray caster has been replaced with the industry-standard **Intel Embree 4.4.0** BVH builder and intersection kernels.
- **Selective Shadowing:** Alpha-tested surfaces (foliage, grates) now cast accurate, per-pixel shadows via a custom `AlphaFilter` integration.
- **Unified Data Cache:** The toolchain pre-calculates and caches world-space origins and normals for all texels and voxels, sharing them across all lighting stages to eliminate redundant reconstruction.
- **Intelligent Light Culling:** An advanced reach-based culling system ignores surfaces outside a light's influence radius with zero overhead.
- **Performance:** Combined with Intel's optimized kernels, this results in a **10x-20x performance gain** over traditional tools.

### 2. 32-bit Floating Point Pipeline
Internally, the toolchain operates entirely in a **high-precision 32-bit floating point** space.
- **Internal Buffers:** All lighting data is accumulated in float32 buffers to prevent rounding artifacts and light clipping.
- **Dynamic Normalization:** 8-bit output conversion is deferred to the final stage, allowing for superior tonemapping and highlight compression.

### 3. First-Class Model Support (misc_model)
`misc_model` entities are fully integrated into the world geometry.
- **Automatic Collision (HACD):** Every model is solid by default. Optimized convex collision hulls are automatically generated using the **HACD** algorithm.
- **Omnidirectional Lightmapping:** Triangle soups receive high-quality, seamless lightmaps via a specialized UV-to-world rasterizer.
- **Seamless Integration:** All models are solid and lightmapped by default, behaving exactly like world brushes.

### 4. Brush-to-Light Generation (func_light)
The `func_light` entity creates complex light setups directly from brush geometry.
- **Instantly Emissive:** Turn any brush surface into an emissive light source or spotlight without shader scripting.
- **Direct Control:** Control intensity, color, and attenuation directly via entity keys.

### 5. In-Editor Configurable Surface Colors
Tint and recolor materials directly within the map editor.
- **vertexcolor Overrides:** Use the `vertexcolor` key on `misc_model` or `func_group` to tint surfaces without authoring duplicate shader files.
- **Rapid Iteration:** Reuse assets (like cars or crates) with unique colors per instance exclusively via entity keys.

### 6. Automatic Light Halos
Spotlights can automatically generate volumetric "halos" (billboards) to simulate atmospheric scattering.
- **Dynamic Sizing:** Halo dimensions match the light's intensity and radius.
- **Customizable:** Override effects via the `haloshader` key or disable them for specific lights.

### 7. Modernized Color & Lighting Models
- **Global Color Pipeline:** Unified support for RGB (0..1), integer RGB (0..255), and Hex codes (`#RRGGBB`).
- **Artistic Integrity:** No automatic normalization; light colors are used exactly as intended by the mapper.
- **Advanced Kernels:** Support for Inverse-Square, Unreal, Smoothstep attenuation, and Half-Lambert kernels.

### 8. Macro Ambient Occlusion (MAO)
A new volumetric AO pass computes spatial "openness" for depth.
- **Atmospheric Depth:** Large-scale features (caves, plazas) naturally darken/brighten based on enclosure.
- **Bent Normals:** Ambient energy includes directional hints, ensuring normal maps respond correctly to openings.

### 9. High-Quality 2D Filtering
Rebuilt post-processing for seamlessness.
- **Stitch Filtering:** Automatically matches adjacent world-space edges to eliminate seams.
- **Volumetric Filtering:** Ensures smooth gradients on complex models with disconnected UVs.
- **Per-Surface Control:** Use the `smooth` key to override global smoothing per-entity.
- **GPU Accelerated:** Fully OpenCL-accelerated multi-pass smoothing.

### 10. Deluxe Mapping V2
An improved, irradiance-preserving directional lighting system.
- **Iterative Accumulation:** Prevents "rim bloom" artifacts and ensures stable highlights in complex lighting scenarios.

### 11. Customizable Game Profiles (JSON)
All engine settings are externalized into JSON profiles.
- **Adaptability:** Easily define new profiles for different engines with custom BSP versions and limits.
- **Global Defaults:** Tune all lighting and filesystem parameters per-project without recompiling.

---

## 🛠 Recommended Workflow

Follow this configuration hierarchy for the best results:

1.  **Game Profile (Global Defaults):** Define the baseline "look" and engine limits in the `games/*.json` files.
2.  **Worldspawn (Map-Specific):** Use Worldspawn keys (e.g., `samplesize`, `ambient`, `shading`) for map-level overrides.
3.  **CLI Arguments (Build Modifiers):** Use command-line switches to toggle between fast dev builds (`-fast`) and high-quality final bakes (`-supersample 1.0`).

---

## 🎨 Shader Modifications

### New Shader Directives
- `q3map_vertexcolor <R G B>`: Overrides vertex color.
- `q3map_surfacelight_glow <value>`: Sets backface glow (default on lava/slime).
- `q3map_lightColor <R G B>`: Alias for light emission color.

### Color Handling
The new pipeline autodetects formats (Float, Byte, Hex) and **prevents normalization**, preserving original artistic intent.

### Stage Directives
- `material <image>`: QFusion fallback for deriving surface/light colors.

### Surface Parameters
- `nosolid`: Alias for `nonsolid`.
- `nowalljump`: Disables QFusion wall jumping.

---

## 📦 Entity Keys Reference

### Worldspawn
| Key | Description |
| :--- | :--- |
| `ambient` | Global ambient intensity. |
| `ambient_sky` / `ambient_ground` | Hemispherical ambient colors. |
| `shading` | Global model (halflambert, lambert, unreal, etc). |
| `attenuation` | Falloff model (standard, soft, unreal, etc). |
| `exposurefilter` | Tonemapping filter (softknee, reinhard, filmic). |
| `supersample` | Global trace radius (e.g., 1.0). |

### misc_model / func_group
| Key | Description |
| :--- | :--- |
| `vertexcolor` | Tints all surfaces of the entity. |
| `smooth` | Local smoothing radius override. |
| `upscale` | Enable 2x lightmap resolution. |
| `collisiontype` | HACD type (object, wrap, walkable, none). |

### light / func_light
| Key | Description |
| :--- | :--- |
| `type` | light type (point, spot, surface). |
| `haloshader` | Override volumetric halo. |
| `backsplash` | Percentage of light reflected back. |

---

## 💻 CLI Command Reference

### Makebsp
- `-samplesize <N>`: Default lightmap resolution.
- `-enforceSampleSize <0\|1>`: Strictly follow shader resolutions.
- `-guessuvs`: Optimal UV packing for models.
- `-vis`: Visibility mode (`-fast`, `-merge`).
- `-exportmodels <bsp>`: Export models to `.obj`.

### Light
- `-deluxe <0\|1>`: Directional lightmaps.
- `-mao_samples <N>`: Macro ambient occlusion quality.
- `-rad_passes <N>`: Number of light bounces.
- `-antialiasing <N>`: Post-process AA passes.
- `-opencl <0\|1>`: GPU acceleration.
- `-lowmem`: Memory-mapped mode for massive maps.
