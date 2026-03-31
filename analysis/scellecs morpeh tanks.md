# source

- https://github.com/scellecs/morpeh.examples.tanks


# UX and flavour

```
feels
  [neg] KB's diagonal movement is composite, and more often than not, tank resets to 4 cardinal directions on stop
```


# morpeh ECS

```
// it's cools that `morpeh` devs profiled performance and opted to
// surgically replace `List<T>` and `T[]` with faster custom containers
// by either replacing lists with arrays or GC-pinning memory. the latter,
// I suppose is beneficial in terms of reducing GC-load in general, given
// a potentially large amount of data being juggled each frame

// so far I see multiple approaches for implementing ECS events
// - component-based posting (tanks' `IComponent`s)
// - variable-based posting (pong's `GlobalEvent`s)
// both have upsides and downsides. generally speaking, nothing new
// for me here, but it's still funny to encounter the same ideas

// initialization approaches
// - static          : do it at authoring time in `MonoProvider`s
// - dynamic passive : do it in code at initialization
// - dynamic active  : do it in code at update
// the third one clearly shows the lack of events for state mutation
// indicating changes in the lists of entities and components;
// such interfaces do not need to be reactive, it's enough to
// trigger events in an orderly manner: say, before all ticks

// what I do like is CMS (Content Management System) based on `ScriptableObject`s and `MonoBehviour`s
// at the same time the lack of edit-time inspector (there's is runtime World inspector already) is
// quite prominent for me. probably the dire need for one for experienced decelopers is less notable
// one could argue, but as by me, it's a misconception. such a tool will help with onboarding greatly,
// and be an immense time saver for content creators and people who debug systems in general
```


# architecture

code
```
- Bases -
  TeamBaseInitSystem
    -> for TeamBase & InTeam & ~InitializedMarker
       > init `InitializedMarker` and update visuals

  TeamBaseDestroySystem
    -> for TeamBase & InTeam & IsDeadMarker
       > mark with `LosingTeamMarker` and update visuals

  TeamBaseProvider with `TeamBase`
    // links team's base entity with the team entity
    > add `InTeam`


- Collisions -

  CanCollide     - an `IComponent`
  CollisionEvent - an `IComponent`

  CollisionCleanSystem -> for CollisionEvent
    // automatically cleans them on late update
    // so, yeah, note to self, it's a convenient way to reliably
    // perform cleanups for otherwise infinitely stacking events

  CollisionDetector
    // engine-facing unity component, catching `OnCollisionEnter2D` events
    > add `CollisionEvent`

  CollisionInitSystem
    -> for (Tank | Bullet | Wall | TeamBase) & ~CanCollide
       > init `CanCollide`
    // so... instead of manually add this component
    // upon init, this system centralizes


- GameInput -

  ControlledByUser - an `IComponent`
  GameUser         - an `IComponent`

  GameInputSystem
    > tick `InputSystem`
      // @note set to manual processing
      // `InputActions` codegen was used
    > create `GameUser`
      // when a device is paired


- Healthcare -

  DamageEvent  - an `IComponent`
  IsDeadMarker - an `IComponent`

  DamageCleanSystem -> for DamageEvent
    // automatically cleans them on late update

  DamageSystem
    -> for Health & DamageEvent
       // tracks and updates hitpoints
       > add `IsDeadMarker` (isn't used here as a filter though)

  DamageTextSystem  -> for Health & DamageEvent
    // does double job: yes, it shows damage, but for some reason
    // checks for the presence of `IsDeadMarker` and changes the text;
    // as per me, might have been `DamageSystem` part or an event
    > send `TextInWorldSystem.Request` component-event

  HealthProvider with `Health`
    // umm, might have been a pure component, there's no
    // engine-facing parts here; either this, or `CanCollide`
    // should be setup via a provider too

  TankDestroySystem -> for Tank & IsDeadMarker
    > dispatch `OneMoreKillEvent` component for the damage dealer
    > remove `UserWithTank` if was `ControlledByUser`
    // actually also removes the corresponding entity and GO


- Movement -

  MoveDirection - an `IComponent`

  MovementSystem
    -> for Tank & MoveDirection
       > update position and rotation

  TankMovementInitSystem
    -> for Tank & ~MoveDirection
       > init `MoveDirection`

  UserMovementSystem
    -> for MoveDirection & ControlledByUser
       > marshall input


- Scores -

  OneMoreKillEvent - an `IComponent`
  UserScores       - an `IComponent`

  ScoreSystem
    -> for GameUser & ~UserScores
       > init `UserScores`
    -> for OneMoreKillEvent & ControlledByUser
       // again, the disadvantage that I have known since 2013 with my personal clunky ECS framework
       // naive components-are-events approach restricts developer with one-per-tick type of event max
       > increment score
       > send `TextInWorldSystem.Request` component-event


- Teams -

  InTeam           - an `IComponent`
  LosingTeamMarker - an `IComponent`

  GameUserBalanceTeamSystem
    -> for GameUser & ~InTeam
       > find weakest `Team`
       > init `InTeam` with it

  TeamProvider with `Team`
    // spawn points among other things

  TeamUserIdSystem
    -> for Tank & ControlledByUser & ~UserIdText
       > init `UserIdText`

  WinDetectSystem
    -> for Team & ~LosingTeamMarker
       > init `WinMarker` for the single winner when applicable
       > send `TextInWorldSystem.Request` component-event


- UtilSystems -

  TextInWorldSystem
    -> for TextInWorld
       > animate and the remove `TextInWorld`
    -> for Request
       > spawn visuals and init `TextInWorld`


- Walls -

  WallDestroySystem
    -> for Wall & IsDeadMarker
       > remove the corresponding entity and GO


- Weapons -

  Bullet - an `IComponent`

  BulletHitSystem
    -> for CollisionEvent
       // first is expected to be InTeam & Bullet
       // second is expected to be InTeam
       > dispatch `DamageEvent` to the second
       > remove the corresponding entity and GO for the bullet

  BulletWeaponProvider with `BulletWeapon`
    // configs

  BulletWeaponSystem
    -> for BulletWeapon & Tank
       > with a cooldown, on input, spawn `Bullet`
       > mark it non-colliding with the tank itself

  UserBulletWeaponSystem
    -> for BulletWeapon & ControlledByUser
       > marshal input


- %root% -

  UserWithTank - an `IComponent`

  GameUserTankCreateSystem
    -> for GameUser & ~UserWithTank
       > init `UserWithTank` with a spawned tank
         > init `ControlledByUser`
         > init `InTeam`

  PhysicsUpdateSystem
    > tick `Physics2D`
      // @note set to manual processing

  TankProvider
    // config


- Tests -
  + EcsTestFixture
    + CollisionsTests
    + HealthTests
    + MovementTests
    + TeamTests
    + BulletWeaponsTests

// basically unit and integration checks

// typical process is as expected:
// - create entities
// - run systems
// - check results
```

