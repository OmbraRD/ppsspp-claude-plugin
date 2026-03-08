# PPSSPP MCP Server Setup Guide

## Key Configuration Points

PPSSPP includes a built-in MCP server requiring no external processes. The MCP server is available in PPSSPP v1.20.1 and later.

Enable the MCP server in PPSSPP's developer settings. The default port is 27077.

## Communication Protocol

The server operates via Streamable HTTP at the endpoint `POST http://localhost:27077/mcp` using JSON-RPC 2.0 format. The protocol version is `2024-11-05`.

## Server Information

- **Server name:** ppsspp
- **Capabilities:** tools
- **Instructions from server:** "PPSSPP PSP Emulator MCP server. Provides tools for inspecting and controlling the emulated PSP. Memory addresses are in PSP address space (user RAM starts at 0x08800000)."

## PSP Memory Map

| Region | Address Range | Size |
|--------|--------------|------|
| Scratchpad | 0x00010000 | 16 KB |
| VRAM | 0x04000000 - 0x04200000 | 2 MB |
| User RAM | 0x08800000 - 0x0A000000 | 24 MB |

## PSP CPU (Allegrex)

- MIPS32 4K-based, dual-core, 32-bit, little-endian
- Clock: 1-333 MHz (default 222 MHz)
- COP0 (system control), COP1 (FPU), COP2 (VFPU)
- 16 KB I-cache + 16 KB D-cache per core
- Supports selected MIPS IV instructions: ext, ins, wsbw, seb, seh, rotr, clz, clo
- Custom instructions: halt (0x70000000), mfic (0x70000024), mtic (0x70000026)
- No MMU/TLB

## Common Issues

- Ensure the MCP server is enabled in PPSSPP settings before starting
- Confirm port 27077 is not occupied by another service
- The server only binds to localhost -- remote connections are not supported
- Most tools need a game to be running -- load a game first if tools return errors
- Memory read/write max size is 65536 bytes per call
