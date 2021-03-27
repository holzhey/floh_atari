# FLOH ATARI

![Floh in game screenshot](https://github.com/holzhey/floh_atari/raw/main/floh.png)


This is a game for the BASIC 10 Liner Contest 2020 based on Atari 8bit platform. More about the contest can be found here: https://gkanold.wixsite.com/homeputerium/rules2021

## Game info

- Title: Floh (flea)
- Platform: Atari 8bit (developed on a 800XL)
- Author: Alexandre Lehmann Holzhey
- Language: ATARI BASIC
- Category: PUR-120

## Files description

* README.md: This file, with general information about the game.
* FLOH.BAS: Source code of the game, in BASIC langauge.
* floh.png: Screenshot of the game.
* floh.atr: Disk image with the game saved inside, for using with emulator.

## The game

You are a flea (floh, in Deutsch) and like to jump around. You don't like to do it at the same block, so you must jump into another block each time. You can control the flea using joystick at port 0 and jump higher just keeping the button pressed when jumping. You suceed when jumping all blocks, keeping jumping in a different one each time. Be aware that the blocks can be jumped only 5 times, after that they will dissapear!! So think about the order you will jump them, the color of the blocks will change to tell you that there are only a few times mores to jump at the block.

## How to play

The flea keeps jumping all the time, you have just to move it to the right or to the left, using the joystick. You can jump higher pressing the button.

All blocks should be jumped until they dissapear from the screen. That happens when you jump some times on them, so keep an eye on the order you do it and don't jump on the same block twice or you will die! Also, if you miss a block you will die as well.

## Using an emulator

The game was developed on a Atari 800 XL computer. You can use an emulator, i tested on Atari 800 MacX (https://github.com/atarimacosx/Atari800MacX)
Just configure to use `floh.atr` and when ready type `LOAD "D:FLOH.BAS"`, then execute with `RUN`.

## Game logic

### `10 PB=PEEK(106)-8:POKE 106,PB:GRAPHICS 19:POKE 54279,PB:PMB=PB*256+512:POKE 559,46:POKE 53277,3:POKE 623,1`

Move the RAMTOP to give some space for the PM (2K). Setup graphics mode, setup PMBASE page and calculate PMBASE address.
Enable double line resolution for PM at SDMCTL, enable PM at GRACTL and configure priority with GRPRIOR.

### `20 POKE 704,255:POKE 53256,0:FOR I=PMB TO PMB+128:POKE I,0:NEXT I:Y=PMB:IY=1:YL=PMB+200:X=110:DIM GC(8),GX(8),GY(8)`

Configure PM0 color at PCOLR0, PM0 size to normal at SIZEP0 and clear PM0 data.
Configure PM0 Y bsed on top of the screen, setup Y increment (IY) and vertical limiter to bottom of screen (PM0 vertical reference).
Setup PM0 X to the middle of the screen (we know there is a block at that position), to give a bit of difficulty to the player due to the required ordering of jumping around.
Create arrays for handling the blocks.

### `60 U=-1:LP=U:FOR F=0 TO 8:GY(F)=RND(1)*10+7:GX(F)=F*3+5:GC(F)=5:NEXT F:GOSUB 600`

Variable U will provide the block where the flea had jumped and LP keeps the last block it jumped, so we can know if the same block was used.
Initilize the blocks, giving random positions at the screen for the vertical values and pre-fixed horizontal positions.
Amount of allowed jumps are set to 5, then we call the routine to render the blocks.

### `100 POKE Y,0:POKE Y+1,0:POKE 53278,1:Y=Y+IY/10:POKE Y,3:POKE Y+1,3:POKE 53248,X:IY=IY+2:ON Y+IY>=YL GOTO 800`

Start of the main game loop. Clear current vertical position of PM0, calculate next position and plot it. Since we have doubled the horizontal size we do the same for the vertical (2 pokes). We also update horizontal position of PM0 and increment the vertical increment (IY). If we reach the bottom of the screen, jump to die routine.
In between we poke to clean up the collision detection, because we invert the Y increment when the flea jumps but more collisions could happen just after this. We don't care about those collisions and they make the block jumps counter behave strange, so we handle this below and make sure the collisions detection are cleared after that happens.

### `110 X=X+((STICK(0)=7 AND X<200)*2)-((STICK(0)=11 AND X>0)*2):ON PEEK(53252)=0 OR IY<0 GOTO 100:LP=U`

Simple joystick evaluation to move PM0, with lateral limits. Check if there is a collision and repeat the loop.
We also repeat the loop in case the flea is going up, since we don't want to jump upside down having blocks on above. Also, we ignore multiple collisions that could happen.
Having a jump, first we keep last block the flea has jump into LP.

### `500 IY=-20-((STRIG(0)=0)*((Y-PMB)/5)):BX=(X-40)/4:FOR U=0 TO 8:IF BX+1<GX(U) OR BX-1>GX(U)+1 THEN NEXT U`

Simple joystick evaluation for the button. We are in the beggining of a jump, so we just add a negative fixed value for IY. But if the player push the button, we add a bigger value, but limited a bit to avoid jump over the top of the screen.
Then we calculate PM0 position based on the graphics mode resolution and iterate over the blocks to find where the flea has jumped. The loop will always find a block, so a IF condition keeps the loop otherwise. Note that the U variable now have the jumped block reference, so we can compare with LP as described above.

### `505 FOR S=14 TO 0 STEP -2:SOUND 0,S*10,10,S:NEXT S:GC(U)=GC(U)-1:GOSUB 600:ON U=LP GOTO 800:IF P>0 THEN 100`

Cool sound for the jump action! Then we decrease the counting of jumps for the block and call the blocks render routine.
Here we check for the block where the flea has bounced with the previous one, if is the same we go for the die routine.
Remember the -1 for U and LP at the beggining of the code? This allows the flea to jump for the first time and don't die here.
The die routine counts the number of visible blocks, we check for it and go to the main loop if there is still some block to jump.

### `550 FOR R=15 TO 0 STEP -2:FOR S=150 TO 0 STEP -5:SOUND 0,S,10,R:NEXT S:NEXT R:RUN`

Success routine! The flea has jumped over all blocks, we play some nice sound and restart.

### `600 P=0:FOR B=0 TO 8:COLOR (1*GC(B)>0)+(1*GC(B)>2):P=P+GC(B):PLOT GX(B),GY(B):DRAWTO GX(B)+1,GY(B):NEXT B:RETURN`

Blocks rendering routine. We just draw the blocks based on stored values. The color is based on the amount of remaining jumps: having less than 3 jumps remaining the color change and having no jumps the color is black. We use that information to quickly calculate if there is a block to jump, just adding the color into the P variable.

Remark: that multiplication by 1 is a mistake and i had no time to fix it vefore submiting. :-( 

### `800 FOR R=15 TO 0 STEP -5:FOR S=0 TO 150 STEP 50:SOUND 0,S,10,R:SOUND 1,S*1.2,10,R:SOUND 2,S*1.3,10,R:NEXT S:NEXT`

Die routine. Also a nice sound (besides the event) and restart.


## Remarks

This is my first code at all with the Atari. The machine is very nice, with cool features. It requires a lot of reading before starting anything, much more harder to code compared to my previous MSX Floh. I also have lots of difficulty to achive the 120 chars, but it is done!!

