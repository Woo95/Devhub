# Code Block References
---

### Entity Classes
```cpp
#pragma once
#include <iostream>

class CObject
{
public:
	CObject() = default;
	virtual ~CObject() = default;

public:
	virtual void Update(float deltaTime) {}
};
```
```csharp
class CPlayer : public CObject
{
public:
	CPlayer() = default;
	virtual ~CPlayer() = default;

public:
	void PlayerAction(float deltaTime)
	{
		std::cout << "Player Action!\n";
	}
};
```

^bc13cb

```csharp
class CMonster : public CObject
{
public:
	CMonster() = default;
	virtual ~CMonster() = default;

public:
	void MonsterAction(float deltaTime)
	{
		std::cout << "Monster Action!\n";
	}
};
```

^55138a

### Scene Class
```cpp
#include <vector>
#include "Entity.h"
#include "CommandQueue.h"

class CScene
{
public:
	CScene() = default;
	virtual ~CScene()
	{
		for (CObject* obj : mObjects)
			delete obj;
	}

private:
	CCommandQueue mCommandQueue;

	std::vector<CObject*> mObjects;

public:
	void Update(float deltaTime)
	{
		// Process all pending commands
		while (!mCommandQueue.IsEmpty())
		{
			FCommand cmd = mCommandQueue.Dequeue();
			if (cmd.action)
				cmd.action(deltaTime);
		}

		// Update objects
		for (CObject* obj : mObjects)
			obj->Update(deltaTime);
	}

	void AddObject(CObject* obj)
	{
		mObjects.push_back(obj);
	}

	void PushCommand(const FCommand& command)
	{
		mCommandQueue.Enqueue(command);
	}
};
```

^6c2790

### Scene Update Function
```cpp
void CScene::Update(float deltaTime)
{
	// Process all pending commands
	while (!mCommandQueue.IsEmpty())
	{
		FCommand cmd = mCommandQueue.Dequeue();
		if (cmd.action)
			cmd.action(deltaTime);
	}
}
```

^5edefe

### Command Struct
```cpp
#pragma once
#include <functional>

struct FCommand
{
	using Action = std::function<void(float)>;

	Action action;
};
```

^27cad2

### CommandQueue Class
```cpp
#pragma once
#include <queue>
#include "Command.h"

class CCommandQueue
{
public:
	CCommandQueue() = default;
	~CCommandQueue() = default;

private:
	std::queue<FCommand> mQueue;

public:
	void Enqueue(const FCommand& command)
	{
		mQueue.push(command);
	}

	FCommand Dequeue()
	{
		if (IsEmpty())
			return FCommand{};

		FCommand cmd = mQueue.front();
		mQueue.pop();

		return cmd;
	}

	bool IsEmpty() const
	{
		return mQueue.empty();
	}
};
```

^50c1f1

### Example
```cpp
#define FakeDeltaTime 0.016f

int main()
{
	CScene*   scene   = new CScene;
	CPlayer*  player  = new CPlayer;
	CMonster* monster = new CMonster;

	scene->AddObject(player);
	scene->AddObject(monster);

	// This simulates 5 frames, pushing and executing one command per frame.
	for (int frame = 1; frame <= 5; frame++)
	{
		std::cout << ">> [Frame " << frame << "] <<\n";

		FCommand cmd;
		cmd.action = [player, monster, frame](float dt)
		{
			if (frame % 2 == 0)
				player->PlayerAction(dt);
			else
				monster->MonsterAction(dt);
		};
		scene->PushCommand(cmd);
		
		scene->Update(FakeDeltaTime);

		std::cout << "\n";
	}
	delete scene;

	return 0;
}
```

^8d7160