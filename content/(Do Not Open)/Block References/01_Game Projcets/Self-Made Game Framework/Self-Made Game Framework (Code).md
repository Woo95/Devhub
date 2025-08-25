# Code Block References
---
## Eng & Kor

### CAssetManager
```cpp
template <typename T>
T* PlacementNew(void*& memoryBlock)
{
    T* manager  = new (memoryBlock) T;
    memoryBlock = (char*)memoryBlock + sizeof(T);
    
    return manager;
}
```

^aa2822

```cpp
CAssetManager::CAssetManager(void* memoryBlock)
{
    mTextureManager   = PlacementNew<CTextureManager>(memoryBlock);
    mSpriteManager    = PlacementNew<CSpriteManager>(memoryBlock);
    mAnimationManager = PlacementNew<CAnimationManager>(memoryBlock);
    mUIManager        = PlacementNew<CUIManager>(memoryBlock);
    mFontManager      = PlacementNew<CFontManager>(memoryBlock);
    mSoundManager     = PlacementNew<CSoundManager>(memoryBlock);
}
```

^413967

```cpp
CAssetManager* CAssetManager::GetInst()
{
    if (!mInst)
    {
        const size_t totalSize = sizeof(CAssetManager)
            + sizeof(CTextureManager)   + sizeof(CSpriteManager)
            + sizeof(CAnimationManager) + sizeof(CUIManager)
            + sizeof(CFontManager)      + sizeof(CSoundManager);
		
        void* memoryBlock = malloc(totalSize);
        mInst = new (memoryBlock) CAssetManager((char*)memoryBlock + sizeof(CAssetManager));
    }
    return mInst;
}
```

^a4b1a3

---
### CSceneManager
```cpp
void CSceneManager::ChangeRequest(ETransition transition, ESceneState state)
{
	mPending.transition   = transition;
	mPending.pendingState = state;
}
```

^933d3b

```cpp
void CSceneManager::Update(float deltaTime)
{
	if (mPending.transition != ETransition::NONE)
		ChangeApply();
	
	mScenes.back()->Update(deltaTime);
}
```

^017baa

```cpp
void CSceneManager::ChangeApply()
{
	switch (mPending.transition)
	{
	case ETransition::PUSH:
		PushScene();
		break;
	case ETransition::POP:
		PopScene();
		break;
	case ETransition::SWAP:
		SwapScene();
		break;
	case ETransition::CLEAR:
		ClearScenes();
		break;
	case ETransition::CLEAR_THEN_PUSH:
		ClearThenPushScene();
		break;
	default:
		break;
	}
	
	mPending.transition   = ETransition::NONE;
	mPending.pendingState = ESceneState::NONE;
}
```

^daf2a4

```cpp
void CSceneManager::PushScene()
{
	CScene* newScene = GetSceneFromState(mPending.pendingState);
	newScene->LoadResources();
	
	mScenes.push_back(newScene);
	mScenes.back()->Enter();
}
```

^19f52b

```cpp
void CSceneManager::PopScene()
{
	assert(!mScenes.empty());
	
	CScene* oldScene = mScenes.back();
	if (oldScene->Exit())
	{
		oldScene->UnloadResources();
		SAFE_DELETE(oldScene);
		mScenes.pop_back();
	}
}
```

^d37f71

```cpp
void CSceneManager::SwapScene()
{
	CScene* newScene = GetSceneFromState(mPending.pendingState);
	newScene->LoadResources();
	
	PopScene();
	
	mScenes.push_back(newScene);
	mScenes.back()->Enter();
}
```

^459179

```cpp
void CSceneManager::ClearScenes()
{
	while (!mScenes.empty())
		PopScene();
}
```

^c06fde

```cpp
void CSceneManager::ClearThenPushScene()
{
	CScene* newScene = GetSceneFromState(mPending.pendingState);
	newScene->LoadResources();
	
	ClearScenes();
	
	mScenes.push_back(newScene);
	mScenes.back()->Enter();
}
```

^e38631

```cpp
void CSceneManager::SwapScene()
{
	CScene* newScene = GetSceneFromState(mPending.pendingState);
	newScene->LoadResources();
	
	CScene* oldScene = mScenes.back();
	if (oldScene->Exit())
	{
		oldScene->UnloadResources();
		SAFE_DELETE(oldScene);
		mScenes.pop_back();
	}
	
	mScenes.push_back(newScene);
	mScenes.back()->Enter();
}
```

^71791d

```cpp
class CScene abstract
{
	friend class CSceneManager;
	
protected:
	CScene();
	virtual ~CScene();
	
protected:
	std::vector<std::shared_ptr<class CTexture>> mTextures;
    std::vector<std::shared_ptr<class CFont>> mFonts;
    std::vector<std::shared_ptr<class CSFX>>  mSFXs;
    std::vector<std::shared_ptr<class CBGM>>  mBGMs;
    
protected:
	virtual void LoadResources() = 0;
	
    void UnloadResources()
    {
	    mTextures.clear();
	    mFonts.clear();
	    mSFXs.clear();
	    mBGMs.clear();
    }
}
```

^185ec2

```cpp
class CTextureManager
{
private:
	std::unordered_map<std::string, std::weak_ptr<CTexture>> mTextures;
	
public:
	std::shared_ptr<CTexture> LoadTexture(...);
}
//---------------------------------------------------------------//
class CFontManager
{
private:
	std::unordered_map<std::string, std::weak_ptr<CFont>> mFonts;
	
public:
	std::shared_ptr<CFont> LoadFont(...);
}
//---------------------------------------------------------------//
class CSoundManager
{
private:
	std::unordered_map<std::string, std::weak_ptr<CSFX>> mSFXs;
	std::unordered_map<std::string, std::weak_ptr<CBGM>> mBGMs;
	
public:
	std::shared_ptr<CSFX> LoadSFX(...);
	std::shared_ptr<CBGM> LoadBGM(...);
}
```

^a53e5d

---
### CScene
```cpp
class CScene abstract
{
	friend class CSceneManager;
	
protected:
	CScene();
	virtual ~CScene();
	
protected:
    std::vector<CLayer*> mLayers;
	
protected:
	virtual bool Enter() = 0;
	virtual bool Exit()  = 0;
	
    virtual void Update(float deltaTime);
    virtual void LateUpdate(float deltaTime);
    virtual void Render(SDL_Renderer* renderer);
	
public:
    template <typename T, int initialCapacity = 50>
    T* InstantiateObject(const std::string& name, ELayer::Type type);
};
```

^36e7b4

