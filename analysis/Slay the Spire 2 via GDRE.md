# source

- buy the game:      https://store.steampowered.com/app/2868840/Slay_the_Spire_2/
- download the tool: https://github.com/GDRETools/gdsdecomp


# patterns

- prototype (blueprint objects can be cloned to create instance objects)  
  https://en.wikipedia.org/wiki/Prototype_pattern  
  https://refactoring.guru/design-patterns/prototype  
  https://gameprogrammingpatterns.com/prototype.html  
  see `AbstractModel` and its hierarchy

- strategy  (implement logic via an interface, decide which to use at runtime)  
  https://en.wikipedia.org/wiki/Strategy_pattern  
  https://refactoring.guru/design-patterns/strategy  
  see `IControllerInputStrategy`, `IAchievementStrategy`, `ILeaderboardStrategy`, `IPlatformUtilStrategy`, `ILogPrinter`, `ModManagerFileIo`, `INetGameService`, `NCardPlay`, `NetHost`, `NetClient`, `FontPathSet`, `IFormatter`

- state     (succinctly switch between dynamic behaviours)  
  https://en.wikipedia.org/wiki/State_pattern  
  https://refactoring.guru/design-patterns/state  
  https://gameprogrammingpatterns.com/state.html  
  see `CreatureAnimator`, `MonsterMoveStateMachine`

- factory   (centralize objects creation for certain purposes)  
  https://en.wikipedia.org/wiki/Abstract_factory_pattern  
  https://refactoring.guru/design-patterns/abstract-factory  
  see `HoverTipFactory`, `CardFactory`, `PotionFactory`, `RelicFactory`


# code architecture

```
virtual/abstract functions/properties
  base class -> regular function that calls a virtual function
  sub  class -> implements virtual function
  result     -> no chance messing the order or frogetting to call `base.Something()`
  // this is violated only for Godot's classes, because it has no control over "userland"

async/await functions
  // `Task -> Run` is called at a few places, without cancellation tokens
  // `CancellationTokenSource` and `CancellationToken` are used interchangeably at times
  // `ThrowIfCancellationRequested` is a less popular async flow control than `IsCancellationRequested`
  `OperationCanceledException` -> source is `ThrowIfCancellationRequested`
  `TaskCanceledException`      -> if `Task -> Run` was cancelled before the invoke
```


# services

- https://sentry.io/  
  Application monitoring software considered "not bad" by millions of developers

- https://weblate.org/  
  Web-based continuous localization

- https://www.heroku.com/  
  https://herokuapp.com/  
  Heroku Metrics gives you powerful insights on the runtime characteristics of your applications.

- https://time.megacrit.com/  
  responds with... time. for dailies


# third-party

- https://github.com/axuno/SmartFormat  
  SmartFormat is a is a lightweight text templating library written in C# which can be a drop-in replacement for string.Format. More than that SmartFormat can format data with named placeholders, lists, localization, pluralization and other smart extensions.

- https://www.fmod.com/  
  Made for games - FMOD is the solution for adaptive audio.

- https://www.codeandweb.com/texturepacker  
  Create sprite sheets and optimize your game graphics.


# garbage collection and memory optimization
```
// these places trigger `GC.Collect`
PreloadManager -> LoadRunAssets
PreloadManager -> LoadActAssets
PreloadManager -> LoadRoomAssets
LocManage      -> TriggerLocaleChange
NMainMenu      -> AbandonRun

// great: game makes a big transition - time to clean up !

NodePool<T> - can be accessed via non-generic `NodePool`
  // is being used in tandem with `GodotTreeExtensions -> QueueFreeSafely`
  // which detaches a `Node` from its parent, leaving it floating in the tree
  // then the pool does its part, without Godot's tree interactions, except
  // for the fallback, in case someone tries to pool a non-pool allocate `Node`
  > INodePool
    > Get  -> fetch from pool or instantiate
    > Free -> return to pool or free
  > IPoolable
    > OnInstantiated     - empty so far
    > OnReturnedFromPool - implementations set up defaults if `Node -> IsNodeReady` (no logs though)
    > OnFreedToPool      - implementations do cleanups

LINQ and temporary objects
  // does it happen NOT each frame      ? might be ok
  // profile says it's not a bottleneck ? it is likely ok
```


# high-level overview

