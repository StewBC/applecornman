# APPLECORN MAN

Applecorn Man is a BBC Basic program, made to work with Applecorn.  Applecorn is for running BBC ROMS on the Apple II line of computers.  See the Applecorn project here: https://github.com/bobbimanners/Applecorn  

This Pac Man game will work only on machines that have a working TIME command in Basic (so the IIgs or a //e with Mockingboard).  Apart from the need for time, acceleration is really also needed.  I do play it on my IIgs with Applesqueezer but the game is best played in an emulator.  This video showing a little bit of gameplay is using the GSplus emulator with g_limit_speed = 3.  
Video: https://youtu.be/zjvO9Fhhlcw

Applecorn Man Features:
* Animating UI with character intro's
* 1 or 2 players
* 1 Pac Man level and 2 Ms Pac Man levels (3 levels to play through)
* 4 frames of animation per direction, per "sprite" (currently repeating 2 characters except for player death)
* Acorn Man is faster in open spaces, slower when eating dots (maybe not tuned as well as can be)
* Ghosts are slower when vulnerable
* AI similar to Pac Man AI
* Level layouts can be added (modified, really, as memory is now full)
* . The ghost house must have the same dimensions (but can be moved)
* . The ghost directions at start are hard coded
* . The patrol locations are hard coded (but this can easily be adapted with a new layout type)
* . Wrapping can only happen left/right, not up/down
* . Ghost A must be outside house (hard coded for patrol) and will go left
* No sound (not really a feature)
* And lastly, almost certainly bugs

If I drop a level or find more RAM some other way, and Applecorn starts to support user definable characters (VDU 23) in MODE 1, say, then I can make this game look pretty nice but till then, this uses regular characters in a 40 col text mode (mode 6).  

I wrote the code in VS Code and used a tokenizer program to turn my text into a valid BBC Basic program. The basic program doesn't have line numbers (they are all 0) since I use only PROC and FN for flow - no GOTO or GOSUB.  Also, THEN is not needed after IF which I accidentally discovered, so I have no THEN.  

Thank you Bobbi for Applecorn - it's awesome!  

Thank you  
Stefan Wessels  
swessels@email.com  
20 November 2022  

## Key to variables and data/state definitions.

### Global Variables
Name |Desc
--- |---
AL | active level (0 based)  
AP | active player 0/1  
ET | elapsed time (only for dt calculation)  
GGT | ghost vulnerable time when power dot eaten  
HS | high score  
K | keyboard key pressed  
LC | Num levels -1  
NAP | number of active players 0 or 1  
NT | no time bool is true when TIME doesn't work   
PAF | player animation frame 0..3  
PAT | player animation time - move/animate < 0  
PD | player direction (0=up,1=right,2=down,3=left)  
PDX | player desired x (-1/0/1) Joy direction  
PDY | player desired y (-1/0/1)  
PNX | player next x  
PNY | player next y  
PWF | power dot animation frame  
PWT | power dot animation time  
PX | player x  
PY | player y  
RU | running (alive) bool (-1 is true)  
SL | show level bool  
ST | level state - patrol or hunt  
STM | time to level state switch  

### Global Arrays
Name |Desc
--- |---
DIX(3) | x components of direction (0,1,0,-1)  
DIY(3) | y components of direction (-1,0,1,0)  
GA(4,3,3) | ghost animation frame indices - each ghost has 4 frames per direction  
GAF(3) | ghost animation frame counter  
GAT(3) | ghost time to move/animate  
GCT(3) | ghost anim/move rate (GAT=GCT when GAT<0)  
GD(3) | ghost direction (0=up, clockwise ..3)  
GM(3) | ghost mode (=1 if ghost was alive when power dot was eaten)  
GS(3) | ghost state (see below)  
GST(3) | ghost state timer (only used to release ghosts from hold)  
GSX(1,3) | ghosts start x  
GSY(1,3) | ghosts start y  
GTX(3) | ghost x position  
GTY(3) | ghost y position  
GX(3) | ghosts X locations  
GY(3) | ghosts Y locations  
HOX(3,3) | ghost hunt offsets x  
HOY(3,3) | ghost hunt offsets y  
LEX(3) | level exit x location  
LEY(3) | level exit y location  
LL$(5) | characters to print for level design  
LS(LC,24,23) | level layout data (see below)  
LSS(LC) | # dots in each level  
LTX(3) | ghost patrol targets X  
LTY(3) | ghost patrol targets Y  
NE(3,3) | next dir for a dir (same,right,left,reverse per direction)  
OP(3) | opposite directions ex. OP(0)=2  
OS(1,24,23) | of screen (back store) level layout for each player  
PA$(4,3) | player animation frames to print  
PDE(1) | player dots eaten  
PL(1) | player lives  
PNL(1) | player needs next level init bool  
PS(1) | player score  
PSX(1) | per level player start x  
PSY(1) | per level player start y  
PV(1) | player current level  
PWA$(1) | power dot animation characters to print  
PWX(LC,3) | location of the power dots X on each level  
PWY(LC,3) | location of the power dots Y on each level  
UIX(4) | ui marquee starting draw dots x  
UIY(4) | ui marquee starting draw dots x  

### Level Layout Data
Name |Desc
--- |---
a | walls  
b | ghost house door  
c | blank (space)  
d | dots to eat  
e | power dots  
f | ghost (starts alive outside house)  
g | ghost  
h | ghost  
i | ghost  
j | player  

### Ghost States
Name |Desc
--- |---
gsPTRL =1 | patrol (target corners)  
gsHUNT =2 | hunt (target Applecorn Man)  
gsHOLD =3 | hold (in house at start)  
gsEXIT =4 | exit house  
gsDEAD =5 | dead (seek house)  

### Indices in OST%
Name |Desc
--- |---
gtGDS  =0 | Ghost Dead movement Speed (movement and animation are tied)  
gtGNS  =1 | Ghost Normal movement Speed  
gtGR   =2 | Ghost Release from house (index * this value)  
gtIH   =3 | Intro Hold for UI  
gtM    =4 | Marquee speed  
gtPS   =5 | Player movement Speed  
gtPGH  =6 | Pre Go Hold (How long READY stays up)  
gtSH   =7 | State Hunt (duration of hunt mode)  
gtSP   =8 | State Patrol (duration of patrol mode)  
gtGML  =9 | Ghost Mode Len (duration of ghost mode after powerup)  
gtPA   =10 | Power Anim (animation speed of power up)  

-- OST% is for timing.  By setting dt to 1 in PROCtime and zero-ing out some of these, this will work on a machine without TIME but the fps at 1 or even 2.8 MHz makes it not worth-while.  

## Level design constraints
1. The ghost house must have the same dimensions (but can be moved)  
2. The ghost directions at start are hard coded  
3. The patrol locations are hard coded (but this can easily be adapted with a new layout type)  
4. Wrapping can only happen left/right, not up/down  
5. Ghost f must be outside house (hard coded for patrol) and will go left  
