# source

- buy the game:      https://www.gog.com/en/game/warhammer_40000_rogue_trader
- buy the game:      https://store.steampowered.com/app/2186680/Warhammer_40000_Rogue_Trader/
- download the tool: https://www.jetbrains.com/decompiler/

```
create an empty Unity project
export `Code.dll` et al into "Assets"
be sure to create a root assembly for cleanliness
regenerate project files (maybe delete them first)
let Unity create project files - better than dotPeek ones
```


# patterns

- Command (Encapsulate a request as an object, thereby letting users parameterize clients with different requests, queue or log requests, and support undoable operations.)  
  https://en.wikipedia.org/wiki/Command_pattern  
  https://refactoring.guru/design-patterns/command  
  https://gameprogrammingpatterns.com/command.html  
  see `GameAction`

- Builder  
  https://en.wikipedia.org/wiki/Builder_pattern  
  https://refactoring.guru/design-patterns/builder  
  see `BlackBoardBuilder` (actually don't) et lots of other `*Builder.cs`


# services


# third-party

- https://assetstore.unity.com/packages/tools/sprite-management/2dxfx-2d-sprite-fx-42566  
  2DxFX is an advanced 2D Sprite Tool. Create enhanced visual effect is very easy. Controls and manipulate your effect for 2D sprites.

- https://www.audiokinetic.com/en/wwise/overview  
  Wwise, the most advanced, feature-rich interactive audio solution.

- https://arongranberg.com/astar/  
  https://assetstore.unity.com/packages/tools/behavior-ai/a-pathfinding-project-pro-87744  
  The A* Pathfinding Project is a powerful and easy to use pathfinding system for Unity. With blazing fast pathfinding your AIs will be able to find the player in complex mazes in no time at all.

- https://www.demigiant.com/plugins/demilib/index.php  
  My second collection of open source utility libraries for Unity.

- https://www.demigiant.com/plugins/dotween/index.php  
  https://assetstore.unity.com/packages/tools/visual-scripting/dotween-pro-32416  
  Animate gameObjects visually (move, shake, fade, rotate, change camera properties, and a lot more), draw paths in the editor and follow them, plus power-up DOTween's core with extra features

- https://github.com/JoeCooper/LimbHacker  
  Limb Hacker cuts skinned mesh characters in Unity3D.

- https://github.com/Cysharp/MemoryPack  
  Zero encoding extreme performance binary serializer for C# and Unity.

- https://github.com/neuecc/UniRx  
  UniRx - Reactive Extensions for Unity

- https://assetstore.unity.com/packages/tools/level-design/non-convex-mesh-collider-automatic-generator-117273  
  Non-convex collider is a solved problem now. With this package, you can auto-generate a set of convex colliders to have non-convex ones. The industry-standard algorithm, other big engines are using!

- Octave3D (World Builder/Prefab Placer)  
  dilisted, new iteration is there: https://assetstore.unity.com/packages/tools/level-design/gspawn-level-designer-45021  
  // fucking vendor everything and don't rely on package managers  
  GSpawn is a powerful level design toolkit for Unity. Snap, scatter, paint, and build modular environments directly in the Scene view using fast, intuitive, and smart placement tools.

- https://github.com/ssell/VariablePoissonSampler  
  https://www.vertexfragment.com/ramblings/variable-density-poisson-sampler/  
  Variable Density Poisson-Disk Sampler

- https://superluminal.eu/unity/  
  The profiler with the accessible, fast UI now has built-in Unity support

- https://assetstore.unity.com/packages/tools/animation/final-ik-14290  
  The final Inverse Kinematics solution for Unity.

- https://github.com/slavniyteo/rect-ex  
  RectEx is a tool that simplify drawing EditorGUI (not Layout) elements with Unity3d.

- https://www.photonengine.com/  
  Photon Fusion is the high-end state transfer netcode SDK made for Unity Professionals. Give players the best experience for any gameplay with multiple Network Topology choices.


# code architecture

```
disclaimer: I try and export more and more assemblies, use search, request for usages, etc
but it's a non-zero chance possibility that I miss something

a metric ton of assemblies. yeah, I get it, big project, you need faster iteration times
but bruh =) the amount of dlls can use some brevity. but and idea has been given to me:
it's like splitting inclusions into atomic headers (without zealotry) to avoid cyclic dependencies

all right... let's list something interesing first

// Code, Kingmaker.Core, Owlcat.Runtime.Core, RogueTrader.GameCore
// looks like the root-enough assemblies, and if imported will pull a lot
// of dependencies along - it's a good start; if anything is missing, it can
// be added on the run. I recommend looking up keywords and export basing off of them

// CircularBuffer (T[], index, count)
// could have been: T[], read, write
// not an advice, it's not from the experience, it's a side note for learning purposes
// with caveats of wrapping, and tradeoffs regarding `read == write` case
// as per https://www.snellman.net/blog/archive/2016-12-13-ring-buffers/

not only the number of assemblies is crushing, but depth and number of classes hierarchies
is, let's say, impressive. true technopriests have done the job

given the frightening number of blueprints and stuff, Owlcat simple MUST have a visual
node editor for everything, lest be buried under the weight and mental exhaustion

a fair share of pooling of different object exists; no auto dispose though with `using blah`

main folders
- Kingmaker - the main game
- Owlcat    - tools & utils
- Warhammer - space battles
```

# high-level overview

initialization
```
GameStarter -> Awake -> InitProcess
  -> initialize logging and debugging
  -> WaitForPreviousProcessToFinish (allow only one instance)
  -> initialize wwise audio, settings, analytics
  -> disable batching for OSX (???)
  -> initialize sound controller, localization, text templates, lifetime services
  -> mark InitComplete
  -> initialize music, mods
  -> mark InitComplete / call OnInitComplete

SplashScreenController -> Start
  -> wait GameStarter for InitComplete / OnInitComplete
  -> ShowSplashScreen -> OnComplete
    -> MainMenuLoadingScreen -> StartLoading
      -> GameStarter -> StartGame

GameStarter -> StartGame -> StartGameCoroutine
  -> SettingsController -> StartSyncSettings
    -> initialize bundles load service, astar
    -> load bundles, apply mods, jsons, UI
    -> fix OSX saves, fix text mesh pro
    -> initialize gamepad
    -> load "UI_Common_Scene", "LoadingScreen"
    -> unload "Start" for presets (???)
    -> load "MainMenu"
    -> initialize keyboard
    -> unload game starter scene
    -> initizalize QA stuff, photon

StartGameLoader
```

rendering
```
// lol - https://warhammer40k.fandom.com/wiki/WAAAGH!

+ RenderPipeline
  + WaaaghPipeline - kekw
```

# assets handling

```
BundlesLoadService
  // see `AssetBundleNames` for the listing
  > m_Bundles : string to (`AssetBundle` et all)

ResourcesLibrary
  > s_LoadedResources : string to (`UnityEngine.Object` et al)
  > BlueprintsCache : BlueprintsCache

BlueprintsCache
  // stored @ "blueprints-pack.bbp" (binary)
  // there's also "*.jbp" (json) format (only for dev and mods ?)
  > m_LoadedBlueprints : string to (`SimpleBlueprint` et al)

+ WeakResourceLink
  // I guess, "everyone" does it, even though Unity already have its own `AssetReference` weak handle
  > AssetId : string

  + WeakResourceLink<T>
    > m_Handle : BundledResourceHandle<T>
      // cache only, `WeakReference<T>` among other things

    + AnomalyViewLink            - with `AnomalyView`
    + EquipmentEntityLink        - with `EquipmentEntity`
    + MaterialLink               - with `Material`
    + PrefabLink                 - with `GameObject`
    + ProjectileLink             - with `ProjectileView`

    + ScriptableObjectLink       - self referenced generic
      + AnimationClipWrapperLink - with `AnimationClipWrapper`
      + UnitAnimationActionLink  - with `UnitAnimationAction`

    + SpriteLink                 - with `Sprite`
    + Texture2DLink              - with `Texture2D`

    + UIViewLink                 - with `ViewBase<TVM>` and `IViewModel`
      > Target : Transform
      > View   : ViewBase<TVM>

      + UIDestroyViewLink        - same data
        > override `Unbind` -> destroy object first

    + UnitViewLink               - with `UnitEntityView`
    + VideoLink                  - with `VideoClip`
    + SceneLightConfig.Link      - with `SceneLightConfig`
```


# gameplay

entities
```
// non-exaustive tree, left here as breadcrumbs

+ Entity : IEntity
  // and a multitude of subclasses
  > Facts : EntityFactsManager
  > Parts : EntityPartsManager

  + MechanicEntity
    + MechanicEntity<T> - with `BlueprintMechanicEntityFact`
      + Area            - for `BlueprintArea`
      + CargoEntity     - for `BlueprintCargo`
      + ItemEntity      - for `BlueprintItem`
        + ItemEntity<T> - with `BlueprintItem`
      + MapObjectEntity - for `BlueprintMechanicEntityFact`

  + Player
    > AiDataCollector : AiDataCollector

  + QuestBook
  + SimpleEntity

// some OOP crimes all over again, a great game nevertheless
// not a constructive feedback, only momentary thoughts

+ EntityPart
  + EntityPart<T>           - with `IEntity`
    + MechanicEntityPart<T> - with `MechanicEntity`
      + BaseUnitPart<T>     - with `BaseUnitEntity`
        + BaseUnitPart      - for `BaseUnitEntity`
        + UnitPart          - for `UnitEntity`
        + StarshipPart      - for `StarshipEntity`
      + AbstractUnitPart<T> - with `AbstractUnitEntity`
        + AbstractUnitPart  - for `AbstractUnitEntity`
          + PartUnitState
          + PartLifeState

// - mmm, free OOP ! mmm, tasty !
// - the secret ingredient is crime
// https://www.youtube.com/clip/UgkxLLErUZLYQiDKckUyC4IP0oeXu1vrdGOK

// I mean, yeah, being data-driven (not data-oriented!), this and that,
// but it feels like a lot of mental load TBH

EntityFactsManager
  // ?
  > m_Facts : EntityFact[]
  > m_Cache : EntityFactsCache

EntityPartsManager
  // ?
  > m_Parts : EntityPart[]
  > m_PartsCache : EntityPart[]

// clearly you can see a great deal of composition here, even though obscure ATM
```

blueprints (custom json-based scriptable objects)
```
// yeah, yeah, memory packer and stuff, but I assume, for the development
// at edit time these are really somewhat human-readable jsons

+ SimpleBlueprint
  > m_AllElements : Element[]
  // and a multitude of subclasses

  + BlueprintScriptableObject
    > Components : BlueprintComponent[]
    // and a multitude of subclasses

    + BlueprintBrainBase

// it's admirable engineering feat... but why ? NIH ?
// I love creating systems... probably, I lack understanding ?

// I see hints of scriptable objects here and there, but it looks like
// Owlcats either opted out of using them or were cheeky from the start
```

blueprint elements
```
+ Element
  // and a multitude of subclasses

  + Condition
    + ContextCondition

  + GameAction
    + ContextAction
    + ContextActionAdapter

  + Evaluator
    + GenericEvaluator<T>

  + PropertyGetter
    + BlueprintPropertyGetter

so Owlcat wasn't content with `BlueprintComponent`s, and added this ?
```

blueprint components
```
+ BlueprintComponent
  // and a multitude of subclasses

so Owlcat wasn't content with `Element`s, and added this ?
```

blueprint references
```
+ BlueprintReferenceBase
  // and a multitude of subclasses
  > Cached : BlueprintScriptableObject (property)

  + BlueprintReference<T> - with `T : BlueprintScriptableObject`
```

resources
```
ResourcesLibrary
  > TryGetBlueprint : SimpleBlueprint with `guid : string`
```

# gameplay: AI

behviour: generic
```
BehaviourTree
  > root : BehaviourTreeNode
  > blackboard : Blackboard
    // see it for "tick" optimization

// N.B.: read notes below

+ BehaviourTreeNode
  // and a multitude of subclasses
  > Tick -> Status
    > drive `Blackboard -> Stack`'s "push" and "pop"
    > run `TickInternal`
  > TickInternal -> Status
    // subclasses implement and execute their logic here

  + Composite
    > children : BehaviourTreeNode[]

    + Loop     - err, this should have been iterating "children", but it's not
    + Selector - tick item, return first "running" or "success", otherwise "failure"
    + Sequence - tick item, return first "running" or "failure", otherwise "success"

  + Decorator
    > child : BehaviourTreeNode

    + Condition - evaluate confition, tick, then return
      > elseNode : BehaviourTreeNode

    + Inverter  - tick child, invert "failure" or "success", then return
    + Repeater  - tick child, return "running", get N "success" or "failure", then "success"
    + Succeeder - tick child, return "running", then "success"

  + TaskNode - abstract leaf

    + AsyncTaskNode
      + AsyncTaskNodeCreateMoveVariants
      + AsyncTaskNodeExecute
      + AsyncTaskNodeInitializeDecisionContext
        // set `TargetInfo -> AiConsideredMoveVariants`

    + CoroutineTaskNode
      + TaskNodeCastAbility
      + TaskNodeExecuteMoveCommand

    + TaskNodeExecute
    + TaskNodeWithResult

// well, all in all it's a textbook generic behaviour tree;
// some non-leaf nodes' naming is debatable, but that's a non-issue.
// N.B.: the collection of compositions and decorations is open-ended

// @note BTs are modular, and you can altenatively make if-else branch like that:
// `node_select[ node_check{ %branch_1_body% }, %branch_2_body% ]`, with variation on required success state
// where `node_check` conditionally executes its "child" node or return "failure". it might well be that
// the current `Condition` node is a perf optimization, but I wanted to note, that there are variations

BehaviourTreeBuilder
  // not a builder, rather a factory, but whatever
  > Create for `MechanicEntity`
    > option: CreateForUnit     for `UnitEntity`
    > option: CreateForSquad    for `UnitSquad`
    > option: CreateForStarship for `StarshipEntity`
  // so, yeah, only 3 statically created trees, instanced per entity, of course
  // might have used prototypes and reflected copies. not because I like it more;
  // besides straightforward code should be faster, I suppose. however, this place
  // - should not be a bottleneck
  // - could have used some pooling

// specific factories for `BehaviourTreeNode`s

+ AiStrategy
  // is not used as a "strategy" - just a "contract"
  + BodyGuardStrategy
  + HideAwayStrategy
  + LuredStrategy
  + MoveAndCastStrategy
  + ResponseToAoOThreatStrategy
```

memory: generic
```
Blackboard
  // what's odd though, this one is not shared,
  // therefore I would rather name it "local memory"
  > Stack : stack of `BehaviourTreeNode`
    // stores current branch of execution
    // might have been used as an optimization, especially crucial for deep hierarchies
    // but seems to be applied only as a debug tool
  > Entity : MechanicEntity
    // I assume, it's the owner
  > DecisionContext : DecisionContext
    // this part kinda seems like the real blackboard memory
    // - lazily updated via flags on demand, not by external events
    // - multiple sources may exist for each of the getters

// would it be a better fit to have a canonical shared blackboard, which
// effectively precaches world state, and entites read the baked info ?
// after all, that concept is intended as an optimization.
// - new target appeared ? put it into relevant lists and dictionaries
// - new arena hazard ? account it in the same manner
// etc. with this approach, N entities won't need to bother themselves with caching,
// only with reading and rather simpler postprocessing
```

memory: attacks
```
// again, it's not a blackboard, because data is not centralized, but effectively
// plays the same role, because code is free to access someone else's data anyway

AiDataCollector
  > m_Collectors : UnitDataCollector

+ UnitDataCollector
  // this OOP hierarchy could have been an email # 1
  > m_Unit : BaseUnitEntity
  > m_Storage : UnitDataStorage
  + AttackDataCollector

UnitDataStorage
  > m_Datas : CollectedUnitData[]

CollectedUnitData
  > AttackDataCollection : AttackDataCollection

+ DataCollection<T>
  // this OOP hierarchy could have been an email # 2
  + ListBasedDataCollection<T>
    + AttackDataCollection -> with `AttackData`
      > m_AbilityThreatDatas : AbilityThreatData[]
      > GetThreatRange -> int via "m_AbilityThreatDatas"

AbilityThreatData
  > Ability : string
    // not and ID. huh ?
  > TotalDamage : int
  > MaxRange : int

+ TileScorer
  // evaluate utility of a position
  > GetHighestScoreNode -> GraphNode

  + LuredTileScorer
  + MinDistanceToEnemyTileScorer
  + ProtectionTileScorer
    + AttackEffectivenessTileScorer
      > CalculateEnemyTargetThreatScore
        > request `AttackDataCollection -> GetThreatRange`
```

decisions: brains
```
+ BlueprintBrainBase
    // ... the following is only for non-ships (units and squads, I suppose)
    > TargetOthersIfCantReachHated : bool
    > GetHatedTargets
    > GetPriorityDestroyTarget
    > GetCustomAbilitySettings
    // ... the following is only for ships
    > GetAbilityValue

  + BlueprintBrain
  + BlueprintStarshipBrain

// N.B. these properties and methods are virtual, but meaningful only
// for specific subclasses. look like a telltale that ships were fastened
// to the codebase late with no desire to change architecture
```

decisions: abilities
```
+ AbilityTargetSelector
  + AOETargetSelector
  + ScatterShotTargetSelector
  + SingleTargetSelector

AbilityInfo
  > GetAbilityTargetSelector : AbilityTargetSelector
    // this method could have been a static function
    // because the constructor of this class (not struct)
    // is mostly for caching

TargetInfo
  > Entity : MechanicEntity
  > Node : CustomGridNodeBase
    // nearest to the "entity" `GetNearestNodeXZ`
  > AiConsideredMoveVariants : GraphNode[]
    // see `AttackEffectivenessTileScorer`
```

specifics
```
+ AiScenario
  // soooo, this is kinda utilities for the AI, eh ?
  + BreachScenario
  + HoldPositionScenario
  + PriorityTargetScenario
```


# gameplay: pathfinidng

```
+ GraphNode
  // a class ? why ?
  + CustomGridNodeBase
    + CustomGridNode

CustomGridMeshNode
  // oh, this one is a struct

+ NavGraph
  + CustomGridGraph
    > meshNodes : CustomGridMeshNode[]
    > nodes : CustomGridNode[]
```


# save system

```
```


# messaging

```
EventBus
  // the whole game encompassing messaging system
  > GlobalSubscribers : SubscriptionManager<ISubscriber>
  > EntitySubscribers : dictionary for managers for `IEntity`
  // each `ISubscriber` subscribes itself

// I've tried using a simplistic implementation of this one
// - on the one hand, it works, it's convenient, it's universal
// - on the other, tagged VS untagged subscriptions can be just a little bit confusing
//   for the reason of possibly having the same interface API, but de facto implicitly different calling conditions
```


# UI

```
// VM stands for View-Model
// https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel
// - Model      - ground truth data
// - View       - presentation layer (unity engine)
// - View-Model - business logic for presentation

%anything in the game%
  // "Model" part

ViewBase<T> - with `T : IViewModel`
  // "View" part
  // typical class is named "ClassView"

IViewModel
  // "View-Model" part
  // typical class is named "ClassVM"
  // "UniRx" is the binder
```


# misc

utilities
```
FxHelper
  // VFX manager

UIUtility
  // UI messages, among all other things
  > ShowMessageBox
    > EventBus.RaiseEvent for `IDialogMessageBoxUIHandler`

AiBrainHelper
  // find threats, check if action has damage
```

formulas
```
PropertyCalculator
  > Operation : OperationType
  > Getters : PropertyGetter[]
  > GetValue : int -> with `PropertyContext`
    > run over "getters" an apply "operation" to them

+ PropertyGetter
  // and a multitude of subclasses
  > Settings : PropertyGetterSettings
  > GetBaseValue : int
  > GetValue : int -> with `PropertyCalculator`
    > get base value, apply settings if any, return

// the rabbit hole goes deeper and deeper. hmmm, chances are in the game
// exists a set of generic formulas, for which a set of monolithic strategies
// could have been made, and for the rest of the "calculators" a set of modular
// strategies. would it be possible, would it be viable ? I don't know, but
// I do strive for clarity too, not only for extremely high modularity

// fair point still: Rogue Trader is a huge RPG, with a lot of numbers, and players enjoyed it
```
