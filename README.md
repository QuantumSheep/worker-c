# Worker library
Multithreading can be very complicated in C, POSIX and Windows multithreading functions are not the same for instance.
This library was made in order to palliate this issue and facilitate the use and creation of new threads in C.


# How to install
Just drop `worker.h` and `worker.c` in your project and include `worker.h` using a relative path.

Sometimes you will need to add `-pthread` in gcc options (only if not on Windows).

# Usage
## worker()
```c
WorkerId
worker(void *f, void *args, WorkerErr *err);
```
This function is used to start any functions as a new thread. 

### Parameters
**f**: the function that will be run as a new thread.  
**args**: arguments to be passed to the given function.  
**err**: the error code number, literally an `unsigned long`. It can be `NULL` but if not, see [`pthread_create` POSIX information page](http://pubs.opengroup.org/onlinepubs/009695399/functions/pthread_create.html) for Linux or OSX and [`GetLastError()` documentation](https://msdn.microsoft.com/fr-fr/d852e148-985c-416f-a5a7-27b6914b45d4) for Windows.

### Return value
It returns a `WorkerId` which is a `pthread_t` (`pthread.h`) on POSIX operating systems and a `HANDLE` (`windows.h`) on Windows.
The returned value can be `NULL` if their is an error.


## worker_wait()
```c
WorkerErr
worker_wait(WorkerId thread);
```
This function is used to wait for a thread to end before continuing the program.

### Parameters
**thread**: a `WorkerId`, returned by the [`worker()`](#worker) function.

### Return value
It's the error code number, literally an `unsigned long`. It can be `NULL` but if not, see [`pthread_create` POSIX information page](http://pubs.opengroup.org/onlinepubs/009695399/functions/pthread_create.html) for Linux or OSX and [`GetLastError()` documentation](https://msdn.microsoft.com/fr-fr/d852e148-985c-416f-a5a7-27b6914b45d4) for Windows.


## worker_stop()
```c
WorkerErr
worker_stop(WorkerId thread);
```
Terminate a thread. Considered as an unsafe instruction.

### Parameters
**thread**: a `WorkerId`, returned by the [`worker()`](#worker) function.

### Return value
It's the error code number, literally an `unsigned long`. It can be `NULL` but if not, see [`pthread_create` POSIX information page](http://pubs.opengroup.org/onlinepubs/009695399/functions/pthread_create.html) for Linux or OSX and [`GetLastError()` documentation](https://msdn.microsoft.com/fr-fr/d852e148-985c-416f-a5a7-27b6914b45d4) for Windows.


## Example
```c
#include "worker/worker.h"

#include <unistd.h>
#include <stdio.h>

void
timeout(int time)
{
    sleep(time);
    printf("Wake up\n");
}

void
handleError(WorkerErr err)
{
    if (err)
    {
        printf("Error code: %ld\n", err);
        _exit(0);
    }
}

int
main(int argc, char** argv)
{
    WorkerErr err;

    void* args = { 2 };
    WorkerId thread = worker(timeout, args, &err);
    handleError(err);

    printf("The thread has started!\n");

    err = worker_wait(thread);
    handleError(err);

    printf("Timeout finished.\n");
}
```

This example will print the following:
```
The thread has started!
Wake up!
Timeout finished
```
« `Wake up!` » printed 2 seconds after « `The thread has started!` » like wanted with the `sleep(2)`. And « `Timeout finished.` » directly printed after the thread's end.
