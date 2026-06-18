# Plan: Re-release env-sdl v0.2 (atmos 0.7-2)

## RESUME HERE (state @ 2026-06-18) -- handoff to another machine

DONE & PUSHED (nothing to carry):
- env-sdl: migrated + tested + pushed. `v0.2`==`main`==origin
  @ a31c8d1. Rock stays `0.2-1` (0.2-2 bump SKIPPED).
- sdl-birds: migrated + tested + COMMITTED + PUSHED.
  `v0.5`==`main`==origin @ edc8203 ("v0.5 : task, xtask, loop_on").

>>> CRITICAL -- UNCOMMITTED, LOCAL TO THIS MACHINE ONLY <<<
sdl-rocks & sdl-pingus are migrated + TESTED but their edits are
WORKING-TREE ONLY (not committed). They will NOT be on another
machine. Options: (a) commit+push HERE before switching, or
(b) redo the migration there (rules below + RULE 9 + pitfalls).

NEXT STEPS (explicit):
1. [ ] sdl-rocks (on branch `v0.5`; modified: battle/main/ts.lua):
       commit -> push v0.5 -> ff `master` -> push master.
       Base origin @ 719f2e1. Use msg like sdl-birds:
       "v0.5 : task, xtask, loop_on".
2. [ ] sdl-pingus (on branch `v0.5`; modified: level/menu/pingu.lua):
       commit -> push v0.5 -> ff `main` -> push main.
       Base origin @ 1352516. Same commit msg.
3. [ ] (optional) spot-rerun an example per repo after checkout
       on the other machine to confirm (luac -p + run).
4. [ ] Back in atmos plan (06-08-release-v0.7.md): mark sdl-rocks/
       sdl-pingus DONE under §4 apps; then continue atmos §2 docs.

If redoing on another machine, apply per file:
`every (`/`every(` -> `loop_on`; `task()` -> `xtask()`;
`spawn(function` -> `do_spawn(function`; NAMED protos: wrap the
DEFINITION `Name = task(function..end)` (see RULE 9). Then
`luac -p *.lua` and run an example.

DECISION (2026-06-18): SKIP the rock rev bump. Rockspec content is
byte-identical to `0.2-1` (same branch `v0.2`, `atmos ~> 0.7`,
module list). Rocks branch-track `v0.2`, so pushing the fix serves
it under `0.2-1` automatically -> no `0.2-2`, no re-upload, no
`dev-2`. Steps 4/5/7 dropped.

PRIOR CUT (frozen, see bottom): env-sdl v0.2 / rock `0.2-1`
was released for atmos 0.7-1. That work stands. Since then
atmos v0.7 grew BREAKING changes (shipping as 0.7-2):
`every`->`loop_on`, `task()` me-accessor -> `xtask()`,
`spawn(fn)` -> `do_spawn`. This re-cuts env-sdl on the new core.

Breaking sites (scan @ 2026-06-18):
- `exs/hello.lua:6` `every(500*_ms_, ...)`
- `exs/click-drag-cancel.lua:21` `spawn(function ...)`
- `exs/click-drag-cancel.lua:22` `every('sdl.draw', ...)`
- `exs/click-drag-cancel.lua:49` `every({tag='sdl', ...}, ...)`
- no `task()` accessor

Mechanical migration:
- `every(`            -> `loop_on(`  (ALSO spaced `every (`!)
- `spawn(function...` -> `do_spawn(function...` (self-contained)
                      else `spawn(task(function...))`
- `task()` accessor  -> `xtask()`

NEW RULE 9 (found 2026-06-18 via runtime err in downstream):
0.7-2 `spawn`/`spawn_in` require a TASK PROTOTYPE, not a bare
function. Wrap the DEFINITION once (not each call site):
- `function Bird(a)...end`  -> `local Bird = task(function(a)...end)`
- global / module / `Tbl.Field` protos: keep scope, drop `local`
  (`Bird = task(function...end)`), keep any `return Bird`.
