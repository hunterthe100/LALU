# LALU
An assignment during the 2014-2015 year in my Digital Electronics class, with some extra goodies.  Most documentation is done, let me know if I missed anything or if  you want some help getting it running!

Public Domain

If you right click on the leftmost RAM and edit contents you will be able to program the LALU

Their are two parts to the instructions for the LALU
The first four bits are for the memory address to be used during certain operations.
The last four bits are for the operation being preformed.

These are encoded using HEX (0123456789ABCDEF), as hex is 4 “bits” long it is fairly simple to use for this purpose.

The LALU is equipped with a carry select adder (though I believe that this design is simple, and if you are really looking for a challenge to build, try a Lookahead Carry Unit adder which is also in this repo).  This carry select adder has two inputs, the A and B registers.

The LALU’s instructions execute 3 clock cycles after they are read out by the there are however, two exceptions to this, the djump and dcjump which execute two clock cycles after they are read.  This leads to some nuances while programing.  In addition their is a “dead” clock cycle which is initiated after the jump is initiated, their will be an example of this later in the programing section. Additionally these two commands also have two different ways to be accessed due to the instruction memory being five bits long.  The enchoding scheme for both of these goes as follows [xxxxx] [xxx], where the normal encoding scheme goes [xxxx] [xxxx] for all other commands.

