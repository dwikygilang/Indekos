# Indekos

**A narrative life-simulation game set around student boarding-house life - day-based progression, quest-driven story, and minigames.**

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

- **Day-based progression** - Time advances by days; each day has quests, study, and story.
- **Multi-location exploration** - Campus, kos, gang kos, minimarket, kelas, guitar room, warteg, etc., with additive scene loading.
- **Quest-driven flow** - Main quests define objectives, trigger story/dialogue, and can spawn triggers, NPCs, and components (e.g. PC parts, cameras).
- **Study system** - In-class dialogue with multiple-choice answers (benar/salah) affecting quests.
- **Story vs chit-chat** - Story dialogue (tied to quests) vs optional NPC chit-chat.
- **Minigames** - Guitar (practice + final), Mingsut, Endless Run, Rakit PC, Warteg, Tarot, HP UI, Find Part PC, Freelance (QTE).
- **Save/Load** - Encrypted JSON saves (position, level, quest/story/study indices, time, money, tarot, inventory, dialogue log, map list, etc.).

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

**Documentation only.** This README is for understanding and maintenance. No source or non-documentation files were modified.
