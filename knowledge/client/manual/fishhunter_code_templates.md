---
inclusion: manual
---

# 捕魚機程式碼範本 (FishHunter Code Templates)

本文件包含各類魚種控制器、技能系統、演出控制器的完整程式碼範本。
開發新魚種或新演出時，請參考對應範本。

---

## 1. 一般魚種控制器範本

透過 WeaponSystem w2 封包處理捕獲，參考 BuddhaEx、DoubleDragon 等實作：

```csharp
/// <summary>
/// 一般魚種控制器範本
/// 透過 WeaponSystem w2 封包處理捕獲
/// </summary>
public class NormalFishCtrl : Fish2D
{
    #region SerializeField
    [BoxGroup("UI"), SerializeField, LabelText("倍率文字")]
    private Text _oddsText;
    
    [BoxGroup("動畫"), SerializeField, LabelText("魚種本體")]
    private Transform _fishBody;
    #endregion
    
    #region Private Fields
    private int _currentOdds;
    private bool _isCaptured;
    private const string TweenKey = "NormalFish";
    #endregion
    
    #region Lifecycle
    public override void Init(enumFishType type, Vector3 vPosition, float fOrientation,
        enumFishSubType SubType = enumFishSubType.SpecialFish_None, int groupType = 0)
    {
        RegisterEvents();
        base.Init(type, vPosition, fOrientation, SubType, groupType);
        ResetState();
        InitFromExtraData();
    }
    
    public override void Despawn()
    {
        UnregisterEvents();
        DOTween.Kill(TweenKey);
        ResetState();
        base.Despawn();
    }
    
    private void OnDestroy() => UnregisterEvents();
    #endregion
    
    #region Event Registration
    private void RegisterEvents()
    {
        FishHunter_EventManager.Registration(
            FishHunter_EventType.EventType.MyFish_Capture, OnCaptureEvent);
        FishHunter_EventManager.Registration(
            FishHunter_EventType.EventType.MyFish_OddsUpdate, OnOddsUpdateEvent);
    }
    
    private void UnregisterEvents()
    {
        FishHunter_EventManager.Cancellation(
            FishHunter_EventType.EventType.MyFish_Capture, OnCaptureEvent);
        FishHunter_EventManager.Cancellation(
            FishHunter_EventType.EventType.MyFish_OddsUpdate, OnOddsUpdateEvent);
    }
    #endregion
    
    #region Private Methods
    private void ResetState()
    {
        _isCaptured = false;
        _currentOdds = 200;
    }
    
    private void InitFromExtraData()
    {
        if (ExtraData == null) return;
        if (ExtraData.ContainsKey("odds"))
            _currentOdds = ExtraData.ToInt("odds");
        RefreshUI();
    }
    
    private void RefreshUI()
    {
        if (_oddsText != null)
            _oddsText.text = $"*{_currentOdds:N0}";
    }
    
    private void HandleCapture(CaptureData data)
    {
        _isCaptured = true;
        state = enumFishState.enumFishCatched;
        
        // 本家：鎖定砲台防止誤觸
        if (data.IsPlayerSelf)
        {
            FishHunter_EventManager.SendEvent(
                FishHunter_EventType.EventType.LockPlayerShoot, true);
        }
        
        PlayCaptureAnimation(() => {
            SpawnFeatureObject(data);
            End();
        });
    }
    
    private void PlayCaptureAnimation(Action onComplete)
    {
        SetAnimationSpeed(0f);
        _fishBody.DOShakePosition(2f, 30f).OnComplete(() => onComplete?.Invoke());
    }
    
    private void SpawnFeatureObject(CaptureData data)
    {
        FishHunter_EventManager.SendEvent(
            FishHunter_EventType.EventType.MyFish_SpawnFeatureObj,
            data.TotalWin, data.PlayerSeat, data.PlayerId, data.IsPlayerSelf);
    }
    #endregion
    
    #region Event Callbacks
    private void OnCaptureEvent(FishHunter_EventManager.EventBase e)
    {
        var eventData = e as FishHunter_EventManager.FishEventData;
        if (eventData == null) return;
        var data = new CaptureData(eventData.args);
        if (sid != data.TargetSid) return;
        HandleCapture(data);
    }
    
    private void OnOddsUpdateEvent(FishHunter_EventManager.EventBase e)
    {
        // 處理倍率更新...
    }
    #endregion
}
```

---

## 2. 技能魚種控制器範本 (Fish2D 子類別)

