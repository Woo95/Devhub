# **Scene Manager and Resource Optimization Design**
---
## **Overview**
`CSceneManager` is a **singleton** that manages scenes in a **stack(FILO) structure** during gameplay. Transitions ensure **consistent state changes** and **safe resource management**, while smart pointer-based handling allows shared resources to be reused without reloading, enabling **efficient memory use** and **fast switching**.

[**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Manager/SceneManager.h)

---
## **Scene Transition Types**
Scene transitions are requested using the [[Self-Made Game Framework (Code)#^933d3b|ChangeRequest(ETransition, ESceneState)]] function. When executed, the transition information is stored in `mPending` and processed at the start of the next frame in [[Self-Made Game Framework (Code)#^017baa|Update(float)]] via [[Self-Made Game Framework (Code)#^daf2a4|ChangeApply()]].

| Transition Types                                                      | Role                                                              | Step Details                                                                                                                                                                                                               |
| --------------------------------------------------------------------- | ----------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **[[Self-Made Game Framework (Code)#^19f52b\|PushScene()]]**          | Add a new scene to the top of the stack.                          | 1. Create a new scene.<br>2. Load resources for the new scene.<br>3. Add the new scene to the top of the stack.<br>4. Execute `Enter()` on the new scene.                                                                  |
| **[[Self-Made Game Framework (Code)#^d37f71\|PopScene()]]**<br>       | Remove the scene from the top of the stack.                       | 1. Execute `Exit()` on the top scene.<br>2. Unload resources of the top scene.<br>3. Remove the top scene from the stack.                                                                                                  |
| **[[Self-Made Game Framework (Code)#^459179\|SwapScene()]]**          | Remove the scene from the top of the stack, then add a new scene. | 1. Create a new scene.<br>2. Load new scene resources, reusing shared resources from the top scene.<br>3. Execute `PopScene()`.<br>4. Add the new scene to the top of the stack.<br>5. Execute `Enter()` on the new scene. |
| **[[Self-Made Game Framework (Code)#^c06fde\|ClearScenes()]]**        | Remove all scenes.                                                | 1. Repeatedly execute `PopScene()` until the stack is empty.                                                                                                                                                               |
| **[[Self-Made Game Framework (Code)#^e38631\|ClearThenPushScene()]]** | Remove all scenes, then add a new scene to the top of the stack.  | 1. Load new scene resources, carrying over shared resources from the previous scene.<br>2. Execute `ClearScenes()`.<br>3. Add the new scene to the top of the stack.<br>4. Execute `Enter()` on the new scene.             |

---
## **Scene Transition Resource Management (Code Breakdown)**
> ⚠️ Note: This code has been **simplified** to illustrate the scene transition resource management approach.

![[Self-Made Game Framework (Code)#^a53e5d]]
- Each `Resource Manager` holds resources using only `std::weak_ptr`. This maintains **weak references** without taking ownership of the resources, and the actual memory is automatically released when all references are gone in the scene that owns the resources via `shared_ptr`.

---
![[Self-Made Game Framework (Code)#^185ec2]]
- The `CScene` abstract class manages various resources such as `CTexture`, `CFont`, `CSFX`, `CBGM` using `std::shared_ptr`. This ensures that **each scene clearly owns the resources it needs**, and calling `UnloadResources()` clears the vectors to release all references. With smart pointer-based reference management, **unused resources can be safely release**.

---
![[Self-Made Game Framework (Code)#^71791d]]
- `SwapScene()` first creates the new scene and loads its resources, then calls `Exit()` on the existing scene, unloads its resources, and removes it from the stack. During this transition, **shared resources between the previous and current scenes are reused without reloading**, minimizing **transition delays** and improving **memory usage efficiency**.

---
## **Conclusion**
> The **scene stack structure** of `CSceneManager` and its **smart pointer-based resource management** enhance not only simple scene swapping but also the **stability of transitions** and the **reliability of memory management**. This allows various scene transition patterns to be applied as needed and supports **flexible architecture extension** and **long-term maintainability** across the framework.