```cpp
CScene::CScene()
{
    mLayers.resize(ELayer::Type::MAX);
    for (int i = 0; i < ELayer::Type::MAX; i++)
    {
        mLayers[i] = CMemoryPoolManager::GetInst()->Allocate<CLayer>();
    }
}
```

^31e555

```cpp
CScene::~CScene()
{
    for (CLayer* layer : mLayers)
    {
        CMemoryPoolManager::GetInst()->DeallocateButKeepPool<CLayer>(layer);
    }
}
```

^1b2665

```cpp
void CScene::Update(float deltaTime)
{
    for (CLayer* layer : mLayers)
        layer->Update(deltaTime);
    
    if (mSceneCollision)
        mSceneCollision->Update(deltaTime);
    
    if (mCamera)
        mCamera->Update(deltaTime);
    
    if (mSceneUI)
        mSceneUI->Update(deltaTime);
}
```

^0c3634

```cpp
void CScene::LateUpdate(float deltaTime)
{
    for (CLayer* layer : mLayers)
        layer->LateUpdate(deltaTime);
    
    if (mSceneCollision)
	    mSceneCollision->LateUpdate(deltaTime);
	
    if (mSceneUI)
        mSceneUI->LateUpdate(deltaTime);
}
```

^c1f92b

```cpp
void CScene::Render(SDL_Renderer* renderer)
{
    for (CLayer* layer : mLayers)
        layer->Render(renderer);
	
    if (mSceneUI)
        mSceneUI->Render(renderer);
}
```

^8dbf95

```cpp
template <typename T, int initialCapacity = 50>
T* InstantiateObject(const std::string& name, ELayer::Type type)
{
	if (!CMemoryPoolManager::GetInst()->HasPool<T>())
		CMemoryPoolManager::GetInst()->CreatePool<T>(initialCapacity);
	
	T* obj = CMemoryPoolManager::GetInst()->Allocate<T>();
	obj->SetName(name);
	obj->mScene = this;
	obj->mLayer = mLayers[type];
	
	if (!obj->Init())
	{
		CMemoryPoolManager::GetInst()->Deallocate<T>(obj);
		return nullptr;
	}
	
	mLayers[type]->AddObject(obj);
	return obj;
}
```

^144420

---
### CLayer
```cpp
class CLayer
{
	friend class CScene;
	
public:
	CLayer();
	~CLayer();
	
private:
    std::vector<class CObject*> mObjects;
	ESort::Type mSort = ESort::Y;
	
protected:
	void Update(float deltaTime);
	void LateUpdate(float deltaTime);
	void Render(SDL_Renderer* renderer);
	
public:
	void AddObject(CObject* obj) { mObjects.emplace_back(obj); }
	
private:
	static bool SortY(CObject* objA, CObject* objB);
};
```

^70bdb1

```cpp
CLayer::~CLayer()
{
    for (CObject* obj : mObjects)
    {
        obj->Release();
    }
}
```

^65f9d6

```cpp
void CLayer::Update(float deltaTime)
{
    for (CObject* obj : mObjects)
    {
        if (!obj->GetActive())
        {
            obj->Destroy();
            continue;
        }
        else if (!obj->GetEnable())
        {
            continue;
        }
        obj->Update(deltaTime);
    }
}
```

^d0565b

```cpp
void CLayer::LateUpdate(float deltaTime)
{
    for (size_t i = mObjects.size(); i > 0; i--)
    {
        CObject* obj = mObjects[i - 1];
		
        if (!obj->GetActive())
        {
            std::swap(mObjects[i - 1], mObjects.back());
            mObjects.pop_back();
            
            obj->Release();
            continue;
        }
        else if (!obj->GetEnable())
        {
            continue;
        }
        obj->LateUpdate(deltaTime);
    }
}
```

^e4106e

```cpp
void CLayer::Render(SDL_Renderer* renderer)
{
    if (mSort == ESort::Type::Y)
	    std::sort(mObjects.begin(), mObjects.end(), SortY);
	    
    for (CObject* obj : mObjects)
    {
        if (!obj->GetActive() || !obj->GetEnable())
        {
            continue;
        }
        obj->Render(renderer);
    }
}
```

^549045

---
### CObject
```cpp
class CObject abstract : public CEntityBase
{
	friend class CScene;
	friend class CLayer;
	
protected:
	CObject();
	virtual ~CObject();
	
protected:
	CScene* mScene;
	CLayer* mLayer;
	
	CComponent* mRootComponent;
	
protected:
	virtual bool Init();
	virtual void Update(float deltaTime);
	virtual void LateUpdate(float deltaTime);
	virtual void Render(SDL_Renderer* renderer);
	
	virtual void Release() = 0;
	
public:
	CTransform* GetTransform() const { return mRootComponent->GetTransform(); }
	CComponent* GetComponent(const std::string& name = "")
	{
		if (name.empty())
			return mRootComponent;
			
		size_t hashID = std::hash<std::string>()(name);
		return mRootComponent->FindComponent(hashID);
	}
	template <typename T>
	T* GetComponent() const { return mRootComponent->FindComponent<T>(); }
	
	template <typename T, int initialCapacity = 10>
	T* AllocateComponent(const std::string& name);
};
```

^24ed16

```cpp
CObject::CObject() :
	mScene(nullptr),
	mLayer(nullptr)
{
	mRootComponent = new CComponent;
	
	mRootComponent->SetName("RootComponent");
	mRootComponent->mObject = this;
}
```

^b8195f

```cpp
CObject::~CObject()
{
	SAFE_DELETE(mRootComponent);
}
```

^b2dfbe

```cpp
bool CObject::Init()
{
	return mRootComponent->Init();
}
```

^6c4ee4

```cpp

void CObject::Update(float deltaTime)
{
	mRootComponent->Update(deltaTime);
}
```

^321393

```cpp
void CObject::LateUpdate(float deltaTime)
{
	mRootComponent->LateUpdate(deltaTime);
}
```

^2a3d8a

```cpp
void CObject::Render(SDL_Renderer* renderer)
{
	mRootComponent->Render(renderer);
}
```

^fba18c

```cpp
template <typename T, int initialCapacity = 10>
T* AllocateComponent(const std::string& name)
{
	if (!CMemoryPoolManager::GetInst()->HasPool<T>())
		CMemoryPoolManager::GetInst()->CreatePool<T>(initialCapacity);
	
	T* component = CMemoryPoolManager::GetInst()->Allocate<T>();
	component->SetName(name);
	
	return component;
}
```

^ab3424

