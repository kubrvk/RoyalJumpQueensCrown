# Royal Jump: Queen's Crown

<img align="left" width="50%"  src="https://github.com/kubrvk/RoyalJumpQueensCrown/blob/main/Content/RoyalJump/banner-git.jpg?raw=true"/>
<h3><a href="https://github.com/kubrvk/RoyalJumpQueensCrown">4-) Royal Jump: Queen's Crown</a> <a href="https://play.google.com/store/apps/details?id=com.Kubrick.RoyalJump"><img src="https://img.shields.io/badge/Google_Play:-com.Kubrick.RoyalJump-000000?style=flat-square&logo=google-play&logoColor=white&labelColor=000000" height="25"/> </a></h3>

![](https://img.shields.io/badge/Mobile-0c5299?style=) ![](https://img.shields.io/badge/Platformer-759651?style=) ![C++](https://img.shields.io/badge/C++-00599C?style=logo=c%2B%2B&logoColor=white)  ![C++](https://img.shields.io/badge/Unreal_Engine_5.7-0E1128?style=for-the-badges&logo=unrealengine&logoColor=white)  ![C++](https://img.shields.io/badge/Status-Shipped-success?style=for-the-badges) 
<br>
A mobile platformer targeting Android. Guide the Queen through vertical levels where momentum and timing determine progress, built around a physics-driven movement model with touch-optimized controls tuned for mobile input and screen constraints.

Royal Jump: Queen's Crown is a precision-based mobile platformer built in Unreal Engine 5.7 using C++, targeting Android. The player controls the Queen through vertically-ascending levels where momentum, timing, and clean input execution determine success. The game is built around a physics-driven movement model with touch-optimized controls , all input handling, movement feel, and jump behavior are tuned specifically for mobile touch input latency and screen real estate constraints.

The technical challenge of this project centered on delivering responsive, frame-accurate platformer movement on mobile hardware , a context where input latency, thermal throttling, variable frame rates, and draw call budgets impose constraints absent from PC development. All gameplay systems, character controller, level architecture, UI, and assets were developed by a single developer.

<br clear="left"/>
<p align="center">
<img src="https://play-lh.googleusercontent.com/-fdLRWCer5tiFiKGO1p7D0F7QxnlWcDOV2Z8186qe1AdkEEwy071xpKT2kkop6-6d-ye5cFubVUPCW9rCGjp-A=w2560-h1440-rw" width="25%"/><img src="https://play-lh.googleusercontent.com/5YFNzifSfpoiCP6yAruXSyrZ-fNxXJld1JW1zpePrs6ES_v_rMg1JJMH5gre2cqs0nD8gSfce3JvEpG58ha7JXg=w2560-h1440-rw" width="25%"/><img src="https://play-lh.googleusercontent.com/hPN4-zzlTGuOfldYgNKfQHpns5ldD79F5hrK1KZM7RlX4rkeerdfl6E-ELWJyDnaSwk1Ya87NP8XQrpCADCDIQ=w2560-h1440-rw" width="25%"/><img src="https://play-lh.googleusercontent.com/0WAcluaE8b9i8B4wu7QGceP6LiBVJbtwXbVi_MyG4P1Meh3Cst3ClFW81317vW0fx0Wbw_A2yg__AMOnTWkHfyA=w2560-h1440-rw" width="25%"/>
</p>







---

## Engine & Technical Stack

| Layer | Technology |
|---|---|
| Engine | Unreal Engine 5.7 |
| Primary Language | C++ (movement, input, systems) |
| Platform | Android (primary), scalable to iOS |
| Input | Custom touch input component , no default UE touch abstraction |
| Physics | Chaos , physics-driven movement, collision response |
| Rendering | Mobile forward renderer, scalable quality tiers |
| Animation | UE5 Animation Blueprint + `UAnimInstance` C++ subclass |
| Profiling | Unreal Insights, Android GPU Debugger (Mali/Adreno) |
| Build | Android SDK/NDK, Gradle, UE5 Android packaging pipeline |
| 3D Pipeline | ZBrush , Maya , Substance Painter , UE5 |

---

## Architecture Overview

```
RoyalJump/
├── Source/
│   ├── Core/
│   │   ├── RJCharacter.h/.cpp                 # Player character, movement integration
│   │   ├── RJGameMode.h/.cpp                  # Level sequencing, progression gating
│   │   └── RJPlayerController.h/.cpp          # Touch input routing, HUD init
│   ├── Systems/
│   │   ├── TouchInputSystem/                   # Raw touch , intent mapping, gesture recognition
│   │   ├── MovementSystem/                     # Physics movement, jump, momentum, coyote time
│   │   ├── ParkourSystem/                      # Wall jump, ledge grab, slide, roll
│   │   ├── MomentumSystem/                     # Speed accumulation, momentum loss conditions
│   │   ├── LevelSystem/                        # Level progression, checkpoint, fail state
│   │   ├── CameraSystem/                       # Mobile-tuned follow camera, vertical bias
│   │   └── PerformanceSystem/                  # Adaptive quality, thermal management
│   ├── Levels/
│   │   ├── LevelBase/                          # Shared platform components, hazard actors
│   │   └── LevelData/                          # Per-level configs, spawn points, checkpoints
│   └── UI/
│       ├── HUD/                                # Touch zones, jump button, progress indicator
│       ├── TouchOverlay/                        # Visual touch feedback, zone boundaries
│       └── Menus/                              # Main menu, level select, settings
```

---

## Core Systems: Technical Detail

### 1. Touch Input System
![image](https://play-lh.googleusercontent.com/5YFNzifSfpoiCP6yAruXSyrZ-fNxXJld1JW1zpePrs6ES_v_rMg1JJMH5gre2cqs0nD8gSfce3JvEpG58ha7JXg=w2560-h1440-rw)

Touch input on mobile requires a fundamentally different approach from keyboard/gamepad , input arrives with variable latency, simultaneous multi-touch events, and no analog stick precision. The entire input pipeline is built as a custom `UTouchInputComponent` rather than relying on UE's default `UPlayerInput` touch abstraction.

**Raw Touch Processing:**
- `UTouchInputComponent` overrides `APlayerController::InputTouch` to intercept all touch events before UE's input system processes them.
- Touch events stored as `TArray<FTouchState>` , one entry per active finger, tracking `TouchIndex`, `ScreenPosition`, `StartPosition`, `StartTime`, `Phase` (`Began / Moved / Stationary / Ended`).
- Multi-touch handled explicitly , up to 4 simultaneous touch points tracked.

**Intent Mapping:**
- Screen divided into logical zones defined in `FTouchZoneConfig` (designer-configurable in data asset):
  - **Left zone**: movement direction (virtual joystick or swipe-based)
  - **Right zone**: jump input, parkour actions
  - **Full-screen swipe**: momentum boost gestures

```cpp
ETouchIntent UTouchInputComponent::ClassifyTouchIntent(const FTouchState& Touch)
{
    const float ScreenWidth = GEngine->GameViewport->Viewport->GetSizeXY().X;

    if (Touch.ScreenPosition.X < ScreenWidth * LeftZoneRatio)
    {
        return ClassifyMovementInput(Touch);
    }
    else
    {
        return ClassifyActionInput(Touch);
    }
}
```

**Gesture Recognition:**
- Tap: touch duration < `TapMaxDuration`, displacement < `TapMaxDrift` , triggers jump.
- Hold: duration > `HoldThreshold` with minimal displacement , triggers crouch/slide.
- Swipe: displacement > `SwipeMinDistance` within `SwipeMaxDuration` , direction vector determines parkour intent (up = wall jump attempt, lateral = dash).
- Double-tap: two taps within `DoubleTapWindow` on the same zone , triggers roll.

**Input Buffering:**
- Jump and action inputs buffered for `InputBufferWindow` frames (configurable, default ~6 frames).
- Buffer consumed on first valid execution frame , prevents missed inputs from slightly early presses.
- Critical for mobile where tap events can arrive 1–3 frames ahead of the movement state they intend to act on.

```cpp
void UMovementSystem::TickComponent(float DeltaTime, ...)
{
    if (JumpBuffer > 0.f)
    {
        JumpBuffer -= DeltaTime;
        if (CanJump())
        {
            ExecuteJump();
            JumpBuffer = 0.f;
        }
    }
}
```

---

### 2. Physics Movement System
![image](https://play-lh.googleusercontent.com/hPN4-zzlTGuOfldYgNKfQHpns5ldD79F5hrK1KZM7RlX4rkeerdfl6E-ELWJyDnaSwk1Ya87NP8XQrpCADCDIQ=w2560-h1440-rw)
Movement is driven by direct physics force application rather than UE's `UCharacterMovementComponent` kinematic preset. This gives precise control over acceleration curves, friction models, and mid-air behavior , necessary for a precision platformer where movement feel is the primary gameplay layer.

**Core Movement Model:**
- `UMovementSystem` applies forces to the character's `UCapsuleComponent` each tick via Chaos physics.
- Horizontal movement: `AddForce` with a configurable acceleration curve; speed clamped to `MaxGroundSpeed` via velocity projection.
- Deceleration: friction applied as a counter-force when no input is active; `GroundFriction` and `AirFriction` are separate values , air control deliberately reduced.
- Gravity: custom gravity scale per movement state (normal, glide, wall-slide) set via `UCharacterMovementComponent::GravityScale` override.

**Jump System:**
- Variable-height jump: jump force applied as an initial impulse; holding the jump button applies a continued upward force for `JumpHoldDuration` seconds, scaling down via a `UCurveFloat` (tap = short hop, hold = full jump height).
- Gravity is increased post-apex to make descent feel snappier than ascent (asymmetric gravity model).
- Jump count: standard double-jump supported; wall jump resets jump count.

```cpp
void UMovementSystem::ExecuteJump()
{
    FVector JumpImpulse = FVector(0.f, 0.f, JumpInitialForce);
    CharacterOwner->GetMesh()->GetBodyInstance()->AddImpulse(JumpImpulse, true);

    bJumpHeld = true;
    JumpHoldTimer = JumpHoldDuration;
    JumpCount++;
}

void UMovementSystem::TickJumpHold(float DeltaTime)
{
    if (bJumpHeld && JumpHoldTimer > 0.f)
    {
        float HoldScale = JumpHoldCurve->GetFloatValue(1.f - (JumpHoldTimer / JumpHoldDuration));
        FVector HoldForce = FVector(0.f, 0.f, JumpHoldForce * HoldScale);
        CharacterOwner->GetMesh()->GetBodyInstance()->AddForce(HoldForce, true);
        JumpHoldTimer -= DeltaTime;
    }
}
```

**Coyote Time:**
- `CoyoteTimeWindow` (default: 0.12s) , player retains jump availability for this duration after walking off a ledge.
- `bGroundedLastFrame` flag set each tick; on ground loss, `CoyoteTimer` starts. Jump available while `CoyoteTimer > 0`.
- Critical for mobile: coyote time window is slightly wider than typical PC platformers to compensate for touch input latency.

**Landing:**
- Landing detection via `OnCapsuleHit` , distinguishes floor vs wall vs ceiling hits via surface normal dot product against world up.
- Landing with high vertical velocity triggers a landing recovery animation (brief movement lock) , severity scales with fall height.
- Landing on moving platforms: velocity inherited from platform via `UPrimitiveComponent::GetPhysicsLinearVelocity` on the hit component.

---

### 3. Momentum System
![image](https://play-lh.googleusercontent.com/8OhHYAID-OCwc9eR0Sh8WSEmYskKNQ21ZF26q1WmzCkM-IbX8f-Hk-XCQK0EfH6C7oY5Ze-rlTgNnvisYvm9gQ=w2560-h1440-rw)
Clean, uninterrupted movement builds and preserves momentum , sloppy play bleeds speed. This is the core skill expression layer.

**Momentum Accumulation:**
- `FMomentumState` struct: `float CurrentMomentum`, `float MaxMomentum`, `float DecayRate`.
- `CurrentMomentum` increments on: clean landings (no stumble), consecutive jumps without touching walls, successful wall jumps, roll completions.
- Momentum translates to a speed multiplier applied on top of base `MaxGroundSpeed`: `EffectiveSpeed = MaxGroundSpeed * (1.f + MomentumSpeedBonus * NormalizedMomentum)`.

**Momentum Loss Conditions:**
- Hard stop (full reset): falling to death, hitting a hazard, missing a platform and grabbing a ledge.
- Partial decay: standing still for > `IdleDecayDelay` seconds, taking a hit.
- Decay is continuous via per-tick subtraction scaled by `DecayRate`; rate increases when idle.

**Visual Feedback:**
- Momentum level communicated to player via: character animation speed blend (higher momentum = faster run cycle playrate), trail VFX intensity (Niagara particle emission rate scales with `NormalizedMomentum`), and subtle camera FOV expansion at high momentum.
- All feedback is visual-only , no UI number displayed; feel over numbers.

---

### 4. Parkour System

`UParkourSystem` extends `UMovementSystem` with vertical traversal mechanics tuned for a climbing platformer.

**Wall Jump:**
- Wall detection: continuous line traces from capsule sides during air state.
- On wall contact in air: wall jump available for `WallContactWindow` frames (input buffer compatible).
- Wall jump direction: reflected off wall normal with a configurable upward bias.
- Successive wall jumps on the same surface blocked by `SameWallCooldown` , forces player to alternate walls for sustained climbing.

```cpp
void UParkourSystem::AttemptWallJump()
{
    FHitResult WallHit;
    if (!DetectWall(WallHit)) return;
    if (LastWallActor == WallHit.GetActor() && SameWallTimer > 0.f) return;

    FVector WallNormal = WallHit.Normal;
    FVector JumpDirection = FVector::VectorPlaneProject(WallNormal, FVector::UpVector).GetSafeNormal();
    JumpDirection += FVector(0.f, 0.f, WallJumpUpBias);

    GetOwner()->GetComponentByClass<UMovementSystem>()->ApplyJumpImpulse(
        JumpDirection * WallJumpForce
    );

    LastWallActor = WallHit.GetActor();
    SameWallTimer = SameWallCooldown;
    JumpCount = 0; // Reset double jump on wall jump
}
```

**Ledge Grab:**
- Ledge detection: forward trace at character top + downward trace to find ledge surface.
- On detection within grab range: character velocity zeroed, capsule position snapped to hang offset relative to ledge.
- Hang state: character suspends via physics constraint; pull-up triggered by upward swipe or hold input.
- Pull-up: plays a ledge-climb montage, repositions character above ledge on completion.

**Wall Slide:**
- Holding against a wall during descent reduces fall speed via an upward counter-force , `WallSlideFriction` multiplier on gravity.
- Consumes momentum at a moderate rate.
- Visual: dedicated wall-slide animation, dust particle on contact surface.

**Roll:**
- Triggered by double-tap; brief movement burst forward with a lower collision profile (capsule half-height reduced temporarily).
- Allows passing under obstacles.
- Brief i-frame window during roll , hazard immunity for roll duration.
- Adds momentum on completion if landing immediately follows.

**Slide:**
- Triggered by hold input while running.
- Reduces capsule height, applies forward momentum burst, then decelerates.
- Can transition into a roll on slide end for chained movement.

---

### 5. Level System & Progression

**Level Architecture:**
- Levels are vertically-oriented , primary progression axis is upward.
- Platform actors: `AStaticPlatform`, `AMovingPlatform`, `AFallingPlatform`, `ABouncePlatform` , each a distinct C++ class with configurable behavior.
- `AMovingPlatform`: `FVector` path defined by waypoints, speed, and loop mode (ping-pong / one-way / circular).
- `AFallingPlatform`: on player contact, triggers a `FTimerHandle`-delayed fall , platform begins `AddForce` downward after delay.
- `ABouncePlatform`: on player landing, applies an upward impulse of configurable magnitude , bypasses normal jump system.

**Checkpoints:**
- `ACheckpointActor` placed at interval heights , on player overlap, saves `FRJCheckpointData` (player position index, current momentum, level elapsed time).
- On death: respawn at last checkpoint, momentum reset, no level restart required.
- Checkpoint data stored in-memory during a run; persisted to `USaveGame` on level complete.

**Fail State:**
- Falling below a `KillPlane` `ATriggerVolume` triggers respawn , no death animation, immediate respawn to minimize friction.
- Death counter tracked per run; displayed on level complete screen.

**Level Progression:**
- `ARJGameMode` manages level sequence via an ordered `TArray<TSoftObjectPtr<UWorld>> LevelSequence`.
- Level complete condition: player reaches the `AGoalActor` at the top of the level.
- `UGameplayStatics::OpenLevelBySoftObjectPtr` used for level transitions , async load to avoid hitch.
- Level select UI unlocks entries as levels are completed, saved in `FRJProgressSaveData`.

---

### 6. Mobile Camera System

Camera for a vertical mobile platformer requires different behavior from standard third-person follow cameras.

**Vertical Bias:**
- Camera position lerps not to the player's current position but to a point slightly *above* the player's current height , providing look-ahead in the climb direction.
- Bias magnitude: `VerticalLookAheadDistance` scales with `CurrentMomentum` , faster movement reveals more of the upcoming level.

**Horizontal Follow:**
- Horizontal follow uses a tighter lag than vertical , horizontal position tracked closely to keep controls responsive.
- `UCameraLagComponent::CameraLagSpeed` tuned separately per axis.

**Death / Fall:**
- On falling below a threshold relative to camera, camera briefly locks position , prevents disorienting camera movement during fall.
- Respawn: camera cuts (no lerp) to checkpoint position to eliminate disorientation.

**Aspect Ratio Handling:**
- Camera FOV adjusted per device aspect ratio via `ARJPlayerController::AdjustCameraForAspectRatio()` , called on viewport resize event.
- Portrait mode (9:16, 9:19.5 etc.) receives a wider FOV to compensate for reduced horizontal screen space.

---

### 7. Mobile Performance & Optimization

Shipping a physics-driven platformer on Android requires explicit budgeting at every layer. Target: **60 fps on mid-range Android hardware** (Snapdragon 6-series, Mali-G57 equivalent).

**Rendering Budget:**
- Mobile forward renderer , no deferred shading.
- Draw call target: < 150 per frame on target hardware.
- Static geometry merged via `HLOD` (Hierarchical LOD) for level sections not near the player.
- Texture budget: max 1024×1024 for character; 512×512 for environment tiles; compressed with ASTC (Android).
- No Lumen, no Nanite on mobile , baked lighting used for static environment sections; dynamic light count capped at 2 per frame.
- Particle systems: Niagara effects budget-capped; LOD distance set aggressively to disable off-screen particles.

**Physics Budget:**
- Chaos physics simulation only runs on the player character and active platform actors within a `PhysicsActivationRadius`.
- Platforms outside radius are tick-disabled via `PrimaryActorTick.bCanEverTick = false`; re-enabled on player proximity.
- `AFallingPlatform` uses a lightweight custom tick (not physics simulation) until fall is triggered , reduces idle physics overhead.

**Tick Optimization:**
- Non-critical systems (momentum decay, camera bias lerp) moved to a fixed `0.05s` (20Hz) tick via `FTimerHandle` instead of per-frame.
- Only movement, input processing, and hit detection run at full frame rate.
- UI updates driven by delegate events (not polled each tick): `OnMomentumChanged`, `OnCheckpointReached`, `OnDeathCountChanged`.

**Adaptive Quality System:**
- `UPerformanceSystem` monitors frame time via `FPlatformTime::Seconds()` rolling average.
- If frame time exceeds `TargetFrameTime * 1.2` for > `ThermalWindowDuration` frames: drops one quality tier.
- Quality tiers control: shadow distance, particle count cap, post-process features.
- Tiers restored when frame time returns to budget for a sustained window.
- Prevents thermal throttling on sustained gameplay sessions , common on Android devices without active cooling.

```cpp
void UPerformanceSystem::TickComponent(float DeltaTime, ...)
{
    FrameTimeHistory.Add(DeltaTime);
    if (FrameTimeHistory.Num() > HistoryWindowSize)
        FrameTimeHistory.RemoveAt(0);

    float AvgFrameTime = 0.f;
    for (float FT : FrameTimeHistory) AvgFrameTime += FT;
    AvgFrameTime /= FrameTimeHistory.Num();

    if (AvgFrameTime > TargetFrameTime * ThermalThreshold)
        DropQualityTier();
    else if (AvgFrameTime < TargetFrameTime * RecoveryThreshold)
        RaiseQualityTier();
}
```

**Memory Management:**
- Level sections loaded async via `FStreamableManager` , geometry ahead of player loaded while geometry behind is unloaded.
- Asset references use `TSoftObjectPtr` throughout , no hard references in headers that would force eager loading.
- Character textures and meshes unloaded between levels; reloaded at level start.

**Android-Specific:**
- `AndroidManifest.xml` configured for: target SDK 34, `VIBRATE` permission (haptic feedback), landscape lock off (portrait-only).
- Input handled via `FAndroidInputInterface` , touch events intercepted before UE's default input system.
- Haptic feedback: `FAndroidApplication::Vibrate(Duration)` called on death, checkpoint, and level complete events , configurable in settings.
- APK optimized with Android App Bundle format for reduced download size; texture compression split by GPU architecture (ASTC for Adreno/Mali, ETC2 fallback).

---

### 8. Touch UI Architecture

Touch controls are designed to be as invisible as possible , the interface should not compete with the game for screen space.

**Touch Zones:**
- Left half of screen: movement input zone , no visible joystick by default; gesture-based.
- Right half: jump and action input.
- Optional visible joystick mode available in settings for players who prefer explicit thumb positioning , `UTouchInputComponent` switches between `EInputMode::Gesture` and `EInputMode::Joystick`.

**Visual Feedback:**
- Touch ripple effect spawned at each touch point (Niagara 2D screen-space system) , confirms input registration visually.
- Jump button highlights briefly on valid tap detection , communicates whether intent was recognized.
- Input zone boundaries shown as faint overlay in tutorial levels; hidden in standard play.

**HUD Elements:**
- Progress bar: vertical fill representing height within current level , minimal, positioned at screen edge.
- Momentum indicator: subtle color shift on character or edge glow (no number display).
- Death counter: small text, appears briefly after respawn, fades.
- All HUD elements designed for **one-thumb and two-thumb play** , no critical UI in the center of the screen.

**Settings:**
- Touch sensitivity: scales `SwipeMinDistance` and `TapMaxDuration` thresholds.
- Haptic toggle: enables/disables vibration calls.
- Input mode: gesture vs joystick.
- FPS display toggle (developer/accessibility option).

---

## Build & Packaging

**Android Build Configuration:**
- Minimum SDK: API 26 (Android 8.0)
- Target SDK: API 34 (Android 14)
- ABI: `arm64-v8a` primary; `armeabi-v7a` fallback build available
- Texture format: ASTC primary, ETC2 fallback via Android App Bundle split
- Orientation: Portrait locked
- Gradle build via UE5's built-in Android packaging pipeline
- Signing: release keystore configured in UE project settings

**UE5.7 Mobile Notes:**
- Mobile HDR disabled , standard mobile forward renderer for compatibility and performance.
- `r.MobileHDR=0` in `DefaultDeviceProfiles.ini`.
- Shadow maps: `r.Shadow.CSM.MaxCascades=1` on mobile profile , single cascade shadow map.
- Shader complexity managed via UE5 mobile shader permutation reduction settings.

---

## Performance Targets

| Target | Hardware Reference | Approach |
|---|---|---|
| 60 fps | Snapdragon 6-series, Mali-G57 | Forward renderer, < 150 draw calls, ASTC textures |
| < 200MB RAM | Mid-range Android | Async streaming, `TSoftObjectPtr`, level unload |
| < 150MB APK | Google Play | App Bundle, texture split, asset compression |
| Thermal stable | 30-min session | Adaptive quality system, physics budget, tick reduction |
| < 2s level load | Mid-range storage | Async level load, minimal hard dependencies |

---

## Development Scope

| Category | Detail |
|---|---|
| Developer count | 1 |
| Engine | Unreal Engine 5.7 |
| Languages | C++ |
| Platform | Android (Google Play) |
| Input model | Custom touch input component |
| Movement model | Physics force-based |
| 3D Assets | All original |
| Gameplay systems | 8 discrete systems (see above) |
| Development tools | UE5 Editor, Android Studio, Blender, Substance Painter |

---

## Related Projects

| Project | Description |
|---|---|
| [TIME SOUL](https://store.steampowered.com/app/2928270/TIME_SOUL) | Souls-like action platformer; parkour, time-as-resource , UE5.1 |
| [U.N. Owen Was Her](https://store.steampowered.com/app/3420540/UN_Owen_Was_Her) | Third-person horror; AI, bullet-hell boss , UE5.3 |
| [Olympus of the Heavens](https://store.steampowered.com/app/3358020/Olympus_of_the_Heavens) | Isometric co-op ARPG; 12 bosses, Steam co-op , UE5.3 |
| [Blood Garden](https://kubrik.itch.io/bloodgarden) | Souls-like melee combat; stamina, parry, elemental , UE5.4 |
| [ArtStation Portfolio](https://www.artstation.com/kubrik) | 3D modeling , characters, creatures, props, environments |

---

## Developer

**Kubrik** , Developer & 3D Artist  
[Google Play](https://play.google.com/store/apps/details?id=com.Kubrick.RoyalJump) · [ArtStation](https://www.artstation.com/kubrik) · [Steam](https://store.steampowered.com/search/?developer=Kubrik)

---

*All code, design, and custom assets produced by a single developer. No third-party gameplay code used in core systems.*
