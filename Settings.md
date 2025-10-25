## ITSv2 Component: Detailed Settings Reference

This is the core component that gives your vehicle its AI brain. All driving behaviors, from high-level decision-making to low-level control inputs, are configured here.

### Follow

This category controls the AI's primary goal and how it navigates the world.

*   **`Drive Mode`**: The most important setting, which determines the AI's fundamental behavior.
    *   `Follow Target Actor`: The AI will use the road network defined by `VehiclePathSpline` actors to dynamically calculate the best route to the `Target Actor`. This is ideal for "mission" AI, taxis, or police cars that need to reach a specific destination.
    *   `Follow Spline`: The AI will follow a specific `VehiclePathSpline` assigned to its `Current Path` variable. This is used by the `ITSv2_Spawner` to create ambient traffic that follows the road network.

*   **`Target Actor`**: (Active only in `Follow Target` mode) The actor the AI will attempt to drive to. This can be set to the player, another vehicle, or any actor in the world. The AI will find the nearest point on the road network to this actor and navigate there.

*   **`Follow Traffic Rules When Following Target`**: (Active only in `Follow Target` mode) If `true`, the AI will behave like a normal traffic vehicle, yielding at intersections and obeying traffic lights. If `false`, it will act as a high-priority vehicle (like an ambulance or police car), ignoring normal yielding rules and assuming right-of-way at intersections.

*   **`High Priority Speed Multiplier`**: (Active only in `Follow Target` mode) A multiplier applied to the AI's `Max Follow Speed Kmh` when `bFollowTrafficRulesWhenFollowingTarget` is `false`. A value of 1.5 would make the emergency vehicle drive 50% faster than its normal speed limit.

*   **`Current Path`**: (Active only in `Follow Spline` mode) A reference to the `VehiclePathSpline` actor the AI should currently be following. The `ITSv2_Spawner` sets this automatically when spawning a vehicle.

*   **`Straight Lookahead Distance`**: (Active only in `Follow Spline` mode) How far ahead (in cm) on the spline the AI "looks" to calculate its target point on straight sections. Larger values result in smoother driving but can cause the AI to cut corners.

*   **`Corner Lookahead Distance`**: (Active only in `Follow Spline` mode) How far ahead (in cm) on the spline the AI "looks" when it detects a sharp turn. A smaller value forces the AI to stay closer to the spline's path, preventing it from cutting the corner too widely.

*   **`Lane To Follow`**: (Active only in `Follow Spline` mode on a `Traffic Path`) Determines which lane of the `VehiclePathSpline` the AI will follow. The `ITSv2_Spawner` sets this automatically.
    *   `Forward Lane`: Follows the lane in the direction of the spline points (0 -> n).
    *   `Backward Lane`: Follows the lane in the opposite direction of the spline points (n -> 0).

*   **`Acceptance Radius`**: The distance (in cm) from the final destination at which the AI considers its goal "reached".

*   **`Max Follow Speed Kmh`**: The general speed limit for this AI in kilometers per hour. The actual top speed is also influenced by the vehicle's physics and the active Vehicle Profile.

### Vehicle Profile

This category defines the physical performance and driving feel of the vehicle.

*   **`Default Profile`**: Choose a pre-configured profile for quick setup.
    *   `Custom`: Unlocks the `Vehicle Profile` slot, allowing you to use a custom Data Asset for fine-grained control.
    *   `Slow (Truck)`, `Standard (Sedan)`, `Fast (Sports Car)`, `Hyper (Race Car)`: Built-in presets that configure the engine, transmission, and AI control settings to match the vehicle type.

*   **`Vehicle Profile`**: (Active only when `Default Profile` is `Custom`) Assign a `UITSv2VehicleProfile` Data Asset here to apply your custom vehicle performance settings.

*   **`Apply Vehicle Profile Settings` (Button)**: A utility button that you can click in the editor to immediately apply the selected profile's settings to the vehicle's movement component.

### Control

These settings tweak the low-level inputs and state-based behaviors of the AI.