---
### CComponent
```cpp
class CComponent : public CEntityBase
{
	friend class CObject;
	
protected:
	CComponent();
	virtual ~CComponent();
	
protected:
	size_t mTypeID = -1;
	
	CObject*    mObject    = nullptr;
	CTransform* mTransform = nullptr;
	
	CComponent* mParent = nullptr;
	std::vector<CComponent*> mChilds;
	
protected:
	virtual bool Init();
	virtual void Update(float deltaTime);
	virtual void LateUpdate(float deltaTime);
	virtual void Render(SDL_Renderer* renderer);
	
	virtual void Release();
	
public:
	CObject* GetObject() const { return mObject; }
	CTransform* GetTransform() const { return mTransform; }
	
	void AddChild(CComponent* child);
	bool DeleteChild(CComponent* child);
	
private:
	CComponent* FindRootComponent();
	CComponent* FindComponent(size_t id);
	
	template <typename T>
	T* FindComponent();
};
```

^47760c

```cpp
CComponent::CComponent()
{
	mTransform = CMemoryPoolManager::GetInst()->Allocate<CTransform>();
}
```

^750ae3

```cpp
CComponent::~CComponent()
{
	for (CComponent* child : mChilds)
	{
		child->Release();
	}
	CMemoryPoolManager::GetInst()->DeallocateButKeepPool<CTransform>(mTransform);
}
```

^0e00e0

```cpp
bool CComponent::Init()
{
	for (CComponent* child : mChilds)
	{
		if (!child->Init())
			return false;
	}
	return true;
}
```

^3286fc

```cpp
void CComponent::Update(float deltaTime)
{
	for (CComponent* child : mChilds)
	{
		if (!child->GetActive())
		{
			child->Destroy();
			continue;
		}
		else if (!child->GetEnable())
		{
			continue;
		}
		child->Update(deltaTime);
	}
}
```

^34b1f6

```cpp
void CComponent::LateUpdate(float deltaTime)
{
	for (size_t i = mChilds.size(); i > 0; i--)
	{
		CComponent* child = mChilds[i - 1];
		
		if (!child->GetActive())
		{
			std::swap(mChilds[i - 1], mChilds.back());
			mChilds.pop_back();
			
			std::vector<CTransform*> transChilds = mTransform->GetChilds();
			
			std::swap(mTransform->GetChilds()[i - 1], mTransform->GetChilds().back());
			mTransform->GetChilds().pop_back();
			
			child->Release();
			continue;
		}
		else if (!child->GetEnable())
		{
			continue;
		}
		child->LateUpdate(deltaTime);
	}
}
```

^d23f4e

```cpp
void CComponent::Render(SDL_Renderer* renderer)
{
	for (CComponent* child : mChilds)
	{
		if (!child->GetActive() || !child->GetEnable())
			continue;
		
		child->Render(renderer);
	}
}
```

^7c581f

---

### CAnimation
```cpp
class CAnimation
{
    friend class CAnimationManager;
    friend class CSpriteComponent;
    friend class CVFXComponent;
    
public:
    CAnimation();
    ~CAnimation();
    
protected:
    std::unordered_map<EAnimationState, std::shared_ptr<FAnimationData>> mAnimationStates;
    EAnimationState mCurrentState;
    
    // for EAnimationType::MOVE
    CTransform* mTransform;
    FVector2D mPrevPos;
    
    // for EAnimationType::MOVE & EAnimationType::TIME
    float mFrameInterval;
    int   mCurrIdx;
    bool  mLooped;
    
private:
    void Update(float deltaTime);
    void Release();
    
    CAnimation* Clone() const;
    
public:
    const SDL_Rect& GetFrame();
    bool GetLooped() const;
    void ResetLoop();
    
    void SetState(EAnimationState state);
    
private:
    void AddState(EAnimationState state, std::shared_ptr<FAnimationData> data);
};
```

^d17f0e

```cpp
enum class EAnimationState : unsigned char
{
	NONE,
	
	IDLE,
	WALK,
	JUMP,
	
	VFX
};
```

^3718fb

```cpp
struct FAnimationData
{
	EAnimationType type = EAnimationType::NONE;
	
	bool  isLoop = 0;
	float intervalPerFrame = 0;
	
	std::vector<SDL_Rect> frames;
};
```

^475f27

```cpp
void CAnimation::Update(float deltaTime)
{
	FAnimationData* aniData = mAnimationStates[mCurrentState].get();
	
	switch (aniData->type)
	{
		case EAnimationType::NONE:
			break;
			
		case EAnimationType::MOVE:
		{
			const FVector2D& currentPos = mTransform->GetWorldPos();
			
			FVector2D posDelta = currentPos - mPrevPos;
			
			mFrameInterval += posDelta.Length();
			
			float frameTransitionDistance = aniData->intervalPerFrame / aniData->frames.size();
			
			if (mFrameInterval >= frameTransitionDistance)
			{
				mCurrIdx = (mCurrIdx + 1) % aniData->frames.size();
				
				mFrameInterval -= frameTransitionDistance;
			}
			mPrevPos = currentPos;
		}
		break;
		
		case EAnimationType::TIME:
		{
			mFrameInterval += deltaTime;
			
			if (mFrameInterval >= aniData->intervalPerFrame)
			{
				if (aniData->isLoop)
				{
					mCurrIdx = (mCurrIdx + 1) % aniData->frames.size();
				}
				else
				{
					mLooped = (mCurrIdx >= aniData->frames.size() - 1) ? true : false;
					if (!mLooped)
						mCurrIdx++;
				}
				mFrameInterval = 0.0f;
			}
		}
		break;
	}
}
```

^bfc61d

```cpp
void SetState(EAnimationState state)
{
	if (mCurrentState != state)
	{
		mCurrentState = state;
		mFrameInterval = 0.0f;
		mCurrIdx = 0;
	}
}
```

^947c38

```cpp
CAnimation* CAnimation::Clone() const
{
	CAnimation* clone = CMemoryPoolManager::GetInst()->Allocate<CAnimation>();
	*clone = *this;
	
	return clone;
}
```

^63afad

---

### CSpriteComponent
```cpp
class CSpriteComponent : public CComponent
{
public:
	CSpriteComponent();
	virtual ~CSpriteComponent();
	
private:
	std::shared_ptr<CTexture> mTexture;
	CAnimation* mAnimation;
	SDL_Rect mFrame;
	
	SDL_RendererFlip mFlip;
	
private:
	virtual void Update(float deltaTime)        final;
	virtual void Render(SDL_Renderer* renderer) final;
	virtual void Release()                      final;
	
public:
	std::shared_ptr<CTexture> GetTexture() const { return mTexture; }
	CAnimation* GetAnimation() const { return mAnimation; }
	
	void SetTexture(const std::string& key);
	void SetAnimation(const std::string& key);
	void SetFrame(const std::string& key);
	
	void SetFlip(SDL_RendererFlip flip) { mFlip = flip; }

private:
	const SDL_Rect& GetFrame() const;
	SDL_Rect GetDest() const;
	
	bool IsVisibleToCamera() const;
	SDL_Rect GetCameraSpaceRect() const;
};
```