structure
```
"src"                  - ...
  "Core"               - gameplay/networking, models/views, testing/meta, etc
    "Achievements"     - 
    "Animation"        - `CreatureAnimator`
    "Assets"           - `AssetManager`, `AtlasResourceLoader`
    "Audio"            - names for fmod and stuff
    "AutoSlay"         - autoplay
    "Bindings"         - `MegaSpineBinding`
    "CardSelection"    - some localization stuff, eh ?
    "Combat"           - `CombatState`
    "Commands"         - static conveniency functions
    "Context"          - singleplayer / multiplayer detections helpers
    "ControllerInput"  - ``, `IControllerInputStrategy`
    "Daily"            - 
    "Debug"            - 
    "DevConsole"       - `DevConsole`, `AbstractConsoleCmd`
    "Entities"         - kinda additional data for models, eh ? `Creature`, `Player`, etc
    "Events"           - kinda additional data for events, eh ?
    "Exceptions"       - 
    "Extensions"       - 
    "Factories"        - for `CardModel`, `PotionModel`, `RelicModel`; tips are listed separately
    "GameActions"      - `GameAction`
    "Helpers"          - 
    "Hooks"            - mass processing of `IRunState` and `CombatState`
    "HoverTips"        - `HoverTipFactory`, `IHoverTip`
    "Leaderboard"      - 
    "Localization"     - `DynamicVar`, `FontPathSet`, `LocManager`
    "Logging"          - 
    "Map"              - `ActMap`, and stuff
    "Modding"          - `ModManager`, `Mod`, and stuff
    "Models"           - `AbstractModel`
    "MonsterMoves"     - `MonsterMoveStateMachine`, `AsbtractIntent`
    "Multiplayer"      - 
    "Nodes"            - interfacing with Godot directly
    "Odds"             - wrappers for `Rng`
    "Platform"         - GAMING platform impl, Steam and Nil currently
    "Random"           - wrappers for `System.Random`
    "Rewards"          - model for them
    "RichTextTags"     - `RichTextEffect` implementations
    "Rooms"            - `AbstractRoom` and stuff
    "Runs"             - `RunState`
    "Saves"            - handlers and data
    "Settings"         - aspect, fast mode, vsync
    "TestSupport"      - something for tests and autoruns
    "TextEffects"      - visual convenience for `DamageVar`
    "Timeline"         - implementation of `EpochModel` and related stuff
    "Unlocks"          - convenience for `EpochModel`
    "ValueProps"       - flags for `DamageVar` and `BlockVar`
  "GameInfo"           - some json adapters, dunno
  "gdscript"           - fmod adapter and VFX testing
"addons"               - ...
  "atlas_generator"    - generation of `*.tres` sprites from `*.png` and TexturePacker sheet into `.sprites` folder
  "dev_tools"          - dotnet and nuget
  "fmod"               - audio
  "mega_text"          - wrappers for label and rich text
  "megacontentcreator" - creator tools for: cards, powers, relics, monsters
  "sentry"             - SDK for external logging
"MegaCrit"             - pooling and... migrations ? aight
"RiderTestRunner"      - well, now we know what IDE they love
"System"               - regex codegen, ignore
```

initialization
```
Project Settings -> Global -> Autoload
  // [autoload] in "project.godot"

  SentryInit="*res://addons/sentry/SentryInit.gd" with `SentryInit.gd`
    // external logging service

  OneTimeInitialization="*res://scenes/one_time_initialization.tscn" with `NOneTimeInitialization`
    ... essential
    > Godot's ResourceLoader    -> AddResourceFormatLoader with AtlasResourceLoader
    > AtlasManager              -> LoadEssentialAtlases
    > SaveManager               -> InitSettingsData
    > ModManager                -> Initialize
    > UserDataPathProvider      -> set IsRunningModded (oh, cool, modded saves are separate)
    > LocManager                -> Initialize
    > ModelDb                   -> Init (reflection-load all canonicals)
    > ModelIdSerializationCache -> Init (that's network related, my guess)
    > ModelDb                   -> InitIds (for some unused sorting ?)
    ... deferred (was it meant to be Godot's Callable ?)
    > AtlasManager              -> LoadAllAtlases
    ... non-editor
    > ModelDb                   -> Preload (cache data into fields)
    > .                         -> PrewarmJit (oh, fancy, manually doing this)

  AssetLoader="*res://scenes/asset_loader.tscn" with `NAssetLoader`
    // singleton, dispatches `AssetLoadingSession -> Process`

  DevConsole="*res://scenes/debug/dev_console.tscn" with `NDevConsole`
    // singleton, handles input, autocomplete and all good UX stuff
    // dispatches string commands to `DevConsole` and stuff

  CommandHistory="*res://scenes/debug/command_history.tscn" with `NCommandHistory`
    // singleton, displays `CombatManager -> History`

  MemoryMonitor="res://scenes/debug/memory_monitor.tscn" with `NMemoryMonitor`
    // RAM usage, asset cache load, etc

  FmodManager="*res://addons/fmod/FmodManager.gd" with `FmodManager`
    // sound


Project Settings -> General -> Application
  // [application] in "project.godot"

  run/main_scene="res://scenes/game.tscn" with `NGame`
    // -> Run
    // entry point
```

