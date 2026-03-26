# Minecraft-Like Crafting System Block References
---

### eItemType Enum
```csharp
public enum eItemType   // XX = empty
{
    XX,
    B1, C1, C2, C3, 
    C4, C5, C6, F1, 
    G1, G2, G3, O1,
    P1, P2, S1, S2, 
    T1, Y1
}
```

^7e13a8

### Item Related Classes
```csharp
[CreateAssetMenu(fileName = "New Item", menuName = "Items/New Item")]
public class ItemInfo : ScriptableObject
{
	public Sprite icon;
	public string description = "";
	public eItemType itemType;
}
```

^c616c6

```csharp
public class ItemInstance : MonoBehaviour, IPointerEnterHandler, IPointerExitHandler
{
    [SerializeField] private Image itemIcon;
    [SerializeField] private TextMeshProUGUI itemCountText;
    
    ItemInfo m_ItemInfo = null;
    int count = 0;

    public int Count
    {
        get { return count; }
        set
        {
            count = value;
            UpdateGraphic();
        }
    }

    public void SetItem(Item item)
    {
        m_ItemInfo = item;
        count = Mathf.Max(1, count);  // ensure count starts at 1
        UpdateGraphic();
    }

    private void UpdateGraphic()
    {
        bool hasItem = count > 0;
        itemIcon.gameObject.SetActive(hasItem);
        itemCountText.gameObject.SetActive(hasItem);
        if (hasItem)
        {
            itemIcon.sprite = m_Item.icon;
            itemCountText.text = count.ToString();
        }
    }

    public void OnPointerEnter(PointerEventData eventData)
    {
        if (m_Item != null)
            Inventory.instance.DisplayMessage(m_Item);
    }

    public void OnPointerExit(PointerEventData eventData)
    {
        if (m_Item != null)
            Inventory.instance.DisplayMessage();
    }
}
```

^c6f0e7

### ItemSlot Related Classes
```csharp
public class ItemSlot : MonoBehaviour, IPointerClickHandler
{
	/*
	OnPointerClick 이벤트 함수들:
		- 아이템 상호작용을 처리하는 함수.
		- 종류들:
			- PickItem
			- PlaceItem
			- SwapItem
			- PickHalfOfItem
	*/
	/* 
	OnPointerClick 조건 함수:
		- 슬롯과 커서 상태를 판별하는 함수.
		- 종류들:
			- ItemOnSlotAndCursor
			- ItemOnSlotAndNoCursor
			- SlotAndCursorSameItem
	*/
}
```

^6bfaef

```csharp
public class ItemSlot : MonoBehaviour, IPointerClickHandler
{
	public virtual void OnPointerClick(PointerEventData eventData)
	{
		if (eventData.button == PointerEventData.InputButton.Left)
		{
			if (ItemOnSlotAndCursor())
			{
				if (!SlotAndCursorSameItem())
					SwapItem();
				else 
					StackAllItemCursorToSlot();
			}
			else if (ItemOnSlotAndNoCursor())
			{
				PickItem();
			}
			else if (ItemOnNoSlotAndCursor())
			{
				PlaceItem();
			}
		}
	    else if (eventData.button == PointerEventData.InputButton.Right)
	    {
			if (ItemOnSlotAndCursor())
			{
				if (!SlotAndCursorSameItem())
					SwapItem();
				else
					StackOneItemCursorToSlot();
			}
			else if (ItemOnSlotAndNoCursor())
			{
				PickHalfOfItem();
			}
			else if (ItemOnNoSlotAndCursor())
			{
				DropOneItemCursorToSlot();
			}
		}
	}
}
```

^cc1460

```csharp
public class CraftingInputSlot : ItemSlot
{
	public Crafting crafting;
	
	public override void OnPointerClick(PointerEventData eventData)
	{
        base.OnPointerClick(eventData);
        
		StartCoroutine(CO_CraftInputHandler());
    }
    
	// for ItemSlot ClickHandler to run first
	IEnumerator CO_CraftInputHandler()
	{
		yield return null;
		crafting.InteractInputPanel();
	}
}
```

^0d06b1

```csharp
public class CraftingOutputSlot : ItemSlot
{
	private List<ItemSlot> m_CraftInputList;
	
	public override void OnPointerClick(PointerEventData eventData)
	{
		if (SlotAndCursor())
		{
			if (SlotAndCursorSameItem())
			{
				StackAllItemSlotToCursor();
				UpdateInputPanel();
			}
		}
		else if (SlotAndNoCursor())
        {
			PickItem();
			UpdateInputPanel();
		}
	}

	// OutputItem Handlers
	public void CreateOutputItem(Recipe recipe, List<ItemSlot> craftInput) 
	{ . . . }
	int MinInputItemAmount()
	{ . . . }
	ItemData OutputItemData()
	{ . . . }

	// InputPanel Handler
	void UpdateInputPanel()
	{ . . . }
}
```

^9be179

```csharp
public class CraftingOutputSlot : ItemSlot
{
	public override void OnPointerClick(PointerEventData eventData)
	{
		if (SlotAndCursor())
		{
			if (SlotAndCursorSameItem())
			{
				StackAllItemSlotToCursor();
				UpdateInputPanel();
			}
		}
		else if (SlotAndNoCursor())
		{
			PickItem();
			UpdateInputPanel();
		}
	}
}
```

^8a723c

