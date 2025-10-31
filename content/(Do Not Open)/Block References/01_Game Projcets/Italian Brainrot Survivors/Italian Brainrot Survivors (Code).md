# Code Block References
---
## Eng & Kor

### DataLoader
```cpp
void CDataLoader::LoadAllMobData()
{
	std::string filePath = CPathManager::GetInst()->FindPath(DATA_PATH);
	filePath += "GameData\\Mob_Data.csv";
	
	std::ifstream file(filePath);
	if (!file.is_open())
	{
		std::cerr << "Cannot open file at: " << filePath << "\n";
		return;
	}
	
	std::string line;
	while (std::getline(file, line))
	{
		if (line[0] == '#')
			continue;
			
		std::vector<std::string> row = Split(line, ',');
		
		FRegularMobData data;
		data.type = static_cast<ERegularMobType>(std::stoi(row[0]));
		data.baseHP     = std::stof(row[1]);
		data.baseAttack = std::stof(row[2]);
		data.baseSpeed  = std::stof(row[3]);
		data.baseExp    = std::stof(row[4]);
		
		CGameDataManager::GetInst()->GetMobDataManager()->AddMobData(data);
		
		row.clear();
	}
	file.close();
}
```

^4357bd

### ScrollMapComponent
#### Kor
```cpp
void CScrollMapComponent::Render(SDL_Renderer* renderer)
{
    SDL_Rect viewRect = mCamera->GetViewRect();
    viewRect.x = (int)roundf(viewRect.x);
    viewRect.y = (int)roundf(viewRect.y);
	
    const SDL_Rect& texRect = mTexture->GetTextureFrame();
    const FVector2D& mapScale = mTransform->GetWorldScale();
	
    // 텍스처 내에서 카메라 오프셋 계산
    int offsetX = viewRect.x % (int)mapScale.x;
    int offsetY = viewRect.y % (int)mapScale.y;
    if (offsetX < 0) offsetX += (int)mapScale.x;
    if (offsetY < 0) offsetY += (int)mapScale.y;
	
    // 텍스처 크기를 월드 스케일에 맞춰 비율로 계산
    float texW = texRect.w / mapScale.x;
    float texH = texRect.h / mapScale.y;
	
    // 카메라 뷰가 텍스처 경계를 넘어가는지 확인
    bool overX = offsetX + viewRect.w > (int)mapScale.x;
    bool overY = offsetY + viewRect.h > (int)mapScale.y;
	
    // 내부 영역과 넘침 영역 계산
    int innerW = overX ? (int)mapScale.x - offsetX : viewRect.w;
    int innerH = overY ? (int)mapScale.y - offsetY : viewRect.h;
    int outerW = viewRect.w - innerW;
    int outerH = viewRect.h - innerH;
	
    SDL_Rect src, dst;
    
    // 좌상단
    src.x = (int)roundf(offsetX * texW);
    src.y = (int)roundf(offsetY * texH);
    src.w = (int)roundf(innerW * texW);
    src.h = (int)roundf(innerH * texH);
    dst = { 0, 0, innerW, innerH };
    SDL_RenderCopy(renderer, mTexture->GetTexture(), &src, &dst);
    // 우상단
    if (outerW > 0)
    {
        src.x = 0;
        src.y = (int)roundf(offsetY * texH);
        src.w = (int)roundf(outerW * texW);
        src.h = (int)roundf(innerH * texH);
        dst = { innerW, 0, outerW, innerH };
        SDL_RenderCopy(renderer, mTexture->GetTexture(), &src, &dst);
    }
    // 좌하단
    if (outerH > 0)
    {
        src.x = (int)roundf(offsetX * texW);
        src.y = 0;
        src.w = (int)roundf(innerW * texW);
        src.h = (int)roundf(outerH * texH);
        dst = { 0, innerH, innerW, outerH };
        SDL_RenderCopy(renderer, mTexture->GetTexture(), &src, &dst);
    }
    // 우하단
    if (outerW > 0 && outerH > 0)
    {
        src.x = 0;
        src.y = 0;
        src.w = (int)roundf(outerW * texW);
        src.h = (int)roundf(outerH * texH);
        dst = { innerW, innerH, outerW, outerH };
        SDL_RenderCopy(renderer, mTexture->GetTexture(), &src, &dst);
    }
    
    CComponent::Render(renderer);
}
```

^207b8a