技能魚種的指令註冊在 `SkillXxxObject` 中，`Fish2D` 子類別僅負責魚種本體行為：

```csharp
/// <summary>
/// 技能魚種控制器範本 (Fish2D 子類別)
/// 注意：指令註冊在 SkillXxxObject 中，不在此處
/// </summary>
public class SkillFishCtrl : Fish2D
{
    #region SerializeField
    [BoxGroup("技能"), SerializeField, LabelText("攻擊範圍")]
    private float _attackRange = 300f;
    #endregion
    
    private bool _isSkillActive;
    
    public override void Init(enumFishType type, Vector3 vPosition, float fOrientation,
        enumFishSubType SubType = enumFishSubType.SpecialFish_None, int groupType = 0)
    {
        base.Init(type, vPosition, fOrientation, SubType, groupType);
        _isSkillActive = false;
    }
    
    public override void Despawn()
    {
        _isSkillActive = false;
        base.Despawn();
    }
    
    /// <summary>被 SkillModel 呼叫以啟動技能演出</summary>
    public void OnSkillActivated(int seat)
    {
        _isSkillActive = true;
    }
    
    /// <summary>取得攻擊範圍內的魚</summary>
    public int[] GetFishesInRange()
    {
        return new int[0];
    }
}
```

---

## 3. 傳統技能物件範本 (SkillXxxObject)

以電磁蟹為例，展示 SkillXxxObject 的標準結構：

```csharp
/// <summary>
/// 傳統技能物件範本 - 以電磁蟹為例
/// </summary>
public class SkillElectricObject
{
    private SkillSystem m_SkillSystem;
    private GameSystem m_GameSystem;
    public DynamicList<SkillElectric> m_Skills;
    
    public SkillElectricObject(SkillSystem SkillSys, GameClient m_Client)
    {
        m_SkillSystem = SkillSys;
        m_GameSystem = m_Client.getGameSystem();
        m_Skills = new DynamicList<SkillElectric>();
        SetRegister();
    }
    
    /// <summary>在建構子中註冊 Server 指令</summary>
    private void SetRegister()
    {
        m_SkillSystem.Register("sk_electric", OnInfoReceive);
        m_SkillSystem.Register("sk_electric_use", OnShootReceive);
        m_SkillSystem.Register("sk_electric_hit", OnHitReceive);
        
        m_SkillSystem.Event_SkillDestroy += OnDestroySkill;
        m_SkillSystem.Event_ClearSkillObject += OnClear;
    }
    
    /// <summary>接收 Server 技能啟動指令</summary>
    private void OnInfoReceive(JSON JsonData)
    {
        int seat = JsonData.ToInt("seat");
        int crabID = JsonData.ToInt("crab");
        
        m_SkillSystem.AddSkill(ESkillType.Electric, seat, 0, 1, delegate(SkillModel Skill)
        {
            Fish2D crab = FishMaintainer.Instance.GetFishBySID(crabID);
            Vector3 pos = crab != null ? crab.transform.position : Vector3.zero;
            crab?.Kill(seat);
            
            SkillElectric skillElectric = Skill as SkillElectric;
            skillElectric.SetObject(this);
            skillElectric.UseSkill(seat, pos, CrabID: crabID);
            m_Skills.Add(skillElectric);
        });
    }
    
    public void SendShoot(int seat, float x, float y)
    {
        JSON send = new JSON();
        send["seat"] = seat;
        send["x"] = x;
        send["y"] = y;
        m_SkillSystem.Send("sk_electric_use", send);
    }
    
    private void OnDestroySkill(SkillModel Skill, int SeatID)
    {
        if (Skill.SkillType == ESkillType.Electric)
            m_Skills.Remove(Skill as SkillElectric);
    }
}
```

---

## 4. 技能演出實體範本 (SkillModel)

