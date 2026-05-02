# Toolpaths

## -- IN DEVELOPMENT -- closed beta release--

Toolpaths is a Grasshopper plugin for generating and simulating G-code. Its goal is to enable new ways of 3D printing and CNC milling while giving novices and experts alike full control of the machines movement.

## Toolpaths core features

- **Object-Oriented Toolpaths**  
The core data type is the Toolpath, which encapsulates a curve with its associated metadata (speed, extrusion, etc.) into a single object.  
*Granularity*: Assign parameters per-path or per-segment.  
*Compatibility*: A Toolpath object remains a standard Grasshopper geometry type, allowing you to use native components for transformations without losing metadata.
- **Inheritance & Settings**  
Settings follow a simple priority: **Global Defaults** (lowest) → **Linked Template** → **Local Override** (highest). You can use any existing toolpath as a template for a new one, inheriting all properties automatically and overriding only what is necessary.
- **Simulation**  
The FDM engine simulates material deposition rather than just visualizing a mesh pipe. By calculating volume buildup the solver enables features like automatic flow adjustment.

#### Features FDM

- Variable layer height and vase mode slicing.
- Automatic extrusion width calculation and volume-based extrusion modes.
- Generator Components for infill and walls 
- Modulators: Per-vertex control of extrusion parameters like flow, speed, etc
- Masking: Filters to isolate modulator effects to specific sections of a toolpath.
- Path optimization via TSP sorting 
- Real-time playback of the simulation
- UV-mapped output meshes for rendering 
- Z-hop and retractions
- Gcode upload for Klipper, RepRap, and Octoprint.

#### Features CNC (currently not publicly available)

- LinuxCNC and Fusion 360 tool library support with tool/holder visualization
- High-performance stock removal simulation
- LinuxCNC-flavor G-code compiler

### Installation + Licensing

- run `_PackageManager`  > check include Pre-Releases > search for TOOLPATHS > install
- choose trail or cloud key > paste your key  > activate it

For more details, please refer to the [Licensing Documentation](Docs/CORE/licensing.md).

## Quickstart

![Vo9zaG3C3o](Images/Vo9zaG3C3o-2.png)

- **FDM Toolpath:** The central component which defines the toolpath for the printer. Right-Click to reveal properties that can be defined on a per-object level.
- **FDM Extruder:** set extruder number, nozzle diameter, filament diameter and preview color
- **FDM Machine:** bundles all setting for the individual printer. 
- **FDM Defaults:** enables global default values for all toolpaths that are not set on a per-object level.  Right-Click to reveal properties 
- **FDM Processor:** combines all toolpaths and settings into one program that is past to simulation or gcode output
- **FDM Simulator:** creates a mesh preview
- **FDM G-Code Output:** compiles the final G-Code and uploads it to the printer

#### Overview UI


|                                   |                                                                                                                               |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| ![](Images/Rhino_c4XbruyvUU.avif) | right click components to reveal parameters.                                                                                  |
| ![](Images/Rhino_1zHBruW1qc.avif) | Toolpaths objects are geometry. you can transform them with the standart grasshopper components. (move, array, transform ...) |
| ![](Images/Rhino_IU8lO9lQS6.avif) | use **Toolpaths Modulators** to varie different parameters per segment like speed, flow, displacement                         |
| ![](Images/Rhino_e2WsY8M4wt.avif) | combine Modulators with **Masks** to restrict the effect to specific regions or along a gradient                              |


## Extrusion Modes

TOOLPATHS has 4 extrusion modes which are different ways to define the amount of extruded material per mm linear movement. 

![LYrHOhfWVO](Images/LYrHOhfWVO-2.png)

1. **Volume Mode:** This is the most direct way to control the extrusion. It defines the volume extruded per mm of linear movement. e.g., 3 mm³/1 mm  meaning 3 cubic millimeter extruded per one mm traveled. As layer height is actually the distance from the nozzle to the next layer it is not explicitly defined in this mode -- the FDM Simulator then used the volume and actual distance to the layer below to create a accurate preview.
2. **Static Mode:** Sometimes the simulation of the extruded material is too heavy on large models and slows down the workflow. Static Mode disregards the distance to the next layer and  uses explicitly defined width and height values. This allows for extrusions that occupy the same space when in reality the extrusion would actually squish.![XsDMSWZAtk-2](Images/XsDMSWZAtk-2-4.png)
3. **Auto Width Mode:** Automatically calculates the extrusion amount based on the height below the nozzle. Specify a target width, and the system computes the required extrusion volume to achieve it. This mode is particularly useful for non-planar printing applications where the layer height varies continuously. See Extrusion Calculation below for more details.
4. **Auto Ratio Mode:** Similarly to Auto Width Mode, it defines a target ratio between width and height and adjusts the extusion amount arcordingly.

