# Super Mario Eclipse Audio Streaming Fix

## Problem
Super Mario Eclipse has music going silent in Nintendont. The game requires audio streaming to work properly (turning it off causes the same issue), but the current implementation isn't working correctly.

## Solution
Disable DSP handler patching for Super Mario Eclipse, similar to how Majora's Mask is handled. This allows the game to use its native audio streaming implementation without interference from Nintendont's DSP handler.

## Implementation

### Step 1: Identify the Game
First, you need to identify Super Mario Eclipse. You can do this by:
1. Running the game and checking the debug output for the Game ID
2. Looking at the game's header/ISO
3. Checking the TITLE_ID value (GAME_ID >> 8)

The debug output will show: `Patch:Game ID = XXXXXXXX`

### Step 2: Add the Fix
Add the following code in `kernel/Patch.c` after the Majora's Mask patches (around line 1627, before the Ocarina of Time patches):

```c
// Super Mario Eclipse - Audio streaming fix
// TODO: Replace TITLE_ID with the actual game ID once identified
// TODO: Replace DOLMaxOff and memory signature with actual values
else if(TITLE_ID == 0xXXXXXXXX) // Super Mario Eclipse - Replace with actual TITLE_ID
{
	dbgprintf("Patch:[Super Mario Eclipse] audio streaming fix applied\r\n");
	// Disable DSP handler patching to allow game's native audio streaming
	DSPHandlerNeeded = 0;
}
```

### Alternative: If Game Signature is Known
If you know the DOL size and a unique memory signature (like how Majora's Mask is detected), you can use:

```c
// Super Mario Eclipse - Audio streaming fix
// TODO: Replace DOLMaxOff and memory signature with actual values
else if(DOLMaxOff == 0xXXXXXX && read32(0xXXXXXX) == 0xXXXXXXXX) // Super Mario Eclipse
{
	dbgprintf("Patch:[Super Mario Eclipse] audio streaming fix applied\r\n");
	// Disable DSP handler patching to allow game's native audio streaming
	DSPHandlerNeeded = 0;
}
```

### Step 3: Testing
1. Build Nintendont with the fix
2. Run Super Mario Eclipse
3. Verify that music plays correctly
4. Check debug output to confirm the patch is applied

## How It Works

The fix works by:
1. Detecting Super Mario Eclipse when it loads
2. Setting `DSPHandlerNeeded = 0` to skip DSP handler patching
3. Allowing the game to use its native audio streaming implementation

This is similar to how Majora's Mask is handled (lines 1564-1627 in Patch.c), where the game has its own audio streaming implementation that conflicts with Nintendont's DSP handler.

## Location in Code

The fix should be added in `kernel/Patch.c` in the `DoPatches()` function, around line 1627, after the Majora's Mask PAL patch and before the Ocarina of Time patches.

## Notes

- The exact TITLE_ID or game signature needs to be determined first
- This fix assumes the game has its own audio streaming that conflicts with Nintendont's implementation
- If this doesn't work, we may need to investigate the audio streaming implementation further or add special handling similar to Majora's Mask's MajoraAudioStream patch