^aa0928

```cpp
void CSpriteComponent::Update(float deltaTime)
{
	CComponent::Update(deltaTime);
	
	if (mAnimation)
		mAnimation->Update(deltaTime);
}
```

^31ef87

```cpp
void CSpriteComponent::Render(SDL_Renderer* renderer)
{
	if (mTexture && IsVisibleToCamera())
	{
		const SDL_Rect& frame = GetFrame();
		const SDL_Rect  dest  = GetCameraSpaceRect();
		
		SDL_RenderCopyEx(renderer, mTexture->GetTexture(), &frame, &dest, 0.0, nullptr, mFlip);
	}
	CComponent::Render(renderer);
}
```

^5639d3

```cpp
void CSpriteComponent::SetTexture(const std::string& key)
{
	mTexture = CAssetManager::GetInst()->GetTextureManager()->GetTexture(key);
}
```

^f68fb3

```cpp
void CSpriteComponent::SetAnimation(const std::string& key)
{
	CAnimation* base = CAssetManager::GetInst()->GetAnimationManager()->GetAnimation(key);
	
	if (base)
	{
		mAnimation = base->Clone();
		mAnimation->mTransform = mTransform;
	}
}
```

^c0c05f

```cpp
void CSpriteComponent::SetFrame(const std::string& key)
{
	const SDL_Rect* const framePtr = CAssetManager::GetInst()->GetSpriteManager()->GetSpriteFrame(key);
	
	mFrame = *framePtr;
}
```

^bc4e7a

---

### CVFXComponent
```cpp
class CVFXComponent : public CComponent
{
public:
	CVFXComponent();
	virtual ~CVFXComponent();
	
private:
	std::shared_ptr<CTexture> mTexture;
	CAnimation* mAnimation;
	
	bool mPlayVFX;
	
private:
	virtual void Update(float deltaTime)        final;
	virtual void Render(SDL_Renderer* renderer) final;
	virtual void Release()                      final;
	
public:
	std::shared_ptr<CTexture> GetTexture() const { return mTexture; }
	CAnimation* GetAnimation() const { return mAnimation; }
	
	void SetTexture(const std::string& key);
	void SetAnimation(const std::string& key);
	
	void PlayVFX(const FVector2D& pos);
	
private:
	SDL_Rect GetDest() const;
	
	bool IsVisibleToCamera() const;
	SDL_Rect GetCameraSpaceRect() const;
};
```

^17041f

```cpp
void CVFXComponent::Update(float deltaTime)
{
	CComponent::Update(deltaTime);
	
	if (!mPlayVFX || !mAnimation)
		return;
	
	mAnimation->Update(deltaTime);
	
	if (mAnimation->GetLooped())
	{
		mPlayVFX = false;
		mAnimation->ResetLoop();
	}
}
```

^14b509

```cpp
void CVFXComponent::Render(SDL_Renderer* renderer)
{
	if (mPlayVFX && mTexture && IsVisibleToCamera())
	{
		const SDL_Rect& frame = mAnimation->GetFrame();
		const SDL_Rect  dest  = GetCameraSpaceRect();
		
		SDL_RenderCopy(renderer, mTexture->GetTexture(), &frame, &dest);
	}
	CComponent::Render(renderer);
}
```

^71386f

```cpp
void CVFXComponent::SetTexture(const std::string& key)
{
	mTexture = CAssetManager::GetInst()->GetTextureManager()->GetTexture(key);
}
```

^9ffdbb

```cpp
void CVFXComponent::SetAnimation(const std::string& key)
{
	CAnimation* base = CAssetManager::GetInst()->GetAnimationManager()->GetAnimation(key);
	
	if (base)
	{
		mAnimation = base->Clone();
		mAnimation->mTransform = mTransform;
	}
}
```

^788ffb

```cpp
void CVFXComponent::PlayVFX(const FVector2D& pos)
{
	if (mPlayVFX || !mAnimation)
		return;
	
	mPlayVFX = true;
	
	mTransform->SetWorldPos(pos);
	mAnimation->mCurrIdx = 0;
	mAnimation->mFrameInterval = 0.0f;
}
```

^9192ae

---
### CInputUtils
```cpp
enum class EKeyAction : unsigned char
{
	PRESS,
	HOLD,
	RELEASE,
	MAX
};
```

^0b8d0a

```cpp
struct FKeyState
{
	bool Press   = false;
	bool Hold    = false;
	bool Release = false;
};
```

^186449

```cpp
struct FBindFunction
{
	void* obj = nullptr;
	std::function<void()> func;
};
```

^0cf9f9

```cpp
struct FBinder
{
	std::vector<std::pair<SDL_Scancode, EKeyAction>> Keys;
	std::vector<std::pair<Uint8, EKeyAction>> Mouses;
	
	std::vector<FBindFunction*> Functions;
};
```

^1318b8

---

### CInputManager
```cpp
class CInputManager
{
	friend class CGameManager;
	
private:
	CInputManager();
	~CInputManager();
	
private:
	static CInputManager* mInst;
	
	// for keyboard
	std::unordered_map<SDL_Scancode, FKeyState> mKeys;
	
	// for mouse
	std::unordered_map<Uint8, FKeyState> mMouses;
	FVector2D mMousePos = FVector2D::ZERO;
	
private:
	bool Init();
	bool RegisterKey(SDL_Scancode keyCode);
	bool RegisterMouse(Uint8 button);
	
	void Update();
	void UpdateInputState();
	void HandleInputState(bool& press, bool& hold, bool& release, bool isPressed);
	
public:
	bool GetKeyState(SDL_Scancode keyCode, EKeyAction action);
	bool GetMouseButtonState(Uint8 button, EKeyAction action);
	const FVector2D& GetMousePos() const { return mMousePos; }
	
public:
	static CInputManager* GetInst()
	{
		if (!mInst)
			mInst = new CInputManager;
		return mInst;
	}
	
	static void DestroyInst()
	{
		SAFE_DELETE(mInst);
	}
};
```

^1fdcb1

```cpp
bool CInputManager::RegisterKey(SDL_Scancode keyCode)
{
	if (mKeys.find(keyCode) != mKeys.end())
		return false;
	
	FKeyState state;
	mKeys[keyCode] = state;
	
	return true;
}
```

^9a862b

