# **Built-in Widget Set**
---
## **Overview**
This document introduces the game framework's **built-in widget set and roles**, which are used to construct the UI, display elements on the screen, and enable user interaction. Custom **widgets can be implemented as needed**, but **most engines provide commonly used UI features as built-in widgets**. This framework also provides several built-in widgets.

> All widgets are created and released through the [[MemoryPool Library|memory pool]], **optimizing performance and memory management**.

[**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/tree/main/Template/Client/Include/Widget)

---
## **Widget Set**

| Widget            | Role & Features                                                                                                                          |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **AnimatedImage** | Plays animations using sequential images, allowing dynamic visual effects.                                                               |
| **Button**        | An interactive widget that handles various user inputs (hover, unhover, click, hold, release) and executes specific actions accordingly. |
| **Image**         | Displays a static image and is used for visual elements such as icons or backgrounds.                                                    |
| **ProgressBar**   | Visually represents progress as a bar and can be used for various purposes, such as health, mana, or other status indicators.            |
| **Slider**        | An interactive widget that allows users to adjust a value within a minimum and maximum range.                                            |
| **TextBlock**     | Displays text on the screen with customizable properties including font, color, style, and shadow.                                       |

---
## **Conclusion**
> By using the built-in widgets, UI can be **implemented quickly and easily**. Custom **widgets** can also be implemented by inheriting from `CUserWidget`, and **built-in widgets can be combined** as needed. This approach improves **development efficiency**, **code reusability**, and **flexibility** throughout the game development process.