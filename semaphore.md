**What is Semaphores?**

A semaphore is a synchronization primitive used to manage access to shared resources in concurrent programming. It helps prevent race conditions by controlling how many threads or processes can access a resource simultaneously.

Semaphores typically have two main operations:
- Wait (P): Also called acquire, this operation decreases the semaphore count. If the count is greater than zero, it allows the thread/process to proceed. If it's zero, the thread/process will block until the count becomes positive again.
- Signal (V): Also called release, this operation increases the semaphore count, potentially unblocking a waiting thread.

**Key Concepts**
- Race Conditions: It occurs when some common shared resources are accessed by multiple processes or threads, leading to unpredictable behavior.
- Deadlocks: a situation occurs when two or more processes are holding the resources and do not release them causing a deadlock to occur because another is waiting for that resources. Deadlock prevention and detection mechanisms are critical in process synchronization.

**Synchronization Techniques Include:**
- Mutex(Mutual Exclusion): only one process can access the critical section at a time, it is a locking mechanism.
- Semaphores: Signaling mechanisms to control access to shared resources.

**Types of Semaphores**
- Binary Semaphore: This is also known as a mutex lock. It can have only two values – 0 and 1. Its value is initialized to 1. It is used to implement the solution of critical section problems with multiple processes.
- Counting Semaphore: Its value can range over an unrestricted domain. It is used to control access to a resource that has multiple instances.

**Example semaphore handle multithread:**

Example code binary semaphore:
```c++
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

sem_t semaphore;

void* thread_func(void* arg) {
    printf("Thread %d waiting to enter critical section...\n", *(int*)arg);
    sem_wait(&semaphore); // Lock semaphore
    printf("Thread %d in critical section\n", *(int*)arg);
    sleep(2);  // Simulate some work
    printf("Thread %d leaving critical section\n", *(int*)arg);
    sem_post(&semaphore); // Unlock semaphore
    return NULL;
}

int main() {
    pthread_t t1, t2;
    int t1_id = 1, t2_id = 2;

    // Initialize semaphore to 1 (binary semaphore)
    sem_init(&semaphore, 0, 1);

    // Create two threads
    pthread_create(&t1, NULL, thread_func, &t1_id);
    pthread_create(&t2, NULL, thread_func, &t2_id);

    // Wait for both threads to finish
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    // Destroy the semaphore
    sem_destroy(&semaphore);

    return 0;
}
```
Example code for counting semaphore:
```C++
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

#define NUM_THREADS 8   // Total number of threads
#define MAX_RESOURCES 3 // Number of threads allowed in the critical section at once

sem_t semaphore;  // Counting semaphore

void* thread_func(void* arg) {
    int thread_id = *(int*)arg;
    
    printf("Thread %d waiting to enter the critical section...\n", thread_id);
    
    // Try to enter critical section
    sem_wait(&semaphore);
    printf("Thread %d entered the critical section.\n", thread_id);

    // Simulate some work inside the critical section
    sleep(2);

    // Exit the critical section
    printf("Thread %d leaving the critical section.\n", thread_id);
    sem_post(&semaphore);

    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];
    int thread_ids[NUM_THREADS];

    // Initialize the semaphore with MAX_RESOURCES
    sem_init(&semaphore, 0, MAX_RESOURCES);

    // Create multiple threads
    for (int i = 0; i < NUM_THREADS; i++) {
        thread_ids[i] = i + 1;
        pthread_create(&threads[i], NULL, thread_func, &thread_ids[i]);
    }

    // Wait for all threads to finish
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    // Destroy the semaphore
    sem_destroy(&semaphore);

    return 0;
}
```
In this case,  3 threads can enter the critical section at the same time.

Only 3 threads can access a connection at the same time, while the other 2 must wait.

As soon as one thread finishes and releases the connection, one of the waiting threads can proceed.

**Real-World Analogy:**

Think of a counting semaphore like a group of parking spaces (e.g., 3 spaces). When a car (thread) arrives.

**Different between semaphore and mutex:**