scenes hierarchy
```
NSceneContainer - (not Godot's, but gameplay's)
  // current "scene", all of them are "PackedScene.GenEditState.Disabled"
  `NGame` - entry point
    // `RootSceneContainer : NSceneContainer -> SetCurrentScene`
  - "res://scenes/screens/main_menu/logo_animation.tscn"         with `NLogoAnimation`         - splash
  - "res://scenes/screens/main_menu.tscn"                        with `NMainMenu`              - main menu
  - "res://scenes/run.tscn"                                      with `NRun`                   - top bar among other things
      // `NRun -> SetCurrentRoom` to `_roomContainer : NSceneContainer -> SetCurrentRoom`
    - "res://scenes/rooms/combat_room.tscn"                      with `NCombatRoom`            - piles, energies, etc
    - "res://scenes/rooms/map_room.tscn"                         with `NMapRoom`
    - "res://scenes/rooms/merchant_room.tscn"                    with `NMerchantRoom`
    - "res://scenes/rooms/rest_site_room.tscn"                   with `NRestSiteRoom`
    - "res://scenes/rooms/treasure_room.tscn"                    with `NTreasureRoom`
    - "res://scenes/rooms/event_room.tscn"                       with `NEventRoom`
        // `_eventContainer : NSceneContainer -> SetCurrentScene`
      > "res://scenes/events/default_event_layout.tscn"          with `NEventLayout`           - "_portrait", narrative
      > "res://scenes/events/combat_event_layout.tscn"           with `NCombatEventLayort`     - "EmbeddedCombatRoom"
      > "res://scenes/events/ancient_event_layout.tscn"          with `NAncientEventLayout`    - "_ancientBgContainer", room zero
      > "res://scenes/events/customfake_merchant.tscn"           with `NFakeMerchant`
      > "res://scenes/events/customfake_merchant_inventory.tscn" with `NFakeMerchantInventory`

// `NCombatRoom` is accessed universally via `NRun -> CombatRoom`
// some cases handle `NMerchantRoom` and `NFakeMerchant`, but not universally
// all in all there's a lot of code spread thin over the project
```

types hierarchies
```
text effects     via `AbstractMegaRichTextEffect` part of `MegaRichTextLabel`
                 sup `RichTextEffect`             part of `RichTextLabel` of Godot itselfs
console commands via `AbstractConsoleCmd`         part of `DevConsole`
enemy intents    via `AbstractIntent`             part of `MonsterMoveStateMachine`
game objects     via `AbstractModel`              part of `ModelDb`
act rooms        via `AbstractRoom`               part of `RunState`
diverse chances  via `AbstractOdds`               wrapper for `Rng`, which is wrapper for `System.Random`
                 sub `CardRarityOdds`             part of `PlayerOddsSet`
                 sub `PotionRewardOdds`           part of `PlayerOddsSet`
                 xxx `PlayerRngSet`               part of `PlayerOddsSet`
                 sub `UnknownMapPointOdds`        part of `RunOddsSet`
```


# assets handling

```
NAssetLoader
  // Godot-facing runner
  // guarded with `ConcurrentQueue`
  // what cases have more than one session, given 
  // that launching new one unloads current assets ?
  // is it like, "you click back and forth too fast" ?

AssetSets
  // DB, but not the only source, just a convenience

PreloadManager
  // dispatcher of async tasks
  // each `LoadAssetSets` request unloads all non-requested entries

AssetCache
  // storage of `Resource` instances
  // anything non-cached can be loaded, but you will get scolded for that
  // guarded with a `ConcurrentDictionary`; not sure what use cases it does help

AssetLoadingSession
  // an async batch that will populate `AssetCache`
  // work is time sliced, expect `Process` called multiple times
  // - move from finilizing to cache
  // - dispatch a new batch
  // - update loading queue:
  //   - dequeue
  //   - loaded      -> move to finalizing (why not move to cache ?)
  //   - failed      -> move to failed
  //   - invalid     -> try sync loading or fail
  //   - in progress -> enqueue back
```


