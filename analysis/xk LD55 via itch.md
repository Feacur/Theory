# source

- https://xk.itch.io/summoning
- code is not of the release version


# scenes flow

- title     -> interlude  
  "SUMMONING"

- interlude -> main  
  "100 years later"

- main      -> victory or main  
  gameplay

- victory   -> title  
  statistics


# UX and flavour

```
feels
  look offset for camera
  telegraphing attacks
  [neg] automatic and manual sword modes seem to interfere; a bit clunky

looks
  glorious ms paint art
  tilt vampire when walking
  floating, breathing, blinking
  blob shadows, trails
  VFX, damage numbers

environment
  bloom, color grading, vignette
  floor decals, fog

audio
  exists, suits
```


# gameplay

```
kill and survive         - touching, different projectiles
get updgrades            - HP, dash, damage, speed, blood, unique
upgrade types            - blood-based, grimoires, chests
synergies                - attack dash, speed scales damage, combo hits, etc
spawning enemies         - regular, reinforced, mini-bosses, final boss
leaderboard and tooltips - release version has more

notes
  moves transforms
  doesn't use physics
  checks 2D distances manually
```


# CMS

```
G - static
  > generic -> list of `IGameSystem`
  > cmd     -> gets `Cmd`
  > vampire -> gets `VampireMain`
  > Init    -> clear "generic"
  > Add     -> adds `IGameSystem` to "generic"

IGameSystem
  > OnGameStarted - a broadcast message

CMS - static
  > all : CMSTable
  > Init    -> auto init all of `CMSEntity` subclasses via reflection
  > Get     -> finds `CMSEntity` by its id inside "all"

CMSEntity
  // its id is `Type -> FullName`, probably statically cached
  > Define  -> adds `EntityComponentDefinition` to "components"

ManagedBehaviour
  // abstract wrapper
  > Start         - regular unity function, invokes `Init`
  > Init          - virtual function
  > Update        - regular unity function, invokes `ManagedUpdate`
  > ManagedUpdate - virtual function, invokes `ManagedFixedUpdate`
    // umm, not so fixed, but whatever

Cmd - implementation for `IGameSystem`
  > commandQueue -> list of `ICommand`

ICommand
  > Execute - coroutine

ManagedCommand - abstract implementation for `ICommand`
  > `Execute` - regular coroutine function, invokes `OnTick`
  > `OnTick`  - virtual function
    // good, base class defines desired behaviour

AudioSystem - implementation for `IGameSystem`
  > Play    -> get`CMSEntity` by its id, then act accordingly
```


# scene: title

"Main" with Main`
```
Start
  CMS.Init
  add system instance  `ScreenFader`
  add system instance  `ScreenShake`
  add system instance  `AudioSystem`
  add system prototype `Cmd`
```

"cutscene" with `IntroCutscene`
```
Start
  animate blademan
  animate cultists
  animate blades with `LaunchBlade`
  add command `CmdScreenShake`
  add command `CmdLoadScene` "interlude"

LaunchBlade
  `AudioSystem -> Play` SFXWoosh
  animate blade
  `AudioSystem -> Play` SFXImpact
```


# scene: interlude

"Main" with `LoadNextScene`
```
sceneName : string

Start
  CMS.Init
  add system instance  `ScreenFader`
  add system instance  `AudioSystem`
  add system prototype `Cmd`
  broadcast `OnGameStarted`
  add command `CmdWait`
  add command `CmdScreenShake`
  add command `CmdLoadScene` sceneName
```

"text" with `ShakeTextAnimation` and `TextWriter`

# scene: main

```
UpgradeList
  upgrades - list of `VUpgrade`

player have autosword by default
```

entities
```
+ VampireDamageReceiver
  + VampireBoss
    ManagedFixedUpdate
      handle shooting `VampireProjectile`
  Init               - register enemy
  OnDestroy          - unregister enemy
  ManagedFixedUpdate
    handle sword damage
    initially enemies could have targeted pentagrams
      // pentagram health is vampire health
    handlde movement
    damage vampire or pentagram

VampireProjectile
  ManagedFixedUpdate
    handlde movement
    damage vampire or pentagram
```

"main" with `VampireMain`
```
upgrades : UpgradeList

Start
  CMS.Init
  add system instance  `ScreenFader`
  add system instance  `AudioSystem`
  add system instance  `ScreenShake`
  add system prototype `Cmd`
  add system instance  this
  broadcast `OnGameStarted`
  `AudioSystem -> Play` BackgroundMusic
  add upgrade "Smartblade"

Update
  track playtime
  UpdateUI
    reloads main on click in gameover state
  handle pause
  handle final boss spawn (triggered by the amount of collected blood)
  handle input (movement, dash, cheats)
  switch between vampire and sword states
    launch a shockwave if have upgrade

FixedUpdate
  update game events (array of timed entries)
  tick perks (damaging radiance)
  tick vampire and sword, switch states
    // initial idea seems to control only one of them at a time, but was discarded
  spawn regular enemy followers on the edges
  collect blood
  tick followers
```

# scene: victory

"Text (TMP) (1) with" with `TimeView`
```
  Start
    CMS.Init
    add system instance  `ScreenFader`
    add system prototype `Cmd`
    broadcast `OnGameStarted`
    setup text

  Update()
    on click
      add command `CmdLoadScene` "title"
```

and a bunch of text
