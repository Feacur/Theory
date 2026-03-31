# source

- https://github.com/scellecs/morpeh.examples.pong


# UX and flavour

```
feels
  [neg] holding LMB for no reason is clunky
```


# morpeh ECS

```
+ ScriptableObject
  // seems to be the load-bearing engine feature for the whole framework...

  + BaseSingleton
    + Singleton
    + BaseGlobal
      > implicit operator bool => instance != null && instance.IsPublished;
        // a bit weird, explicitly checking the property would be clearer

      + BaseGlobalEvent<TData>
        + GlobalEventBool
        + GlobalEventFloat
        + GlobalEventInt
          + GlobalEvent
            > Publish
              // mark as invoked
              // N.B. can be called from UI too
        + GlobalEventObject
        + GlobalEventSceneReference
        + GlobalEventString

  + Initializer
  + UpdateSystem
  + CleanupSystem
  + LateUpdateSystem
  + FixedUpdateSystem


+ MonoBehaviour
  // ... well, unless you need to interface with the scene hierarchy, of course

  + WorldViewer
    + BaseInstaller
      + Installer
        // registers systems and stuff

  + EntityProvider
    // entity is `gameObject.GetInstanceID`
    > map          : static collection of refcounted `Entity`
    > cachedEntity : Entity
    > OnEnable
      > invoke `CheckEntityInitialization`
        > caches "cachedEntity"
        > from "map" or `World.Default.CreateEntity()`
          > increment "refcount" if needed
      > invoke `PreInitialize`
    > OnDisable
      > invoke `PreDeinitialize`
      > decrement "refcount"
        > remove "cachedEntity" from "map" if needed

    + MonoProvider<T>
      // the data is fetched as a `ref`, so modifications would still be posible
      > serializedData : T
        // local copy of the data
        // used only as a fallback (prefab mode, missing entity, etc)
      > stash          : Stash
        // reference to the world's components data and some statics
      > OnValidate
        > caches "serializedData"
      > PreInitialize
        > puts "serializedData" into "stash" for "cachedEntity"
      > PreDeinitialize
        > removes "cachedEntity" from the "stash"
        // which clues us, that each "stash" stores this type of components;
        // although, no need to be a detective really, the doc says so:
        // - Components are stored in "stashes", which are also known as "pools" in other frameworks.

// even though there is an option to create multiple worlds, statically only `Default => [0]` is being used
```


# architecture

code
```
- UI - 
  CanvasGroupSwitcherProvider with `CanvasGroupSwitcher`
    - "Switch Game State Event" + field
  GlobalIntToTextProvider with `GlobalIntToText`
    - "[Scores] Current Scores" + field + format
    - "[Scores] High Scores"    + field + format
  SliderToColorProvider with `SliderToColor`
    - "[User] Ball Color"       + field

// OK. That's how UI is decoupled from the logic and vice versa:
// create a global object, write data into it, let a view read it

- State -
  GamePauseSystem
    - on "Switch Game State Event", toggle time scale
  GameQuitSystem
    - on "Quit Game Event", exit play mode
  KeyToEventSystem
    // on `ESC`, publish "Switch Game State Event"

- Scores -
  HitScoreCounterProvider with `HitScoreCounter`
    > OnCollisionEnter2D
      > set "hit" in "ref GetData" (true)
  ScoreSystem
    > OnUpdate
      > on "Reset High Scores Event", reset "[Scores] High Scores"
      > on "Ball Is Out Of Screen Event", decrease "[Scores] Current Scores"
      > for each of `HitScoreCounter`
        > increment "[Scores] Current Scores" if `HitScoreCounter -> hit` set
        > update "[Scores] High Scores" if needed

- Paddles -
  AlignToScreenSideProvider with `AlignToScreenSide`
  AlignToScreenSideSystem
    > OnAwake
      > create "walls" `Filter` for `AlignToScreenSide`
      > initialize other dependencies
    > Update
      > for each entity in "walls" - snap
  PaddleProvider with `Paddle`
  PaddleSystem
    > OnAwake
      > create "paddles" `Paddle` for `AlignToScreenSide`
      > initialize other dependencies
    > Update
      > for each entity in "paddles" - move with mouse
        // kinda conflicts with `AlignToScreenSideSystem`
        // depending on the parameters set up in providers
        // should have used `AlignToScreenSide` for that
        // see patches section at the bottom

- Balls -
  BallOutOfScreenCheckSystem
    // publishes "Ball Out Of Screen Check System" (seems unused)
    // reset ball position (likely initially was triggered by an event)
  BallProvider with `Ball`
    > OnCollisionEnter2D
      > set "hit" in "ref GetData" (position, normal, entity)
  BallVelocitySystem
    > Update
      > on "hit"               - reflect
      > on stop                - start moving
      > on "[User] Ball Speed" - set speed, keep direction
  ColorizedProvider with `Colorized`
    > "[User] Ball Color"
  ColorizedSystem
    > OnUpdate
      > set fields' colors
        // might have checked for "[User] Ball Color" being published
        // irrelevant to simple cases, but keep it in mind though
        // see patches section at the bottom
```

