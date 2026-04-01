# source

- buy the game:      https://store.steampowered.com/app/1043810/Tactical_Breach_Wizards/
- download the tool: https://www.jetbrains.com/decompiler/

```
create an empty Unity project
export `Assebly-CSharp.dll` into "Assets"
be sure to create a root assembly for cleanliness
regenerate project files (maybe delete them first)
let Unity create project files - better than dotPeek ones
```


# patterns

- blackboard (centralizes shared data, making it accessible to all AI agents in a game)  
  https://school.gdquest.com/glossary/ai_blackboard  
  https://tonogameconsultants.com/ai-blackboard/  
  https://en.wikipedia.org/wiki/Blackboard_(design_pattern)
  https://en.wikipedia.org/wiki/Blackboard_system  
  see `DamagePlanner`, `HazardTileManager`

- type object (Allow the flexible creation of new “classes” by creating a single class, each instance of which represents a different type of object.)  
  https://gameprogrammingpatterns.com/type-object.html  
  see `ClassData`

- Utility AI  
  https://www.aiandgames.com/p/ai-101-introducing-utility-ai (https://www.youtube.com/watch?v=p3Jbp2cZg3Q)  
  https://en.wikipedia.org/wiki/Utility_system  
  see `PositionSelectorSO`, `MoveSelectorSO`, `TargetSelectorSO`

- prototype (blueprint objects can be cloned to create instance objects)  
  https://en.wikipedia.org/wiki/Prototype_pattern  
  https://refactoring.guru/design-patterns/prototype  
  https://gameprogrammingpatterns.com/prototype.html  
  see `AbilitySO` and its hierarchy, `Person` and its hierarchy, `PrefabSaveBehaviour`


# services

- https://trello.com/  
  Capture, organize, and tackle your to-dos from anywhere.


# third-party

- https://assetstore.unity.com/packages/vfx/shaders/flat-kit-toon-shading-and-water-143368  
  Flat Kit is a complete solution to achieve the stylish cel-shaded look: shaders, models, image effects, presets, examples.

- https://gist.github.com/darktable/1411710  
  Unity3D: MiniJSON Decodes and encodes simple JSON strings. Not intended for use with massive JSON strings, probably < 32k preferred. Handy for parsing JSON from inside Unity3d.

- https://assetstore.unity.com/packages/vfx/shaders/fullscreen-camera-effects/mk-glow-bloom-lens-glare-90204  
  MK Glow simulates bright surface light scattering, creating an uniquely stylized glow with customizable bloom, lens surface, flare, and glare effects. Perfect for adding distinctive visual flair.

- http://staggart.xyz/unity/stylized-water-shader/documentation/  
  N.B.: there's a more moder one at https://assetstore.unity.com/packages/vfx/shaders/stylized-water-3-287769  
  Focuses on providing artistic freedom, rather than a realistic water simulation. Spans a wide range of applications requiring water. Easy to use and self-documented


# code architecture

```
game actively operates "instance ids" - a sequential counter,
provided by `SaveManager -> GetNextInstanceID`,
and saved alongside serializable objects

lots of pluggable objects: type, data, behavior

lots of smart and useful decisions, lots of OOP crimes, and one great game
```


# gameplay: entities

```
// these are monobehaviours

+ SaveableGameplayObject : ISaveableComponent

  + MildfireTile
    + ThornTile

  + TransferrenceObject

  + ActiveObject : ISaveableComponent
    + TargetableObject

      + ThrowableCoverTarget

      + ExplosiveBarrel
        + ServerObjective
        + DestroyableObjective
        + ExplodingPlant

      + NeutraliserField
      + StunCoffin
      + StunCoffinProxy
      + TurretCorpse

      + Person
        > classSO : ClassData
          > moveSelector : MoveSelectorSO
        > abilityInstanceList : list of `AbilitySO`

        + NPC
          + PrologueHostageNPC
          + LivKennedyNPC

        + Enemy - opponents (unless a status effect says otherwise)
          + Corpse
          + LivKennedyBoss
            > takeCoverSelector : MoveSelectorSO

          + Drone
          + Turret

        + Wizard - allies (unless a status effect says otherwise)
          + BadZan
          + DruidWizard
          + FlashbackNavySeer
          + PoisonedWizard
            + PoisonedSeer

    + AreaShield
      + ShieldDrone

    + Throwable
      + Grenade : IActionQueueEntry
        + SiegeMissile
        + CrowdGrenade
        + GaleGrenade
        + SedativeGrenade
        + ThrowingTapePlayer
        + TimeBoostGrenadeObject

      + ThrowableCover : IActionQueueEntry
      + DruidPlantGrenade

  + Interactible

    + CommentaryTape : PlaceableOnCover
    + DeathsDoorInteractible
    + ShieldGenerator
    + SignalFlare
    + TakeableCoverInteractible
    + ToggleThrowableCoverHeight
    + VaultBreachGlyph
    + VaultManaStash

    + ControlTerminal : ISaveableComponent
      + UnlockFriendlyDoorTerminal
      + TurretControlsTerminal

    + DeadDrop
    + DefusableMissile : ISaveableComponent
    + Door

    + EnemyDoor : IActionQueueEntry, InfoPanelMouseOver
      + Helicopter
        > moveSelector : PositionSelectorSO

    + Intel
    + NeutraliserFieldInteractible
    + TurretInteractible

  + SecurityDoor : ISaveableComponent, IDoor, InstantiateArtCallback
    + Gangway

// all right. THIS place could have used some share of composition

also there is a `CharacterManager -> characterMap`
  // represents each of unique `ClassData` of
  // each `Person` in `SaveManager -> saveablesList`
```


