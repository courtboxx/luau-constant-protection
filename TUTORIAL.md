# Tutorial — How To Prevent Constant Dumping

## Step 1 — Understand The Problem

If you write:

```lua
local secret = "my_webhook_url"
```

That string becomes part of the function’s constant table.

A constant dumper can extract it instantly.

---

## Step 2 — Remove Plain Strings

Never store sensitive values directly.

Bad:
```lua
local url = "https://example.com/script.lua"
```

---

## Step 3 — Convert String To Character Codes

You can convert any string into numeric byte values.

Example conversion of:
```
test
```

Becomes:
```
116 101 115 116
```

You can generate these using:
```lua
for i = 1, #str do
    print(string.byte(str, i))
end
```

---

## Step 4 — Rebuild At Runtime

```lua
local function zyzo_build()
    local t = {116,101,115,116}
    local s = ""
    for i = 1, #t do
        s = s .. string.char(t[i])
    end
    return s
end

local result = zyzo_build()
```

Now the full string does not exist as a constant.

---

## Step 5 — Optional Hardening

You can increase difficulty by:

- Splitting the numeric table into multiple tables
- Offsetting numbers (add/subtract values)
- Shuffling and reordering
- Decoding only when needed

Example with offset:

```lua
local encoded = {200,201,202}

local function zyzo_decode(t)
    local s = ""
    for i = 1, #t do
        s = s .. string.char(t[i] - 100)
    end
    return s
end
```

---

## Step 6 — Know The Limitations

This method:

Prevents:
- Basic constant dumping
- Plain-text extraction

Does NOT prevent:
- Runtime memory inspection
- Function hooking
- Advanced debugging tools

If a client receives the data, it can eventually be extracted.

---

## Final Rule

If something must remain private:
Keep it server-side.
Never give it to the client.