![image](https://github.com/user-attachments/assets/fd6a3b29-149c-49e0-ae27-fb3ed59bb6d9)

**Using semaphore when 2 process try to write in shared memory at the same time.**

**How Semaphores Work in Multiprocessing**

In a multiprocessing context, semaphores help coordinate access to shared memory or other resources between processes.

If you're using semaphores for multiprocessing (not threads), **you need to use sem_open, sem_wait, and sem_post for named semaphores**, which can be shared between processes.

Named semaphores (sem_open, sem_close, sem_unlink) are used because they allow sharing between multiple processes, whereas unnamed semaphores are typically used for threads within a single process.

```c++
// process1.c
#include <stdio.h>
#include <stdlib.h>
#include <semaphore.h>
#include <fcntl.h>  // For O_CREAT, O_EXCL
#include <unistd.h> // For sleep

#define SEMAPHORE_NAME "/example_named_semaphore"

int main() {
    // Open or create a named semaphore with an initial value of 2
    sem_t *semaphore = sem_open(SEMAPHORE_NAME, O_CREAT, 0644, 2);
    if (semaphore == SEM_FAILED) {
        perror("sem_open failed");
        exit(EXIT_FAILURE);
    }

    // Wait (decrement) the semaphore
    sem_wait(semaphore);
    printf("Process 1 is in the critical section.\n");
    sleep(2);  // Simulate some work
    printf("Process 1 is leaving the critical section.\n");

    // Signal (increment) the semaphore
    sem_post(semaphore);

    // Close the semaphore
    sem_close(semaphore);

    return 0;
}
```
```c++
// process2.c
#include <stdio.h>
#include <stdlib.h>
#include <semaphore.h>
#include <fcntl.h>  // For O_CREAT, O_EXCL
#include <unistd.h> // For sleep

#define SEMAPHORE_NAME "/example_named_semaphore"

int main() {
    // Open the named semaphore (created by Process 1)
    sem_t *semaphore = sem_open(SEMAPHORE_NAME, 0);
    if (semaphore == SEM_FAILED) {
        perror("sem_open failed");
        exit(EXIT_FAILURE);
    }

    // Wait (decrement) the semaphore
    sem_wait(semaphore);
    printf("Process 2 is in the critical section.\n");
    sleep(2);  // Simulate some work
    printf("Process 2 is leaving the critical section.\n");

    // Signal (increment) the semaphore
    sem_post(semaphore);

    // Close the semaphore
    sem_close(semaphore);

    return 0;
}
```
sem_open(SEMAPHORE_NAME, O_CREAT, 0644, 2); creates a named semaphore called /example_named_semaphore. The initial value is set to 2, allowing two processes to enter the critical section at the same time.

Process 2 uses sem_open(SEMAPHORE_NAME, 0); to open the same named semaphore without creating it (since Process 1 already created it).

**Approach 2:**
Using sem_init for Multiprocessing: For multiprocessing, if you want to share a semaphore between processes, you need to place the semaphore in shared memory (using pshared = 1).

sem_init(&semaphore, 0, 2) is typically used for thread synchronization within a process.

For multiprocessing, you need to initialize the semaphore with pshared = 1 and use shared memory (e.g., mmap) to allow processes to share the semaphore.

```c++
#include <stdio.h>
#include <stdlib.h>
#include <semaphore.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/mman.h>  // For mmap (shared memory)

void critical_section(int process_num) {
    printf("Process %d is in the critical section.\n", process_num);
    sleep(2);  // Simulate some work
    printf("Process %d is leaving the critical section.\n", process_num);
}

int main() {
    // Create shared memory for the semaphore using mmap
    sem_t *semaphore = mmap(NULL, sizeof(sem_t), PROT_READ | PROT_WRITE, MAP_SHARED | MAP_ANONYMOUS, -1, 0);
    if (semaphore == MAP_FAILED) {
        perror("mmap failed");
        exit(EXIT_FAILURE);
    }

    // Initialize the semaphore with a value of 2 (allow 2 processes in the critical section at once)
    if (sem_init(semaphore, 1, 2) != 0) {
        perror("sem_init failed");
        exit(EXIT_FAILURE);
    }

    pid_t pid1 = fork();
    if (pid1 == 0) {  // Child process 1
        sem_wait(semaphore);  // Wait (decrement) the semaphore
        critical_section(1);
        sem_post(semaphore);  // Signal (increment) the semaphore
        exit(0);
    }

    pid_t pid2 = fork();
    if (pid2 == 0) {  // Child process 2
        sem_wait(semaphore);  // Wait (decrement) the semaphore
        critical_section(2);
        sem_post(semaphore);  // Signal (increment) the semaphore
        exit(0);
    }

    pid_t pid3 = fork();
    if (pid3 == 0) {  // Child process 3
        sem_wait(semaphore);  // Wait (decrement) the semaphore
        critical_section(3);
        sem_post(semaphore);  // Signal (increment) the semaphore
        exit(0);
    }

    // Parent process waits for all child processes to complete
    wait(NULL);
    wait(NULL);
    wait(NULL);

    // Clean up: Destroy the semaphore
    sem_destroy(semaphore);

    // Unmap the shared memory
    munmap(semaphore, sizeof(sem_t));

    return 0;
}
```
