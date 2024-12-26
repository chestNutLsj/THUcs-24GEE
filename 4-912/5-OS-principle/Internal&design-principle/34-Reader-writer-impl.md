## 读者优先
### impl in Cpp
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>
#include <random>
#include <condition_variable>

using namespace std;

const int BUFFER_SIZE = 5; // 缓冲区大小
vector<int> buffer(BUFFER_SIZE); // 缓冲区
int readcount = 0;
mutex mtx; // 用于互斥访问缓冲区
mutex readMutex; // 用于保护读者数量
condition_variable cv; // 用于条件变量
bool writerActive = false; // 写者是否在写

void reader() {
    while (true) {
        unique_lock<mutex> lock(readMutex);
        while (writerActive) {
            cv.wait(lock);
        }
        readcount++;

        if (readcount == 1) {
            mtx.lock(); // 第一个读者需要锁定缓冲区
        }

        lock.unlock();

        // 读取数据
        int randomIndex = rand() % BUFFER_SIZE;
        int data = buffer[randomIndex];
        cout << "Reader is reading data " << data << " from buffer." << endl;

        lock.lock();
        readcount--;

        if (readcount == 0) {
            mtx.unlock(); // 最后一个读者释放缓冲区
        }

        cv.notify_all(); // 唤醒其他等待的线程
        lock.unlock();

        // 模拟读操作
        this_thread::sleep_for(chrono::milliseconds(100)); 
    }
}

void writer() {
    int data = 1;
    while (true) {
        unique_lock<mutex> lock(readMutex);
        writerActive = true;
        while (readcount > 0) {
            cv.wait(lock);
        }

        // 写入数据到缓冲区
        int emptyIndex = -1;
        for (int i = 0; i < BUFFER_SIZE; ++i) {
            if (buffer[i] == 0) {
                emptyIndex = i;
                buffer[i] = data;
                break;
            }
        }

        if (emptyIndex != -1) {
            cout << "Writer is writing data " << data << " to buffer." << endl;
            data++;
        }

        writerActive = false;
        cv.notify_all(); // 唤醒其他等待的线程
    }
}

int main() {
    thread readerThread(reader);
    thread writerThread(writer);

    readerThread.join();
    writerThread.join();

    return 0;
}


```

### impl in C and semaphore
```c
#include <pthread.h>
#include <semaphore.h>
#include <stdio.h>

/*
This program provides a possible solution for first readers writers problem using mutex and semaphore.
I have used 10 readers and 5 producers to demonstrate the solution. You can always play with these values.
*/

sem_t wrt;
pthread_mutex_t mutex;
int cnt = 1;
int numreader = 0;

void *writer(void *wno)
{   
    sem_wait(&wrt);
    cnt = cnt*2;
    printf("Writer %d modified cnt to %d\n",(*((int *)wno)),cnt);
    sem_post(&wrt);

}
void *reader(void *rno)
{   
    // Reader acquire the lock before modifying numreader
    pthread_mutex_lock(&mutex);
    numreader++;
    if(numreader == 1) {
        sem_wait(&wrt); // If this id the first reader, then it will block the writer
    }
    pthread_mutex_unlock(&mutex);
    // Reading Section
    printf("Reader %d: read cnt as %d\n",*((int *)rno),cnt);

    // Reader acquire the lock before modifying numreader
    pthread_mutex_lock(&mutex);
    numreader--;
    if(numreader == 0) {
        sem_post(&wrt); // If this is the last reader, it will wake up the writer.
    }
    pthread_mutex_unlock(&mutex);
}