# gameplay: abilities

```
// these are scriptable objects

+ AbilitySO
  > targetSelector : TargetSelectorSO
  > highlightPositions : static hashset of `GridPosition` (XZ)
    // a solution that just works =)
  // it's a CMS (Content Management System) in itself.
  // visual data ? it's here ! data model ? also there !
  // code, behaviour, access - whatever might come handy.
  // team fat struct says hello: almost all params are present
  // but zeroed (or minus-oned) out as a default value, which
  // processing logic will silently consume with no errors 

  + AbsorbMildfire

  + AbstractThrowDrone
    + ThrowDrone
    + ThrowShieldDrone

  + BeamAttack : ILineOfSightAbility
    + DruidAcidDart
    + PullAttack
    + WandShot

  + BlinkOutSurvive
  + Broom
  + CastMildfireInstant
  + CenserSmash
  + Charge
  + CorrosiveVine

  + CustomChainLightning : ILineOfSightAbility
    + ChainPull

  + DogBecomeHuman
  + FalseProphet
  + HellClamp
  + Incenirate
  + LeaveCover
  + MeleeOverwatch

  + Move
    + DogPounce
      + DogInterBite

  + PredictedAbilitySO
    + Abduct
    + ChaseMelee : IReactionAbility
    + ConditionBeam
    + CrushHour
    + LivChainShock

    + Overwatch
      + ConeOverwatch
      + SeerOverwatch

    + PredictedCone
    + PredictedDamageZone
    + PredictedDroneActions
    + PredictedMedicAttack

    + PredictedMeleeAttack
      + ShotgunDash

    + PredictedShootWizard : ITargetVariableDamage, IReactionAbility
      + LivQuickshot
      + MedicBeam
      + PredictedShootFromCover
      + HeavyGunnerShot

    + PredictedThrowInstantGrenade
      + CastMildfire
      + ChapelMortar
      + RocketShot

    + SoundAlarm
      + PlantBomb

    + TwitchShootWizard : IReactionAbility

  + Reposition
  + Resurrect : ILineOfSightAbility
  + SayALine
  + SeerShot : IReactionAbility, ILineOfSightAbility
  + ShootWizard
  + SpectralKevlar
  + StealMana : ILineOfSightAbility
  + StormSpell
  + StunShot

  + Swap : ILineOfSightAbility
    + LivSwap

  + TakeCover
  + ThrowProximityGrenade

  + ThrowThrowable : ILineOfSightAbility
    + ThrowCover
    + ThrowGrenade
      + LaunchMissiles : IReactionAbility
      + ThrowInstantGrenade
        + ThrowCorpseGrenade
        + ThrowCrowdGrenade
        + ThrowPrologueGrenade
        + ThrowTapePlayer

  + TimeBoost : ILineOfSightAbility
  + Transference

  + UseInteractibleSO
    + BlinkOutPrologue
    + BreachDoor
    + DeathsFloor
    + SealDoor
    + ToggleCoverHeight
    + UseIntel
    + UseObjective

  + WitchPunch

  + WizardPreciseAbility
    + DeathsDoor
    + GhostShot

// I'm not tired saying the word "interesting..."
// but abilities include non-cimbat and meta-combat actions
```


# gameplay: perks

```
// these are scriptable objects

+ CharacterPerk
  > ApplyPerk
    // abstract, implementation defined
  > AddPerkToWizard
    // a unique instance only
    > run `ApplyPerk`
  // and a multitude of subclasses, implementing the logic
  // mostly it's numerical buffs, plus some markers and temporary abilities
```


# gameplay: AI

