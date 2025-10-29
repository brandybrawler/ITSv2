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
5.  [Component Deep Dive](#5-component-deep-dive)
    *   [ITSv2 Component](#itsv2-component)
    *   [Vehicle Path Spline Actor](#vehicle-path-spline-actor)
    *   [ITSv2 Spawner Actor](#itsv2-spawner-actor)
    *   [Traffic Light Actor](#traffic-light-actor)
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

## 5. Component Deep Dive

### ITSv2 Component

This is the main component containing all the settings that define an AI's behavior.

#### Key Settings Categories

*   **Follow**: Configures the AI's primary goal, such as following a target or a spline.
*   **Vehicle Profile**: Defines the physical driving characteristics (engine, gears, etc.).
*   **Control**: General driving behavior like steering aggression and stopping logic.
*   **Smarter AI | Overtaking (Traffic)**: Enables and configures overtaking logic for regular traffic.
*   **Smarter AI | Racing**: Specialized settings for competitive racing behavior.
*   **Reverse**: Controls if and when the AI is allowed to reverse.
*   **Smarter AI | Unstuck**: Logic for detecting and recovering from being stuck.
*   **Sensors**: The heart of the obstacle avoidance system.
    *   `SensorTraceChannel`: This should be set to the **`VehicleSensor`** channel you created.
*   **Debug**: Toggles for visualizing AI behavior in real-time.

### Vehicle Path Spline Actor

This actor is used to visually define the paths for your AI.

*   **Path Type**: Can be `Traffic Path`, `Racing Path`, or `Parking Spot`.
*   **Path Priority**: Determines right-of-way at intersections (`Low`, `Normal`, `High`).
*   **Connections**: Defines valid exits from this spline. Best configured automatically by the **`AITSv2_Spawner`** by overlapping trigger boxes.

### ITSv2 Spawner Actor

The spawner is a powerful manager for your AI traffic population.

*   **Junctions**:
    *   `bAutoAssignNextPaths`: When enabled, analyzes the level on `BeginPlay` by detecting **overlapping trigger boxes** and automatically connects the road network.
    *   `bAutoAssignPathPriorities`: When enabled, attempts to identify "main roads" and grant them `High` priority automatically.
    *   `bDriveOnLeft`: Global setting for traffic direction rules.

### Traffic Light Actor

A simple actor to control traffic flow by setting the `bFullStop` variable on vehicles that enter its trigger volume and match its configured path.

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
