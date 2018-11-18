# Chapter-2 Multithreaded Programming

Some core concepts in this chapter:
- Threads creation & termination, handling multiple arguments
- Difference between process and thread
- Synchronization: mutual exclusion, semaphores, deadlocks, producer-consumer problem
- Thread Safety & cancellation
- Unix Signal

## Threads Creation & Termation
### Creation: pthread_create() function
<code>
      pthread_create(&thread, 0, server, argument){}<br>
</code><br>

The parameters are:
- &thread: a pthread_t struct which contains a pthread ID(an output parameter)
- 0: some attributes declarations: e.g. the size of stacks
- server: start routine
- argument: the input parameters to the start routine
- how to pass multiple arguments in the pthread_create()? The problem is that when the current thread creates a new pthread,
the current thread continues without waiting for the new thread to start, so there is a chance when the new thread needs to refer to paramter struct defined inside the current thread scope, it is out of there and the stack is collected by the OS
- sol1: pass a pointer to local storage containing the arguments, but to avoid case mentioned above, the programmer himself need to be certain when the new thread wants to refer to structs inside current thread, the current thread's info stack is not collected by OS
- sol2: pass a pointer to static or global storage containing arguments: this works when we are certain only one thread at a time is using the storage
- sol3: pass a pointer a a dynamically allocated area which contains the arguments, this works when we are certain when the new thread terminates, it will release the area in case of memory leakage

### Termination
#### pthread_join()

<code>
      pthread_join(thread, 0)
</code>

The parameters are:
- thread is also pthread struct which indicates a thread ID, it tells the current thread which thread to wait for termination, and the second parameter, if nonzero, tells the current thread when the thread is gonna terminate, the return value will go to the loation of second parameter

#### self-termination
How does a thread terminate itself? One way is to wait until the thread itself returns a value of type (void*), another way is to wait for pthread_exit((void*)), the parameter inside is of (void*) type, e.g.
<pre><code>
  void CreatorProc() {
    pthread_t createe;
    void*      result;
    pthread_create(&createe, 0, CreateeProc, 0);
    ...
    pthread_join(createe, &result);
    ...
  }
  
  void* CreateeProc(void* arg) {
    ...
    if (should_terminate_now)
        pthread_exit((void*) 1);
    ...
    return ((void*) 2);  
  }
</code></pre>

#### Termination by Other Threads: Unix Signal & pthread_cancellation()
##### Method-1: Unix Signal
In Unix, we have a mechanism called signal to deal with hardware interrupts. e.g. When you press ctrl+c, a SIGNIT type signal is generated and passed to the current runnning process, as a result, the current process will create a handler for it, the handler has already declared with an entry function, in this way we can kill the process or do something else, e.g.
<pre><code>
  computation_state_t state;
  
  int main() {
    void handler(int);
    sigset(SIGINT, handler);
    long_running_process();
  }
  
  long_running_process() {
    while(a_long_time){
      // need do something here
      update_state(&state);
      // need do something here 
      compute_more();
    }
  }
  
  void handler(int sig) {
    display(&state);
  }
</code></pre>
The problem here is, the struct state is shared by display() and update_state(), suppose when we are in update_state() and then gets a SIGINT signal, we need to jump into handler() and then long_running_process() again, but we cannot finish the update state process, so we need a protocol to synchronize between them.<br>
Sol-1: sigpromask() method The idea is that we can maintain a set of signals, before update_state(), we block the signal, makes it unavailable to be interrupt, and then after the function, we make it available again so we can interrupt at any time.<br>
Sol-2: sigwait() method The idea is that the handler function is always listening, and the sigwait() waits for the SIGINT signal, when it hits, the handler gets the mutex and execute display, in this case we just need to lock and unlock mutex before and after display()/update_state() function.
##### Method-2: pthread cancellation by pthread_cleanup_push() / pthread_cleanup_pop()
This methods applies to the case where thread-1 wants to actively terminates thread-2. The problem is that when the thread is executing some tasks, it is hard to tell when to terminate because some functions maybe in the middle point, so we have to ensure that all the related spaces are deallocated and mutexes are unlocked (not a zombie), and something more. 