![lcAq2X4jid](Images/lcAq2X4jid-2.png)

**Flow:**  Flow acts like a multiplier on top of the chosen extrusion mode. In combination with e.g. Auto Width mode TOOLPATHS will calculate first the extrusion amount needed for the target width and then multiply it with the supplied flow value. Flow can be modulated with the Flow Modulator.

### Extrusion Calculation

TOOLPATHS simulates all extrusions in a global heightfield. The heightfield, extrusion calculation and preview mesh are tightly related.

![fAgLSqPQ64](Images/fAgLSqPQ64-2.png)  

 Settings for the heightfield are exposed in FDM Defaults:

![0N4zjvTrfB](Images/0N4zjvTrfB-2.png)

- Heightfield Resolution: HFRes defines the resolution of the heightfield. Details smaller than this can not be captured
- Meshsing Resolution: during simulation the toolpath is resampled based on this distance and at every point the heightfield is sampled. In Auto Width Mode the extrusion amount is calculated at every sample point
- Smoothing Window: The preview mesh is slightly smoothed by default, as extrusion can not change instantly. Affects preview only.

#### Auto Extrusion and Degenerate Extrusion Detection

Auto Width Mode is convenient, but it can produce unintended results.

Example: the extrusion width is set to Auto Width 2 mm and the toolpath bridges over a gap. The algorithm evaluates the available height, which might be large (for example 10 cm if the bridge occurs higher in the print). Based on this, it attempts to deposit enough material so the extrusion approaches the target 2 mm width. This can lead to excessive material being extruded. Degenerate Extrusion Detection and layer-height limits are used to handle these cases by capping the amount of material that can be extruded.

#### Degenerate Behavior

A degenerate extrusion is an extrusion that has zero height or zero width. This can happen as toolpath is too close to existing printed geometry.  Degenerate modes handle zero-thickness points by either calculating replacement values from neighboring samples to maintain continuity  (0) or flagging them as suppressed to omit them from the simulation (1).

##### Degenerate Aspect Ratio:

Extrusions with a width / height aspect ratio larger than this are considered degenerate.

##### Layerheight Minimum:

Extrusion with a layerheight smaller than this are considered degenerate.

##### Layerheight Maximum:

Extrusion with a layerheight larger than this are capped at this layerheight.

## Modulators

![wNPmRyUg24](Images/wNPmRyUg24.png)  
Modulators change a Toolpath after it has been created. They are used to vary parameters along a path or  reshape the path geometry. A modulator takes a Toolpath as input and outputs a new Toolpath with the modulation applied. Most modulators work per segment or per vertex. For example, the Flow Modulator writes a flow multiplier for every segment, while displacement modulators move the vertices of the Toolpath. 

Typical uses include:

- varying extrusion flow along a path
- changing print speed per segment
- changing extruder temperature along a path
- displacing a path with vectors, normals, or an interpolated vector field

Numeric modulators such as Flow, Speed, and Extruder Temperature use a shared mapping system. A single value can be applied to the whole Toolpath, or a list of values can be mapped onto the path using a Vertex Mapping Strategy.

![veoBqzWTWN](Images/veoBqzWTWN-2.png)

**Vertex Mapping Strategies:**

- **Constant**: use the first value everywhere
- **OneToOne**: one value for every vertex
- **Wrap**: repeat the value list along the path
- **RepeatLast**: use the final value after the list runs out
- **Normalized-Stepped**: distribute values along the normalized length of the path in steps
- **Normalized-Interpolated**: interpolate smoothly between values along the normalized length of the path

## Masks

![MJQBQnKHZt](Images/MJQBQnKHZt-3.png)

Masks control the strength of a modulator along a Toolpath. They are lists of numeric values, usually mapped per segment or per vertex.

Common values:

- 0 = no effect
- 1 = full effect
- 0..1 = blended effect

Some modulators clamp masks to 0..1. Others use the mask as a direct multiplier, so values above 1 can amplify the effect and negative values can invert it. For predictable results, use 0..1 unless overdriving is intentional.

Masks do not modify Toolpaths by themselves. They are connected to modulators to restrict, fade, or scale effects, for example by region or along a gradient.

## Beta Testing

TOOLPATHS is currently in closed beta. We are beta testing with a small team of dedicated designers and fabricators. If you want to contribute, ask for a key at [toolpaths@juengerkuehn.com](mailto:toolpaths@juengerkuehn.com).

#### Changelog

###### 0.2.16-beta16

