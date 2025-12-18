# System Architecture Diagram

## How Everything Works Together

```
┌─────────────────────────────────────────────────────────────────┐
│                    CROSSWORD SPRINT SYSTEM                       │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────────┐           ┌──────────────────────┐
│   PLAYER CLIENT      │           │   ADMIN CLIENT       │
│   (index.html)       │           │   (admin.html)       │
├──────────────────────┤           ├──────────────────────┤
│ • Login screen       │           │ • Login screen       │
│ • Wait screen        │           │ • Dashboard          │
│ • Game screen        │           │ • Start Game button  │
│ • Puzzle display     │           │ • Stop Game button   │
│ • Score counter      │           │ • Leaderboard        │
│ • Timer              │           │ • Player count       │
│ • Auto-submit puzzle │           │ • Puzzle count       │
└──────────────────────┘           └──────────────────────┘
         │                                   │
         │ (Real-time sync via Firestore)   │
         │                                   │
         └──────────────────┬────────────────┘
                            │
                            ▼
        ┌──────────────────────────────────────────┐
        │        FIREBASE (Backend)                │
        ├──────────────────────────────────────────┤
        │                                          │
        │  ┌────────────────────────────────────┐  │
        │  │  Firestore Database                │  │
        │  ├────────────────────────────────────┤  │
        │  │ Collections:                       │  │
        │  │                                    │  │
        │  │ competitions/                      │  │
        │  │  └─ sprint_001/                    │  │
        │  │      ├─ status: "active"           │  │
        │  │      ├─ puzzleSet: [...]           │  │
        │  │      ├─ admins: ["email@"]         │  │
        │  │      └─ players/                   │  │
        │  │          ├─ user123/               │  │
        │  │          │  ├─ name: "John"        │  │
        │  │          │  ├─ solvedCount: 5      │  │
        │  │          │  └─ lastSolved: time    │  │
        │  │          └─ user456/               │  │
        │  │             ├─ name: "Jane"        │  │
        │  │             ├─ solvedCount: 7      │  │
        │  │             └─ lastSolved: time    │  │
        │  │                                    │  │
        │  └────────────────────────────────────┘  │
        │                                          │
        │  ┌────────────────────────────────────┐  │
        │  │  Authentication                    │  │
        │  │  (Google Sign-In)                  │  │
        │  └────────────────────────────────────┘  │
        │                                          │
        └──────────────────────────────────────────┘
```

---

## Data Flow: Player Solves a Puzzle

```
┌─────────────┐
│   Player    │
│  opens app  │
└──────┬──────┘
       │
       ▼
   ┌───────────────────┐
   │  Load Firebase    │
   │  & auth config    │
   └─────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │  Listen to game  │  ← Checks status every second
    │  state change    │     (waiting/active/stopped)
    │                  │     Fetches puzzleSet array
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │  Admin clicks    │
    │  "START GAME"    │
    └────────┬─────────┘
             │
             ├─► Updates: competitions/sprint_001/status = "active"
             │
             ▼
    ┌──────────────────┐
    │  Player detects  │  ← Real-time listener fires
    │  status changed  │     (via onSnapshot)
    │  to "active"     │
    └────────┬─────────┘
             │
             ├─► Start timer
             ├─► Load puzzle #1
             │
             ▼
    ┌──────────────────┐
    │  Load Exolve     │
    │  puzzle into DOM │
    │  (puzzle text)   │
    └────────┬─────────┘
             │
             ├─► Create Exolve object
             ├─► Destroy old puzzle
             ├─► Render new grid
             │
             ▼
    ┌──────────────────┐
    │  Player types    │
    │  answers into    │
    │  the grid        │
    └────────┬─────────┘
             │
             ├─► Every 500ms, check if complete
             ├─► Check: all cells filled?
             ├─► Check: all answers correct?
             │
             ▼
    ┌──────────────────┐
    │  Puzzle is       │
    │  complete!       │
    └────────┬─────────┘
             │
             ├─► Update Firestore:
             │   players/{userID}/solvedCount++
             │
             ▼
    ┌──────────────────┐
    │  Score updates   │  ← Real-time listener fires
    │  in Firestore    │     (both player and admin see)
    │  & leaderboard   │
    └────────┬─────────┘
             │
             ├─► Player score: 5 → 6
             ├─► Admin leaderboard: John moves to #2
             ├─► Player count: 15 online
             │
             ▼
    ┌──────────────────┐
    │  Load next       │
    │  puzzle (auto)   │
    └────────┬─────────┘
             │
             ├─► currentPuzzleIndex++
             ├─► Load puzzleSet[1]
             ├─► Destroy old puzzle
             │
             ▼
    ┌──────────────────┐
    │  Player sees     │
    │  "Next puzzle    │
    │   loading..."    │
    └────────┬─────────┘
             │
             └─► Back to: Load Exolve puzzle
```

---

## Admin Flow: Start/Stop Game