```cpp
bool CInputManager::RegisterMouse(Uint8 button)
{
	if (mMouses.find(button) != mMouses.end())
		return false;
	
	FKeyState state;
	mMouses[button] = state;
	
	return true;
}
```

^991446

```cpp
void CInputManager::UpdateInputState()
{
	// FOR KEYBOARD //
	{
		const Uint8* keyboardState = SDL_GetKeyboardState(nullptr);
		
		std::unordered_map<SDL_Scancode, FKeyState>::iterator iter = mKeys.begin();
		std::unordered_map<SDL_Scancode, FKeyState>::iterator iterEnd = mKeys.end();
		
		for (; iter != iterEnd; iter++)
		{
			bool isPressed = keyboardState[iter->first];
			FKeyState& key = iter->second;
			
			HandleInputState(key.Press, key.Hold, key.Release, isPressed);
		}
	}
	
	// FOR MOUSE //
	{
		int mouseX, mouseY;
		Uint32 mouseState = SDL_GetMouseState(&mouseX, &mouseY);
		
		mMousePos = { (float)mouseX, (float)mouseY };
		
		std::unordered_map<Uint8, FKeyState>::iterator iter = mMouses.begin();
		std::unordered_map<Uint8, FKeyState>::iterator iterEnd = mMouses.end();
		
		for (; iter != iterEnd; iter++)
		{
			bool isPressed = mouseState & SDL_BUTTON(iter->first);
			FKeyState& mouse = iter->second;
			
			HandleInputState(mouse.Press, mouse.Hold, mouse.Release, isPressed);
		}
	}
}
```

^242c9b

```cpp
void CInputManager::HandleInputState(bool& press, bool& hold, bool& release, bool isPressed)
{
	if (isPressed)
	{
		if (!hold)
		{
			press = true;
			hold  = true;
		}
		else
			press = false;
	}
	else
	{
		if (hold)
		{
			press   = false;
			hold    = false;
			release = true;
		}
		else if (release)
			release = false;
	}
}
```

^646ee8

```cpp
bool CInputManager::GetKeyState(SDL_Scancode keyCode, EKeyAction action)
{
	switch (action)
	{
	case EKeyAction::PRESS:
		return mKeys[keyCode].Press;
	case EKeyAction::HOLD:
		return mKeys[keyCode].Hold;
	case EKeyAction::RELEASE:
		return mKeys[keyCode].Release;
	default:
		return false;
	}
}
```

^a61a23

```cpp
bool CInputManager::GetMouseButtonState(Uint8 button, EKeyAction action)
{
	switch (action)
	{
		case EKeyAction::PRESS:
			return mMouses[button].Press;
		case EKeyAction::HOLD:
			return mMouses[button].Hold;
		case EKeyAction::RELEASE:
			return mMouses[button].Release;
		default:
			return false;
	}
}
```

^7fc416

---
### CInputComponent
```cpp
class CInputComponent : public CComponent
{
public:
	CInputComponent();
	virtual ~CInputComponent();
	
private:
	std::unordered_map<std::string, FBinder*> mBinders;
	
private:
	virtual void Update(float deltaTime) final;
	virtual void Release() final;
	
public:
	template <typename T>
	void AddFuncToBinder(const std::string& key, T* obj, void(T::* func)());
	void AddFuncToBinder(const std::string& key, void* obj, const std::function<void()>& func);
	
	void DeleteFuncFromBinder(const std::string& key, void* obj);
	
	void AddInputToBinder(const std::string& key, SDL_Scancode keyCode, EKeyAction action);
	void AddInputToBinder(const std::string& key, Uint8 button, EKeyAction action);
};
```

^a6482f

```cpp
void CInputComponent::Update(float deltaTime)
{
	CComponent::Update(deltaTime);
	
	std::unordered_map<std::string, FBinder*>::iterator iter = mBinders.begin();
	std::unordered_map<std::string, FBinder*>::iterator iterEnd = mBinders.end();
	
	for (; iter != iterEnd; iter++)
	{
		FBinder* binder = iter->second;
		if (binder->Keys.empty() && binder->Mouses.empty())
			continue;
		
		bool match = true;
		
		// KEYBOARD //
		for (const auto& binderKey : binder->Keys)
		{
			const SDL_Scancode& key = binderKey.first;
			const EKeyAction& action = binderKey.second;
			
			if (!CInputManager::GetInst()->GetKeyState(key, action))
			{
				match = false;
				break;
			}
		}
		
		if (!match)
			continue;
		
		// MOUSE //
		for (const auto& binderMouse : binder->Mouses)
		{
			const Uint8& mouse = binderMouse.first;
			const EKeyAction& action = binderMouse.second;
			
			if (!CInputManager::GetInst()->GetMouseButtonState(mouse, action))
			{
				match = false;
				break;
			}
		}
		
		if (match)
		{
			for (FBindFunction* bindFunc : binder->Functions)
				bindFunc->func();
		}
	}
}
```

^196a5f

```cpp
// Bind functions (lambda, std::function)
void CInputComponent::AddFunctionToBinder(const std::string& key, void* obj, const std::function<void()>& func)
{
	FBinder* binder = mBinders[key];
	
	if (!binder)
	{
		binder = CMemoryPoolManager::GetInst()->Allocate<FBinder>();
		mBinders[key] = binder;
	}
	
	FBindFunction* binderFunc = CMemoryPoolManager::GetInst()->Allocate<FBindFunction>();
	
	binderFunc->obj = obj;
	binderFunc->func = func;
	
	binder->Functions.emplace_back(binderFunc);
}

// Bind functions (member)
template <typename T>
void CInputComponent::AddFunctionToBinder<T>(const std::string& key, T* obj, void(T::* func)())
{
	AddFunctionToBinder(key, obj, std::bind(func, obj));
}
```

^22375e

```cpp
void CInputComponent::DeleteFunctionFromBinder(const std::string& key, void* obj)
{
	FBinder* binder = mBinders[key];
	
	if (!binder)
		return;
	
	std::vector<FBindFunction*>& functions = binder->Functions;
	
	for (size_t i = functions.size(); i > 0; i--)
	{
		FBindFunction* bindFunc = functions[i - 1];
		
		if (bindFunc->obj == obj)
		{
			std::swap(functions[i - 1], functions.back());
			functions.pop_back();
			
			CMemoryPoolManager::GetInst()->DeallocateButKeepPool<FBindFunction>(bindFunc);
		}
	}
}
```

^94cde9

