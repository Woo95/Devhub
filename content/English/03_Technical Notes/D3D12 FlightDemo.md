# **D3D12 FlightDemo**
---
## **Overview**
This project is a lightweight **3D flight demo** in C++ using the **DirectX 12 graphics API**. Developed as a **learning exercise**, it focuses on the **CPU-side setup of the D3D12 rendering pipeline**. The demo also implements **design patterns** for the game-specific systems such as a **[[Command Queue for Actions|command queue for actions]]** and **scene stack management**.

![[D3D12 FlightDemo (Media)#^8ce308]]
[**View Repository**](https://github.com/Woo95/DirectX12_FlightDemo)

---
## **CPU-side Stages of the D3D12 Rendering Pipeline**
This focuses on the **CPU-side setup** and **execution of the DirectX 12 rendering pipeline**, covering **five core stages** from **device initialization** to **rendering**.
> ⚠️ Note: It does not detail GPU pipeline internals like shading or rasterization.

Initialization stages 1-5, are all executed once within `Game::Initialize()`:
![[D3D12 FlightDemo (Code)#^db2b1a|Game::Initialize()]]

---
### `Stage 1 - Device Initialization`
- #### What is the D3D12 Device?
	- The **D3D12 device** is the core interface between your program and the GPU. It manages all GPU-related tasks in D3D12, including:
		- **Creating GPU resources** like buffers and textures
		- **Managing** descriptor heaps and command objects
		- **Controlling** rendering operations and querying hardware capabilities
- #### Key Function
	- [[D3D12 FlightDemo (Code)#^771272|bool D3DApp::InitDirect3D()]]:
		- Called once by [[D3D12 FlightDemo (Code)#^cdc89e|D3DApp::Initialize()]]. It **creates the graphics device** and sets up communication with the GPU. It selects the best graphics adapter and initializes device resources like **descriptor sizes** (used to access GPU resources) and **multisampling support** (for anti-aliasing).

---
### `Stage 2 - Graphics Pipeline Configuration`
- #### What is it?
	- This stage defines **how rendering is performed** by configuring the GPU pipeline. It prepares the **root signature**, compiles **shader code**, and creates **Pipeline State Objects**.
- #### Key Functions
	- [[D3D12 FlightDemo (Code)#^3dc4c3|void Game::BuildRootSignature()]]:
		- Defines what **shader resources** ([[D3D12 FlightDemo (Code)#^1ee3b0|CBVs]], [[D3D12 FlightDemo (Code)#^c55cc0|SRVs]], [[D3D12 FlightDemo (Code)#^801570|Samplers]]) are available.
	- [[D3D12 FlightDemo (Code)#^9e39a9|void Game::BuildShadersAndInputLayout()]]:
	    - Compiles [[D3D12 FlightDemo (Code)#^e6dd51|HLSL shader code]] ([[D3D12 FlightDemo (Code)#^c15357|VS]]/[[D3D12 FlightDemo (Code)#^b15a68|PS]]) and tells the GPU how to read each vertex's data (e.g., a position is 3 floats, texture coordinates are 2 floats, etc.).
	- [[D3D12 FlightDemo (Code)#^95ff5b|void Game::BuildPSOs()]]:
		- Creates **Pipeline State Objects** that define the full rendering behavior — including shaders, how triangles are rasterized, how pixels are blended, and depth settings.

---
### `Stage 3 - Resource Asset Setup`
- #### What is it?
    - This stage prepares the **resources that will be used during rendering**; it loads **texture data**, defines **material properties**, and sets up the **descriptor heap** that allows shaders to access those resources.
- #### Key Functions
    - [[D3D12 FlightDemo (Code)#^d68072|void Game::LoadTextures()]]:
        - Loads **texture files** into GPU memory and associates them with shader-visible [[D3D12 FlightDemo (Code)#^c55cc0|SRVs]].
    - [[D3D12 FlightDemo (Code)#^1e6763|void Game::BuildMaterials()]]:
        - Defines **material properties** such as base color, roughness, and texture index to be referenced during rendering.
    - [[D3D12 FlightDemo (Code)#^a35546|void Game::BuildDescriptorHeaps()]]:
        - Allocates and populates the **descriptor heap** ([[D3D12 FlightDemo (Code)#^1ee3b0|CBVs]]/[[D3D12 FlightDemo (Code)#^c55cc0|SRVs]]/[[D3D12 FlightDemo (Code)#^2dd457|UAVs]]) that the GPU uses to look up textures and other bound resources.

---
### `Stage 4 - Geometry and Vertex Buffer Setup`
- #### What is it?
	- This stage generates the **meshes** and **shapes** that will be rendered; it creates **vertex and index buffers** for the GPU.
- #### Key Function
	- [[D3D12 FlightDemo (Code)#^e40791|void Game::BuildShapeGeometry()]]:
		- Creates **geometry data** (vertices, indices), generates **GPU buffers**, and organizes draw call parameters for rendering.
    

---
### `Stage 5 - Frame Resources Setup`
- #### What is it?
	- This stage prepares **per-frame memory structures** to store [[D3D12 FlightDemo (Code)#^f5f00b|constant buffers]] and [[D3D12 FlightDemo (Code)#^c80301|command allocators]], supporting CPU-GPU synchronization across frames.
- #### Key Function
	- [[D3D12 FlightDemo (Code)#^925a50|void Game::BuildFrameResources()]]: 
		- Allocates a **set of resources for each frame** (e.g., constant buffers and command allocators) to let the CPU prepare upcoming frames while the GPU works on the current one.

---
## **Conclusion**
> This project applies core **DirectX 12 concepts** through a clear, step-by-step pipeline setup. Beyond rendering, it applies systems like **[[Command Queue for Actions|command queue for actions]]** and **scene stack management**, enabling the systematic development and implementation of **scalable 3D applications** in C++. It served as a strong entry point into **low-level graphics programming** and **engine architecture** design.