# data model: hierarchy

```
- AbstractModel -

+ ActModel
  + Glory
  + Hive
  + Overgrowth
  + Underdocks

+ AchievementModel -> there's no achievement in the early access, but code is being worked on
  // names of the subclasses are not that important, but, let's list useful bits
  // - LocalContext.IsMe, LocalContext.IsMine    - networking
  // - AchievementsUtil.Unlock, enum Achievement - common code and data

+ AfflictionModel
  // cards' curses
  // mocks
  + Bound
  + Entangled
  + Galvanized
  + Hexed
  + Ringing
  + Smog

+ CardModel - also mocks
+ CardPoolModel
  // sets of cards representing different categories
  // class-bound   : ironclad, silent, regent, necrobinder, defect
  // run-generated : colorless, events, curses, quests, statuses, tokens
  // other         : deprecated (loaded a removed thing), mocks

+ CharacterModel
  + Ironclad
  + Silent
  + Regent
  + Necrobinder
  + Defect
  + RandomCharacter
  + Deprived - mock

+ EnchantmentModel
  // card modificators
  // mocks

+ EncounterModel
  // battle encounter
  // mocks

+ EventModel
  // narrative room encounter
  // mocks

+ ModifierModel
  // run modificators

+ MonsterModel
  // all the enemies
  // mocks

+ OrbModel
  // Defect's special objects
  // interestingly enough, Regent has its Seeking Edge too - but it's coded as a power

+ PotionModel

+ PowerModel
  // all the results of "Power" type cards
  // mocks

+ RelicModel
+ RelicPoolModel
  // sets of relics representing different categories
  // class-bound   : ironclad, silent, regent, necrobinder, defect
  // run-generated : event, shared, fallback
  // other         : deprecated (loaded a removed thing)

+ SingletonModel
  // interestingly enough, this "singleton" is "mutably copied" and valid only through a run
  + MultiplayerScalingModel - difficulty scaling


- EpochModel -
  // has its own subclasses hierarchy
  // is not an `AbstractModel`'s subclass, which makes sense as it's not for core gameplay
  // it is the metaprogression shown as narrative cards in the main menu
  GetTimelineExpansion -> lists `EpochModel` unlocks
  QueueUnlocks         -> shows unlocked `EpochModel`, `EventModel`, `CardModel`, `RelocModel`, `PotionModel`
```

database
```
is called so... because they're the ground-truth data models, I suppose;
both in sense of immutable database and mutable runtime state

ModelDb is populated via reflection, takes into account mods too
then caching for convenience happens
```


# data model: "blueprint" / "instance" concept

it's the root class for any in-game thing and can be in two kinds:
- an instance, when field    `IsMutable`   is true
- a blueprint, when property `IsCanonical` is true (`=> !IsMutable`)

it is set via these two functions (one, if you disregard tests)
```
// effectively it means "instantiate this blueprint or instance"
. -> MutableClone, calls DeepCloneFields
. -> NeverEverCallThisOutsideOfTests_SetIsMutable
```

which is called from a handful of sites
```
// class `AbstractModel`
. -> ClonePreservingMutability

// subclasses of `AbstractModel`, some of them store `CanonicalInstance`
// basically it means "instantiate this blueprint"
ActModel         -> ToMutable
AfflictionModel  -> ToMutable
CardModel        -> ToMutable
CardPoolModel    -> ToMutable
EnchantmentModel -> ToMutable
EncounterModel   -> ToMutable
EventModel       -> ToMutable
ModifierModel    -> ToMutable
MonsterModel     -> ToMutable
OrbModel         -> ToMutable
PotionModel      -> ToMutable
PowerModel       -> ToMutable
RelicModel       -> ToMutable

// subclasses of `RelicModel`
ArchaicTooth -> GetTranscendenceTransformedCard for EnchantmentModel
Claws        -> CreateMaulFromOriginal          for Maul
PaelsTooth   -> AfterObtained                   for CardModel

// root classes
HoverTipFactory -> FromCard     for CardModel
RunState        -> CreateShared for MultiplayerScalingModel

// engine interfacing classes (nodes)
NGridCardHolder    -> UpdateCardModel            for CardModel
NCardLibrary       -> UpdateFilter -> TextFilter for CardModel
NInspectCardScreen -> UpdateCardDisplay          for CardModel
```