```cpp
// keyboard
void AddInputToBinder(const std::string& key, SDL_Scancode keyCode, EKeyAction action)
{
	FBinder* binder = mBinders[key];
	
	if (!binder)
		return;
	
	binder->Keys.emplace_back(std::make_pair(keyCode, action));
}

// mouse
void AddInputToBinder(const std::string& key, Uint8 button, EKeyAction action)
{
	FBinder* binder = mBinders[key];
	
	if (!binder)
		return;
	
	binder->Mouses.emplace_back(std::make_pair(button, action));
}
```

^cc72ca

---
### CCollisionManager
```cpp
class CCollisionManager
{
private:
	CCollisionManager();
	~CCollisionManager();
	
private:
	static CCollisionManager* mInst;
	
	std::unordered_map<std::string, FCollisionProfile*>	mProfileMap;
	
public:
	bool Init();
	
	bool CreateProfile(const std::string& name, 
		ECollision::Channel myChannel, ECollision::Interaction interaction);
	
	bool SetCollisionInteraction(const std::string& name,
		ECollision::Channel otherChannel, ECollision::Interaction interaction);
	
	FCollisionProfile* FindProfile(const std::string& name);
	
public:
	bool AABBCollision(CBoxCollider* collider1, CBoxCollider* collider2);
	bool CircleCircleCollision(CCircleCollider* collider1, CCircleCollider* collider2);
	bool AABBCircleCollision(CBoxCollider* collider1, CCircleCollider* collider2);
	bool AABBPointCollision(CBoxCollider* collider, const FVector2D& point);
	bool CirclePointCollision(CCircleCollider* collider, const FVector2D& point);
	
	bool AABBPointCollision(const SDL_Rect& rect, const FVector2D& point);
	
public:
	static CCollisionManager* GetInst();
	static void DestroyInst();
};
```

^f68f25

```cpp
bool CCollisionManager::AABBCollision(CBoxCollider* collider1, CBoxCollider* collider2)
{
	const SDL_FRect& box1 = collider1->GetRect();
	const SDL_FRect& box2 = collider2->GetRect();
	
	if (box1.x + box1.w < box2.x ||
		box1.x > box2.x + box2.w ||
		box1.y + box1.h < box2.y ||
		box1.y > box2.y + box2.h)
	{
		return false;
	}
	
	FVector2D hitPoint;
	hitPoint.x = (std::max(box1.x, box2.x) + std::min(box1.x + box1.w, box2.x + box2.w)) * 0.5f;
	hitPoint.y = (std::max(box1.y, box2.y) + std::min(box1.y + box1.h, box2.y + box2.h)) * 0.5f;
	
	collider1->mHitPoint = hitPoint;
	collider2->mHitPoint = hitPoint;
	
	return true;
}
```

^fb1690

```cpp
bool CCollisionManager::CircleCircleCollision(CCircleCollider* collider1, CCircleCollider* collider2)
{
	const FCircle& circle1 = collider1->GetCircle();
	const FCircle& circle2 = collider2->GetCircle();
	
	float distance  = circle1.center.Distance(circle2.center);
	float sumRadius = circle1.radius + circle2.radius;
	
	if (distance > sumRadius)
	{
		return false;
	}
	
	FVector2D hitPoint = (circle1.center + circle2.center) * 0.5f;
	
	collider1->mHitPoint = hitPoint;
	collider2->mHitPoint = hitPoint;
	
	return true;
}
```

^0855e2

```cpp
bool CCollisionManager::AABBCircleCollision(CBoxCollider* collider1, CCircleCollider* collider2)
{
	const SDL_FRect& box  = collider1->GetRect();
	const FCircle& circle = collider2->GetCircle();
	
	FVector2D closestPointOnBox = circle.center.Clamp(box.x, box.x + box.w, box.y + box.h, box.y);
	
	float distance = circle.center.Distance(closestPointOnBox);
	
	if (circle.radius < distance)
	{
		return false;
	}
	
	FVector2D hitPoint = closestPointOnBox;
	
	collider1->mHitPoint = hitPoint;
	collider2->mHitPoint = hitPoint;
	
	return true;
}
```

^c34b99

---
### CSceneCollision
```cpp
class CSceneCollision
{
	friend class CScene;
	
public:
	CSceneCollision() = delete;
	CSceneCollision(CCamera* camera);
	~CSceneCollision();
	
private:
	CQuadTree* mQuadTree;
	std::unordered_map<FColliderPair, EPair::Status> mPairs;
	
public:
	void Update(float deltaTime);
	void LateUpdate(float deltaTime);
	
public:
	void AddCollider(CCollider* collider);
	void HandleCollision(CCollider* collider1, CCollider* collider2);
	
private:
	void CleanPairs();
};
```

^9b935a

```cpp
void CSceneCollision::HandleCollision(CCollider* collider1, CCollider* collider2)
{
	FColliderPair  pair = { collider1, collider2 };
	EPair::Status& status = mPairs[pair];
	
	if (status == EPair::DNE)
		status = EPair::NOT_COLLIDED;
	
	if (collider1->Intersect(collider2))
	{
		if (status == EPair::NOT_COLLIDED)
		{
			collider1->OnCollisionEnter(collider2);
			collider2->OnCollisionEnter(collider1);
			
			status = EPair::COLLIDED;
		}
		else
		{
			collider1->OnCollisionStay(collider2);
			collider2->OnCollisionStay(collider1);
			
			CPhysicsManager::GetInst()->ResolveOverlapIfPushable(collider1, collider2);
		}
	}
	else
	{
		if (status == EPair::COLLIDED)
		{
			collider1->OnCollisionExit(collider2);
			collider2->OnCollisionExit(collider1);
		}
		mPairs.erase(pair);
	}
}
```

^d34e98

```cpp
void CSceneCollision::CleanPairs()
{
	std::unordered_map<FColliderPair, EPair::Status>::iterator iter = mPairs.begin();
	std::unordered_map<FColliderPair, EPair::Status>::iterator iterEnd = mPairs.end();
	
	while (iter != iterEnd)
	{
		CCollider* collider1 = iter->first.collider1;
		CCollider* collider2 = iter->first.collider2;
		const EPair::Status& status = iter->second;
		
		if (!collider1->GetActive() || !collider2->GetActive())
		{
			if (status == EPair::COLLIDED)
			{
				collider1->OnCollisionExit(collider2);
				collider2->OnCollisionExit(collider1);
			}
			iter = mPairs.erase(iter);
			continue;
		}
		iter++;
	}
}
```

^fa3f6c