int main()
{   

    pthread_t read[10],write[5];
    pthread_mutex_init(&mutex, NULL);
    sem_init(&wrt,0,1);

    int a[10] = {1,2,3,4,5,6,7,8,9,10}; //Just used for numbering the producer and consumer

    for(int i = 0; i < 10; i++) {
        pthread_create(&read[i], NULL, (void *)reader, (void *)&a[i]);
    }
    for(int i = 0; i < 5; i++) {
        pthread_create(&write[i], NULL, (void *)writer, (void *)&a[i]);
    }

    for(int i = 0; i < 10; i++) {
        pthread_join(read[i], NULL);
    }
    for(int i = 0; i < 5; i++) {
        pthread_join(write[i], NULL);
    }

    pthread_mutex_destroy(&mutex);
    sem_destroy(&wrt);

    return 0;
    
}
```
**Compiling**: `gcc reader-writer.c -pthread`

**Running**: ./a.out

## 写者优先
### 信号量实现
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <semaphore>

using namespace std;

int readcount = 0, writecount = 0;
sem_t x, y, z, wsem, rsem;

void reader() {
    while (true) {
        sem_wait(&z);
        sem_wait(&rsem);
        sem_wait(&x);
        readcount++;
        if (readcount == 1)
            sem_wait(&wsem);
        sem_post(&x);
        sem_post(&rsem);
        sem_post(&z);

        // Reading is performed
        cout << "Reader is reading..." << endl;
        this_thread::sleep_for(chrono::milliseconds(100));  // Simulate reading

        sem_wait(&x);
        readcount--;
        if (readcount == 0)
            sem_post(&wsem);
        sem_post(&x);
    }
}

void writer() {
    while (true) {
        sem_wait(&y);
        writecount++;
        if (writecount == 1)
            sem_wait(&rsem);
        sem_post(&y);

        sem_wait(&wsem);
        
        // Writing is performed
        cout << "Writer is writing..." << endl;
        this_thread::sleep_for(chrono::milliseconds(200));  // Simulate writing

        sem_post(&wsem);

        sem_wait(&y);
        writecount--;
        if (writecount == 0)
            sem_post(&rsem);
        sem_post(&y);
    }
}

int main() {
    sem_init(&x, 0, 1);
    sem_init(&y, 0, 1);
    sem_init(&z, 0, 1);
    sem_init(&wsem, 0, 1);
    sem_init(&rsem, 0, 1);

    thread readerThread(reader);
    thread writerThread(writer);

    readerThread.join();
    writerThread.join();

    sem_destroy(&x);
    sem_destroy(&y);
    sem_destroy(&z);
    sem_destroy(&wsem);
    sem_destroy(&rsem);

    return 0;
}

```

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>
#include <random>
#include <condition_variable>

using namespace std;

const int BUFFER_SIZE = 10; // 缓冲区大小
vector<char> buffer(BUFFER_SIZE); // 缓冲区
int readcount = 0;
mutex mtx; // 用于互斥访问缓冲区
mutex readMutex; // 用于保护读者数量
condition_variable cv; // 用于条件变量
bool writerActive = false; // 写者是否在写

char nextChar = '1'; // 下一个写入的字符

void reader() {
    while (true) {
        unique_lock<mutex> lock(readMutex);
        while (writerActive) {
            cv.wait(lock);
        }
        readcount++;

        if (readcount == 1) {
            mtx.lock(); // 第一个读者需要锁定缓冲区
        }

        lock.unlock();

        // 读取数据
        int randomIndex = rand() % BUFFER_SIZE;
        char data = buffer[randomIndex];
        cout << "Reader is reading data " << data << " from buffer." << endl;

        lock.lock();
        readcount--;

        if (readcount == 0) {
            mtx.unlock(); // 最后一个读者释放缓冲区
        }

        cv.notify_all(); // 唤醒其他等待的线程
        lock.unlock();

        // 模拟读操作
        this_thread::sleep_for(chrono::milliseconds(100)); 
    }
}

void writer() {
    while (true) {
        unique_lock<mutex> lock(readMutex);
        writerActive = true;
        while (readcount > 0) {
            cv.wait(lock);
        }

        // 写入数据到缓冲区
        for (int i = 0; i < BUFFER_SIZE; ++i) {
            buffer[i] = nextChar;
            if (nextChar == '9') {
                nextChar = 'A';
            } else if (nextChar == 'Z') {
                nextChar = '1';
            } else {
                nextChar++;
            }
        }

        cout << "Writer is writing data to buffer." << endl;

        writerActive = false;
        cv.notify_all(); // 唤醒其他等待的线程
    }
}

int main() {
    thread readerThread(reader);
    thread writerThread(writer);

    readerThread.join();
    writerThread.join();

    return 0;
}

