# **2D Platformer Game**
---
## **Overview**
This project is a **2D platformer mobile game** developed in a month as part of a coursework project in 2023. It covers the **core gameplay features** typical of the genre and is structured around a simple **finite state machine (FSM)** to manage overall game flow and state transitions.

> ⚠️ Note: No design documents or assets were provided. I planned the game, built the system, and chose all assets myself.

[**View Repository**](https://github.com/Woo95/Unity_Mobile_Game_Woo)<br/>[**Watch Demo**](https://youtu.be/klfbza0nP0Q)

---
## **Game Showcase**
Reach the red flag at the top within 5 minutes. Avoid traps, navigate moving platforms, and either dodge or defeat monsters. You have 3 lives to complete the challenge.
![[2D Platformer Game (Media)#^c8d93c]]

---
## **Core Features**
> ⚠️ Note: All code snippets are **simplified versions**. Full implementation in the [**repository**](https://github.com/Woo95/Unity_Mobile_Game_Woo)
### `1. GameManager Class (FSM structure)`
The `GameManager` script functions as the **central controller** for runtime game flow. It uses a **FSM pattern** to transition between three major states: `PLAY`, `PAUSE`, and `GAMEOVER`.
![[2D Platformer Game (Code)#^96f9de]]
`InPlay()`, `InPause()`, and `InGameOver()` serve as entry points for each state, while `Update()` calls the appropriate state handler such as `ModifyPlay()` or `ModifyGameOver()` every frame, depending on the current game state.

---
### `2. Tilemap Layer Structure`
To manage both **visual clarity** and **development efficiency**, the **Tilemap objects** in this project were structured into a **clean hierarchy**. Although not code-based, this design choice played a key role in **organizing gameplay elements** and maintaining **consistent render order**.
![[2D Platformer Game (Media)#^8796f2]]
This **hierarchical setup** contributed to **faster iteration** and **more reliable scene organization**.

---
### `3. Parallax Scrolling Background`
Background layers scroll based on the camera's position, which itself follows the player, creating a **parallax effect** tied to player movement.
![[2D Platformer Game (Media)#^d8b9db]]
This system is implemented through two core classes, `CameraControl` for camera tracking and `BackgroundController` for background scrolling based on depth.

---
#### `CameraControl Class`
![[2D Platformer Game (Code)#^4abc81]]
The camera smoothly follows the player using `Lerp()`, then clamps its position within level bounds via `UpdateCameraBoundary()`, and finally updates background layers each frame.

---
##### `BackgroundController Class`
![[2D Platformer Game (Code)#^6de630]]
Each background layer moves at a different horizontal speed based on its `m_Percent` value. Layers farther from the camera scroll more slowly, creating depth through parallax scrolling.

---
### `4. Player Interaction Zones`
This section explains how the **player interacts** with the environment and game objects. These interactions include stomping enemies, taking damage, collecting items, and detecting ground contact. The system supports **modular collision handling with a focus on precision**.
![[2D Platformer Game (Media)#^060aae]]

- The player object uses **one collider** and **two detection zones**:
	- 🟢 **Green Capsule**:
	    - CapsuleCollider2D.
	    - Collides only with ground.
	    - Disabled while jumping up; re-enabled when falling.
	- 🟨 **Yellow box**:
	    - Hit detection zone.
	    - Only detects traps, enemies, and items.
	- 🟥 **Red box**:
	    - Stomp zone.
	    - Detects enemies underfoot.
	    - Used to kill enemies on landing.

---
#### `PlayerController Class - Key Methods`

##### `Method #1: Green Capsule - Ground collision via CapsuleCollider2D`
![[2D Platformer Game (Code)#^15ab87]]
The green capsule is the main `CapsuleCollider2D` for **ground collision**. It's disabled when jumping up to pass through platforms and re-enabled when falling to detect landing.

---
##### `Method #2: Yellow box - Hit zone for traps, enemies, and items`
![[2D Platformer Game (Code)#^9d9c5c]]
The yellow box is a Gizmo-drawn **hit zone** used to detect traps, enemies, and coins. It ignores platforms, allowing the player to pass through them when jumping.

---
##### `Method #3: Red box - Stomp zone for killing enemies on landing`
![[2D Platformer Game (Code)#^d56094]]
The red box is a Gizmo-drawn **stomp zone** below the player's feet. It detects enemies during a downward jump; if hit, the enemy is killed and the player bounces slightly upward.

---
## **Conclusion**
> This project focused on refining **player interaction** and **game responsiveness** through precise **collision handling** and trigger logic. While simple in scope, it served as a strong foundation for understanding **Unity's 2D platformer** and building **scalable gameplay features**.