---
### CQuadTree
```cpp
class CQuadTree
{
	friend class CSceneCollision;
	
private:
	CQuadTree() = delete;
	CQuadTree(CCamera* camera);
	~CQuadTree();
	
private:
	CQTNode* mRoot = nullptr;
	std::vector<CCollider*> mColliders;
	
public:
	void Update(float deltaTime);
	void LateUpdate(float deltaTime);
	
public:
	void AddCollider(CCollider* collider);
	
private:
	void UpdateBoundary();
};
```

^3cceff

```cpp
CQuadTree::CQuadTree(CCamera* camera) :
	mRoot(nullptr)
{
	int totalNodes = (int)((pow(4, MAX_SPLIT + 1) - 1) / 3);
	CMemoryPoolManager::GetInst()->CreatePool<CQTNode>(totalNodes);
	
	if (!mRoot)
		mRoot = CMemoryPoolManager::GetInst()->Allocate<CQTNode>();
	
	mRoot->mCamera = camera;
}
```

^033b8a

```cpp
void CQuadTree::Update(float deltaTime)
{
	UpdateBoundary();
	
	for (size_t i = mColliders.size(); i > 0; i--)
	{
		CCollider* collider = mColliders[i - 1];
		
		if (!collider->GetActive())
		{
			std::swap(mColliders[i - 1], mColliders.back());
			mColliders.pop_back();
			
			continue;
		}
		else if (!collider->GetEnable())
		{
			continue;
		}
		mRoot->AddCollider(collider);
	}
	mRoot->Update(deltaTime);
}
```

^860dd5

---
### CQTNode
```cpp
class CQTNode
{
	friend class CQuadTree;
	
public:
	CQTNode();
	~CQTNode();
	
private:
	CCamera* mCamera;
	
	CQTNode* mParent;
	CQTNode* mChilds[4];
	std::vector<CCollider*> mColliders;
	
	SDL_FRect mBoundary;
	
	int mSplitLevel = 0;
	
public:
	void Update(float deltaTime);
	void Render(SDL_Renderer* renderer);
	
public:
	bool HasChild();
	bool ShouldSplit();
	void Split();
	
	bool IsWithinNode(CCollider* collider);
	void AddCollider(CCollider* collider);
	
	void MoveCollidersToChildren();
	
	void Clear();
};
```

^3f6061

```cpp
void CQTNode::Update(float deltaTime)
{
	if (HasChild())
	{
		for (int i = 0; i < 4; i++)
		{
			mChilds[i]->Update(deltaTime);
		}
	}
	else
	{
		size_t size = mColliders.size();
		if (size >= 2)
		{
			CSceneCollision* sceneCollision = CSceneManager::GetInst()->GetCurrentScene()->GetCollision();
			
			for (size_t i = 0; i < size; i++)
			{
				for (size_t j = i + 1; j < size; j++)
				{
					FCollisionProfile* profile1 = mColliders[i]->GetProfile();
					FCollisionProfile* profile2 = mColliders[j]->GetProfile();
					
					if (profile1->collisionResponses[profile2->channel] == ECollision::Interaction::IGNORE ||
						profile2->collisionResponses[profile1->channel] == ECollision::Interaction::IGNORE)
					{
						continue;
					}
					sceneCollision->HandleCollision(mColliders[i], mColliders[j]);
				}
			}
		}
	}
}
```

^03c03d

```cpp
void CQTNode::AddCollider(CCollider* collider)
{
	if (!IsWithinNode(collider))
		return;
		
	if (HasChild())
	{
		for (int i = 0; i < 4; i++)
		{
			if (mChilds[i]->IsWithinNode(collider))
			{
				mChilds[i]->AddCollider(collider);
			}
		}
	}
	else
	{
		mColliders.emplace_back(collider);
		
		if (ShouldSplit())
		{
			Split();
			
			MoveCollidersToChildren();
		}
	}
}
```

^b0d555

---
### CPhysicsManager
```cpp
class CPhysicsManager
{
	friend class CScene;
	
private:
	CPhysicsManager();
	~CPhysicsManager();
	
private:
	static CPhysicsManager* mInst;
	
	const float CONST_MinMtvLSq = 0.5f;
	
public:
	void ResolveOverlapIfPushable(CCollider* collider1, CCollider* collider2);
	
private:
	void ResolveOverlap(CCollider* collider1, CCollider* collider2, bool pushObj1, bool pushObj2);
	
	void ResolveAABBOverlap(CBoxCollider* collider1, CBoxCollider* collider2, bool pushObj1, bool pushObj2);
	void ResolveCircleCircleOverlap(CCircleCollider* collider1, CCircleCollider* collider2, bool pushObj1, bool pushObj2);
	void ResolveAABBCircleOverlap(CBoxCollider* collider1, CCircleCollider* collider2, bool pushObj1, bool pushObj2);
	
	void MoveBy(CCollider* collider, const FVector2D& mtv);
	
public:
	static CPhysicsManager* GetInst();
	static void DestroyInst();
};
```

^6ed2ab

```cpp
void CPhysicsManager::ResolveOverlapIfPushable(CCollider* collider1, CCollider* collider2)
{
	FCollisionProfile* profile1 = collider1->GetProfile();
	FCollisionProfile* profile2 = collider2->GetProfile();
	
	ECollision::Interaction response1to2 = profile1->collisionResponses[profile2->channel];
	ECollision::Interaction response2to1 = profile2->collisionResponses[profile1->channel];
	
	CRigidbody* rb1 = nullptr;
	CRigidbody* rb2 = nullptr;
	
	if (response1to2 == ECollision::Interaction::BLOCK)
		rb1 = collider1->GetObject()->GetComponent<CRigidbody>();
	
	if (response2to1 == ECollision::Interaction::BLOCK)
		rb2 = collider2->GetObject()->GetComponent<CRigidbody>();
	
	bool pushObj1 = rb1 != nullptr && rb1->GetType() == ERigidbodyType::DYNAMIC;
	bool pushObj2 = rb2 != nullptr && rb2->GetType() == ERigidbodyType::DYNAMIC;
	
	if (!pushObj1 && !pushObj2)
		return;
	
	ResolveOverlap(collider1, collider2, pushObj1, pushObj2);
}
```

^fba119

---
### CRigidbody
```cpp
class CRigidbody : public CComponent
{
public:
	CRigidbody();
	virtual ~CRigidbody();
	
private:
	ERigidbodyType mType;
	
	float mMass;
	float mGravityScale;
	
	FVector2D mVelocity;
	FVector2D mAcceleration;
	FVector2D mAccumulatedForce;
	
private:
	virtual void Update(float deltaTime) final;
	virtual void Release() final;
	
public:
	void AddForce(const FVector2D& force);
	void AddImpulse(const FVector2D& impulse);
	
	const FVector2D& GetVelocity() const { return mVelocity; }
	ERigidbodyType GetType() const { return mType; }
	float GetMass() const { return mMass; }
	
	void SetVelocity(const FVector2D& velocity)
	{
		mVelocity = velocity;
	}
	void SetType(ERigidbodyType type)
	{
		mType = type;
	}
	void SetMass(float mass)
	{ 
		mMass = mass;
	}
	
private:
	void ApplyGravity();
	void ApplyForces();
	void ApplyAcceleration(float deltaTime);
	void ApplyDrag(float deltaTime);
	void UpdateObjectPos(float deltaTime);
	void ClearForces();
};
```

