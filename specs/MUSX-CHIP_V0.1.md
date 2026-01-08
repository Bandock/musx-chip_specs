# MUSX-CHIP Extension V0.1 (Work In Progress)

This extension builds upon the foundations of CHIP-8, SuperCHIP (V1.0/V1.1), and XO-CHIP (By John Earnest).  It also has some support for instructions from CHIP-8E.

Technical Specifications:
- 64KB RAM Support (First 512 bytes reserved)
- 16 Color Support
- Introduces Tracker Mode (Must be enabled) that works alongside Beeper Mode
- Two Pitch Values (Beeper Pitch Value and Tracker Pitch Value)
- Tempo Support for Tracker Mode (15 Hz-240 Hz, 4-bit in increments of 15)
- Note Duration Timer Register for Tracker Mode (8-bit)
- Volume Support for Beeper and Tracker Modes
- Introduces the TNSI (Tracker Note Source Index) Register (Serves as the pointer for notes in Tracker Mode)
- Introduces the TDSI (Tracker Dynamics Source Index) Register (Serves as the pointer for dynamics in Tracker Mode)
- Tracker Mode Audio Buffer Support

  
Formulas:
- Audio Playback Rate = 4000 * 2 ^ ((Pitch Register - 64) / 48)
- Tracker Mode Pitch = 20 + (Octave * 48) + ((Note Value - 1) * 4)
- Tracker Volume = 255 * ((Dynamic Target + 1) / 16)
- Note Duration Timer and Dynamics Duration Timer = 255 >> (7 - Duration Value)


Beeper Mode and Tracker Mode Operation:
- Tracker Mode must be enabled using the 00F1 instruction for it to start operating as it is disabled by default.  This gives the opportunity to set the Tracker Note Source Index register that points to 16-bit data using the Tracker Mode Note Word Format along with the Tracker Dynamics Source Index register that points to 16-bit data using the Tracker Mode Dynamics Word Format.  Once Tracker Mode is enabled, it will start playing audio using that data whenever the Sound Timer register is 0.  Tracker Mode is excellent for music or special sound effects.
- Whenever the Sound Timer Register is nonzero, it will switch to Beeper Mode.  This will operate in the fashion dating back to CHIP-8.  Beeper Mode is excellent for producing sound effects.
- Beeper Mode and Tracker Mode use their own audio buffers, allowing for Beeper Mode to play its own sound effects separately from Tracker Mode.
- Tracker Mode's Dynamics Buffer enables volume control as notes are played.  You can apply multiple dynamics on the same note or multiple notes on the same dynamic.


Registers:

|Register |Description |Value Range |Accessible |
|---------|------------|------------|-----------|
|V0-VF|General Purpose|0-255|Yes|
|DT|Delay Timer|0-255|Yes|
|ST|Sound Timer|0-255|Only through one instruction|
|NDT|Note Duration Timer|0-255|No, it is only accessed by Tracker Mode|
|DDT|Dynamics Duration Timer|0-255|No, it is only accessed by Tracker Mode|
|PTCH|Pitch|0-255|Only through one instruction|
|BVOL|Beeper Volume|0-255|Only through one instruction|
|TVOL|Tracker Volume|0-255|No, it is only accessed by Tracker Mode|
|PC|Program Counter|0-65535|Only by various instructions|
|I|Address|0-65535|Yes|
|TNSI|Tracker Note Source Index|0-65535|Yes|
|TDSI|Tracker Dynamics Source Index|0-65535|Yes|


Tracker Mode Note Word Format (Big Endian):

|Bits |Description |Value Range |
|-----|------------|------------|
|10-15|Next Note Offset (1 + Offset by Word Alignment)|-32 to 31|
|9|Unused|0 to 1|
|6-8|Note Duration|0 to 7|
|4-5|Octave (Affects Tracker Mode Pitch)|0 to 3|
|0-3|Note (0 = Indicates Rest, otherwise affects Tracker Mode Pitch)|0 to 12|


Tracker Mode Dynamics Word Format (Big Endian):

|Bits |Description |Value Range |
|-----|------------|------------|
|10-15|Next Dynamics Offset (1 + Offset by Word Alignment)|-32 to 31|
|7-9|Dynamics Duration|0 to 7|
|5-6|Dynamics Mode (0 = Static, 1 = Crescendo, 2 = Decrescendo, 3 = Unused)|0 to 3|
|0-3|Dynamics Target|0 to 15|


Supported Instructions:

