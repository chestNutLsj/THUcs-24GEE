---
url: https://github.com/Tomek52/hardcore_dining_philosophers/blob/master/README.md
---

## 信号量方案
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>
#include <semaphore>

std::array<std::mutex, 5> forks;
std::counting_semaphore<4> room(4);

void think(int philosopherIndex) {
    // Simulate thinking
    std::cout << "Philosopher " << philosopherIndex << " is thinking." << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
}

void eat(int philosopherIndex) {
    // Simulate eating
    std::cout << "Philosopher " << philosopherIndex << " is eating." << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
}

void philosopher(int i) {
    while (true) {
        think(i);
        room.acquire();  // Wait for available seat in the room
        forks[i].lock();  // Wait for left fork
        forks[(i + 1) % 5].lock();  // Wait for right fork
        eat(i);
        forks[(i + 1) % 5].unlock();  // Release right fork
        forks[i].unlock();  // Release left fork
        room.release();  // Leave the room
    }
}

int main() {
    std::array<std::thread, 5> philosopherThreads;

    for (int i = 0; i < 5; ++i) {
        philosopherThreads[i] = std::thread(philosopher, i);
    }

    for (int i = 0; i < 5; ++i) {
        philosopherThreads[i].join();
    }

    return 0;
}

```

## 管程方案
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <chrono>

class DiningController {
public:
    void getForks(int pid) {
        int left = pid;
        int right = (pid + 1) % 5;

        std::unique_lock<std::mutex> lock(mutex);

        if (!fork[left])
            forkReady[left].wait(lock); // Queue on condition variable

        fork[left] = false;

        if (!fork[right])
            forkReady[right].wait(lock); // Queue on condition variable

        fork[right] = false;
    }

    void releaseForks(int pid) {
        int left = pid;
        int right = (pid + 1) % 5;

        std::unique_lock<std::mutex> lock(mutex);

        if (forkReady[left].empty()) // No one is waiting for this fork
            fork[left] = true;
        else
            forkReady[left].notify_one(); // Awaken a process waiting on this fork

        if (forkReady[right].empty()) // No one is waiting for this fork
            fork[right] = true;
        else
            forkReady[right].notify_one(); // Awaken a process waiting on this fork
    }

private:
    std::mutex mutex;
    std::array<std::condition_variable, 5> forkReady;
    std::array<bool, 5> fork = {true, true, true, true, true};
};

DiningController controller;

void philosopher(int pid) {
    while (true) {
        std::cout << "Philosopher " << pid << " is thinking." << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(100));

        controller.getForks(pid);

        std::cout << "Philosopher " << pid << " is eating." << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(100));

        controller.releaseForks(pid);
    }
}

int main() {
    std::array<std::thread, 5> philosopherThreads;

    for (int i = 0; i < 5; ++i) {
        philosopherThreads[i] = std::thread(philosopher, i);
    }

    for (int i = 0; i < 5; ++i) {
        philosopherThreads[i].join();
    }

    return 0;
}

```

## impl in C
```c
#include <pthread.h> 
#include <semaphore.h> 
#include <stdio.h> 

#define N 5 
#define THINKING 2 
#define HUNGRY 1 
#define EATING 0 
#define LEFT (phnum + 4) % N 
#define RIGHT (phnum + 1) % N 

int state[N]; 
int phil[N] = { 0, 1, 2, 3, 4 }; 

sem_t mutex; 
sem_t S[N]; 

void test(int phnum) 
{ 
	if (state[phnum] == HUNGRY 
		&& state[LEFT] != EATING 
		&& state[RIGHT] != EATING) { 
		// state that eating 
		state[phnum] = EATING; 

		sleep(2); 

		printf("Philosopher %d takes fork %d and %d\n", 
					phnum + 1, LEFT + 1, phnum + 1); 

		printf("Philosopher %d is Eating\n", phnum + 1); 

		// sem_post(&S[phnum]) has no effect 
		// during takefork 
		// used to wake up hungry philosophers 
		// during putfork 
		sem_post(&S[phnum]); 
	} 
} 

// take up chopsticks 
void take_fork(int phnum) 
{ 

	sem_wait(&mutex); 

	// state that hungry 
	state[phnum] = HUNGRY; 

	printf("Philosopher %d is Hungry\n", phnum + 1); 

	// eat if neighbours are not eating 
	test(phnum); 

	sem_post(&mutex); 

	// if unable to eat wait to be signalled 
	sem_wait(&S[phnum]); 

	sleep(1); 
} 

// put down chopsticks 
void put_fork(int phnum) 
{ 

	sem_wait(&mutex); 

	// state that thinking 
	state[phnum] = THINKING; 

	printf("Philosopher %d putting fork %d and %d down\n", 
		phnum + 1, LEFT + 1, phnum + 1); 
	printf("Philosopher %d is thinking\n", phnum + 1); 

	test(LEFT); 
	test(RIGHT); 

	sem_post(&mutex); 
} 

void* philospher(void* num) 
{ 

	while (1) { 

		int* i = num; 

		sleep(1); 

		take_fork(*i); 

		sleep(0); 

		put_fork(*i); 
	} 
} 

int main() 
{ 

	int i; 
	pthread_t thread_id[N]; 

	// initialize the semaphores 
	sem_init(&mutex, 0, 1); 

	for (i = 0; i < N; i++) 

		sem_init(&S[i], 0, 0); 

	for (i = 0; i < N; i++) { 

		// create philosopher processes 
		pthread_create(&thread_id[i], NULL, 
					philospher, &phil[i]); 

		printf("Philosopher %d is thinking\n", i + 1); 
	} 

	for (i = 0; i < N; i++) 

		pthread_join(thread_id[i], NULL); 
} 
```

