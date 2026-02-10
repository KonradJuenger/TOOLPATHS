# Toolpaths

## -- IN DEVELOPMENT -- closed beta release--

Toolpaths is a Grasshopper plugin for generating and simulating G-code. It's goal is to enable new ways of 3D printing and CNC milling while giving novices and experts alike full control of the machines movement.

### Toolpaths core features

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
- Modulators: Per-vertex control of extrusion prarmeters like flow, speed, etc
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

#### Installation

For installation and licensing instructions, please refer to the [Licensing Documentation](Docs/CORE/licensing.md).

#### Beta Testing

TOOLPATHS is currently in closed beta. We are beta testing with a small team of dedicated designers and fabricators. If you want to contribute, ask for a key at toolpaths@juengerkuehn.com.

#### overview ui

|                                                                                     |                                                                                                                               |
| ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| <img src="Images/Rhino_c4XbruyvUU.avif" alt="" style="max-width:100%;height:auto;"> | right click components to reveal parameters.                                                                                  |
| <img src="Images/Rhino_1zHBruW1qc.avif" alt="" style="max-width:100%;height:auto;"> | Toolpaths objects are geometry. you can transform them with the standart grasshopper components. (move, array, transform ...) |
| <img src="Images/Rhino_6G4t7lFxec.avif" alt="" style="max-width:100%;height:auto;"> | use **Toolpaths Generators** to create vasemode prints or infill                                                              |
| <img src="Images/Rhino_IU8lO9lQS6.avif" alt="" style="max-width:100%;height:auto;"> | use **Toolpaths Modulators** to varie different parameters per segment like speed, flow, displacement                         |
| <img src="Images/Rhino_e2WsY8M4wt.avif" alt="" style="max-width:100%;height:auto;"> | combine Modulators with **Masks** to restrict the effect to specific regions or along a gradient                              |

#### Changelog

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