#### Eng
```cpp
void CScrollMapComponent::Render(SDL_Renderer* renderer)
{
	SDL_Rect viewRect = mCamera->GetViewRect();
	viewRect.x = (int)roundf(viewRect.x);
	viewRect.y = (int)roundf(viewRect.y);
	
	const SDL_Rect& texRect = mTexture->GetTextureFrame();
	const FVector2D& mapScale = mTransform->GetWorldScale();
	
	// Camera offset within the texture
	int offsetX = viewRect.x % (int)mapScale.x;
	int offsetY = viewRect.y % (int)mapScale.y;
	if (offsetX < 0) offsetX += (int)mapScale.x;
	if (offsetY < 0) offsetY += (int)mapScale.y;
	
	// Texture coordinate ratio relative to world scale
	float texW = texRect.w / mapScale.x;
	float texH = texRect.h / mapScale.y;
	
	// Check if the camera view exceeds the texture boundary
	bool overX = offsetX + viewRect.w > (int)mapScale.x;
	bool overY = offsetY + viewRect.h > (int)mapScale.y;
	
	// Calculate inner and overflow regions
	int innerW = overX ? (int)mapScale.x - offsetX : viewRect.w;
	int innerH = overY ? (int)mapScale.y - offsetY : viewRect.h;
	int outerW = viewRect.w - innerW;
	int outerH = viewRect.h - innerH;
	
	SDL_Rect src, dst;
	
	// Top-left 
	src.x = (int)roundf(offsetX * texW);
	src.y = (int)roundf(offsetY * texH);
	src.w = (int)roundf(innerW * texW);
	src.h = (int)roundf(innerH * texH);
	dst = { 0, 0, innerW, innerH };
	SDL_RenderCopy(renderer, mTexture->GetTexture(), &src, &dst);
	// Top-right
	if (outerW > 0)
	{
		src.x = 0;
		src.y = (int)roundf(offsetY * texH);
		src.w = (int)roundf(outerW * texW);
		src.h = (int)roundf(innerH * texH);
		dst = { innerW, 0, outerW, innerH };
		SDL_RenderCopy(renderer, mTexture->GetTexture(), &src, &dst);
	}
	// Bottom-left
	if (outerH > 0)
	{
		src.x = (int)roundf(offsetX * texW);
		src.y = 0;
		src.w = (int)roundf(innerW * texW);
		src.h = (int)roundf(outerH * texH);
		dst = { 0, innerH, innerW, outerH };
		SDL_RenderCopy(renderer, mTexture->GetTexture(), &src, &dst);
	}
	// Bottom-right
	if (outerW > 0 && outerH > 0)
	{
		src.x = 0;
		src.y = 0;
		src.w = (int)roundf(outerW * texW);
		src.h = (int)roundf(outerH * texH);
		dst = { innerW, innerH, outerW, outerH };
		SDL_RenderCopy(renderer, mTexture->GetTexture(), &src, &dst);
	}
	
	CComponent::Render(renderer);
}
```

^de19b5

### ScrollEnvObj
```cpp
void CScrollEnvObj::Update(float deltaTime)
{
    CObject::Update(deltaTime);
	
    const FVector2D& cameraPos = mCamera->GetLookAt();
    const FVector2D& objPos = GetTransform()->GetWorldPos();
    FVector2D delta = cameraPos - objPos;
	
    float snapDistance = delta.Length();
    if (snapDistance >= mSnapThreshold)
    {
        int directionX = (int)roundf(delta.x / mMapScale.x);
        int directionY = (int)roundf(delta.y / mMapScale.y);
		
        float offsetX = mMapScale.x * directionX;
        float offsetY = mMapScale.y * directionY;
		
        GetTransform()->SetWorldPos(objPos + FVector2D(offsetX, offsetY));
    }
}
```

^7dbaf2

### MobSpawner
```cpp
void CMobSpawner::Update(float deltaTime)
{
	if (!mScene->GetPlayer())
		return;

	mRegularSpawnTime -= deltaTime;
	mSubBossSpawnTime -= deltaTime;

	SpawnMob();
	RespawnMob();
}
```

^d971b1

