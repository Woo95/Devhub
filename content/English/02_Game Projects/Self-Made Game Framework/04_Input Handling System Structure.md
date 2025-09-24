# **Input Handling System Structure**
---
## **Overview**
The in-game input processing system consists of `CInputManager` and `CInputComponent`. It functions using the **keyboard and mouse input states**, along with the **structures that bind these inputs**. Each object binds the inputs it needs, and executes the **corresponding callback functions** in response to the inputs received.

**The key advantage of this system** is that it can handle multiple callback functions by combining various input conditions (input value + input state) for keyboard and mouse, allowing **flexible design and extension of input configurations**.

---
## **Input System Operating Structure | [[Input System Architecture Diagram_Eng.png|Full View]]**
![[Input System Architecture Diagram_Eng.png]]

- Execution order: `Initialization` → `Update`
- This diagram briefly illustrates the **interaction** of the **input manager** and **input component**.

---
## **InputUtils.h Descriptions** | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Core/Utils/InputUtils.h)
This utility defines common types and structures needed for managing keyboard and mouse inputs, input states, and event bindings, which are essential for in-game input processing.
### `1. EKeyAction Enum Class`
![[Self-Made Game Framework (Code)#^0b8d0a]]
- Enum class used to distinguish different types of key inputs.

---
### `2. FKeyState Struct`
![[Self-Made Game Framework (Code)#^186449]]
- Structure that stores the current input state of a single key.

---
### `3. FBindFunction Struct`
![[Self-Made Game Framework (Code)#^0cf9f9]]
- Structure that stores a pair of object pointer and function (callback).

---
### `4. FBinder Struct`
![[Self-Made Game Framework (Code)#^1318b8]]
- Structure that combines keyboard and mouse inputs and links them to corresponding callback functions.

---
## **Class Descriptions**
### `1. CInputManager Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Manager/InputManager.h)
![[Self-Made Game Framework (Code)#^1fdcb1]]
- **Role**:
    - A **singleton** that **manages the registered** keyboard and mouse **input states**.
    - Each frame, it **detects registered keyboard and mouse inputs** and **updates their input states**.
- **Key Variables**:
    - `mKeys`:
        - **Stores the input values and states** of registered keyboard keys.
    - `mMouses`:
        - **Stores the input values and states** of registered mouse buttons.
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^9a862b|bool RegisterKey(SDL_Scancode keyCode)]]:
        - Registers the keyboard keys to be used in `mKeys`.
    - [[Self-Made Game Framework (Code)#^991446|bool RegisterMouse(Uint8 button)]]:
        - Registers the mouse buttons to be used in `mMouses`.
    - [[Self-Made Game Framework (Code)#^242c9b|void UpdateInputState()]]:
        - **Detects the input states** of registered `mKeys` and `mMouses` every frame.
    - [[Self-Made Game Framework (Code)#^646ee8|void HandleInputState(bool& press, bool& hold, bool& release, bool isPressed)]]:
        - Called by `UpdateInputState()`.
        - **Updates the input states** of registered `mKeys` and `mMouses`.
    - [[Self-Made Game Framework (Code)#^a61a23|bool GetKeyState(SDL_Scancode keyCode, EKeyAction action)]]:
        - **Checks the current input state** of a specific keyboard key.
    - [[Self-Made Game Framework (Code)#^7fc416|bool GetMouseButtonState(Uint8 button, EKeyAction action)]]:
        - **Checks the current input state** of a specific mouse button.

---
### `2. CInputComponent Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Entity/Component/InputComponent.h)
![[Self-Made Game Framework (Code)#^a6482f]]
- **Role**:
    - Registers **input conditions** (input values and states) and **callback functions** in `mBinders` per object, and **executes the callbacks when the conditions are met**.
- **Key Variable**:
    - `mBinders`:
        - Each `FBinder` is stored with a string key, and each binder **stores input conditions** (input values and states) and **callback functions**.
- **Key Methods**:
    - [[Self-Made Game Framework (Code)#^196a5f|virtual void Update(float deltaTime)]]:
        - Compares registered input conditions in `mBinders` with `CInputManager`, and **executes callback functions** if matched.
    - [[Self-Made Game Framework (Code)#^22375e|void AddFunctionToBinder(...) & void AddFunctionToBinder\<T\>(...)]]:
        - **Registers callback functions** to `mBinders` based on string keys.
    - [[Self-Made Game Framework (Code)#^94cde9|void DeleteFunctionFromBinder(const std::string& key, void* obj)]]:
        - **Removes callback functions** from `mBinders` based on string keys.
    - [[Self-Made Game Framework (Code)#^cc72ca|void AddInputToBinder(...)]]:
        - **Registers input conditions** to `mBinders` based on string keys.

---
## **Conclusion**
> Through this implementation, I learned that separating the `CInputManager` and `CInputComponent` allows for **clean handling of custom key mappings, various input patterns, and complex input conditions**. Additionally, it provided a deep understanding of how to **design the input system in a flexible and extensible manner**.