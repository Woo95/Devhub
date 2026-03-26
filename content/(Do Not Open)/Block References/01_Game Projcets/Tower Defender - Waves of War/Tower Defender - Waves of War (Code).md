# Code Block References
---

### Spawner
```csharp
void GenerateObjects(int amount)
{
	for (int i = 0; i < amount; i++)
	{
		int prefabIdx = Random.Range(0, pickUpPrefabList.Count);
		PickUp prefab = pickUpPrefabList[prefabIdx];
		CapsuleCollider2D collider = 
			prefab.GetComponent<CapsuleCollider2D>();

		for (int attempt = 0; attempt < 500; attempt++)
		{
			Vector2 spawnPos =
				new(Random.Range(p00.x, p11.x), Random.Range(p00.y, p11.y));

			Collider2D[] overlapList = Physics2D.OverlapBoxAll(spawnPos, collider.size, 0, blockLayer);
			if (overlapList.Length == 0)
			{
				PickUp clonedObj = Instantiate(prefab, spawnPos, Quaternion.identity, transform);
				pickUpSpawnedList.Add(clonedObj);
				clonedObj.Init();

				break;
			}
		}
	}
}
```

^895f15

```csharp
void SpawnEnemy()
{
	if (enemyPrefabList.Count > 0)
	{
		int prefabIdx = Random.Range(0, enemyPrefabList.Count);

		Enemy clonedObj = Instantiate(enemyPrefabList[prefabIdx], RandomSpawnPos(), Quaternion.identity, transform);
		enemySpawnedList.Add(clonedObj);
		clonedObj.Init();
	}
}

Vector3 RandomSpawnPos()
{
	Vector3 spawnPosition = Vector3.zero;
	int randomSide = Random.Range(0, 4);
	float interval = Random.Range(0f, 1f);
	switch (randomSide)
	{
		case 0: spawnPosition = Vector3.Lerp(p00, p10, interval); break;
		case 1: spawnPosition = Vector3.Lerp(p01, p11, interval); break;
		case 2: spawnPosition = Vector3.Lerp(p00, p01, interval); break;
		case 3: spawnPosition = Vector3.Lerp(p10, p11, interval); break;
	}
	return spawnPosition;
}
```

^d238ba

### Simple AI Behaviour Unit
```csharp
void SearchEnemy()
{
	if (Time.time < checkTime || m_Target != null)
		return;

	checkTime = Time.time + CONST_CHECKTIME;

	Vector3 center = centerTrans.position;
	Collider2D[] hits = Physics2D.OverlapCircleAll(center, m_GuardianData.searchRadius, layerMask);

	float closestDist = float.MaxValue;

	foreach (Collider2D hit in hits)
	{
		Enemy enemy = hit.GetComponent<Enemy>();
		if (enemy == null) continue;

		float dist = Vector3.Distance(center, enemy.transform.position);
		if (dist < closestDist)
		{
			closestDist = dist;
			m_Target = enemy;
		}
	}
}
```

^e563a5

### Camera Control
```csharp
void MobileInput()
{
	if (Input.touchCount == 0) 
		return;
	
	Touch t0 = Input.GetTouch(0);

	if (Input.touchCount == 1)
	{
		if (t0.phase == TouchPhase.Began)
		{
			m_TouchBeganPos = m_Camera.ScreenToWorldPoint(t0.position);
		}
		else if (t0.phase == TouchPhase.Moved)
		{
			Vector3 moved = m_TouchBeganPos - m_Camera.ScreenToWorldPoint(t0.position);
			m_CameraTrans.position += moved;
		}
	}
	else if (Input.touchCount == 2)
	{
		Touch t1 = Input.GetTouch(1);

		Vector2 prev0 = t0.position - t0.deltaPosition;
		Vector2 prev1 = t1.position - t1.deltaPosition;

		float prevMag = (prev0 - prev1).magnitude;
		float currMag = (t0.position - t1.position).magnitude;
		float delta = currMag - prevMag;

		Zoom(delta * m_ZoomSensitivity);
	}
	
	ClampCameraPosition();
}
```

^2f3221

```csharp
void ClampCameraPosition()
{
	Vector3 pos = m_CameraTrans.position;
	pos.x = Mathf.Clamp(pos.x, p00.x, p11.x);
	pos.y = Mathf.Clamp(pos.y, p00.y, p11.y);
	m_CameraTrans.position = pos;
}
```

^089f5d

```csharp
void Zoom(float amount)
{
	m_Camera.orthographicSize = Mathf.Clamp
	(
		m_Camera.orthographicSize - amount,
		m_ZoomMin,
		m_ZoomMax
	);

	// Recalculate camera bounds based on zoom level
	float halfHeight = m_Camera.orthographicSize;
	float halfWidth  = m_Camera.orthographicSize * m_Camera.aspect;

	p00 = new Vector3(p01.x + halfWidth, p10.y + halfHeight);
	p11 = new Vector3(p10.x - halfWidth, p01.y - halfHeight);
}
```

^34e87e
