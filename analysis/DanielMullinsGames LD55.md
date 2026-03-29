# source

- https://github.com/DanielMullinsGames/LD55


# scenes flow

- Title   -> Intro  
  "VOIR DIRE"

- Intro   -> Story_1  
  cutscene

- Story_* -> Main  
  5 cutscenes

- Main    -> Story_*  
  gameplay with 4 defendants


# UX and flavour

```
feels
  responsive cursor
  ability to fast-forward
  drag'n'drop is satisfying
  [neg] juror's cards are shining, but "Go to trial" banner does not; a bit misleading

looks
  glorious ms paint art
  stylized cursor
  animations, shines

environment
  bloom, chromatic aberration, film grain, monochrome
  minimal reusable elements

audio
  exists, suits
```


# gameplay

```
you have 5 trial rounds
  mock 1, mock 2
  pre 1,  pre 2
  final

phase: meta
  defendant #0               - tutorial, hard to lose
  defendant #1               - rounds 2 and 4, kill the first juror
  defendant #2               - after round 1,  spawn "innocent" juror
  defendant #3               - need to prove "innocent"
phase: buy
  shuffle available lineup   - might find a better match
  buy jurors to the bench    - have their own principles
  sell jurors from the bench - your need money after all
  rearrange jurors           - some of them affect or are affected by others
phase: trial
  process    - left-to-right
  cast votes - innocent if "none | innocent | uncertain"
  after vote - triggers all the actions

defendats
  different number of votes per round

jurors
  > disposition         - ...
    none                - only happens if bought slime first
    guilty              - 
    innocent            - 
    unceritain          - 
  > decision method     - ... // might have been merged with disposition, replacing "uncertain"
    oppostite to left   - none is none, innocent if "guilty | uncertain"
    same as left        - 
    pro majority        - guilty if # of "guilty" >= "innocent"
    contre majority     - guilty if # of "guilty" <  "innocent"
  > after vote          - ...
    left is innocent    - 
    extra vote          - 
    double activate     - trigger "after vote" for the jurors to the left
    switch all          - innocent if "none | guilty | uncertain"
    gain cash           - 
    left is guilty      - 
    force left vote     - 
  > special             - ... havs a diverse set of triggers
    dies                - 50% chance
    can't sell          - 
    extra sell          - 100% more cash
    summon skeleton     - 
    dies always         - 
    clone               - become same as left-most juror
```


# components

```
AudioController
  > autoload SFX and music loops from resources
  > handle crossfades, spawns SFX sources

ManagedBehaviour - actual purpose "pausable behaviour"
  // ok, it's the origin of the idea, from what I've heard
  // convnient, but generally wastefull; would be better to
  // have a singular engine-facing manager object which
  // dispatches such call to its subscribers; still, it's a
  // game jam, the author has a ton of steam-released titles,
  // which hints us, that it works to - you need to make games
  // first of all, not an ideal archtecture without application

ReferenceSetToggle - used for pause, disabling input, etc, but not in this game
  // kinda a bool value, but driven by the fact of
  // objects adding and removing themselves into its
  // internal hashmap

SineWaveTransform, SineWaveMaskCutoff
  // basically tweener, flovours game with subtle animations

GameCursor
  // hell yeah, allocating raycasts each frame and then
  // even more garbage; aaaaaanyway... it just works (tm)
  > tracks current and previous `Interactable`, sends input
  > hides and locks the system cursor
  > draws a sprite in its place
  > animates it on clicks

+ Interactable
  + DraggableInteractable
    + JurorInteractable
  > gets input from `GameCursor`
  // there's no universal drag'n'drop system though,
  // all instances are responsible to be dropped somewhere

SceneTransition
  // automates fade in and fade out animations

ResolutionOptions
  // it's a fullsceen toggle with a text hint
  // hardcoded to the `F` key; probably to not
  // conflict with the browsers' `F11` ?

FastForwardButton
  // toggles time scale

GameFlowManager
  > defendants         - list of `Defendant`

Defendant
  > guiltPhases - list of int
    // represents number of required votes

JurorData - scriptable object
```


# architecture

```
doesn't use `Canvas` - everything is in world-space

need a button ? use an `Interactable`

coroutines a heavily used for sequencing

some data, some hardcode
```


# prefabs

```
"CameraRig" prefab
  - parameters and postprocessing

"AudioController" prefab
  // only partially responsible for the game audio, albeit mostly
  - controlling script `AudioController`
  - a set of audio sources for loops (i.e. non-SFX)
  - an audio listener

"Background" prefab is a combination of
  - a couple of crossed hatch-pattern sprites and a mask
  - sine wave animations

"Cursor" prefab
  - controlling script `GameCursor`
  - visual replacement game object with a sprite

"Transition" prefab is a combination of
  - controlling script `SceneTransition`
  - white-black perlin noise texture
  - cutoff animation
```


# scene: Title

"CameraRig", "AudioController", "Background", "Cursor", "Transition", "ResolutionOptions"

"Start" with `StartScreen`
```
ManagedUpdate
  with an initial delay, after a click
    load scene "Intro"
```


# scene: Intro

"CameraRig", "AudioController", "Cursor", "ResolutionOptions"

"TextScroll" with `IntroScene`
```
ManagedUpdate
  animate
  load scene "Story_1"
```


# scene: Story_*

"CameraRig", "AudioController", "Transition", "Cursor"

"ResolutionOptions" for the #1

"StoryScene" with `StoryScene`
```
ManagedUpdate
  animate
  load scene "Main" or quit
```


# scene: Main

"CameraRig", "AudioController", "Background", "Cursor", "Transition"

"FastForwardUI" with `FastForwardButton` and `ResolutionOptions`

"GameFlowManager" with `GameFlowManager`
```
Start
  launch and forget `GameSequence` coroutine
    run `DefendantIntroSequence` coroutine - cutscene
    run `DefendantSequence` coroutine      - gameplay
      5 rounds of (meta, buy, trial)
      run `SentencingSequence` coroutine
        animate, then load scene "Story_*"
```

"Bench/Jurors" with `BenchArea`
```
ManagedUpdate
  UpdateJurorOrder     - update array of jurors
  UpdateJurorPositions - update positions of jurors
```
