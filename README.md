Toolpaths is a Grasshopper plugin for generating and simulating G-code. Its goal is to enable new ways of 3D printing and CNC milling while giving novices and experts alike full control of the machines movement.

## Toolpaths core features

- **Object-Oriented Toolpaths**

The core data type is the Toolpath, which encapsulates a curve with its associated metadata (speed, extrusion, etc.) into a single object.  
*Granularity*: Assign parameters per-path or per-segment.  
*Compatibility*: A Toolpath object remains a standard Grasshopper geometry type, allowing you to use native components for transformations without losing metadata.

- **Toolpath Inheritance**

Settings follow a simple priority: **FDM Defaults** (lowest) → **inherited Toolpath settings** → **local Toolpath overrides** (highest). Connect one Toolpath to another to inherit its settings, then override only the values that need to change.

- **Simulation**

The FDM engine simulates material deposition rather than just visualizing a mesh pipe. By calculating volume buildup the solver enables features like automatic flow adjustment.

## Install TOOLPATHS

1. Get a trial key here: [ link ]
2. In Rhino, run `_PackageManager`, enable **Include pre-releases**, search for **TOOLPATHS**, and install it.
3. When the licensing dialog opens, paste your trial key under **License key — local or trial** and click **Activate license**. To use a Rhino account license instead, choose **Rhino account — Cloud Zoo** and click **Continue with Rhino**.

See the [licensing guide](deprecated/Docs/CORE/licensing.md) for other license types and details.

# Quickstart

A **Toolpath** combines geometry with the properties used to print it. The **FDM Processor** collects the Toolpaths, applies defaults and machine settings, and creates one program. The **FDM Simulator** displays that program as a mesh, and **FDM G-Code Output** writes the program as G-code.

![Quickstart TOOLPATHS workflow](docs-v3/images/quickstart.png)

[Download the Quickstart Grasshopper definition](docs-v3/examples/quickstart.gh) to explore the complete workflow.

### Vase Mode Example

![Vase mode print setup](docs-v3/images/vasemode.png)

This example shows a vase-mode print with a solid bottom. The Vase Mode Generator creates the helical wall path and can also output planar base curves. These curves are used with the Walls and Infill Generators to fill the bottom. The definition also shows additional Toolpath settings and options such as staggering.

[Download the Vase Mode Grasshopper definition](docs-v3/examples/vasemode.gh)

## The main components

You can send multiple Toolpaths to the processor to make one program. Toolpath settings take precedence over defaults: a value set on a Toolpath is used for that Toolpath; otherwise, the processor uses the corresponding value from FDM Defaults.

For more advanced workflows, Toolpaths can also inherit settings from other Toolpaths and be modified along their geometry.

<details>
<summary>FDM Toolpath</summary>

The **FDM Toolpath** combines a curve with printing properties such as extrusion volume. Right-click the component to reveal its optional property inputs.

![FDM Toolpath component](docs-v3/images/toolpaths-object.png)

Connect a curve to the `Curve` input; a polyline is recommended. Other curve types are automatically converted to polylines with 0.3 mm sampling. You can also connect another Toolpath to inherit its settings.

<details>
<summary>Toolpath Inheritance</summary>

![Toolpath inheritance example](Images/pasted_20260511-101654.png)

Toolpath components can be chained. A Toolpath inherits settings from the Toolpath connected to its input, then applies its own local overrides.

In the example, each Toolpath keeps its own speed, while the Z-Hop value is set to **3.2** for both.

If a property is not set locally or inherited from the connected Toolpath, the value from **FDM Defaults** is used.

</details>

<details>
<summary>Toolpath Geometry</summary>

![Transformed Toolpath geometry](Images/Rhino_1zHBruW1qc.avif)

Toolpaths are geometry, so you can transform them with standard Grasshopper components such as Move, Array, and Transform. Toolpath properties stay attached through these transformations.

</details>

<details>
<summary>Extrusion Modes</summary>

TOOLPATHS has five extrusion modes. They define how much material is deposited for each millimeter of travel, or specify that no material is deposited.

![Extrusion mode options](Images/LYrHOhfWVO-2.png)

1. **Volume Mode:** Sets the volume of material deposited per millimeter of travel. For example, `3 mm³/mm` deposits 3 mm³ of material for every millimeter traveled. The simulator uses the volume and the height of the material below the nozzle to calculate the preview geometry.
2. **Static Mode:** Sets a fixed extrusion width and height. The simulation does not adapt the extrusion shape to the height of the material below the nozzle. This can be faster for large models.
  ![Static extrusion mode](Images/XsDMSWZAtk-2-4.png)
3. **Auto Width Mode:** Sets a target extrusion width. TOOLPATHS calculates the required volume from the available height below the nozzle. This is useful when layer height varies, such as in non-planar printing.
4. **Auto Ratio Mode:** Sets a target ratio between extrusion width and height. TOOLPATHS adjusts the extrusion volume to maintain that ratio.
5. **No Extrusion Mode:** Moves the printer along the path without depositing material.

**Flow** multiplies the extrusion amount calculated by the selected mode. For example, Auto Width Mode first calculates the volume needed to reach the target width, then applies the Flow multiplier. Flow can also be varied along the path with the Flow Modulator.

