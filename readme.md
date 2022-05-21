###  <center> Lab 2 Report

******

The specific implementation of this lab is a multi-process scheduling system which called the Hypothetical Operating System Testbed. The process in **HOST** is scheduled by four priority scheduler and also limited by software and hardware resources. 

In the **HOST**, the real-time job is the highest priority and obeys the **FCFS** principle, and the user job follow the three-level feedback scheduler whose quantum is one second. The main structure of the **HOST** is shown as below.

![image-20220521152037646](/Users/frodo/Library/Application Support/typora-user-images/image-20220521152037646.png)

Furthermore, the scheduler is also limited by the following I/O resources

* Two printers
* One scanner
* One modem
* Two CD driver
* 1024Mb memory

Therefore, the whole dispatcher logic flow is shown as below

![image-20220521152401984](/Users/frodo/Library/Application Support/typora-user-images/image-20220521152401984.png)

Until now, we finished a short introduction of this Lab, and then we begin to focus on the details of the project.

****

* **Describe and discuss the memory allocation algorithm you used, and prove your final design choice**
  The project in mab.c implements four memory allocation algorithms: First Fit, Next Fit, Best Fit and Worst Fit. The First Fit is easy to implement by finding the first memory space that is big enough and allocated it to the process will be OK. And then the Next Fit can be implemented based on that First Fit because it just allocated the second memory space that is big enough to the process. Remember if not find till the end, it will start search again from the beginning. The  implements of the Best Fit and Worst Fit is actually the same, for one it search the biggest memory space of all the memory spaces that is big enough, and for another it search the smallest memory space of all the memory spaces that is big enough, and then allocated it.
  In this project, we must ensure the allocated memory is a continuous memory space, which always belong to the specific process during its execution. At the same time, in order to ensure that the real-time process will not be blocked, at least 64MB of continuous memory space must be reserved at all times. Here we choose the First-Fit for default, because it anyway provide continuous memory space for real-time processes and is suitable for FCFS algorithm. We test different algorithms and find that it can meet the required conditions better than other algorithms.

****

* **Describe and discuss the structure and logical framework of process queuing, scheduling, memory allocation and resource allocation**

  The complete process scheduling process is described as follows.

  1. Initialization. Including all dispatcher queues, memory and resource allcation structures;
  
  2. Fill dispatcher queue from dispatch list file;
  
  3. Start dispatcher timer;
  
  4. While there's anything in any of the queues or there is a currently running process:
  
     1. Unload any pending processes from the input queue. Noting the process need fit three conditions:
        1. Enough memory
        
        2. Enough I/O resource
        
        3. Memory allocateable
        
     
     Also considering the problem of priority that the real-time jobs is awlays prior to the user jobs 
     
     2. Unload pending processes from the user job queue:
        1. dequeue process from user job queue;
        2. allocate memory to the process;
        3. allocate I/O resources to the process;
        4. enqueue on appropriate priority feedback queue.
        
     3. If a process is currently running:
        1. Decrement process remainingcputime;
        2. If time's up, terminate it and free the process;
        3. If it is a user process and other processes are waiting in any of the queues, suspend it and reduce its priority and enqueue it to approriate feedback queue.
        
     4. If no process currently running and real time queue and feedback queues are not all empty
        1. Dequeue process from the highest priority queue that is not empty;
        2. If already started but suspended, restart it (send SIGCONT to it)else start it (fork and exec);
        3. Set it as currently running process.
        
     5. sleep for one second;
     
     6. Increment dispatcher timer;
     
     7. Go back to 1;
     
  5. Exit.
  Notice that the condition of resource allocation and the condition of I/O resource allocation will be checked before each exec like that in the 5.1.
  Moreover, the program will print the error messages on the terminal, which can play an important role in more in-depth robustness testing.
  
  ***
  