*   **`bFull Stop`**: A master override. If `true`, the AI will immediately brake to a halt and hold its position, ignoring all other logic. This is used by the `TrafficLight` actor.
*   **`bIsStoppedInTrafficQueue` / `bIsWaitingInTrafficQueue`**: Read-only state flags for debugging, indicating if the AI is stopped behind another car.
*   **`Min/Max Post Queue Move Delay`**: When a car in front of the AI moves, it will wait for a random duration between these two values (in seconds) before it starts moving. This creates more natural, less robotic traffic flow.
*   **`Intersection Speed Multiplier`**: A speed multiplier (0.0 - 1.0) applied when the AI is actively crossing an intersection, making it more cautious.
*   **`Max Intersection Yield Time`**: The maximum time (in seconds) the AI will wait for another car at an intersection before re-evaluating if the path is clear.
*   **`Intersection Wait Time Until Force`**: To prevent gridlock, if an AI waits at an intersection for this duration, it will ignore yielding rules and force its way through.
*   **`Intersection Deadlock Wait Time`**: How long a vehicle waits in a gridlock *inside* an intersection before attempting to reverse to clear the way.
*   **`Post Blockage Wait Duration`**: How long a high-priority vehicle will wait after a car blocking it starts to move away.
*   **`Steering Gain`**: A multiplier for steering input. Higher values make steering more responsive.
*   **`Throttle Gain`**: A multiplier for throttle input. Higher values make the AI accelerate more aggressively.
*   **`Brake Distance`**: The distance from a target stop point where the AI will begin applying brakes.
*   **`Hard Brake Angle Deg`**: If the angle to the target point is greater than this, the AI will slam on the brakes to avoid overshooting.
*   **`Overspeed Brake Intensity`**: How strongly the AI brakes (0.0 - 1.0) when it is going faster than its current speed limit.
*   **`Prevent Illegal Overtaking`**: If `true`, the AI will avoid steering into oncoming traffic to get around an obstacle unless it is performing a deliberate, safe overtake maneuver.
*   **`Following Time`**: The minimum time gap (in seconds) the AI will try to maintain from the vehicle in front of it. This is a key factor in determining stopping distance in traffic.

### Smarter AI

This section contains advanced behavioral logic for more complex and realistic driving.

#### Overtaking
(Active only in `Follow Spline` mode)

*   **`bAllow Traffic Overtaking`**: If `true`, AI traffic can decide to overtake slower vehicles on two-way roads.
*   **`Overtake Speed Advantage`**: The speed difference (in cm/s) required before an overtake is considered.
*   **`Overtake Speed Boost`**: A speed multiplier applied during the overtake maneuver.
*   **`Overtake Oncoming Clearance`**: How far ahead (in cm) the oncoming lane must be free of other vehicles for the AI to start overtaking.
*   **`Overtake Pull In Distance`**: How far in front (in cm) of the passed vehicle the AI must be before it starts merging back into its lane.
*   **`Min Path Length For Overtake`**: The AI will not attempt to overtake if the remaining distance on its current `VehiclePathSpline` is less than this value.

#### Racing
(Active in `Follow Spline` mode on a `Racing` path type)

*   **`Racing Line Aggressiveness`**: How much the AI prioritizes the inside line of a corner (0 = center, 1 = fully cuts to the apex).
*   **`Racing Exit Unwind Distance`**: How far past a corner's apex the AI continues to steer towards the outside of the track.
*   **`Racing Apex/Entry/Straight Lookahead`**: Various lookahead distances (in cm) used by the racing logic to plan its path through upcoming corners.
*   **`Racing Steering Gain`**: A separate, typically higher, steering gain used only for racing.
*   **`Racing Corner Braking Aggression`**: A multiplier that affects how early and hard the AI brakes for corners.
*   **`Track Half Width` / `Lateral Safety Margin`**: Defines the drivable surface of the racetrack for the AI.
*   **`AIAggressiveness`**: A personality trait (0-1) that influences decisions like when to attempt an overtake.
*   **`Draft Distance` / `Draft Speed Bonus`**: Defines the range and effect of slipstreaming behind another car.
*   **`Overtake Lateral Offset`**: How far to the side (in cm) the AI will move when attempting a racing overtake.
*   **`Lane Change Speed`**: Smoothing time for lateral movements. Lower is faster/snappier.

#### Turns

*   **`bSlow For Sharp Turns`**: If `true`, the AI will analyze the spline ahead and reduce speed for sharp corners.
*   **`Sharp Turn Lookahead` / `Sharp Turn Angle Deg`**: Defines how far ahead to check for turns and what angle is considered "sharp".
*   **`Stuck On Turn Angle Deg`**: If the AI is stopped and the angle to its target is greater than this, it will assume it's stuck on a corner and initiate a reverse maneuver.

#### Unstuck

*   **`Time Until Stuck`**: If the AI's speed is below the `Stuck Speed Threshold Cms` for this many seconds, it will consider itself stuck.
*   **`Stuck Speed Threshold Cms`**: The speed (in cm/s) below which the AI is considered "stopped" for the purpose of stuck detection.
*   **`Initial Grace Period Duration`**: For this many seconds after spawning, the stuck detection is disabled to prevent false positives as the AI gets up to speed.
*   **`Unstuck Cooldown Duration`**: After performing an unstuck reverse, the AI will enter a cooldown period where it drives forward assertively for a short time.