now, let's look at `ClonePreservingMutability`  
notice that this type of cloning is specific to cards only
```
// class `CardModel`
. -> DeepCloneFields for EnchantmentModel
. -> DeepCloneFields for AfflictionModel

// implementors of `ICardScope`
CombatState -> CloneCard for CardModel
RunState    -> CloneCard for CardModel

// subclasses of `CardModel`
Misery -> OnPlay for PowerModel
```

these models implement `DeepCloneFields`
```
CardModel
EventModel
PowerModel
RelicModel

ActModel
EnchantmentModel
MockCardPool
MockSkillCard
```

these models implement `AfterClones`
```
// mostly doing cleanups after `Object -> MemberwiseClone`

CardModel
EventModel
PowerModel
RelicModel

AfflictionModel
OrbModel
PotionModel
SovereignBlade
ArchaicTooth
PaelsTooth
TouchOfOrobas
YummyCookie
```


# data model: structure

```
- abstract -
  ModelId (Category, Entry)
    > AbstractModel : part of Аbase type name, type name
    > EpochModel    : "epoch", id
    > none          : "NONE", "NONE"
  
  CategorySortingId, EntrySortingId
    // int mapped `ModelId`

  PreviewOutsideOfCombat
    // true for EnchantmentModel

  ShouldReceiveCombatHooks
    // exists albeit not used or a decompilation issue ?
    // `CombatState -> IterateHookListeners` ignores it
    // it is mainly called from `Hook`

  // a heck-ton of functions
  // Before/After : `Task` -> reactions
  // Modify       : `T`    -> mutate input, oftentimes `T` is `decimal`
  // TryModify    : `bool` -> mutate input
  // Should       : `bool` -> check conditions

  // again, what fascinates me, some decisions are giving light legacy vibes
  // which never impedes the game to be great; this shows that it was/is more
  // important to get things done than to have the most perfect architecture

- models with CanonicalVars AND DynamicVars -
  CardModel
  EnchantmentModel
  EventModel
  PotionModel
  PowerModel
  RelicModel

  DynamicVarSet
    // container for `DynamicVar`
    // is generated with CanonicalVars as the input

  + DynamicVar
    > Name      : string
    > BaseValue : decimal
    // some strings could have been enums. but it works, so - meh !
    // there are subclasses to it, many of them simply automate name change, others are not used
    + CalculatedVar -> lambda-modifed with additional API
      // relies on external vars `CalculationBaseVar` and `CalculationExtraVar` to work
      // question: why `BaseValue` is ignored ? an these two - for some convenience of access ?
      + CalculatedBlockVar
      + CalculatedDamageVar
    + BlockVar / DamageVar / OstyDamageVar
      > Props : enum ValueProp
    + ColorPrefix
      > ColorPrefix : string
    + ExtraDamageVar
      > IsFromOsty : bool
    + UpgradeDisplay
      > upgradeDisplay : enum UpgradeDisplay
    + PowerVar<T>
      > where T : PowerModel
    + StringVar
      > StringValue : string
      // decimal `BaseValue` is a dead weight; but I guess who cares

  // my guess:
  // - `MutableClone` easily copies `CanonicalVars`
  // - less manual work to do in `DeepCloneFields`
  // - just one call to `DynamicVarSet -> Clone`
  // which reduces chances to forget and fuck up
```


# UI

```
NRun -> GlobalUi : NGlobalUi
  // handles aspect ratio
  > Overlays : NOverlayStack
  > TopBar   : NTopBar
  > etc

`NOverlayStack` and its `IOverlayScreen`, a superclass of `IScreenContext`
  // all the expected operations like push and remove, but besides that
  // - map screen has the precedence over any overlay
  // - doubles as a container for overlays

`ActiveScreenContext` and its `IScreenContext`
  > `GetCurrentScreen` is a fuckton of ifs
  // thouhts: might have been a stack too, but ok
```


# gameplay: wrapping objects

```
- locations -
  + AbstractRoom
    + CombatRoom
      > EncounterModel
    + EventRoom
      > EventModel

- entities -
  Creature
    > MonsterModel
    > Player
      > CharacterModel
```


