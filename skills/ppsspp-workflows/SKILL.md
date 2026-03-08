# PPSSPP Reverse Engineering Workflows

## Workflow A: Find What Code Writes to a RAM Address

Use this when you know the address of a variable and want to find the code responsible for modifying it.

1. `pause()` -- pause emulation
2. `set_breakpoint(address=0x08XXXXXX)` -- set an execute breakpoint near suspicious code, or use memory search to find the address first
3. `resume()` -- resume and trigger the write in-game
4. Wait for breakpoint to hit (emulator pauses automatically)
5. `read_registers()` -- check the `pc` register to find the current instruction
6. `disassemble(address=pc-16, count=16)` -- view surrounding code for context
7. `lookup_symbol(address=pc)` -- identify the function name if available

**Tip:** Since PPSSPP only has execute breakpoints (not write watchpoints), use `search_memory` to find the current value at the target address, then use static analysis or disassembly to find store instructions that reference that address. Set execute breakpoints on those store instructions.

## Workflow B: Find a Game Variable by Value (Cheat Search)

Use this when you can see a value on screen (HP, money, items) but do not know its RAM address.

1. `pause()` at a known state (e.g. when HP is displayed as 100)
2. Convert the value to hex little-endian bytes. For a 32-bit value 100: `"64000000"`
3. `search_memory(hex="64000000")` -- find all locations containing that value
4. Note the candidate addresses
5. `resume()` -- change the value in-game (e.g. take damage so HP becomes 85)
6. `pause()`
7. `search_memory(hex="55000000")` -- search for the new value (85 = 0x55)
8. Compare the two result sets -- addresses present in both searches are strong candidates
9. Verify: `read_memory(address=candidate, size=4)` -- confirm the value matches
10. Test: `write_memory(address=candidate, hex="E7030000")` -- write 999 (0x3E7) to confirm it's the right variable

**Tip:** For 16-bit values (halfwords), search with 2-byte patterns. For values that might be stored as floats, convert to IEEE 754 hex first.

## Workflow C: Compare RAM Before and After an Action

Use this to understand what a game action modifies in memory.

1. `pause()` before performing the action
2. `read_memory(address=0x08800000, size=65536)` -- save a section of RAM as baseline (repeat for multiple 64KB chunks if needed)
3. `resume()` -- perform the action in-game
4. `pause()` immediately after
5. `read_memory(address=0x08800000, size=65536)` -- read the same region again
6. Compare the two hex dumps to find changed bytes

**Tip:** Focus on specific memory regions rather than scanning all 24 MB. Use `search_memory` to narrow down areas of interest first.

## Workflow D: Verify ASM Patches at Runtime

Use this after patching code to confirm patches were applied correctly in RAM.

1. Load the game in PPSSPP
2. `pause()` -- pause at a point after the patched code is loaded
3. `read_memory(address=patch_address, size=N)` -- read bytes at the patched location
4. Compare the hex bytes with the expected patch bytes
5. `disassemble(address=patch_address, count=M)` -- verify instruction mnemonics are correct
6. Alternatively, use `assemble()` to apply patches at runtime:
   - `assemble(address=0x08900000, instruction="li a0, 999")` -- write a new instruction directly

**Tip:** For overlays loaded from disc at runtime, advance to the game state that triggers the overlay load first. Use `list_threads()` to see if the relevant module's thread is active.

## Workflow E: Inspect Thread State

Use this to debug multi-threaded games or understand the PSP kernel's thread scheduling.

1. `pause()` -- pause emulation
2. `list_threads()` -- see all kernel threads, their status, and PCs
3. `disassemble(address=thread_pc, count=16)` -- examine what a specific thread is executing
4. `lookup_symbol(address=thread_pc)` -- identify the function the thread is in
5. `read_registers()` -- inspect the current thread's register state

**Tip:** PSP games typically have dedicated threads for main game logic, rendering (GE callbacks), audio decoding, and file I/O. Identifying which thread handles what is key to effective debugging.

## Workflow F: Symbol-Guided Exploration

Use this when starting reverse engineering on an unfamiliar game.

1. Load the game and `pause()` once it reaches a stable state
2. `lookup_symbol(name="sceDisplay")` -- look up known PSP HLE function names to find relevant code
3. `lookup_symbol(name="main")` or other common entry points
4. `disassemble(address=found_address, count=32)` -- explore the code around known symbols
5. Follow function calls by disassembling the target addresses of `jal` instructions
6. Use `set_breakpoint()` on interesting functions to observe when they're called and with what arguments

**Common PSP HLE modules to look up:**
- `sceDisplay` -- display/framebuffer functions
- `sceCtrl` -- controller/input functions
- `sceAudio` -- audio output
- `sceIo` -- file I/O
- `sceKernel` -- kernel/thread management
- `sceGe` -- graphics engine (GE) commands

## Workflow G: Live Memory Patching for Testing

Use this for quick experimentation without rebuilding or reassembling.

1. `pause()` -- pause the game
2. Find the target address (using Workflows B or A)
3. `read_memory(address=target, size=4)` -- note the original value
4. `write_memory(address=target, hex="NEWVALUE")` -- write new data
5. `resume()` -- observe the effect in-game
6. `take_screenshot()` -- capture the result

For code patches:
1. `disassemble(address=target, count=4)` -- see the original instructions
2. `assemble(address=target, instruction="nop")` -- NOP out an instruction
3. Or `assemble(address=target, instruction="li v0, 1")` -- change a return value
4. `resume()` -- test the patch

**Tip:** Always note original values before patching so you can restore them. Use `write_memory` to restore original bytes if `assemble` doesn't support the reverse operation.