#### Tight Spaces

*   **`bEnable Tight Space Maneuvering`**: If `true`, the AI will actively look for gaps between obstacles when its path is blocked, rather than just stopping.
*   **`Gap Preference Distance`, `Desired Gap Width`, etc.**: These settings fine-tune the gap-finding logic, such as how wide a gap must be and how the AI scores potential paths.

### Reverse

*   **`bAllow Reverse`**: Master toggle for whether the AI can reverse at all.
*   **`Reverse Angle Threshold Deg` / `Reverse Distance Threshold`**: If the angle to the target is greater than the threshold AND the distance is less than this threshold, the AI will perform a planned reverse (a three-point turn).
*   **`Reverse Speed Limit Kmh`**: The maximum speed when reversing.
*   **`Reverse Throttle Scale`**: A multiplier (0-1) to reduce throttle input when reversing, making it less aggressive.

### Navigation
(Active in `Follow Target` mode)

*   **`bUse Navigation`**: If `true`, the AI will use Unreal's Navigation System to generate a point-to-point path if a route along the road network cannot be found.
*   **`Repath Interval`**: How often (in seconds) the AI recalculates its path to the target.
*   **`Path Point Acceptance Radius`**: How close the AI must get to a navigation point before moving to the next one.

### Sensors

This crucial category defines the AI's perception of the world. **It is highly recommended to use the `Update Sensor Visualization` button to see how these settings affect the sensor layout.**

*   **`bUse Sensors`**: Master toggle for the entire sensor-based obstacle avoidance system.
*   **`Traffic Queue Stopping Distance`**: The desired gap (in cm) the AI will leave when stopping behind another vehicle in traffic.
*   **`AI Vehicle Class`**: The AI needs to know what class to look for to identify other traffic vehicles for behaviors like queuing. This should be set to your base vehicle pawn Blueprint.
*   **`Sensor Trace Channel`**: The collision channel to use for the sensor traces. You should set up a dedicated channel for your vehicles in your project's collision settings.
*   **`bUse Rear Sensors`**: If `true`, enables a second set of sensors for reversing.
*   **`Num Sensor Rays`**: The number of line traces to cast in the AI's vision cone. More rays give better resolution but cost more performance. 11 is a good starting point.
*   **`Sensor Side Angle Deg`**: The total angle of the sensor arc. 85 degrees means the sensors will fan out in an 170-degree arc in front of the vehicle.
*   **`Sensor Distance`**: The maximum length (in cm) of the sensor rays.
*   **`Steering Avoidance Start Distance`**: The distance at which the AI will begin to apply avoidance steering.
*   **`Sensor Sweep Radius`**: The thickness of the sensor traces (they are sphere-sweeps, not simple lines). This should be large enough to prevent the AI from trying to squeeze through tiny gaps.
*   **`Front/Rear Sensor Origin Left/Right`**: The local-space starting positions for the sensor arrays. Use the `MakeEditWidget` gizmos in the viewport to position these at the corners of your vehicle.
*   **`Update Sensor Visualization` (Button)**: Click this to draw debug visualizations of the sensor origins in the editor viewport.
*   **`Sensor Brake Distance` / `Sensor Hard Brake Distance`**: The distance from a detected obstacle at which the AI will begin to brake or perform a hard brake, respectively.
*   **`bUse Side Sensors`**: Enables extra rays cast directly to the sides of the vehicle to prevent clipping walls or other cars during tight maneuvers.

### Vehicle Dimensions

These settings help the AI understand its own physical size for maneuvering.

*   **`Front/Rear Left/Right Corner`**: The local-space positions of the four corners of the vehicle's chassis. These are used for debug drawing and some gap calculations.
*   **`Path Safety Margin`**: An extra buffer (in cm) added to the vehicle's width when checking if a gap is wide enough to pass through.

### Debug

*   **`bDraw Debug`**: The master toggle for all real-time debug visualizations. This is incredibly useful for understanding the AI's behavior. It will draw sensor rays, the current seek point, corner locations, and more.
*   **`bVerbose Debug`**: If `true`, prints the AI's current state (e.g., "Driving", "Reversing", "Yielding") as text above the vehicle.
*   **`bDraw State Debug`**: A toggle for the on-screen state text, requires `bVerboseDebug` to be active.
