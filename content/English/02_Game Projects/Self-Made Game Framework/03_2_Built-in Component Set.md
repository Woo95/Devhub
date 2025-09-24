# **Built-in Component Set**
---
## **Overview**
This document introduces the game framework's **built-in component set and roles**, which enable objects to function. During game development, custom **components can be implemented as needed**, but **most engines provide commonly used features as built-in components**. This framework also provides several built-in components.

> All components are created and released through the [[MemoryPool Library|memory pool]], **optimizing performance and memory management**.

[**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/tree/main/Template/Client/Include/Entity/Component)

---
## **Component Set**

> ⚠️ Note: Since most components are tied to other systems, detailed code explanations are covered in separate documents.

| Component          | Role                                                        |
| ------------------ | ----------------------------------------------------------- |
| **BoxCollider**    | Defines the box-shaped collision area of an object.         |
| **CircleCollider** | Defines the circular collision area of an object.           |
| **Input**          | Detects input on the object and executes actions.           |
| **Movement**       | Updates the object's position based on direction and speed. |
| **Rigidbody**      | Moves the object under physics control.                     |
| **Sprite**         | Draws the object's image or plays animations.               |
| **VFX**            | Plays animation-based visual effects on the object.         |
| **Widget**         | Attaches widgets to the object to display UI.               |

---
## **Conclusion**
> By using the built-in components, the functionality of objects can be **implemented quickly and easily**, while custom **components can be implemented as needed**. This approach improves **development efficiency**, **code reusability**, and **flexibility** throughout the game development process.