## impl in Cpp atomic
### include
```hpp
// Fork.hpp
#pragma once

#include <mutex>

class Fork
{
    public:
    std::mutex mutex;
};
```

```hpp
// Philosopher.hpp
#pragma once

#include <string>
#include <thread>

#include "Table.hpp"

class Philosopher
{
    const std::string name_;
    Table const& dinnerTable_; 
    Fork& leftFork_;
    Fork& rightFork_;
    mutable std::mutex mtxLockPrint;

    public:
    Philosopher(const std::string& name, 
                Table const& dinnerTable, 
                Fork& leftFork, 
                Fork& rightFork);
    std::string getName() const;
    void print(std::string text) const;
    void think();
    void eat();
    void dine();
    void joinToTable();
};
```

```hpp
// Table.hpp
#pragma once

#include <atomic>
#include <array>

#include "Fork.hpp"

constexpr int numberOfPhilosophers = 5;

class Table
{    
    public:
    std::atomic<bool> ready{false};
    std::array<Fork, numberOfPhilosophers> forks;
};
```

### src
```cpp
// Philosopher.cpp
#include <iostream>
#include <string>

#include "Philosopher.hpp"

Philosopher::Philosopher(const std::string& name, 
                Table const& dinnerTable, 
                Fork& leftFork, 
                Fork& rightFork)
                : name_(name)
                , dinnerTable_(dinnerTable)
                , leftFork_(leftFork)
                , rightFork_(rightFork)
{}

std::string Philosopher::getName() const
{
    return name_;
}

void Philosopher::print(std::string text) const
{
    std::lock_guard<std::mutex> lg(mtxLockPrint);
    std::cout<<name_<<" "<<text<<'\n';
}

void Philosopher::think()
{
    print("started thinking");
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    print("finished thinking");
}

void Philosopher::eat()
{
    std::lock(leftFork_.mutex, rightFork_.mutex);
    std::lock_guard<std::mutex> leftLock(leftFork_.mutex, std::adopt_lock);
    std::lock_guard<std::mutex> rightLock(rightFork_.mutex, std::adopt_lock);

    std::cout<<"started eating"<<std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    std::cout<<"finished eating"<<std::endl;
}

void Philosopher::dine()
{
    while(!dinnerTable_.ready)
    do
    {
        think();
        eat();
    } while(dinnerTable_.ready);
}

void Philosopher::joinToTable()
{
    std::thread t(&Philosopher::dine, this);
    t.detach();
}
```

### test
```cpp
// PhilosopherTests.cpp
#include <gtest/gtest.h>
#include "Philosopher.hpp"
#include "Table.hpp"
#include "Fork.hpp"

struct PhilosopherTests : public ::testing::Test
{
    //GIVEN
    Table table;
    std::string philosopherName = "Platon";
    Philosopher philosopher1{philosopherName, table, table.forks[0], table.forks[1]};
};

TEST_F(PhilosopherTests, checkIfTrueIsEqTrue)
{
    ASSERT_TRUE(true);
}

TEST_F(PhilosopherTests, checkIfFunctionGetNameReturnProperValue)
{
    ASSERT_EQ(philosopher1.getName(), philosopherName);
}

TEST_F(PhilosopherTests, checkIfTrueIsEqTrue2)
{
    //GIVEN
    std::string text = "testing";
    std::string expectedStdOutput = philosopher1.getName() + " " + text + '\n';
    testing::internal::CaptureStdout();
    philosopher1.print(text);
    std::string stdOutput = testing::internal::GetCapturedStdout();
    ASSERT_EQ(expectedStdOutput, stdOutput);
}
```

### cmakelist
```cmake
#include <gtest/gtest.h>
#include "Philosopher.hpp"
#include "Table.hpp"
#include "Fork.hpp"

struct PhilosopherTests : public ::testing::Test
{
    //GIVEN
    Table table;
    std::string philosopherName = "Platon";
    Philosopher philosopher1{philosopherName, table, table.forks[0], table.forks[1]};
};

TEST_F(PhilosopherTests, checkIfTrueIsEqTrue)
{
    ASSERT_TRUE(true);
}

TEST_F(PhilosopherTests, checkIfFunctionGetNameReturnProperValue)
{
    ASSERT_EQ(philosopher1.getName(), philosopherName);
}

TEST_F(PhilosopherTests, checkIfTrueIsEqTrue2)
{
    //GIVEN
    std::string text = "testing";
    std::string expectedStdOutput = philosopher1.getName() + " " + text + '\n';
    testing::internal::CaptureStdout();
    philosopher1.print(text);
    std::string stdOutput = testing::internal::GetCapturedStdout();
    ASSERT_EQ(expectedStdOutput, stdOutput);
}
```