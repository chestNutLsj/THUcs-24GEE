## Peterson 算法实现
```cpp
#include <iostream>
#include <vector>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <chrono>
#include <random>

constexpr int BSIZE = 8; // buffer size
constexpr int PWT = 2; // producer wait time limit
constexpr int CWT = 10; // consumer wait time limit
constexpr int RT = 10; // program run-time in seconds

std::mutex mtx; // 互斥锁，用于保护共享数据的访问
std::condition_variable cv; // 条件变量，用于线程间的等待和通知

bool SHM1[2] = {false, false}; // 用于进程间的标志位共享
int SHM2 = 0; // 用于进程间的共享变量
std::vector<int> SHM3(BSIZE, 0); // 用于进程间的共享数组
int SHM4 = 1; // 用于进程间的共享变量

int myrand(int n) {
    static std::mt19937 gen(std::chrono::high_resolution_clock::now().time_since_epoch().count());
    std::uniform_int_distribution<int> dist(1, n);
    return dist(gen);
}

void producer() {
    while (true) {
        SHM1[1] = true; // 表示生产者准备好了
        std::cout << "Producer is ready now." << std::endl << std::endl;
        SHM2 = 0; // 生产者将共享变量设置为0

        std::unique_lock<std::mutex> lock(mtx); // 获取互斥锁
        cv.wait(lock, [&]{ return !SHM1[0] && SHM2 == 0; }); // 等待条件变量满足，解锁互斥锁

        // 临界区开始
        int index = 0;
        while (index < BSIZE) {
            if (SHM3[index] == 0) {
                int tempo = myrand(BSIZE * 3);
                std::cout << "Job " << tempo << " has been produced" << std::endl;
                SHM3[index] = tempo;
                break;
            }
            index++;
        }
        if (index == BSIZE) {
            std::cout << "Buffer is full, nothing can be produced!!!" << std::endl;
        }
        std::cout << "Buffer: ";
        for (int value : SHM3) {
            std::cout << value << " ";
        }
        std::cout << std::endl;
        // 临界区结束

        SHM1[1] = false; // 生产者完成
        lock.unlock(); // 解锁互斥锁
        cv.notify_one(); // 通知等待的线程

        if (SHM4 == 0) { // 如果共享变量SHM4为0，退出循环
            break;
        }

        int wait_time = myrand(PWT);
        std::cout << "Producer will wait for " << wait_time << " seconds" << std::endl << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(wait_time));
    }
}

void consumer() {
    std::this_thread::sleep_for(std::chrono::seconds(5));
    while (true) {
        SHM1[0] = true; // 表示消费者准备好了
        std::cout << "Consumer is ready now." << std::endl << std::endl;
        SHM2 = 1; // 消费者将共享变量设置为1

        std::unique_lock<std::mutex> lock(mtx); // 获取互斥锁
        cv.wait(lock, [&]{ return !SHM1[1] && SHM2 == 1; }); // 等待条件变量满足，解锁互斥锁

        // 临界区开始
        if (SHM3[0] != 0) {
            std::cout << "Job " << SHM3[0] << " has been consumed" << std::endl;
            SHM3[0] = 0;
            for (size_t i = 1; i < BSIZE; ++i) {
                SHM3[i - 1] = SHM3[i];
            }
            SHM3[BSIZE - 1] = 0;
        } else {
            std::cout << "Buffer is empty, nothing can be consumed!!!" << std::endl;
        }
        std::cout << "Buffer: ";
        for (int value : SHM3) {
            std::cout << value << " ";
        }
        std::cout << std::endl;
        // 临界区结束

        SHM1[0] = false; // 消费者完成
        lock.unlock(); // 解锁互斥锁
        cv.notify_one(); // 通知等待的线程

        if (SHM4 == 0) { // 如果共享变量SHM4为0，退出循环
            break;
        }

        int wait_time = myrand(CWT);
        std::cout << "Consumer will sleep for " << wait_time << " seconds" << std::endl << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(wait_time));
    }
}

int main() {
    std::thread producerThread(producer);
    std::thread consumerThread(consumer);

    std::this_thread::sleep_for(std::chrono::seconds(RT));
    SHM4 = 0;

    cv.notify_all(); // 通知所有等待的线程
    producerThread.join();
    consumerThread.join();

    std::cout << "The clock ran out." << std::endl;
    return 0;
}
```