- `do_spawn(fn)` is unaffected (wraps internally).
Spawn sites stay bare: `spawn(Bird,..)`, `spawn_in(pool,Bird,..)`.
Blind-sed pitfall: `\bevery\(` / `\bspawn\(` MISS spaced calls
`every (` / `spawn (`; use `\s*` before `\(`. Definition-wrap is
multi-line (match the closing `end` -> `end)`); verify `luac -p`.

Rocks branch-track `v0.2`, so pushing the fix to `v0.2` already
serves it under `0.2-1`; a new rock rev `0.2-2` (+ `dev-2`,
replaces `dev-1`) is only to re-publish. Mirror atmos `0.7-2`.

## Steps (this re-cut)

1. [x] Migrate the 4 sites above
2. [x] Grep clean: no `every(` / `task()` / bare `spawn(function`
3. [x] Test local (LUA_PATH): hello, across, click-drag-cancel
4. [~] SKIP rock bump (identical content; branch-tracked) -- DROPPED
5. [~] SKIP global luarocks make / upload -- DROPPED
6. [x] Commit, push `v0.2`, ff `main`, sync (39ee15e; both @ a31c8d1)
7. [~] SKIP `luarocks upload` (no new rev) -- DROPPED
8. [~] Downstream apps MIGRATED + TESTED (all 3 repos run).
       - sdl-birds: COMMITTED + PUSHED (done).
       - sdl-rocks / sdl-pingus: UNCOMMITTED (local only) --
         see "NEXT STEPS" at top for commit/push handoff.

## Downstream apps (no own plan -- handle here)

Apps hard-break on 0.7-2 too. Same mechanical renames
(`every`->`loop_on`, `task()`->`xtask()`, `spawn(fn)`->`do_spawn`);
git-only, push branch (no rock). Test against the new env rock:
All rules applied (loop_on incl spaced, xtask, do_spawn,
DEFINITION-wrap protos in task()). All files pass `luac -p`.
EDITED, not committed/tested:
- [x] sdl-birds  (on `main`): ALL `birds-01..11.lua`
      -- 51 loop_on, 16 xtask; `local Bird = task(...)` x11.
- [x] sdl-rocks  (on `master`): `ts/main/battle.lua`
      -- 15 loop_on, 7 xtask, 5 do_spawn; protos wrapped:
      Move_T/Meteor/Shot/Ship (ts), Battle (battle).
- [x] sdl-pingus (on `main`): `pingu/menu/level/main.lua`
      -- 6 loop_on, 2 do_spawn; protos wrapped: Sprite/Pingu/
      Faller/Walker (pingu), Menu.Button/Menu.Main (menu),
      Level (level).

TESTED OK (2026-06-18, all 3 repos run). PENDING: commit + push
each repo (your git; e.g. birds-11, rocks/main, pingus/main run).

--------------------------------------------------------------

## PRIOR CUT (frozen -- atmos 0.7-1 era, for reference)

# Plan: Release env-sdl v0.2 (atmos v0.7)

## STATUS (@ 2026-06-09): COMPLETE — only luarocks upload left

Migration v0.6->v0.7 / v0.1->v0.2 done across all repos.
All code committed, pushed, tested; every `main`/`master`
fast-forwarded to its release branch and synced to origin.

| repo       | release | main==vX | tested |
|------------|---------|----------|--------|
| atmos      | v0.7    | ✅       | ✅     |
| env-sdl    | v0.2    | ✅       | ✅     |
| sdl-birds  | v0.5    | ✅ (11 exs) | ✅  |
| sdl-pingus | v0.5    | ✅       | ✅     |
| sdl-rocks  | v0.5*   | ✅ (*master) | ✅ |

atmos also gained the `'clock'` await primitive + `_us_.._day_`
constants (run.lua / init.lua), installed locally.

### ONLY REMAINING: publish rockspecs to luarocks.org
- [ ] `atmos-env-sdl-0.2-1.rockspec` -> `luarocks upload`
- [ ] atmos: rev-bump to `0.7-2` if `0.7-1` already published,
      then `luarocks upload` (source.branch=v0.7 is pushed)

### Key v0.7 migration findings (see §8 rules)
- clock: bare-number us; timers `N*_s_`; delta via `'clock'`
  primitive (`every('clock', fn(us))`, rescale us->ms)
