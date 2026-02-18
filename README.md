# Project Documentation

This document is the central technical reference for the project. It is split into two parts: **Part I** describes the **Indekos** game (architecture, systems, flow). **Part II** describes the **StudioDocumentation** editor tool (documentation framework, script indexing, and usage).

**Reading order for first-time readers**

- **Game developers**: Start with Part I — Project Overview, Architecture, Core Systems, then Safe Modification and Glossary.
- **Tool users**: Start with Part II — Section 1 (Title & Overview), Quick start, then Section 19 (Example Workflow). Use the FAQ (Section 14) and Troubleshooting (Section 13) as needed.
- **Tool extenders**: Part II — Sections 3 (Architecture), 4 (Folder Structure), 5 (Data Model), 6 (Script Indexing), 10 (Extending), 15 (Best Practices), 16 (Contribution). Use the API summary and code snippets for reference.

---

## 📑 Full Table of Contents

### Part I — Indekos Game Project
| Link | Description |
|------|-------------|
| [Badges](#badges) | Engine, architecture, project type |
| [Navigation](#-navigation) | Quick jump links |
| [Project Overview](#-project-overview) | What the game does |
| [Architecture](#-architecture-overview) | State + managers, data flow |
| [System Map](#-system-map) | Visual hierarchy |
| [Core Systems](#-core-systems) | Game state, managers, quest, dialogue, save, trigger, minigames |
| [Folder Structure](#-folder-structure) | indekos/ layout |
| [Important Classes](#-important-classes) | Class table |
| [Game Flow](#-game-flow-lifecycle) | Boot, init, runtime |
| [Execution Chain](#-runtime-execution-chain) | Per-frame and event chains |
| [State Machine](#-state-machine-visualization) | State graph |
| [Dependency Map](#-dependency-map) | Who depends on whom |
| [Configuration](#-configuration-guide) | Inspector, ScriptableObjects |
| [Safe Modification](#-safe-modification-guide) | Do not touch / extension points |
| [Glossary](#-glossary) | Project terms |

### Part II — StudioDocumentation System
| Link | Description |
|------|-------------|
| [SD 1. Title & Overview](#1-title--overview) | What is StudioDocumentation, who should use it |
| [SD 2. Key Features](#2-key-features-very-detailed) | Feature-by-feature deep dive |
| [SD 3. System Architecture](#3-system-architecture) | Assembly separation, asset pipeline |
| [SD 4. Folder Structure](#4-folder-structure-breakdown) | Every folder and file explained |
| [SD 5. Data Model](#5-data-model-deep-explanation) | DocumentationData, ScriptInfo, etc. |
| [SD 6. Script Indexing](#6-script-indexing-engine) | How indexing works |
| [SD 7. Auto Generation](#7-auto-documentation-generation) | When it runs, rebuild process |
| [SD 8. Editor UI](#8-editor-ui-system) | Renderer, tabs, search |
| [SD 9. Error Handling](#9-error-handling--stability) | Why creation can fail, defensive design |
| [SD 10. Extending](#10-extending-the-system) | Custom categories, tabs, integrations |
| [SD 11. Security](#11-security--sensitive-files) | SensitiveFile, classification |
| [SD 12. Performance](#12-performance-considerations) | Reflection, caching, lazy load |
| [SD 13. Troubleshooting](#13-troubleshooting-guide-very-detailed) | Errors and fixes |
| [SD 14. FAQ](#14-faq-section) | 85 Q&A |
| [SD 15. Best Practices](#15-best-practices) | Architecture, naming |
| [SD 16. Contribution](#16-contribution-guide) | How to add features |
| [SD 17. Versioning](#17-versioning-strategy) | Semantic versioning |
| [SD 18. Roadmap](#18-roadmap) | Planned features |
| [SD 19. Example Workflow](#19-example-workflow) | Step-by-step usage |
| [SD 20. Studio Usage](#20-internal-studio-usage-scenario) | Onboarding, auditing |

---

# Part I — Indekos Game Project

## Indekos

**A narrative life-simulation game set around student boarding-house life — day-based progression, quest-driven story, and minigames.**

---

## Badges

| Badge | Value |
|-------|--------|
| **Engine** | Unity |
| **Architecture** | State + Singleton + Interfaces |
| **Project Type** | Narrative / Life-Sim / Adventure |
| **Status** | Internal Documentation |
| **Input** | Unity Input System |
| **Dialogue** | Adventure Creator (AC) |

---

## 📑 Navigation

| Section | Description |
|---------|-------------|
| [🎮 Project Overview](#-project-overview) | What the project does and core concept |
| [🏗 Architecture Overview](#-architecture-overview) | Pattern, communication, data flow |
| [🗺 System Map](#-system-map) | Visual system hierarchy |
| [🧠 Core Systems](#-core-systems) | Collapsible breakdown per system |
| [📂 Folder Structure](#-folder-structure) | Tree and folder purpose table |
| [🧩 Important Classes](#-important-classes) | Class responsibility table |
| [🔄 Game Flow Lifecycle](#-game-flow-lifecycle) | Boot → runtime step-by-step |
| [⏱ Runtime Execution Chain](#-runtime-execution-chain) | Per-frame and event chains |
| [🎯 State Machine Visualization](#-state-machine-visualization) | State graph and transitions |
| [🔗 Dependency Map](#-dependency-map) | Who depends on whom |
| [⚙ Configuration Guide](#-configuration-guide) | Inspector, ScriptableObjects, paths |
| [🛡 Safe Modification Guide](#-safe-modification-guide) | Do not touch / safe extension points |
| [📖 Glossary](#-glossary) | Project-specific terms |

---

## 🎮 Project Overview

> **🟦 PROJECT SUMMARY**  
> **Indekos** is a narrative / life-simulation game set around student life (“indekos” = boarding house). The player moves through days, locations, and story beats while managing resources (money, hunger), completing quests, and engaging in minigames and dialogue. A **state machine** drives the entire game: each screen or mode is a **game state**; the active state controls input mapping and UI. **Managers** (singletons) hold global data and services; **triggers** in the world react to the player and call into managers to change state or load levels.

### Main pillars

- **Day-based progression** — Time advances by days; each day has quests, study, and story.
- **Multi-location exploration** — Campus, kos, gang kos, minimarket, kelas, guitar room, warteg, etc., with additive scene loading.
- **Quest-driven flow** — Main quests define objectives, trigger story/dialogue, and can spawn triggers, NPCs, and components (e.g. PC parts, cameras).
- **Study system** — In-class dialogue with multiple-choice answers (benar/salah) affecting quests.
- **Story vs chit-chat** — Story dialogue (tied to quests) vs optional NPC chit-chat.
- **Minigames** — Guitar (practice + final), Mingsut, Endless Run, Rakit PC, Warteg, Tarot, HP UI, Find Part PC, Freelance (QTE).
- **Save/Load** — Encrypted JSON saves (position, level, quest/story/study indices, time, money, tarot, inventory, dialogue log, map list, etc.).

---

## 🏗 Architecture Overview

### Pattern

- **Hybrid**: **State pattern** (game states) + **Singleton managers** + **Interface-based dependency injection** (managers depend on each other via interfaces, resolved in `Start()` with `*.Instance`).
- **Adventure Creator (AC)** is used for player, camera, dialogue (Conversation/DialogueOption), menus, cursor. The game enables/disables AC systems per state and switches input via a custom **Input Manager** keyed by `EGameState`.

### Communication

- **Singleton access** — All core managers inherit `singleton<T>`; access via `XxxManager.Instance`.
- **Interfaces** — In `Start()`, each manager assigns `game_manager = GameManager.Instance`, etc., using interfaces. Communication is **direct method calls** on these references.
- **State transitions** — States call `game_manager.GetGameState().ChangeState(EGameState.xxx)`. `GameState` activates the correct input map on change.
- **Callbacks (limited)** — `Action` delegates (e.g. `SetCameraKos`, `GetActionPartPC`); `QuestController.OnExecute`; AC `EventManager.OnStartSpeech_Alt`, `OnClickConversation`.
- **Trigger-based flow** — Triggers set `GameManager.SetTriggerFor(TriggerFor)`; the current state reads it on interact and calls LevelManager/QuestManager/StoryManager or `ChangeState`.

### Data flow (simplified)

1. **Boot** — Persistent scene → Awake (singletons) → Start (interface refs) → GameManager decides first scene → AsyncLoadLevel → GameState.Init(level) → first state Enter().
2. **Runtime** — GameManager.Update() → gamestate.Update() → current state Update() (and substate SubUpdate()). States react to input and call managers / ChangeState.
3. **Level change** — LevelManager.LoadLevel(...) → loading UI → scene unload/load additive → player moved → ChangeState → loading UI hidden → BGM/ambient/NPC refreshed.
4. **Quest flow** — QuestManager EnterQuest/NextQuest/Format(); ExitQuest() drives UI or state changes (minigames, HP, etc.).

### Architecture diagram

```
                    GameManager
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
  LevelManager     QuestManager      UIManager
        │                │                │
        │                │                │
  TriggerSystem    StorySystem     InputManager
        │                │                │
        └────────────────┴────────────────┘
                         │
              GameState (current state)
                         │
              Enter / Update / Exit
```

---

## 🗺 System Map

High-level hierarchy of systems and their roles.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           INDEKOS — SYSTEM MAP                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌───────────┐ │
│  │ Game State  │────▶│   Manager   │────▶│   Quest /   │────▶│  Trigger  │ │
│  │   Machine   │     │   Layer    │     │   Format    │     │  System   │ │
│  └──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └─────┬─────┘ │
│         │                   │                   │                   │       │
│         ▼                   ▼                   ▼                   ▼       │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌───────────┐ │
│  │   Input     │     │    UI       │     │  Component  │     │  Player   │ │
│  │  (per state)│     │  (AC menus) │     │  / Character│     │  (AC)     │ │
│  └─────────────┘     └─────────────┘     └─────────────┘     └───────────┘ │
│         │                   │                   │                             │
│         └───────────────────┴───────────────────┘                             │
│                                     │                                         │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌───────────┐   │
│  │   Study     │     │   Story     │     │  ChitChat   │     │ Save/Load  │   │
│  │   Dialogue  │     │   Dialogue  │     │   Dialogue  │     │            │   │
│  └─────────────┘     └─────────────┘     └─────────────┘     └───────────┘   │
│                                                                             │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                     │
│  │  DateTime   │     │   Audio     │     │  Minigames  │                     │
│  │  (day/time) │     │  BGM / SFX  │     │  (Guitar…)  │                     │
│  └─────────────┘     └─────────────┘     └─────────────┘                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🧠 Core Systems

<details>
<summary><b>🎮 Game State System</b></summary>

- **Scripts**: `GameState.cs`, `stateinterface`, all `*State.cs` under `core/pattern/state/game state/`, substates under `sub state/`.
- **Responsibilities**: Single active state; each state implements `Enter()`, `Update()`, `Exit()`. On change: previous state `Exit()`, new state `Enter()`, `InputManager.InputActivate(newState)`. Substates (per-day) provide `SubEnter`, `SubUpdate`, `SubExit`.
- **Interactions**: States get managers via `*.Instance` and call `GetGameState().ChangeState(...)`. LevelManager and QuestManager also call `ChangeState` (e.g. None during load, then target state; or minigames on quest exit).

</details>

<details>
<summary><b>📦 Manager Layer (Singletons + Interfaces)</b></summary>

- **Scripts**: All `*Manager.cs` in `core/`, plus interfaces in `core/pattern/interface manager/`.
- **GameManager**: Global flags, camera, options (sensitivity, graphics, resolution, fullscreen, AA, texture, VSync), tarot, inventory, dialogue log, map list, substate data, money/hunger, trigger string, PC part callback.
- **LevelManager**: Current/previous level, LoadLevel (loading UI, player teleport), DayController/AmbientController/NpcController refs, ChangeState + EnterQuest() after load.
- **UIManager**: Show/Hide AC menus, update date/objective/location/minimap UI, PlayUi(name, duration) with async queue.
- **InputManager**: InputMapping (Unity Input System); InputActivate(EGameState); exposes gameplay/guitar/pause/etc. readers.
- **QuestManager**: Main quest (copy per day), index, do_quest_list; EnterQuest/NextQuest/ExitQuest; Format() spawns triggers/components; drives state on exit.
- **StudyManager**, **StoryManager**, **DateTimeManager**, **SaveLoadManager**, **ComponentManager**, **CharacterManager**, **ChitChatManager**, **AudioManager**: See Important Classes table.
- **Interactions**: All resolve interfaces in Start(). GameManager is the hub; LevelManager and QuestManager drive state and scene flow.

</details>

<details>
<summary><b>📜 Quest System</b></summary>

- **Scripts**: `QuestManager`, `Quest.cs`, `QuestContent`, `ComponentManager`, `CharacterManager`.
- **Responsibilities**: Quest format strings (e.g. `trigger-input_pos_rot_action`, `npc-doni_pos_rot`, `part-pc_...`, `map_btn-name`) parsed in `Format()`; spawns prefabs from ComponentManager; ComponentManager/CharacterManager handle lifetime by quest index and day.
- **Interactions**: NextQuest() calls Format(); EnterQuest() checks Enter condition and may call StoryManager.ResumeStory or NextQuest; ExitQuest() drives UI or state changes (minigames, HP, endless run).

</details>

<details>
<summary><b>💬 Dialogue System (Study / Story / ChitChat)</b></summary>

- **Study**: AC DialogueOption + Conversation; multiple choice; answers (a/b/c) → benar/salah; StudyManager drives flow and notifies QuestManager.
- **Story**: StoryManager; tied to quest (ResumeStory with EStoryType.IncludeQuest advances quest).
- **ChitChat**: ChitChatManager; optional NPC dialogue, no quest advancement.
- All use `EventManager.OnStartSpeech_Alt` for subtitles and speaker-based UI scaling.

</details>

<details>
<summary><b>💾 Save / Load System</b></summary>

- **Scripts**: `SaveLoadManager`, `SaveData`, `Encryption`.
- **Responsibilities**: SaveData holds position, level, quest/story/study indices, time, day, money, tarot, inventory, do_quest_list, index_ignore, dialogue_log, map_list (with serialization wrappers for dictionaries). Save: JSON → encrypt → file; Load: file → decrypt → JSON → SaveData.
- **Interactions**: SaveLoadManager reads from other managers to fill SaveData; load would push SaveData back into those systems (via menu or bootstrap).

</details>

<details>
<summary><b>🔲 Trigger System</b></summary>

- **Scripts**: `TriggerManager` + `TriggerAction`, `TriggerDoor`, `TriggerInput`, `TriggerInputChitChat`, `TriggerInputDestroy`, `TriggerDestroyAction`, `TriggerDialogueNPC`, `TriggerSleep`.
- **Responsibilities**: OnTriggerEnter sets `GameManager.SetTriggerFor(TriggerFor)` and shows interact prompt (layer swap); OnTriggerExit clears trigger. Subclasses override Enter/Exit or perform actions (e.g. TriggerDestroyAction invokes `GetActionPartPC` for "partPC").
- **Interactions**: Player collides → trigger sets global TriggerFor → current state reads it on interact key and calls LevelManager/QuestManager/StoryManager/ChitChatManager or ChangeState.

</details>

<details>
<summary><b>🎲 Minigames</b></summary>

- **Guitar**: GuitarState, GuitarFinalState; chord/pluck/strum; AudioManager for clips and GuitarFinal data.
- **Mingsut**: MingsutState; gajah/semut/manusia.
- **Endless Run**: EndlessRunState; UIManager can switch (e.g. after warteg cutscene).
- **Rakit PC / Find Part PC**: FindPartPcState → GetActionPartPC; RakitPcState → Freelance (QTE).
- **Warteg**: WartegState → Gameplay.
- **Tarot / Book**: BookState, TarotState.
- **HP**: HpState; DoniChatWA → FindPartPC.
- **QTE**: QTEBarEvent in Freelance and Study substates.

</details>

---

## 📂 Folder Structure

```
Assets/
├── indekos/
│   ├── script/
│   │   ├── core/                    # Managers, singleton, state machine, interfaces
│   │   │   ├── pattern/             # state/, interface manager/, singleton, abstract
│   │   │   └── additional/          # TriggerManager family, Encryption
│   │   ├── system/                  # Game features, Quest, audio, save, UI, MiniGame
│   │   │   ├── core/game/           # Minimap, Tarot, Inventory, NPC, HP, camera
│   │   │   ├── Quest/
│   │   │   ├── audio/
│   │   │   ├── save/
│   │   │   ├── Scene/
│   │   │   ├── UI/
│   │   │   ├── loading/
│   │   │   ├── beat/
│   │   │   ├── MiniGame/
│   │   │   └── NPC/
│   │   └── editor/                  # Editor-only (PlayModeSave, BeatmapAnalyzerEditor)
│   ├── data/                        # Dialogue, beatmap, minimap, loading, input assets
│   ├── scene/                       # main.unity; sub-scene/*.unity (additive levels)
│   ├── prefabs/                     # UI, camera holders, triggers
│   └── assets/                      # Art, UI sprites, icons
├── AdventureCreator/                # Third-party: player, camera, dialogue, menus
└── Plugins/                         # e.g. Quibli
```

| Folder | Purpose |
|--------|---------|
| `indekos/script/core/` | Manager singletons, GameState, singleton pattern |
| `indekos/script/core/pattern/` | stateinterface, GameState, all *State classes, substates, manager interfaces |
| `indekos/script/core/additional/` | TriggerManager + subclasses, Encryption |
| `indekos/script/system/core/game/` | Minimap, TarotBook, Inventory, NpcController, HP chat, CameraGameManager |
| `indekos/script/system/Quest/` | Quest ScriptableObject, QuestController |
| `indekos/script/system/audio/` | BGM, SFX ScriptableObjects |
| `indekos/script/system/save/` | SaveData structure |
| `indekos/script/system/MiniGame/` | QTE, guitar, guitar final, endless run |
| `indekos/data/` | Dialogue assets, beatmap, minimap/loading render textures |
| `indekos/scene/sub-scene/` | Additively loaded levels (opening, mainmenu, kampus, gangkos, kos, kelas, etc.) |

---

## 🧩 Important Classes

| Class | Responsibility | Key Methods | Key Variables |
|-------|----------------|-------------|----------------|
| **GameManager** | Central singleton: state, options, tarot/inventory/dialogue log, trigger, PC part callback, substate data | SetTriggerFor/GetTriggerFor, GetGameState, SetCameraKos/UpdateCameraKos, GetSubstateData\<T\>, GetTarot/GetInventoryData, AsyncLoadLevel, ACAction | gamestate, triggerfor, substate_data, dialogue_log, inventory_data, map_list, action_part_pc, hold_part_pc, money, hunger |
| **GameState** | State machine: dictionary of states, current/previous/next enum | Init(level), ChangeState(EGameState), Update(), GetCurrentState/GetCurrentEnum/GetPreviousEnum | states, currentstate, currentEnum, previousEnum, nextEnum, input_manager |
| **stateinterface** | Contract for all game states | Enter(), Update(), Exit() | — |
| **LevelManager** | Scene loading, current level, day/ambient/NPC refs | LoadLevel(…), SetCurrentLevel, GetCurrentLevel/GetPreviousLevel, SetDayController/SetAmbientController/SetNpcController | current_level, previous_level, day_controller, ambient_controller, npc_controller, b_is_load |
| **UIManager** | AC menu show/hide, date/objective/location/minimap updates | ShowUI/HideUI, GetUI, UpdateDateUi/UpdateObjectiveUi/UpdateMiniMapUi/UpdateLocationUi, PlayUi, DisableUiGameplay/EnableUiGameplay | locationNames, icon_assets, ui_queue |
| **InputManager** | Input System per-state activation and read helpers | InputActivate(EGameState), GetMoveInput/GetCameraInput/GetInteractInput/GetMapInput/GetPauseInput, GetChord*Input, GetPetik*Input, GetInput/GetInputQTSBar | input (InputMapping), bIsLocked |
| **QuestManager** | Main quest, index, enter/next/exit, format parsing and spawn | EnterQuest, NextQuest, ExitQuest, Format, AddDoQuestList, GetGoalQuest/GetEnterQuest/GetCurrentTime, QuestNexDay, SpawnTrigger/SpawnComponent | mainQuest, indexMainQuest, do_quest_list, quest_list |
| **StudyManager** | Study dialogue flow and answers | ResumeStudy, PlayNgedumelKelas, SetIndexStudy/GetIndexStudy, StudyNextDay | conversations, answers_list, index_study, dialogueLog, action_end_study_dialogue, action_after_answer |
| **StoryManager** | Story dialogue tied to quest | ResumeStory(EStoryType), StoryNextDay, SetIndexStory/GetIndexStory, AddIndexIgnore/GetIndexIgnore | story, index_story, index_ignore, conversations, conversation_partner |
| **DateTimeManager** | In-game date and day index | NextDay, SetTime/GetCurrentTime, SetDay/GetCurrentDay | currentTime, currentDay |
| **SaveLoadManager** | Persist/load game | Save(slot), Load(slot), CheckLoad(slot), GetSaveData, SaveFileName | savedata |
| **ComponentManager** | Named prefabs and lifetime spawns | GetComponentScene, SetTempComponent, DestroyCurrentComponentSceneTemp, LifetimeComponent, AddLifetimeIgnore | component_scene, component_scene_temp, temp_component, lifetime, lifetime_spawn |
| **CharacterManager** | Character spawn by quest/day | LifetimeCharacter, SpawnCharacter, AddCharacterIgnore, DestroyAll | characters, character_spawn, name_character_ignore |
| **TriggerManager** | Base for all triggers | OnTriggerEnter/Exit, Enter()/Exit() virtual | TriggerFor, icon_interact, interact, manager refs |
| **AudioManager** | BGM/SFX and guitar/beatmap data | PlayBGM/StopBGM, PlaySFX, GetAudioClip, PlayPluck/PlayStrum, PlayBgmGuitarFinal, GetBeatmapData | bgm, sfx, audio_source_bgm/sfx, guitarData, guitar_final_data, beatmap_datas |
| **ChitChatManager** | Optional NPC dialogue | SetNewDialog, ResumeStory | story, target |

---

## 🔄 Game Flow Lifecycle

1. **Boot** — Persistent scene loads; `singleton` Awake (DefaultExecutionOrder -100) sets instances; manager `Start()` assigns interface references (GameManager, LevelManager, UIManager, …).
2. **Initialization** — GameManager.Awake: resolutions, map_list, InsertTarot, InsertInventory, InitDialogueLog, ACAction(false). GameManager.Start: resolves LevelManager, AudioManager, SaveLoadManager, InputManager; EnableInput(); checks SaveLoadManager.CheckLoad("save1"…"save4") → first scene = mainmenu_scene if any save exists, else opening_scene (editor may force mainmenu_scene).
3. **State Init** — AsyncLoadLevel(scene) → on done: LevelManager.SetCurrentLevel, AudioManager.PlayBGM(), new GameState(), gamestate.Init(level). GameState.Init: creates all state instances, registers in dictionary; current state = Opening (if opening_scene) or MainMenu (otherwise); InputActivate(currentEnum); currentstate.Enter().
4. **Runtime Loop** — Every frame: GameManager.Update() → gamestate.Update() → currentstate.Update() (and substate SubUpdate()). States read input, call managers, and call ChangeState when changing mode. Triggers set TriggerFor; the state that handles interact reads it and performs the action.
5. **Scene Change** — Trigger or state calls LevelManager.LoadLevel(...) → loading UI shown → UnloadSceneAsync(current_level) + LoadScene additive → WaitLoading coroutine: wait → ChangeState(target) → move player → EnterQuest() → hide loading UI → PlayBGM(), day_controller?.UpdateLight(), ambient_controller?.UpdateAmbient(), npc_controller?.UpdateTotalNPC().
6. **Quest Flow** — EnterQuest() checks Enter (scene name, state name, "quest", "skip"); may call StoryManager.ResumeStory or NextQuest. NextQuest() increments index, Format(), SetTime, UpdateDateUi/UpdateObjectiveUi, ExitQuest(). ExitQuest() switch on Exit string → PlayUi(...) or ChangeState(minigame/HP/endlessrun).

---

## ⏱ Runtime Execution Chain

> **🟦 PER-FRAME CHAIN**  
> `GameManager.Update()` → `gamestate?.Update()` → `currentstate.Update()` → (if substate) `substates[day].SubUpdate()` → state reads InputManager and GetTriggerFor(), calls managers or ChangeState.

> **🟦 LEVEL LOAD CHAIN**  
> `LevelManager.LoadLevel(...)` → b_is_load = true, DestroyCurrentComponentSceneTemp, ShowUI(loading), ChangeState(eNone), (optional disable player), StartCoroutine(WaitLoading), UnloadSceneAsync + LoadScene additive, UpdateMiniMapUi → WaitLoading: WaitForSeconds(wait_a) → ChangeState(state) → (optional move player, EnterQuest()) → WaitForSeconds(wait_b) → HideUI(loading), PlayBGM(), day_controller?.UpdateLight(), ambient_controller?.UpdateAmbient(), npc_controller?.UpdateTotalNPC()`.

> **🟦 INTERACT CHAIN (e.g. GameplayState)**  
> InputInteract() → GetInteractInput() → switch (GetTriggerFor()): level-* → LevelManager.LoadLevel(...); map → ChangeState(eMap); story-quest → StoryManager.ResumeStory(IncludeQuest); chitchat-npc → ChitChatManager.ResumeStory(); etc.

---

## 🎯 State Machine Visualization

```
                    ┌─────────────┐
                    │   Opening   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  MainMenu  │◄──────────────────────┐
                    └──────┬──────┘                      │
                           │                             │
                    ┌──────▼──────────┐                   │
                    │ SelectCharacter │                   │
                    └──────┬──────────┘                   │
                           │                             │
                    ┌──────▼──────┐     ┌────────────┐   │
              ┌─────│  Gameplay  │─────▶│   Pause    │───┘
              │     └──────┬──────┘     └────────────┘
              │            │
              │     ┌─────┴─────┬─────────┬─────────┬─────────┬─────────┐
              │     ▼           ▼         ▼         ▼         ▼         ▼
              │  Map       NextDay      Book      Guitar   Minimarket  Warteg
              │     │           │         │         │         │         │
              │     │           │         ▼         │         │         │
              │     │           │      Tarot       │         │         │
              │     │           │         │         │         │         │
              │     └───────────┴─────────┴─────────┴─────────┴─────────┘
              │                        │
              │     ┌─────────────────┼─────────────────┐
              │     ▼                 ▼                 ▼
              │  Study               HP            FindPartPC
              │     │                 │                 │
              │     │                 │                 ▼
              │     │                 │            RakitPC
              │     │                 │                 │
              │     │                 │                 ▼
              │     │                 │            Freelance
              │     │                 │
              │     ▼                 ▼
              │  Mingsut         GuitarFinal
              │     │                 │
              │     ▼                 ▼
              │  EndlessRun           │
              │     │                 │
              └─────┴─────────────────┘
                           │
                    ┌──────▼──────┐
                    │    None     │  (e.g. during load)
                    └─────────────┘
```

---

## 🔗 Dependency Map

- **GameManager** — Hub: holds GameState, trigger, substate data, tarot/inventory/dialogue/map/money; used by almost every system.
- **GameState** — Depends on InputManager (InputActivate). All state classes depend on GameManager (and often LevelManager, UIManager, QuestManager) via Instance.
- **LevelManager** — GameManager, UIManager, QuestManager, ComponentManager, AudioManager.
- **QuestManager** — GameManager, UIManager, LevelManager, AudioManager, DateTimeManager, StoryManager, ComponentManager.
- **UIManager** — GameManager, InputManager, LevelManager, QuestManager, DateTimeManager.
- **InputManager** — None (only InputMapping asset).
- **StudyManager** — UIManager, QuestManager, AudioManager, DateTimeManager.
- **StoryManager** — GameManager, InputManager, UIManager, QuestManager, AudioManager, DateTimeManager, ComponentManager.
- **SaveLoadManager** — GameManager, LevelManager, QuestManager, DateTimeManager, StoryManager, StudyManager.
- **ComponentManager** — QuestManager, DateTimeManager.
- **CharacterManager** — QuestManager, DateTimeManager.
- **ChitChatManager** — InputManager, UIManager, AudioManager, ComponentManager.
- **TriggerManager** — GameManager, AudioManager, InputManager, ComponentManager, UIManager, LevelManager, QuestManager.

### Small diagram

```
                    GameManager
                         │
    ┌────────┬───────────┼───────────┬────────┐
    ▼        ▼           ▼           ▼        ▼
LevelMgr  QuestMgr    UIManager  InputMgr   GameState
    │        │           │           │           │
    └────────┴───────────┴───────────┴───────────┘
                         │
    StudyMgr, StoryMgr, SaveLoadMgr, ComponentMgr,
    CharacterMgr, ChitChatMgr, AudioMgr, DateTimeMgr
```

---

## ⚙ Configuration Guide

### Inspector variables (summary)

| Component | Key inspector fields |
|-----------|----------------------|
| **GameManager** | substate_data (SubstateData list), inventory (Inventory SO), data (TarotBook) |
| **UIManager** | icon_assets (e.g. sun/moon) |
| **StudyManager** | conversations (ConversationList per day), answers_list (AnswersList per day) |
| **QuestManager** | quest_list (QuestStruct: day + Quest SO) |
| **ComponentManager** | component_scene (name + Transform), lifetime (LifetimeComponents) |
| **CharacterManager** | characters (CharacterComponents) |
| **AudioManager** | bgm, sfx (SO), bgm_mixer, sfx_mixer, guitar_final_data, guitarData, beatmap_datas |
| **Trigger prefabs** | TriggerFor string (e.g. "level-kos", "map", "story-quest", "chitchat-npc", "partPC") |

### ScriptableObjects

| Asset | Menu | Use |
|-------|------|-----|
| TarotBook | Data/Tarot | Tarot content list |
| Inventory | Data/Inventory | Inventory content list |
| Quest | Data/Quest | QuestContent list (Enter, questGoal, time, format, Exit) |
| BGM | Data/BGM | level → AudioClip |
| SFX | Data/SFX | action → AudioClip |
| GameplayStateScriptableObject | SubState/Gameplay_State_Variation | GameplaySubstateDay1/Day2 data |
| StudyStateScriptableObject | SubState/Study_State_Variation | StudySubstateDay1/Day2 data |
| GuitarFinalData | Data/GuitarFinalData | Guitar final minigame |
| DataGuitar | Data/Guitar Data | Chord clips for guitar |
| ReadmeIndekos | Data/Readme | Optional |

### Paths and config

- **Save path**: `Application.persistentDataPath/saves/v{Application.version}/`; slot names e.g. save1…save4; content = encrypted JSON (SaveData).
- **Input**: Unity Input System asset (InputMapping); action maps per EGameState (opening, mainmenu, gameplay, pause, map, guitar, study, etc.).

---

## 🛡 Safe Modification Guide

> **⚠ DO NOT MODIFY CARELESSLY**
>
> - **GameManager** — Core holder of state, trigger, substate_data, dialogue log. Changing GetTriggerFor/SetTriggerFor or substate_data layout can break triggers and substates.
> - **EGameState enum + GameState.Init()** — Adding/removing states requires: enum, dictionary in Init(), and InputManager.InputActivate() branch. Missing map = no input in that state.
> - **QuestManager.Format()** — Format string parsing and switch (trigger-input, trigger-door, npc-*, part-pc, map). New format types need a new case and possibly ComponentManager entries.
> - **SaveData structure** — New fields need serialization (and possibly wrappers); Save/Load and bootstrap must read/write and apply them.
> - **TriggerFor strings** — Hardcoded across triggers and state switch (e.g. GameplayState). Change only with full consistency.
> - **AC integration** — ACAction, cursor, input enable/disable are tied to states. Disabling AC where it is expected breaks dialogue/camera.
> - **Singleton Awake** — Order -100 is intentional; duplicate destroys self. New singletons depending on these in Awake can cause null refs.
> - **Manager Start()** — All interface resolution in Start(). Disabled or different-scene Start = null refs.
> - **Dialogue hierarchy** — StudyManager/StoryManager use GetChild(day).GetChild(index) for DialogueOption. Renaming/re-parenting breaks indexing.
> - **Quest Enter/Exit strings** — ExitQuest() and EnterQuest() use fixed strings ("minigame-mingsut", "hp", "endless-run"). Align with quest data.

> **✅ SAFE EXTENSION POINTS**
>
> - **New game state**: Add EGameState, new state class, register in GameState.Init(), add InputManager.InputActivate() branch; optionally GetSubstateData.
> - **New trigger type**: Subclass TriggerManager, set TriggerFor; add case in state that handles interact (e.g. GameplayState).
> - **New quest exit/enter**: Add case in QuestManager.ExitQuest() and EnterQuest(); extend Format() and ComponentManager/CharacterManager for new spawns.
> - **New UI screen**: Add AC menu; UIManager.ShowUI/HideUI by name; consider dedicated state + input map for full-screen mode.
> - **New level**: Add scene in sub-scene; add LoadLevel call with scene name, position, rotation, ELoadType, ELoadingType, EGameState; add BGM entry in AudioManager if needed.

---

## 📖 Glossary

- **AC** — Adventure Creator: player, camera, dialogue, menus, cursor.
- **Indekos** — Boarding house; game theme (student life).
- **Kos** — Boarding house (location); "camera kos" = boarding house camera callback.
- **Gangkos** — Gang kos (boarding house alley); scene name.
- **Warteg** — Warung Tegal (food stall); scene and state.
- **Ngedumel** — Dialogue/conversation (e.g. "ngedumel kelas" = in-class dialogue).
- **Mingsut** — Minigame (gajah/semut/manusia ≈ rock-paper-scissors).
- **Rakit PC** — PC assembly minigame; Find Part PC = collecting parts; Freelance = follow-up QTE.
- **TriggerFor** — String set by trigger when player is inside; read on interact to decide action.
- **Substate** — Per-day variation of a state (e.g. GameplaySubstateDay1); uses ScriptableObject from GameManager.substate_data.
- **Format (quest)** — String parsed by QuestManager.Format() to spawn triggers/NPCs/cameras/parts or add map buttons.
- **Enter / Exit (quest)** — QuestContent.Enter (when quest step is "entered"); Exit (what to do when leaving, e.g. "minigame-mingsut", "hp").
- **Benar / Salah** — Right / wrong (study answers).
- **do_quest_list** — List of completed quest step IDs; used for conditions and save.
- **datetime** — In-game time (date + ETime: Morning/Afternoon/Evening/Night).
- **LifetimeComponents / CharacterComponents** — Spawn rules by quest index and day; ComponentManager/CharacterManager spawn/despawn accordingly.

---

## Full ASCII Architecture Diagram (Reference)

```
                    ┌─────────────────────────────────────────────────────────────┐
                    │                      GameManager                             │
                    │  (state, trigger, tarot, inventory, dialogue_log, substate)  │
                    └───────────────────────────┬─────────────────────────────────┘
                                                │
         ┌─────────────────────────────────────┼─────────────────────────────────────┐
         │                                     │                                     │
         ▼                                     ▼                                     ▼
┌─────────────────┐                 ┌─────────────────┐                 ┌─────────────────┐
│   GameState     │                 │  LevelManager   │                 │  QuestManager   │
│ (state machine) │◄────────────────│ (load level,    │────────────────►│ (main quest,    │
│                 │  ChangeState()   │  current level) │  EnterQuest()   │  Format, spawn) │
└────────┬────────┘                 └────────┬────────┘                 └────────┬────────┘
         │                                     │                                     │
         │ Update()                             │                                     │
         ▼                                     ▼                                     ▼
┌─────────────────┐                 ┌─────────────────┐                 ┌─────────────────┐
│ *State (current)│                 │   UIManager     │                 │ ComponentManager│
│ Enter/Update/   │────────────────│ (menus, date,   │                 │ (prefabs,       │
│ Exit            │  ShowUI/HideUI  │  minimap)       │                 │  lifetime)       │
└────────┬────────┘                 └─────────────────┘                 └─────────────────┘
         │
         │ GetTriggerFor() / interact
         ▼
┌─────────────────┐     SetTriggerFor()     ┌─────────────────┐
│ TriggerManager  │◄─────────────────────────│  Player (AC)     │
│ (colliders)     │                          │  OnTriggerEnter  │
└─────────────────┘                          └─────────────────┘

┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  InputManager   │  │  StudyManager   │  │  StoryManager   │  │ DateTimeManager │
│ (per-state map) │  │ (study dialogue)│  │ (story dialogue)│  │ (day, time)     │
└────────┬────────┘  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘
         │                     │                     │                     │
         └─────────────────────┴─────────────────────┴─────────────────────┘
                                         │
                    All resolved in Start() via *.Instance (interfaces)
                                         │
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  AudioManager   │  │ SaveLoadManager │  │ ChitChatManager  │  │ CharacterManager│
└─────────────────┘  └─────────────────┘  └─────────────────┘  └─────────────────┘
```

---

# Part II — StudioDocumentation System

### Document conventions (Part II)

- **Code blocks**: C# examples use `csharp` or no language tag; paths and commands use backticks when inline.
- **Menu paths**: Written as **Tools → Studio → Documentation** or **Tools → Studio → Documentation → Rebuild Documentation**.
- **File paths**: Relative to project root (e.g. `Assets/StudioDocumentation/Resources/DocumentationData.asset`).
- **Bold** is used for UI labels (e.g. **Rebuild Documentation**), important terms (e.g. **ScriptableObject**), and key concepts (e.g. **data assembly**).
- **Code** in tables or inline uses backticks. C# snippets use a `csharp`-tagged block. Paths are given relative to the project root unless stated otherwise.
- **Tables** are used for quick reference (symptoms/fixes, FAQ, API summary). **ASCII diagrams** show structure and flow without external images.

### Index of Part II sections (by number)

| § | Section |
|---|--------|
| 1 | Title & Overview |
| 2 | Key Features (Very Detailed) |
| 3 | System Architecture |
| 4 | Folder Structure Breakdown |
| 5 | Data Model Deep Explanation |
| 6 | Script Indexing Engine |
| 7 | Auto Documentation Generation |
| 8 | Editor UI System |
| 9 | Error Handling & Stability |
| 10 | Extending the System |
| 11 | Security & Sensitive Files |
| 12 | Performance Considerations |
| 13 | Troubleshooting Guide |
| 14 | FAQ Section (85 Q&A) |
| 15 | Best Practices |
| 16 | Contribution Guide |
| 17 | Versioning Strategy |
| 18 | Roadmap |
| 19 | Example Workflow |
| 20 | Internal Studio Usage Scenario |

### When to use which section

| If you want to… | Read section(s) |
|------------------|------------------|
| Understand what the tool is and who it’s for | 1 |
| See what each feature does and how it works | 2 |
| Understand assemblies, asset pipeline, domain reload | 3 |
| Find where a file lives and what it’s for | 4 |
| Understand data types (DocumentationData, ScriptInfo, etc.) | 5 |
| Understand how scripts are discovered and parsed | 6 |
| Know when auto-generation runs and how to rebuild | 7 |
| Understand the window layout and UI flow | 8 |
| Understand why creation can fail and how we handle errors | 9 |
| Add custom tabs, categories, or integrations | 10 |
| Use or extend sensitive-file tracking | 11 |
| Optimize or understand performance | 12 |
| Fix a specific error or symptom | 13 |
| Get quick answers to common questions | 14 |
| Follow coding and doc style rules | 15 |
| Contribute code or features | 16 |
| Understand versioning and migration | 17 |
| See planned features | 18 |
| Follow step-by-step workflows | 19 |
| See how a studio would use this | 20 |

---

## 1. Title & Overview

### What is StudioDocumentation?

**StudioDocumentation** is an editor-only Unity framework that provides an in-Editor documentation window. It allows teams to maintain project documentation (architecture, systems, managers, game flow, configuration, glossary) and to auto-generate script documentation by scanning C# scripts and extracting class names, namespaces, base types, public methods, public fields, and XML summary comments.

It is implemented as a **modular, ScriptableObject-driven** system: documentation content is stored in a **DocumentationData** asset; the Editor window reads that asset and renders it in a tabbed, searchable UI with a sidebar for navigation.

### Problem it solves

- **Scattered knowledge**: Game and tooling documentation often lives in READMEs, wikis, or comments. StudioDocumentation centralizes it in one place inside the Unity Editor.
- **Out-of-date script docs**: Manually keeping a list of classes and methods is tedious. The Script Indexing Engine scans the project and builds a script index; Rebuild Documentation can generate a "Project Scripts" category from that index.
- **Onboarding**: New developers can open **Tools → Studio → Documentation** and browse architecture, systems, and auto-generated script overviews without leaving the editor.
- **Safe modification**: The Safe Modification and Sensitive Files sections help teams document which scripts are critical and how to extend the project without breaking it.

### Why it exists

The framework exists to provide a **production-grade, internal documentation experience** similar to premium Asset Store tools or AAA studio internal wikis—without requiring external tools or markdown files that are easy to forget to update. By keeping documentation in Unity and optionally auto-generating part of it from code, the project stays self-describing.

### Who should use it

- **Beginners**: Use the window to read overviews, architecture, and step-by-step workflows without editing code.
- **Senior engineers**: Use the data model, assembly layout, and extension points to add custom categories, tabs, or integrations (e.g. AI summaries, markdown, code preview).
- **Tech leads**: Use it to define safe modification guidelines, sensitive file lists, and best practices in one place.
- **Studios**: Use it for onboarding, code auditing, knowledge transfer, and compliance (e.g. documenting critical systems).

### Comparison with other approaches

| Approach | Pros | Cons |
|----------|------|------|
| **StudioDocumentation (this tool)** | Lives inside Unity; ScriptableObject-based; optional auto-generation from code; single asset; no external files. | Editor-only; no built-in markdown or HTML export yet; search is in-memory (no full-text index). |
| **External wiki (Confluence, Notion)** | Rich formatting; access from browser; permissions. | Out of sync with code; no automatic script index; context switch. |
| **Markdown in repo** | Versioned with code; simple. | Often forgotten; no Unity integration; no script indexing or links to assets. |
| **Code comments only** | Always next to code. | No single place to read; no high-level structure; no sensitive-file list or safe-modification doc. |
| **Doxygen / DocFX** | Full API docs from code. | External tool; not integrated into Unity Editor; no custom categories or studio-specific content. |

StudioDocumentation is best when you want **one place inside Unity** for both hand-written documentation (architecture, safe modification, onboarding) and **auto-generated script overview** (class list, methods, XML summary), with optional sensitive-file tracking and no dependency on external services.

### Quick start (minimal steps)

1. **Open the window**: **Tools → Studio → Documentation**. The asset is created automatically at `Assets/StudioDocumentation/Resources/DocumentationData.asset` if it did not exist.
2. **First-time content**: If the asset had no sections, the window runs **TryAutoPopulateIfEmpty** and builds "Project Scripts" from your code. If not, click **Rebuild Documentation** in the toolbar.
3. **Navigate**: Use the left sidebar to expand **Project Scripts** and click any script name. The main area shows class summary, namespace, base type, and public methods/fields.
4. **Search**: Type in the toolbar search box and press Enter (or use the search logic); click **Open** on a result to jump to that section.
5. **Edit by hand**: Select **DocumentationData.asset** in the Project window and edit categories/sections in the Inspector. Add links (Target = script/scene/prefab), message boxes (Warning/Critical), and rich text. Save the project to persist.

---

## 2. Key Features (Very Detailed)

### 2.1 Custom menu entry

- **What it does**: Adds **Tools → Studio → Documentation** and **Tools → Studio → Documentation → Rebuild Documentation** to the Unity menu.
- **Why it matters**: Single entry point for opening the doc window and for forcing a full re-index and regeneration of the "Project Scripts" category.
- **How it works**: `[MenuItem("Tools/Studio/Documentation")]` and `[MenuItem("Tools/Studio/Documentation/Rebuild Documentation")]` on static methods in `DocumentationWindow` and `DocumentationRebuild`.
- **Example**: User clicks **Tools → Studio → Documentation** → window opens; user clicks **Rebuild Documentation** → categories cleared, script index rebuilt, "Project Scripts" category populated and saved.

### 2.2 Main documentation window

- **What it does**: A dockable, resizable EditorWindow that shows a toolbar, tabs, a left sidebar (tree navigation), and a main content area (scrollable).
- **Why it matters**: Central place to read and navigate all documentation.
- **How it works**: `DocumentationWindow` (EditorWindow) uses IMGUI. In `OnGUI` it calls `DocumentationRenderer.DrawToolbar`, `DrawTabs`, then a horizontal split: `DocumentationSidebar.OnGUI()` (left) and `DocumentationRenderer.DrawContent(rect)` (right). Scroll position per tab is stored in `DocumentationRouter`.
- **Architecture**: Window owns `DocumentationRouter`, `DocumentationSidebar`, `DocumentationRenderer`, and `DocumentationData`; it does not own the data asset (loaded via `DocumentationDataUtility.LoadOrCreate()`).

### 2.3 Left sidebar navigation (tree)

- **What it does**: Lists all categories (e.g. "Project Scripts") as foldouts; under each category, lists section titles. Clicking a section navigates to it and sets the active tab from the section’s `tab` field.
- **Why it matters**: Fast navigation without scrolling through one long page.
- **How it works**: `DocumentationSidebar` iterates `_data.categories`, draws a foldout per category, then draws a button per `section` in `category.sections`. On click it calls `_router.NavigateTo(category, section)`. The active section is highlighted (bold) by comparing `_router.ActiveSection == section`.

### 2.4 Top toolbar (search, refresh, rebuild, debug)

- **What it does**: Search bar (filters sections by title/summary/content/code/diagram), **Refresh** (AssetDatabase.Refresh), **Rebuild Index** (ScriptIndexer.BuildIndex → scriptIndex only), **Rebuild Documentation** (full clear + re-index + "Project Scripts" + save), **Debug** toggle (shows active tab, category count, script index count, sensitive file count).
- **Why it matters**: Search finds content quickly; Rebuild Index updates the script list without regenerating categories; Rebuild Documentation fully regenerates from code; Debug helps diagnose empty or missing data.
- **How it works**: `DocumentationRenderer.DrawToolbar()` uses `EditorStyles.toolbar` and `GUILayout.TextField` for search, buttons for Refresh/Rebuild Index/Rebuild Documentation, and `GUILayout.Toggle` for Debug. Rebuild Documentation calls `DocumentationRebuild.Rebuild(_data)`.

### 2.5 Tabs system

- **What it does**: Ten tabs: Overview, Architecture, Systems, Managers, GameFlow, DependencyGraph, Configuration, SafeModification, Glossary, TechnicalDebt. Switching tabs changes the main content context and stores scroll position per tab.
- **Why it matters**: Organizes documentation by theme; Dependency Graph tab shows the ASCII script dependency view; other tabs can filter or group content by topic.
- **How it works**: `DocumentationTab` enum; `DocumentationRouter.ActiveTab` and `SetScrollPosition(tab, pos)` / `GetScrollPosition(tab)`. Tabs are drawn as toggles in `DrawTabs()`; content in `DrawContent()` uses `_router.ActiveTab` and, when active tab is DependencyGraph, draws the ASCII graph from `DependencyVisualizer.BuildAsciiGraph(_data.scriptIndex)`.

### 2.6 Script indexing engine

- **What it does**: Scans all `MonoScript` assets under `Assets`, skips paths containing `/Editor/`, `/Packages/`, or `StudioDocumentation`, and for each script extracts class name, namespace, base type, `isMonoBehaviour`, `isScriptableObject`, `isAbstract`, public instance methods (excluding property getters/setters), public instance fields, and XML class summary from the source file.
- **Why it matters**: Provides the raw data for "Project Scripts" and for the Dependency Graph without manual maintenance.
- **How it works**: `ScriptIndexer.BuildIndex()` (in `StudioDocumentation.Editor` namespace, internal static). Uses `AssetDatabase.FindAssets("t:MonoScript", new[] { "Assets" })`, filters by `ShouldSkipPath`, then `MonoScript.GetClass()` and reflection for methods/fields; `TryAttachXmlComments` reads the .cs file and `ExtractClassSummary` parses `///` lines above the class. Returns `List<ScriptInfo>`.

### 2.7 Auto documentation generation (Rebuild)

- **What it does**: Clears all categories, rebuilds the script index, creates one category "Project Scripts", and for each `ScriptInfo` creates a `DocumentationSection` (title = class name, summary = XML or namespace, richTextContent = namespace, base, flags, methods list, fields list). Saves the asset.
- **Why it matters**: One click to refresh the entire script-based documentation from current code.
- **How it works**: `DocumentationRebuild.Rebuild(data)` clears `data.categories`, sets `data.scriptIndex = ScriptIndexer.BuildIndex()`, builds one `DocumentationCategory` with many `DocumentationSection`s, adds it to `data.categories`, then `EditorUtility.SetDirty(data)` and `AssetDatabase.SaveAssets()`. Wrapped in try/catch with error log.

### 2.8 Sensitive files list

- **What it does**: `DocumentationData.sensitiveFiles` is a list of `SensitiveFile` entries (label, asset reference, SensitiveLevel, notes). The UI can use this to show warnings or to filter views (the current window does not yet render a dedicated sensitive-files view; the data model is in place).
- **Why it matters**: Teams can mark core or do-not-touch assets and document why they are sensitive.
- **How it works**: Data-only in `DocumentationData`; optional future rendering in a tab or sidebar.

### 2.9 Collapsible sidebar categories

- **What it does**: Each category in the left sidebar is drawn as a foldout (expand/collapse). The expanded state is stored in `DocumentationCategory.isExpanded` and persists in the asset.
- **Why it matters**: Users can collapse "Project Scripts" when it has hundreds of entries and expand only the categories they need.
- **How it works**: `EditorGUILayout.Foldout(category.isExpanded, category.name, ...)` in DocumentationSidebar. The result is written back to the category so the state is serialized (if the asset is marked dirty and saved after layout).

### 2.10 Scroll position memory per tab

- **What it does**: When you switch tabs, the scroll position of the previous tab is stored; when you switch back, the scroll view restores that position so you do not lose your place.
- **Why it matters**: Improves usability when browsing long content (e.g. Dependency Graph or many sections) across different tabs.
- **How it works**: `DocumentationRouter` holds a `Dictionary<DocumentationTab, Vector2> _scrollPositions`. In `DrawContent`, the scroll view uses `GetScrollPosition(ActiveTab)` as the initial value and, when the scope ends, `SetScrollPosition(ActiveTab, scrollPosition)`.

### 2.11 Message boxes (Info, Warning, Critical)

- **What it does**: Each section can have a message type (None, Info, Warning, Critical) and message text. The renderer draws this in a styled box (info = default helpBox; warning = tinted; critical = red-tinted).
- **Why it matters**: Authors can highlight important caveats (e.g. "Do not modify this system without reviewing GameState.Init()") directly in the doc.
- **How it works**: `DocumentationSection.messageType` and `messageText`. In `DrawMessage(section)` we choose a GUIStyle and draw the text in a vertical scope. Colors are adjusted for dark skin in DocumentationStyle.

### 2.12 Links with Ping and Open

- **What it does**: Sections can have a list of links (label + Object reference). In the UI, each link is shown as an object field plus "Ping" and "Open" buttons. Ping highlights the asset in the Project window; Open opens the asset (e.g. script in IDE).
- **Why it matters**: Quick navigation from documentation to the actual script, scene, or prefab without searching.
- **How it works**: `DocumentationSection.links` (List of DocumentationLink). In the renderer we iterate and call `EditorGUIUtility.PingObject(link.target)` and `AssetDatabase.OpenAsset(link.target)` for the buttons.

---

## 3. System Architecture

### Assembly separation (Runtime vs Editor)

The project uses **two assemblies**:

1. **StudioDocumentation** (runtime/data)
   - **Path**: `Assets/StudioDocumentation/StudioDocumentation.asmdef`
   - **Include platforms**: Empty (all platforms). The assembly is included in builds but contains only data types (ScriptableObject and serializable classes); no game logic runs from it at runtime.
   - **Contents**: `DocumentationData`, `DocumentationCategory`, `DocumentationSection`, `DocumentationTab`, `DocumentationMessageType`, `DocumentationLink`, `ScriptInfo`, `SensitiveFile`, `SensitiveLevel`.
   - **References**: None (only Unity engine).

2. **StudioDocumentation.Editor**
   - **Path**: `Assets/StudioDocumentation/Editor/StudioDocumentation.Editor.asmdef`
   - **Include platforms**: Editor only.
   - **Contents**: All scripts under `Assets/StudioDocumentation/Editor/` (DocumentationWindow, DocumentationRouter, DocumentationSidebar, DocumentationRenderer, DocumentationSearchEngine, DocumentationStyle, DocumentationDataUtility, ScriptIndexer, DocumentationRebuild, DependencyVisualizer, etc.).
   - **References**: `StudioDocumentation` (the data assembly).

### Why ScriptableObject must be in the runtime assembly

Unity’s **asset pipeline** serializes and deserializes assets. When you call `AssetDatabase.CreateAsset(instance, path)`, the type of `instance` must be a type that Unity can resolve and serialize. Types defined in **Editor-only assemblies** are not available to the same degree during asset import and in the player build. If `DocumentationData` were in the Editor assembly, creating or reimporting the asset could fail with errors like "Creating asset at path ... failed" or "Unable to import newly created asset". Putting **ScriptableObject and all serialized data types** in a **non-Editor assembly** (StudioDocumentation) ensures the asset can be created and imported reliably. The Editor assembly then references this assembly and uses the types (e.g. `ScriptableObject.CreateInstance<DocumentationData>()`) so the created instance is of a serializable, non-Editor type.

### Asset pipeline behavior

- **CreateAsset**: Writes a new asset file and triggers import. The type of the object must be serializable and from an assembly that the asset pipeline can use.
- **LoadAssetAtPath**: Loads an asset by path; the type parameter must match the asset’s script type (from the data assembly).
- **SaveAssets**: Writes dirty assets to disk. Used after changing `DocumentationData` in memory.

### Domain reload

When scripts are recompiled (e.g. after editing a .cs file), Unity performs a **domain reload**: the app domain is torn down and recreated, and `[InitializeOnLoad]` static constructors run again. If asset creation or heavy work runs in a static constructor, it can run while the asset database or other systems are not fully ready. **EditorApplication.delayCall** is used to defer creation to the next editor tick, after the domain and asset database are ready, avoiding "Creating asset failed" during reload.

### AssetDatabase behavior

- **IsValidFolder**: Returns whether a folder exists in the project. Used to guard creation (e.g. only create Resources if the StudioDocumentation folder exists).
- **CreateFolder**: Creates a folder; used to ensure `Assets/StudioDocumentation/Resources` exists before creating the asset there.
- **Refresh**: Reimports assets; used after creating a folder so the new path is known.
- **DeleteAsset**: Removes an asset at a path; used to remove a legacy/broken asset at the old path when migrating to the Resources path.

### EditorApplication.delayCall reasoning

The static constructor of `DocumentationDataUtility` subscribes to `EditorApplication.delayCall`. On the next editor update, `EnsureDataAssetExists()` runs. This avoids running `CreateAsset` during domain reload when the assembly or asset database might not be ready, and prevents the "Creating asset failed" error that can occur when creating an asset whose type lives in an Editor-only assembly (we fixed that by moving the type to the data assembly; the delay still improves robustness).

### ASCII: Folder and assembly layout

```
Assets/StudioDocumentation/
├── StudioDocumentation.asmdef          ← Runtime/Data assembly (all platforms)
├── Data/                               ← Part of StudioDocumentation assembly
│   ├── DocumentationData.cs
│   ├── DocumentationCategory.cs
│   ├── DocumentationSection.cs
│   ├── ScriptInfo.cs
│   └── SensitiveFile.cs
├── Resources/
│   └── DocumentationData.asset         ← Asset (ScriptableObject instance)
└── Editor/
    ├── StudioDocumentation.Editor.asmdef  ← Editor-only assembly (references StudioDocumentation)
    ├── Core/
    │   ├── DocumentationWindow.cs
    │   ├── DocumentationRouter.cs
    │   ├── DocumentationSidebar.cs
    │   ├── DocumentationRenderer.cs
    │   ├── DocumentationSearchEngine.cs
    │   └── DocumentationStyle.cs
    ├── Rendering/
    │   └── DependencyVisualizer.cs
    └── Utilities/
        ├── DocumentationDataUtility.cs
        ├── ScriptIndexer.cs
        └── DocumentationRebuild.cs
```

### ASCII: Assembly dependency graph

```
                    ┌─────────────────────────┐
                    │  StudioDocumentation    │
                    │  (Data assembly)        │
                    │  - DocumentationData   │
                    │  - ScriptInfo           │
                    │  - DocumentationSection │
                    │  - SensitiveFile        │
                    └────────────┬────────────┘
                                 │ referenced by
                                 ▼
                    ┌─────────────────────────┐
                    │  StudioDocumentation    │
                    │  .Editor                │
                    │  - DocumentationWindow │
                    │  - ScriptIndexer       │
                    │  - DocumentationRebuild│
                    │  - DocumentationRenderer│
                    └─────────────────────────┘
```

### ASCII: Asset lifecycle

```
[Editor Load / Domain Reload]
        │
        ▼
DocumentationDataUtility static ctor
        │
        ▼
EditorApplication.delayCall += EnsureDataAssetExists
        │
        ▼
[Next editor tick] EnsureDataAssetExists()
        │
        ├─ Folder "Resources" missing? → CreateFolder → Refresh
        ├─ Asset at path exists? → return
        ├─ Legacy path asset? → DeleteAsset (try/catch)
        └─ CreateInstance<DocumentationData>() → CreateAsset → SaveAssets
        │
        ▼
[User opens Tools → Studio → Documentation]
        │
        ▼
LoadOrCreate() → LoadAssetAtPath<DocumentationData>(path)
        │
        ▼
TryAutoPopulateIfEmpty(data) → if sectionCount == 0 → Rebuild(data)
        │
        ▼
Window shows sidebar + content; user can Rebuild Index / Rebuild Documentation
```

### Data flow: User actions

| User action | Flow |
|-------------|------|
| **Rebuild Index** | Toolbar click → ScriptIndexer.BuildIndex() → data.scriptIndex = result → EditorUtility.SetDirty(data) → (user saves or auto-save). Categories and sections are unchanged; only the script list is updated. |
| **Rebuild Documentation** | Toolbar or menu → DocumentationRebuild.Rebuild(data) → data.categories.Clear() → BuildIndex() → new category "Project Scripts" with one section per ScriptInfo → SetDirty → SaveAssets. All previous categories/sections are replaced. |
| **Search** | User types in toolbar → DocumentationSearchEngine.Search(query, tabFilter) → linear scan over all sections (title, summary, richTextContent, codeBlock, diagramText) → renderer shows result list; click **Open** → router.NavigateTo(category, section) and tab set to Overview. |
| **Click sidebar section** | DocumentationSidebar draws buttons → OnGUI click → router.NavigateTo(category, section) → renderer DrawContent shows that section; scroll position restored from router for current tab. |
| **Switch tab** | DrawTabs click → router.SetActiveTab(tab) → DrawContent branches (Overview, Architecture, …, Dependency Graph, Debug). Scroll position is stored per tab in the router. |
| **Refresh (toolbar)** | Reloads data from disk: LoadOrCreate() or LoadAssetAtPath again so in-memory _data matches the saved asset. Useful after editing the asset in the Inspector from another window or after an external change. |

---

## 4. Folder Structure Breakdown

| Path | Responsibility | Dependency | Allowed references |
|------|----------------|------------|--------------------|
| **StudioDocumentation.asmdef** | Defines the data assembly; no Editor code. | None | Unity engine only |
| **StudioDocumentation.Editor.asmdef** | Defines the Editor assembly; all tooling. | StudioDocumentation | StudioDocumentation, UnityEditor, UnityEngine |
| **Data/** | ScriptableObject and serializable types used by the asset and by the Editor. | — | UnityEngine (no UnityEditor) |
| **Editor/Core/** | Main window, router, sidebar, renderer, search engine, styles. | Data (via reference), Utilities, Rendering | StudioDocumentation (data), UnityEditor, UnityEngine |
| **Editor/Utilities/** | Asset creation, script indexing, rebuild logic. | Data | StudioDocumentation (data), UnityEditor, UnityEngine |
| **Editor/Rendering/** | ASCII dependency graph builder and drawer. | Data | StudioDocumentation (data), UnityEditor, UnityEngine |
| **Resources/** | Holds `DocumentationData.asset` so it can be loaded by path; optional `Resources.Load` in future. | — | — |

### File-by-file responsibility (Editor)

| File | Responsibility |
|------|----------------|
| **DocumentationWindow.cs** | Entry point; menu item; owns router, sidebar, renderer; loads data; calls TryAutoPopulateIfEmpty; draws split layout and resize handle. |
| **DocumentationRouter.cs** | Holds ActiveTab, ActiveCategory, ActiveSection; Get/SetScrollPosition per tab; SetActiveTab; NavigateTo(category, section). |
| **DocumentationSidebar.cs** | Draws left panel; foldouts per category; buttons per section; calls router.NavigateTo on click; highlights active section. |
| **DocumentationRenderer.cs** | DrawToolbar (search, Refresh, Rebuild Index, Rebuild Documentation, Debug); DrawTabs; DrawContent (section, search results, dependency graph, debug). |
| **DocumentationSearchEngine.cs** | Search(query, tabFilter) over all sections; matches title, summary, richTextContent, codeBlock, diagramText; returns (category, section) pairs. |
| **DocumentationStyle.cs** | Static GUIStyles (Header, SubHeader, TabHeader, SidebarSection, SidebarItem, InfoBox, WarningBox, CriticalBox, RichText); InitIfNeeded(); dark skin tweaks. |
| **DocumentationDataUtility.cs** | [InitializeOnLoad]; delayCall to EnsureDataAssetExists; LoadOrCreate(); creates Resources folder and asset if missing; deletes legacy path. |
| **ScriptIndexer.cs** | BuildIndex(); FindAssets MonoScript; filter paths; GetClass(); reflect methods/fields; read file for XML summary; return List&lt;ScriptInfo&gt;. |
| **DocumentationRebuild.cs** | RebuildFromMenu (menu item); TryAutoPopulateIfEmpty(sectionCount==0); Rebuild(data) (clear, index, "Project Scripts", save); BuildSectionContent(info). |
| **DependencyVisualizer.cs** | BuildAsciiGraph(scripts) groups by inheritance and formats lines; DrawGraphBox(graph) draws in helpBox. |

- **Data/** must not reference **Editor** or **UnityEditor** so the data assembly stays non-Editor and the asset can be serialized.
- **Editor** scripts may reference **Data** and use types like `DocumentationData`, `ScriptInfo`, etc., for both UI and asset creation.

---

## 5. Data Model Deep Explanation

### DocumentationData (ScriptableObject)

- **Purpose**: Root asset for all documentation content and script index.
- **Fields**:
  - `categories`: List of `DocumentationCategory`. Each category has a name and a list of sections.
  - `scriptIndex`: List of `ScriptInfo`. Filled by ScriptIndexer; used by Dependency Graph and by Rebuild to generate "Project Scripts" sections.
  - `sensitiveFiles`: List of `SensitiveFile`. Optional list of assets marked as sensitive with level and notes.
- **Serialization**: Standard Unity ScriptableObject serialization; lists are serialized as arrays.
- **Inspector**: Create via **Assets → Create → Studio Documentation → Documentation Data**; or the tool creates it at `Assets/StudioDocumentation/Resources/DocumentationData.asset` if missing.
- **Example (JSON-like)**:
```json
{
  "categories": [
    { "name": "Project Scripts", "sections": [ { "title": "GameManager", "summary": "Central singleton.", "tab": 0 } ] }
  ],
  "scriptIndex": [
    { "className": "GameManager", "namespace": "", "inheritance": "singleton`1", "assetPath": "Assets/indekos/...", "isMonoBehaviour": true, "isScriptableObject": false, "isAbstract": false, "xmlSummary": "", "publicMethods": ["..."], "publicFields": ["..."] }
  ],
  "sensitiveFiles": []
}
```

### DocumentationCategory (Serializable class)

- **Purpose**: Groups sections under a name (e.g. "Project Scripts") with optional icon and expanded state.
- **Fields**: `_id` (serialized, unique), `name`, `icon` (Texture2D), `isExpanded`, `sections` (List of DocumentationSection).
- **Serialization**: Serialized as part of the parent ScriptableObject.
- **Inspector**: Shown when editing the root asset; categories are list elements.

### DocumentationSection (Serializable class)

- **Purpose**: One documentation page: title, summary, tab, rich text, message box, code block, diagram, links.
- **Fields**: `_id`, `title`, `summary`, `tab` (DocumentationTab enum), `richTextContent`, `messageType`, `messageText`, `codeBlock`, `diagramText`, `links` (List of DocumentationLink).
- **Serialization**: Same as category; nested in categories.
- **Inspector**: Each section has headers for Identity, Content, Messages, Code/Diagrams, Links. Links have label + Object reference (script, scene, prefab, etc.) and can be Ping/Open from the UI.
- **Example (JSON-like)**:
```json
{
  "title": "GameManager",
  "summary": "Central singleton: state, options, trigger, substate data.",
  "tab": 0,
  "richTextContent": "Namespace: \nBase: singleton`1\nMonoBehaviour\n\nPublic methods:\n  • ...",
  "messageType": 0,
  "messageText": "",
  "codeBlock": "",
  "diagramText": "",
  "links": []
}
```

### DocumentationTab (enum)

- **Values**: Overview, Architecture, Systems, Managers, GameFlow, DependencyGraph, Configuration, SafeModification, Glossary, TechnicalDebt.
- **Purpose**: Assigns a section to a tab; sidebar click sets active tab from section; scroll position is stored per tab.

### DocumentationLink (Serializable class)

- **Fields**: `label` (string), `target` (UnityEngine.Object). Target can be a MonoScript, SceneAsset, Prefab, or any Object.
- **Usage**: Drag-and-drop in Inspector; in the UI, Ping and Open buttons use `EditorGUIUtility.PingObject` and `AssetDatabase.OpenAsset`.

### ScriptInfo (Serializable class)

- **Purpose**: One row of script index data: class identity, inheritance, and extracted methods/fields/XML.
- **Fields**: `className`, `@namespace`, `inheritance` (base type name), `assetPath`, `isMonoBehaviour`, `isScriptableObject`, `isAbstract`, `xmlSummary`, `publicMethods` (List<string>), `publicFields` (List<string>).
- **Serialization**: Stored in `DocumentationData.scriptIndex`; all fields are serializable.
- **Inspector**: Visible when inspecting the asset; scriptIndex is a list of ScriptInfo.
- **Example (JSON-like)**:
```json
{
  "className": "AudioManager",
  "namespace": "",
  "inheritance": "singleton`1",
  "assetPath": "Assets/indekos/script/core/AudioManager.cs",
  "isMonoBehaviour": true,
  "isScriptableObject": false,
  "isAbstract": false,
  "xmlSummary": "Manages BGM and SFX.",
  "publicMethods": ["Void PlayBGM()", "Void PlaySFX(String action)", "..."],
  "publicFields": []
}
```

### SensitiveFile (Serializable class)

- **Purpose**: Mark an asset as sensitive (core, high, medium, low) with notes.
- **Fields**: `label`, `asset` (Object), `level` (SensitiveLevel enum), `notes` (TextArea).
- **Usage**: Add entries in the Inspector on the DocumentationData asset; the UI can later show warnings or filter by sensitivity (current window does not yet render a dedicated view).

### SensitiveLevel (enum)

- **Values**: Core, High, Medium, Low.
- **Purpose**: Classify how critical an asset is (e.g. Core = do not modify without review).

### Serialization behavior summary

| Type | Serialized | Notes |
|------|------------|-------|
| DocumentationData | Yes (ScriptableObject) | Root asset; lists are serialized as arrays. |
| DocumentationCategory | Yes (nested) | List elements; _id is SerializeField. |
| DocumentationSection | Yes (nested) | List elements; _id is SerializeField. |
| DocumentationLink | Yes (nested) | Object reference stored by Unity (fileID, guid). |
| ScriptInfo | Yes (in scriptIndex) | All fields are serializable primitives or lists. |
| SensitiveFile | Yes (in sensitiveFiles) | Object reference for asset. |

### Unity Inspector behavior

- **DocumentationData**: Create via **Assets → Create → Studio Documentation → Documentation Data**. Inspector shows three lists (Categories, Script Index, Sensitive Files). Expanding a category shows its sections; each section has foldouts for Identity, Content, Messages, Code/Diagrams, Links.
- **Script Index**: Read-only in practice (filled by Rebuild). You can inspect ScriptInfo entries to see className, namespace, methods, fields, xmlSummary.
- **Links**: The **Target** field accepts any UnityEngine.Object (MonoScript, SceneAsset, Prefab, etc.). Drag-and-drop from the Project window.

### Naming and IDs (data model)

- **DocumentationCategory** and **DocumentationSection** use a serialized `_id` field (e.g. string or int) so that references or UI state can identify a section without relying on title (which can change). The sidebar and router use object identity (category/section references) for navigation; _id is useful for persistence or cross-session reference.
- **Category names** should be unique for clarity (e.g. "Project Scripts", "Architecture Notes"). The sidebar shows the name; search does not key off category name by default but could be extended to filter by category.
- **Section titles** can duplicate across categories (e.g. two sections titled "Overview" in different categories). Navigation is by (category, section) pair, not by title alone.

### Serialization format (YAML)

- Unity serializes ScriptableObject assets as **YAML** (e.g. `DocumentationData.asset` is a text file with `%YAML 1.1` and `MonoBehaviour:` blocks). Lists appear as `- element` or nested lists. Object references are stored as `fileID` and `guid`. This allows the asset to be version-controlled and diffed; merge conflicts can be resolved by hand or by re-running Rebuild for the "Project Scripts" part.

### When is data saved?

- **After Rebuild**: `DocumentationRebuild.Rebuild()` calls `EditorUtility.SetDirty(data)` and then `AssetDatabase.SaveAssets()`, so the asset is written to disk immediately after a full rebuild.
- **After Rebuild Index**: Only `SetDirty(data)` is called from the renderer; the user must save the project (Ctrl+S) or rely on Unity’s auto-save to persist the updated script index. To get immediate persistence after Rebuild Index, you could add `AssetDatabase.SaveAssets()` in the same code path.
- **Manual edits in Inspector**: Changes are in memory until the user saves (Ctrl+S) or the project is saved. There is no auto-save on every Inspector change.
- **Creating the asset**: `DocumentationDataUtility.EnsureDataAssetExists()` calls `CreateAsset` and then `SaveAssets()`, so the newly created asset is written once at first load.

---

## 6. Script Indexing Engine

### How AssetDatabase.FindAssets works

- **API**: `AssetDatabase.FindAssets(string filter, string[] searchInFolders)`.
- **Use**: `FindAssets("t:MonoScript", new[] { "Assets" })` returns GUIDs of all MonoScript assets under the Assets folder. MonoScript is the asset type for C# scripts.
- **Result**: Array of GUIDs; each GUID is converted to path with `GUIDToAssetPath(guid)`.

### Filtering rules in detail (ScriptIndexer)

Scripts are **skipped** (excluded from the index) when the asset path contains any of the following substrings (case-sensitive in the current implementation):

| Substring | Purpose | Example paths excluded |
|-----------|---------|------------------------|
| `/Editor/` | Editor-only scripts are not part of the game’s public API. | `Assets/MyGame/Editor/MyEditor.cs`, `Assets/Plugins/SomeTool/Editor/Helper.cs` |
| `/Packages/` | Package code is external; indexing can be slow and is usually not needed for project docs. | `Packages/com.unity.xxx/...`, any script under Packages |
| `StudioDocumentation` | Avoid indexing the documentation tool’s own scripts. | `Assets/StudioDocumentation/Editor/ScriptIndexer.cs`, `Assets/StudioDocumentation/Data/DocumentationData.cs` |

- **Included**: Any MonoScript under `Assets` whose path does **not** contain any of the above. Examples: `Assets/indekos/script/core/AudioManager.cs`, `Assets/Game/Scripts/Player.cs`.
- **Additional exclusion**: If `MonoScript.GetClass()` returns **null** (script does not compile or has no type), the script is skipped and no ScriptInfo is added.
- **Extending**: To exclude more folders (e.g. `Tests`, `Samples`), add a check in `ScriptIndexer.ShouldSkipPath` or the equivalent path filter in your codebase.

### How MonoScript.GetClass works

- **API**: `MonoScript.GetClass()` returns the `System.Type` of the class defined in that script (the first or only top-level type). Returns null if the script does not compile or has no type (e.g. only usings).
- **Use**: We get the Type to read `Name`, `Namespace`, `BaseType`, `IsAbstract`, and to call `GetMethods` / `GetFields` with reflection.

### Reflection usage

- **GetMethods(BindingFlags.Public | Instance | DeclaredOnly)**: Only public instance methods declared on that type (not inherited). We skip `IsSpecialName` and names starting with `get_` or `set_` to exclude property accessors.
- **GetFields(BindingFlags.Public | Instance | DeclaredOnly)**: Public instance fields. We skip `IsSpecialName` (e.g. backing fields).
- **FormatMethodSignature**: Builds a string like `ReturnType MethodName(ParamType paramName, ...)` for display.

### Method filtering logic

- **Excluded**: Property getters/setters (`get_*`, `set_*`), other special names (e.g. op_*). This keeps the list focused on real methods and avoids clutter from auto-properties.

### XML comment parsing

- **Source**: Full path is `Path.Combine(Application.dataPath, "..", assetPath)`; we read the .cs file with `File.ReadAllText`.
- **ExtractClassSummary**: We search for a line containing `class <className>`; then we walk backward and collect lines that start with `///`, strip `<summary>`/`</summary>`, and join them into `xmlSummary`. This is a simple heuristic and does not handle multi-line XML or other tags.

### Performance considerations

- **Cost**: One `FindAssets` call; then for each script we load MonoScript, get Type, reflect methods/fields, and read the file. On large projects (hundreds of scripts) this can take a few seconds. Running it on demand (Rebuild Index or Rebuild Documentation) avoids running it on every domain reload.
- **Caching**: The result is stored in `DocumentationData.scriptIndex` and only recomputed when the user clicks Rebuild.

### Limitations

- Only the **first** class summary above the class line is taken; nested or partial classes may not be detected correctly.
- Reflection returns **declared** members only; inherited public methods are not listed (by design, to keep each script’s section focused).
- Scripts that do not compile (GetClass() == null) are skipped.
- Paths containing "Editor", "Packages", or "StudioDocumentation" are skipped so the index focuses on game/runtime scripts.

### Indexing pipeline (ASCII)

```
BuildIndex()
     │
     ▼
FindAssets("t:MonoScript", ["Assets"])
     │
     ▼
For each GUID → path = GUIDToAssetPath(guid)
     │
     ▼
ShouldSkipPath(path)? ──Yes──► skip
     │ No
     ▼
LoadAssetAtPath<MonoScript>(path)
     │
     ▼
type = monoScript.GetClass()  ──null──► skip
     │
     ▼
BuildScriptInfo(type, path)
     │
     ├─► className, namespace, inheritance, assetPath
     ├─► isMonoBehaviour, isScriptableObject, isAbstract
     ├─► GetMethods(Public|Instance|DeclaredOnly) → filter get_/set_/special → publicMethods
     ├─► GetFields(Public|Instance|DeclaredOnly) → publicFields
     └─► TryAttachXmlComments(path, info) → read file → ExtractClassSummary → xmlSummary
     │
     ▼
result.Add(info)
     │
     ▼
Return result (List<ScriptInfo>)
```

---

## 7. Auto Documentation Generation

### When it runs

- **TryAutoPopulateIfEmpty**: Called from `DocumentationWindow.Initialize()` after `LoadOrCreate()`. It counts total sections across all categories; if the count is **0**, it calls `Rebuild(data)`. So the first time the window is opened and the asset has no sections, it auto-generates "Project Scripts" from the script index.
- **Rebuild Documentation (menu or toolbar)**: Always runs when the user explicitly clicks **Tools → Studio → Documentation → Rebuild Documentation** or the **Rebuild Documentation** button in the window. Clears categories, rebuilds index, creates "Project Scripts", saves.

### When it does NOT run

- **Not on every domain reload**: We do not call Rebuild or TryAutoPopulateIfEmpty from `[InitializeOnLoad]` or static constructors. Auto-populate runs only when the window is opened and section count is 0.
- **Not on Rebuild Index only**: The "Rebuild Index" button only updates `scriptIndex`; it does not clear or repopulate categories. So existing manual categories are preserved when you only rebuild the index.

### Manual rebuild process

1. User clicks **Rebuild Documentation** (menu or toolbar).
2. `DocumentationRebuild.Rebuild(data)` runs.
3. `data.categories.Clear()`.
4. `data.scriptIndex = ScriptIndexer.BuildIndex()`.
5. New category "Project Scripts" is created; for each `ScriptInfo` a section is added with title, summary, and richTextContent (namespace, base, flags, methods, fields).
6. Category is added to `data.categories`.
7. `EditorUtility.SetDirty(data)` and `AssetDatabase.SaveAssets()`.

### Safety checks

- Null checks on `data`, `data.categories`, `data.scriptIndex` before use.
- Rebuild is wrapped in try/catch; on exception we log and do not clear the asset.
- EnsureDataAssetExists checks folder validity and existing asset before creating; uses try/catch around CreateAsset.

### Error handling

- Rebuild: `catch (Exception ex) { Debug.LogError(...); }`.
- EnsureDataAssetExists: `catch (Exception ex) { Debug.LogWarning(...); }` so the editor does not crash and the user can retry (e.g. by opening the window again).

### Complete code flow: Rebuild Documentation

1. User clicks **Rebuild Documentation** (toolbar or menu).
2. **DocumentationRenderer** (toolbar button handler) or **DocumentationRebuild.RebuildFromMenu()** (menu item) is invoked.
3. **DocumentationRebuild.Rebuild(data)** is called with the current `_data` (from LoadOrCreate).
4. Inside Rebuild:
   - `data.categories.Clear()` — all existing categories and sections are removed.
   - `data.scriptIndex = ScriptIndexer.BuildIndex()` — full scan: FindAssets("t:MonoScript", ["Assets"]), filter paths, for each script get Type via GetClass(), reflect methods/fields, read .cs for XML summary, build List&lt;ScriptInfo&gt;.
   - Create a new **DocumentationCategory** with name "Project Scripts" (or the configured constant).
   - For each **ScriptInfo** in scriptIndex: create a **DocumentationSection** with title = className, summary = xmlSummary or namespace, tab = Overview (or default), richTextContent = BuildSectionContent(info) (namespace, base type, flags, bullet list of methods, bullet list of fields). Add the section to the category’s sections list.
   - Add the category to `data.categories`.
   - `EditorUtility.SetDirty(data)` and `AssetDatabase.SaveAssets()`.
5. If an exception occurs at any step, it is caught, logged with Debug.LogError, and the method returns without saving (existing data is already cleared in memory but not saved if we exit before SaveAssets; in the current flow we clear first so partial failure may leave data in a cleared state — consider wrapping in try/catch and only clearing when index build succeeds if you need stronger consistency).
6. The window’s _data reference still points to the same asset; the next repaint shows the updated sidebar and content.

---

## 8. Editor UI System

### DocumentationRenderer

- **Role**: Draws toolbar, tabs, and main content (section body, search results, dependency graph, debug panel). Uses IMGUI (`EditorGUILayout`, `EditorStyles`).
- **State**: Holds `_router`, `_data`, `_search` (DocumentationSearchEngine); `SearchQuery`, `DebugMode`.
- **Content flow**: If search query non-empty → draw search results (from `_search.Search()`); else if active section non-null → draw section (title, summary, message box, richTextContent, codeBlock, diagramText, links with Ping/Open); else draw overview message. If active tab is DependencyGraph, also draw ASCII graph. If DebugMode, draw debug stats.

### Drawing order and layout

- **Top to bottom**: Toolbar (search, Refresh, Rebuild Index, Rebuild Documentation, Debug toggle) → Tab row (Overview, Architecture, …) → Horizontal split: left = sidebar (categories/sections), right = scrollable content area.
- **Sidebar**: One foldout per category; inside each foldout, one button per section. Active section is drawn with a different style (e.g. bold). Clicking a section calls `router.NavigateTo(category, section)` and sets the active tab from the section’s `tab` field.
- **Content area**: If search query is set, the content area shows search results (list of section titles with **Open**). Otherwise it shows the active section’s title, summary, message box (if any), rich text, code block, diagram, and links. For Dependency Graph tab, the content is the ASCII graph from DependencyVisualizer. For Debug tab, it shows counts (categories, script index, sensitive files) and active tab name.
- **Scroll**: One scroll view for the content area; the scroll position is stored in DocumentationRouter per active tab so switching tabs restores the previous scroll position.

### IMGUI vs UI Toolkit reasoning

- The current implementation uses **IMGUI** (OnGUI, EditorGUILayout) for simplicity and compatibility. No UXML/USS or runtime UI Document is required; everything is drawn in one EditorWindow. IMGUI is well-supported in Unity Editor and is easy to extend (add a new section type or tab by adding draw logic).
- **UI Toolkit** could be used for a more modern look and better scalability (e.g. virtualized list for many sections); migration is listed in the Roadmap.

### How tabs are rendered

- Tabs are drawn as a row of toggles (`GUILayout.Toggle(isActive, tab.ToString(), TabHeaderStyle)`). Clicking sets `_router.SetActiveTab(tab)`. The main content area uses `_router.ActiveTab` to decide what to show (e.g. Dependency Graph when that tab is active) and uses `_router.GetScrollPosition(ActiveTab)` for the scroll view so each tab remembers its scroll position.

### How search works

- **DocumentationSearchEngine**: Constructor takes `DocumentationData`. `Search(query, tabFilter)` iterates all categories and sections, optionally filters by `tabFilter`, and returns pairs `(category, section)` where the section’s title, summary, richTextContent, codeBlock, or diagramText contains the query (case-insensitive).
- **UI**: Toolbar has a text field bound to `SearchQuery`. When non-empty, `DrawContent` calls `DrawSearchResults()` which displays matching sections and an "Open" button that navigates to that section and clears the search.

### Performance handling

- Search runs in the same frame as the text field change; for very large documentation sets, search could be deferred or debounced (not implemented yet).
- Scroll position is stored per tab to avoid recalculating layout when switching tabs.
- Script index and rebuild are run on demand, not every frame.

---

## 9. Error Handling & Stability

### Why asset creation can fail

- **Type in Editor assembly**: If the ScriptableObject type is defined in an Editor-only assembly, Unity’s asset pipeline may fail to serialize or import the asset, leading to "Creating asset at path ... failed" or "Unable to import newly created asset". **Fix**: Move the ScriptableObject and all serialized types to a non-Editor assembly (StudioDocumentation).
- **Asset database not ready**: During domain reload, calling CreateAsset too early can fail. **Fix**: Defer with `EditorApplication.delayCall`.
- **Invalid path**: Creating an asset in a non-existent folder can fail. **Fix**: Ensure folder exists with `AssetDatabase.CreateFolder` and `Refresh` before CreateAsset.

### Why Editor assemblies cannot host ScriptableObject (for asset creation)

- The asset is a serialized blob that references a script type by GUID/ID. At import time, Unity needs to resolve that type. Editor-only assemblies are not part of the player build; the asset pipeline may treat them differently and fail to create or reimport assets that reference Editor-only types. Putting the type in a non-Editor assembly ensures the type is always available for serialization and import.

### Domain reload issues

- Static constructors run after reload; at that moment other systems (e.g. AssetDatabase) might not be fully initialized. Running heavy or asset-modifying code in static constructors can cause flaky failures. **Mitigation**: Use `EditorApplication.delayCall` to defer work to the next tick.

### Defensive programming decisions

- **Try/catch** around CreateAsset, DeleteAsset, and Rebuild so one failure does not crash the editor.
- **Null checks** on `_data`, `data.categories`, `data.scriptIndex` before use in renderer and rebuild.
- **Section count check** before auto-populate so we only rebuild when there are zero sections (avoid overwriting user content on every open).

### Try/catch strategy

- **EnsureDataAssetExists**: Catch around CreateAsset; log warning and return. No rethrow.
- **Rebuild**: Catch around the whole rebuild; log error with message and stack trace; no rethrow so the window stays open and the user can fix data or try again.

### Decision log (why we chose this design)

| Decision | Rationale |
|----------|-----------|
| **Two assemblies (data vs Editor)** | ScriptableObject and serialized types must live in a non-Editor assembly so Unity’s asset pipeline can create and import the asset. Editor-only assemblies are not guaranteed to be available in the same way during asset operations. |
| **ScriptableObject in data assembly** | Avoids "Creating asset failed" and "Unable to import newly created asset" errors that occur when the asset’s type is in an Editor assembly. |
| **EditorApplication.delayCall for asset creation** | Static constructors run during domain reload when the asset database may not be ready. Deferring creation to the next editor tick ensures CreateAsset runs in a safe state. |
| **IMGUI first, UI Toolkit later** | IMGUI requires no UXML/USS, works with the existing EditorWindow lifecycle, and is quick to extend. UI Toolkit would improve scalability (e.g. virtualized lists) but requires a larger refactor; it is planned in the roadmap. |
| **Single asset (DocumentationData.asset)** | One place for all categories, script index, and sensitive files; simple load/save and no need to manage multiple assets. Optional multi-asset support could be added later. |
| **TryAutoPopulateIfEmpty only when section count is 0** | Prevents overwriting user-added categories on every window open. First-time users get content automatically; returning users keep their manual edits until they explicitly run Rebuild. |
| **Rebuild clears all categories** | Keeps the rebuild logic simple and avoids complex merge rules. Users who need manual categories can use Rebuild Index only and add categories by hand, or extend the code to merge. |
| **Reflection with DeclaredOnly for methods** | Each script’s section shows only that class’s API surface, not inherited methods, so the doc stays focused and avoids duplication across inheritance hierarchies. |
| **Search as linear scan** | Simple and correct for moderate section counts. For very large docs, a prebuilt index or filtered search could be added later. |

---

## 10. Extending the System

### How to add custom categories

- **Data**: Add a new `DocumentationCategory` to `DocumentationData.categories` (in code or via a custom editor). Set `name`, optional `icon`, and add `DocumentationSection` items to `sections`.
- **UI**: The sidebar already iterates `_data.categories`; no code change needed. New categories appear in the tree automatically.

### How to add custom tabs

- **Data**: Add a new value to the `DocumentationTab` enum in `DocumentationSection.cs`. Rebuild the data assembly.
- **UI**: In `DocumentationRenderer.DrawTabs()`, the enum is iterated with `foreach (DocumentationTab tab in Enum.GetValues(typeof(DocumentationTab)))`, so the new tab appears automatically. In `DrawContent()`, add a branch for the new tab (e.g. draw custom content when `_router.ActiveTab == DocumentationTab.YourNewTab`).

### How to integrate AI summary generator

- **Idea**: Before or after `ScriptIndexer.BuildIndex()`, call an external API or local model to generate a short summary per class; write it into `ScriptInfo.xmlSummary` or into a new field. Then when building sections in `DocumentationRebuild.BuildSectionContent`, use that summary.
- **Place**: Extend `DocumentationRebuild.Rebuild()` or add a post-process step that iterates `data.scriptIndex` and enriches `xmlSummary`. Keep API keys out of the repo (e.g. environment variable or EditorPrefs).

### How to add markdown support

- **Data**: Store markdown in `DocumentationSection.richTextContent` (or a new field like `markdownContent`). 
- **Rendering**: In `DocumentationRenderer`, when drawing a section that has markdown, use a markdown-to-rich-text converter (e.g. a small parser or a library that outputs Unity rich text) and pass the result to `EditorGUILayout.LabelField(..., RichTextStyle)`. Alternatively, use UI Toolkit and a Markdown renderer component.

### How to integrate code preview

- **Data**: `DocumentationSection` already has `codeBlock` (plain text). You can fill it from `ScriptInfo` or from file read (e.g. first N lines of the script).
- **Rendering**: In `DrawContent()`, the code block is already drawn in a helpBox with a monospace-style label. For syntax highlighting, you would need a custom drawer or a third-party editor syntax highlighter and draw the code in a scrollable area with colored segments.

### How to add search indexing

- **Current**: Search is in-memory over title, summary, richTextContent, codeBlock, diagramText. No separate index.
- **Extension**: For very large docs, build a simple index (e.g. list of (sectionId, list of words)) when data loads or when sections change; search could then look up words in the index and jump to section IDs. Alternatively, use a full-text index library (e.g. Lucene-style) if you need fuzzy or ranked search.

### Code example: Adding a new tab

1. In `DocumentationSection.cs` (data assembly), add to the enum:
   ```csharp
   public enum DocumentationTab
   {
       // ... existing ...
       TechnicalDebt,
       MyCustomTab   // new
   }
   ```
2. In `DocumentationRenderer.DrawContent()`, add a branch:
   ```csharp
   if (_router.ActiveTab == DocumentationTab.MyCustomTab)
   {
       EditorGUILayout.LabelField("My Custom Tab", DocumentationStyle.HeaderStyle);
       EditorGUILayout.HelpBox("Custom content here.", MessageType.Info);
   }
   ```
   The tab will appear in the tab bar automatically because `DrawTabs()` iterates the enum.

### Code example: Adding a category programmatically

```csharp
var data = DocumentationDataUtility.LoadOrCreate();
if (data != null)
{
    var cat = new DocumentationCategory { name = "My Category", isExpanded = true };
    cat.sections.Add(new DocumentationSection
    {
        title = "First Section",
        summary = "Summary text.",
        tab = DocumentationTab.Overview,
        richTextContent = "Body content."
    });
    data.categories.Add(cat);
    EditorUtility.SetDirty(data);
    AssetDatabase.SaveAssets();
}
```

---

## 11. Security & Sensitive Files

### What SensitiveFile is

- **SensitiveFile** is a serializable class with `label`, `asset` (Object reference), `level` (SensitiveLevel: Core, High, Medium, Low), and `notes` (TextArea). It is stored in `DocumentationData.sensitiveFiles`.

### Why we track sensitive files

- To document which scripts or assets are **critical** or **do-not-touch** so that new or external developers do not modify them without review. It supports compliance and code ownership.

### How to classify sensitivity

- **Core**: Central systems (e.g. GameManager, SaveLoadManager); changing them can break the whole project.
- **High**: Important but more localized (e.g. a specific manager or quest format parser).
- **Medium**: Feature-level scripts that should be changed with care.
- **Low**: Optional or experimental; lower risk.

### Example of internal studio usage

- Add entries in the Inspector for `GameManager`, `GameState`, `QuestManager.Format`, `SaveData`, etc. In notes, write "Do not change trigger strings without updating GameplayState switch" or "Format string protocol: see QuestManager.Format()." A future tab could filter or highlight "Sensitive files" and show these entries with their level and notes.

---

## 12. Performance Considerations

- **Reflection cost**: ScriptIndexer uses Type.GetMethods and GetFields once per script during BuildIndex. On hundreds of scripts this can take 1–3 seconds. We do not run it every frame; only on Rebuild Index or Rebuild Documentation.
- **Asset scanning cost**: FindAssets("t:MonoScript") is a single DB query; loading each MonoScript and calling GetClass() has a cost proportional to script count. Acceptable for on-demand rebuild.
- **Caching**: The script index is stored in the asset; no need to re-scan until the user requests it.
- **Lazy loading**: The DocumentationData asset is loaded when the window opens (LoadOrCreate); we do not load it at editor startup.
- **Avoiding domain reload regeneration**: We do not call Rebuild or full index in static constructors or InitializeOnLoad; only EnsureDataAssetExists (create asset if missing) is deferred with delayCall. Auto-populate runs only when the window is opened and section count is 0.

### Performance expectations (rough guide)

| Script count | Rebuild Index / Rebuild Documentation | Sidebar draw | Search (in-memory) |
|--------------|----------------------------------------|--------------|---------------------|
| &lt; 50       | &lt; 1 s                                 | Negligible   | &lt; 10 ms           |
| 50–200       | 1–3 s                                  | Fast         | &lt; 50 ms           |
| 200–500      | 3–10 s                                 | May lag      | &lt; 100 ms          |
| 500+         | 10 s+                                  | Consider filtering or virtualization | Linear scan cost |

These are approximate; actual times depend on disk speed, reflection cost, and Unity version. For very large projects, consider excluding more folders in ScriptIndexer or adding a progress bar and optional background indexing.

### EditorPrefs and optional persistence

- The documentation window does **not** currently save window state (active tab, scroll position, sidebar width) to EditorPrefs. State is in-memory and resets when the window is closed. To persist:
  - Store **ActiveTab** and **scroll positions** in OnDisable (e.g. EditorPrefs.SetInt("StudioDoc.ActiveTab", (int)_router.ActiveTab)) and restore in OnEnable.
  - Store **sidebar width** if you use a resizable split (EditorPrefs.SetFloat("StudioDoc.SidebarWidth", width)).
- **Category isExpanded** is stored in the asset (DocumentationCategory.isExpanded), so expand/collapse state persists with the asset.

---

## 13. Troubleshooting Guide (Very Detailed)

| Symptom | Root cause | Fix |
|--------|------------|-----|
| **CS0103: ScriptIndexer does not exist** | ScriptIndexer is in namespace `StudioDocumentation.Editor` and the caller (e.g. DocumentationRenderer) had no `using StudioDocumentation.Editor`. | Add `using StudioDocumentation.Editor;` in the file that calls ScriptIndexer (e.g. DocumentationRenderer.cs). |
| **Missing assembly references** | Editor asmdef does not reference the data assembly, so types like DocumentationData or ScriptInfo are not found. | In `StudioDocumentation.Editor.asmdef`, set `references` to `["StudioDocumentation"]`. Ensure StudioDocumentation.asmdef exists and compiles. |
| **ScriptIndexer not found** | Same as CS0103; or ScriptIndexer is in a different assembly. | Ensure ScriptIndexer.cs is under Assets/StudioDocumentation/Editor/ and has namespace StudioDocumentation.Editor. Ensure DocumentationRenderer has `using StudioDocumentation.Editor`. |
| **Asset import failed / Creating asset failed** | The ScriptableObject type was in an Editor-only assembly, so Unity could not serialize the asset. | Move DocumentationData and all serialized types (DocumentationCategory, DocumentationSection, ScriptInfo, SensitiveFile, etc.) to a non-Editor assembly (e.g. StudioDocumentation) and reference that assembly from the Editor asmdef. Create the asset again at a valid path (e.g. Resources/DocumentationData.asset). |
| **NullReference when opening window** | DocumentationData asset missing or not loaded; LoadOrCreate returned null. | Ensure Assets/StudioDocumentation/Resources/DocumentationData.asset exists. If not, let EnsureDataAssetExists create it (open window once; if it still fails, check Console for "Could not create DocumentationData.asset" and fix assembly/type as above). |
| **Documentation not generating** | Section count was not 0, so TryAutoPopulateIfEmpty did not run; or Rebuild was never clicked. | Click **Rebuild Documentation** in the window toolbar or **Tools → Studio → Documentation → Rebuild Documentation**. If the index is empty, click **Rebuild Index** first, then Rebuild Documentation. |
| **Menu not appearing** | Editor script not compiled or MenuItem not reached. | Check Console for compile errors in StudioDocumentation.Editor. Fix any CS0103 or missing reference. Ensure DocumentationWindow and DocumentationRebuild have correct MenuItem attributes. |
| **Editor freeze** | Rebuild runs on a very large project and blocks the main thread. | Expected for hundreds of scripts; wait for it to finish. Future improvement: run BuildIndex in a background task or show a progress bar. |
| **Performance slow** | Large script count; reflection and file read for each script. | Use Rebuild only when needed. Consider excluding more folders in ShouldSkipPath or adding a "Max scripts" limit for the index (not implemented). |

| **Sidebar empty** | DocumentationData has no categories or categories have no sections. | Run **Rebuild Documentation** to generate "Project Scripts", or add categories/sections manually in the asset Inspector. |
| **Search returns nothing** | Query does not match any section title, summary, content, code, or diagram (case-insensitive substring). | Broaden the search term or check that sections have content in at least one of those fields. |
| **Rebuild overwrote my manual categories** | Rebuild clears all categories and creates only "Project Scripts". | By design. If you need to keep manual categories, do not use Rebuild; use **Rebuild Index** only and then add a new category manually, or extend the code to merge instead of replace. |
| **Window does not dock** | EditorWindow is dockable by default. | Use the window tab and drag it to the desired dock area. If it does not dock, check Unity version and that the window class extends EditorWindow. |
| **Duplicate type errors** | Same type (e.g. DocumentationData) defined in both the data assembly and the Editor assembly. | Remove the duplicate definition from the Editor assembly; keep only the definition in the data assembly and reference it from the Editor. |
| **Build fails with StudioDocumentation** | The data assembly is included in the build. | The data assembly contains only data types and no MonoBehaviours or runtime logic; it should not affect the build. If the build fails, ensure no Editor code (e.g. UnityEditor) is referenced from the data assembly. |

### Common pitfalls

- **Putting ScriptableObject in the Editor assembly**: Causes "Creating asset failed" or import errors. Always keep ScriptableObject and serialized data types in a non-Editor assembly (StudioDocumentation).
- **Calling CreateAsset in a static constructor**: Can run during domain reload when the asset database is not ready. Use `EditorApplication.delayCall` to defer creation.
- **Forgetting to add `using StudioDocumentation.Editor`**: DocumentationRenderer and DocumentationWindow call ScriptIndexer and DocumentationRebuild; without the using, you get CS0103 (ScriptIndexer does not exist). Add the namespace.
- **Expecting inherited methods in the script index**: We use DeclaredOnly; only methods declared on the class are listed. This is intentional so each section stays focused on that script.
- **Rebuilding and losing manual categories**: Rebuild clears *all* categories and creates only "Project Scripts". If you added custom categories, they will be removed. Use Rebuild Index only if you want to keep existing categories and only refresh the script list.
- **Search returns no results**: Search is case-insensitive substring in title, summary, richTextContent, codeBlock, diagramText. If the section has no content in those fields, it will not match. Add at least a title and summary.
- **Dependency Graph empty**: The graph is built from `data.scriptIndex`. If you never ran Rebuild Index or Rebuild Documentation, the list is empty. Click Rebuild Index or Rebuild Documentation once.

---

## 14. FAQ Section

1. **Q: What is StudioDocumentation?**  
   A: An editor-only documentation framework: a window (Tools → Studio → Documentation) that shows project docs and can auto-generate script documentation from C# files.

2. **Q: Does it run in builds?**  
   A: No. The Editor assembly is excluded from builds. The data assembly is included but has no runtime logic; it only holds data types.

3. **Q: Where is the documentation stored?**  
   A: In a ScriptableObject asset at `Assets/StudioDocumentation/Resources/DocumentationData.asset`.

4. **Q: Can I edit the asset manually?**  
   A: Yes. Select the asset and edit categories/sections in the Inspector. You can add links (scripts, scenes, prefabs) and edit rich text, code blocks, and diagrams.

5. **Q: What does "Rebuild Index" do?**  
   A: It runs the script indexer (scans all C# scripts under Assets, excluding Editor/Packages/StudioDocumentation) and saves the result into DocumentationData.scriptIndex. It does not change categories.

6. **Q: What does "Rebuild Documentation" do?**  
   A: It clears all categories, rebuilds the script index, creates one category "Project Scripts" with one section per script (title, summary, methods, fields), and saves the asset.

7. **Q: When does auto-generation run?**  
   A: Only when you open the documentation window and the asset has zero sections. Then TryAutoPopulateIfEmpty runs once and calls Rebuild.

8. **Q: Why is my ScriptableObject creation failing?**  
   A: Usually because the type is in an Editor-only assembly. Move the type to a non-Editor assembly (e.g. StudioDocumentation) and reference it from the Editor assembly.

9. **Q: Can I add more categories?**  
   A: Yes. Add DocumentationCategory instances to DocumentationData.categories (in code or via a custom inspector). The sidebar will show them.

10. **Q: Can I add more tabs?**  
    A: Yes. Add a value to DocumentationTab enum; then in DocumentationRenderer.DrawContent add handling for that tab.

11. **Q: How do I exclude a script from the index?**  
    A: Scripts under folders containing "/Editor/", "/Packages/", or "StudioDocumentation" are already excluded. To exclude more, add a path check in ScriptIndexer.ShouldSkipPath.

12. **Q: What are SensitiveFile entries for?**  
    A: To mark assets (e.g. core scripts) as sensitive and add notes. The data is stored for future UI (e.g. a "Sensitive" tab or warnings when opening those assets).

13. **Q: Why two assemblies?**  
    A: So that the ScriptableObject and serialized types live in a non-Editor assembly and Unity can create/import the asset reliably. The Editor assembly references the data assembly and uses those types.

14. **Q: Can I use UI Toolkit instead of IMGUI?**  
    A: Yes, but it would require rewriting DocumentationWindow, DocumentationSidebar, and DocumentationRenderer to use UIDocument and UI Builder. Planned in the roadmap.

15. **Q: Does search work across all tabs?**  
    A: Search filters by section content; you can optionally filter by active tab (DocumentationSearchEngine.Search has a tabFilter parameter). The current UI passes the active tab so results are relevant to the current tab.

16. **Q: Why is Dependency Graph empty?**  
    A: The script index is empty. Click "Rebuild Index" or "Rebuild Documentation" to populate it.

17. **Q: Can I export documentation to HTML or PDF?**  
    A: Not built-in. Planned in the roadmap. You could write an editor script that reads DocumentationData and writes HTML/Markdown/PDF.

18. **Q: How do I add XML summaries to my scripts?**  
    A: Add `/// <summary>Your text</summary>` above the class (and optionally above methods). The indexer reads the class summary and stores it in ScriptInfo.xmlSummary.

19. **Q: Are inherited methods included?**  
    A: No. We use BindingFlags.DeclaredOnly so only methods declared on that class are listed. This keeps each script’s section focused.

20. **Q: What if a script doesn’t compile?**  
    A: MonoScript.GetClass() returns null and the script is skipped. Fix compile errors and rebuild the index.

21. **Q: Can I run Rebuild from script?**  
    A: Yes. Call `DocumentationRebuild.Rebuild(data)` with your DocumentationData reference (e.g. from LoadOrCreate()).

22. **Q: Where is the legacy asset path?**  
    A: Previously the asset was at Assets/StudioDocumentation/DocumentationData.asset. It was moved to Resources to avoid import issues. The code tries to delete the old path when creating the new asset so no broken asset remains.

23. **Q: Why EditorApplication.delayCall?**  
    A: To defer asset creation to the next editor tick so it does not run during domain reload when the asset database might not be ready.

24. **Q: Can I have multiple DocumentationData assets?**  
    A: The window loads one asset (Resources/DocumentationData.asset). You could extend the tool to support multiple assets or a project setting that points to another path.

25. **Q: How do I add a link to a script in a section?**  
    A: In the Inspector, add an element to the section’s Links list: set label (e.g. "GameManager") and drag the script or asset into Target. In the window, use Ping or Open to jump to it.

26. **Q: What is DocumentationRouter?**  
    A: It holds the current tab, current category/section, and per-tab scroll positions. It is the navigation state for the window.

27. **Q: What is DocumentationStyle?**  
    A: A static class that initializes GUIStyles (headers, sidebar, info/warning/critical boxes, rich text) once, with light/dark skin awareness.

28. **Q: Why internal static for ScriptIndexer?**  
    A: So it is only used inside the Editor assembly (no need to expose it to other assemblies). Static so no instance is required; BuildIndex is a static method.

29. **Q: Can I customize the "Project Scripts" category name?**  
    A: Yes. In DocumentationRebuild, change the constant `ProjectScriptsCategoryName`.

30. **Q: How do I add a new section type (e.g. video)?**  
    A: Add fields to DocumentationSection (e.g. videoUrl) and in DocumentationRenderer.DrawContent add drawing logic for that field when present.

31. **Q: Is the script index stored in the asset?**  
    A: Yes. DocumentationData.scriptIndex is a list of ScriptInfo; it is serialized with the asset and saved to disk.

32. **Q: What happens if I delete the DocumentationData asset?**  
    A: Next time you open the window, EnsureDataAssetExists will create a new empty one at Resources/DocumentationData.asset. You will lose previous content unless you have a backup.

33. **Q: How do I backup my documentation?**  
    A: Copy the file `Assets/StudioDocumentation/Resources/DocumentationData.asset` (and its .meta) to a safe location or version control. The asset is a YAML file and can be diffed and merged like other Unity assets.

34. **Q: Can I use this in a team with version control?**  
    A: Yes. Commit the DocumentationData asset and the StudioDocumentation folder. Rebuild Documentation will change the asset; team members can pull and get the updated script list. Resolve conflicts like any other asset (prefer keeping one side and re-running Rebuild if needed).

35. **Q: Why is the data assembly included in the build?**  
    A: Unity includes all assemblies that are not Editor-only by default. The data assembly has no MonoBehaviour or runtime code; it only defines types. It does not execute in the player and has negligible impact. Keeping it out of the build would require an asmdef that excludes Player, which is possible but not required for correctness.

36. **Q: Can I run the indexer on a subset of folders?**  
    A: Yes. Change `AssetDatabase.FindAssets("t:MonoScript", new[] { "Assets" })` to pass a different folder list (e.g. `new[] { "Assets/Game", "Assets/Shared" }`). You can also add more skip rules in ShouldSkipPath.

37. **Q: What Unity version is required?**  
    A: The code uses EditorWindow, ScriptableObject, AssetDatabase, EditorGUILayout, and Reflection APIs that exist in Unity 2019.4+ and 2020+. For older versions, some APIs may need adjustment.

38. **Q: How do I change the asset path from Resources?**  
    A: Change the `AssetPath` constant in DocumentationDataUtility and ensure the folder exists. LoadOrCreate() and EnsureDataAssetExists() use that path. If you move the asset, delete the old one to avoid duplicates.

39. **Q: Can sections have multiple code blocks?**  
    A: The current model has one `codeBlock` and one `diagramText` per section. For multiple blocks, put them in `richTextContent` with labels, or extend DocumentationSection with a list of code/diagram strings.

40. **Q: Is there a way to export only "Project Scripts" to a text file?**  
    A: Not built-in. You can write an editor script that loads DocumentationData, finds the "Project Scripts" category, and writes each section’s title and content to a file (e.g. Markdown or CSV).

---

41. **Q: Why is the window slow with hundreds of sections?**  
    A: The sidebar draws every section as a button and search scans all sections linearly. For very large projects, consider filtering the sidebar by tab or adding pagination; optimize search with a prebuilt index or limit search scope.

42. **Q: Can I show method-level XML summaries in the generated section?**  
    A: The indexer currently extracts only the class-level `/// <summary>`. To show method summaries, extend ScriptIndexer to parse XML above each method and add a field to ScriptInfo (e.g. methodSummaries), then in DocumentationRebuild.BuildSectionContent include them in the rich text.

43. **Q: What if two scripts have the same class name in different namespaces?**  
    A: The script index stores both; sections are keyed by section title (class name). The sidebar and content will show both; the section title could be extended to show namespace (e.g. "NS1.MyClass") to avoid ambiguity.

44. **Q: Can I use this in a package (not under Assets)?**  
    A: The current path constants point to Assets/StudioDocumentation/Resources. To use inside a package, change the asset path to a package path (e.g. Packages/com.yourcompany.studiodocumentation/...) and ensure the data assembly is in that package. FindAssets would need to include package paths if you want to index package scripts.

45. **Q: How do I reset the window layout (sidebar width, scroll)?**  
    A: Scroll position is in-memory (DocumentationRouter); closing and reopening the window resets it. Sidebar width is typically a fixed or resizable split; if stored in EditorPrefs, clear the key for that window to reset.

46. **Q: Can I have different DocumentationData assets per branch or project?**  
    A: The code uses a single path constant. To support multiple assets, you would need to change LoadOrCreate and the toolbar to allow selecting or switching between assets (e.g. via a dropdown or project setting).

47. **Q: Does the indexer run in the background?**  
    A: No. BuildIndex runs on the main thread when you click Rebuild Index or Rebuild Documentation. For large projects it can block for several seconds; a future improvement could be to run it on a background thread or with a progress bar.

48. **Q: What encoding are the .cs files read with?**  
    A: Typically System.IO.File.ReadAllText uses UTF-8 by default (or the system default). For XML parsing we assume the file is valid UTF-8 or ASCII; if your scripts use another encoding, the class summary might be wrong or empty.

49. **Q: Can I add images to a section?**  
    A: The current model has no image field. You could store a texture reference in a new DocumentationSection field and draw it in the renderer with GUILayout.Box or similar, or embed image URLs in richTextContent if you add HTML/rich-text image support.

50. **Q: How do I rename the DocumentationData asset?**  
    A: Rename the file and .meta in the Project window, then update the `AssetPath` constant in DocumentationDataUtility to the new path so LoadOrCreate and EnsureDataAssetExists find it.

51. **Q: Why are some scripts missing from "Project Scripts"?**  
    A: Scripts under paths containing "/Editor/", "/Packages/", or "StudioDocumentation" are skipped. Scripts that do not compile (GetClass() returns null) are skipped. Check ScriptIndexer.ShouldSkipPath and fix compile errors.

52. **Q: Can I sort sections within a category?**  
    A: Sections are stored in a list; order is preserved. You can sort the list in code (e.g. by title) before or after adding sections, or implement a custom inspector that reorders the list.

53. **Q: Is there a way to diff DocumentationData between two commits?**  
    A: The asset is a YAML file; use git diff or any text diff tool on DocumentationData.asset. Merge conflicts can be resolved manually; after merging, you may want to run Rebuild Documentation to regenerate "Project Scripts" if script list changed.

54. **Q: Can the window open a specific section by ID or title from script?**  
    A: Not by default. You could add a static or menu method that finds a section by title (or _id), then opens the window and calls router.NavigateTo(category, section) with the right category/section reference.

55. **Q: What happens if I move the StudioDocumentation folder?**  
    A: Update the path in DocumentationDataUtility (and any other path constants). The asset path is relative to the project; if you move the whole folder, Unity will see the asset at the new location. Update asmdef paths if they reference the folder.

56. **Q: Can I use custom GUIStyles for section content?**  
    A: Yes. DocumentationStyle exposes static styles; you can add new ones in DocumentationStyle and use them in DocumentationRenderer when drawing section content (e.g. a custom code style or header level).

57. **Q: Why is the script index a list and not a dictionary?**  
    A: Unity serializes lists natively; order is preserved (e.g. alphabetical or discovery order). Lookup by class name is done by linear scan when building sections or search; for very large indexes a dictionary could be built at load time if needed.

58. **Q: Can I run the documentation window in headless or CI?**  
    A: Editor windows require a display; headless Unity (e.g. batch mode) typically does not show windows. You could run Rebuild from a script that loads the asset and calls DocumentationRebuild.Rebuild(data) without opening the window, which may work in batch mode if the Editor assembly is loaded.

59. **Q: How do I add a "Last updated" timestamp to the doc?**  
    A: Add a string field to DocumentationData (e.g. lastRebuildTime) and set it in DocumentationRebuild.Rebuild after a successful rebuild (e.g. DateTime.Now.ToString()). Draw it in the renderer (e.g. in the toolbar or at the top of content).

60. **Q: Can sections reference other sections?**  
    A: The model has no built-in "section link" type. You can put a section title or ID in richTextContent and rely on search for navigation, or add a new field (e.g. List&lt;DocumentationSection&gt; relatedSections) and draw links that call router.NavigateTo.

61. **Q: What Unity Editor version is the minimum?**  
    A: The code uses APIs available in Unity 2019.4 LTS and 2020.x. For 2018.x or older, some APIs (e.g. AssetDatabase, EditorGUILayout) may need minor adjustments.

62. **Q: Can I hide the "Project Scripts" category from the sidebar?**  
    A: The sidebar iterates all categories; to hide one, you could add a "hidden" flag to DocumentationCategory and skip drawing it in DocumentationSidebar, or filter by name (e.g. skip if name == ProjectScriptsCategoryName and a pref is set).

63. **Q: How do I get a list of all script paths from the index?**  
    A: Iterate data.scriptIndex and read each ScriptInfo.assetPath. No API is exposed; do it in an editor script that loads DocumentationData.

64. **Q: Is the documentation window state (active tab, scroll) saved when Unity closes?**  
    A: Not by default. DocumentationRouter holds state in memory. To persist, you could save ActiveTab and scroll positions to EditorPrefs in OnDisable and restore in OnEnable.

65. **Q: Can I have multiple windows open?**  
    A: Each window would have its own DocumentationWindow instance and thus its own router/sidebar/renderer. Opening the menu twice may create two windows; they would share the same underlying DocumentationData asset (loaded via LoadOrCreate), so edits in one would reflect in the other after refresh.

66. **Q: What is the maximum number of sections or categories?**  
    A: There is no hard limit in the code. Unity and IMGUI can handle hundreds of sections; performance may degrade with very large lists (sidebar draw time, search scan). Use filtering or pagination for thousands of sections.

67. **Q: Can I change the default tab when opening the window?**  
    A: Set the initial ActiveTab in DocumentationRouter when the window opens (e.g. in DocumentationWindow.Initialize or when creating the router). The default is typically Overview.

68. **Q: How do I copy a section from one category to another?**  
    A: In code: create a new DocumentationSection with the same field values (or clone) and add it to another category’s sections list; SetDirty and SaveAssets. In the Inspector: duplicate the section in the list, then move the copy to another category if the list UI allows reordering across categories (otherwise do it in code).

69. **Q: Does the indexer handle nested or partial classes?**  
    A: MonoScript.GetClass() returns the first top-level type in the file. Nested types and additional top-level types in the same file are not indexed separately; each .cs file yields at most one ScriptInfo.

70. **Q: Can I add a keyboard shortcut to open the documentation window?**  
    A: Yes. MenuItem supports a second parameter: `[MenuItem("Tools/Studio/Documentation %#d")]` (Ctrl+Shift+D). Use Unity’s shortcut system; avoid conflicts with existing shortcuts.

71. **Q: What file extension does the asset have?**  
    A: The asset is saved as `DocumentationData.asset` (Unity’s default for ScriptableObject). The actual format is YAML inside.

72. **Q: How do I validate that DocumentationData is not corrupted?**  
    A: Load the asset and check for null categories or scriptIndex; ensure each category has a name and sections list; ensure each section has a non-null title. You could add a menu item "Validate Documentation Data" that runs these checks and logs warnings.

73. **Q: Can the Dependency Graph show references between scripts?**  
    A: The current graph groups by inheritance (base type). To show "uses" or "references" you would need to analyze code (e.g. parse or use Roslyn) to build a reference graph and then render it in DependencyVisualizer.

74. **Q: Is there a way to export the Dependency Graph as text?**  
    A: BuildAsciiGraph already returns a string. You could add a "Copy to clipboard" button that copies that string, or write it to a file from a menu item.

75. **Q: How do I document a non-MonoBehaviour class (e.g. static helper)?**  
    A: If it’s in a .cs file under Assets (and not skipped), it will be indexed. GetClass() returns the type; the section will show isMonoBehaviour: false. No special handling needed.

76. **Q: Can I use rich text (bold, color) in section content?**  
    A: Yes. Use Unity rich text tags in richTextContent (e.g. &lt;b&gt;, &lt;i&gt;, &lt;color=#rrggbb&gt;). Draw with a GUIStyle that has richText enabled (e.g. DocumentationStyle.RichTextStyle).

77. **Q: What if my script has multiple top-level classes?**  
    A: GetClass() returns the first. Only one ScriptInfo is created per file. To index all types, you would need to parse the file or use a different API.

78. **Q: Can I reorder categories in the sidebar?**  
    A: Categories are a list; order is the order in the list. Reorder in the Inspector by moving list elements, or in code by reordering data.categories.

79. **Q: How do I find which category a section belongs to?**  
    A: Iterate data.categories and check category.sections.Contains(section) or match by section._id.

80. **Q: Is DocumentationData.asset in .gitignore?**  
    A: Typically no; commit it so the team shares the same doc. If you want per-developer local docs, add the asset to .gitignore and document that each developer runs Rebuild locally.

81. **Q: Can I show a progress bar during Rebuild?**  
    A: Use EditorUtility.DisplayProgressBar in a loop (e.g. while iterating scriptIndex) and ClearProgressBar when done. Wrap in try/finally so the bar is cleared on error.

82. **Q: Why is there no "Save" button in the window?**  
    A: Rebuild saves automatically. Manual edits are in the Inspector; the user saves the project (Ctrl+S). A "Save" button could call SetDirty and SaveAssets for the current _data.

83. **Q: Can I use ScriptableObject.CreateInstance from the Editor assembly?**  
    A: Yes. The type is in the data assembly; the Editor assembly references it. CreateInstance&lt;DocumentationData&gt() works from Editor code and returns an instance of the data type.

84. **Q: How do I debug ScriptIndexer output?**  
    A: After BuildIndex(), log result.Count or iterate and Debug.Log each ScriptInfo.className and assetPath. Or inspect data.scriptIndex in the asset after Rebuild Index.

85. **Q: What is the order of sections in "Project Scripts"?**  
    A: The order of ScriptInfo in scriptIndex, which follows the order returned by FindAssets (often alphabetical by path or discovery order). You could sort scriptIndex by className or namespace before building sections.

### Part II — StudioDocumentation glossary

| Term | Meaning |
|------|--------|
| **DocumentationData** | The ScriptableObject root asset that stores categories, script index, and sensitive files. |
| **Script index** | List of ScriptInfo produced by ScriptIndexer; one entry per C# class (excluding Editor/Packages/StudioDocumentation). |
| **Rebuild Index** | Toolbar action that runs ScriptIndexer and saves the result into DocumentationData.scriptIndex only. |
| **Rebuild Documentation** | Full regeneration: clear categories, run ScriptIndexer, create "Project Scripts" category with one section per script, save. |
| **TryAutoPopulateIfEmpty** | Called when the window opens; if total section count is 0, runs Rebuild so first-time users get content. |
| **Data assembly** | StudioDocumentation.asmdef; holds ScriptableObject and serializable types; not Editor-only. |
| **Editor assembly** | StudioDocumentation.Editor.asmdef; holds all window and indexing code; Editor-only. |
| **SensitiveFile** | An entry (label, asset, level, notes) marking an asset as critical or do-not-touch. |
| **DocumentationRouter** | Holds active tab, active category/section, and per-tab scroll positions. |
| **DocumentationSection** | One "page" of content: title, summary, tab, rich text, message, code, diagram, links. |
| **DocumentationCategory** | A group of sections (e.g. "Project Scripts") with a name and optional icon. |
| **Legacy path** | The old asset path (Assets/StudioDocumentation/DocumentationData.asset); removed in favor of Resources path. |

### API summary (Part II) — main entry points

| Class | Purpose | Key methods / entry points |
|-------|---------|----------------------------|
| **DocumentationWindow** | Editor window; menu and layout. | `[MenuItem("Tools/Studio/Documentation")]`; `OnGUI()`; `Initialize()` → LoadOrCreate, TryAutoPopulateIfEmpty. |
| **DocumentationRouter** | Navigation and scroll state. | `ActiveTab`, `ActiveCategory`, `ActiveSection`; `NavigateTo(cat, sec)`; `SetActiveTab(tab)`; `GetScrollPosition(tab)`, `SetScrollPosition(tab, pos)`. |
| **DocumentationSidebar** | Left panel tree. | `OnGUI()` — foldouts, section buttons, NavigateTo on click. |
| **DocumentationRenderer** | Toolbar, tabs, content. | `DrawToolbar()`, `DrawTabs()`, `DrawContent(rect)`; uses SearchQuery, DebugMode; calls ScriptIndexer.BuildIndex, DocumentationRebuild.Rebuild. |
| **DocumentationSearchEngine** | In-memory search. | `Search(query, tabFilter)` → list of (category, section). |
| **DocumentationStyle** | GUIStyles. | `InitIfNeeded()`; static styles: HeaderStyle, SubHeaderStyle, TabHeaderStyle, SidebarSectionStyle, SidebarItemStyle, InfoBox, WarningBox, CriticalBox, RichTextStyle. |
| **DocumentationDataUtility** | Asset load/create. | `[InitializeOnLoad]` → delayCall `EnsureDataAssetExists`; `LoadOrCreate()` → LoadAssetAtPath or create at Resources path. |
| **ScriptIndexer** | Build script index. | `BuildIndex()` → `List<ScriptInfo>`; uses FindAssets, GetClass, reflection, TryAttachXmlComments. |
| **DocumentationRebuild** | Full rebuild. | `Rebuild(data)` — clear categories, BuildIndex, create "Project Scripts", save; `TryAutoPopulateIfEmpty(data)` — if section count 0 then Rebuild; `[MenuItem(".../Rebuild Documentation")]`. |
| **DependencyVisualizer** | ASCII graph. | `BuildAsciiGraph(scriptIndex)` → string; `DrawGraphBox(graph)` in renderer. |
| **DocumentationData** (data) | Root asset. | `categories`, `scriptIndex`, `sensitiveFiles`. |
| **DocumentationSection** (data) | One page. | `title`, `summary`, `tab`, `richTextContent`, `messageType`, `messageText`, `codeBlock`, `diagramText`, `links`. |
| **ScriptInfo** (data) | One script row. | `className`, `namespace`, `inheritance`, `assetPath`, `isMonoBehaviour`, `isScriptableObject`, `isAbstract`, `xmlSummary`, `publicMethods`, `publicFields`. |

---

## 15. Best Practices

- **Clean architecture**: Keep data (ScriptableObject, serializable types) in the data assembly; keep all Editor-only code (windows, indexing, rebuild) in the Editor assembly. Do not reference Editor from data.
- **Assembly boundaries**: Do not move ScriptableObject or types that are serialized in the asset into the Editor assembly. Always reference the data assembly from the Editor assembly, not the other way around.
- **Documentation style guide**: Use clear titles and short summaries for sections; use code blocks for snippets and diagrams for ASCII art; use message boxes (Warning/Critical) for "do not modify" or important caveats.
- **Naming conventions**: Use PascalCase for class and enum names; use descriptive names (DocumentationData, ScriptInfo, SensitiveFile). Use constants for paths and category names (e.g. ProjectScriptsCategoryName).
- **XML comment standards**: Use `/// <summary>...</summary>` above classes (and optionally methods) so the indexer can pick them up. Keep summaries one or two sentences for readability in the generated docs.

### Code conventions (Part II)

- **Namespaces**: Data types live in the default namespace (or a single `StudioDocumentation` namespace if you add one). Editor code uses `StudioDocumentation.Editor` so that ScriptIndexer and DocumentationRebuild are found when you add `using StudioDocumentation.Editor` in the window and renderer.
- **Static vs instance**: ScriptIndexer and DocumentationRebuild use static methods so they can be called without holding a reference. DocumentationWindow, DocumentationRouter, DocumentationSidebar, and DocumentationRenderer are instance-based and hold references to _data and _router.
- **Constants**: Use named constants for paths (e.g. `DocumentationDataUtility.AssetPath`), category names (e.g. `DocumentationRebuild.ProjectScriptsCategoryName`), and menu paths so they can be changed in one place.
- **Error handling**: Wrap asset creation and Rebuild in try/catch; log errors with Debug.LogError and do not throw so the window remains usable. Use null checks after LoadOrCreate() and before iterating categories or scriptIndex.
- **GUIStyle**: DocumentationStyle.InitIfNeeded() is called from the renderer/sidebar so styles are created once. Use the static styles (Header, SubHeader, InfoBox, etc.) for consistency and dark-skin support.

---

## 16. Contribution Guide

- **How to add new features**: Add new tabs in the enum and handle them in DrawContent; add new section fields in the data model and draw them in the renderer; add new menu items or toolbar buttons that call into existing utilities (e.g. Rebuild). Keep data types in the data assembly and UI/editor logic in the Editor assembly.
- **Coding standards**: Follow the existing namespace and assembly layout; use try/catch for asset and rebuild operations; use null checks before using _data or lists.
- **Pull request guidelines**: Ensure no compile errors; ensure the documentation window still opens and Rebuild works; do not add runtime code to the data assembly.
- **Testing process**: Open the window, run Rebuild Index and Rebuild Documentation, switch tabs, search, and navigate sidebar. Verify the asset is created at the correct path and that no errors appear in the Console.

### Testing checklist (before release or PR)

| Check | How to verify |
|-------|----------------|
| Window opens | **Tools → Studio → Documentation** opens without errors. |
| Asset exists | `Assets/StudioDocumentation/Resources/DocumentationData.asset` exists after first open. |
| Auto-populate | With a fresh asset (0 sections), opening the window creates "Project Scripts" and sections. |
| Rebuild Index | Click **Rebuild Index**; scriptIndex in the asset updates; no Console errors. |
| Rebuild Documentation | Click **Rebuild Documentation**; categories replaced by single "Project Scripts"; sections match scripts. |
| Sidebar navigation | Click categories and sections; content area updates; correct section title/summary/body. |
| Tab switch | Switch between Overview, Architecture, …, Dependency Graph; content and scroll position per tab. |
| Search | Type a class name or keyword; results list appears; **Open** navigates to section. |
| Dependency Graph | Tab shows ASCII graph; scripts grouped by base type. |
| Links (Inspector) | Add a link in a section; set Target to a script; save; in window, section shows link; Ping/Open work if implemented. |
| Sensitive files | Add SensitiveFile entry in Inspector; save; data persists (no dedicated UI required for minimal test). |
| No Editor in data asm | Data assembly has no reference to UnityEditor; builds succeed. |
| Domain reload | Change a script outside StudioDocumentation; recompile; reopen window; asset still loads. |

---

## 17. Versioning Strategy

- **Semantic versioning**: Use Major.Minor.Patch (e.g. 1.0.0). Major for breaking changes (e.g. data layout or assembly split), Minor for new features (e.g. new tab, new section type), Patch for bug fixes and small improvements.
- **Breaking changes**: Moving ScriptableObject to another assembly or renaming fields in DocumentationData/DocumentationSection can break existing assets. Document in release notes and provide migration steps (e.g. re-save asset or run Rebuild).
- **Migration notes**: When adding new fields to serialized types, use default values or [FormerlySerializedAs] if renaming so existing assets still load.

### Migration from legacy asset path

- **Old path**: `Assets/StudioDocumentation/DocumentationData.asset` (no Resources folder).
- **New path**: `Assets/StudioDocumentation/Resources/DocumentationData.asset`.
- **On first run**: `DocumentationDataUtility.EnsureDataAssetExists()` checks for the asset at the new path. If missing, it creates the Resources folder and the asset there. It then attempts to delete the legacy path asset (in try/catch) so the project does not keep two copies. If you had custom content in the old asset, copy it manually to the new asset before the old one is deleted, or keep a backup. New installations never use the old path.

---

## 18. Roadmap

- **Planned features**: Richer XML parsing (method-level summaries), optional UI Toolkit migration for the window, progress bar for Rebuild on large projects.
- **AI integration**: Optional step after BuildIndex to generate or enhance xmlSummary via an external API or local model.
- **UI Toolkit migration**: Replace IMGUI with UIDocument for better scalability and theming.
- **Cloud sync**: Optional sync of DocumentationData (or export) to a cloud doc or wiki (out of scope for current version).
- **Documentation export (HTML/PDF)**: Editor script or menu to export categories/sections to static HTML or PDF for sharing outside Unity.

---

## 19. Example Workflow

### Step-by-step: From new script to documented

1. **Create a new script** in your game (e.g. `MyNewManager.cs`) under Assets, with a class and a few public methods. Add a class XML summary:
   ```csharp
   /// <summary>Manages feature X for the game.</summary>
   public class MyNewManager : MonoBehaviour
   {
       public void DoSomething() { }
       public int GetCount() => 0;
   }
   ```
2. **Open** **Tools → Studio → Documentation**. The window opens; if the asset had no sections, **TryAutoPopulateIfEmpty** may have already run and created "Project Scripts". If not, continue.
3. Click **Rebuild Documentation** (or **Rebuild Index** then **Rebuild Documentation** if the index was empty). Wait for the progress to finish (a few seconds on large projects).
4. In the sidebar, expand **Project Scripts** (if collapsed) and click **MyNewManager**.
5. **View result**: The main content shows:
   - Title: MyNewManager
   - Summary: "Manages feature X for the game." (from your XML)
   - Body: namespace, base type (MonoBehaviour), and bullet lists of public methods and fields.
6. **Optional**: Switch to the **Dependency Graph** tab to see where MyNewManager appears (grouped by base type). Use **Search** in the toolbar to find other sections that mention "Manager".

### Step-by-step: Adding a manual section

1. In the Project window, select **Assets/StudioDocumentation/Resources/DocumentationData.asset**.
2. In the Inspector, under **Categories**, add a new element (or use an existing category). Set **Name** to e.g. "Architecture Notes".
3. Under that category’s **Sections**, add a new element. Set **Title** to "High-level design", **Summary** to a short line, **Tab** to Overview, and **Rich Text Content** to your narrative. Optionally set **Message Type** to Warning and **Message Text** to "This is a living document."
4. Under **Links**, add an element: **Label** "GameManager script", **Target** = drag `GameManager.cs` from the Project.
5. Save the asset (Ctrl+S). Reopen the documentation window; in the sidebar you will see "Architecture Notes" and "High-level design". Click it to view; use **Ping** or **Open** on the link to jump to the script.

### Step-by-step: Marking a sensitive file

1. Select **DocumentationData.asset**.
2. Under **Sensitive Files**, add a new element.
3. Set **Label** to "GameManager", **Asset** to the GameManager script or the scene that contains it, **Level** to Core, **Notes** to "Central singleton; do not change trigger strings without updating GameplayState and all trigger prefabs."
4. Save. The data is stored for future use (e.g. a dedicated Sensitive tab or tooling that warns when opening these assets).

### Workflow: Rebuild after adding new scripts

1. Add one or more new C# scripts to the project (under Assets, outside Editor/Packages/StudioDocumentation).
2. Optionally add `/// <summary>...</summary>` above each class for a better summary in the doc.
3. Open **Tools → Studio → Documentation** (if the window is not already open).
4. Click **Rebuild Documentation** in the toolbar. Wait for the operation to finish (a few seconds for small projects).
5. The new scripts appear under **Project Scripts** in the sidebar. Click a script to see its section (class name, summary, namespace, base type, public methods, public fields).
6. If you only want to update the script index without replacing all categories, click **Rebuild Index** instead; then manually add a category or use a custom script to generate sections from the updated scriptIndex.

### Workflow: Refreshing after external edit

1. You (or a teammate) edit **DocumentationData.asset** in the Inspector or via version control (e.g. pull changes that modify the asset).
2. If the documentation window is already open, the in-memory `_data` may be stale. Click **Refresh** in the toolbar (or close and reopen the window).
3. **Refresh** triggers a reload (e.g. LoadOrCreate or LoadAssetAtPath again) so the window shows the latest categories and sections.
4. Save the project after making edits so the asset is written to disk and others can pull the changes.

### Workflow: Using search to audit

1. Open **Tools → Studio → Documentation** and ensure "Project Scripts" (or your categories) are built.
2. Use the **search** box to find all sections that mention a keyword (e.g. "Manager", "State", "Save").
3. Review the result list; click **Open** on each to jump to the section and read the summary and content.
4. Use this to audit which scripts touch a given concept, or to find all sections that need a Warning message (e.g. "do not modify").
5. Optionally switch to the **Dependency Graph** tab to see how scripts are grouped by base type for a high-level audit.

---

## 20. Internal Studio Usage Scenario

- **Onboarding**: New hires open the documentation window, read Part I (game architecture) and Part II (tool usage). They use "Project Scripts" to browse classes and the Safe Modification section to see what not to touch. Sensitive files list highlights core systems. The FAQ and Troubleshooting sections answer common setup and usage questions without requiring a senior to be available.
- **Code auditing**: Auditors use the script index and dependency graph to see what exists and how types are grouped. They use the Safe Modification and Configuration sections to verify that critical paths are documented. The Dependency Graph tab groups scripts by base type (e.g. singleton, MonoBehaviour), making it easy to review inheritance and usage patterns.
- **Knowledge transfer**: When a senior leaves, the documentation (architecture, systems, managers, game flow, and auto-generated script list) serves as a single source of truth. Rebuild Documentation keeps the script list up to date. New owners can extend sections with links to scripts and with Warning/Critical message boxes for critical areas.
- **Security compliance**: SensitiveFile entries and Safe Modification notes document which assets are critical. Teams can extend the tool to add a "Sensitive" tab that lists these assets and their notes for compliance reviews. The data model supports levels (Core, High, Medium, Low) for prioritization.

### Studio Documentation quick reference (Part II)

| Action | How |
|--------|-----|
| Open documentation window | **Tools → Studio → Documentation** |
| Rebuild full doc from code | **Tools → Studio → Documentation → Rebuild Documentation** or toolbar **Rebuild Documentation** |
| Update script index only | Toolbar **Rebuild Index** |
| Search | Type in toolbar search field; results show below; click **Open** to go to section |
| Change tab | Click tab name (Overview, Architecture, …, Dependency Graph) |
| Navigate | Use left sidebar; expand category, click section |
| Edit content | Select **DocumentationData** asset in Project; edit categories/sections in Inspector |
| Add link to script | In section’s **Links**, add element; set **Target** to script/scene/prefab |
| Mark sensitive file | In **DocumentationData**, **Sensitive Files**, add entry with label, asset, level, notes |

### Key paths and constants (Part II)

| What | Value |
|------|--------|
| Data asset path | `Assets/StudioDocumentation/Resources/DocumentationData.asset` (DocumentationDataUtility.AssetPath) |
| Data assembly | `Assets/StudioDocumentation/StudioDocumentation.asmdef` |
| Editor assembly | `Assets/StudioDocumentation/Editor/StudioDocumentation.Editor.asmdef` |
| Default category name (Rebuild) | "Project Scripts" (DocumentationRebuild.ProjectScriptsCategoryName) |
| Menu: Open window | `Tools/Studio/Documentation` |
| Menu: Rebuild | `Tools/Studio/Documentation/Rebuild Documentation` |
| Indexer path filters (skip) | Paths containing `/Editor/`, `/Packages/`, `StudioDocumentation` |

### Common code snippets (Part II)

- **Load or get DocumentationData** (e.g. in an editor script):
  ```csharp
  var data = DocumentationDataUtility.LoadOrCreate();
  if (data == null) return;
  ```
- **Run Rebuild from code**:
  ```csharp
  var data = DocumentationDataUtility.LoadOrCreate();
  if (data != null) DocumentationRebuild.Rebuild(data);
  ```
- **Add a section to a category**:
  ```csharp
  var cat = data.categories.FirstOrDefault(c => c.name == "My Category");
  if (cat != null) {
      cat.sections.Add(new DocumentationSection { title = "New", summary = "Summary", tab = DocumentationTab.Overview, richTextContent = "Body." });
      EditorUtility.SetDirty(data);
      AssetDatabase.SaveAssets();
  }
  ```

### Summary checklists

**For beginners**

- Open **Tools → Studio → Documentation** to view docs.
- Click **Rebuild Documentation** if the sidebar is empty to generate "Project Scripts" from your code.
- Use the **sidebar** to expand categories and click a section to read it.
- Use the **search** box to find sections by keyword; click **Open** on a result to go there.
- Edit content by selecting **DocumentationData.asset** in the Project window and editing in the Inspector; save the project to persist.
- Add **XML summaries** (`/// <summary>...</summary>` above classes) so the generated doc shows better descriptions.
- If something fails, check the **Troubleshooting** table (section 13) and the **FAQ** (section 14).

**For senior engineers**

- **Data** lives in `StudioDocumentation` (non-Editor) assembly; **Editor** code in `StudioDocumentation.Editor` and references the data assembly.
- **ScriptableObject** and all serialized types must stay in the data assembly so asset creation and import succeed.
- **Asset path**: `Assets/StudioDocumentation/Resources/DocumentationData.asset`; creation is deferred with `EditorApplication.delayCall` in DocumentationDataUtility.
- **Rebuild** = clear categories, run ScriptIndexer.BuildIndex(), create "Project Scripts" with one section per ScriptInfo, save. **Rebuild Index** = update scriptIndex only.
- **ScriptIndexer** skips paths containing `/Editor/`, `/Packages/`, or `StudioDocumentation`; uses GetClass() and reflection (DeclaredOnly for methods).
- **Extend**: add enum value for new tab and handle in DrawContent; add category/section in code or Inspector; integrate AI/markdown/code preview by extending Rebuild or the renderer.
- **Namespace**: use `using StudioDocumentation.Editor` where you call ScriptIndexer or DocumentationRebuild to avoid CS0103.

### Related documents and resources (Part II)

- **Part I of this README**: Indekos game architecture, systems, and safe modification guide. Use it to document *what* the game does and *what* not to touch; Part II documents the *tool* that holds that documentation. Cross-reference Part I sections (e.g. Safe Modification, Configuration) when authoring content in DocumentationData.
- **Unity Manual**: [ScriptableObjects](https://docs.unity3d.com/Manual/class-ScriptableObject.html) for asset creation and serialization; [AssetDatabase](https://docs.unity3d.com/ScriptReference/AssetDatabase.html) for FindAssets, CreateAsset, SaveAssets; [EditorWindow](https://docs.unity3d.com/ScriptReference/EditorWindow.html) for dockable windows and OnGUI.
- **Assembly definitions**: [Unity asmdef](https://docs.unity3d.com/Manual/script-compilation-assembly-definition-files.html) for understanding the two-assembly split and why the data assembly must not reference Editor.
- **Editor scripting**: [MenuItem](https://docs.unity3d.com/ScriptReference/MenuItem.html), [InitializeOnLoad](https://docs.unity3d.com/ScriptReference/InitializeOnLoadAttribute.html), [EditorApplication.delayCall](https://docs.unity3d.com/ScriptReference/EditorApplication-delayCall.html) for menu items and deferred execution.
- **Reflection**: [Type.GetMethods](https://docs.microsoft.com/en-us/dotnet/api/system.type.getmethods), [BindingFlags](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.bindingflags) for ScriptIndexer method/field discovery.

### Related Unity concepts (Part II)

- **ScriptableObject**: Asset that lives on disk; serialized as YAML; created with CreateInstance and CreateAsset. Used as the root type for DocumentationData so the doc is a single asset.
- **AssetDatabase**: Editor API for finding, loading, creating, and saving assets. FindAssets returns GUIDs; LoadAssetAtPath loads by path; CreateAsset writes a new asset; SaveAssets flushes dirty assets.
- **EditorWindow**: Base class for dockable editor windows. OnGUI is called every frame; use GUILayout and EditorGUILayout for IMGUI controls.
- **Domain reload**: When scripts recompile, the app domain is recreated. Static constructors run; avoid doing asset creation or heavy work there—use EditorApplication.delayCall to defer.
- **Assembly Definition (asmdef)**: Splits code into assemblies. Editor-only assemblies are excluded from builds; non-Editor assemblies can be referenced by Editor assemblies so that data types live in a "runtime" assembly and Editor code in an "Editor" assembly.

### Document structure summary

- **Part I** (Indekos): ~400 lines. Game overview, architecture, systems, folder structure, classes, game flow, dependencies, configuration, safe modification, glossary. No code changes; documentation only.
- **Part II** (StudioDocumentation): ~1300+ lines. Tool overview, features, architecture, folder layout, data model, script indexing, auto-generation, UI, error handling, extending, security, performance, troubleshooting, FAQ (85), best practices, contribution, versioning, roadmap, example workflow, studio usage. Production-grade reference for both beginners and senior engineers.

### Summary of Part II sections (one-line each)

| § | One-line description |
|---|------------------------|
| 1 | What StudioDocumentation is, who it’s for, comparison with other approaches, quick start. |
| 2 | Twelve key features (menu, window, sidebar, toolbar, tabs, script index, rebuild, sensitive files, foldouts, scroll memory, message boxes, links) with what/why/how. |
| 3 | Two assemblies, why ScriptableObject is in data assembly, asset pipeline, domain reload, delayCall, folder/assembly diagrams, asset lifecycle, data flow (user actions). |
| 4 | Folder and file breakdown; file-by-file responsibility table; allowed references. |
| 5 | DocumentationData, DocumentationCategory, DocumentationSection, Tab, Link, ScriptInfo, SensitiveFile; fields, serialization, Inspector, JSON-like examples, when saved, naming/IDs, YAML. |
| 6 | FindAssets, GetClass, reflection, method filtering, XML parsing, performance, limitations, filtering rules table, indexing pipeline diagram. |
| 7 | When auto-generation runs (and when not), manual rebuild steps, safety checks, error handling, complete Rebuild code flow. |
| 8 | DocumentationRenderer, drawing order, IMGUI vs UI Toolkit, tabs, search, performance. |
| 9 | Why creation can fail, why Editor can’t host ScriptableObject, domain reload, defensive code, try/catch, decision log. |
| 10 | Add categories, tabs, AI/markdown/code preview, search indexing; code examples. |
| 11 | SensitiveFile, why we track, classification, studio usage example. |
| 12 | Reflection/scan cost, caching, lazy load, domain reload, performance table, EditorPrefs/persistence. |
| 13 | Troubleshooting table (symptoms, cause, fix), common pitfalls. |
| 14 | FAQ (85 Q&A). |
| 15 | Best practices, code conventions. |
| 16 | Contribution, testing checklist. |
| 17 | Versioning, migration (legacy path). |
| 18 | Roadmap. |
| 19 | Example workflows (new script, manual section, sensitive file, rebuild after new scripts, refresh, search audit). |
| 20 | Studio usage (onboarding, auditing, knowledge transfer, compliance); quick reference; paths/constants; code snippets; summary checklists; related docs; document structure and history. |

### Document history

- This README is maintained as the single source of truth for the project and the StudioDocumentation tool. No changelog is embedded; for version history use version control (e.g. git log). When making large edits, update the Table of Contents and the Document structure summary so links and line estimates stay accurate.

---

**Documentation only.** This README is for understanding and maintenance. No source or non-documentation files were modified.