data
```
// Pong.Content/Scenes
  - "GameScene"
    - "Ball" : BallProvider, ColorizedProvider, HitScoreCounterProvider

// Pong.Content/Prefabs
  - "Paddle"    : PaddleProvider, AlignToScreenSideProvider
  - "PauseMenu" : CanvasGroupSwitcher, SliderToColor
  - "Scores"    : GlobalIntToTextProvider

// Pong.Globals/Events
  - "Ball Is Out Of Screen Event"
  - "Quit Game Event"
  - "Reset High Scores Event"
  - "Switch Game State Event"

// Pong.Globals/Variables
  - "[Scores] Current Scores"
  - "[Scores] High Scores"
  - "[User] Ball Color"
  - "[User] Ball Speed"

// Pong.Systems
  - "[Installer] Gameplay"
  - "Align To Screen Side System"
  - "Ball Out Of Screen Check System"
  - "Ball Velocity System"
  - "Colorized System"
  - "Game Pause System"
  - "Game Quit System"
  - "Key To Event System"
  - "Paddle System"
  - "Score System"

// Pong.Systems/UI
  - "[Installer] UI"
  - "Canvas Group Switch System"
  - "Global Int To Text System"
  - "Slider To Color System"
```

# patch: use `AlignToScreenSide` instead of `xAxis / yAxis`