* **Discuss and describe the overall framework of the program, describe each module in the code and the main functions**
  
  The structure of the program is actually relatively simple, consisting of a series of .c and  .h file and they have no inclusion relationship. They are all placed in the src folder. It can be compiled by the makefile file in the same level directory of src to obtain the binary file host, which can be run through the following instructions.
  
  ```c
  ./hostd [-mf|-mn|-mb|-mw] <dispatchlist>
  ```
  
  -mf|-mn|-mb|-mw is choosing different memory allocation methods.
  
  The .c file contains the main functions and scheduler logic, and the .h file contains some constants and classes for its corresponding .c file, so the following only some main api functions in .c file are introduced.
  
  * sigtrap.c
  
  The functions contained in this file are mainly used to run the program at the terminal, which is for the visual display of process scheduling. They tick away reporting process id and tick count every second to help identify specific processes. No change is made in this file.
  
  * rsrc.c
  
    1. *rsrcChk()--check that resources are available now, return TRUE or FALSE- no allocation is actally done*
    2. *rsrcChkMax()--check that resources will ever be available, return TRUE or FALSE*
    3. *rsrcAlloc()--allocate resources, return TRUE or FALSE if not enough resources available*
    4. *rsrcFree()--free resources*
  
  * pcb.c
  
    The functions inside is about set the process information module PCB.
  
    1. *PstartPcb() - start (or restart) a process*
  
       ​    *returns:*
  
       ​      *PcbPtr of process*
  
       ​      *NULL if start (restart) failed*
  
    2. *suspendPcb() - suspend a process*
  
       ​    *returns:*
  
       ​      *PcbPtr of process*
  
       ​      *NULL if suspend failed*
  
    3. *terminatePcb() - terminate a process*
  
       ​    *returns:*
  
       ​      *PcbPtr of process*
  
       ​      *NULL if terminate failed*
  
    4. *printPcb()*
  
       ​    *- print process attributes on iostream*
  
       ​    *returns:*
  
       ​      *PcbPtr of process*
  
    5. *printPcbHdr() - print header for printPcb*
  
       ​    *returns:*
  
       ​      *void*
  
    6. *createnullPcb() - create inactive Pcb.*
  
       ​    *returns:*
  
       ​      *PcbPtr of newly initialised Pcb*
  
       ​      *NULL if malloc failed*
  
    7. *enqPcb ()*
  
       ​     *- queue process (or join queues) at end of queue*
  
       ​      *- enqueues at "tail" of queue.* 
  
       ​    *returns head of queue*
  
    8. *deqPcb ()- dequeue process - take Pcb from "head" of queue.*
  
       ​    *returns:*
  
       ​      *PcbPtr if dequeued,*
  
       ​      *NULL if queue was empty*
  
       ​      *sets new head of Q pointer in adrs at 1st arg*
  
  * mab.c
  
    The functions in side is about the memory allocation.
  
    1. *memChk ()- check for memory available (any algorithm)*
  
       ​    *returns address of "First Fit" block or NULL*
  
    2. *memChkMax () - check for over max memory*
  
       ​    *returns TRUE/FALSE OK/OVERSIZE*
  
    3. *memAlloc ()- allocate a memory block*
  
       ​    *returns address of block or NULL if failure*
  
    4. *memFree ()- de-allocate a memory block*
  
       ​    *returns address of block or merged block*
  
    5. *memMerge()- merge m with m->next*
  
       ​    *returns m*
  
    6. *memSplit()- split m into two with first mab having size*
  
       ​    *returns m or NULL if unable to supply size bytes*
  
    7. *memPrint()- print contents of memory arena*
  
          *no return*
  
  * hostd.c
  
  The functions inside is the main logic of the host.
    1. main()- the main logic of the host
  
    2. *CheckQueues()*
  
         *check array of dispatcher queues*
  
         *return priority of highest non-empty queue*
  
       ​          *-1 if all queues are empty*
  
    3. *StripPath();*
  
       *strip path from file name*
  
  ***
  
  * **Discuss why host should use this multi-level scheduling strategy, and compares it with the real operating system scheme. Summarize the shortcomings of this strategy and put forward possible improvement suggestions. Including memory and resource allocation scheme**
  
    On the one hand, the system design can ensure that the real-time process with the highest priority will never be blocked, and FCFS can also prevent the problem of hunger to a certain extent. On the other hand, the three-level feedback queue can well classify the user processes, so that the processes with priority can run in the corresponding queue without affecting each other. Overall, such a strategy has completeness to some extend.
  
    Compared with the process scheduling system of Linux, one of the main differences is that its system has more priority classification (nice value) and different quantums. On the other hand, it is more intelligent to calculate and set the real-time relative priority of ordinary processes, and calculate the length of the corresponding quantum according to the nice value. Such a system is obviously more complete.
  
    Shortcomings and possible improvement are shown in comparison. Firstly, the quantum is fixed, and such setting in the three-level feedback queue can not well reflect the difference of priority. At the same time, the resources  occupation may be stuck at the entrance. It will be better to use the banker algorithm to judge whether a deadlock will occur. Finally, the priority classification is not complete enough.
  
  
  
  
  ​         
  
  

