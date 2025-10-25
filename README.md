# ITSv2: Intelligent Traffic System v2 Documentation

Created by: **Nathan Wanjau, Inc.**

## Table of Contents
1.  [Plugin Overview](#1-plugin-overview)
2.  [Core Components](#2-core-components)
3.  [Quick Setup Guide](#3-quick-setup-guide)
    *   [Step 1: Setting Up Your Vehicle Pawn](#step-1-setting-up-your-vehicle-pawn)
    *   [Step 2: Creating the Road Network](#step-2-creating-the-road-network)
    *   [Step 3: Setting Up the AI Spawner](#step-3-setting-up-the-ai-spawner)
4.  [Component Deep Dive](#4-component-deep-dive)
    *   [ITSv2 Component](#itsv2-component)
    *   [Vehicle Path Spline Actor](#vehicle-path-spline-actor)
    *   [ITSv2 Spawner Actor](#itsv2-spawner-actor)
    *   [Traffic Light Actor](#traffic-light-actor)
5.  [Advanced Topics](#5-advanced-topics)
    *   [Creating Custom Vehicle Profiles](#creating-custom-vehicle-profiles)
    *   [How Intersections Work](#how-intersections-work)
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
*   **intelligent Intersection Handling:** AI vehicles can navigate complex intersections, yielding to traffic based on road priority or right-of-way rules to prevent collisions and deadlocks.
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

## 3. Quick Setup Guide

Follow these steps to get your first AI vehicle driving in minutes.

### Step 1: Setting Up Your Vehicle Pawn

1.  **Create a Vehicle Pawn:** Start with a Blueprint based on the `Chaos Wheeled Vehicle Pawn` class.
2.  **Add the ITSv2 Component:** In the Components tab of your vehicle Blueprint, click "+ Add" and search for `ITSv2Component`.
3.  **Assign the AI Controller:**
    *   In the Class Defaults for your vehicle Blueprint, search for the "AI" category.
    *   Set the **`AI Controller Class`** to **`ITSv2_AIController`**. This controller is included in the plugin's content folder.

 <!-- Placeholder for an image -->

### Step 2: Creating the Road Network

The AI needs `VehiclePathSpline` actors to know where to drive.

1.  **Place Spline Actors:** From the "Place Actors" panel, search for `VehiclePathSpline` and drag them into your level to create roads. You can edit the spline points just like a standard engine spline.
2.  **Configure Path Type:**
    *   Select a spline actor. In its Details panel, find the **`Path Type`** setting.
    *   Set it to `Traffic Path` for a normal road. You can adjust the **`Lane Offset`** and mark it as **`bIsOneWay`** if needed.
3.  **Connect the Splines:** For an AI to drive from one spline to the next, you must connect them.
    *   Select the first spline. At the end of the spline, a trigger box is visualized.
    *   Find the **`Forward Lane Next Paths`** array in the Details panel.
    *   Add a new element to the array.
    *   Use the eyedropper to pick the *next* `VehiclePathSpline` actor in the world.
    *   The editor will draw a green debug curve showing the generated path through the intersection.

> **Pro Tip:** The `AITSv2_Spawner` can do this for you automatically! See the next step.

### Step 3: Setting Up the AI Spawner

The spawner brings your world to life by populating it with AI vehicles.

1.  **Place the Spawner:** Drag an `ITSv2_Spawner` actor from the "Place Actors" panel into your level.
2.  **Assign Vehicle Classes:**
    *   In the spawner's Details, find the **`Vehicle Classes`** array.
    *   Add elements to this array and assign the Vehicle Pawns you created in Step 1.
    *   You can set a **`Spawn Probability`** for each class to control how frequently they appear.
3.  **Automatic Network Setup:** The spawner is the easiest way to build your road network.
    *   Ensure **`bAuto Assign Next Paths`** and **`bAuto Assign Path Priorities`** are enabled.
    *   When you press "Play", the spawner will automatically detect all `VehiclePathSpline` actors that are close to each other and treat them as intersections. It will create the path connections and determine which roads have priority (i.e., main roads vs. side roads).
    *   You can run this process in the editor by selecting the spawner and clicking the **`Find Intersections In Editor`** and **`Assign Next Paths In Editor`** buttons.
4.  **Press Play!** AI vehicles should now spawn and begin driving along the spline network, respecting each other at intersections.

---

## 4. Component Deep Dive

### ITSv2 Component

This is the main component containing all the settings that define an AI's behavior.

#### Key Settings Categories

*   **Follow**
    *   `Drive Mode`: The primary behavior of the AI.
        *   `Follow Target Actor`: The AI will use the road network to find the best path to the specified `Target Actor`. It will dynamically update its path if the target moves.
        *   `Follow Spline`: The AI will follow the specified `Current Path` spline. This is used for traffic simulation spawned by the `ITSv2_Spawner`.
    *   `Max Follow Speed Kmh`: The general maximum speed limit for the vehicle in kilometers per hour.

*   **Vehicle Profile**
    *   This section allows you to define the physical driving characteristics of the vehicle. You can choose a `Default Profile` (like `Slow`, `Standard`, `Fast`) for quick setup, or select `Custom` to use a `Vehicle Profile` Data Asset for maximum control. (See [Advanced Topics](#5-advanced-topics) for more info).

*   **Control**
    *   `bFullStop`: An important flag that can be set externally (e.g., by a `TrafficLight` actor) to force the vehicle to a complete stop.
    *   `Steering/Throttle Gain`: Multipliers that control how aggressively the AI steers and accelerates. Higher values result in more responsive, but potentially less stable, driving.

*   **Smarter AI | Overtaking (Traffic)**
    *   `bAllowTrafficOvertaking`: If enabled, traffic vehicles will attempt to overtake slower vehicles on two-way roads.
    *   `OvertakeOncomingClearance`: How far ahead the opposite lane must be clear before the AI will commit to an overtake.

*   **Smarter AI | Racing**
    *   These settings are active when `Drive Mode` is `Follow Spline` and the `VehiclePathSpline` `Path Type` is `Racing`.
    *   `RacingLineAggressiveness`: Controls how tightly the AI tries to hit the apex of a corner (0 = stays in the middle, 1 = cuts to the inside edge).
    *   `DraftDistance`: The distance behind another car at which the AI is considered to be "drafting" and receives a speed boost.

*   **Reverse**
    *   `bAllowReverse`: Determines if the AI is capable of reversing.
    *   `ReverseAngleThresholdDeg`: If the angle to the next seek point is greater than this value, the AI will perform a three-point turn by reversing.

*   **Smarter AI | Unstuck**
    *   `TimeUntilStuck`: If the vehicle's speed is below `StuckSpeedThresholdCms` for this duration, it will trigger its "unstuck" logic (usually reversing).

*   **Sensors**
    *   `bUseSensors`: Master toggle for the obstacle avoidance system.
    *   `NumSensorRays`, `SensorSideAngleDeg`, `SensorDistance`: These define the AI's "vision cone". More rays provide more detailed information but have a minor performance cost.
    *   `Front/Rear Sensor Origin`: These FVectors define the local-space starting points for the sensor raycasts. You should position them at the corners of your vehicle for best results. Use the `UpdateSensorVisualization` button to see them in the editor.
    *   `SensorBrakeDistance` / `SensorHardBrakeDistance`: The distances at which the AI will begin to apply the brakes or slam on the brakes, respectively.

*   **Debug**
    *   `bDrawDebug`: Draws a wealth of real-time debug information, including sensor rays, corner locations, and the current seek point.
    *   `bVerboseDebug`: Prints state information to the screen for detailed debugging.

### Vehicle Path Spline Actor

This actor is used to visually define the paths for your AI.

*   **Path Type**
    *   `Traffic Path`: Creates a standard road with one or two lanes. The AI will drive along offset splines generated from the central `GuideSpline`. Use this for all city/road networks.
    *   `Racing Path`: Creates a single racing line with left and right boundary splines. The AI will use specialized racing logic to find the optimal line between these boundaries.
    *   `Parking Spot`: Defines a parking spot. The AI can pull into this path and will remain "parked" for a random duration before attempting to exit and rejoin traffic.

*   **Path Priority**
    *   When the spawner auto-detects an intersection, it needs to know which roads have the right-of-way. Paths with `High` priority will be treated as main roads, and vehicles on `Normal` priority paths will yield to them.

*   **Connections**
    *   `ForwardLaneNextPaths` / `BackwardLaneNextPaths`: These arrays define the valid exits from the current spline. While you can set these manually, it is highly recommended to use the **`AITSv2_Spawner`** to automatically configure them.

### ITSv2 Spawner Actor

The spawner is a powerful manager for your AI traffic population.

*   **Vehicle Classes**
    *   A list of vehicle Blueprints that the spawner is allowed to instantiate. You can set the probability for each to control the variety of traffic.
*   **Spawn/Despawn Distances**
    *   `MinSpawnDistance` / `MaxSpawnDistance`: The range from the player where new vehicles can be spawned.
    *   `DespawnDistance`: If a vehicle gets this far away from the player, it will be destroyed to save performance.
    *   > The spawner includes occlusion checks, so vehicles will not spawn directly in the player's line of sight.
*   **Junctions**
    *   `bAutoAssignNextPaths`: When enabled, the spawner analyzes the level on `BeginPlay` and automatically populates the `NextPaths` arrays on all `VehiclePathSpline` actors. **This is the recommended way to build your network.**
    *   `bAutoAssignPathPriorities`: When enabled, the spawner will attempt to identify "main roads" (those that pass straight through an intersection) and automatically set their `PathPriority` to `High`.
    *   `bDriveOnLeft`: A global setting that tells all AI how to behave. It affects which side of the road they drive on and right-of-way rules at intersections.

### Traffic Light Actor

A simple actor to control traffic flow.

1.  **Placement:** Place the actor so its `TriggerVolume` covers the area where you want vehicles to stop for the light.
2.  **Configuration:**
    *   Set the **`PathToControl`** to the `VehiclePathSpline` that the light should affect.
    *   Set the **`LaneToControl`** to either the `ForwardLane` or `BackwardLane` of that path.
    *   Set the **`VehicleClassToControl`** to your base AI vehicle class.
    *   Adjust the `Green/Yellow/Red Light Duration` values to control the timing.

When a configured vehicle enters the trigger, the traffic light will take control by setting its `bFullStop` property. When the light turns green or the vehicle exits the trigger, the light will release control.

---

## 5. Advanced Topics

### Creating Custom Vehicle Profiles

For ultimate control over AI driving behavior, you can create a custom Vehicle Profile.

1.  In the Content Browser, right-click and go to `Miscellaneous -> Data Asset`.
2.  Select `ITSv2VehicleProfile` as the class.
3.  Open the new Data Asset. You will find three main structs:
    *   **`Drivetrain`**: Controls the engine's torque curve, max RPM, gear ratios, and shift points.
    *   **`Aerodynamics`**: Sets the drag and downforce coefficients, which affect high-speed stability.
    *   **`AIControl`**: Contains AI-specific driving parameters like `SteeringGain`, `ThrottleGain`, and `FollowingTime`, overriding the defaults in the `ITSv2Component`.
4.  Once configured, select your AI vehicle, find its `ITSv2Component`, set the `Default Profile` to `Custom`, and assign your new Data Asset to the `Vehicle Profile` slot.

### How Intersections Work

The AI uses a multi-step process to navigate intersections safely:
1.  **Trigger:** The AI's pawn overlaps with the trigger box at the end of its current `VehiclePathSpline`.
2.  **Path Selection:** It randomly chooses a valid exit from the spline's `NextPaths` array.
3.  **Path Generation:** A smooth Bézier curve is generated from its current position to the entry point of the next spline.
4.  **Safety Checks:** Before entering the intersection, it performs several checks:
    *   **Physical Clearance:** Is the curved path physically blocked by another vehicle?
    *   **Exit Clearance:** Is the exit of the intersection blocked by a stopped vehicle? (This prevents the AI from "blocking the box").
    *   **Yielding:** It checks for other vehicles approaching the intersection. It will yield based on:
        *   Another vehicle is already in the intersection.
        *   The other vehicle is on a road with a higher `PathPriority`.
        *   Right-of-way rules (e.g., yield to the right if priorities are equal).
5.  **Proceed:** If all checks pass, it proceeds along the generated curve. If a check fails, it enters a "waiting" state and repeats the checks until the path is clear. To prevent deadlocks, a timer (`IntersectionWaitTimeUntilForce`) will eventually force the AI to proceed if it waits too long.

### Behavior Tree Logic

The included Behavior Tree (`BT_Vehicle`) orchestrates the AI's high-level states.
*   **`BTS_UpdateAIState` (Service):** This is the most important node. It runs constantly and reads data from the `ITSv2Component` (like `IsStuck`, `DriveMode`, etc.) and writes it to the Blackboard. This keeps the tree's knowledge of the world up-to-date.
*   **Branches:** The tree is divided into several main branches based on priority.
    1.  **Stuck/Reverse Logic:** The highest priority is to handle being stuck. If the `IsStuck` key is true, it will execute the `BTT_ExecuteStuckReverse` task. It also handles planned reverses for tight turns.
    2.  **Driving Logic:** If not stuck, it will execute the main driving logic.
    3.  **Drive Task (`BTT_DriveToSeekPoint`):** This task runs continuously while the AI is driving. On each tick, it gets the latest `SeekPoint` from the `ITSv2Component` and calls the `ExecuteDrive` function, which calculates the final steering, throttle, and brake inputs.
    4.  **Brake Task (`BTT_Brake`):** A simple task that is called if the AI has no valid goal, telling it to apply the brakes.