data
```
// Scenes
  - "BattleScene"
    - "BlueTeam" : EntityProvider, TeamProvider
      - "Base"   : EntityProvider, TeamBaseProvider, HealthProvider
    - "RedTeam"  : EntityProvider, TeamProvider
      - "Base"   : EntityProvider, TeamBaseProvider, HealthProvider

// Tanks.Content/Destroyable
  - "BrickWall" : EntityProvider, WallProvider, HealthProvider
  - "TeamBase"  : EntityProvider, TeamBaseProvider, HealthProvider

// that's interesting, this example uses standalone `EntityProvider` script
// for it to be a cenral entity source for the game object. pong example
// was content with only using a set of `MonoProvider` subclasses

// Tanks.Content/Tanks
  - "TankRepository" - SO data, a content provider, basically
  - "PzIV/PzIV"      : EntityProvider, TankProvider, HealthProvider, BulletWeaponProvider
  - "PzIV/PzIVConfg" - SO data, referenced by `TankProvider`

// Tanks.Content/Weapons
  - "Cannon/Cannon"       : BulletWeaponConfig - SO data, referenced by `BulletWeaponProvider`
  - "Cannon/CannonBullet" : BulletConfig       - SO data, referenced by `BulletWeaponConfig`
  - "Cannon/CannonBullet"                      - referenced by `BulletConfig`

// Tanks.Systems
  - "[Installer] Main"
  - "Bullet Hit System"
  - "Bullet Weapon System"
  - "Clean Damage System"
  - "Collision Clean System"
  - "Collision Init System"
  - "Damage System"
  - "Damage Text System"
  - "Game Input System"
  - "Game User Balance Team System"
  - "Game User Tank Create System"
  - "MovementSystem"
  - "PhysicsUpdateSystem"
  - "Score System"
  - "Tank Destroy System"
  - "Tank Movement Init System"
  - "Team Base Destroy System"
  - "Team Base Init System"
  - "Team User Id System"
  - "Text In World System"
  - "User Bullet Weapon System"
  - "User Movement System"
  - "Wall Destroy System"
  - "Win Detect System"

// difference to the pong example: no global events whatsoever
```
