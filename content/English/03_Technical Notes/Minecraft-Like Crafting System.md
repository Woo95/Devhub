# **Minecraft-Like Crafting System**

---
## **Overview**
This project features a **basic item inventory** and **Minecraft-style crafting system**. Players can manage items and combine them in crafting slots to create new ones. It's built for **easy expansion**, allowing developers to **quickly add new recipes**.

[**View Repository**](https://github.com/Woo95/Unity_Minecraft_Like_Crafting_System)<br/>[**Watch Demo**](https://www.youtube.com/watch?v=wVEw_mPlB5I)

---
## **Crafting Showcase**
This video demonstrates the crafting process of items using 8 predefined recipes in the system.
![[Minecraft-Like Crafting System (Media)#^737682]]

---
## **Class Descriptions**

#### `Item Related Classes`
- [[Minecraft-Like Crafting System (Code)#^c616c6|ItemInfo class]]:
	- A `ScriptableObject` that holds an item's icon, description, and `eItemType`, defining properties shared across all instances of the item.
	- [[Minecraft-Like Crafting System (Code)#^7e13a8|eItemType enum]]:
		- Unique item IDs used to categorize items, specifically **for the crafting algorithm**.
- [[Minecraft-Like Crafting System (Code)#^c6f0e7|ItemInstance class]]:
	- Represents an `ItemSlot` in the inventory UI. Tracks its `ItemInfo` and count, updates the icon and count display, and shows item details on hover.
#### `ItemSlot Related Classes`
- [[Minecraft-Like Crafting System (Code)#^db1b81|ItemSlot class]]:
	- **Manages item interactions** like picking, placing, swapping, and stacking.
	- Used as a **base class for `ItemSlot`** across both **inventory** and **crafting systems**.
- [[Minecraft-Like Crafting System (Code)#^0d06b1|CraftingInputSlot class]]:
	- **Inherits** from `ItemSlot` and handles item placement for crafting **input**.
	- Triggers updates to the crafting system after item interactions.
- [[Minecraft-Like Crafting System (Code)#^9be179|CraftingOutputSlot class]]:
	- **Inherits** from `ItemSlot` and handles crafting result **output**.
	- Generates crafted items and updates input materials when items are taken from the output slot.
#### `Crafting Related Classes`
- [[Minecraft-Like Crafting System (Code)#^24120e|Recipe class]]:
	- A `ScriptableObject` that **stores a crafting formula**, including the required **input items** and the resulting **output item**.
- [[Minecraft-Like Crafting System (Code)#^b3c61a|Crafting class]]:
	- Manages the **crafting system logic**.
	- It **checks for matching recipes** from crafting **input slots** and generates **output items** accordingly.

---
## **Project Highlights**

### `1. Customizable ItemSlot Interaction`
- Reusable event & condition logic.
- Easily extendable via `ItemSlot` inheritance.
#### **How It Works**
The `ItemSlot` class serves as the foundation of a flexible, modular framework for slot interactions.
![[Minecraft-Like Crafting System (Code)#^cc1460]]
- The `virtual void OnPointerClick()` method defines how the slot responds to clicks.
- Inside this method, **pre-built event and condition functions** are **rearranged like building blocks** to create custom behaviors.
- This approach allows you to **define complex interactions** without rewriting core logic.

---
Derived classes like `CraftingOutputSlot` simply `override` this method to customize behavior.
![[Minecraft-Like Crafting System (Code)#^8a723c]]
**Result:** You get clean, reusable logic that's **easy to extend**, **debug**, and **customize** per slot type.

---
### `2. Easy Recipe Setup`
- Add new recipes via `ScriptableObject`.
- `RecipeEditor` for intuitive visual configuration.
#### **How It Works**
[[Minecraft-Like Crafting System (Code)#^24120e|Recipe]] is a `ScriptableObject`, **edited easily** with [[Minecraft-Like Crafting System (Code)#^f0a932|RecipeEditor]]'s drag-and-drop interface.
![[Minecraft-Like Crafting System (Media)#^4c0250]]
**Result:** The `RecipeEditor` allows **easy setup** of the `Recipe`, as shown above, making the creation and management of crafting recipes **quick**, **error-resistant**, **easily configurable**, and **accessible** to both **developers** and **designers**.

---
### `3. Flexible Crafting Algorithm`
- Works with any n x n crafting grid.
- Shape-normalized recipe matching.
#### **How It Works**
![[Minecraft-Like Crafting System (Media)#^32d465]]
**The crafting algorithm is very straightforward:**
1. Combine all [[Minecraft-Like Crafting System (Code)#^7e13a8|eItemType]] **unique IDs** from the craft input slots into a single string, **row by row**.
	- If items are placed **consecutively on different rows** (e.g., like in Sample 3), **insert an extra character** between the [[Minecraft-Like Crafting System (Code)#^7e13a8|eItemType]] IDs to preserve the structure.
2. **Remove the outer** **"Empty slot item type"** chars from the created string.
3. You'll notice that they **all align correctly**, even if placed in different locations, **as long as their shapes match**.
4. Then, compare the resulting string against all defined recipe strings to check for a match.

---
## **Conclusion**
> This crafting system is **flexible**, **easy to expand**, and provides developers with the tools to quickly implement new **recipes**. Its **intuitive design** ensures smooth **integration** and **customization** for any project.