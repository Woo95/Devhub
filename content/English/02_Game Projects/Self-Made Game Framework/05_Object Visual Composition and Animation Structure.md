# **Object Visual Composition and Animation Structure**
---
## **Overview**
This document explains the **visual elements and animation structure** of objects through `CSpriteComponent` and `CVFXComponent`. It covers how object's visual elements operate **consistently and flexibly** by managing resources, ensuring animation independence via shallow copies, and controlling visual effects.

![[Self-Made Game Framework (Media)#^fb911d]]

---
## **Resource Relationship of Objects and Visual Components**
![[Object Visual Components & Animation Structure_Eng.png]]

When multiple objects share the same **animation**, they are **synchronized and executed simultaneously**. To allow **individual control**, animations are **shallow-copied**, and textures are shared across components using `std::shared_ptr`.

---
## **Class Descriptions**
### `1. CAnimationManager & CTextureManager Classes`
- **Role**:
    - `CAnimationManager` and `CTextureManager` are resource managers. Each handles animation and texture resources by **loading and storing them by key**, **provides them upon request**, and **manages their lifetimes**.

---
### `2. CAnimation Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Resource/Animation.h)
![[Self-Made Game Framework (Code)#^d17f0e]]
- **Role**:
    - Manages animation states and frame data. `EAnimationType::TIME` switches frames based on elapsed time, while `EAnimationType::MOVE` switches frames based on movement distance.
    - Creates individual instances through **shallow copying**, allowing each object to play animations independently.
- **Key Variable**:
    - `mAnimationStates`:
        - Container that stores data ([[Self-Made Game Framework (Code)#^475f27|FAnimationData]]) corresponding to each animation state key ([[Self-Made Game Framework (Code)#^3718fb|EAnimationState]]).
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^bfc61d|void Update(float deltaTime)]]:
        - Updates the animation logic according to the current state's type (`TIME` or `MOVE`).
    - [[Self-Made Game Framework (Code)#^63afad|CAnimation* Clone()]]:
        - Duplicates animation instances through a **shallow copy**.
    - [[Self-Made Game Framework (Code)#^947c38|void SetState(EAnimationState state)]]:
        - Changes the animation state and resets the frame index.

---
### `3. CSpriteComponent Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Entity/Component/SpriteComponent.h)
![[Self-Made Game Framework (Code)#^aa0928]]
- **Role**:
    - Component that **renders the object's sprite**.
    - Performs **camera culling** to render only when visible on the screen.
    - Supports **single frames; animation frames** via [[Self-Made Game Framework (Code)#^d17f0e|CAnimation]].
- **Key Variables**:
    - `mTexture`:
        - A texture resource managed by `std::shared_ptr`.
    - `mAnimation`:
        - An animation instance **cloned** for each object.
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^31ef87|void Update(float deltaTime)]]:
        - Updates the animation if it exists.
    - [[Self-Made Game Framework (Code)#^5639d3|void Render(SDL_Renderer* renderer)]]:
        - **If passes the camera culling**, get a single or animation frame and renders in **camera-transformed coordinates**.
    - [[Self-Made Game Framework (Code)#^f68fb3|void SetTexture(const std::string& key)]]:
        - **Stores the texture to the component** through `CTextureManager` by the specified key.
    - [[Self-Made Game Framework (Code)#^bc4e7a|void SetFrame(const std::string& key)]]:
	    - **Stores the single frame to the** `mFrame` through `CSpriteManager` by the specified key.
    - [[Self-Made Game Framework (Code)#^c0c05f|void SetAnimation(const std::string& key)]]:
	    - Gets the animation through `CAnimationManager` by the specified key, then clones it with [[Self-Made Game Framework (Code)#^63afad|Clone()]] and stores it in `mAnimation`.

---
### `4. CVFXComponent Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Entity/Component/VFXComponent.h)
![[Self-Made Game Framework (Code)#^17041f]]
- **Role**:
    - Component that **plays visual effect animations** for the object.
    - Performs **camera culling** to render only when visible on the screen.
    - Visual effects **render only while playing**; the next effect does not start until the current one completes.
- **Key Variables**:
    - `mTexture`:
        - A texture resource managed by `std::shared_ptr`.
    - `mAnimation`:
        - An animation instance **cloned** for each object.
    - `mPlayVFX`:
        - Flag indicating whether the VFX is currently playing.   
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^14b509|void Update(float deltaTime)]]:
        - Updates the animation **only if VFX is not playing**.
    - [[Self-Made Game Framework (Code)#^71386f|void Render(SDL_Renderer* renderer)]]:
        - Renders animation frames **only if VFX is not playing and passes camera culling**, then renders in **camera-transformed coordinates**.
    - [[Self-Made Game Framework (Code)#^9ffdbb|void SetTexture(const std::string& key)]]:
        - **Stores the texture to the component** through `CTextureManager` by the specified key.
    - [[Self-Made Game Framework (Code)#^788ffb|void SetAnimation(const std::string& key)]]:
        - Gets the animation through `CAnimationManager` by the specified key, then clones it with [[Self-Made Game Framework (Code)#^63afad|Clone()]] and stores it in `mAnimation`.
    - [[Self-Made Game Framework (Code)#^9192ae|void PlayVFX(const FVector2D& pos)]]:
        - Plays the visual effect at the specified `pos` **only if VFX is not playing**.

---
## **Conclusion**
> By using the `CSpriteComponent` and `CVFXComponent`, the visual elements of objects can be **managed efficiently and flexibly**. The texture structure managed by `std::shared_ptr` **optimizes resources**, and animations **cloned with shallow copies** allow **independent control per object**.