# gameplay: combat flow

```
`NGame -> StartRun`
  // there are options like loading, debug, etc
  > assets for the run and act get loaded
  > `RunManager -> Launch`
  > `RootSceneContainer -> SetCurrentScene` with `NRun -> Create`
  > `RunManager -> EnterAct`
    > generate map and enter room

`MoveToMapCoordAction -> ExecuteAction`
  // there are options like fresh runs and tests
  > `NMapScreen -> TravelToMapCoord`
    > `RunManager -> EnterMapCoord`
      > `CreateRoom` and `EnterRoom`
        > `AbstractRoom -> Enter`

`CombatRoom -> Enter -> StartCombat`
  // note that `EventRoom` can have combat and we then get deeper hierarchy
  > assets get loaded
  > monsters creatures are instantiated
  > `NRun -> SetCurrentRoom` with `NCombatRoom -> Create`
  > `CombatManager -> SetUpCombat` with a `CombatState` instance
  > `Hook -> AfterRoomEntered`
  > `CombatManager -> AfterCombatRoomLoaded -> StartCombatInternal`
    > `Hook -> BeforeCombatStart`

`EventRoom -> Enter`
  // `Punchoff`, `TheArchitect`, `TheLanternKey`
  > assets get loaded
  > `RunManager -> EventSynchronizer -> BeginEvent`
    > `EventModel -> BeginEvent -> BeforeEventStarted`
  > `EventModel -> GenerateInternalCombatState`
  > `NRun -> SetCurrentRoom` with `NEventRoom -> Create`
  > `Hook -> AfterRoomEntered`
  > `EventModel -> AfterEventStarted`

`NRun -> _roomContainer.CurrentScene` - two options
  > as `NCombatRoom`
  > as `NEventRoom -> _eventContainer.CurrentScene`
    > as `NEventLayout`, as `NCombatEventLayout`
      > `.EmbeddedCombatRoom : NCombatRoom`

// a bit weird organization, but I guess you need to store things
// somewhere, and at the same time have a generic battle actor type
```


# state machines

```
CreatureAnimator        - graph of AnimState
  both for players and monsters; spine as a backend
  transitions are in a dictionary [trigger : conditional targets]
  always has "anyState" node

MonsterMoveStateMachine - graph of MonsterState
  variant MoveState              - single link, logic node, Func (logic) / AbstractIntent (visual)
  variant RandomBranchState      - multi links, transition, weighted random
  variant ConditionalBranchState - multi links, transition, choices

// what's cool - these systems are abstract and extendable
// switching over an enum is convenient, but there's a catch:
// there's no system under it except for the `swtich / case`;
// it does have its place, but only for one-off behaviours.
// OTOH task-specific states wouldn't be less cumbersome either
// and that is why I like so much this particular implementation:
// FSM gives structure and flow, and you plug any logic into it
```


# input

```
// `InputMap` is handled too, but not in sense of game commands,
// but to read Godot's data and dispatch `Controller` actions
// question: why doesn't it conflict with the steam input ?
// also there's quite a lot of abandoned experiments' leftovers

NHotkeyManager
  // tracks contextual input actions
  // different parts of the game can push and pop their overrides
  // blocks input behind nodes provided in the same fashion

MegaInput - game actions
  // some of them are keyboard-only

Controller - `InputMap` actions

ControllerConfig
  // GlyphMap                  : Controller to images
  // DefaultControllerInputMap : MegaInput  to Controller
  // SteamInputControllerMap   : Steam      to Controller

NInputManager
  // receives `_UnhandledKeyInput` and `_UnhandledInput`
  // dispatches `MegaInput`

NControllerManager
  // runs `IControllerInputStrategy -> ProcessInput`
  // and dispatches `Controller` actions
```


# modding

```
ModManager
  // collection of `Mod`
  // much of the functions read meta and sort
  `Initialize -> TryLoadMod` does the interesting job

Mod
  // meta and save data
```


# singleplayer

```
+ GameAction -> ToNetAction : INetAction
  // action data has networked counterparts
```


# multiplayer

```
// @todo there's a lot more
// steam and non-steam paths

-- game --
  + INetGameService
    + NetSingleplayerGameService
    + NetReplayGameService
    + NetClientGameService
    + NetHostGameService

  NetMessageBus
  // reader-writer conveniency

-- serialization --
  PackerReader
  PackerWriter

  + INetAction -> ToGameAction : GameAction
    // action packets has local counterparts

  + INetMessage - (some of them are classes for an unknown reason)
    // message packets are network only, unlike actions

-- transport --
  + NetHost
    + ENetHost
    + SteamHost

  + NetClient
    + ENetClient
    + SteamClient
```


