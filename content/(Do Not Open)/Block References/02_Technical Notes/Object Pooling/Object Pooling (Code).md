# Code Block References
---
### MapGenerator Class - Key Variables
```csharp
public class MapGenerator : MonoBehaviour
{
	// map prefabs to instantiate
	public GameObject[] m_MapPrefabs;
	
	// total maps in the pool
	public int m_MapPoolSize;
	
	// array of maps in the pool
	private GameObject[] m_MapPool;
	
	// list of available pool indices
	private List<int> m_FreeIndices;
    
	// number of maps to spawn initially
	public int m_SpawnAmount;
}
```

### MapGenerator Class - Key Functions
```csharp
private void Start()
{
	CreateMapPool();
	
	InitializeFreeIndices();
	
	SpawnMapsAtStart();
}
```

^b06806

```csharp
private void CreateMapPool()
{
	m_MapPool = new GameObject[m_MapPoolSize];
	
	for (int i = 0; i < m_MapPoolSize; i++)
	{
		GameObject obj = Instantiate(m_MapPrefabs[i % m_MapPrefabs.Length], Vector3.zero, Quaternion.identity, transform);
		obj.SetActive(false);
	
		m_MapPool[i] = obj;
	
		MapBehaviour mapBehaviour = obj.GetComponent<MapBehaviour>();
		mapBehaviour.SetPoolIdx(i);
	}
}
```

^27dd05

```csharp
private void InitializeFreeIndices()
{
	m_FreeIndices = new List<int>(m_MapPoolSize);
	
	for (int i = 0; i < m_MapPoolSize; i++)
	{
		m_FreeIndices.Add(i);
	}
}
```

^31cd3b

```csharp
private int GetRandomFreeIdx()
{
return Random.Range(0, m_FreeIndices.Count);
}
```
```csharp
private void SpawnMapsAtStart()
{
	for (int i = 0; i < m_SpawnAmount; i++)
	{
		int randListIdx = GetRandomFreeIdx();
		int poolIdx = m_FreeIndices[randListIdx];
		m_FreeIndices.RemoveAt(randListIdx);

		m_NextSpawnPoint += new Vector3(0.0f, 0.0f, m_GroundLength);
		m_MapPool[poolIdx].transform.position = m_NextSpawnPoint;

		m_MapPool[poolIdx].SetActive(true);
	}
}
```

^302dda

```csharp
private void ReplaceMap(int oldIdx)
{
	int randListIdx = GetRandomFreeIdx();
	int newIdx = m_FreeIndices[randListIdx];
	m_FreeIndices[randListIdx] = oldIdx;

	m_NextSpawnPoint = m_MapPool[oldIdx].transform.position + new Vector3(0.0f, 0.0f, m_GroundLength * m_SpawnAmount);
	m_MapPool[newIdx].transform.position = m_NextSpawnPoint;

	m_MapPool[newIdx].SetActive(true);
	m_MapPool[oldIdx].SetActive(false);
}
```

^5232ac

```csharp
private void OnTriggerEnter(Collider other)
{
	if (other.CompareTag("Map"))
	{
		MapBehaviour mB = other.gameObject.GetComponent<MapBehaviour>();
		if (mB != null)
		{
			ReplaceMap(mB.GetPoolIdx());
		}
		else
		{
			Destroy(other.gameObject);
		}
	}
}
```

^612f56