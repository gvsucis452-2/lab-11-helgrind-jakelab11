Jacob Parsons

To begin, type make to build all the different programs. Examine the Makefile for more details on how that works.

1. First build main-race.c. Examine the code so you can see the (hopefully obvious) data race in the code. Now run helgrind (by typing valgrind \--tool=helgrind ./main-race) to see how it reports the race.  
   1. Does it point to the right lines of code?  
      1. Yes the output shows a conflict in main at offset 0x109236 and 0x10923F, and worker at offset 0x1091BE  
   2. What other information does it give to you?  
      1. It gives the variable name, address 0x10c014 is 0 bytes inside “balance”  
      2. Access type: distinguishes between a read an a write  
      3. Lock status  
      4. Thread creation trace  
      5. Conflict history  
2. What happens when you remove (e.g., comment out) one of the offending lines of code?  
   1. Valgrind reports no errors  
3. Add a lock around one of the updates to the shared variable. What does helgrind report?  
   1. It reports 2 errors with thread 2 having one lock at address 0x10c060 and no locks held on thread 1  
4. Now add locks around both. What does helgrind report?  
   1. It reports no errors   
5. Next examine main-deadlock.c. This code has a problem known as deadlock (which we discuss in much more depth in a forthcoming chapter). Based on this code,  
   1. Describe what a deadlock is.  
      1. When at least two threads are blocked due to each thread waiting for a resource held by another thread  
   2. Why specifically does this code have a deadlock?  
      1. Thread 1 grabs lock 1, and thread 2 grabs lock 2 at the same time. The program hits a wait, meaning neither can move forward, therefore freezing the program.   
6. Run helgrind on this code. What does it report?  
   1. It reports 1 error from one context, being a lock order violation  
7. Examine main-deadlock-global.c.  
   1. Does it have the same problem that main-deadlock.c has?  
      1. No  
   2. Why or why not?  
      1. The global lock ensures that only one thread can be in the “acquisition phase” at a time. This breaks the wait condition required for a deadlock.  
   3. Should helgrind be reporting the same error?  
      1. No  
   4. What does this tell you about tools like helgrind?  
      1. That helgrind monitors execution paths and lock sequences.  
8. Look at main-signal.c. This code uses a variable (done) to signal that the child is done and that the parent can now continue. Why is this code inefficient? (what does the parent end up spending its time doing, particularly if the child thread takes a long time to complete?)  
   1. There is a busy-waiting while loop being called which wastes CPU cycles  
9. Run helgrind on this program.  
   1. What does it report?  
      1. It reports 23 errors from 2 contexts, both having data races during the read and write.  
   2. Is the code correct?  
      1. No the code is not correct due to no the data race and no synchronization.  
10. Now look at the slightly modified version of the code found in main-signal-cv.c. This version uses a condition variable to do the signaling (and associated lock).  
    1. Why is this code preferred to the previous version?  
       1. It is properly synchronized. Instead of wasted cycles, it calls wait and is put to sleep by the OS.  
    2. Is it correctness, or performance, or both?  
       1. Both due to the “busy-waiting” being eliminated and freeing up the CPU.  
11. Once again run helgrind on main-signal-cv. Does it report any errors?  
    1. No error are reported.