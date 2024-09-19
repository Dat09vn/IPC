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
