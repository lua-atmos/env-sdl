# Plan: Release env-sdl v0.2 (atmos v0.7)

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

- [ ] Move `open` body to module top-level
- [ ] Rename `close` -> `quit`
- [ ] Clock emits single-arg microseconds
- [ ] Verify `atmos.env(M)` still registers (`step`/`quit`/`mode`)

#### 1.1 INVESTIGATE: custom event matching (`__atmos`)

v0.7 `await` is single-arg, but examples await with two args
(type + predicate), and the env's matcher reads `awt[1]`/`awt[2]`.

Findings:
- Core calls `__atmos` on BOTH sides:
    - pattern metatable: `__atmos(awt, emt)` (run.lua:557)
    - event metatable:   `__atmos(emt, awt)` (run.lua:599)
- env currently attaches `meta` to the EMITTED event
  (`emit(setmetatable(e, meta))`), so core calls
  `__atmos(event, pattern)`.
- But the env fn `function (awt, e)` is written as if it gets
  `(pattern, event)` -> mismatch under v0.7.

Decide and implement:
- [ ] Make await patterns single-arg tables carrying
      `[1]=type`, `[2]=name|predicate`
- [ ] Put `__atmos` on the PATTERN (so `__atmos(pat, evt)`
      fires via run.lua:557), or fix the event-side arg order
- [ ] Provide a helper/ctor so examples build patterns cleanly

### 2. Migrate examples (`exs/`)

| file | change |
|------|--------|
| `exs/hello.lua` | `clock{s=5}` -> `5*_s_`; `clock{ms=500}` -> `500*_ms_` |
| `exs/across.lua` | raw SDL loop + `SDL.delay`; verify, likely no change |
| `exs/click-drag-cancel.lua` | single-arg awaits per 1.1: `await(MouseButtonDown, pred)`, `await(KeyDown, 'Escape')`, `await(MouseMotion)`, etc.; `every('sdl.draw', ...)` stays (string tag ok) |

- [ ] `exs/hello.lua`
- [ ] `exs/across.lua`
- [ ] `exs/click-drag-cancel.lua`

### 3. README.md

- [ ] atmos version `v0.6` -> `v0.7`
- [ ] env version `v0.1` -> `v0.2`
- [ ] Update any inline example syntax (clock/await/emit)

### 4. Rockspec

New `atmos-env-sdl-0.2-1.rockspec` from `0.1-1`:

| field | v0.1 | v0.2 |
|-------|------|------|
| `version` | `0.1-1` | `0.2-1` |
| `source.branch` | `v0.1` | `v0.2` |
| `dependencies` atmos | `atmos >= 0.6` | `atmos ~> 0.7` |

- [ ] Create rockspec
- [ ] Keep `lua-sdl2` dep; verify module list

### 5. Phase 1 tests (local, `LUA_PATH` trick)

- [ ] `exs/hello.lua`
- [ ] `exs/across.lua`
- [ ] `exs/click-drag-cancel.lua`

### 6. Phase 2 tests (global, `luarocks make`)

- [ ] `luarocks make atmos-env-sdl-0.2-1.rockspec`
- [ ] `exs/hello.lua`
- [ ] `exs/across.lua`
- [ ] `exs/click-drag-cancel.lua`

### 7. Commit, push, branch

- [ ] Commit, push main
- [ ] Create/update branch `v0.2`, push

### 8. Dependent apps (separate repos)

Each needs the same await/clock migration + version bumps:

- [ ] `sdl-birds` (`birds-11.lua`)
- [ ] `sdl-rocks` (`main.lua`)
- [ ] `sdl-pingus` (`main.lua`)
