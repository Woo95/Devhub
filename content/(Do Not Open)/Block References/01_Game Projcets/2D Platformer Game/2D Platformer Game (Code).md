# Code Block References
---

### GameManager (FSM)
```csharp
public class GameManager : MonoBehaviour
{
    enum eGameState { PLAY, PAUSE, GAMEOVER };
    eGameState m_StateBeforePause;
    eGameState m_GameState;
    
    void Start()
    {
        InPlay();
    }
    
	void Update()
	{
		switch (m_GameState)
		{
			case eGameState.PLAY:
				ModifyPlay();
				break;
			case eGameState.GAMEOVER:
				ModifyGameOver();
				break;
		}
	}
    
	// FSM Play //
    void InPlay()
    {
	    m_GameState = eGameState.PLAY;
    }
    void ModifyPlay()
    {
	    if (!Player.instance.IsAlive())
	    {
		    InGameOver();
		    return;
	    }
    }

	// FSM Pause //
	void InPause()
	{
		m_StateBeforePause = m_GameState;
		m_GameState = eGameState.PAUSE;
		
		Time.timeScale = 0.0f;
		UIPause.instance.Invoke_Pause(true);
		SoundManager.instance.PauseBGM();
	}
	void UnPause()
	{
		m_GameState = m_StateBeforePause;
		
		Time.timeScale = 1.0f;
		UIPause.instance.Invoke_Pause(false);
		SoundManager.instance.UnPauseBGM();
	}

	// FSM GameOver //
    void InGameOver()
    {
	    m_GameState = eGameState.GAMEOVER;
    }
    void ModifyGameOver()
    {
	    SceneManager.LoadScene("GameOverScene");
    }
}
```

^96f9de

### Parallax Scrolling Background
```csharp
public void TrackPlayer()
{
	Vector3 targetPos = new Vector3
	(
		m_Target.position.x + m_Offset.x,
		m_Target.position.y + m_Offset.y,
		m_FixedZ
	);

	transform.position = Vector3.Lerp(transform.position, targetPos, m_Smooth * Time.deltaTime);

	UpdateCameraBoundary();

	BackgroundController.instance.TrackBackGround();
}
```

^4abc81

```csharp
public void TrackBackGround()
{
	foreach (BackgroundInfo bgInfo in m_BackgroundList)
	{
		Vector3 pos = m_Cam.position + new Vector3(-m_Cam.position.x * bgInfo.m_Percent, 0, m_DepthZ);
		
		bgInfo.m_Background.position = pos;
	}
}
```

^6de630

### Player Interaction Mechanics
```csharp
bool IsGrounded()
{
	if (!m_PlayerFootCollider.enabled)
		return false;

	Collider2D groundCollider = Physics2D.OverlapArea(m_FeetPoint1.position, m_FeetPoint2.position, m_PlatformLayerMask);

	return groundCollider != null;
}

void UpdateFootColliderState()
{
	if (!IsGrounded())
	{
		if (m_Rb.velocity.y > 0.0f)
			m_PlayerFootCollider.enabled = false;
		else
			m_PlayerFootCollider.enabled = true;
	}
}
```

^15ab87

```csharp
void CheckPlayerHitbox()
{
	Vector2 p1 = m_BodyPoint1.position;
	Vector2 p2 = m_BodyPoint2.position;

	// Death check
	Collider2D threat = Physics2D.OverlapArea(p1, p2, m_DeathLayerMask);
	if (threat != null)
	{
		PlayerManager.instance.LoseLife();
	}

	// Coin check
	Collider2D coin = Physics2D.OverlapArea(p1, p2, LayerMask.GetMask("Coin"));
	if (coin != null)
	{
		PlayerManager.instance.ObtainedCoin();
		Destroy(coin.gameObject);
		SoundManager.instance.PlaySFX("Coin");
	}
}
```

^9d9c5c

```csharp
private void CheckEnemyUnderfoot()
{
	Collider2D enemy = Physics2D.OverlapArea(m_FeetPoint1.position, m_FeetPoint2.position, m_KillLayerMask);
	if (enemy == null)
		return;

	m_Rb.velocity = Vector3.up * m_JumpVelocity * 0.5f;
	PlayerManager.instance.KilledEnemy();
	Destroy(enemy.gameObject);
	SoundManager.instance.PlaySFX("EnemyKill");
}
```

^d56094
