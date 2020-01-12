---
layout: post
tags: python multithreading
title: Multithreading In Python - Introduction
---

Multithreading In Python - Introduction

## Multi Threading Overview

Every modern operating system supports Threads. When threads were introduced in earlier days, most of the machines(not including mainframes and other mini computers) had a single processor. Popular word-processors, browsers and any other GUI based program will not prevent the user interaction when such programs had to work behind the scenes on several tasks. Example, a word-processor receiving input from user at the same time writing several blocks of data onto the harddisk. In this example, input from the user will be managed by a GUI thread and writing to the disk will be handled by a writer thread.

## Multithreading on UniProcessor Systems

Remember, the CPC is just single CPU. Was the single CPU capable of executing more than one instruction or a thread at that time?
No. Those CPUs were not multicore ones they were capable of executing just one instruction at a time.
Then how multiple threads were executed in such single CPU systems? To continue further we need to define what is a thread.
A thread is a set of instructions that forms a control flow within the same program/process.
When a CPU governed by a modern operating system executes a process it does not execute all the instructions as a single stream. Instead, it executes instructions from multiple threads. When the current instruction  under execution belongs to Thread A, all other threads are not in CPU. They are under the status "Under Execution". They are in some other
status. They might be in wait state, they might be in ready state and so on.
Be aware of the fact that on a single CPU with single core only one thread can be executed at one point in time.
The time duration in which the instruction(s) belonging to a thread are inside the CPU under execution status is called time slice. Operating systems make time slices and assign to these threads based on their priority so that these threads get more time or less time.

So it is very clear from above that the threads while executed under a Single Processor System, create an illusion to the user multiple activities within a process are executed at the same time. Yes, it is just an illusion that these threads run parallel in a  Single Processor System. In reality they are executed one by one in a time sliced manner.

## Multithreading on MultiProcessor Systems

With the advent of multi core systems CPUs come with multiple cores with each one of the core being capable of executing one thread at a time. When multiple threads are run in multiple cores these threads are executed in parallel at the same time. With multi core systems true parallelism is a reality, not an illusion.

Now a quick summary of multithreading: A thread is a control of execution, sequence of instructions with in a process.
A process is a program that runs on the operating system.

Execution of a process moves forward as multiple threads belonging to different processes are executed inside the CPU.

## Multithreading in Python

A thread in a Python program is represented by the Thread Class.
In a Python program a thread can be constructed in any of the two ways below

1. By passing a callable object inside the thread constructor.
1. By overriding the run() method of the thread class.

### 1. Create a Python thread with a callable object

When creating the thread with a callable object, the function that forms the callable object, becomes the thread body. Whatever the python statements given inside that function forms a thread.

Once an object is created using a callable object the thread can be started by calling the start() method on the thread object.

e.g.,

```python
PrimeNumberThread = Thread(target=ProducePrimeNumbers)
PrimeNumberThread.start()
```

In the above example ProducePrimeNumbers is a python function i.e., a callable object passed as a target parameter of the Thread constructor.

Example: A Multithread Python program to print prime numbers

```python
from threading import Thread

#Callable function forming the thread body for the Prime Number Producer

def ProducePrimeNumbers():
    print("Prime number thread started")
    #2 is a prime number
    print(2)
    for PrimeNumberCandidate in range (2,20):
        IsPrime = False
        for divisor in range(2,PrimeNumberCandidate):
            #Skip the Composite Number
            if PrimeNumberCandidate%divisor == 0:
                IsPrime = False;
                break
            else:
            #Mark the Prime Number
                IsPrime = True;
        #Print the Prime Number
        if IsPrime == True:
            print(PrimeNumberCandidate)
    print("Prime number thread exiting")
print("Main thread started")
PrimeNumberThread = Thread(target=ProducePrimeNumbers)
#Start the prime number thread
PrimeNumberThread.start()
#Let the Main thread wait for the prime thread to complete
PrimeNumberThread.join()
print("Main thread resumed")
print("Main thread exiting")
```

### 2. Create a Python thread by overriding the run() method

```python
from threading import Thread
# A thread class to produce Fibonacci Numbers
class FibonacciThread(Thread):
    def __init__(self):
        Thread.__init__(self)
    #override the run method to define thread body
    def run(self):
        print("Fibonacci Thread Started")
        firstTerm   = 0
        secondTerm  = 1
        nextTerm    = 0
        for i in range(0,20):
            if ( i <= 1 ):
                nextTerm = i
            else:
                nextTerm    = firstTerm + secondTerm
                firstTerm   = secondTerm
                secondTerm  = nextTerm
            print(nextTerm)
        print("Fibonacci Thread Ending")
print("Main Thread Started")
MyFibonacciThread =  FibonacciThread()
MyFibonacciThread.start()
print("Main Thread Started to wait for the Fibonacci Thread to complete");
MyFibonacciThread.join()
print("Main Thread Resumed");
print("Main Thread Ending");
```

### Python Thread Methods (will be updated)