The quark with the two different ways to access jump and dcjump could be fixed with a different encoding scheme (that being putting the instructions before the access to memory, but this might be a bit harder to encode.

00 = add
01 = subtract
x2 = load, uses the memory address inputted to load whatever is at that address into the “a register”
03 = exchange, simply exchanges the contents of the a and b registers
x4 = str, short for store, store the contents of the A register into the place specified in the memory address
x5 or xd = djump, since the instruction memory is 5 bits worth of information two different instructions are needed to call 
x6 or xe - dcjump, essentially a conditional jump which only will jump if the signed bit in the adder is positive (so queue up your add or subtract, then call this instruction right after).
07 = loadA, loads a value from (unless changed, will be detailed later) the first four bits in the a register.
08 = nop, nothing happens if you execute this maybe useful? (I think so)
09 = storeA, I forget exactly what this was supposed to do, but currently it just puts what was in Breg into the data memory (at least from my wiring this is what it does)

0a-0c are currently unused (have any ideas?)

0f = I never implemented jumpA fully so don’t use it, it was supposed to jump to an address from the memory,it essentially functions the same as jump i believe.

I am considering the addition of 32 bit addition capabilities (it’s only 16 right now) as the adder which is built in does have overflow detection, but no promises

Programs, 

The MAX and OPTIMIZED MAX programs work but i could never get any of the sort programs to work properly (it would have been cool, but I moved on to programing FPGA’s in class before I could get it done).  I wrote the find the min, but I forget if it works as I only tested it once.  Also for most of these you will have to program the data memory, but I will work on possibly including some files to load in.  Also remember with the max programs that this machine uses signed integers, which means the highest integer is efff(0111 1111 1111 1111), and the lowest is ffff(1000 0000 0000 0000) (I believe this is right, but feel free to correct me if I am wrong!).  


MAX PROGRAM
—————————————————————————————————
——Setup
00 - load min index to a 				62
01 - store a in index counter			44
02 - load the value at min index to maxval	07
03 - store a to the max				24
——this part increments the index & start of loop
04 - load index					42
05 - exchange					03
06 - load 1 to a					12
07 - add a and b					00
08 - store a to index					44
—— check if val at index is greater than max & jumps to code that puts new value in max & jumps to start
09 - load value at [index] to a			07
0a - exchange a and b				03
0b - load max to a					22
0c - subtract a and b				01
0d - jump conditional to instr 10			86
0e - nop						08
0f - jump to instr 05					2d
——this puts the new value in the max 
10 - load index					42
11 - load value at index to a			07
12 - store value at max				24
13 - jump to instr 04					26 // if we want to exit we have to exit here and add a few inst’s
14 - nop						08
15 - nop						08
16 - nop						08
17 - nop						08
18 - nop						08
19 - nop						08
1a - nop						08
1b - nop						08
1c - nop						08
1d - nop						08
1e - nop						08
1f - nop						08
——————————————————————————————————


SORT PROGRAM (contains modified max program)

-storeA takes value at breg and stores it at [addr areg 0:15] in memory
——————————————————————————————————


——Setup for max
00 - load min index to a 				62
01 - store a in index counter			44
02 - load the value at min index to maxval	07
03 - store a to the max				24


——this part increments the index & start of loop
04 - load index					42
05 - exchange					03
06 - load 1 to a					12
07 - add a and b					00
08 - store a to index					44
—— check if val at index is greater than max & jumps to code that puts new value in max & jumps to start
09 - load value at [index] to a			07
0a - exchange a and b				03
0b - load max to a					22
0c - subtract a and b				01
0d - jump conditional to instr 10			86
0e - nop						08
0f - jump to instr 05					2d  // (make sure this part works)
——this puts the new value in the max & exits if index = max
10 - load index					42
11 - load value at index to a			07
12 - store value at max				24

13 - load maximum index				72
14 - swap						03
15 - load index					42
16 - index - max					01
17 - jump conditional to instr 04			27 // CHANGED TO JUMP CONDITIONAL


18 - load max					22
19 - exchange					03
1a - load maximum index				72
1b - write(storeA) max to maximum index	0f	


1c - load 1						12
1d - exchange					03
1e - load maximum index				72	
1f - index - 1						01
00 - write a to maximum index			74
00 - jump to beginning 04				25		addr[0010 0]101


———————————————————————————————————


OPTIMIZED MAX PROGRAM (slower start)
——————————————————————————————————
——Setup
00 - load min index to a 				62
01 - store a in index counter			44

—— new start of loop, load index is out of loop because we load again at the end of the loop
—— check if val at index is greater than max & jumps to code that puts new value in max & jumps to start
02 - load value at [index] to a			07
03 - exchange a and b				03
04 - load max to a					22
05 - subtract a and b				01
06 - jump conditional to instr 09			4e //	01001[110]
07 - nop						08
08 - jump to instr 0d					6d  //	01101[101] // should be 0c
——this puts the new value in the max 
09 - load index					42
0a - load value at index to a			07
0b - store value at max				24

——this part increments the index (used to be at start of loop)
0c - load index					42 // put increment here
0d - exchange					03
0e - load 1 to a					12
0f - add a and b					00
10 - store a to index					44

11 - jump to instr 02					15 // if we want to exit we have to exit here and add a few inst’s 
12 - nop						08
13 - nop						08
14 - nop						08
15 - nop						08
16 - nop						08
17 - nop						08
18 - nop						08
19 - nop						08
1a - nop						08
1b - nop						08
1c - nop						08
1d - nop						08
1e - nop						08
1f - nop						08
———————————————————————————————————

FIND MIN PROGRAM
——————————————————————————————————
——Setup
00 - load min index to a 				62
01 - store a in index counter			44

—— new start of loop, load index is out of loop because we load again at the end of the loop
—— check if val at index is greater than max & jumps to code that puts new value in min & jumps to start
02 - load max to a					22
03 - exchange a and b				03

04 - load index to a					42

05 - load [index] to a				07
06 - subtract a and b				01
07 - jump conditional to instr 09			4e //	01001[110]
08 - nop						08
09 - jump to instr 0d					6d  //	01101[101]
——this puts the new value in the min
0a - load index					42
0b - load value at index to a			07
0c - store value at min				24

——this part increments the index (used to be at start of loop)
0d - load index					42 // put increment here
0e - exchange					03
0f  - load 1 to a					12
10 - add a and b					00
11 - store a to index					44

——added code
12 - load maximum index				72
13 - swap						03
14 - load index					42
15 - index - max					01

16 - jump conditional to instr 02			17 // jump if index = max (note index will be 1 greater than max if this executes, moving the increment code makes sure that this exits when the computed index = max)

00 - load min
00 - 

13 - nop						08
14 - nop						08
15 - nop						08
16 - nop						08
17 - nop						08
18 - nop						08
19 - nop						08
1a - nop						08
1b - nop						08
1c - nop						08
1d - nop						08
1e - nop						08
1f - nop						08
———————————————————————————————————


SORT PROGRAM ( with optimized max)

-storeA takes value at breg and stores it at [addr areg 0:15] in memory
——————————————————————————————————
——Setup
00 - load min index to a 				62
01 - store a in index counter			44

—— new start of loop, load index is out of loop because we load again at the end of the loop
—— check if val at index is greater than max & jumps to code that puts new value in max & jumps to start
02 - load value at [index] to a			07
03 - exchange a and b				03
04 - load max to a					22
05 - subtract a and b				01
06 - jump conditional to instr 09			4e // change again		01001[110]
07 - nop						08
08 - jump to instr 0d					6d  // change again		01101[101]
——this puts the new value in the max 
09 - load index					42
0a - load value at index to a			07
0b - store value at max				24

——this part increments the index (used to be at start of loop)
0c - load index					42 // put increment here
0d - exchange					03
0e - load 1 to a					12
0f - add a and b					00
10 - store a to index					44

——added code
11 - load maximum index				72
12 - swap						03
13 - load index					42
14 - index - max					01

15 - jump conditional to instr 02			17 // jump if index = max (note index will be 1 greater than max if this executes, moving the increment code makes sure that this exits when index = max)


16 - load max					22
17 - exchange					03
18 - load maximum index				72
19 - write(storeA) max to maximum index	0f	


1a - load 1						12
1b - exchange					03
1c - load maximum index				72	
1d - index - 1						01
1e - write a to maximum index			74
1f - jump to beginning 00				05		addr[0010 0]101
———————————————————————————————————