```

### 消息传递实现
```cpp
#include <iostream>
#include <thread>
#include <queue>
#include <condition_variable>

using namespace std;

// Message types
enum MessageType {
    ReadRequest,
    WriteRequest,
    Finished
};

struct Message {
    MessageType type;
    int id;
};

int count = 0;
int writer_id = -1;

queue<Message> readRequests;
queue<Message> writeRequests;
queue<Message> finishedMessages;
mutex mtx;
condition_variable cv;

void reader(int i) {
    while (true) {
        Message rmsg;
        rmsg.type = ReadRequest;
        rmsg.id = i;
        {
            unique_lock<mutex> lock(mtx);
            readRequests.push(rmsg);
            cv.notify_all();  // Notify the controller
            cv.wait(lock, [&]() { return rmsg.id == -1; });  // Wait for finished message
        }
        
        // Reading is performed
        cout << "Reader " << i << " is reading..." << endl;
        this_thread::sleep_for(chrono::milliseconds(100));  // Simulate reading
        
        rmsg.type = Finished;
        rmsg.id = i;
        {
            unique_lock<mutex> lock(mtx);
            finishedMessages.push(rmsg);
            cv.notify_all();  // Notify the controller
        }
    }
}

void writer(int j) {
    while (true) {
        Message rmsg;
        rmsg.type = WriteRequest;
        rmsg.id = j;
        {
            unique_lock<mutex> lock(mtx);
            writeRequests.push(rmsg);
            cv.notify_all();  // Notify the controller
            cv.wait(lock, [&]() { return rmsg.id == -1; });  // Wait for finished message
        }
        
        // Writing is performed
        cout << "Writer " << j << " is writing..." << endl;
        this_thread::sleep_for(chrono::milliseconds(200));  // Simulate writing
        
        rmsg.type = Finished;
        rmsg.id = j;
        {
            unique_lock<mutex> lock(mtx);
            finishedMessages.push(rmsg);
            cv.notify_all();  // Notify the controller
        }
    }
}

void controller() {
    while (true) {
        unique_lock<mutex> lock(mtx);
        
        if (count > 0) {
            if (!finishedMessages.empty()) {
                Message msg = finishedMessages.front();
                finishedMessages.pop();
                count++;
                lock.unlock();
                cv.notify_all();  // Notify waiting threads
            } else if (!writeRequests.empty()) {
                Message msg = writeRequests.front();
                writeRequests.pop();
                writer_id = msg.id;
                count = count - 100;
            } else if (!readRequests.empty()) {
                Message msg = readRequests.front();
                readRequests.pop();
                count--;
                lock.unlock();
                cv.notify_all();  // Notify waiting threads
                // Simulate sending "OK" back to the reader
                cout << "Sending OK to reader " << msg.id << endl;
                this_thread::sleep_for(chrono::milliseconds(10));
                lock.lock();
            }
        }
        
        if (count == 0) {
            if (writer_id != -1) {
                cout << "Sending OK to writer " << writer_id << endl;
                writer_id = -1;
            }
            if (!finishedMessages.empty()) {
                Message msg = finishedMessages.front();
                finishedMessages.pop();
                count++;
                lock.unlock();
                cv.notify_all();  // Notify waiting threads
            } else {
                lock.unlock();
            }
        }
        
        while (count < 0) {
            Message msg = finishedMessages.front();
            finishedMessages.pop();
            count++;
            lock.unlock();
            cv.notify_all();  // Notify waiting threads
            lock.lock();
        }

        cv.wait(lock);  // Wait for signals
    }
}

int main() {
    thread controllerThread(controller);
    thread readerThreads[5];
    thread writerThreads[3];

    for (int i = 0; i < 5; ++i) {
        readerThreads[i] = thread(reader, i);
    }

    for (int j = 0; j < 3; ++j) {
        writerThreads[j] = thread(writer, j);
    }

    for (int i = 0; i < 5; ++i) {
        readerThreads[i].join();
    }

    for (int j = 0; j < 3; ++j) {
        writerThreads[j].join();
    }

    controllerThread.join();

    return 0;
}

```

