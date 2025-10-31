# ITSv2: Intelligent Traffic System v2 Documentation

Created by: **Nathan Wanjau.**

## Table of Contents
1.  [Plugin Overview](#1-plugin-overview)
2.  [Core Components](#2-core-components)
3.  [**IMPORTANT: Initial Project Setup**](#3-important-initial-project-setup)
    *   [Step 1: Create the VehicleSensor Trace Channel](#step-1-create-the-vehiclesensor-trace-channel)
    *   [Step 2: Configure Collision Responses](#step-2-configure-collision-responses)
4.  [Quick Setup Guide](#4-quick-setup-guide)
    *   [Step 1: Setting Up Your Vehicle Pawn](#step-1-setting-up-your-vehicle-pawn)
    *   [Step 2: Creating the Road Network](#step-2-creating-the-road-network)
    *   [Step 3: Setting Up the AI Spawner](#step-3-setting-up-the-ai-spawner)
5.  [Detailed Settings Reference](#5-detailed-settings-reference)
    *   [ITSv2 Spawner Actor](#itsv2-spawner-actor)
    *   [Vehicle Path Spline Actor](#vehicle-path-spline-actor)
    *   [ITSv2 Component](#itsv2-component)
6.  [Advanced Topics](#6-advanced-topics)
    *   [Creating Custom Vehicle Profiles](#creating-custom-vehicle-profiles)
    *   [How Intersections Work](#how-intersections-work)
    *   [Debugging Your AI](#debugging-your-ai)
    *   [Behavior Tree Logic](#behavior-tree-logic)

---

## 1. Plugin Overview

**ITSv2 (Intelligent Traffic System v2)** is a powerful and flexible AI plugin for Unreal Engine designed to bring your Chaos vehicles to life. It provides a complete framework for creating dynamic traffic systems, competitive racing AI, and other autonomous vehicle behaviors.

The system is built around a core `UITSv2Component` that acts as the AI's brain, which can be easily added to any Chaos Wheeled Vehicle Pawn. The world navigation is defined by a custom `AVehiclePathSpline` actor, allowing for the easy creation of complex road networks, racing lines, and parking areas.

### Key Features:
*   **Multiple Drive Modes:**
    *   **Follow Spline:** Ideal for creating predictable traffic flow, complex racing lines, and parking maneuvers.
    *   **Follow Target:** AI dynamically finds and follows a route to any target Actor in the world using the road network.
*   **Advanced Obstacle Avoidance:** Uses a configurable array of sensor rays to detect and dynamically maneuver around obstacles.
*   **Intelligent Intersection Handling:** AI vehicles can navigate complex intersections, yielding to traffic based on road priority or right-of-way rules to prevent collisions and deadlocks.
*   **Dynamic Traffic Spawning:** The `AITSv2_Spawner` automatically manages the vehicle population around the player, spawning and despawning AI to create a lively world without sacrificing performance.
*   **Racing & Overtaking AI:** Specialized logic for racing scenarios, including drafting, calculating optimal racing lines through corners, and aggressive overtaking. Traffic vehicles can also perform overtakes on two-way roads.
*   **Vehicle Profiles:** Fine-tune AI driving characteristics, from engine and transmission performance to steering and throttle response, using Data Assets.
*   **Ready-to-Use Content:** The plugin includes a pre-configured AI Controller, Behavior Tree, and Blackboard to get you started immediately.

---

## 2. Core Components

The plugin is composed of several key C++ classes and content assets that work together.

| Component/Asset         | Type                  | Description                                                                                                                               |
| ----------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **`ITSv2Component`**    | Actor Component       | The core AI brain. Add this to your vehicle pawn to give it driving capabilities. Contains all AI settings.                               |
| **`VehiclePathSpline`** | Actor                 | Used to build your road network. Can be configured as a traffic road, a racing line, or a parking spot.                                   |
| **`ITSv2_Spawner`**     | Actor                 | Manages the spawning and despawning of AI vehicles in the world. Also discovers and builds the road network graph for pathfinding.          |
| **`TrafficLight`**      | Actor                 | A simple but effective traffic light actor that can control the flow of traffic on a specific `VehiclePathSpline`.                         |
| **`ITSv2_AIController`**| AI Controller Asset   | A pre-made AI Controller that is already set up to run the plugin's behavior tree.                                                          |
| **`BT_Vehicle`**        | Behavior Tree Asset   | The decision-making logic for the AI. It handles states like driving, reversing, being stuck, etc.                                          |
| **`BB_Vehicle`**        | Blackboard Asset      | The memory for the Behavior Tree. It stores variables like the target actor, current path, and AI state flags.                            |
| **`ITSv2VehicleProfile`**| Data Asset            | A Data Asset used to store detailed driving physics and AI control parameters for different vehicle types (e.g., truck, sports car).        |

---

## 3. IMPORTANT: Initial Project Setup

Before you begin, you must configure your project's collision settings correctly. **The AI's obstacle avoidance system will not work properly without these steps.**

### Step 1: Create the VehicleSensor Trace Channel

1.  Go to **Edit -> Project Settings...**
2.  Under the **Engine** section on the left, click on **Collision**.
3.  Click the **New Trace Channel...** button.
4.  A dialog box will appear. Enter the following:
    *   **Name:** `VehicleSensor`
    *   **Default Response:** `Block`
5.  Click **Accept**.

### Step 2: Configure Collision Responses

For the AI sensors to work correctly, you must tell them what to **ignore**. If you skip this step, the AI may see invisible trigger boxes or the road itself as a wall and refuse to move.

*   **Road Network Trigger Boxes:**
    *   **Why:** The `AVehiclePathSpline` actors use trigger boxes for intersection logic. The AI should drive *through* these, not avoid them.
    *   **How:** For your `AVehiclePathSpline` Blueprint (or each instance in the level), select the `ForwardLaneTrigger` and `BackwardLaneTrigger` components. In the Details panel, go to the Collision section, set **Collision Presets** to `Custom...`, and change the response for the **`VehicleSensor`** trace channel to **`Ignore`**.

*   **Roads, Landscapes, and Ground Surfaces:**
    *   **Why:** The AI's sensors point slightly downwards to work correctly on hills and slopes. If the ground itself blocks the `VehicleSensor` channel, the AI will constantly detect the road immediately in front of it as an obstacle and may stop or drive erratically.
    *   **How:** Select your road meshes, landscape actors, and any other ground surfaces in your level. In their Collision settings, set the response for the **`VehicleSensor`** trace channel to **`Ignore`**.

---

## 4. Quick Setup Guide

After completing the Initial Project Setup above, you can now set up your AI vehicles.

### Step 1: Setting Up Your Vehicle Pawn

1.  **Create a Vehicle Pawn:** Start with a Blueprint based on the `Chaos Wheeled Vehicle Pawn` class.
2.  **Add the ITSv2 Component:** In the Components tab of your vehicle Blueprint, click "+ Add" and search for `ITSv2Component`.
3.  **Assign the AI Controller:**
    *   In the Class Defaults for your vehicle Blueprint, search for the "AI" category.
    *   Set the **`AI Controller Class`** to **`ITSv2_AIController`**. This controller is included in the plugin's content folder.

### Step 2: Creating the Road Network

The AI needs `VehiclePathSpline` actors to know where to drive.

1.  **Place Spline Actors:** From the "Place Actors" panel, search for `VehiclePathSpline` and drag them into your level to create roads. You can edit the spline points just like a standard engine spline.
2.  **Configure Path Type:**
    *   Select a spline actor. In its Details panel, find the **`Path Type`** setting.
    *   Set it to `Traffic Path` for a normal road.
3.  **Connect the Splines at Junctions:**
    *   **CRUCIAL:** For the `ITSv2_Spawner` to automatically detect an intersection, the trigger boxes at the ends of your `VehiclePathSpline` actors **must physically overlap** in the world. Position the end point of one spline and the start point of another so their trigger volumes intersect.

### Step 3: Setting Up the AI Spawner

The spawner brings your world to life by populating it with AI vehicles.

1.  **Place the Spawner:** Drag an `ITSv2_Spawner` actor from the "Place Actors" panel into your level.
2.  **Assign Vehicle Classes:**
    *   In the spawner's Details, find the **`Vehicle Classes`** array.
    *   Add elements to this array and assign the Vehicle Pawns you created in Step 1.
    *   You can set a **`Spawn Probability`** for each class to control how frequently they appear.
3.  **Automatic Network Setup:** The spawner is the easiest way to build your road network.
    *   Ensure **`bAuto Assign Next Paths`** and **`bAuto Assign Path Priorities`** are enabled.
    *   When you press "Play", the spawner will automatically detect all `VehiclePathSpline` actors **whose end-point trigger boxes overlap** and treat them as intersections. It will create the path connections and determine which roads have priority (i.e., main roads vs. side roads).
    *   You can run this process in the editor by selecting the spawner and clicking the **`Find Intersections In Editor`** and **`Assign Next Paths In Editor`** buttons.
4.  **Press Play!** AI vehicles should now spawn and begin driving along the spline network.

---

## 5. Detailed Settings Reference

### ITSv2 Spawner Actor

This actor manages the AI population in your level and performs the crucial task of building the road network for navigation.

#### Spawner Settings
*   `Vehicle Classes`: This is where you define *what* the spawner can create. It's an array where you can add multiple types of vehicles. For each entry, you specify the vehicle's Blueprint class and its spawn probability (`Low`, `Medium`, `High`, `VeryHigh`).
*   `Max Spawned Vehicles`: The maximum number of AI vehicles that can exist in the world at any one time. This is a key performance setting.
*   `Min Spline Length For Spawning`: The spawner will ignore any `VehiclePathSpline` shorter than this value (in cm) to prevent spawning on small, unusable paths.
*   `Min Spawn Distance`: The minimum distance (in cm) from the player that an AI vehicle can be spawned. This prevents vehicles from "popping in" right in front of the camera.
*   `Max Spawn Distance`: The maximum distance (in cm) from the player where a vehicle can be spawned. Spawning happens in a ring between the min and max distances.
*   `Despawn Distance`: If an AI vehicle gets further than this distance (in cm) from the player, it will be destroyed to save resources. This should always be greater than `Max Spawn Distance`.
*   `Spawn Wait Time`: The delay in seconds between each spawn attempt.
*   `bAllow Spawning On Parking Spots`: If `true`, vehicles can be spawned directly onto splines that are marked with the `Parking` path type.
*   `Max Spawn Distance From Spline Start`: To prevent vehicles from spawning in the middle of a road, this value (in cm) limits how far along a spline a vehicle can be created.

#### Junctions
*   `bAuto Assign Next Paths`: **(Crucial)** When enabled, the spawner will automatically detect intersections on `BeginPlay` by finding `VehiclePathSpline` actors whose end trigger boxes overlap. It then automatically populates the `ForwardLaneNextPaths` and `BackwardLaneNextPaths` arrays for each spline, building the road network graph.
*   `bAuto Assign Path Priorities`: **(Recommended)** When enabled, the spawner analyzes the connections at each intersection and attempts to identify which paths form a "straight" road. It will automatically set the `PathPriority` of these splines to `High`, making them main roads.
*   `bDrive On Left`: A global setting that tells all AI vehicles whether to follow left-hand or right-hand traffic rules, primarily affecting intersection turns and overtaking logic.

#### AI Settings
These are default values that will be applied to the `ITSv2Component` of every vehicle created by this spawner.
*   `Sensor Distance`: The default maximum range (in cm) of the vehicle's obstacle avoidance sensors.
*   `Max Follow Speed Kmh`: The default speed limit (in km/h) for spawned vehicles. Note that a vehicle's final top speed is still limited by its physics profile.

#### Debug
*   `bDraw Debug Spheres`: If `true`, the spawner will draw colored spheres at potential spawn locations, indicating why a spawn succeeded or failed (e.g., too close, visible to player, blocked by another car).
*   `bDraw Intersection Debug Spheres`: If `true`, the spawner will draw persistent cyan boxes around every intersection it automatically detects, helping you visualize the junction areas.
*   `Find Intersections In Editor` (Button): Runs the intersection detection logic in the editor, drawing the debug boxes so you can verify your setup without needing to play the game.
*   `Assign Next Paths In Editor` (Button): Runs the full network connection and priority assignment logic in the editor. This is useful for pre-calculating the network.

---
### Vehicle Path Spline Actor

This actor is the fundamental building block of your road network. You place and shape these in your level to define where the AI can drive.

#### Path | Setup
*   `Path Type`: Defines the purpose of this spline.
    *   `Traffic Path`: A standard road. The actor will automatically generate a `ForwardLaneSpline` and `BackwardLaneSpline` based on the central `GuideSpline`.
    *   `Racing Path`: A path for racing AI. It generates `LeftBoundarySpline` and `RightBoundarySpline` to define the track limits.
    *   `Parking Spot`: A path for a parking maneuver. The AI will follow the `GuideSpline` directly to park.
*   `Parking Type`: (If `Path Type` is `Parking`) Defines how the AI should exit the spot.
    *   `Forward Exit`: The AI will drive forward to exit.
    *   `Reverse Exit`: The AI will perform a guided reverse maneuver to exit.
*   `Min/Max Parking Duration`: (If `Path Type` is `Parking`) The random time range (in seconds) that an AI will wait in a parking spot before attempting to leave.
*   `Path Priority`: (If `Path Type` is `Traffic`) Determines right-of-way at intersections. Vehicles on a `High` priority path will be given precedence over vehicles on `Normal` or `Low` priority paths.
*   `bIs One Way`: (If `Path Type` is `Traffic`) If `true`, traffic will only flow along the `ForwardLaneSpline`. The `BackwardLaneSpline` will not be generated, and the system will not create connections from it.
*   `Lane Offset`: (If `Path Type` is `Traffic`) The distance (in cm) from the central `GuideSpline` to the generated forward and backward lanes.
*   `Trigger Box Extent`: The size of the trigger volumes at the start and end of the spline. These triggers are crucial for intersection detection, so they must be large enough to overlap with the triggers of connecting splines.

#### Path | Racing
*   `Racing Path Left/Right Offset`: (If `Path Type` is `Racing`) The distance (in cm) from the central `GuideSpline` to the generated left and right track boundaries.
*   `Close Spline Loop` (Button): A utility button for racing paths. It connects the last point of the `GuideSpline` to the first point, creating a seamless loop for a racetrack.

#### Path | Connections
This is where you define how splines connect to each other. **It is highly recommended to let the `ITSv2_Spawner` manage these automatically by setting `bAuto Assign Next Paths` to true.**
*   `Forward Lane Next Paths`: An array defining all the possible splines an AI can drive to after reaching the end of the `ForwardLaneSpline`.
*   `Backward Lane Next Paths`: An array defining all the possible splines an AI can drive to after reaching the end of the `BackwardLaneSpline`.
*   `Clear Next Path Connections` (Button): Manually clears both `Next Paths` arrays.
*   `Intersection Curve Tangent Scale`: A visual-only setting that controls the "roundness" of the green debug curves that are drawn in the editor to show intersection connections.
*   `Junction Point Debug Sphere Radius`: Controls the size of the orange debug sphere drawn on the visualized intersection curves.

---
### ITSv2 Component

This is the core component that gives your vehicle its AI brain. All driving behaviors, from high-level decision-making to low-level control inputs, are configured here.

#### Follow

This category controls the AI's primary goal and how it navigates the world.

*   `Drive Mode`: The most important setting, which determines the AI's fundamental behavior.
    *   `Follow Target Actor`: The AI will use the road network to dynamically calculate the best route to the `Target Actor`. Ideal for "mission" AI, taxis, or police cars.
    *   `Follow Spline`: The AI will follow a specific `VehiclePathSpline` assigned to its `Current Path` variable. Used by the spawner to create ambient traffic.
*   `Target Actor`: (In `Follow Target` mode) The actor the AI will attempt to drive to.
*   `Follow Traffic Rules When Following Target`: (In `Follow Target` mode) If `true`, the AI behaves like normal traffic. If `false`, it acts as a high-priority vehicle (like an ambulance), ignoring yielding rules.
*   `High Priority Speed Multiplier`: (In `Follow Target` mode) A multiplier applied to the AI's `Max Follow Speed Kmh` when it is in high-priority mode.
*   `Current Path`: (In `Follow Spline` mode) A reference to the `VehiclePathSpline` actor the AI should currently be following.
*   `Straight Lookahead Distance`: (In `Follow Spline` mode) How far ahead (in cm) on the spline the AI "looks" to calculate its target point on straight sections. Larger values result in smoother driving.
*   `Corner Lookahead Distance`: (In `Follow Spline` mode) A shorter lookahead distance (in cm) used for sharp turns to prevent cutting corners too widely.
*   `Lane To Follow`: (In `Follow Spline` mode) Determines which side of the `VehiclePathSpline` the AI will follow (`Forward` or `Backward` lane).
*   `Acceptance Radius`: The distance (in cm) from the final destination at which the AI considers its goal "reached".
*   `Max Follow Speed Kmh`: The general speed limit for this AI in kilometers per hour.

#### Vehicle Profile

Defines the physical performance and driving feel of the vehicle.

*   `Default Profile`: Choose a pre-configured profile (`Slow`, `Standard`, `Fast`, `Hyper`) for quick setup. Select `Custom` to use your own Data Asset.
*   `Vehicle Profile`: (When `Custom` is selected) Assign a `UITSv2VehicleProfile` Data Asset here to apply your custom engine, transmission, and AI control settings.
*   `Apply Vehicle Profile Settings` (Button): Click this in the editor to immediately apply the selected profile's settings to the vehicle's movement component.

#### Control

These settings tweak the low-level inputs and state-based behaviors of the AI.

*   `bFull Stop`: A master override. If `true`, the AI will immediately brake to a halt and hold its position. Used by the `TrafficLight` actor.
*   `Min/Max Post Queue Move Delay`: Creates natural, less robotic traffic flow by making the AI wait for a random duration (in seconds) between these values before moving after a car in front of it moves.
*   `Intersection Speed Multiplier`: A speed multiplier (0.0 - 1.0) applied when crossing an intersection to make the vehicle more cautious.
*   `Max Intersection Yield Time`: The maximum time (in seconds) the AI will wait for another car at an intersection before re-evaluating if the path is clear.
*   `Intersection Wait Time Until Force`: To prevent gridlock, if an AI waits at an intersection for this duration, it will ignore yielding rules and force its way through.
*   `Intersection Deadlock Wait Time`: How long a vehicle waits in a gridlock *inside* an intersection before attempting to reverse to clear the way.
*   `Post Blockage Wait Duration`: How long a high-priority vehicle will wait after a car blocking it starts to move away.
*   `Steering Gain`: Multiplier for steering input. Higher values make steering more responsive.
*   `Throttle Gain`: Multiplier for throttle input. Higher values make the AI accelerate more aggressively.
*   `Brake Distance`: The distance from a target stop point where the AI will begin applying brakes.
*   `Hard Brake Angle Deg`: If the angle to the target point is greater than this, the AI will slam on the brakes to avoid overshooting.
*   `Overspeed Brake Intensity`: How strongly the AI brakes (0.0 - 1.0) when it is going faster than its current speed limit.
*   `Prevent Illegal Overtaking`: If `true`, the AI will avoid steering into oncoming traffic unless performing a deliberate, safe overtake.
*   `Following Time`: The minimum time gap (in seconds) the AI will try to maintain from the vehicle in front of it, determining its stopping distance in traffic.

#### Smarter AI | Overtaking
(Active only in `Follow Spline` mode)

*   `bAllow Traffic Overtaking`: If `true`, AI traffic can decide to overtake slower vehicles on two-way roads.
*   `Overtake Speed Advantage`: The minimum speed difference (in cm/s) required before an overtake is considered.
*   `Overtake Speed Boost`: A speed multiplier applied during the overtake maneuver.
*   `Overtake Oncoming Clearance`: How far ahead (in cm) the oncoming lane must be free of other vehicles for the AI to start overtaking.
*   `Overtake Pull In Distance`: How far in front (in cm) of the passed vehicle the AI must be before it starts merging back into its lane.
*   `Min Path Length For Overtake`: The AI will not attempt to overtake if the remaining distance on its current spline is less than this value.

#### Smarter AI | Racing
(Active in `Follow Spline` mode on a `Racing` path type)

*   `Racing Line Aggressiveness`: How much the AI prioritizes the inside line of a corner (0 = center, 1 = fully cuts to the apex).
*   `Racing Exit Unwind Distance`: How far past a corner's apex the AI continues to steer towards the outside of the track.
*   `Racing Apex Lookahead`, `Racing Entry Setup Distance`, `Racing Straight Setup Lookahead`: Various lookahead distances (in cm) used by the racing logic to plan its path through upcoming corners.
*   `Track Half Width` & `Lateral Safety Margin`: Defines the drivable surface of the racetrack for the AI.
*   `Draft Distance` & `Draft Speed Bonus`: Defines the range and effect of slipstreaming behind another car.
*   `Overtake Lateral Offset`: How far to the side (in cm) the AI will move when attempting a racing overtake.
*   `Lane Change Speed`: Smoothing time for lateral movements. Lower is faster/snappier.

#### Reverse

*   `bAllow Reverse`: Master toggle for whether the AI can reverse.
*   `Reverse Angle Threshold Deg` / `Reverse Distance Threshold`: If the angle to the target is greater than the threshold AND the distance is less than this threshold, the AI will perform a planned reverse (a three-point turn).
*   `Reverse Speed Limit Kmh`: The maximum speed when reversing.
*   `Reverse Throttle Scale`: A multiplier (0-1) to reduce throttle input when reversing.

#### Sensors

This crucial category defines the AI's perception of the world.
*   `bUse Sensors`: Master toggle for the entire sensor-based obstacle avoidance system.
*   `Traffic Queue Stopping Distance`: The desired gap (in cm) the AI will leave when stopping behind another vehicle in traffic.
*   `AI Vehicle Class`: The base Blueprint class for your AI vehicles. The AI uses this to identify other traffic for behaviors like queuing.
*   `Sensor Trace Channel`: The collision channel to use for sensor traces. **This must be set to the `VehicleSensor` channel you created in the project settings.**
*   `Num Sensor Rays`: The number of line traces to cast in the AI's vision cone. More rays give better obstacle resolution but cost more performance.
*   `Sensor Side Angle Deg`: The total angle of the sensor arc. 85 degrees means the sensors will fan out in a 170-degree arc in front of the vehicle.
*   `Sensor Distance`: The maximum length (in cm) of the sensor rays.
*   `Steering Avoidance Start Distance`: The distance at which the AI will begin to apply avoidance steering.
*   `Sensor Sweep Radius`: The thickness of the sensor traces. This should be large enough to prevent the AI from trying to squeeze through tiny gaps.
*   `Front/Rear Sensor Origin Left/Right`: The local-space starting positions for the sensor arrays. Use the gizmos in the viewport to position these at the corners of your vehicle.
*   `Sensor Brake Distance` / `Sensor Hard Brake Distance`: The distance from a detected obstacle at which the AI will begin to apply normal or hard braking.
*   `bUse Side Sensors`: Enables extra rays cast directly to the sides of the vehicle to prevent clipping walls or other cars during tight maneuvers.

#### Vehicle Dimensions

These settings help the AI understand its own physical size for maneuvering.

*   `Front/Rear Left/Right Corner`: The local-space positions of the four corners of the vehicle's chassis. Use the gizmos to position these correctly.
*   `Path Safety Margin`: An extra buffer (in cm) added to the vehicle's width when checking if a gap is wide enough to pass through.

#### Debug

*   `bDraw Debug`: The master toggle for all real-time debug visualizations. This is incredibly useful for understanding the AI's behavior. It will draw sensor rays, the current seek point, corner locations, and more.
*   `bVerbose Debug`: If `true`, prints the AI's current state (e.g., "Driving", "Reversing", "Yielding") as text above the vehicle in-game.
*   `bDraw State Debug`: Toggles the on-screen state text (`bVerboseDebug` must also be enabled).

---
## 6. Advanced Topics

### Creating Custom Vehicle Profiles

For ultimate control over AI driving behavior, create a custom `ITSv2VehicleProfile` Data Asset. This allows you to define a vehicle's Drivetrain, Aerodynamics, and AI Control parameters, which can then be assigned in the `ITSv2Component`.

### How Intersections Work

1.  **Detection:** The `AITSv2_Spawner` identifies intersections by finding `VehiclePathSpline` actors whose **end-point trigger boxes physically overlap**.
2.  **Trigger:** An AI vehicle enters the trigger box at the end of its current path.
3.  **Safety Checks:** Before proceeding, the AI checks if the path is physically clear, if the intersection exit is blocked, and yields to other vehicles based on priority and right-of-way rules.
4.  **Proceed:** If all checks pass, it drives through the intersection. A failsafe timer prevents permanent deadlocks.

### Debugging Your AI

Troubleshooting AI is easier with the built-in visualization tools in the `ITSv2Component`.

*   **`bDrawDebug`**: The master switch. Enables real-time drawing of sensor rays, vehicle corners, the AI's current seek point, and its avoidance path.
*   **`bVerboseDebug`**: When enabled, prints the AI's current high-level state (e.g., "Driving", "Reversing", "Yielding") as text above the vehicle in the game world.

### Behavior Tree Logic

The included Behavior Tree (`BT_Vehicle`) orchestrates the AI's high-level states.
*   **`BTS_UpdateAIState` (Service):** The core service that runs constantly, reading data from the `ITSv2Component` (like `IsStuck`, `DriveMode`) and writing it to the Blackboard to inform the tree's decisions.
*   **Main Branches:** The tree prioritizes logic in this order: 1) Handling being stuck or needing to reverse, 2) Executing the main driving logic, 3) Braking if there is no valid goal.
