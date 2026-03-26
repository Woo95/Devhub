# Code Block References
---

### PlayerBehaviour Class - Key Functions
```csharp
void Start()
{
	m_camera = Camera.main;
	
	m_PlayerSprite = GetComponent<SpriteRenderer>();
	if (m_PlayerSprite != null)
	{
		SetPlayerBoundaryCameraPerspective();
		SetPlayerPositionCameraPerspective();
	}
}
```

^1f0edb

```csharp
void SetPlayerBoundaryCameraPerspective()
{
	float playerHalfSizeY = m_PlayerSprite.bounds.size.y * 0.5f;
	float cameraPosY = m_camera.transform.position.y;
	
	m_boundary.min = cameraPosY - m_camera.orthographicSize + playerHalfSizeY;
	m_boundary.max = cameraPosY + m_camera.orthographicSize - playerHalfSizeY;
}
```

^1db78b

```csharp
void SetPlayerPositionCameraPerspective()
{
	float playerSizeX = m_PlayerSprite.bounds.size.x;
	float cameraPosX = m_camera.transform.position.x;
	float startX = cameraPosX - m_camera.orthographicSize * m_camera.aspect + playerSizeX;
	float startY = (m_boundary.min + m_boundary.max) * 0.5f;

	transform.position = new Vector3(startX, startY, 0f);
	m_fixedPlayerPosX = startX;
}
```

^58ca00

```csharp
void ClampPlayerYPosition()
{
    float clampedY = Mathf.Clamp(transform.position.y, m_boundary.min, m_boundary.max);
    
    transform.position = new Vector2(m_fixedPlayerPosX, clampedY);
}
```

^f510cd