- events: table patterns `{tag='sdl', type=, name=}`;
  predicate via `{tag='until', <pat>, pred}`; `__atmos` removed
- emit single-arg: toggle driven by `emit{tag='Show', false}`
- tasks-pool await needs `{tag='tasks', mode='any', tasks=}`

## Context

`env-sdl` v0.1 targets atmos v0.6. atmos v0.7 introduces breaking
changes that require an env update and a new `v0.2` release:

- Env API: `open`+`close` -> main body + `quit`
    - `loop` no longer calls `open`; `stop` calls `env.quit`
- Events: `emit`/`await` are single-arg only
- Clock: a bare number in microseconds (no `clock{...}` table,
  no `'clock'` tag)
    - constants `_us_ _ms_ _s_ _min_ _h_ _day_`
- Matching: custom types via `__atmos(pat, evt)` metamethod
  on the await pattern (core: run.lua:556-557)

Reference impl: atmos built-in clock `atmos/env/clock/init.lua`
emits `emit(now - old)` (plain microseconds) and registers an
env table with only `step` (+ optional `quit`, `mode`).

This plan uses release branches (not tags) for versioning.

## Steps

### 1. Migrate `init.lua`

| place | v0.6 | v0.7 |
|-------|------|------|
| `M.open()` (L17) | init SDL/IMG/TTF/MIX in `open` | run as module main body (top-level at `require`); `loop` no longer calls `open` |
| `M.close()` (L143) | `function M.close()` | rename -> `function M.quit()` |
| clock fast (L113) | `emit('clock', dt, M.now)` | `emit(dt * 1000)` (SDL ticks ms -> us); keep `M.now = cur` |
| clock mpf (L119) | `emit('clock', M.mpf, M.now)` | `emit(M.mpf * 1000)` |

- [x] Move `open` body to module top-level
- [x] Rename `close` -> `quit`
- [x] Clock emits single-arg microseconds (`emit(dt*1000)`)
- [ ] Verify `atmos.env(M)` still registers (`step`/`quit`/`mode`)

#### 1.1 DECISION: drop `__atmos`, use built-in table matching

v0.7 `await` is single-arg. Instead of a custom `__atmos`
matcher, rely on core combinators (run.lua):

- Table match (L612-625): `await{ tag, k=v, ... }` accepts an
  event table where every pattern field satisfies `M.is`.
- `until`/`while` (L514-538): `await{ tag='until', <pat>,
  <pred>... }` re-awaits `<pat>` until all predicates hold
  (returns event); `while` until any fails.

Env change: emit events as plain tables with a `tag` field
(no metatable, no `meta`/`__atmos`):
- `M.step` (L137): `e.tag='sdl'; emit(e)` (drop
  `setmetatable(e, meta)`)
- delete `meta`/`__atmos` block (L32-50)

Await forms (used by examples):

| filter | v0.7 form |
|--------|-----------|
| type only | `await{ tag='sdl', type=MouseMotion }` |
| field eq | `await{ tag='sdl', type=KeyDown, name='Escape' }` |
| predicate | `await{ tag='until', {tag='sdl', type=...}, pred }` |

- [x] `M.step`: `e.tag='sdl'; emit(e)`
- [x] Delete `meta`/`__atmos` (L32-50)

### 2. Migrate examples (`exs/`)

| file | change |
|------|--------|
| `exs/hello.lua` | `clock{s=5}` -> `5*_s_`; `clock{ms=500}` -> `500*_ms_` |
| `exs/across.lua` | raw SDL loop + `SDL.delay`; verify, likely no change |
| `exs/click-drag-cancel.lua` | table patterns per 1.1: click -> `await{tag='until', {tag='sdl', type=MouseButtonDown}, pred}`; `await{tag='sdl', type=KeyDown, name='Escape'}`; `await{tag='sdl', type=MouseMotion}`; `every('sdl.draw', ...)` stays (string tag ok) |

- [x] `exs/hello.lua`
- [x] `exs/across.lua` (no change: raw SDL loop)
- [x] `exs/click-drag-cancel.lua`

### 3. README.md