```
┌─────────────┐
│    Admin    │
│  opens app  │
└──────┬──────┘
       │
       ▼
   ┌─────────────────────────┐
   │  Check if user is in    │
   │  ADMIN_EMAILS list      │
   │  (security check)       │
   └────────┬────────────────┘
            │
            ├─ YES → Show admin dashboard
            │
            ▼
   ┌─────────────────────────┐
   │  Listen to game state   │  ← Real-time updates
   │  & players collection   │
   └────────┬────────────────┘
            │
            ├─► Show current status
            ├─► Show player list
            ├─► Show leaderboard
            │
            ▼
   ┌─────────────────────────┐
   │  Admin clicks           │
   │  "START GAME"           │
   └────────┬────────────────┘
            │
            ├─► Update Firestore:
            │   competitions/sprint_001/status = "active"
            │   competitions/sprint_001/startTime = now
            │
            ▼
   ┌─────────────────────────┐
   │  All connected players  │  ← Real-time listener fires
   │  see puzzle load        │
   │  (onSnapshot fires)     │
   └────────┬────────────────┘
            │
            ├─► Timer starts for all
            ├─► First puzzle loads for all
            ├─► Players can start solving
            │
            ▼
   ┌─────────────────────────┐
   │  Admin watches          │
   │  leaderboard update     │  ← Updates every submission
   │  in real-time           │
   └────────┬────────────────┘
            │
            ├─► Players solving shown live
            ├─► Rankings update automatically
            ├─► Player count shown
            │
            ▼
   ┌─────────────────────────┐
   │  Admin clicks           │
   │  "STOP GAME"            │
   └────────┬────────────────┘
            │
            ├─► Update Firestore:
            │   competitions/sprint_001/status = "stopped"
            │   competitions/sprint_001/endTime = now
            │
            ▼
   ┌─────────────────────────┐
   │  All players see        │  ← Real-time listener fires
   │  final score screen     │
   │  (cannot play more)     │
   └────────┬────────────────┘
            │
            └─► Game is over for everyone
```

---

## Real-Time Sync Details

```
┌──────────────────────────────────────┐
│  What Triggers Real-Time Updates?    │
├──────────────────────────────────────┤
│                                      │
│ 1. Admin changes status              │
│    → All players see it (instant)    │
│                                      │
│ 2. Player solves puzzle               │
│    → Admin leaderboard updates       │
│    → Other players can see new rank  │
│                                      │
│ 3. Player submits score               │
│    → Firestore increments counter    │
│    → onSnapshot listener fires       │
│    → UI updates immediately          │
│                                      │
│ Technology: Firestore onSnapshot()   │
│ (Real-time listeners)                │
│                                      │
└──────────────────────────────────────┘
```

---

## File Structure & Dependencies

```
index.html
├── imports Firebase SDK
├── imports Firestore SDK
├── imports Auth SDK
├── loads exolve-m.js
├── loads exolve-m.css
└── JavaScript logic (inline)
    ├── Authentication
    ├── Game state listener
    ├── Puzzle loader
    ├── Puzzle completion check
    └── Real-time updates

admin.html
├── imports Firebase SDK
├── imports Firestore SDK
├── imports Auth SDK
└── JavaScript logic (inline)
    ├── Admin authentication
    ├── Game controls
    ├── Leaderboard listener
    └── Stats display

exolve-m.js (Puzzle Engine)
├── createExolve() function
├── Puzzle parser
├── Grid renderer
├── Input handling
└── State management

exolve-m.css (Styling)
├── Grid styles
├── Button styles
├── Clue styles
└── Responsive styles

Firestore Database
├── Collections/Documents (JSON)
└── Real-time listeners (onSnapshot)
```

---

## Security Architecture

```
┌─────────────────────────────────┐
│  Firestore Security Rules       │
├─────────────────────────────────┤
│                                 │
│ PUBLIC READ (anyone):           │
│ ├─ competitions/{id}/status    │
│ ├─ competitions/{id}/puzzleSet │
│ ├─ players/{id}/* (leaderboard)│
│                                 │
│ ADMIN UPDATE ONLY:              │
│ ├─ competitions/{id}/status    │
│ └─ competitions/{id}/puzzleSet │
│    (requires email in admins[])│
│                                 │
│ SELF UPDATE ONLY:               │
│ ├─ players/{userId}/solvedCount│
│ └─ players/{userId}/lastSolved │
│    (only your own document)     │
│                                 │
│ NO DELETES:                     │
│ └─ All delete operations denied │
│                                 │
└─────────────────────────────────┘
```

This architecture ensures:
✓ Players can't cheat (can only update own score)
✓ Players can't access admin functions
✓ Admin has full control
✓ Everyone can see the game state
✓ Leaderboard is visible to all
```

---

## Performance Considerations

```
┌──────────────────────────────┐
│  Real-Time Update Frequency  │
├──────────────────────────────┤
│                              │
│ Puzzle completion check:     │ Every 500ms
│ (grid state polling)         │
│                              │
│ Leaderboard updates:         │ Instant (listener)
│ (when score changes)         │
│                              │
│ Timer update:                │ Every 100ms
│ (mm:ss display)              │
│                              │
│ Game status changes:         │ Instant (listener)
│ (waiting→active→stopped)     │
│                              │
│ Firebase Free Tier:          │
│ • 50,000 reads/day ✓         │
│ • 20,000 writes/day ✓        │
│ • 100 concurrent users ✓     │
│                              │
│ For 1-hour competition:      │
│ ~5,000 reads + 500 writes ✓  │
│ (plenty of headroom)         │
│                              │
└──────────────────────────────┘
```