![[33-生产者消费者问题-peterson.png]]

## 信号量方法的实现
```cpp
#include <iostream>  
#include <thread>  
#include <mutex>  
#include <condition_variable>  
#include <vector>

class BiSemaphore {  
public:  
    explicit BiSemaphore(int initial) : value(initial) {}  
  
    void signal() {  
        std::unique_lock<std::mutex> lock(mutex);  
        value++;  
        cv.notify_all();  
    }  
  
    void wait() {  
        std::unique_lock<std::mutex> lock(mutex);  
        cv.wait(lock, [this]() { return value > 0; });  
        value--;  
    }  
  
    void waitWithTimeout(std::chrono::milliseconds timeout) {  
        std::unique_lock<std::mutex> lock(mutex);  
        if (!cv.wait_for(lock, timeout, [this]() { return value > 0; })) {  
            // Handle timeout if needed  
        }  
        value--;  
    }  
    std::mutex mutex;  
private:  
    int value;  
  
    std::condition_variable cv;  
};  
  
int n = 0;  
BiSemaphore s(1);  
BiSemaphore delay(0);  
  
std::vector<int> buffer; // 缓冲区  
  
void produce() {  
    std::this_thread::sleep_for(std::chrono::seconds(2));  
    std::cout << "Producer is ready now." << std::endl;  
  
    // Simulate a production operation  
    int produced_data = n;  
    std::this_thread::sleep_for(std::chrono::milliseconds(100));  
  
    std::cout << "Job " << produced_data << " has been produced" << std::endl;  
  
    {  
        std::unique_lock<std::mutex> lock(s.mutex);  
        buffer.push_back(produced_data);  
        std::cout << "Buffer: ";  
        for (const int& data : buffer) {  
            std::cout << data << " ";  
        }  
        std::cout << std::endl;  
    }  
  
    s.wait();  
    n++;  
    if (n == 1) {  
        delay.signal();  
    }  
    s.signal();  
    std::this_thread::sleep_for(std::chrono::seconds(2));  
}  
  
void consume() {  
    std::this_thread::sleep_for(std::chrono::seconds(8));  
    std::cout << "Consumer is ready now." << std::endl;  
  
    int consumed_data;  
    {  
        std::unique_lock<std::mutex> lock(s.mutex);  
        if (!buffer.empty()) {  
            consumed_data = buffer.back(); // 从缓冲区取出数据  
            buffer.pop_back();  
            std::cout << "Job " << consumed_data << " has been consumed" << std::endl;  
            std::cout << "Buffer: ";  
            for (const int& data : buffer) {  
                std::cout << data << " ";  
            }  
            std::cout << std::endl;  
        }  
    }  
    s.wait();  
    n--;  
    s.signal();  
    std::this_thread::sleep_for(std::chrono::seconds(8));  
}  
  
void producer() {  
    for (int i = 0; i < 20; i++) { // 生产20个数据  
        produce();  
    }  
}  
  
void consumer() {  
    delay.wait();  
    for (int i = 0; i < 15; i++) { // 消费15个数据  
        consume();  
    }  
    // 停止代码运行  
    std::cout << "Consumer: Consumer finished." << std::endl;  
    exit(0);  
}  
  
int main() {  
    n = 0;  
    std::thread producerThread(producer);  
    std::thread consumerThread(consumer);  
  
    producerThread.join();  
    consumerThread.join();  
  
    return 0;  
}

```

## 管程方法实现
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <vector>
#include <chrono>

class BoundedBuffer {
public:
    BoundedBuffer(int bufferSize) : buffer(bufferSize), nextin(0), nextout(0), count(0) {}

    void append(char x) {
        std::unique_lock<std::mutex> lock(mutex);
        notFull.wait(lock, [this]() { return count < buffer.size(); });
        buffer[nextin] = x;
        std::cout << "Producer produced: " << x << std::endl;
        nextin = (nextin + 1) % buffer.size();
        count++;
        std::cout << "Buffer content: ";
        for (size_t i = 0; i < count; i++) {
            std::cout << buffer[(nextout + i) % buffer.size()] << " ";
        }
        std::cout << std::endl;
        notEmpty.notify_one();
    }