- [x] Events section rewritten for v0.7 (clock us, table
      patterns, drop stale `'sdl.step (ms)'`)
- n/a no literal `v0.6`/`v0.1` strings in README (version
      lives in rockspec)

### 4. Rockspec

New `atmos-env-sdl-0.2-1.rockspec` from `0.1-1`:

| field | v0.1 | v0.2 |
|-------|------|------|
| `version` | `0.1-1` | `0.2-1` |
| `source.branch` | `v0.1` | `v0.2` |
| `dependencies` atmos | `atmos >= 0.6` | `atmos ~> 0.7` |

- [x] Create rockspec (`atmos-env-sdl-0.2-1.rockspec`)
- [x] Keep `lua-sdl2` dep; module list unchanged

### 5. Phase 1 tests (local, `LUA_PATH` trick)

Done via `sudo luarocks make atmos-env-sdl-dev-1.rockspec`
(installs working copy over stale rock). All pass.

- [x] `exs/hello.lua`
- [x] `exs/across.lua`
- [x] `exs/click-drag-cancel.lua`

### 6. Phase 2 tests (global, `luarocks make`)

- [x] `luarocks make atmos-env-sdl-0.2-1.rockspec`
- [x] `exs/hello.lua`
- [x] `exs/across.lua`
- [x] `exs/click-drag-cancel.lua`

### 7. Commit, push, branch

- [x] Commit, push
- [x] Branch `v0.2` pushed

### 8. Dependent apps (separate repos)

Apply the SAME transformations as §1.1/§2 to each app's
source (these are OUTSIDE this worktree — edit on their repos):

Transformation rules (per file):
1. clock timers: `clock{s=N}` -> `N*_s_`;
   `clock{ms=N}` -> `N*_ms_`
   (constants `_us_ _ms_ _s_ _min_ _h_ _day_`)
1b. clock DELTA (frame physics): `'clock'` is now a core
   primitive (atmos run.lua:607) that wakes on any bare-number
   emit and returns the delta in us. Rescale us->ms:
       every('clock', function (_,ms) BODY end)   -- v0.6 (ms)
       every('clock', function (us) local ms=us/1000; BODY end)  -- v0.7 (us)
2. type-only await: `await(SDL.event.X)` ->
   `await{ tag='sdl', type=SDL.event.X }`
3. field await: `await(SDL.event.KeyDown, 'Esc')` ->
   `await{ tag='sdl', type=SDL.event.KeyDown, name='Esc' }`
   (button -> `but='...'`)
4. predicate await: `await(SDL.event.X, pred)` ->
   `await{ tag='until', {tag='sdl', type=SDL.event.X}, pred }`
5. `every(SDL.event.X, handler)` ->
   `every({tag='sdl', type=SDL.event.X}, handler)`
6. toggle: `emit` is single-arg in v0.7. Drive `toggle(E,..)`
   with table emits, not 2-arg:
       emit('Show', false) -> emit{tag='Show', false}
       emit('Show', true)  -> emit{tag='Show', true}
7. tasks-pool await: bare `await(pool)`/`watching(pool,..)`
   no longer fires in v0.7 (pool has no tag/mode). Use:
       watching({tag='tasks', mode='any', tasks=pool}, body)
   returns `(deadTask.ret, deadTask, pool)` -> grab 2nd value:
       local _, dead = watching({tag='tasks',mode='any',...}, ..)
8. their rockspec dep: bump to `atmos-env-sdl ~> 0.2` and
   `atmos ~> 0.7`

Apps (all tested + pushed; main/master==v0.5; READMEs ->
0.7/0.2/v0.5):
- [x] `sdl-birds` — all 11 (`birds-01..11.lua`), v0.5;
      also fixed branch mess (restored v0.3 -> 377f590,
      v0.2/v0.4 untouched)
- [x] `sdl-rocks` (`main.lua`, `battle.lua`, `ts.lua`), v0.5;
      incl. tasks-pool restart fix (rule 7)
- [x] `sdl-pingus` (`main.lua`, `menu.lua`, `level.lua`,
      `pingu.lua`), v0.5
- n/a apps have no rockspecs (tutorials, not packages)
