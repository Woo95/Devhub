---
title: Self-Made Grame Framework
draft: false
---

# **Self-Made Grame Framework**
---
## **Overview**
This project is a **2D game framework** created based on the **C++ language** and **SDL2 graphics API**. A game made using this framework can be found at [[Italian Brainrot Survivors]].

> ⚠️ Note: This is **not a copy of open-source projects on the internet**, but a framework **designed and implemented from scratch** after considering the required structures.

[**View Repository**](https://github.com/Woo95/SDL2_Game_Framework)

---
## **Framework Execution Flow Diagram | [[Framework Structure Diagram_Eng.png|Full View]]**
![[Framework Structure Diagram_Eng.png]]

The above image is a **simplified design diagram**, focused on the core execution flow.

---
## **📂 Project Exploration**
### **1. Key Implementations** (Note in Progress)
1. **[[Asset Manager Memory Optimization Design]]**
2. **[[Scene Manager and Resource Optimization Design]]**
3. **[[Scene-Layer-Object-Component Hierarchy]]**
	- **[[Built-in Component Set]]**
4. **[[Input Handling System Structure]]**
	- Related Component: `CInputComponent`
5. **[[Object Visual Composition and Animation Structure]]**
	- Related Component: `CSpriteComponent`, `CVFXComponent`
6. **[[Collision and Physics System Structure]]**
	- Related Component: `CBoxCollider`, `CCircleCollider`, `CRigidbody`
7. **[[Widget System Structure]]**
	- Related Component: `CWidgetComponent`
	- **[[Built-in Widget Set]]**

---
### **2. Goals and Directions**
Before starting this project, I clearly set the **goals and directions** that would serve as standards for the entire development process. Based on these, I aimed not only to implement simple functions but also to achieve a **framework with structural completeness**.
- **Goals**:
	1. **C++ and Low-Level Technical Skills**
		- Memory and Object Lifecycle Management
		- Design from a Cache Optimization Perspective
		- Debugging and Performance Analysis Capabilities
	2. **Understanding Commercial Engine Structure**
		- Overall Engine Execution Flow
		- Principles of Core Subsystems
		- Hierarchical Systems Interaction
- **Directions**:
	1. Consistent and Smooth Structure Design
	2. Proper Optimized Data Structure Choice
	3. Flexible Extensibility and Maintainability

---
### **3. Reflections and Challenges**
Throughout the development process, I remained faithful to the **goals and direction** I had set. In order to improve **technical completeness**, I repeatedly redesigned and reimplemented even functions that were already finished, applying **better structures and theoretically more appropriate data structures or flows**. These cycles of **repeated modification and reimplementation** were the result of prioritizing high technical quality, and their traces remain clearly in the **Git commit history**.

However, this approach led to the problem of **exceeding the planned schedule**, and the total development period extended to about **seven months**. Through this, I realized the importance of maintaining a **balance between technical obsession and practical judgment**, as well as making decisions within a **limited development timeframe**.

---
### **4. Results and Achievements**
This project was carried out based on the initial goals of **enhancing low-level technical skills**, **understanding engine architecture**, and **maintaining consistent design**, and it ultimately succeeded in **practically achieving these objectives**.

All major implemented features were completed through **self-designed and optimized** processes. In particular, **memory pool based object management**, **resource optimization design**, and **scene transition and collision optimization structures** significantly improved stability and performance.

Through this process, I built not just functional code, but a **flexible and extensible game framework reflecting the structural characteristics of commercial game engines**. At the same time, I enhanced my **C++ low-level optimization experience**, **comprehensive understanding of engine architecture**, and **ability to design consistent structures**.