</details>

</details>

<details>
<summary>FDM Machine</summary>

The **FDM Machine** bundles the static settings that describe the 3D printer.

![FDM Machine component and settings](<docs-v3/images/fdm machine.png>)

| Setting          | Description                                                                                                                                      |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Rapid Speed**  | Travel speed for non-printing moves.                                                                                                             |
| **Max Z Speed**  | Maximum speed for Z-axis moves, including Z-hops.                                                                                                |
| **Bounds**       | The printer's build volume. Connect a Box.                                                                                                       |
| **Extruders**    | Connect one or more FDM Extruder components to define each extruder's index, nozzle and filament diameters, and material color.                  |
| **Start G-code** | Supply custom startup G-code. If none is supplied, TOOLPATHS generates basic initialization code; check that it is compatible with your printer. |
| **End G-code**   | Supply custom shutdown G-code. By default, TOOLPATHS moves up by the Z clearance distance and turns off the heaters.                             |
| **Toolchange**   | G-code to run at each toolchange. Use `[next_extruder]` for the next extruder index, for example `T[next_extruder]`.                             |
| **Center**       | Moves the print to the center of the build plate.                                                                                                |

</details>

<details>
<summary>FDM Defaults</summary>

**FDM Defaults** provides global process settings to the FDM Processor. These values are used when a Toolpath does not define or inherit the corresponding property. Right-click the component to add optional inputs.

![FDM Defaults component and settings](docs-v3/images/fdm-defaults.png)

| Setting                      | Nickname      | Description                                                                                                                                                   |
| ---------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **E Mode**                   | `Mode`        | Default extrusion mode: `0` Volume, `1` Auto, `2` Auto Ratio, `3` Static, `4` No Extrusion.                                                                   |
| **Safe Clearance**           | `safeClear`   | Clearance used for program entry and exit. The target is the greater of current Z plus clearance or the clearance value; min/max Z-Hop limits apply when set. |
| **Z-Hop Height**             | `ZHop`        | Default height for Z-hop moves between Toolpaths.                                                                                                             |
| **Min Z-Hop Height**         | `ZHopMin`     | Absolute minimum height for Z-hop moves, in millimeters.                                                                                                      |
| **Max Z-Hop Height**         | `ZHopMax`     | Absolute maximum height for Z-hop moves, in millimeters.                                                                                                      |
| **Speed**                    | `Speed`       | Default speed for extrusion moves, in millimeters per second.                                                                                                 |
| **Extruder Temperature**     | `Temp`        | Target extruder temperature.                                                                                                                                  |
| **Bed Temperature**          | `BedTemp`     | Target bed temperature.                                                                                                                                       |
| **Heightfield Resolution**   | `HFRes`       | Heightfield cell size, in millimeters. Smaller values capture finer detail at the cost of more computation.                                                   |
| **Meshing Resolution**       | `MeshRes`     | Spacing used when sampling intermediate points for the simulation mesh.                                                                                       |
| **Smoothing Window**         | `SmoothWin`   | Number of samples used by the Gaussian smoothing window.                                                                                                      |
| **Retraction**               | `Retraction`  | Default retraction distance, in millimeters.                                                                                                                  |
| **Retraction Extra Restart** | `RetExtra`    | Extra filament to push during unretraction, in millimeters.                                                                                                   |
| **Retraction Speed**         | `RetSpeed`    | Retraction speed, in millimeters per second.                                                                                                                  |
| **Unretraction Speed**       | `UnretSpeed`  | Unretraction speed, in millimeters per second. If unset, the retraction speed is used.                                                                        |
| **Degenerate Behavior**      | `DegenMode`   | How to handle degenerate samples: `0` interpolates from neighboring samples; `1` suppresses extrusion.                                                        |
| **Degenerate Aspect Ratio**  | `DegenAspect` | Width-to-height ratio above which an extrusion sample is considered degenerate.                                                                               |
| **Width Clamp**              | `WClamp`      | Nozzle-based width allowance for the aspect-ratio check. `0` disables this clamp; positive values set the allowance as a multiple of nozzle diameter.         |
| **Min Layer Height**         | `Hmin`        | Minimum layer height in millimeters. Samples below this are treated as degenerate; `0` uses the nozzle-based default.                                         |
| **Max Layer Height**         | `Hmax`        | Maximum layer height in millimeters. In Auto mode, extrusion is limited to stay within this height; `0` uses the nozzle-based default.                        |

The component outputs one **Defaults** object. Connect it to the `Defaults` input of the FDM Processor.

</details>

<details>
<summary>FDM Processor</summary>

The **FDM Processor** resolves Toolpaths, machine settings, and process defaults into one program and provides the data needed for simulation and output.

![FDM Processor component](docs-v3/images/processor.png)

<details>
<summary>Inputs</summary>

