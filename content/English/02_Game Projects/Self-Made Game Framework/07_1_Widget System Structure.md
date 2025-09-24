# **Widget System Architecture**
---
## **Overview**
The widget system **manages the UI in a hierarchical structure and handles input processing**. By inheriting from `CUserWidget`, **custom widgets** can be easily implemented. [[07_2_Built-in Widget Set|Built-in widgets]] that are implemented by inheriting from `CWidget`, already support basic interactions such as dragging. This provides a **developer-friendly UI that can be applied with minimal effort**.

![[Self-Made Game Framework (Media)#^eabc68]]

---
## **Widget Input System Processing Flowchart | [[Widget Input System Processing Flowchart_Eng.png|Full View]]**
![[Widget Input System Processing Flowchart_Eng.png]]

This diagram illustrates the **update and input processing flow** of the widget system.

---
## **Widget Related Class Descriptions**
### `1. CSceneUI Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Scene/Extension/SceneUI.h)
![[Self-Made Game Framework (Code)#^e06264]]
- **Role**:
    - Class that **manages the UI elements in the scene** and handles mouse input to process **interaction with widgets**.
- **Key Variables**:
    - `mWidgets`:
	    - **List of root widgets directly registered** in the scene UI.
    - `mCurrHovered`:
	    - **Widget currently hovered by the mouse cursor**.
    - `mHeldWidget`:
	    - **Widget currently held by the mouse**.
- **Key Methods**:
	- [[Self-Made Game Framework (Code)#^7bf0af|CWidget* FindHoveredWidget(const FVector2D& mousePos)]]:
		- Returns a pointer to the **widget currently hovered by the mouse cursor**.
	- [[Self-Made Game Framework (Code)#^5cc348|void UpdateInput()]]:
		- **Checks the mouse position and button states**, calls the **unhover function** if a held widget leaves its area, and calls the **hover function** if a new widget is hovered.

---
### `2. CWidget Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Widget/Widget.h)
![[Self-Made Game Framework (Code)#^50ddf5]]
- **Role**:
	- Base class of all [[07_2_Built-in Widget Set|built-in widgets]] and an **abstract class**.
	- Organizes the UI using a **hierarchical structure** and **improves extensibility**.
	- Provides **interaction functionality** for all widgets.
- **Key Variables**:
	- `mParent`:
		- **Parent widget** of the current widget.
	- `mChilds`:
		- **Child widgets** of the current widget.
- **Key Methods**:
	- `virtual void HandleHovered(const FVector2D&, bool, bool, bool)`:
		- **Virtual function**, called when the **mouse is over the widget**.
	- `virtual void HandleUnhovered(const FVector2D&, bool, bool)`:
		- **Virtual function**, called when the **mouse leaves the widget**.

---
### `3. CUserWidget Class` | [**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/blob/main/Template/Client/Include/Widget/UserWidget.h)
![[Self-Made Game Framework (Code)#^93523d]]
- **Role**:
	- Base class of **user-defined widgets** and an **abstract class**.
	- Provides **basic interaction features** such as **mouse dragging**.
- **Key Variables**:
	- `mIsMovable`:
		- Indicates whether the widget is **draggable**.
	- `mDragOffset`:
		- Stores the **offset between the mouse and the widget position** during dragging.
- **Key Method**:
	- [[Self-Made Game Framework (Code)#^3df818|void HandleDragging(const FVector2D&, bool, bool, bool)]]:
		- On mouse click, processes **dragging** of the widget.
		    - **Drag Start**: Brings the widget under the mouse to the top.
		    - **Drag Move**: Updates the widget position based on the mouse.
		    - **Drag End**: Releases the mouse button and ends the drag.

---
## **Conclusion**
> The widget system provides **clear and structured UI management** through its hierarchical architecture, integrated **input handling and interactive features** to support an accessible UI. Designed similarly to Unreal Engine, it provided an understanding of **how built-in and custom widgets functioned within the UI hierarchy in practice**.