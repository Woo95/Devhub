# **Collision and Physics System Structure**
---
## **Overview**
(1) The in-game collision and physics system **minimizes unnecessary collision checks** by utilizing **collision channels and profiles** within a `CQuadTree` based **spatial partitioning algorithm**. The `CCollisionManager` provides **collision detection algorithms** to determine if **selected colliders** actually collide.

![[Self-Made Game Framework (Media)#^b7e46a]]

---
(2) **Collisions are handled between collider pairs** and **collision events** (ENTER/STAY/EXIT) are notified. **When physics processing is required**, `CPhysicsManager` and `CRigidbody` resolve overlaps and handle physical responses to ensure **stable collision handling**.

![[Self-Made Game Framework (Media)#^4ba351]]

---
## **Collision and Physics System Flowchart**
### `1. Initialization Flowchart` | **[[Collision & Physics System Initialization Flowchart_Eng.png|Full View]]**
![[Collision & Physics System Initialization Flowchart_Eng.png]]

This diagram briefly illustrates the **initialization process** of the collision and physics system.

---
### `2. Update Flowchart` | **[[Collision & Physics System Update Flowchart_Eng.png|Full View]]**
![[Collision & Physics System Update Flowchart_Eng.png]]

This diagram briefly illustrates the **update process** of the collision and physics system.

---
## **Collision Related Class Descriptions**
### `1. CCollisionManager Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Manager/CollisionManager.h)
![[Self-Made Game Framework (Code)#^f68f25]]
- **Role**:
    - Stores and manages all **collision profiles**.
    - Provides **collision calculation algorithms** to determine collisions among various colliders.
- **Key Variable**:
    - `mProfileMap`:
        - Manages **collision profiles for all colliders** and stores the interaction settings between them.
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^fb1690|bool AABBCollision(CBoxCollider* collider1, CBoxCollider* collider2)]]:
        - **Determines AABB overlap between two box colliders** and records the intersection point in `mHitPoint` for both if they overlap.
    - [[Self-Made Game Framework (Code)#^0855e2|bool CircleCircleCollision(CCircleCollider* collider1, CCircleCollider* collider2)]]:
        - **Determines if two circle colliders overlap** based on the **sum of their radius compared to the distance between their centers**, and records the intersection point in `mHitPoint` for both if they overlap.
    - [[Self-Made Game Framework (Code)#^c34b99|bool AABBCircleCollision(CBoxCollider* collider1, CCircleCollider* collider2)]]:
        - **Determines collision between a box and a circle** based on the **distance from the circle's center to the nearest point on the box's edge**, and records that closest point in `mHitPoint` if they overlap.

---
### `2. CSceneCollision Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Scene/Extension/SceneCollision.h)
![[Self-Made Game Framework (Code)#^9b935a]]
- **Role**:
    - **Scene-level collision management system**, using `CQuadTree` to **manage registered collider pairs** and track their states (ENTER/STAY/EXIT).
    - Delivers collision events to each collider and performs **physical collision handling** via `CPhysicsManager` if needed.
- **Key Variables**:
    - `mQuadTree`:
        - Pointer to `CQuadTree`, which **manages the scene's colliders through spatial partitioning**.
    - `mPairs`:
        - Manages all collider pairs (`FColliderPair`) and their collision states (`EPair::Status`).
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^d34e98|void HandleCollision(CCollider* collider1, CCollider* collider2)]]:
        - Determines whether two colliders intersect and updates their states.
            - On collision enter, performs `OnCollisionEnter(CCollider* other)`.
            - On collision stay, performs `OnCollisionStay(CCollider* other)`.
	            - resolves **physical interactions** via `CPhysicsManager` if needed.
            - On collision exit, performs `OnCollisionExit(CCollider* other)`.
    - [[Self-Made Game Framework (Code)#^fa3f6c|void CleanPairs()]]:
        - If any **collider pairs are inactive**, triggers exit events and removes the pair from `mPairs`.

---
### `3. CQuadTree Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Scene/Extension/QuadTree.h)
![[Self-Made Game Framework (Code)#^3cceff]]
- **Role**:
    - **Manages all colliders in the scene** and **manages the root node of the quadtree** according to the camera's view.
- **Key Variables**:
    - `mRoot`:
        - **The root node of the quadtree** (`CQTNode`). All colliders are distributed based on this node.
    - `mColliders`:
        - A container that **stores all colliders in the scene**.
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^033b8a|CQuadTree(CCamera* camera)]]:
        - Parameterized constructor, which **calculates the total number of nodes** based on `MAX_SPLIT`, creates a [[memoryPool library|memory pool]] for `CQTNode` of that size, and allocates memory to `mRoot` for initialization.
            
    - [[Self-Made Game Framework (Code)#^860dd5|void Update(float deltaTime)]]:
	    - Updates the area of `mRoot` based on the camera.
	    - Adds all colliders to `mRoot` and performs its update function.

---
### `4. CQTNode Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Scene/Extension/QTNode.h)
![[Self-Made Game Framework (Code)#^3f6061]]
- **Role**:
    - Represents **individual nodes in the quadtree**, handles **storing of colliders, node splitting, and reassigning colliders**.
    - Collision calculations are not performed directly, but are delegated to `CSceneCollision`.
- **Key Variables**:
    - `mColliders`:
        - A container that **stores colliders belonging to this node**.
    - `mChilds[4]`:
        - An **array of child nodes** created when the quadtree is subdivided.
    - `mBoundary`:
        - Defines the **spatial area** of the node.
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^03c03d|void Update(float deltaTime)]]:
        - If child nodes exist, **recursively** calls `Update` on each child node.
        - If no child nodes exist, checks collisions for all pairs of colliders in the node **if collision is enabled between the pair**.
    - [[Self-Made Game Framework (Code)#^b0d555|void AddCollider(CCollider* collider)]]:
        - **Checks if the collider** from the parent node **is within this node's bounds**.
            - If out of bounds, **ends the function**.
            - If within bounds:
                - If child nodes exist, forwards the collider to the appropriate child node.
                - If no child nodes exist, adds the collider to this node.
                    - If collider count exceeds threshold, **splits node** and **moves existing colliders to the child node**.

---
## **Physics Related Class Descriptions**
### `1. CPhysicsManager Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Manager/PhysicsManager.h)
![[Self-Made Game Framework (Code)#^6ed2ab]]
- **Role**:
    - **Manages the physical interactions** between colliders in the scene and, when they meet **physical processing conditions**, calculates the **minimum translation vector (MTV)** based on the `CRigidbody` type to **move the colliders**.
- **Key Variable**:
    - `CONST_MinMtvLSq`:
        - The **minimum squared length** used to ignore the MTV if its value is too small.
- **Key Method**:
    - [[Self-Made Game Framework (Code)#^fba119|void ResolveOverlapIfPushable(CCollider* collider1, CCollider* collider2)]]:
        - **Called when physical interaction is needed**; pushes colliders if one has `CRigidbody` of **DYNAMIC type with BLOCK interaction**.

---
### `2. CRigidbody Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Entity/Component/Rigidbody.h)
![[Self-Made Game Framework (Code)#^77c587]]
- **Role**:
    - Provides **physical properties** to colliders and **controls physical responses** according to `CRigidbody` type.
    - Moves the object by applying the displacement calculated by `CPhysicsManager`.
- **Key Variables**:
    - `mType`:
	    - Indicates the type of `CRigidbody` (DYNAMIC/STATIC/KINEMATIC).
    - `mMass`, `mGravityScale`, `mVelocity`, `mAcceleration`, `mAccumulatedForce`:
        - Indicates the motion state and accumulated forces, including mass and gravity, and is **used for collisions and physics**.
- **Key Method**:
    - [[Self-Made Game Framework (Code)#^a4c1df|void Update(float deltaTime)]]:
        - If `CRigidbody` is DYNAMIC type, it sequentially updates gravity, accumulated force, acceleration, velocity, and position based on mass.

---
## **Conclusion**
> This collision and physics system is based on **spatial partitioning algorithms** and **collision channels** to **reduce unnecessary collision checks** and **handle collision detection and physical responses reliably**. This allows for **efficient performance optimization**, **flexible collision control design**, and **makes interactions between various objects easier to implement**.