|Instruction |Description |Extension Inherited From |MUSX-CHIP Modified |
|------------|------------|-------------------------|---------------------|
|00CN|Scroll Display N Pixels Down|SuperCHIP V1.1|No|
|00DN|Scroll Display N Pixels Up|XO-CHIP|No|
|00E0|Clear Screen|CHIP-8|No|
|00EE|Return Subroutine|CHIP-8|No|
|00F0|Disable Tracker Mode|MUSX-CHIP V0.1|No|
|00F1|Enable Tracker Mode|MUSX-CHIP V0.1|No|
|00FB|Scroll Right 4 Pixels|SuperCHIP V1.1|No|
|00FC|Scroll Left 4 Pixels|SuperCHIP V1.1|No|
|00FD|Exit Interpreter|SuperCHIP V1.0|No|
|00FE|Disable High Resolution Mode|SuperCHIP V1.0|No|
|00FF|Enable High Resoltuion Mode|SuperCHIP V1.1|No|
|1NNN|Jump to Address at NNN|CHIP-8|No|
|2NNN|Call Subroutine at Address NNN|CHIP-8|No|
|3XNN|Skip the Following Instruction If VX == NN|CHIP-8|No|
|4XNN|Skip the Following Instruction If VX != NN|CHIP-8|No|
|5XY0|Skip the Following Instruction If VX == VY|CHIP-8|No|
|5XY2|Store VX to VY in memory starting at I (Does not increment I)|CHIP-8E|No|
|5XY3|Load VX to VY from memory starting at I (Does not increment I)|CHIP-8E|No|
|6XNN|Set VX to NN|CHIP-8|No|
|7XNN|Add NN to VX|CHIP-8|No|
|8XY0|Set VX to VY|CHIP-8|No|
|8XY1|Set VX to VX OR VY|CHIP-8|No|
|8XY2|Set VX to VX AND VY|CHIP-8|No|
|8XY3|Set VX to VX XOR VY|CHIP-8|No|
|8XY4|Add VY to VX (VF = 00 for Borrow, 01 for No Borrow)|CHIP-8|No|
|8XY5|Subtract VY from VX (VF = 00 for Borrow, 01 for No Borrow)|CHIP-8|No|
|8XY6|Store VY shifted one bit to the right in VX (VF=LSB)|CHIP-8|
|8XY7|Set VX to VY - VX (VF = 00 for Borrow, 01 for No Borrow)|CHIP-8|No|
|8XYE|Store VY shifted one bit to the left in VX (VF = MSB)|CHIP-8|No|
|9XY0|Skip the Following Instruction If VX != VY|CHIP-8|No|
|ANNN|Set I to NNN|CHIP-8|No|
|BNNN|Jump to Address at NNN + V0|
|CXNN|Set VX to Random Number (Mask = NN)|CHIP-8|No|
|DXYN|Draw Sprite at VX, VY (If N == 0, then draw a 16x16 sprite) (VF = 01 if pixels were unset, 00 if no pixels were unset)|CHIP-8/SuperCHIP V1.0|No|
|EX9E|Skip the Following Instruction If Hex Key Pressed == VX|CHIP-8|No|
|EXA1|Skip the Following Instruction If Hex Key Not Pressed == VX|CHIP-8|No|
|F000 NNNN|Set I to NNNN|XO-CHIP|No|
|FN01|Sets the current drawing bit plane (N = 0 for No Draw, N = 1 for Plane 1, N = 2 for Plane 2, N = 3 for Plane 1 and 2)|XO-CHIP|No|
|F002|Load the Beeper Mode's audio buffer from memory at I|XO-CHIP|No|
|F003|Load the Tracker Mode's audio buffer from memory at I|MUSX-CHIP V0.1|N/A|
|F004|Sets TNSI to I|MUSX-CHIP V0.1|N/A|
|F005|Sets TDSI to I|MUSX-CHIP V0.1|N/A|
|FX07|Store Delay Timer to VX|CHIP-8|No|
|FX18|Set Sound Timer to VX|CHIP-8|No|
|FX1E|Add Value Stored in VX to I|CHIP-8|No|
|FX29|Point I to 5-byte font sprite for digit in VX (0-F)|CHIP-8|No|
|FX30|Point I to 10-byte font sprite for digit in VX (0-F)|SuperCHIP V1.1|No|
|FX33|Store BCD in VX at I, I+1, and I+2|CHIP-8|No|
|FX3A|Sets the Beeper Mode's pitch to the value stored in VX|XO-CHIP|No|
|FX3B|Sets the Beeper Mode's volume to the value stored in VX|MUSX-CHIP V0.1|N/A|
|FX55|Store V0 to VX in memory starting at I (I = I + X + 1, CHIP-8 original behavior)|CHIP-8|No|
|FX65|Load V0 to VX from memory starting at I (I = I + X + 1, CHIP-8 original behavior)|CHIP-8|No|
|FX75|Store V0 to VX in RPL User Flags (X <= 15)|SuperCHIP V1.0, XO-CHIP V1.1|No|
|FX85|Store V0 to VX in RPL User Flags (X <= 15)|SuperCHIP V1.0, XO-CHIP V1.1|No|