- multi extruder support
- when multiple files are open, only the preview of the current file is displayed
- less verbose messages on different compontents and on startup
- bug fixes for simulation time

###### 0.2.15-beta15

- rewrite for vase mode: 4 modes to control pitch and point spacing by angle or distance. removes the 400,001 vertex limit.
- Non-planar slicing: mesh is transformed to planar, sliced, and inverse-transformed to original geometry.
- Upgraded to .NET 8.0: requires Rhino 8.20+ for improved performance.
- Fixed bug causing incorrect infill line heights.

###### 0.2.13-beta13

- package layout for Rhino 7

###### 0.2.12-beta12

- async solver for FMD processor and FDM simulator
- rhino 7 support
- added gyroid and gyroid connected infill
- BREAKING CHANGES:
  - infill generator: "Line width" renamed to "Infill spacing"

###### 0.2.11-beta11

- handles flow = 0 and generates endcaps dynamically at extrusions ends
- more aggressive smoothing, suggested value for mesh smoothing is 0 - 2

###### 0.2.10-beta10

- output for robots (for use with e.g. robots plugin by visose)
- operations are removed and replaced by toolpath inheritance
- toolpath can accept other toolpaths as templates. In this way settings can be used in multiple toolpaths and  adjusted in bulk 
- new curve / toolpath sorting component, sorting is removed in the FDM Processor
- machine and process settings are seperated: FDM machine + FDM Defaults
- FDM Processor checks the build volume, if provided, and gives visual warnings if exceeded
- Z clearance setting check inital Z hop to prevent collisions

###### 0.2.9-beta9

- non-planar example
- Renamed "Initial Z Height" to Safe Clearance: Max(CurrentZ + Clearance, Clearance) logic
- renaming in fdm defaults: StartG → startG-Code, EndG → endG-Code, EPos → ExtruderMode
- fdm simulator: outputs overall program time in human readable format: HH:mm:ss
- vector field modulator: replaced IDW with Gaussian for smoother   displacement
- vector field modulator: introduced per-point "Sigma" radius for individual influence control (removed redundant Falloff)

###### 0.2.1-beta1 to 0.2.8-beta8

- bug fixes
- auto segmentation for curve inputs
- modulated speeds for no extrude moves will be correctly displayed
- gcode output to GH only on request
- faster slicing 
- image map example 
- variable infill example
- better degen defaults
- issue warnings if extusion is limited by nozzle size
- slicing component now accept mesh input (much faster)
- baked preview mesh is now split to match the sim time
- improved stability 
- deconstuct toolpath is now two components: deconstuct CNC and deconstruct FDM
- gha loading sequence fix on mac 
- no extrude curves displayed thicker

###### 0.2.0-beta0

- mesh smoothing 
- refactored infill and wall generator
- modulators accepts linear curves
- demo files
- volume component to calculate the extrusion area based on width / height
- static mode more performant

###### 0.1.14-alpha15

- fdm machine flattens toolpath input
- uv scaling input  for better textures flow along the extrusion
- bugfixes for preview

###### 0.1.13-alpha14

- Infill Generator : robust handling for disjoint regions
- Infill Generator :  Start Point is now hidden ; right click to reveal
- smart selector for extrusion mode based on available inputs 
- bugfix: static mode now correctly ignores sampled heights
- closed paths are rendered more nicely

###### 0.1.12-alpha13

- toolpaths now has an icon

###### 0.1.11-alpha12

- interpolated vector field modulator
- simulation improvement:
  - heigthfield outlier filtering
  - extrusion smoothing 
  - dengenerate extrusion filtering
  - heightfield interpolation
- bugfix: sorting curves off by default
- icons for parameters
- deconstruct toolpath features hidden outputs for clarity
- color component features hidden inputs
- vms now should default to 2 for all modulators

###### 0.1.10-alpha11

- significant perf improvements in fdm program generation and simulation

###### 0.1.9-alpha10

- better icons
- significant perf improvements in fdm preview

###### 0.1.8-alpha9

- icons
- curve divider respects closed/open state
- walls generator reworked to suppress duplicate control points
- introduction of simplify curve component

###### 0.1.7-alpha8

- planar slicer component: generates planar curves for "normal" printing
- bugfix in walls generator: holes are offest correctly

###### 0.1.6-alpha7

- async upload to printer
- refactor of vasemode layerheight 
- introduction of layerheight generator: creates a layerheights based on slope 
- better default values
- licensing popup when license is expired

###### 0.1.5-alpha6

- option to disable licensing , plugin will not try to load automatically until licensing is enabled
- naming conflict resolved between Rhino host plugin and Grasshopper

###### 0.1.4-alpha5

- licensing popup at first install