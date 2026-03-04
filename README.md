# LuaU Constant Dump Prevention

## What Is Constant Dumping?

In Lua and LuaU, every compiled function contains a **constant table**.

See [TUTORIAL.md](TUTORIAL.md) for a complete step-by-step guide.


Constants include:
- Strings
- Numbers
- Booleans
- Some function references

When code is inspected using debug-enabled environments, those constants can be extracted directly from memory.

If you write:

```lua
local webhook = "https://example.com/webhook"
```

That full string exists inside the function’s constant table and can be dumped instantly.

---

## Why This Matters (Roblox / LuaU)

In Roblox (LuaU):

- Client memory is readable.
- Constants inside LocalScripts can be extracted.
- Obfuscation does not remove constants.
- Any plain string secret is exposed.

If you give the client a secret, assume it can be read.

---

## What This Repository Shows

This repo demonstrates:

- How constants are exposed
- How to avoid storing sensitive values directly
- Runtime reconstruction methods
- Limitations of protection

---

# Example 1 — Vulnerable Script

```lua
local url = "https://raw.githubusercontent.com/user/repo/main/script.lua"
loadstring(game:HttpGet(url))()
```

The entire URL is stored as a constant.

A constant dumper will extract it immediately.

---

# Example 2 — Runtime Reconstruction (Numeric Encoding)

```lua
local function zyzo_build()
    local t = {
        104,116,116,112,115,58,47,47,
        114,97,119,46,103,105,116,104,117,98,117,115,101,114,99,111,110,116,101,110,116,46,99,111,109,47,
        117,115,101,114,47,
        114,101,112,111,47,
        109,97,105,110,47,
        115,99,114,105,112,116,46,108,117,97
    }

    local s = ""
    for i = 1, #t do
        s = s .. string.char(t[i])
    end
    return s
end

local zyzo_url = zyzo_build()
loadstring(game:HttpGet(zyzo_url))()
```

Here:

- The full URL does not exist as a readable string constant.
- Only numeric values exist in the constant table.
- The string is reconstructed at runtime.

This prevents basic constant dumping.

---

# Example 3 — Split Table Reconstruction

```lua
local a = {104,116,116,112,115,58,47,47}
local b = {114,97,119,46,103,105,116,104,117,98}
local c = {46,99,111,109}

local function zyzo_merge(...)
    local out = ""
    for _, tbl in ipairs({...}) do
        for i = 1, #tbl do
            out = out .. string.char(tbl[i])
        end
    end
    return out
end

local zyzo_url = zyzo_merge(a,b,c)
```

Now:

- The data is fragmented.
- A simple constant dumper cannot reconstruct the full string directly.
- Reverse engineering effort increases.

---

# Example 4 — Basic Obfuscation Layer

```lua
local encoded = {209,221,221,217}
local function zyzo_decode(t)
    local s = ""
    for i = 1, #t do
        s = s .. string.char(t[i] - 100)
    end
    return s
end

local result = zyzo_decode(encoded)
```

You can:
- Offset numbers
- XOR numbers
- Shuffle order
- Decode conditionally

This further reduces readability.

---

# What This Prevents

- Basic constant dumping
- Plain-text string extraction
- Simple automated scrapers

---

# What This Does NOT Prevent

- Runtime memory inspection
- Hooking `HttpGet`
- VM-level dumping
- Advanced reverse engineering

If a value is reconstructed in memory, it can still be captured.

---

# Real Security Rule

Do not give the client secrets.

If something must remain private:
- Keep it server-side.
- Store it in ServerScriptService.
- Never expose it to LocalScripts.

---

# Summary

Constant dumpers extract static data.

The solution is:
- Avoid static secrets.