```csharp
/// <summary>
/// 技能演出實體範本 - 繼承 SkillModel
/// 由 SkillXxxObject 透過 SkillSystem.AddSkill() 生成
/// </summary>
public class SkillMyFish : SkillModel
{
    private SkillMyFishObject _parentObject;
    private int _fishID;
    private double _totalWin;
    
    public void SetObject(SkillMyFishObject obj) => _parentObject = obj;
    
    /// <summary>開始技能演出</summary>
    public void UseSkill(int seat, Vector3 position, int fishID)
    {
        _fishID = fishID;
        transform.position = position;
        StartSkillPerformance();
    }
    
    private void StartSkillPerformance()
    {
        int[] targetFishes = CollectTargetFishes();
        if (targetFishes.Length > 0)
            _parentObject.SendHitFishes(SeatID, targetFishes);
    }
    
    /// <summary>接收 Server 擊中結果</summary>
    public void OnHitResult(int[] deadFishes, double totalWin, int totalOdds)
    {
        _totalWin = totalWin;
        foreach (var fishId in deadFishes)
            PlayHitEffect(fishId);
        UpdateScore(totalWin);
    }
    
    /// <summary>技能結束</summary>
    public void OnSkillEnd(double finalWin)
    {
        ShowFinalAward(finalWin);
        UnuseSkill(); // 觸發 SkillSystem.UnuseSkill()
    }
}
```

---

## 5. 通用技能模組範本 (SkillCommonObject)

```csharp
/// <summary>
/// 通用技能模組 - 標準化指令格式
/// </summary>
public class SkillCommonObject
{
    SkillSystem m_SkillSystem;
    public DynamicList<SkillCommonModel> m_Skills;
    
    // 魚種類型 → ESkillType 對應表
    Dictionary<int, ESkillType> fishSkillDict = new Dictionary<int, ESkillType>()
    {
        { 224, ESkillType.Cupid2024 },
        { 10028, ESkillType.VermilionBird },
        { 10029, ESkillType.VermilionBirdEX },
        { 230, ESkillType.MuscleRabbit },
    };
    
    public SkillCommonObject(SkillSystem SkillSys, GameClient m_Client)
    {
        m_SkillSystem = SkillSys;
        m_Skills = new DynamicList<SkillCommonModel>();
        SetRegister();
    }
    
    void SetRegister()
    {
        m_SkillSystem.Register("sk_skill_start", OnSkillStart);
        m_SkillSystem.Register("sk_create_army", OnCreateArmy);
        m_SkillSystem.Register("sk_bomb_fish", OnBombFish);
        m_SkillSystem.Register("sk_end", OnSkillEnd);
    }
    
    void OnSkillStart(JSON data)
    {
        int seat = data.ToInt("player_seat");
        int fishID = data.ToInt("fish_id");
        int fishType = data.ToInt("fish_type");
        JSON skillData = data.ToJSON("skill_data");
        ESkillType skillType = fishSkillDict[fishType];
        
        m_SkillSystem.AddSkill(skillType, seat, 0, 1, delegate(SkillModel Skill)
        {
            SkillCommonModel commonSkill = Skill as SkillCommonModel;
            commonSkill.SkillStart(fishID, seat, fishType, skillData, this);
            m_Skills.Add(commonSkill);
        });
    }
    
    public void SendBombFish(int fishID, int fishType, int[] hitList, JSON hitData)
    {
        JSON sendData = new JSON();
        sendData["fish_id"] = fishID;
        sendData["fish_type"] = fishType;
        sendData["hit_list"] = hitList;
        sendData["hit_data"] = hitData;
        m_SkillSystem.Send("sk_bomb_fish", sendData);
    }
    
    public void SendSkillEnd(int fishID, int fishType)
    {
        JSON sendData = new JSON();
        sendData["fish_id"] = fishID;
        sendData["fish_type"] = fishType;
        m_SkillSystem.Send("sk_end", sendData);
    }
}
```

---

## 6. 通用技能演出實體範本 (SkillCommonModel)

```csharp
/// <summary>
/// 通用技能演出實體範本 - 繼承 SkillCommonModel
/// </summary>
public class SkillMyCupid : SkillCommonModel
{
    private CancellationTokenSource _cts;
    
    public override void SkillStart(int fishID, int seat, int fishType, JSON skillData, SkillCommonObject parent)
    {
        base.SkillStart(fishID, seat, fishType, skillData, parent);
        // Init 先清後建
        _cts?.Cancel();
        _cts?.Dispose();
        _cts = new CancellationTokenSource();
        RunSkillAsync(_cts.Token).Forget();
    }
    
    private async UniTaskVoid RunSkillAsync(CancellationToken token)
    {
        try
        {
            await PlayOpeningAsync(token);
            parentObject.SendCreateArmy(skillFishID, skillFishType);
            // 等待 Server 回應...
        }
        catch (OperationCanceledException)
        {
            // 取消時清理中間狀態（砲台、事件等）
        }
    }
    
    public override void CreateArmy(int fishID, int seat, int fishType, int armyID, string army)
    {
        // 生成魚潮演出...
    }
    
    public override void BombFish(int fishID, int seat, int fishType, 
        double credits, double bet, double totalWin, int totalOdds, 
        int[] fishDeadArray, int[] fishOdds, JSON extraData)
    {
        foreach (var deadFishId in fishDeadArray)
            PlayDeathEffect(deadFishId);
    }
    
    public override void SkillEnd(int fishID, int seat, int fishType,
        double credits, double bet, double totalWin, int totalOdds, JSON extraData)
    {
        ShowFinalAward(totalWin);
        UnuseSkill();
    }
    
    private void OnDestroy()
    {
        _cts?.Cancel();
        _cts?.Dispose();
        _cts = null;
    }
}
```

