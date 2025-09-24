# **Scene-Layer-Object-Component Hierarchy**
---
## **Overview**
Self-made framework is structured as `Scene → Layer → Object → Component` hierarchy. Based on **object-oriented design principles**, it clearly defines the **roles at each level** while providing **efficient management** and **flexible extensibility**.

---
## **Hierarchy Diagram**
![[Scene-Layer-Object-Component Hierarchy_Eng.png]]

This diagram visually illustrates the hierarchy of each class.

---
## **Class Descriptions**

> ⚠️ Note: The code has been **simplified** for explanatory purposes regarding the Scene-Layer-Object-Component hierarchy.
### `1. CScene Class`
![[Self-Made Game Framework (Code)#^36e7b4]]
- **Role**:
    - An **abstract class** inherited by all scene classes.
    - Holds **multiple layers**, which **separate elements that need to be drawn sequentially**, such as backgrounds and objects.
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^31e555|CScene()]]:
        - Scene constructor. Creates and initializes layers from the [[MemoryPool Library|memory pool]].
    - [[Self-Made Game Framework (Code)#^1b2665|virtual ~CScene()]]:
        - Scene destructor. Releases layer memory while keeping the [[MemoryPool Library|memory pool]] maintained.
    - [[Self-Made Game Framework (Code)#^0c3634|virtual void Update(float deltaTime)]]:
        - Updates scene elements (layers, collision system, camera, UI).
    - [[Self-Made Game Framework (Code)#^c1f92b|virtual void LateUpdate(float deltaTime)]]:
        - LateUpdates scene elements (layers, collision system, UI).
    - [[Self-Made Game Framework (Code)#^8dbf95|virtual void Render(SDL_Renderer* renderer)]]:
        - Renders scene elements (layers, UI).
    - [[Self-Made Game Framework (Code)#^144420|T* InstantiateObject\<T, int\>(const std::string& name, ELayer::Type type)]]:
        - Creates an object using the [[MemoryPool Library|memory pool]].
            - If the initialization succeeds, it is registered to the specified layer type.
            - If the initialization fails, the memory is returned to the pool.

---
### `2. CLayer Class`
![[Self-Made Game Framework (Code)#^70bdb1]]
- **Role**:
    - A class that manages all objects within a layer.
        - Sequentially update, late-update, and render objects, performing **Y-axis sorting** when necessary.
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^65f9d6|~CLayer()]]:
        - Layer destructor. Releases all objects within the layer through the [[MemoryPool Library|memory pool]].
    - [[Self-Made Game Framework (Code)#^d0565b|void Update(float deltaTime)]]:
        - Update objects sequentially.
            - Objects with !Active state are marked for removal.
            - Objects with !Enable state skip the update.
    - [[Self-Made Game Framework (Code)#^e4106e|void LateUpdate(float deltaTime)]]:
        - Late-update objects in reverse order.
            - Objects with !Active state are removed and returned to the [[MemoryPool Library|memory pool]].
            - Objects with !Enable state skip the late-update.
    - [[Self-Made Game Framework (Code)#^549045|void Render(SDL_Renderer* renderer)]]:
        - If Y-axis sorting is enabled, sort all objects by Y-coordinate 1 time.
        - Render objects sequentially.
            - Objects with !Active or !Enable state skip the render.

---
### `3. CObject Class`
![[Self-Made Game Framework (Code)#^24ed16]]
- **Role**:
    - An **abstract class** inherited by all objects within a scene and layer.
    - Objects organize their **component hierarchy** around `mRootComponent`.
        - Sequentially update, late-update, and render `mRootComponent`.
- **Key Variable**:
    - `CComponent* mRootComponent`:
	    - The object's root component.
	    - The entry point of the object's component hierarchy, which also includes `CTransform`.
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^b8195f|CObject()]]:
        - Object constructor. Creates and initializes the root component.
    - [[Self-Made Game Framework (Code)#^b2dfbe|virtual ~CObject()]]:
        - Object destructor. Removes the root component.
    - [[Self-Made Game Framework (Code)#^6c4ee4|virtual void Init()]]:
        - Calls the initialization function of the root component.
    - [[Self-Made Game Framework (Code)#^321393|virtual void Update(float deltaTime)]]:
        - Calls the update function of the root component.
    - [[Self-Made Game Framework (Code)#^2a3d8a|virtual void LateUpdate(float deltaTime)]]:
        - Calls the late-update function of the root component.
    - [[Self-Made Game Framework (Code)#^fba18c|virtual void Render(SDL_Renderer* renderer)]]:
        - Calls the render function of the root component.
    - [[Self-Made Game Framework (Code)#^ab3424|T* AllocateComponent\<T\>(const std::string& name)]]:
        - Creates a component using the [[MemoryPool Library|memory pool]].

---
### `4. CComponent Class`
![[Self-Made Game Framework (Code)#^47760c]]
- **Role**:
    - Base class for all components that provide actual functionality to objects.
    - **Manages the component hierarchy** and can **contain other components as children**.
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^750ae3|CComponent()]]:
        - Component constructor. Creates and initializes a transform from the [[MemoryPool Library|memory pool]].
    - [[Self-Made Game Framework (Code)#^0e00e0|virtual ~CComponent()]]:
        - Component destructor. Returns all child components and transforms to the [[MemoryPool Library|memory pool]].
    - [[Self-Made Game Framework (Code)#^3286fc|virtual bool Init()]]:
        - Calls the initialization function of all child components, but returns false if any fail.
    - [[Self-Made Game Framework (Code)#^34b1f6|virtual void Update(float deltaTime)]]:
        - Update components sequentially.
            - Components with !Active state are marked for removal.
            - Components with !Enable state skip the update.
    - [[Self-Made Game Framework (Code)#^d23f4e|virtual void LateUpdate(float deltaTime)]]:
        - Late-update components in reverse order.
	        - Components with !Active state are removed and returned to the [[MemoryPool Library|memory pool]].
		        - This process also cleans up the transform hierarchy.
            - Components with !Enable state skip the late-updates.
    - [[Self-Made Game Framework (Code)#^7c581f|virtual void Render(SDL_Renderer* renderer)]]:
        - Render component sequentially.
            - Components with !Active or !Enable state skip the render.
- **Feature**:
    - The component class allows flexible hierarchy structuring, as illustrated in the image.
        ![[Component Hierarchy_Eng.png]]

---
## **Conclusion**
> Designing the Scene–Layer–Object–Component **hierarchy** **provided practical experience in applying object-oriented design principles**. It allowed me to **systematically organize** and **manage complex game elements**, while making **maintenance and future extensions** significantly easier.