# autoplay

```
// "Core/AutoSlay"
```


# audio

```
// example
SfxCmd                    -> Play
  > NAudioManager         -> PlayOneShot     -> _audioNode.Call
    > audio_manager_proxy -> "play_one_shot" -> FmodServer.create_event_instance
      > FmodEvent         -> start
```


# localization

```
FontManager
  // chooses `FontPathSet` by language
  // caches fonts

LocManager
  // chooses `IFormatter`
```


# misc

logging
```
unlike Unity, you need to use `StackTrace` manually
timestamping is also manual and extremely sparing

log level is manual, but they use Godot's trinity too
- info    is the standard OS output
- warning has two kinds: message and debugger
- error   has two kinds: message and debugger
```

utilities
```
TaskHelper
  RunSafely - try-catches, logs, and rethrows; massively "popular" throughout the codebase
    // my theory: either this function shouldn't rethrow anything, and they have forgotten
    // an experimental chunk; or they opted to handle the final catch to the Godot.
    // I doubt about the latter, for it would "silently/screamingly" skip lines
  WhenAny   - convenince, because standard one returns `Task<Task>`

AchievementsUtil
  // foremost, it chooses `IAchievementStrategy`
  // second,   it sends a callback when something has changed
  // finally,  it's and interface for unlocking and revoking

PlatformUtil
  // a dispatcher for `IPlatformUtilStrategy`

LeaderboardManager
  // foremost, it chooses `ILeaderboardStrategy`
  // and ofc,  then it dispatches abstract requests
  // it's called "manager" but behaves as a mere wrapper
  // at the same time `ModManager` uses a strategy for IO, but does far
  // more than that; btw, it's interesting, that it needs it's own IO strategy
  // but I think that makes sense, if mods doesn't come from a res/user directories

Logger
  // foremost, it chooses `ILogPrinter`
  // and ofc,  then it dispatches abstract requests
  // note, that logger is not a static or singleton class
  // different systems create their personal instances.
  // might have been separated into LoggerUtils and Logger parts

Log
  // generic `Logger` instance container

MathHelper
  // some obvious conveniences, alongside bezier curves and stuff

TweenHelper
  // wait tweens using async tasks

GodotTreeExtensions
  // whoa, removing nodes in Godot is it's own thing
  // process hierarchies and lifetime
  // account for multithreading
  // account for pooling

GodotFileIo
  // via MegaCrit's `FileAccessStream`, which uses `Godot.FileAccess`

UserDataPathProvider
  // builds `user://*` paths

NodePool - the non-generic one, might have been a static class
  // static storage, see `NodePool<T>` for more
  // use for cards mostly, but quite universal

GitHelper
  // editor tool, used in absense of `ReleaseInfo`
```

flow handlers
```
// "System.Text.Json.SourceGeneration", don't bother much
// subclass of `JsonSerializerContext`
// for `JsonSerializationUtility`
MegaCritSerializerContext

// for `ProgressState`, `ProgressSaveManager`
DeserializationContext - traces fields

// for `CardSelectCmd` async logic
PlayerChoiceContext             - traces models
+ BlockingPlayerChoiceContext   - empty
+ ThrowingPlayerChoiceContext   - thrown an exception on signal
+ GameActionPlayerChoiceContext - ward GameAction (PlayCardAction, UsePotionAction)
+ HookPlayerChoiceContext       - ward GameAction (Hook, CombatManager, DrawConsoleCmd)

// for one, it's a convenient way to have runtime verification
// but also, the latter context allows scheduling players' actions
// e.g. drink a potion, but it will be applied only when their turn starts.
// so, I think they didn't want a message queue for actions, instead they
// opted to rely on C#'s async tasks system (ofc, superiority of the former is unknown to me,
// but for one thing cancelling tasks doesn't require removing queue elements - that's for sure)
// ADDENDUM: waaaait, there is a queue though ! it's `RunManager -> ActionExecutor -> ActionQueueSet -> actions`
```

reactions
```
NReactionWheel
  // choose a picture

NReactionContainer
  // shows local reaction, sends to network

ReactionSynchronizer
  // dispatch messages to network
```