### Crafting Related Classes
```csharp
[CreateAssetMenu(fileName = "New Recipe", menuName = "Items/New Recipe")]
public class Recipe : ScriptableObject
{
	public ItemInfo[] input = new ItemInfo[9];	// 9 => 3x3 crafting input pannel. Could be any nXn!
	public ItemInfo output;
	public int outputAmount = 1;

	private void OnValidate()
	{
		// prevent invalid crafting output
		outputAmount = Mathf.Max(outputAmount, 1);
	}
}
```

^24120e

```csharp
public class Crafting : MonoBehaviour
{
	[SerializeField] 
	List<Recipe> recipeList = new List<Recipe>();
	[SerializeField]
	List<string> recipeCode = new List<string>();

	[SerializeField]
	GameObject craftingInputPanel;
	[SerializeField]
	CraftingOutputSlot craftingOutputSlot;

	#region GenerateInputCodeFromRecipes before start
	private void OnValidate()   // calls whenever the asset is modified in the Unity Editor.
	{
		GenerateInputCodeFromRecipes();
	}
	void GenerateInputCodeFromRecipes() // add all Recipe input patterns to the recipeCode List
	{
		recipeCode.Clear();
		for (int i=0; i< recipeList.Count; i++)
		{
			if (recipeList[i] == null) // error notifier
			{
				Debug.LogError("recipeList["+ i +"]" + "was null");
				recipeList.RemoveAt(i);
				return;
			}

			ItemInfo[] input = recipeList[i].input;
			string inputCode = GenerateInputCode(input);
			recipeCode.Add(inputCode);
		}
	}
	#endregion

	#region Code Handler
	string GenerateInputCode(ItemInfo[] input)
	{
		string inputCode = "";
		int gridSize = Mathf.CeilToInt(Mathf.Sqrt(recipeList.Count));
		for (int i = 0; i < input.Length; i++)
		{
			if (i > 0 && i % gridSize == 0)
				if (input[i-1] != null && input[i] != null)
					inputCode += '\\';  // if item from current and next line has continous item

			inputCode += (input[i] != null) ?
				input[i].itemType.ToString() : ((eItemType)0).ToString();
		}
		return ModifyInputCode(inputCode);
	}

	string ModifyInputCode(string inputCode)
	{
		int firstCharIndex = 0;
		int lastCharIndex = inputCode.Length - 1;

		while (firstCharIndex < inputCode.Length && inputCode[firstCharIndex] == 'X') // front
			firstCharIndex++;
		while (lastCharIndex >= 0 && inputCode[lastCharIndex] == 'X') // back
			lastCharIndex--;

		if (firstCharIndex < lastCharIndex)	// combine
			return inputCode.Substring(firstCharIndex, lastCharIndex - firstCharIndex + 1);

		return "";
	}
	#endregion

	#region Craft System
	public void InteractInputPanel()    // function calls from the CraftingInputSlot.cs only if any change on InputPanel
	{
		craftingOutputSlot.DestroyCurrentItem();	// to reset output item

		List<ItemSlot> craftInputList =
			new List<ItemSlot>(craftingInputPanel.transform.GetComponentsInChildren<ItemSlot>());

		ItemInfo[] input = new ItemInfo[craftInputList.Count];
		for (int i = 0; i < craftInputList.Count; i++) // stores the 'Item' objects from the craft input slots
		{
			input[i] = craftInputList[i].GetItem();
		}

		string inputCode = GenerateInputCode(input);
		if (inputCode == "") return;

		Debug.Log(inputCode);

		int foundRecipeIndex = recipeCode.FindIndex(code => code == inputCode);
		if (foundRecipeIndex == -1) return; // -1 means not found

		CreateOutputItem(foundRecipeIndex, craftInputList);
	}

	void CreateOutputItem(int foundRecipeIndex, List<ItemSlot> craftInputList)
	{
		Recipe foundRecipe = recipeList[foundRecipeIndex];
		craftingOutputSlot.CreateOutputItem(foundRecipe, craftInputList);
	}
	#endregion
}
```

^b3c61a

### RecipeEditor Class
```csharp
[CustomEditor(typeof(Recipe))]
public class RecipeEditor : Editor
{
	public override void OnInspectorGUI()
	{
		Recipe recipe = (Recipe)target; // cast the target to the Recipe type

		EditorGUI.BeginChangeCheck();

		int gridSize = Mathf.CeilToInt(Mathf.Sqrt(recipe.input.Length));

		EditorGUILayout.LabelField("Recipe Input (" + gridSize + "x" + gridSize + "):", EditorStyles.boldLabel);
		for (int i = 0; i < gridSize; i++) // display the nxn grid of input items
		{
			EditorGUILayout.BeginHorizontal();
			for (int j = 0; j < gridSize; j++)
			{
				int index = i * gridSize + j;
				EditorGUILayout.PropertyField(serializedObject.FindProperty("input").GetArrayElementAtIndex(index), GUIContent.none, GUILayout.Width(100));
			}
			EditorGUILayout.EndHorizontal();
		}

		GUILayout.Space(10);

		// Display the output item field
		EditorGUILayout.LabelField("Output Item:", EditorStyles.boldLabel);
		EditorGUILayout.PropertyField(serializedObject.FindProperty("output"), GUIContent.none, GUILayout.Width(200));

		EditorGUILayout.PrefixLabel("Output Amount:", EditorStyles.boldLabel);
		EditorGUILayout.PropertyField(serializedObject.FindProperty("outputAmount"), GUIContent.none, GUILayout.Width(60), GUILayout.ExpandWidth(false));

		if (EditorGUI.EndChangeCheck())
			serializedObject.ApplyModifiedProperties(); // apply change if change occurs on the end
	}
}
```

^f0a932