^77c587

```cpp
void CRigidbody::Update(float deltaTime)
{
	CComponent::Update(deltaTime);
	
	if (mType == ERigidbodyType::STATIC || mMass <= 0.0f)
		return;
	
	ApplyGravity();
	
	ApplyForces();
	
	ApplyAcceleration(deltaTime);
	
	ApplyDrag(deltaTime);
	
	UpdateObjectPos(deltaTime);
	
	ClearForces();
}
```

^a4c1df

---
### CSceneUI
```cpp
class CSceneUI
{
public:
	CSceneUI();
	virtual ~CSceneUI();
	
private:
	std::vector<CWidget*> mWidgets;
	
	CWidget* mCurrHovered = nullptr;
	CWidget* mHeldWidget  = nullptr;
	
public:
	virtual bool Init();
	virtual void Update(float deltaTime);
	virtual void LateUpdate(float deltaTime);
	virtual void Render(SDL_Renderer* renderer);
	
public:
	CWidget* FindWidget(size_t id);
	void BringWidgetToTop(CWidget* widget);
	CWidget* GetHoveredWidget() const { return mCurrHovered; }
	CWidget* GetHeldWidget() const { return mHeldWidget; }
	void SetHeldWidget(CWidget* heldWidget)
	{
		mHeldWidget = heldWidget;
	}

protected:
	void AddWidget(CWidget* widget);
	
private:
	void SetSceneUI(CWidget* widget);
	CWidget* FindHoveredWidget(const FVector2D& mousePos);
	CWidget* FindHoveredInTree(CWidget* widget, const FVector2D& mousePos);
	void UpdateInput();
};
```

^e06264

```cpp
CWidget* CSceneUI::FindHoveredWidget(const FVector2D& mousePos)
{
	for (size_t i = mWidgets.size(); i > 0; --i)
	{
		CWidget* hovered = FindHoveredInTree(mWidgets[i - 1], mousePos);

		if (hovered)
			return hovered;
	}
	return nullptr;
}
```

^7bf0af

```cpp
void CSceneUI::UpdateInput()
{
	const FVector2D& mousePos = CInputManager::GetInst()->GetMousePos();
	
	bool isPressed  = CInputManager::GetInst()->GetMouseButtonState(SDL_BUTTON_LEFT, EKeyAction::PRESS);
	bool isHeld     = CInputManager::GetInst()->GetMouseButtonState(SDL_BUTTON_LEFT, EKeyAction::HOLD);
	bool isReleased = CInputManager::GetInst()->GetMouseButtonState(SDL_BUTTON_LEFT, EKeyAction::RELEASE);
	
	if (mHeldWidget)
	{
		if (!CCollisionManager::GetInst()->AABBPointCollision(mHeldWidget->GetRect(), mousePos))
		{
			if (isHeld || isReleased)
			{
				mHeldWidget->HandleUnhovered(mousePos, isHeld, isReleased);
				return;
			}
		}
	}
	
	CWidget* newHovered = FindHoveredWidget(mousePos);
	{
		if (mCurrHovered != newHovered)
		{
			if (mCurrHovered)
				mCurrHovered->HandleUnhovered(mousePos, isHeld, isReleased);
			
			mCurrHovered = newHovered;
		}
		if (mCurrHovered)
			mCurrHovered->HandleHovered(mousePos, isPressed, isHeld, isReleased);
	}
}
```

^5cc348

---
### CWidget
```cpp
class CWidget abstract : public CWidgetBase
{
	friend class CSceneUI;
	friend class CWidgetComponent;
	
protected:
	CWidget();
	virtual ~CWidget();
	
protected:
	CSceneUI* mSceneUI = nullptr;
	
	CWidget* mParent = nullptr;
	std::vector<CWidget*> mChilds;
	
	bool mIsInteractable = false;
	bool mWidgetHovered  = false;
	bool mWidgetHeld     = false;
	
protected:
	virtual void Update(float deltaTime);
	virtual void LateUpdate(float deltaTime);
	virtual void Render(SDL_Renderer* renderer, const FVector2D& topLeft = FVector2D::ZERO);
	virtual void Release() = 0;
	
	virtual void HandleHovered(const FVector2D& mousePos, bool isPressed, bool isHeld, bool isReleased);
	virtual void HandleUnhovered(const FVector2D& mousePos, bool isHeld, bool isReleased);
	
public:	
	CWidget* FindRootWidget();
	CWidget* FindWidget(size_t id);
	
	void AddChild(CWidget* child);
	bool DeleteChild(CWidget* child);
	
	CSceneUI* GetOwnerSceneUI() const { return mSceneUI; }
};
```

^50ddf5

---
### CUserWidget
```cpp
class CUserWidget abstract : public CWidget
{
public:
	CUserWidget();
	virtual ~CUserWidget();
	
private:
	bool mIsMovable = false;
	FVector2D mDragOffset = FVector2D::ZERO;
	
protected:
	virtual void Construct() = 0;
	virtual void Release()   = 0;
	
public:
	void SetInteractable(bool interactable)
	{
		mIsInteractable = interactable;
		mIsMovable &= mIsInteractable;
	}
	void SetMovable(bool movable)
	{
		mIsMovable = movable;
		mIsInteractable |= movable;
	}
	
protected:
	void HandleDragging(const FVector2D& mousePos, bool isPressed, bool isHeld, bool isReleased);
};
```

^93523d

```cpp
void CUserWidget::HandleDragging(const FVector2D& mousePos, bool isPressed, bool isHeld, bool isReleased)
{
    if (!mIsInteractable || !mIsMovable)
        return;
		
    if (isPressed)
    {
        mDragOffset = mousePos - GetTransform()->GetWorldPos();
        mSceneUI->BringWidgetToTop(this);
    }
    else if (isHeld && mDragOffset != FVector2D::ZERO)
    {
        FVector2D newPos = mousePos - mDragOffset;
        GetTransform()->SetWorldPos(newPos);
    }
    else if (isReleased)
        mDragOffset = FVector2D::ZERO;
}
```

^3df818