| Input              | Nickname   | Default      | Description                                                         |
| ------------------ | ---------- | ------------ | ------------------------------------------------------------------- |
| **Toolpaths**      | `T`        | —            | Toolpaths to combine into the program. Accepts a tree of Toolpaths. |
| **Machine**        | `M`        | —            | Optional FDM Machine settings.                                      |
| **Defaults**       | `Defaults` | —            | Optional FDM process defaults.                                      |
| **Processor Mode** | `Mode`     | `2` (Hybrid) | Selects preview and simulation behavior.                            |

</details>

<details>
<summary>Processor Modes</summary>

| Value | Mode           | Description                                                                                                                               |
| ----- | -------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `0`   | **Simulation** | Runs the full simulation. Simulation data is available for playback and export.                                                           |
| `1`   | **Preview**    | Builds a fast preview. G-code, robot, and other export outputs are unavailable in this mode.                                              |
| `2`   | **Hybrid**     | Shows a fast preview first, then replaces it with the full simulation. Export outputs become available when the full simulation finishes. |

</details>

<details>
<summary>Outputs</summary>

| Output              | Nickname | Description                                                                            |
| ------------------- | -------- | -------------------------------------------------------------------------------------- |
| **Simulation Data** | `D`      | Program and simulation data. Connect this to the FDM Simulator or an output component. |
| **Errors**          | `E`      | Errors reported while building the program.                                            |
| **Warnings**        | `W`      | Warnings reported while building the program.                                          |

</details>

</details>

<details>
<summary>FDM Simulator</summary>

The **FDM Simulator** displays the program as a mesh preview and provides controls for playback and visualization.

![FDM Simulator component](docs-v3/images/simulator.png)

<details>
<summary>Inputs</summary>

| Input                | Nickname | Default | Description                                                                      |
| -------------------- | -------- | ------- | -------------------------------------------------------------------------------- |
| **Simulation Data**  | `D`      | —       | Simulation data from the FDM Processor.                                          |
| **Time**             | `Time`   | `1.0`   | Simulation position, from `0.0` (start) to `1.0` (end).                          |
| **Play**             | `Play`   | `False` | Plays the simulation in real time.                                               |
| **Speed**            | `Speed`  | `1.0`   | Playback speed multiplier.                                                       |
| **Stadium Segments** | `SS`     | `3`     | Number of segments used for the stadium extrusion profile.                       |
| **Bake Meshes**      | `Bake`   | `False` | Outputs combined meshes to Grasshopper when enabled.                             |
| **Colors**           | `Colors` | —       | Optional color and material settings.                                            |
| **Progress Overlay** | `Prog`   | `True`  | Shows a viewport progress overlay while the processor or simulator calculates.   |
| **Disable Preview**  | `Hide`   | `False` | Disables viewport previews and conduits when enabled.                            |
| **UV Scale**         | `UVs`    | `0.01`  | Optional texture-coordinate scale. Add it from the component's right-click menu. |

</details>

<details>
<summary>Outputs</summary>

| Output               | Nickname | Description                                        |
| -------------------- | -------- | -------------------------------------------------- |
| **Current Time**     | `Ct`     | Current simulated time in seconds.                 |
| **Meshes**           | `M`      | Generated extrusion mesh chunks.                   |
| **Tool Position**    | `P`      | Current toolhead position.                         |
| **Program Duration** | `Dur`    | Total program duration in a human-readable format. |

</details>

</details>

<details>
<summary>FDM G-code Output</summary>

The **FDM G-code Output** component compiles the FDM program into machine-specific G-code. It can save the file to disk or upload it to a supported printer.

![FDM G-code Output component](docs-v3/images/fdm-gcode.png)

<details>
<summary>Inputs</summary>

| Input | Nickname | Default | Description |
| --- | --- | --- | --- |
| **Simulation Data** | `D` | — | Simulation data from the FDM Processor. Full simulation data is required to compile G-code. |
| **Machine Type** | `Machine` | Klipper | Selects the G-code target: Klipper, Klipper (No Z), RepRap, Prusa, or Bambu. |
| **Output Directory** | `Dir` | Desktop | Parent folder for saved output. Files are organized in a `toolpaths gcode` folder and then by date. |
| **Save** | `Save` | `False` | Saves the compiled G-code to disk. |
| **Upload** | `Upload` | `False` | Uploads the saved file to the selected printer. Upload also triggers saving. |
| **Printer IP Address** | `IP` | — | IP address of the printer used for upload. |
| **Start Print** | `Start` | `False` | Starts printing after upload, if supported by the machine. |
| **Template 3MF** | `Template` | — | For Bambu printers, path to a `.gcode.3mf` template file from Bambu Studio. |
| **Output G-code** | `Out` | `False` | Outputs the compiled G-code to Grasshopper. This can be slow for very large files. |

</details>

<details>
<summary>Outputs</summary>

| Output | Nickname | Description |
| --- | --- | --- |
| **G-code** | `G` | Compiled G-code when **Output G-code** is enabled. |
| **Toolpath** | `T` | Resulting toolpath geometry. |
| **Info** | `Info` | Save and upload status. |
| **Verbose Debug** | `D` | Detailed output of individual machine movements when **Output G-code** is enabled. |
| **Toolpath Debug** | `TD` | Summary of toolpath structure and properties when **Output G-code** is enabled. |

</details>

</details>
