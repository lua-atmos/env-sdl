# atmos-env-sdl

An [Atmos][atmos] environment for [SDL][sdl] based on [lua-sdl2][lua-sdl2].

[atmos]:    https://github.com/lua-atmos/atmos/
[sdl]:      https://www.libsdl.org/
[lua-sdl2]: https://github.com/Tangent128/luasdl2/

[
    [`v0.2`](https://github.com/lua-atmos/env-sdl/tree/v0.2)  |
    [`v0.1`](https://github.com/lua-atmos/env-sdl/tree/v0.1)
]

Stable branch is [`v0.2`](https://github.com/lua-atmos/env-sdl/tree/v0.2).

# Install

```
sudo luarocks --lua-version=5.4 install atmos-env-sdl
```

Dependencies: `lua-sdl2`, `atmos v0.7`

# Run

```
lua5.4 <lua-path>/atmos/env/sdl/exs/click-drag-cancel.lua
```

Requires font [DejaVuSans.ttf][deja] in the current directory.

[deja]: https://github.com/lua-atmos/env-sdl/blob/main/DejaVuSans.ttf

# Events

- clock
- `'sdl.draw'`
- SDL events: table patterns tagged `'sdl'`
    - `await{ tag='sdl', type=SDL.event.KeyDown, name='Escape' }`
    - filter: `await{ tag='until', {tag='sdl', type=...}, pred }`

# Source

Assumes this directory structure:

```
.
├── atmos/
├── env-sdl/
└── f-streams/
```

```bash
LUA_PATH="../f-streams/?/init.lua;../atmos/?.lua;../atmos/?/init.lua;;" lua5.4 exs/across.lua
```