```
see https://youtu.be/gEFfBXQAtYE for more context
  // "The brains are in the grenades: how enemies think in Tactical Breach Wizards"

DamagePlanner
  // blackboard memory
  // everything here lives for one frame only, then reset
  > damageAssigned : maps `Person` id to `int` damage
  > peopleTargeted : maps `AbilitySO` id to hashset of `Person` ids
  > wizardIDs      : cache of `Lists -> Wizards`
  // used by:
  // - PredictedAbilitySO      -> AssignPredictedDamageToTarget -> AssignDamageToPerson
  // - TwitchShootWizard       -> AssignPredictedDamageToTarget -> AssignPersonTargeted
  // - LowestRemainingHealth   -> RankItem                      -> GetDamageAssignedToPerson
  // - PreventTargetAllWizards -> RankItem                      -> LastWizardUntargeted

HazardTileManager
  // blackboard memory
  > hazardousTiles         : hashset of `GridPosition`
  > secretlyHazardousTiles : hashset of `GridPosition`

Priority<T>                           - for `GridPosition` or `Person`
  > RankItem
    > evaluate utility of the "action"
  > FilterFrom
    > run `RankItem` for each `input : T`
    > store into `results : T, int rank`
  // and a multitude of subclasses, implementing the logic

PositionSelectorSO -> for `GridPosition`
  // utility-based decision maker
  > CreateSelector : PriorityListSelector
    > add `ClearGroundPriority`
    > instantiate and add all of `priorityOrder : MovePriorityNames`
  // used by:
  // - Helicopter -> moveSelector

MoveSelectorSO -> for `GridPosition`
  // utility-based decision maker
  > CreateSelector : PriorityListSelector
    > add `CanMovePriority`
    > instantiate and add all of `priorityOrder : MovePriorityNames`
  // used by:
  // - LivKennedyBoss -> takeCoverSelector
  // - ClassData      -> moveSelector

TargetSelectorSO -> for `Person`
  // utility-based decision maker
  > CreateSelector : PriorityListSelector
    > add `CanTargetFromCurrentPosition`
    > instantiate and add all of `priorityOrder : TargetPriorityNames`
  // used by:
  // - AbilitySO -> targetSelector

// so, these three selectors are scriptable objects that offer the means
// to create different behaviours as data and plug them into entities;
// Tom Francis calls them "brains", which they are in fact

PriorityFactory
  // via reflection, builds instances of `Priority<T>`
  // - using `static Name : MovePriorityNames`   and `generic T : GridPosition`
  // - or    `static Name : TargetPriorityNames` and `generic T : Person`

PriorityListSelector<T> - for `GridPosition` or `Person`
  > bestOptions : IEnumerable<T>
  > FindBestOptions -> for `Person`, `AbilitySO`, `input : T`
    > run `Priority<T> -> FilterFrom`
    > store into `bestOptions : IEnumerable<T>`
      // actually it's nothing more than `bestOptions = priority`
```

# gameplay: rewind

```
"RewindButton -> on click" or "InputManager -> on rewind" 
  > `TurnManager -> rewindSignal` = true

TurnManager
  > "WizardTurn -> Rewind" or "UnpredictedEnemyTurn -> Rewind"
    > if gameover -> `SaveManager -> LoadStartOfLastWizardTurn`
    > else        -> `SaveManager -> LoadLastMoveState`

// surprisingly, everything is backed on disk immediately, no in-memory for undos
```


# save system

```
+ ISaveableComponent
  // literally save/load contract for MonoBehaviours
  // lots of boxing for primitives

+ IDataBlockRecorder
  + FileWriter
    // custom XML state machine
  + InMemoryDataBlockStore
    > dataBlocks : list of `DataBlock`

DataBlock
  > dataPairs : map of key to (key : value)
  > dataLists : map of key to (key : value1, value2, value3, ...)

+ AbstractSaveBehaviour
  > saveables : array of `ISaveableComponent`
    // cached on awake
  > Save with `IDataBlockRecorder`
  > Load with `DataBlock`

SaveManager
  // the core save/load system

// again. everything works. but it's a bit worrying amount of OOP here.
// kudos for the released game. it's me who needs to adjust his optics probably
```


# misc

utilities
```
Dev
  // a patchwork quilt of helper functions, quite often load-bearing ones
  // - physics requests, that determine line of sights
  // - world requestsm, that determine existence of entities
  // - zip a file, format a string, log, UI navigation
  // everything

Lists
  // world of everything in game runs
  // who need ECS filters when you can
  // - OnEnable  add to list
  // - OnDisable remove from it
  // and it's not sarcasm. this code GSD, at least for this particular
  // turn-based game. although, I am convinced that such a solution not
  // so rare in the wild, especially if you are indie

Mode
  > modes : enum Modes
  // stores and somewhat manages current game state
  // will be useful for navigation in code, btw

Managers
  // statically cached objects
  // most of them are scene-based
  // some of them are plain classes

// who needs dependency injection frameworks when we have unity at home
// although I can admit, that some of the just feels wasteful, too garbage producing,
// but probably in the grand scheme of things and given it's not a bottleneck,
// it's better to just turn a blind eye

MetaMenu
  // manages scenes, presumably
  > calls `Managers -> FindManagers`
```