---

## 7. 演出控制器範本 (FeatureCtrl)

使用 UniTask 非同步流程的標準演出控制器：

```csharp
/// <summary>
/// 演出控制器範本 - 使用 UniTask 非同步流程
/// </summary>
public class FeatureCtrlTemplate : MonoBehaviour
{
    #region SerializeField
    [BoxGroup("播放系統"), SerializeField]
    private FH_SpineActionSystem _spineSystem;
    
    [BoxGroup("播放系統"), SerializeField]
    private FH_AnimatorActionSystem _animatorSystem;
    
    [BoxGroup("UI"), SerializeField]
    private GameObject _blackBG;
    
    [BoxGroup("本家/他家"), SerializeField]
    private Vector3 _selfScale = Vector3.one * 0.65f;
    
    [BoxGroup("本家/他家"), SerializeField]
    private Vector3 _otherScale = Vector3.one * 0.4f;
    #endregion
    
    private CancellationTokenSource _cts;
    private FeatureInitData _initData;
    
    public bool IsPlayerSelf => _initData.IsPlayerSelf;
    public double TotalWin => _initData.TotalWin;
    
    public void Initialize(FeatureInitData data)
    {
        _initData = data;
        // Init 先清後建
        _cts?.Cancel();
        _cts?.Dispose();
        _cts = new CancellationTokenSource();
        SetupForPlayerType();
        RunFeatureAsync(_cts.Token).Forget();
    }
    
    public async UniTask RunFeatureAsync(CancellationToken token)
    {
        try
        {
            await _spineSystem.PlayActionTask(SpineState.Opening, cancellationToken: token);
            
            foreach (var item in ParseFeatureData())
            {
                await _spineSystem.PlayActionTask(item.AnimState, cancellationToken: token);
                UpdateScore(item.Value);
            }
            
            await _animatorSystem.PlayActionTask(AnimState.Ending, cancellationToken: token);
            OnFeatureComplete();
        }
        catch (OperationCanceledException)
        {
            // 取消時也要清理中間狀態（砲台、UI 等）
            ForceEnd();
        }
        catch (Exception e)
        {
            #if DEBUG_LOG
            Debug.LogError($"[Feature] Error: {e}");
            #endif
            ForceEnd();
        }
    }
    
    public void ForceEnd()
    {
        _cts?.Cancel();
        if (IsPlayerSelf)
        {
            FishHunter_EventManager.SendEvent(FishHunter_EventType.EventType.UnlockPlayerShoot);
            FishHunter_EventManager.SendEvent(FishHunter_EventType.EventType.SetButtonsEnable, true);
        }
        Despawn();
    }
    
    private void SetupForPlayerType()
    {
        if (IsPlayerSelf)
        {
            transform.localScale = _selfScale;
            transform.localPosition = Vector3.zero;
            _blackBG?.SetActive(true);
        }
        else
        {
            transform.localScale = _otherScale;
            _blackBG?.SetActive(false);
        }
    }
    
    private void OnFeatureComplete()
    {
        if (IsPlayerSelf)
            FishHunter_EventManager.SendEvent(FishHunter_EventType.EventType.UnlockPlayerShoot);
        
        FishHunter_EventManager.SendEvent(
            FishHunter_EventType.EventType.ShowDeclareBoard,
            TotalWin, _initData.PlayerSeat, _initData.PlayerId, IsPlayerSelf);
        Despawn();
    }
    
    private void Despawn()
    {
        _cts?.Cancel();
        _cts?.Dispose();
        _cts = null;
        Destroy(gameObject);
    }
    
    private void OnDestroy()
    {
        _cts?.Cancel();
        _cts?.Dispose();
    }
}
```
