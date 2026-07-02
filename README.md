# Mini Football — UE5 Multiplayer Prototype

A physics-based 1v1 multiplayer football game built in **Unreal Engine 5.6** using a server-authoritative replication architecture. Players compete to score goals by kicking a physics-driven ball, with all core gameplay logic owned and validated by the server.

Built as a technical prototype to demonstrate multiplayer systems design, UE5 Gameplay Framework usage, and Blueprint/C++ decision-making.

---

## Gameplay

- **1v1 multiplayer** — two players, one ball, two goals
- **Physics-based** — ball movement and collisions are fully physics-driven
- **Server-authoritative** — server owns all gameplay decisions; clients display results
- **Full gamepad and keyboard/mouse support**

---

## Controls

| Action | Keyboard / Mouse | Gamepad |
|---|---|---|
| Move | WASD / Arrow Keys | Left Thumbstick |
| Look | Mouse | Right Thumbstick |
| Shoot | Left Mouse Button | Face Button Left (X/Square) |
| Jump | Space Bar | Face Button Bottom (A/Cross) |

---

## Screenshots

### Enhanced Input & Shoot RPC
When the player triggers `IA_Shoot` and is in range, a Server RPC (`ServerKickBall`) is called — ensuring the kick is always validated and executed on the server before being reflected to clients.

![Enhanced Input and Server Kick RPC](screenshots/01_input_shoot_rpc.png)

---

### Input Mappings
Full input action mappings for `IA_Jump`, `IA_Move`, `IA_Look`, and `IA_Shoot` — supporting both keyboard/mouse and gamepad via the Enhanced Input System.

![Input Mappings](screenshots/02_input_mappings.png)

---

### Out of Bounds — Component-Based Respawn
When the ball or a player enters an out-of-bounds volume, a Sequence node triggers both `AC_BallRespawnComponent` and `AC_PlayerRespawnComponent` — reusable Actor Components that handle respawn logic independently and can be attached to any future actor.

![Out of Bounds Respawn Logic](screenshots/03_out_of_bounds_respawn.png)

---

### Remote Procedure Calls
Four Multicast RPCs handle all goal-related events so every client sees the same result:
- `Multicast_HandleOnScore` — notifies the football actor a goal was scored
- `Multicast_HandleGoalColorChange` — changes the goal colour based on team side with randomised RGB values picked server-side
- `Multicast_TriggerAnimation` — moves the goal mesh on the Z axis
- `Multicast_GoalCelebration` — plays the goal celebration after a 1-second delay

![Remote Procedure Calls](screenshots/04_rpc_goal_events.png)

---

### Goal Scored Flow
The full goal-scored sequence on the server: celebration multicast → goal colour change multicast → goal move multicast → server handles ball respawn. Random colour values are generated once on the server and multicasted to all clients — avoiding desync from independent random calls per machine.

![Goal Scored Server Flow](screenshots/05_goal_scored_flow.png)

---

### Player Spawn Logic
`ChoosePlayerStart` is overridden in the GameMode to find all `PlayerStart` actors by tag, use the current player count from GameState's player array length, and deterministically assign a start position using `GET` — ensuring players always spawn on the correct side without manual placement logic.

![Choose Player Spawn](screenshots/06_choose_player_spawn.png)

---

## Architecture

### Gameplay Framework Responsibilities

| Class | Responsibility |
|---|---|
| `GameMode` | Match rules, win condition, spawn logic, timer |
| `GameState` | Replicated match data — scores, time remaining, match state, player array |
| `PlayerState` | Per-player data |
| `Ball` | Physics actor with server authority; clients receive replicated movement |
| `Goal / OutOfBounds` | Overlap sensors with authority checks — only server drives gameplay events |

### Key Design Decisions

**Interfaces over hard casting** — gameplay interactions (`OnScored`, `OnOutOfBounds`, `ApplyKick`) are routed through UE Interfaces to avoid casting chains and keep ownership boundaries clear between actors.

**Component-based respawn** — `AC_BallRespawnComponent` and `AC_PlayerRespawnComponent` are reusable Actor Components, making respawn logic portable and easy to extend to new actor types.

**Server picks, multicast distributes** — for anything with randomness (e.g. goal colour), the server generates the value once and multicasts it to all clients, rather than letting each machine call random independently.

**Structs for variable management** — default variables are split from Actors into Structs for cleaner, more robust data control and easier future refactoring.

---

## Blueprint vs C++ Approach

**Blueprint** was used for rapid iteration on gameplay feel and visuals — goal overlaps, Niagara VFX, material changes, UI widgets, Timeline animations, and replication wiring. Blueprints are faster to tune and easier to debug visually during prototyping.

**C++ would be the next step** for scaling:
- Replicated gameplay framework classes (GameState/PlayerState logic, score/time replication, match flow)
- Complex mechanics — dribble forces, ball control, client-side prediction and server correction
- Large-scale refactors requiring clean RepNotify definitions and typed RPC signatures
- Performance-critical systems where Blueprint overhead matters

---

## Tradeoffs & Shortcuts

- **No client-side prediction** — ball uses server-authoritative physics; clients receive replicated movement. Jitter reduction via prediction/smoothing is a planned improvement.
- **Simple RPC patterns** — straightforward event/RPC flow over deeper netcode systems, appropriate for a prototype scope.
- **Spawn logic iterated** — started with a join counter, later moved to tag-based `ChoosePlayerStart` override for a more deterministic and scalable approach.

---

## Time Breakdown

| Area | Time |
|---|---|
| Player controls, ball mechanics, standalone project | 2–3 hours |
| VFX (Niagara, materials, animations) | 3–4 hours |
| Replication & RPC structure | 10–15 hours |
| UI + UI replication | 2–3 hours |
| Level layout | 1–2 hours |
| Blueprint abstraction & edge case testing | 3–5 hours |

---

## Planned Improvements

- **Ball prediction & smoothing** — reduce network jitter with client-side prediction and server correction
- **Component-based architecture** — `BallPhysicsComponent`, `GoalLogicComponent`, `OutOfBoundsComponent`, `MatchRulesComponent`, `PlayerInteractionComponent`
- **Match state machine** — Kickoff, Celebration freeze, Resume play, Overtime
- **RepNotify-driven UI** — consistent updates, late-join support, clean teardown on restart
- **Server-side validation** — range checks for kicks, dribble ownership, cooldowns, power-ups
- **Data Assets + Structs** — unified data handling for variables and inputs
- **Polish** — better level assets, more realistic player and goalpost animations

---

## Built With

- Unreal Engine 5.6
- Blueprint + C++ (prototype scope — Blueprint primary, C++ path documented)
- Enhanced Input System
- Niagara VFX
- UE Physics (server-authoritative)
