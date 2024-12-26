
>[!tip] Prerequisite
>- [Process Synchronization](https://www.geeksforgeeks.org/process-synchronization-set-1/),
>- [Inter Process Communication](https://www.geeksforgeeks.org/inter-process-communication/)   

## What is Dekker algorithm?

To obtain such a mutual exclusion, bounded waiting, and progress there have been several algorithms implemented, one of which is **Dekker**’s Algorithm. To understand the algorithm let’s understand the solution to the critical section problem first.

A process is generally represented as:

```
do {
	// entry section
	critical section
	
	// exit section
	reminder section
} while(TRUE);
```

The solution to the critical section problem must ensure the following three conditions:

1.  Mutual Exclusion
2.  Progress
3.  Bounded Waiting

One of the solutions for ensuring above all factors is [Peterson’s solution](https://www.geeksforgeeks.org/petersons-algorithm-for-mutual-exclusion-set-1/).

Another one is **Dekker’s Solution**. Dekker’s algorithm was the first *probably-correct* solution to the critical section problem. It allows two threads to share a single-use resource without conflict, using only shared memory for communication. It avoids the strict alternation of a naive turn-taking algorithm, and was one of the first mutual exclusion algorithms to be invented.

Although there are many versions of Dekker’s Solution, the final or 5th version is the one that satisfies all of the above conditions and is the most efficient of them all.

>[! warning] Here implemetations ensure only two processes!
>**Note –** Dekker’s Solution, mentioned here, ensures mutual exclusion between two processes only, it could be extended to more than two processes with the proper use of arrays and variables.  
>**Algorithm –** It requires both an array of Boolean values and an integer variable:  

```
var flag: array [0..1] of boolean;
turn: 0..1;
repeat

        flag[i] := true;
        while flag[j] do
                if turn = j then
                begin
                        flag[i] := false;
                        while turn = j do no-op;
                        flag[i] := true;
                end;

                critical section

        turn := j;
        flag[i] := false;

                remainder section

until false;
```


## Version 1:

**First Version of Dekker’s Solution –** The idea is to use a common or shared thread number between processes and stop the other process from entering its critical section if the shared thread indicates the former one already running.

### impl in cpp
```cpp
Main()
{
    int thread_number = 1;
    startThreads();
}
 
Thread1()
{
    do {
 
        // entry section
        // wait until threadnumber is 1
        while (threadnumber == 2)
            ;
 
        // critical section
 
        // exit section
        // give access to the other thread
        threadnumber = 2;
 
        // remainder section
 
    } while (completed == false)
}
 
Thread2()
{
 
    do {
 
        // entry section
        // wait until threadnumber is 2
        while (threadnumber == 1)
            ;
 
        // critical section
 
        // exit section
        // give access to the other thread
        threadnumber = 1;
 
        // remainder section
 
    } while (completed == false)
}
```

### impl in java
```java
// Java program for the above approach
import java.lang.Thread;

public class ThreadExample {

	static boolean completed = false;
	static int threadNumber = 1;
	static void Thread1()
	{
		boolean doWhile = false;
		while (!completed || !doWhile) {
			doWhile = true;

			// entry section wait until
			// threadNumber is 1
			while (threadNumber == 2) {
				Thread.yield();
			}

			// critical section

			// exit section give access
			// to the other thread
			threadNumber = 2;

			// remainder section
		}
	}

	static void Thread2()
	{
		boolean doWhile = false;
		while (!completed || !doWhile) {
			doWhile = true;

			// entry section wait until
			// threadNumber is 2
			while (threadNumber == 1) {
				Thread.yield();
			}

			// critical section

			// exit section give access
			// to the other thread
			threadNumber = 1;

			// remainder section
		}
	}

	static void StartThreads()
	{
		Thread t1 = new Thread(ThreadExample::Thread1);
		Thread t2 = new Thread(ThreadExample::Thread2);
		t1.start();
		t2.start();
	}

	// Driver Code
	public static void main(String[] args)
	{
		threadNumber = 1;
		StartThreads();
	}
}

```

### impl in py3
```python
def Thread1():
	doWhile=False
	while not completed or not doWhile:
		doWhile=True
		# entry section
		# wait until threadnumber is 1
		while (threadnumber == 2):
			pass

		# critical section

		# exit section
		# give access to the other thread
		threadnumber = 2

		# remainder section

def Thread2():
	doWhile=False
	while not completed or not doWhile:
		doWhile=True
		# entry section
		# wait until threadnumber is 2
		while (threadnumber == 1):
			pass

		# critical section

		# exit section
		# give access to the other thread
		threadnumber = 1

		# remainder section

if __name__ == '__main__':

	thread_number = 1
	startThreads()

```

### impl in c\#
```c#
using System.Threading;

class ThreadExample {
	static bool completed = false;
	static int threadNumber = 1;

	static void Thread1()
	{
		bool doWhile = false;
		while (!completed || !doWhile) {
			doWhile = true;
			// entry section
			// wait until threadNumber is 1
			while (threadNumber == 2) {
				Thread.Yield();
			}

			// critical section

			// exit section
			// give access to the other thread
			threadNumber = 2;

			// remainder section
		}
	}

	static void Thread2()
	{
		bool doWhile = false;
		while (!completed || !doWhile) {
			doWhile = true;
			// entry section
			// wait until threadNumber is 2
			while (threadNumber == 1) {
				Thread.Yield();
			}

			// critical section

			// exit section
			// give access to the other thread
			threadNumber = 1;

			// remainder section
		}
	}

	static void StartThreads()
	{
		Thread t1 = new Thread(Thread1);
		Thread t2 = new Thread(Thread2);
		t1.Start();
		t2.Start();
	}

	static void Main(string[] args)
	{
		threadNumber = 1;
		StartThreads();
	}
}

// This code is contributed by Shivhack999

```


### impl in js
```javascript
let thread_number = 1;

function Thread1() {
	let doWhile = false;
	while (!completed || !doWhile) {
		doWhile = true;
		// entry section
		// wait until threadnumber is 1
		while (thread_number === 2) {
			// pass
		}

		// critical section

		// exit section
		// give access to the other thread
		thread_number = 2;

		// remainder section
	}
}

function Thread2() {
	let doWhile = false;
	while (!completed || !doWhile) {
		doWhile = true;
		// entry section
		// wait until threadnumber is 2
		while (thread_number === 1) {
			// pass
		}

		// critical section

		// exit section
		// give access to the other thread
		thread_number = 1;

		// remainder section
	}
}

startThreads();

```


The problem arising in the above implementation is lockstep synchronization, i.e each thread depends on the other for its execution. If one of the processes completes, then the second process runs, gives access to the completed one, and waits for its turn, however, the former process is already completed and would never run to return the access back to the latter one. Hence, the second process **waits infinitely then**.

## Version 2:

**Second Version of Dekker’s Solution –** To remove lockstep synchronization, it uses two flags to indicate its current status and updates them accordingly at the entry and exit section.

### cpp
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <atomic>

std::mutex mtx1; // Mutex for thread1's critical section
std::mutex mtx2; // Mutex for thread2's critical section
std::atomic<bool> completed(false);
std::atomic<bool> thread1(false);
std::atomic<bool> thread2(false);

void Thread1()
{
    do {
        // Entry section
        while (thread2)
            ;

        // Indicate thread1 entering its critical section
        thread1 = true;

        // Critical section
        {
            std::lock_guard<std::mutex> lock(mtx1);
            std::cout << "Thread 1 in critical section." << std::endl;
        }

        // Exit section
        // Indicate thread1 exiting its critical section
        thread1 = false;

        // Remainder section

    } while (!completed);
}

void Thread2()
{
    do {
        // Entry section
        while (thread1)
            ;

        // Indicate thread2 entering its critical section
        thread2 = true;

        // Critical section
        {
            std::lock_guard<std::mutex> lock(mtx2);
            std::cout << "Thread 2 in critical section." << std::endl;
        }

        // Exit section
        // Indicate thread2 exiting its critical section
        thread2 = false;

        // Remainder section

    } while (!completed);
}

int main()
{
    std::thread t1(Thread1);
    std::thread t2(Thread2);

    // Simulating completion of threads
    std::this_thread::sleep_for(std::chrono::seconds(2));
    completed = true;

    t1.join();
    t2.join();

    return 0;
}

```

### java
```java
public class MutualExclusion {

	// flags to indicate if each thread is in
	// its critical section or not
	boolean thread1 = false;
	boolean thread2 = false;

	// method to start the threads
	public void startThreads() {
		new Thread(new Runnable() {
			public void run() {
				thread1();
			}
		}).start();
		new Thread(new Runnable() {
			public void run() {
				thread2();
			}
		}).start();
	}

	public void thread1() {
		do {
			// entry section
			// wait until thread2 is in its critical section
			while (thread2 == true);

			// indicate thread1 entering its critical section
			thread1 = true;

			// critical section

			// exit section
			// indicate thread1 exiting its critical section
			thread1 = false;

			// remainder section
		} while (completed == false);
	}

	public void thread2() {
		do {
			// entry section
			// wait until thread1 is in its critical section
			while (thread1 == true);

			// indicate thread2 entering its critical section
			thread2 = true;

			// critical section

			// exit section
			// indicate thread2 exiting its critical section
			thread2 = false;

			// remainder section
		} while (completed == false);
	}

	public static void main(String[] args) {
		MutualExclusion me = new MutualExclusion();
		me.startThreads();
	}
}

```

### py3
```python
def Thread1():
	doWhile=False
	while not completed or not doWhile:
		doWhile=True
		# entry section
		# wait until thread2 is in its critical section
		while (thread2):
			pass

		# indicate thread1 entering its critical section
		thread1 = True

		# critical section

		# exit section
		# indicate thread1 exiting its critical section
		thread1 = False

		# remainder section

def Thread2():
	doWhile=False
	while not completed or not doWhile:
		doWhile=True
		# entry section
		# wait until thread1 is in its critical section
		while (thread1):
			pass

		# indicate thread1 entering its critical section
		thread2 = True

		# critical section

		# exit section
		# indicate thread2 exiting its critical section
		thread2 = False

		# remainder section
		
if __name__ == '__main__':

	# flags to indicate if each thread is in
	# its critical section or not.
	thread1 = False
	thread2 = False

	startThreads()

```
### c\#
```c#
using System;
using System.Threading;

class Program
{
	static bool thread1 = false;
	static bool thread2 = false;
	static bool completed = false;

	static void Main(string[] args)
	{
		// Start both threads
		Thread threadOne = new Thread(new ThreadStart(Thread1));
		Thread threadTwo = new Thread(new ThreadStart(Thread2));

		threadOne.Start();
		threadTwo.Start();
	}

	static void Thread1()
	{
		do
		{
			// entry section
			// wait until thread2 is in its critical section
			while (thread2)
			{
				// Spin-wait
			}

			// indicate thread1 entering its critical section
			thread1 = true;

			// critical section

			// exit section
			// indicate thread1 exiting its critical section
			thread1 = false;

			// remainder section
		}
		while (!completed);
	}

	static void Thread2()
	{
		do
		{
			// entry section
			// wait until thread1 is in its critical section
			while (thread1)
			{
				// Spin-wait
			}

			// indicate thread2 entering its critical section
			thread2 = true;

			// critical section

			// exit section
			// indicate thread2 exiting its critical section
			thread2 = false;

			// remainder section
		}
		while (!completed);
	}
}

```
### js
```js
let thread1InCriticalSection = false;
let thread2InCriticalSection = false;

function thread1() {
// entry section
Promise.resolve().then(() => {
	// wait until thread2 is in its critical section
	while (thread2InCriticalSection) {}

	// indicate thread1 entering its critical section
	thread1InCriticalSection = true;

	// critical section

	// exit section
	// indicate thread1 exiting its critical section
	thread1InCriticalSection = false;

	// remainder section
	thread1();
});
}

function thread2() {
// entry section
Promise.resolve().then(() => {
	// wait until thread1 is in its critical section
	while (thread1InCriticalSection) {}

	// indicate thread2 entering its critical section
	thread2InCriticalSection = true;

	// critical section

	// exit section
	// indicate thread2 exiting its critical section
	thread2InCriticalSection = false;

	// remainder section
	thread2();
});
}

thread1(); // start thread1
thread2(); // start thread2

```

The problem arising in the above version is mutual exclusion itself. If threads are preempted (stopped) during flag updation ( i.e during current_thread = true ) then, both the threads enter their critical section once the preempted thread is restarted, also the same can be observed at the start itself, when both the flags are false.

## Version 3:

**Third Version of Dekker’s Solution –** To re-ensure mutual exclusion, it sets the flags before the entry section itself.

### cpp
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <atomic>

std::mutex mtx1; // Mutex for thread1's critical section
std::mutex mtx2; // Mutex for thread2's critical section
std::atomic<bool> completed(false);
std::atomic<bool> thread1wantstoenter(false);
std::atomic<bool> thread2wantstoenter(false);

void Thread1()
{
    do {
        thread1wantstoenter = true;

        // Entry section
        while (thread2wantstoenter)
            ;

        // Critical section
        {
            std::lock_guard<std::mutex> lock(mtx1);
            std::cout << "Thread 1 in critical section." << std::endl;
        }

        // Exit section
        // Indicate thread1 has completed its critical section
        thread1wantstoenter = false;

        // Remainder section

    } while (!completed);
}

void Thread2()
{
    do {
        thread2wantstoenter = true;

        // Entry section
        while (thread1wantstoenter)
            ;

        // Critical section
        {
            std::lock_guard<std::mutex> lock(mtx2);
            std::cout << "Thread 2 in critical section." << std::endl;
        }

        // Exit section
        // Indicate thread2 has completed its critical section
        thread2wantstoenter = false;

        // Remainder section

    } while (!completed);
}

int main()
{
    std::thread t1(Thread1);
    std::thread t2(Thread2);

    // Simulating completion of threads
    std::this_thread::sleep_for(std::chrono::seconds(2));
    completed = true;

    t1.join();
    t2.join();

    return 0;
}

```
### py3
```python
if __name__=='__main__':
	# flags to indicate if each thread is in
	# queue to enter its critical section
	thread1wantstoenter = False
	thread2wantstoenter = False

	startThreads()


def Thread1():
	doWhile=False
	while (completed == False or not doWhile):
		doWhile=True
		thread1wantstoenter = True

		# entry section
		# wait until thread2 wants to enter
		# its critical section
		while (thread2wantstoenter == True):
			pass
			

		# critical section

		# exit section
		# indicate thread1 has completed
		# its critical section
		thread1wantstoenter = False

		# remainder section


def Thread2():
	doWhile=False
	while (completed == False or not doWhile) :
		doWhile=True
		thread2wantstoenter = True

		# entry section
		# wait until thread1 wants to enter
		# its critical section
		while (thread1wantstoenter == True):
			pass
			

		# critical section

		# exit section
		# indicate thread2 has completed
		# its critical section
		thread2wantstoenter = False

		# remainder section

```


The problem with this version is a deadlock possibility. Both threads could set their flag as true simultaneously and both will wait infinitely later on.

## Version 4:

**Fourth Version of Dekker’s Solution –** Uses small time interval to recheck the condition, eliminates deadlock, and ensures mutual exclusion as well.

### cpp

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <atomic>
#include <chrono>
#include <random>

std::mutex mtx1; // Mutex for thread1's critical section
std::mutex mtx2; // Mutex for thread2's critical section
std::atomic<bool> completed(false);
std::atomic<bool> thread1wantstoenter(false);
std::atomic<bool> thread2wantstoenter(false);

void randomWait()
{
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<> dis(1, 100);
    std::this_thread::sleep_for(std::chrono::milliseconds(dis(gen)));
}

void Thread1()
{
    do {
        thread1wantstoenter = true;

        while (thread2wantstoenter) {

            // Gives access to other thread
            thread1wantstoenter = false;

            randomWait();

            thread1wantstoenter = true;
        }

        // Entry section
        while (thread2wantstoenter)
            ;

        // Critical section
        {
            std::lock_guard<std::mutex> lock(mtx1);
            std::cout << "Thread 1 in critical section." << std::endl;
        }

        // Exit section
        // Indicate thread1 has completed its critical section
        thread1wantstoenter = false;

        // Remainder section

    } while (!completed);
}

void Thread2()
{
    do {
        thread2wantstoenter = true;

        while (thread1wantstoenter) {

            // Gives access to other thread
            thread2wantstoenter = false;

            randomWait();

            thread2wantstoenter = true;
        }

        // Entry section
        while (thread1wantstoenter)
            ;

        // Critical section
        {
            std::lock_guard<std::mutex> lock(mtx2);
            std::cout << "Thread 2 in critical section." << std::endl;
        }

        // Exit section
        // Indicate thread2 has completed its critical section
        thread2wantstoenter = false;

        // Remainder section

    } while (!completed);
}

int main()
{
    std::thread t1(Thread1);
    std::thread t2(Thread2);

    // Simulating completion of threads
    std::this_thread::sleep_for(std::chrono::seconds(2));
    completed = true;

    t1.join();
    t2.join();

    return 0;
}

```

### py3

```python
if __name__ == '__main__':

	# flags to indicate if each thread is in
	# queue to enter its critical section
	thread1wantstoenter = False
	thread2wantstoenter = False

	startThreads()


def Thread1():
	doWhile=False
	while (completed == False or not doWhile):
		doWhile=True
		thread1wantstoenter = True

		while (thread2wantstoenter == True) :

			# gives access to other thread
			# wait for random amount of time
			thread1wantstoenter = False

			thread1wantstoenter = True
		

		# entry section
		# wait until thread2 wants to enter
		# its critical section

		# critical section

		# exit section
		# indicate thread1 has completed
		# its critical section
		thread1wantstoenter = False

		# remainder section

	


def Thread2():
	doWhile=False
	while (completed == False or not doWhile):
		doWhile=True
		thread2wantstoenter = True

		while (thread1wantstoenter == True) :

			# gives access to other thread
			# wait for random amount of time
			thread2wantstoenter = False

			thread2wantstoenter = True
		

		# entry section
		# wait until thread1 wants to enter
		# its critical section

		# critical section

		# exit section
		# indicate thread2 has completed
		# its critical section
		thread2wantstoenter = False

		# remainder section

```

### java
```java
public class TwoThreadMutex {

	// flags to indicate if each thread wants to
	// enter its critical section
	private static boolean thread1wantstoenter = false;
	private static boolean thread2wantstoenter = false;

	public static void main(String[] args) {

		startThreads();
	}

	private static void startThreads() {
		Thread t1 = new Thread(TwoThreadMutex::Thread1);
		Thread t2 = new Thread(TwoThreadMutex::Thread2);

		t1.start();
		t2.start();

		try {
			t1.join();
			t2.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

	private static void Thread1() {

		do {

			thread1wantstoenter = true;

			while (thread2wantstoenter == true) {

				// gives access to other thread
				// wait for random amount of time
				thread1wantstoenter = false;

				thread1wantstoenter = true;
			}

			// entry section
			// wait until thread2 wants to enter
			// its critical section

			// critical section

			// exit section
			// indicate thread1 has completed
			// its critical section
			thread1wantstoenter = false;

			// remainder section

		} while (completed == false);
	}

	private static void Thread2() {

		do {

			thread2wantstoenter = true;

			while (thread1wantstoenter == true) {

				// gives access to other thread
				// wait for random amount of time
				thread2wantstoenter = false;

				thread2wantstoenter = true;
			}

			// entry section
			// wait until thread1 wants to enter
			// its critical section

			// critical section

			// exit section
			// indicate thread2 has completed
			// its critical section
			thread2wantstoenter = false;

			// remainder section

		} while (completed == false);
	}
}

```

The problem with this version is the indefinite postponement. Also, a random amount of time is erratic depending upon the situation in which the algorithm is being implemented, hence not an acceptable solution in business critical systems.

## Version Final

**Dekker’s Algorithm: Final and completed Solution –** -Idea is to use favoured thread notion to determine entry to the critical section. Favoured thread alternates between the thread providing mutual exclusion and avoiding deadlock, indefinite postponement, or lockstep synchronization.

### cpp
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <atomic>
#include <chrono>
#include <random>

std::mutex mtx; // Mutex for thread synchronization
std::condition_variable cv; // Condition variable for thread synchronization
std::atomic<bool> completed(false);
int favouredthread = 1;
bool thread1wantstoenter = false;
bool thread2wantstoenter = false;

void randomWait()
{
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<> dis(1, 100);
    std::this_thread::sleep_for(std::chrono::milliseconds(dis(gen)));
}

void Thread1()
{
    do {
        thread1wantstoenter = true;

        // Entry section
        while (thread2wantstoenter) {
            if (favouredthread == 2) {
                thread1wantstoenter = false;
                while (favouredthread == 2)
                    ;
                thread1wantstoenter = true;
            }
        }

        // Critical section
        {
            std::unique_lock<std::mutex> lock(mtx);
            while (favouredthread == 2)
                cv.wait(lock);
            std::cout << "Thread 1 in critical section." << std::endl;
        }

        // Update thread preference
        favouredthread = 2;

        // Exit section
        thread1wantstoenter = false;
        cv.notify_all();

        // Remainder section

    } while (!completed);
}

void Thread2()
{
    do {
        thread2wantstoenter = true;

        // Entry section
        while (thread1wantstoenter) {
            if (favouredthread == 1) {
                thread2wantstoenter = false;
                while (favouredthread == 1)
                    ;
                thread2wantstoenter = true;
            }
        }

        // Critical section
        {
            std::unique_lock<std::mutex> lock(mtx);
            while (favouredthread == 1)
                cv.wait(lock);
            std::cout << "Thread 2 in critical section." << std::endl;
        }

        // Update thread preference
        favouredthread = 1;

        // Exit section
        thread2wantstoenter = false;
        cv.notify_all();

        // Remainder section

    } while (!completed);
}

int main()
{
    std::thread t1(Thread1);
    std::thread t2(Thread2);

    // Simulating completion of threads
    std::this_thread::sleep_for(std::chrono::seconds(2));
    completed = true;

    cv.notify_all();

    t1.join();
    t2.join();

    return 0;
}

```

### py3
```python
if __name__ == '__main__':

	# to denote which thread will enter next
	favouredthread = 1

	# flags to indicate if each thread is in
	# queue to enter its critical section
	thread1wantstoenter = False
	thread2wantstoenter = False

	startThreads()


def Thread1():
	doWhile=False
	while (completed == False or not doWhile) :
		doWhile=True
		thread1wantstoenter = True

		# entry section
		# wait until thread2 wants to enter
		# its critical section
		while (thread2wantstoenter == True) :

			# if 2nd thread is more favored
			if (favaouredthread == 2) :

				# gives access to other thread
				thread1wantstoenter = False

				# wait until this thread is favored
				while (favouredthread == 2):
					pass
					

				thread1wantstoenter = True
			
		

		# critical section

		# favor the 2nd thread
		favouredthread = 2

		# exit section
		# indicate thread1 has completed
		# its critical section
		thread1wantstoenter = False

		# remainder section

	


def Thread2():
	doWhile=False
	while (completed == False or not doWhile) :
		doWhile=True
		thread2wantstoenter = True

		# entry section
		# wait until thread1 wants to enter
		# its critical section
		while (thread1wantstoenter == True) :

			# if 1st thread is more favored
			if (favaouredthread == 1) :

				# gives access to other thread
				thread2wantstoenter = False

				# wait until this thread is favored
				while (favouredthread == 1):
					pass
					

				thread2wantstoenter = True
			
		

		# critical section

		# favour the 1st thread
		favouredthread = 1

		# exit section
		# indicate thread2 has completed
		# its critical section
		thread2wantstoenter = False

		# remainder section

	

```

This version guarantees a complete solution to the critical solution problem.  
**References –**   
[Dekker’s Algorithm -csisdmz.ul.ie](http://garryowen.csisdmz.ul.ie/~cs4023/resources/oth9.pdf)   
[Dekker’s algorithm – Wikipedia](https://en.wikipedia.org/wiki/Dekker%27s_algorithm)
