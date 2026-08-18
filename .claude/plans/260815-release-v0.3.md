# Plan: env-sdl release v0.3 (atmos v0.8)

Driven from atmos master plan `260815-release-v0.8.md`.
No breaking core change (loop_on removal deferred to v0.9).
0 code commits since v0.2 -- vN bump pairs with atmos 0.8.

# Steps

1. [x] Migrate: NONE needed (no any/all sites; audit clean)
2. [x] README: v0.3 list + stable link; dep `atmos v0.8`
3. [x] Phase 1 tests (local LUA_PATH): hello exit-0;
       across + click-drag-cancel run clean (6s smoke)
4. [x] Rockspec `atmos-env-sdl-0.3-1` (branch=v0.3,
       dep `atmos ~> 0.8`); dev-1 unchanged (unversioned dep)
5. [ ] Make rockspec (sudo -- user):
       `sudo luarocks --lua-version=5.4 make atmos-env-sdl-0.3-1.rockspec`
6. [ ] Phase 2 tests (global): hello / across / click-drag-cancel
7. [ ] Commit, push `v0.3`; ff `main` (user)
8. [x] Branch `v0.3` created (not pushed)
