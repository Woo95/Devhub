# **Italian Brainrot Survivors** | [`Play Game`](https://naver.me/546Ro3q6)
---
## **Overview**
This 2D survival action game was built using the [[English/02_Game Projects/Self-Made Game Framework/index.md|self-made game framework]] in **collaboration with a game artist**, featuring a **Vampire Survivors style clone with Italian Brainrot memes** and serving as an experiment to evaluate the framework's scalability and reusability.

[**View Repository**](https://github.com/Woo95/SDL2_Italian_Brainrot_Survivors)<br/>[**Watch Demo**](https://youtu.be/KV2-Dnr4IIc)

---
## **Game Showcase**
Players must survive endless waves of enemies, with a final boss appearing after five minutes. Defeating enemies grants experience to upgrade weapons and power-ups, helping to maximize survival. Power-ups can also be purchased before the game starts to boost early survival.
![[Italian Brainrot Survivors (Media)#^b8a5d5]]

---
## **Core Features**

> ⚠️ Note: All code snippets are **simplified versions**. Full implementation in the [**repository**](https://github.com/Woo95/SDL2_Italian_Brainrot_Survivors)
### `1. CSV-Based Data Management`
All game-related data (mobs, weapons, power-ups, etc.) are **managed via CSV files**, making it **easy to balance and expand content by editing the CSV files without modifying the code**.
![[Italian Brainrot Survivors (Media)#^5fea91]]

Upon program startup, a `CDataLoader` instance is created and initialized in `CEngine::Init()`, loading all CSV data files and storing the datasets.
![[Italian Brainrot Survivors (Code)#^4357bd]]
Through this approach, developers can flexibly adjust game balance using only data, without modifying the code. This structure can be considered a core aspect of data-driven design, taking both maintainability and scalability into account.

---
### `2. Scrollable Environment System`
![[Italian Brainrot Survivors (Media)#^a2f6d9]]
The map **scrolls to appear endless with just a single texture** as the camera moves. **Objects are repositioned only when they exceed the `snap distance`**, keeping the map and environment naturally synchronized and **efficiently creating the scrolling effect**.

---
![[Italian Brainrot Survivors (Media)#^545232]]
The image uses four combined textures for illustration, though **the actual implementation relies on a single texture**. It **visually illustrates the concepts of** consistent spacing between environment objects, map unit placement, and `snap distance` for position correction.

---
#### `CScrollMapComponent Class`
**The map does not necessarily have to scroll**, **it was implemented as a component**. Therefore, the map used in the game is implemented by adding the `CScrollMapComponent`.

![[Italian Brainrot Survivors (Code)#^de19b5]]
The map works by **rendering only a portion of a single texture based on the camera's position**. When the camera is within the texture, only that area is displayed. If it crosses the boundary, the texture is **split into two or four parts depending on the direction**, creating a **scrolling map effect that appears endless with a single texture**.

---
#### `CScrollEnvObj Class`
This is a **class for environment objects in the scroll map**. Initially **considered separating this logic into a component**, but since multiple components like sprites and colliders **move together**, handling it **at the object level** was deemed **simpler and clearer**.

![[Italian Brainrot Survivors (Code)#^7dbaf2]]
`CScrollEnvObj` references the camera, and when an environment object **exceeds the `snap distance`**, it calculates the relative direction, **normalizes it in x and y, multiplies by the map scale**, and **efficiently updates the position in a single move**.

---
### `3. Mob Spawning and Respawning System`
![[Italian Brainrot Survivors (Media)#^a94b37]]
The `CMobSpawner` class **randomly spawns mobs within a set range around the camera**. The area is divided into four zones, one of which is selected for a random spawn, ensuring **enemies appear around the player with equal probability**.

---
![[Italian Brainrot Survivors (Media)#^797744]]
Mobs that **move beyond a certain distance from the player are respawned**, ensuring **enemies always surround the player, maintaining consistent tension**.

---
#### `CMobSpawner Class - Key Methods`
##### `Method #1: Mob Spawner Update Function`
![[Italian Brainrot Survivors (Code)#^d971b1]]
Called **every frame to manage mob spawning and respawning**, it gradually reduces the time interval and executes `SpawnMob()` and `RespawnMob()` when conditions are met. **If the player doesn't exist, it skips unnecessary calculations and exits immediately**.

---
##### `Method #2: Mob Spawn Function`
![[Italian Brainrot Survivors (Code)#^0425f5]]
Spawns regular mobs and sub bosses at **regular intervals**. Mobs **appear at random positions around the camera** calculated by [[Italian Brainrot Survivors (Code)#^b7fa85|GetRandomSpawnPos()]]. **Over time new mob types are unlocked**, gradually introducing stronger enemies.

---
##### `Method #3: Mob Respawn Function`
![[Italian Brainrot Survivors (Code)#^791190]]
Mobs that are off-screen and beyond a certain distance are respawned at positions calculated by [[Italian Brainrot Survivors (Code)#^b7fa85|GetRandomSpawnPos()]], ensuring **enemies are always present around the player**.

---
### `4. Player: Correlation of Stats, Inventory, and Weapons`
The player essentially has a **Stats Component** and an **Inventory Component**.
- **Stats Component**:
    - Manages the player's **base stats and stats gained from items**.
- **Inventory Component**:
    - Manages the **various weapon components** the player owns.
- **Weapon Component**:
    - **Creates the projectiles** fired by the weapon and **sets their initial state**.

---
#### `Projectile Firing and Damage Calculation Process`
##### 0. Correlation Diagram
![[Player - Correlation of Stats, Inventory, and Weapons_Eng.png]]

A concise diagram showing the correlation between the player and weapons. When attacking, the weapon creates a projectile and applies total damage from both the player's and weapon's attack damage.

---
##### 1. Projectile Creation and Damage Setup
![[Italian Brainrot Survivors (Code)#^1fbde8]]
When attacking with the weapon, it first plays a sound effect, then creates and initializes a projectile. **Applying the combined damage of the owner and the weapon power** to the projectile. This **ensures fired projectiles aren’t affected by later power-up changes**.

---
##### 2. Player Attack Damage Calculation
![[Italian Brainrot Survivors (Code)#^e8745e]]
The player's attack damage is calculated by **summing item stat bonuses** with the base attack. **The advantage of this approach** is that **debugging and maintenance are easier**, and **new weapons or stat systems can be added with minimal changes to existing code**.

---
## **Conclusion**
> While developing a game using the [[English/02_Game Projects/Self-Made Game Framework/index.md|self-made game framework]], unexpected issues were encountered and addressed, **allowing the framework to develop a more stable and reliable structure**. This project was meaningful, as it involved **improving the my game framework while creating a game with it**.