```
diff --git forkSrcPrefix/Assets/Pong/Paddles/PaddleSystem.cs forkDstPrefix/Assets/Pong/Paddles/PaddleSystem.cs
index 990b4f8d1c9600886cca1ebb7e04d2db81b31a97..d70c477cfc8df4c4648d5f70f088d139371be4e2 100644
--- forkSrcPrefix/Assets/Pong/Paddles/PaddleSystem.cs
+++ forkDstPrefix/Assets/Pong/Paddles/PaddleSystem.cs
@@ -1,4 +1,5 @@
 ﻿namespace Pong.Paddles {
+    using Pong.Walls;
     using Scellecs.Morpeh;
     using Scellecs.Morpeh.Systems;
     using UnityEngine;
@@ -10,7 +11,7 @@
 
         public override void OnAwake() {
             camera = Camera.main;
-            paddles = World.Filter.With<Paddle>().Build();
+            paddles = World.Filter.With<Paddle>().With<AlignToScreenSide>().Build();
         }
 
         public override void OnUpdate(float deltaTime) {
@@ -23,25 +24,32 @@
 
             Vector3 positionUnderMouse = camera.ScreenToWorldPoint(mouseInput);
             foreach (Entity entity in paddles) {
-                ProcessPaddle(ref entity.GetComponent<Paddle>(), positionUnderMouse);
+                ProcessPaddle(
+                    ref entity.GetComponent<Paddle>(),
+                    ref entity.GetComponent<AlignToScreenSide>(),
+                    positionUnderMouse
+                );
             }
         }
 
-        private static void ProcessPaddle(ref Paddle paddle, Vector3 worldPosition) {
-            if (!paddle.xAxis && !paddle.yAxis) {
-                Debug.LogWarning($"{nameof(Paddle)} has no direction, are you sure?");
-                return;
-            }
-
+        private static void ProcessPaddle(
+            ref Paddle paddle,
+            ref AlignToScreenSide align,
+            Vector3 worldPosition
+        ) {
             Vector2 newPosition = paddle.body.position;
-            if (paddle.xAxis) {
-                newPosition.x = worldPosition.x;
+            switch (align.side)
+            {
+                case EScreenSide.Right:
+                case EScreenSide.Left:
+                    newPosition.y = worldPosition.y;
+                    break;
+
+                case EScreenSide.Top:
+                case EScreenSide.Bottom:
+                    newPosition.x = worldPosition.x;
+                    break;
             }
-
-            if (paddle.yAxis) {
-                newPosition.y = worldPosition.y;
-            }
-
             paddle.body.position = newPosition;
         }
     }
diff --git forkSrcPrefix/Assets/Pong/Balls/BallVelocitySystem.cs forkDstPrefix/Assets/Pong/Balls/BallVelocitySystem.cs
index 8ff8f60de955e49a55503a00eaec207e18be6e54..1d3334dbbf36ec1e57cc014266b45248c52e2adb 100644
--- forkSrcPrefix/Assets/Pong/Balls/BallVelocitySystem.cs
+++ forkDstPrefix/Assets/Pong/Balls/BallVelocitySystem.cs
@@ -1,6 +1,7 @@
 ﻿namespace Pong.Balls {
     using System;
     using Paddles;
+    using Pong.Walls;
     using Scellecs.Morpeh;
     using Scellecs.Morpeh.Globals.Variables;
     using Scellecs.Morpeh.Systems;
@@ -36,9 +37,10 @@
 
         private void HandleHit(ref Ball ball, in Ball.HitData hit) {
             Paddle paddle = default;
+            AlignToScreenSide align = default;
             Vector2 reflectDirection;
-            if (TryGetPaddle(hit.entity, ref paddle) && (paddle.xAxis || paddle.yAxis)) {
-                reflectDirection = CalculateReflectDirection(ref paddle, hit);
+            if (TryGetPaddle(hit.entity, ref paddle, ref align)) {
+                reflectDirection = CalculateReflectDirection(ref paddle, ref align, hit);
             } else {
                 reflectDirection = Vector2.Reflect(ball.lastVelocity, hit.normal);
             }
@@ -57,23 +59,31 @@
             return direction;
         }
 
-        private static Vector2 CalculateReflectDirection(ref Paddle paddle, in Ball.HitData hit) {
+        private static Vector2 CalculateReflectDirection(ref Paddle paddle, ref AlignToScreenSide align, in Ball.HitData hit) {
             Vector2 hitToPaddle = hit.hitPos - paddle.body.position;
 
             Vector2 faceDir;
             Vector2 maxReflectDir;
             float distFromPadCenter;
-            if (paddle.xAxis) {
-                faceDir = new Vector2(0f, hitToPaddle.y);
-                maxReflectDir = new Vector2(Math.Sign(hitToPaddle.x), hitToPaddle.y);
-                distFromPadCenter = hitToPaddle.x;
-            } else if (paddle.yAxis) {
-                faceDir = new Vector2(hitToPaddle.x, 0f);
-                maxReflectDir = new Vector2(hitToPaddle.x, Math.Sign(hitToPaddle.y));
-                distFromPadCenter = hitToPaddle.y;
-            } else {
-                Debug.LogError("Impossible case");
-                return hit.normal;
+            switch (align.side)
+            {
+                case EScreenSide.Right:
+                case EScreenSide.Left:
+                    faceDir = new Vector2(hitToPaddle.x, 0f);
+                    maxReflectDir = new Vector2(hitToPaddle.x, Math.Sign(hitToPaddle.y));
+                    distFromPadCenter = hitToPaddle.y;
+                    break;
+
+                case EScreenSide.Top:
+                case EScreenSide.Bottom:
+                    faceDir = new Vector2(0f, hitToPaddle.y);
+                    maxReflectDir = new Vector2(Math.Sign(hitToPaddle.x), hitToPaddle.y);
+                    distFromPadCenter = hitToPaddle.x;
+                    break;
+
+                default:
+                    Debug.LogError("Impossible case");
+                    return hit.normal;
             }
 
             faceDir.Normalize();
@@ -84,7 +94,7 @@
             return Vector2.Lerp(faceDir, maxReflectDir, t);
         }
 
-        private static bool TryGetPaddle(Entity entity, ref Paddle paddle) {
+        private static bool TryGetPaddle(Entity entity, ref Paddle paddle, ref AlignToScreenSide align) {
             if (entity == null) {
                 return false;
             }
diff --git forkSrcPrefix/Assets/Pong/Paddles/PaddleProvider.cs forkDstPrefix/Assets/Pong/Paddles/PaddleProvider.cs
index c9240cc5d1c3cfaf9c36c58f2669977863a60424..d0e089e82b1a905d19b24d2a5826800cf9478b8b 100644
--- forkSrcPrefix/Assets/Pong/Paddles/PaddleProvider.cs
+++ forkDstPrefix/Assets/Pong/Paddles/PaddleProvider.cs
@@ -9,8 +9,6 @@
     public struct Paddle : IComponent, IValidatableWithGameObject {
         [Required] public Rigidbody2D body;
         [Required] public BoxCollider2D collider;
-        public bool xAxis;
-        public bool yAxis;
 
         public void OnValidate(GameObject gameObject) {
             if (body == null) {
```

# patch: update colors only if published

```
diff --git forkSrcPrefix/Assets/Pong/Balls/ColorizedSystem.cs forkDstPrefix/Assets/Pong/Balls/ColorizedSystem.cs
index ccf894f48fb4a9e4ce008fc42d48709561477519..40da636c00d9ab57599ffd972e30f265d69cb112 100644
--- forkSrcPrefix/Assets/Pong/Balls/ColorizedSystem.cs
+++ forkDstPrefix/Assets/Pong/Balls/ColorizedSystem.cs
@@ -14,6 +14,7 @@
         public override void OnUpdate(float deltaTime) {
             foreach (Entity entity in filter) {
                 ref Colorized colorized = ref entity.GetComponent<Colorized>();
+                if (!colorized.variableColor.IsPublished) continue;
                 foreach (Renderer renderer in colorized.renderers) {
                     if (renderer is TrailRenderer trailRenderer) {
                         trailRenderer.startColor = colorized.variableColor.Value;
```
