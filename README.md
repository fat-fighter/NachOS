# NachOS Implementation

System calls to be implemented -

* GetReg
* GetPA
* GetPIP
* GetPPID
* NumInstr
* Time
* Yield
* Sleep
* Exec
* Exit
* Join
* Fork

## General Workflow

The system calls are defined within system call wrapper functions which are
called from within a user program. These system calls are responsible for
incrementing the program counter and hence handle the program flow. The system
calls are hard coded in the ExceptionHandler() function in exception.cc file in
an if...else block

The type of system call / exception is passed as an argument *which* in this
function. The values passed to and from these calls are handled through machine
registers. *Register 2* is used to pass return value and *Registers 4 to 7* are
used to pass arguments.

Common steps in most of the functions is reading the arguments from machine
registers, writing the return value to the machine registers and incrementing
the program counter.

**Reading Arguments**

	machine->readRegister(4)

This returns the value read in the 4th register. Here machine is an object
instance of the complete NachOS machine.

**Returning Value**

	machine->writeRegister(2, value)

This writes the *value* in the corresponding register, i.e.  register 2.

**Incrementing Program Counter**

	machine->WriteRegister(PrevPCReg, machine->ReadRegister(PCReg));
	machine->WriteRegister(PCReg, machine->ReadRegister(NextPCReg));
	machine->WriteRegister(NextPCReg, machine->ReadRegister(NextPCReg) + 4);

Since Program Counter is also a register, we can just increment the register
value by 4 to update the program counter. This step in crucial to continue
execution of the user program, otherwise the current system call will keep on
executing in an infinite loop.




## Implementation of Each System Call

Below is the individual implementation of each individual system call -

### GetPID
*Syscall_GetPID*

In order to get the PID of a thread, we need to read the *pid* variable of the
thread instance. In order to access this variable, we created a public method
*int GetPID()* which returns the value of the thread pid.

In order to assign a unique PID to a thread, we kept a global variable *maxPID*
which stores the maximum PID observed yet, and is incremented on initiation of
each new thread. Although this helps in assigning unique PIDs, however it limits
the total number of threads created in the complete running of the OS as the max
PID is in (positive) integer range, and we never free a previously used PID even
after deletion of the thread. Another approach to this could be to have a queue
of the used PIDs and assign the smallest PID that is free to a new thread during
initiation. However this method adds overhead to the thread creation and
requires more space to maintain the global queue of PIDs.

Hence we just need to write the return value from *currentThread->GetPID()* and
write it to register 2 as the return value for this syscall.

Since we need to continue execution in the current program, we increment the
program counter before returning. 

### GetPPID
*Syscall_GetPPID*

In order to get the PPID of a thread, we need to read the ppid variable of the
thread instance. In order to access this variable, we created a public method
*int GetPPID()* which returns the value of the thread ppid.

The Parent PID is obtained during thread initialization.  Since any thread other
than the main thread will only be initialized by some other thread, that thread
needs to be running. Hence the parent thread will be *currentThread* itself. In
case the *currentThread* is NULL, i.e. this thread is the main thread, we will
assign *ppid* = -1.

Hence we just need to write the return value from *currentThread->GetPPID()* and
write it to register 2 as the return value for this syscall.

Since we need to continue execution in the current program, we increment the
program counter before returning.

### Time
*Syscall_Time*

The total time is proportional to the ticks of the machine.  Although not
precise, but we can use this value as the total simulation time of the NachOS
main thread. This value is stored in the variable *totalTicks* of the class
instance *stats* of the Class *Statistics*.

Hence we just need to access the value in this variable, i.e.
*stats->totalTicks* and write it to register 2 as the return value from this
syscall.

Since we need to continue execution in the current program, we increment the
program counter before returning.

### NumInstr
*Syscall_NumInstr*

Since the number of instructions need to be different per thread, we define a
public variable named *numInstructions* in the *Thread* Class. Since for each
user instruction, the method *Interrupt::OneTick()* is called, we just increment
the *numInstructions* variable of the calling thread, i.e.  *currentThread*
whenever *OneTick()* is called. However we increment only when the machine is
not in system mode, otherwise it is not to be considered as an interrupt
generated by a user instruction.

In order to access the value, we call *currentThread->numInstructions* and write
it to register 2 as the return value from this syscall.

Since we need to continue execution in the current program, we increment the
program counter before returning.

### Sleep
*Syscall_Sleep*

In order to keep track of the all the waiting/sleeping threads, we need to keep
a queue, with all blocked threads along with their wake up time. For easier
operations, we keep this queue as sorted. This queue is declared as a global
List in *system.cc* and is defined as *waitingThreadList*.

Although putting a thread to sleep is done by a syscall, however we need to have
a method in order to restore or wake these threads up. We achieve this by having
a method which wakes up all threads whose wake up time has been passed i.e.
wakeUpTime <= totalTicks, and this method is called on every interrupt. This is
implemented in *system.cc* with the method being defined as
*restoreSleepingThreads()* called on every interrupt through the
*TimerInterruptHandler()* function in the same file.

In case the *sleepTime* for the calling thread is 0, we need not put it in the
waiting queue, however we still need to yield the cpu. Otherwise, we put the
thread in the waiting queue.  (Implemented in Threads Class) Also, we put the
thread to sleep by removing it from the readyQueue and putting it's status as
BLOCKED.

Since we need to continue execution of the user program, we need to increment
the program counter.
### Exec system call
*Syscall_Exec*
<!-- TO BE FILLED BY JASKIRAT SINGH -->
First , obtain the string of the file address that is to be executed. That has been calculated as per lines 303 to 313. Next, we create an openfile instance(if file address is not null). Pass it to the current thread, then initialize the registers and restore the context. After this, machine->run is called and then if returns with an error, an assertion of failure is shown.