```cpp
void CMobSpawner::SpawnMob()
{
	// SPAWN REGULAR MOB
	if (mRegularSpawnTime <= 0.0f)
	{
		mRegularSpawnTime = CONST_REGULAR_MOB_SPAWN_INTERVAL;
		CEnemy* mob = SpawnRegularMob(mUnlockedRegIdx);
		mob->GetTransform()->SetWorldPos(GetRandomSpawnPos(1.1f));

		// INDEX CONTROL
		mRegSpawnAmount--;
		if (mRegSpawnAmount <= 0)
		{
			mRegSpawnAmount = CONST_REGULAR_MOB_SPAWN_AMOUNT;
			mUnlockedRegIdx = std::min(mUnlockedRegIdx + 1, (int)ERegularMobType::MAX - 1);
		}
	}

	// SPAWN SUB BOSS
	if (mSubBossSpawnTime <= 0.0f)
	{
		mSubBossSpawnTime = CONST_SUBBOSS_MOB_SPAWN_INTERVAL;
		CEnemy* mob = SpawnSubBossMob(mUnlockedBosIdx);
		mob->GetTransform()->SetWorldPos(GetRandomSpawnPos(1.1f));

		// INDEX CONTROL
		mUnlockedBosIdx = std::min(mUnlockedBosIdx + 1, (int)ESubBossMobType::MAX - 1);
	}
}
```

^0425f5

```cpp
void CMobSpawner::RespawnMob()
{
	for (size_t i = mSpawnedMobs.size(); i > 0; i--)
	{
		CEnemy* mob = mSpawnedMobs[i - 1];
		
		if (!mob->GetActive())
		{
			std::swap(mSpawnedMobs[i - 1], mSpawnedMobs.back());
			mSpawnedMobs.pop_back();
			
			continue;
		}
		
		FVector2D delta = mob->GetTransform()->GetWorldPos() - mScene->GetPlayer()->GetTransform()->GetWorldPos();
		if (delta.Length() >= mDespawnThreshold)
		{
			mob->GetTransform()->SetWorldPos(GetRandomSpawnPos(1.1f));
		}
	}
}
```

^791190

```cpp
FVector2D CMobSpawner::GetRandomSpawnPos(float scale) const
{
	const FVector2D& playerPos = mScene->GetPlayer()->GetTransform()->GetWorldPos();

	float halfW = mScene->GetCamera()->GetResolution().x * 0.5f;
	float halfH = mScene->GetCamera()->GetResolution().y * 0.5f;
	float scaledHalfW = halfW * scale;
	float scaledHalfH = halfH * scale;

	FVector2D pos = FVector2D::ZERO;
	switch (std::rand() % 4)
	{
	case 0: // Left
		pos.x = GetRandomRange(playerPos.x - scaledHalfW, playerPos.x - halfW);
		pos.y = GetRandomRange(playerPos.y + scaledHalfH, playerPos.y - halfH);
		break;
	case 1: // Right
		pos.x = GetRandomRange(playerPos.x + halfW, playerPos.x + scaledHalfW);
		pos.y = GetRandomRange(playerPos.y + halfH, playerPos.y - scaledHalfH);
		break;
	case 2: // Up
		pos.x = GetRandomRange(playerPos.x - scaledHalfW, playerPos.x + halfW);
		pos.y = GetRandomRange(playerPos.y - halfH, playerPos.y - scaledHalfH);
		break;
	case 3: // Down
		pos.x = GetRandomRange(playerPos.x - halfW, playerPos.x + scaledHalfW);
		pos.y = GetRandomRange(playerPos.y + scaledHalfH, playerPos.y + halfH);
		break;
	}
	return pos;
}
```

^b7fa85

### WeaponComponent
```cpp
void CPistolComponent::Shoot()
{
	CSoundManager::GetInst()->GetSound<CSFX>("SFX_Bullet")->Play();
	
	CBullet* bullet = InstantiateObject<CBullet>("Bullet");
	bullet->SetDamage(mOwner->GetAttack() + mPistolAttack);
}
```

^1fbde8

### Player
```cpp
float CPlayer::GetAttack() const
{
	int mightLevel = mStatus->GetMenuPowerUpLvl(EPowerUpType::MIGHT)
		+ mInventory->GetPowerUpLevel(EPowerUpType::MIGHT);
	float mightBonus = mStatus->GetStatModifier(EPowerUpType::MIGHT);
	float mightAttack = mStatus->GetBaseAttack() * mightLevel * mightBonus;
	
	return mStatus->GetBaseAttack() + mightAttack;
}
```

^e8745e