    void take(char &x) {
        std::unique_lock<std::mutex> lock(mutex);
        notEmpty.wait(lock, [this]() { return count > 0; });
        x = buffer[nextout];
        std::cout << "Consumer consumed: " << x << std::endl;
        nextout = (nextout + 1) % buffer.size();
        count--;
        std::cout << "Buffer content: ";
        for (size_t i = 0; i < count; i++) {
            std::cout << buffer[(nextout + i) % buffer.size()] << " ";
        }
        std::cout << std::endl;
        notFull.notify_one();
    }

private:
    std::vector<char> buffer;
    size_t nextin, nextout, count;
    std::mutex mutex;
    std::condition_variable notFull, notEmpty;
};

const int N = 10; // 缓冲区大小
BoundedBuffer boundedBuffer(N);

void produce(char &x) {
    // Simulate a production operation
    x = 'A' + rand() % 26;
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
}

void consume(char x) {
    // Simulate a consumption operation
    std::this_thread::sleep_for(std::chrono::milliseconds(150)); // 加快消费速度
}

void producer() {
    char x;
    for (int i = 0; i < 20; i++) { // 生产20个数据
        produce(x);
        boundedBuffer.append(x);
    }
}

void consumer() {
    char x;
    for (int i = 0; i < 15; i++) { // 消费15个数据
        boundedBuffer.take(x);
        consume(x);
    }
}

int main() {
    std::thread producerThread(producer);
    std::thread consumerThread(consumer);

    producerThread.join();
    consumerThread.join();

    return 0;
}

```

## 消息传递方法实现
```cpp
#include <iostream>
#include <thread>
#include <condition_variable>
#include <queue>
#include <mutex>
#include <chrono>

const int capacity = 25/* buffering capacity */;
int i;

struct Message {
    int value;
};

std::queue<Message> buffer;
std::mutex bufferMutex;
std::condition_variable mayProduce, mayConsume;

Message nullMessage;

Message produce() {
    Message msg;
    msg.value = ++i;
    return msg;
}

void producer() {
    while (i < 20) { // 生产20个数据
        Message pmsg;
        {
            std::unique_lock<std::mutex> lock(bufferMutex);
            mayProduce.wait(lock, [] { return buffer.size() < capacity; });
            pmsg = produce();
            buffer.push(pmsg);
            
            // 显示生产的数据
            std::cout << "Produced: " << pmsg.value << std::endl;
            
            // 显示缓冲区中的数据
            std::cout << "Buffer content: ";
            std::queue<Message> temp = buffer;
            while (!temp.empty()) {
                std::cout << temp.front().value << " ";
                temp.pop();
            }
            std::cout << std::endl;
        }
        mayConsume.notify_one();
        std::this_thread::sleep_for(std::chrono::milliseconds(100)); // 控制生产速度
    }
}

void consume(const Message& msg) {
    std::cout << "Consumed: " << msg.value << std::endl;
}

void consumer() {
    std::this_thread::sleep_for(std::chrono::milliseconds(2000)); // 等待2秒，确保有足够的数据供消费
    while (i < 15) { // 消费15个数据
        Message cmsg;
        {
            std::unique_lock<std::mutex> lock(bufferMutex);
            mayConsume.wait(lock, [] { return !buffer.empty(); });
            cmsg = buffer.front();
            buffer.pop();
        }
        if (cmsg.value == nullMessage.value) {
            // Break the loop if null message is received
            break;
        }
        consume(cmsg);
        mayProduce.notify_one();
        std::this_thread::sleep_for(std::chrono::milliseconds(150)); // 控制消费速度
    }
}

int main() {
    i = 0;

    std::thread producerThread(producer);
    std::this_thread::sleep_for(std::chrono::milliseconds(1000)); // 等待1秒，确保生产者有足够的时间生产数据
    std::thread consumerThread(consumer);

    // Create initial null messages in the buffer
    for (int i = 1; i <= capacity; i++) {
        buffer.push(nullMessage);
    }

    // Start producer and consumer threads
    producerThread.join();
    consumerThread.join();

    return 0;
}

```