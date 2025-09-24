# **Tower Defender: Waves of War**
---
## **Overview**
This project is a **mobile tower defense game** developed in a month as part of a coursework project in 2023. It features the **core gameplay elements** of the tower defense genre and was designed with a structured approach to key defense mechanics.

> ⚠️ Note: No design documents or assets were provided. I planned the game, built the system, and chose all assets myself.

[**View Repository**](https://github.com/Woo95/Unity_Mobile_Game_Woo)<br/>[**Watch Demo**](https://youtu.be/sOtYVJ6maYU)

---
## **Game Showcase**
Defend the central tower across 10 waves by collecting resources, building towers, and deploying autonomous units. Adapt your strategy as monster patterns shift and resource timing becomes critical.
![[Tower Defender - Waves of War (Media)#^735f14]]

---
## **Core Features**
> ⚠️ Note: All code snippets are **simplified versions**. Full implementation in the [**repository**](https://github.com/Woo95/Unity_Mobile_Game_Woo)
### `1. Randomized Resource & Monster Spawner`
Resources and monsters use separate randomized systems to serve distinct gameplay roles.
#### `Randomized Resource Spawner`
- **Resources** are **randomly placed at game start**. Some require multiple taps to collect and can spawn follow-up objects before being removed.
![[Tower Defender - Waves of War (Media)#^7b62fe]]
##### `FieldManager Class`
![[Tower Defender - Waves of War (Code)#^895f15]]
Spawns a given number of resource objects at **random positions** within the field bounds. Each object is placed only if the **spawn area is free of overlaps**, using its collider size for precise spacing. A maximum of 500 attempts per object ensures **safe placement without infinite loops**.

---
#### `Randomized Monster Spawner`
- **Monsters** are **spawned dynamically during gameplay** at regular intervals in each wave. They appear from random edges of the map, and spawn rate increases as the game progresses.
![[Tower Defender - Waves of War (Media)#^698c01]]
##### `EnemyManager Class`
![[Tower Defender - Waves of War (Code)#^d238ba]]
[[Tower Defender - Waves of War (Code)#^d238ba|SpawnEnemy()]] is triggered at every `m_NextSpawnTime` to instantiate a random enemy prefab at a **random edge of the map**. The spawn position blends between predefined corner points, ensuring enemies enter from all directions. Each enemy is initialized and added to `enemySpawnedList`.

---
### `2. Simple AI Pattern for Units (FSM Structure)`
All units follow simple **finite state machine (FSM)** to manage movement, targeting, and attack behaviors in real time. (see the [[2D Platformer Game]] for a similar FSM implementation)
![[Tower Defender - Waves of War (Media)#^fe06c5]]
The **yellow** and **red** circles drawn via `OnDrawGizmos()` visually represent each unit's `searchRadius` and `attackRadius`, respectively.
- **Guardians**
    - **State**:
        - `IDLE → CHASE → ATTACK`
        - Transitions based on enemy proximity.
    - **Targeting**:
        - [[Tower Defender - Waves of War (Code)#^e563a5|SearchEnemy()]] scans surrounding area with `Physics2D.OverlapCircleAll()` to locate the nearest `Enemy`.
    - **Combat Logic**:
        - Pursues target when within `searchRadius`, stops when close enough to attack. Returns to `IDLE` if no valid target remains.
- **Enemies**
    - **State**:
        - `MOVE → ATTACK`
        - Continuously seeks targets and engages when in range.
    - **Multi-Target Logic**:
        - Prioritizes targets in this order: 
	        - `Guardian → GuardianTower → CentralTower`
    - **Fallback Behavior**:
        - If no `Guardian` is within `searchRadius`, enemies continue moving toward the central tower as their final objective.
---
### `3. Mobile-Friendly Camera Control`
Supports **drag and pinch gestures** for camera movement and zoom. Keeps the camera view **within map boundaries** at all times.
![[Tower Defender - Waves of War (Media)#^feed51]]
#### `CameraManager Class`

![[Tower Defender - Waves of War (Code)#^2f3221]]
Handles **touch-based camera interaction** for mobile gameplay.
- [[Tower Defender - Waves of War (Code)#^2f3221| void MobileInput()]]:
    - **One-finger drag** pans the camera.
    - **Two-finger pinch** zooms in or out.
- [[Tower Defender - Waves of War (Code)#^089f5d|void ClampCameraPosition()]]:
    - Ensures the camera view remains **within the game world bounds**.
- [[Tower Defender - Waves of War (Code)#^34e87e|void Zoom()]]:
	- Adjusts **orthographic size** and recalculate the camera bounds to fit the current zoom level.

---
## **Conclusion**
> This project focused on building **tower defense systems** with **dynamic spawning**, **autonomous unit behavior**, and **mobile-friendly controls**. While compact in scope, it provided a solid foundation for designing **scalable game logic** and managing **multiple systems in real-time**.