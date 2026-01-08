# SPARTA - Fork with Zonal Mapping-Based Localized Ablation Framework

This is a fork of the official SPARTA software package, enhanced with additional capabilities for a **Zonal mapping-based localized ablation framework**.

## Original SPARTA Introduction

SPARTA stands for Stochastic PArallel Rarefied-gas Time-accurate Analyzer.

Copyright (2014) Sandia Corporation. Under the terms of Contract DE-AC04-94AL85000 with Sandia Corporation, the U.S. Government retains certain rights in this software. This software is distributed under the GNU General Public License.

SPARTA is a Direct Simulation Monte Carlo (DSMC) code designed to run efficiently on parallel computers. It was developed at Sandia National Laboratories, a US Department of Energy facility, with funding from the DOE. It is an open-source code, distributed freely under the terms of the GNU Public License (GPL).

The primary authors of the code are Steve Plimpton and Michael Gallis, who can be emailed at sjplimp@gmail.com and magalli@sandia.gov. The SPARTA web site at [http://sparta.sandia.gov](http://sparta.sandia.gov) has more information about the code and its uses.

---

## New Feature in This Fork: Zonal Mapping-Based Localized Ablation Framework

This fork extends the standard `fix ablate` command to implement a **Zonal mapping-based localized ablation framework**. This framework allows users to define different ablation behaviors and automatically assign different surface properties across distinct spatial regions (grid groups) of the simulation domain.

### 1. Overview

The standard `fix ablate` command typically applies a single ablation source (like a compute or variable) to the entire defined region.

The novel **Zonal mapping-based localized ablation framework**, enabled by the new `map` keyword, achieves the following:

1. **Spatially Varying Ablation Sources**: Different grid groups can be assigned different drivers for ablation. For example, Region A could be ablated based on chemical reaction rates, while Region B is driven by physical sputtering.
2. **Automatic Surface Property Assignment**: When `create_isurf` or `read_isurf` is called referencing a mapped `fix ablate`, the generated implicit surface elements (triangles/lines) are automatically assigned specific surface collision models and reaction models based on the grid group they reside in.

### 2. Syntax Extension

The base syntax for `fix ablate` is:

```
fix ID ablate group-ID Nevery scale source [maxrandom] keyword value ...

```

To enable the zonal mapping framework, a new keyword `map` has been added.

#### New Keyword: `map`

**Syntax:**
`map groups N grp1 col1 react1 src1 [arg1] grp2 col2 react2 src2 [arg2] ...`

**Arguments Explanation:**

* **`map groups`**: Literal keyword indicating the start of zone definitions.
* **`N`**: Integer. The total number of zones to define immediately following.
* **Zone Definitions (N sets)**: Each set contains 5 arguments defining one zone:
1. **`grpI`** (Grid Group ID): The grid group defining the spatial extent of zone I. Must be a subset of the main `fix ablate` group-ID.
2. **`colI`** (Collision Model ID): The ID of a pre-defined `surf_collide` model. Surfaces generated in this zone will be assigned this model. Use `none` if not needed.
3. **`reactI`** (Reaction Model ID): The ID of a pre-defined `surf_react` model. Surfaces generated in this zone will be assigned this model. Use `none` if not needed.
4. **`srcI`** (Source Type): The specific ablation source for this zone. Can be:
* `c_ID` / `c_ID[n]` (Compute)
* `f_ID` / `f_ID[n]` (Fix)
* `v_name` (Grid-style variable)
* `uniform`
* `random`


5. **`[argI]`** (Optional Argument):
* If `srcI` is `uniform` or `random`, this must be an integer `maxrandom`.
* For other source types, this argument is omitted.





### 3. How it Works

1. **Ablation Step**: Every `Nevery` steps, the fix iterates through grid cells in the main `group-ID`. If a cell belongs to a defined `map` zone (`grpI`), its ablation decrement is calculated using that zone's specific source (`srcI`) multiplied by the global `scale`. If it doesn't belong to any mapped zone, the default `source` (defined before the `map` keyword) is used.
2. **Surface Generation Step**: When `create_isurf` or `read_isurf` is used with this fix ID, the code checks which `map` zone group the newly generated surfaces fall into and automatically assigns the corresponding `colI` and `reactI` to them.

### 4. Example Usage

The following example demonstrates a setup utilizing the zonal mapping-based localized ablation framework. It defines a default ablation behavior (zero ablation) and three distinct zones with specialized properties and ablation sources.

**Prerequisites (assumed defined in input script):**

* Grid groups: `gMRR` (parent), `gWRtl` (zone 1), `gWRtr` (zone 2), `gMS` (zone 3).
* Surface models: Collision IDs 1 & 2; Reaction ID 1.

**Commands:**

```sparta
# Define collision and reaction models
surf_collide    1 diffuse 2670 1
surf_collide    2 specular
surf_react      1 prob airhh.sr

# Define a compute to act as an ablation source
compute 10 react/isurf/grid all 1

# Define fix ablate using the 'map' keyword for zoning
# - Parent group: gMRR
# - Default source: uniform 0 (no ablation outside mapped zones)
fix fab ablate gMRR 1 0.002 uniform 0 &
      map groups 3 &
      gWRtl 2 none uniform 0 &    # Zone 1: Coll mod 2, No react, Src: uniform 0
      gWRtr 1 none uniform 0 &    # Zone 2: Coll mod 1, No react, Src: uniform 0
      gMS   1 1    c_10[2] &      # Zone 3: Coll mod 1, React mod 1, Src: compute 10, index 2
      multiple yes

# Create implicit surfaces based on the fix definition
# Surface properties will be assigned automatically based on the map above.
create_isurf gMRR fab 127.5 ave

```

---

## Standard SPARTA Distribution Info

The SPARTA distribution includes the following files and directories:

* `README`: this file
* `LICENSE`: the GNU General Public License (GPL)
* `bench`: benchmark problems
* `data`: files with species/reaction params, surface files
* `doc`: documentation (Note: Official docs do not reflect the new **Zonal mapping-based localized ablation framework** feature)
* `examples`: simple test problems
* `lib`: additional library files
* `python`: Python wrapper on SPARTA as a library
* `src`: source files
* `tools`: pre- and post-processing tools

Point your browser at any of these files to get started with standard SPARTA features:

* `doc/Manual.html`: the SPARTA manual
* `doc/Section_intro.html`: hi-level introduction to SPARTA
* `doc/Section_start.html`: how to build and use SPARTA
* `doc/Developer.pdf`: SPARTA developer guide
