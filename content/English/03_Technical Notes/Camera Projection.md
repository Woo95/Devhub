# **Camera Projection**
---
## **Overview**
This document explains the concept of **Camera Projection** in 2D and 3D rendering environments. It introduces the differences between perspective and orthographic projection, and demonstrates how orthographic projection is used in an actual project through simple code examples.

---
## **Projection Types**
> There are two primary types of projections used in most game engines.
### `1. Perspective Projection`
In a **Perspective Projection**, objects appear smaller as they get farther from the camera. This creates the visual illusion of depth, making the scene look more natural and life-like.
![[Camera Projection (Media)#^ef5534]]
#### Characteristics:
- **Depth Perception**: Nearby objects to the camera appear larger, and those farther away appear smaller.
- **Vanishing Point**: Parallel lines appear to converge at a point in the distance, creating a sense of perspective.
- **Field of View (FOV)**: The FOV defines the camera's visible scene angle. A larger FOV increases the sense of depth, making objects appear smaller and parallel lines converge more sharply.

---
### `2. Orthographic Projection`
In an **Orthographic Projection**, objects maintain their size regardless of distance from the camera. This projection is often used in 2D and some 3D games for a flat, non-perspective view.
![[Camera Projection (Media)#^a0ca3a]]
#### Characteristics:
- **No Depth Perception:** All objects appear the same size, regardless of how far or near they are to the camera.
- **Parallel Lines:** Parallel lines remain parallel with no convergence.
- **Orthographic Size**: Defines the height of the camera's viewable area, with the width automatically calculated based on the aspect ratio.

---
## **Practical Application Example**
### `Project: Spaceship Battle`
> *Spaceship Battle* is one of my old project, used here to demonstrate **Orthographic Camera-Relative Object Setup**.
> 
> [**View Repository**](https://github.com/Woo95/Unity_2D_SpaceShipBattle_Automatic_CameraSetup_With_Object_Pooling)

In *Spaceship Battle*, the game reacts to changes in orthographic size by adjusting object positions and screen boundaries in real time.
![[Camera Projection (Media)#^ed2b15]]
#### `PlayerBehaviour Class - Execution Flow`
- These methods are called once at the start of the game to set up the player:
	- [[Camera Projection (Code)#^1f0edb|Start()]] → [[Camera Projection (Code)#^1db78b|SetPlayerBoundaryCameraPerspective()]] → [[Camera Projection (Code)#^58ca00|SetPlayerPositionCameraPerspective()]]
- This method is repeatedly called during gameplay:
	- [[Camera Projection (Code)#^f510cd|ClampPlayerYPosition()]]
#### `PlayerBehaviour Class - Code Breakdown`
![[Camera Projection (Code)#^1db78b|SetPlayerBoundaryCameraPerspective()]]
This function calculates the vertical boundaries for the player based on the camera's position and orthographic size, ensuring the player stays within the visible area along the Y-axis.

---
![[Camera Projection (Code)#^58ca00|SetPlayerPositionCameraPerspective()]]
This function sets the initial spawn position of the player, ensuring the player starts at a correct position within the camera's view along both the X and Y axes.

---
![[Camera Projection (Code)#^f510cd|ClampPlayerYPosition()]]
This function clamps the player's Y-position to the boundaries calculated earlier, keeping the player within the vertical limits of the camera's view. The X-position remains fixed.

---
## **Conclusion**
> **Understanding camera projections is crucial** for many aspects of game development. The demonstration **above shows just one way this knowledge can be applied**. In 2D games, understanding how projections work **allows you to ensure consistent** object placement and screen boundaries **across different devices**, maintaining a smooth transition between resolutions and aspect ratios. This **leads to a stable, responsive game environment**.