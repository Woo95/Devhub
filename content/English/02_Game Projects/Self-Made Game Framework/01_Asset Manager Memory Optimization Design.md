# **Asset Manager Memory Optimization Design**
---
## **Overview**
`CAssetManager` is a **singleton** that stores **various resource managers** required for game execution, such as textures, sprites, animations, UI, fonts, and sounds. Through this, all necessary resources can be accessed and managed consistently. During the development, I considered several different implementation approaches, and **I've summarized their pros and cons below**.

> ⚠️ Note: During framework initialization, `CDataLoader` reads CSV data and passes it to the various managers within `CAssetManager`, which then use this data for resource management at runtime.

[**View Repository**](https://github.com/Woo95/SDL2_Game_Framework/tree/main/Template/Client/Include/Manager/Data/Resource)

---
## **Design and Implementation Approaches | [[AssetManager Architectures Diagram_Eng.png|Full View]]**
Three implementation approaches considered during the asset manager design are shown in the table and diagram below.

![[AssetManager Architectures Diagram_Eng.png]]

| Implementation Methods                                                                                                | Pros                                               | Cons                      |
| --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- | ------------------------- |
| (1) `CAssetManager` **dynamically allocated**, `all resource managers` **dynamically allocated separately**.          | low compile dependency                             | high memory fragmentation |
| (2) `CAssetManager` **dynamically allocated**, `all resource managers` **declared as value member variables**.        | low memory fragmentation                           | high compile dependency   |
| (3) Memory block allocated with `malloc`, `CAssetManager` and `all resource managers` **created with placement new**. | low compile dependency,<br>no memory fragmentation | more complex to implement |

After carefully considering the pros and cons, I **ultimately chose approach (3)**.

---
## **Code Breakdown**
![[Self-Made Game Framework (Code)#^aa2822]]
- The `PlacementNew<T>()` function creates an object of type `T` at the specified `memoryBlock` location using `placement new`, and then moves the memory block pointer to the next position for subsequent objects.

---
![[Self-Made Game Framework (Code)#^413967]]
- This constructor takes a memory block allocated with `malloc` and uses `PlacementNew<T>()` to create `all resource managers` **sequentially in contiguous memory**.

---
![[Self-Made Game Framework (Code)#^a4b1a3]]
- This function returns the `CAssetManager` singleton object. On the first call, it allocates memory for `CAssetManager` and `all resource managers` at once using `malloc`, and initializes them using `placement new`.

---
## **Conclusion**
> Initially, I implemented approaches (1) and (2), but their drawbacks were clear. I then chose **approach (3)**, which preserves their advantages and fixes the flaws. Although more complex, using `malloc` to allocate a **single memory block** and initializing with `placement new` was straightforward, thanks to my experience with a [[MemoryPool Library|memory pool library]]. This method **reduces compile-time dependencies**, **prevents memory fragmentation**, and **improves cache efficiency**, offering a balanced solution for performance and memory management.