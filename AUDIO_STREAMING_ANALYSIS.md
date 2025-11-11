# Super Mario Eclipse Audio Streaming Issue Analysis

## Issue Summary
Super Mario Eclipse has music going silent in Nintendont on Wii and Wii U. The issue is related to audio streaming because turning that off on Swiss on GameCube causes the exact same issue.

## Root Cause Analysis

### Current Audio Streaming Implementation

1. **Audio Streaming Control**: 
   - Defined in `kernel/global.h` with `#define AUDIOSTREAM 1`
   - When enabled, patches `__DSPHandler` functions to handle audio streaming
   - When disabled, patches `DVDLowAudioStream` functions with NULL implementations

2. **How Audio Streaming Works**:
   - Audio streaming is handled in `kernel/Stream.c` via `StreamStartStream()` and `StreamEndStream()`
   - The DSP handler (`kernel/asm/__DSPHandler.S`) processes audio streams
   - Games request audio streaming via DI commands (0xE1, 0xE2) in `kernel/DI.c`

3. **Game-Specific Handling**:
   - Some games like Majora's Mask have special audio streaming patches
   - `DSPHandlerNeeded` flag can be set to 0 to skip DSP handler patching for specific games
   - Games are identified by `TITLE_ID` (GAME_ID >> 8)

### The Problem

The issue description states:
- Music goes silent in Nintendont
- Turning off audio streaming on Swiss causes the same issue
- **This indicates the game NEEDS audio streaming to work properly**

This suggests that:
1. Super Mario Eclipse requires audio streaming for its music
2. The current audio streaming implementation may not be compatible with how this game uses it
3. There might be a timing or synchronization issue

## Solution Approach

### Option 1: Disable DSP Handler Patching (Similar to Majora's Mask)
If the game has its own audio streaming implementation that conflicts with Nintendont's:
- Set `DSPHandlerNeeded = 0` for Super Mario Eclipse
- This allows the game to use its native audio streaming

### Option 2: Add Special Audio Streaming Handling
If the game needs special handling:
- Create a custom audio streaming patch similar to Majora's Mask
- This would require analyzing the game's audio streaming implementation

### Option 3: Fix Audio Streaming Compatibility
If there's a general compatibility issue:
- Investigate the audio streaming timing/synchronization
- Check if the game uses audio streaming differently than expected

## Implementation Steps

1. **Identify the Game**:
   - Find the TITLE_ID for Super Mario Eclipse
   - This can be done by checking the game's header or using debug output

2. **Apply the Fix**:
   - Add game detection in `Patch.c` around line 1475-1627 (where Majora's Mask patches are)
   - Set `DSPHandlerNeeded = 0` for Super Mario Eclipse
   - Or add special audio streaming handling if needed

3. **Test**:
   - Verify music plays correctly
   - Ensure no other audio issues occur

## Code Location References

- Audio streaming initialization: `kernel/Patch.c` lines 3086-3121
- Game-specific patches: `kernel/Patch.c` lines 1570-1627 (Majora's Mask example)
- Audio streaming functions: `kernel/Stream.c`
- DSP handler: `kernel/asm/__DSPHandler.S`
- DI audio commands: `kernel/DI.c` lines 488-536

