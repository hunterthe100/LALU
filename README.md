# LALU
An assignment during the 2014-2015 year in my Digital Electronics class, with some extra goodies.  Most documentation is done, let me know if I missed anything or if you want some help getting it running!

If you want to go read the course material we did in class you can find it here http://lasacs.com/de14/ between the Spring break and Microarchitecture bars. Scroll down a little ways, its not the first thing on the list.

How it works:
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

There are some programs I wrote during the school year inside the Programs file

The MAX and OPTIMIZED MAX programs work but I could never get any of the sort programs to work properly (it would have been cool, but I moved on to programing FPGA’s in class before I could get it done).  I wrote the find the min, but I forget if it works as I only tested it once.  Also for most of these you will have to program the data memory, but I will work on possibly including some files to load in.  Also remember with the max programs that this machine uses signed integers, which means the highest integer is efff(0111 1111 1111 1111), and the lowest is ffff(1000 0000 0000 0000) (I believe this is right, but feel free to correct me if I am wrong!).  

In addition if you right click on the leftmost RAM and edit contents you will be able to program the LALU, and that about wraps it up. 