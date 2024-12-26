
## C++20 协程
------------

C++ 20 有一个新玩意，**协程**。这玩意对 C++ 的未来可能是重要一环，也会是让 C++ 成为服务器编程有力工具。

对 C++20 的协程，最简单的理解协程是**可以重入的特殊函数**。就是这个函数在执行的过程，可以（通过 `co_await` , 或者 `co_yield`）挂起，然后在外部（通过 `coroutine_handle`）恢复运行。

我测试的代码都是在 `Visual studio 2022` 上运行的。据说 `GCC 10.0` 也已经支持。

## 协程是特殊的函数
------------

首先再次强调，C++ 20 的协程是一个**特殊函数**。只是这个函数具有**挂起**和**恢复**的能力，可以被挂起（挂起后调用代码继续向后执行），而后可以继续恢复其执行。如下图：

![[coroutine-flow.png]]
<center>Fig 1. Cpp coroutine flow chart</center>

如图所示，协程并没有一次执行完成，可以被反复挂起，挂起后可以恢复到挂起的点继续运行。

## C++ 20 协程的特点
----------------

那我们来看看 C++ 20 协程的一些特点和用途。

首先，C++ 20 协程是一个**无栈（stackless）的协程**。同时，C++ 20 协程是非对称的协程，和 Linux 传统的 Context Switch 有区别。更像 Windows 的纤程。和 C# 的协程也比较相像，毕竟是微软的提案。

> 传统的 Context Switch 是有栈协程，你可以认为 Context 协程都是运行在栈上，Context 协程的切换就是切换栈。同时因为其是有栈协程。切换是对称的，都是栈切换。你可以从主线程上切换为另外一个 Context 协程栈，也可以从一个 Context 协程切换为主线程，也可以 Context 协程之间切换。Context 协程的状态也就是保存在栈上。

C++ 20 的协程可以用来干啥呢？和大部分协程用途类似，就是**异步编程**用的。看图 1 就可以明白，每次一次协程的挂起都可以视为协程进入一个等待状态，比如请求一个网络，需要 HTTP get 一个文件，然后对文件进行分析。那么就可以用协程来包装整个处理，在发起 HTTP 请求后，挂起协程（处理其他事情），等待应答或者超时后，再恢复协程的运行。

但不足的是目前 C++ 20 的协程才是一个开始，说实话，目前的协程只提供基本框架，写起来并不舒服。C++ 目前在 IO 方面，特别是网络 IO 方面还不完善。需要一个大量异步 IO 库，才能用好 C++ 20 协程。

如果 C++ 20 的协程周边更加完整，也许 C++ 又能在服务器编程这块能重新面对 Go 这类语言的威胁。

## C++ 协程的是三个关键字
-----------------

C++ 的协程（协程函数）内部可以用 **co_await** , **co_yield**. 两个关键字挂起协程，**co_return**, 关键字进行返回。

### co_await

`co_await` 调用一个 awaiter 对象（可以认为是一个接口），根据其内部定义决定其操作是挂起，还是继续，以及挂起，恢复时的行为。其呈现形式为

```
cw_ret = co_await  awaiter;
```

cw_ret 记录调用的返回值，其是 awaiter 的 await_resume 接口返回值。

`co_await` 相对比较复杂，后面开一章详细讲。

### co_yield

挂起协程。其出现形式是

```
co_yield  cy_ret;
```

cy_ret 会保存在 promise 承诺对象中（通过 `yield_value` 函数）。在协程外部可以通过 promise 得到。

### co_return

协程返回。其出现形式是

```
co_return cr_ret;
```

cr_ret 会保存在 promise 承诺对象中（通过 `return_value` 函数）。在协程外部可以通过 promise 得到。要注意，cr_ret 并不是协程的返回值。这个是有区别的。

## C ++ 协程的重要概念
---------------

C++ 的编译器如何识别协程函数呢？是通过函数返回值。C++ 协程函数的返回类型有要求，返回类型是`result` ，而`result`里面必须有一个子类型**承诺对象**（promise），呈现为 Result:: promise_type。承诺对象（promise）是一个接口，里面实现 get_return_object 等接口。而通过 std::coroutine_handle<promise_type>:: from_promise ( promise& p ) 这个静态函数，我们可以得到**协程句柄**（coroutine handle）。而协程的运行状态 ，协程函数的形参，内部变量，临时变量，挂起暂停在什么点，被保存在**协程状态** (coroutine state) 中。

好了上面的描述，我们可以看出协程的几个重要概念。

* **协程状态** (`coroutine state`)，记录协程状态，是分配于堆的内部对象：
	- 承诺对象
	- 形参（协程函数的参数）
	* 协程挂起的点
	* 临时变量

* **承诺对象**（`promise`），从协程内部操纵。协程通过此对象提交其结果或异常。
* **协程句柄**（`coroutine handle`），协程的唯一标示。用于恢复协程执行或销毁协程帧。
* **等待体**（`awaiter`），co_await 关键字调用的对象。

### 协程状态（coroutine state）

协程状态（`coroutine state`）是协程启动开始时，new 空间存放协程状态，协程状态记录协程函数的参数，协程的运行状态，变量。挂起时的断点。

注意，协程状态 (`coroutine state`) 并不是就是协程函数的返回值 RET。虽然我们设计的 RET 一般里面也有`promise`和`coroutine handle`，大家一般也是通过 RET 去操作协程的恢复，获取返回值。但`coroutine state`理论上还应该包含协程运行参数，断点等信息。而**协程状态** (`coroutine state`) 应该是协程句柄（`coroutine handle`）对应的一个数据，而由系统管理的。

### 承诺对象（promise）

承诺对象的表现形式必须是`result::promise_type`，`result`为协程函数的返回值。

承诺对象是一个实现若干接口，用于辅助协程，构造协程函数返回值；提交传递`co_yield`，`co_return`的返回值。明确协程启动阶段是否立即挂起；以及协程内部发生异常时的处理方式。其接口包括：

*   `auto get_return_object ()` ：用于生成协程函数的返回对象。
*   `auto initial_suspend ()`：用于明确初始化后，协程函数的执行行为，返回值为等待体（`awaiter`），用`co_wait`调用其返回值。返回值为`std::suspend_always` 表示协程启动后立即挂起（不执行第一行协程函数的代码），返回`std::suspend_never` 表示协程启动后不立即挂起。（当然既然是返回等待体，你可以自己在这儿选择进行什么等待操作）
*   `void return_value (T v)`：调用`co_return v`后会调用这个函数，可以保存`co_return`的结果
*   `auto yield_value (T v)`：调用`co_yield`后会调用这个函数，可以保存`co_yield`的结果，其返回其返回值为`std::suspend_always`表示协程会挂起，如果返回`std::suspend_never`表示不挂起。
*   `auto final_suspend () noexcept`：在协程退出是调用的接口，返回`std::suspend_never` ，自动销毁 coroutine state 对象。若 `final_suspend` 返回 `std::suspend_always` 则需要用户自行调用 `handle.destroy ()` 进行销毁。但值得注意的是返回 std:: suspend_always 并不会挂起协程。

前面我们提到在协程创建的时候，会 new 协程状态（`coroutine state`）。你可以通过可以在 `promise_type` 中重载 `operator new` 和 `operator delete`，使用自己的内存分配接口。

### 协程句柄（coroutine handle）

协程句柄（`coroutine handle`）是一个协程的标示，用于操作协程恢复，销毁的句柄。

协程句柄的表现形式是`std::coroutine_handle<promise_type>`，其模板参数为承诺对象（`promise`）类型。句柄有几个重要函数：

*   `resume ()`函数可以恢复协程。
*   `done ()`函数可以判断协程是否已经完成。返回 false 标示协程还没有完成，还在挂起。

协程句柄和承诺对象之间是可以相互转化的。

*   `std::coroutine_handle<promise_type>::from_promise` ： 这是一个静态函数，可以从承诺对象（promise）得到相应句柄。
*   `std::coroutine_handle<promise_type>:: promise ()` 函数可以从协程句柄`coroutine handle`得到对应的承诺对象（`promise`）

### 等待体（awaiter）

co_wait 关键字会调用一个等待体对象 (`awaiter`)。这个对象内部也有 3 个接口。根据接口 co_wait 决定进行什么操作。

*   `bool await_ready ()`：等待体是否准备好了，返回 false ，表示协程没有准备好，立即调用 await_suspend。返回 true，表示已经准备好了。
*   `auto await_suspend (std::coroutine_handle<> handle)`如果要挂起，调用的接口。其中 handle 参数就是调用等待体的协程，其返回值有 3 种可能

*   void 同返回 true
*   bool 返回 true 立即挂起，返回 false 不挂起。
*   返回某个协程句柄（`coroutine handle`），立即恢复对应句柄的运行。

*   `auto await_resume ()` ：协程挂起后恢复时，调用的接口。返回值作为 co_wait 操作的返回值。

**等待体**（awaiter）值得用更加详细的笔墨书写一章，我们就放一下，先了解其有 2 个特化类型。

*   `std::suspend_never`类，不挂起的的特化等待体类型。
*   `std::suspend_always`类，挂起的特化等待体类型。

前面不少接口已经用了这 2 个特化的类，同时也可以明白其实协程内部不少地方其实也在使用`co_wait` 关键字。

## 例子 ，“七进七出” 的协程。
------------------

好了。所有概念我们介绍基本完成了。先来段代码吧。否则实在憋屈。

这个例子主要展现的是协程函数和主线程之间的切换。协程反复中断，然后在 main 函数内部又恢复其运行。直至最后 co_return。

这个例子虽然简单，但如果你对异步编程有所了解也能明白如何利用 C++20 完成一段异步编程了。源代码获取地址请点击

下面例子中：

*   `coro_ret<int> coroutine_7 in 7 out ()` 就是协程函数。
*   `coro_ret<int> c_r` 就是协程的返回值。在后续，都是通过 c_r 和协程进行交互。
*   `coro_ret<int>::promise_type` 就是承诺对象
*   `std::coroutine_handle<promise_type>` 就是句柄。

```
#include <coroutine>
#include <iostream>
#include <stdexcept>
#include <thread>


//! coro_ret 协程函数的返回值，内部定义 promise_type，承诺对象
template <typename T>
struct coro_ret
{
   struct promise_type;
   using handle_type = std::coroutine_handle<promise_type>;
   //! 协程句柄
   handle_type coro_handle_;

   coro_ret (handle_type h)
      : coro_handle_(h)
  {
  }
   coro_ret (const coro_ret&) = delete;
   coro_ret (coro_ret&& s)
      : coro_handle_(s.coro_)
  {
       s.coro_handle_ = nullptr;
  }
   ~coro_ret ()
  {
       //! 自行销毁
       if (coro_handle_)
           coro_handle_. destroy ();
  }
   coro_ret& operator=(const coro_ret&) = delete;
   coro_ret& operator=(coro_ret&& s)
  {
       coro_handle_ = s.coro_handle_;
       s.coro_handle_ = nullptr;
       return *this;
  }

   //! 恢复协程，返回是否结束
   bool move_next ()
  {
       coro_handle_. resume ();
       return coro_handle_. done ();
  }
   //! 通过 promise 获取数据，返回值
   T get ()
  {
       return coro_handle_. promise (). return_data_;
  }
   //! promise_type 就是承诺对象，承诺对象用于协程内外交流
   struct promise_type
  {
       promise_type () = default;
       ~promise_type () = default;

       //! 生成协程返回值
       auto get_return_object ()
      {
           return coro_ret<T>{handle_type:: from_promise (*this)};
      }

       //! 注意这个函数, 返回的就是 awaiter
       //! 如果返回 std:: suspend_never{}，就不挂起，
       //! 返回 std:: suspend_always{} 挂起
       //! 当然你也可以返回其他 awaiter
       auto initial_suspend ()
      {
           //return std:: suspend_never{};
           return std:: suspend_always{};
      }
       //! co_return 后这个函数会被调用
       void return_value (T v)
      {
           return_data_ = v;
           return;
      }
       //!
       auto yield_value (T v)
      {
           std:: cout << "yield_value invoked." << std:: endl;
           return_data_ = v;
           return std:: suspend_always{};
      }
       //! 在协程最后退出后调用的接口。
       //! 若 final_suspend 返回 std:: suspend_always 则需要用户自行调用
       //! handle.destroy () 进行销毁，但注意 final_suspend 被调用时协程已经结束
       //! 返回 std:: suspend_always 并不会挂起协程（实测 VSC++ 2022）
       auto final_suspend () noexcept
      {
           std:: cout << "final_suspend invoked." << std:: endl;
           return std:: suspend_always{};
      }
       //
       void unhandled_exception ()
      {
           std:: exit (1);
      }
       //返回值
       T return_data_;
  };
};


//这就是一个协程函数
coro_ret<int> coroutine_7 in 7 out ()
{
   //进入协程看 initial_suspend，返回 std:: suspend_always{}; 会有一次挂起

   std:: cout << "Coroutine co_await std:: suspend_never" << std:: endl;
   //co_await std:: suspend_never{} 不会挂起
   co_await std:: suspend_never{};
   std:: cout << "Coroutine co_await std:: suspend_always" << std:: endl;
   co_await std:: suspend_always{};

   std:: cout << "Coroutine stage 1 ,co_yield" << std:: endl;
   co_yield 101;
   std:: cout << "Coroutine stage 2 ,co_yield" << std:: endl;
   co_yield 202;
   std:: cout << "Coroutine stage 3 ,co_yield" << std:: endl;
   co_yield 303;
   std:: cout << "Coroutine stage end, co_return" << std:: endl;
   co_return 808;
}

int main (int argc, char* argv[])
{
   bool done = false;
   std:: cout << "Start coroutine_7 in 7 out ()\n";
   //调用协程, 得到返回值 c_r，后面使用这个返回值来管理协程。
   auto c_r = coroutine_7 in 7 out ();
   //第一次停止因为 initial_suspend 返回的是 suspend_always
   //此时没有进入 Stage 1
   std:: cout << "Coroutine " << (done ? "is done " : "isn't done ")
       << "ret =" << c_r.get () << std:: endl;
   done = c_r.move_next ();
   //此时是，co_await std:: suspend_always{}
   std:: cout << "Coroutine " << (done ? "is done " : "isn't done ")
       << "ret =" << c_r.get () << std:: endl;
   done = c_r.move_next ();
   //此时打印 Stage 1
   std:: cout << "Coroutine " << (done ? "is done " : "isn't done ")
       << "ret =" << c_r.get () << std:: endl;
   done = c_r.move_next ();
   std:: cout << "Coroutine " << (done ? "is done " : "isn't done ")
       << "ret =" << c_r.get () << std:: endl;
   done = c_r.move_next ();
   std:: cout << "Coroutine " << (done ? "is done " : "isn't done ")
       << "ret =" << c_r.get () << std:: endl;
   done = c_r.move_next ();
   std:: cout << "Coroutine " << (done ? "is done " : "isn't done ")
       << "ret =" << c_r.get () << std:: endl;
   return 0;
}
```

### co_await awaiter 的用途？
-------------------------

明确说 C++20 的协程大部分概念还算清晰，就是 yeild，然后外部利用句柄`resume`。对协程这个概念有了解的不应该有什么特别难以理解的地方。

但`co_await awaiter`比较让人疑惑。

上次我已经讲过，`awaiter`其实是是一个对象，一个接口实现，其 3 个接口函数是（详细解释请翻阅第一章）：

*   `await_ready`：等待体是否准备好了，没准备好（`return false`）就调用`await_suspend`
*   `await_suspend`：等待体挂起如何操作。参数为调用其的协程句柄。`return true` ，或者 `return void` 就会挂起协程。
*   `await_resume`：协程挂起后恢复时，调用的接口，同时返回其结果，作为`co_await`的返回值。

不少代码的例子都是在 await_suspend 函数中，直接把 handle.resume ()，就是说这些例子都是在挂起时就理解恢复了协程运行，这样的例子貌似什么异步的感觉都没有，没有体现任何异步操作的效果和优势。

这样`co_await awaiter`能用来干啥就有点让我好奇了。我的直觉是等待体`awaiter`在`await_suspend`应该就是记录协程句柄，同时发起一个异步操作（比如用一个线程完成文件读写），然后在异步操作完成后，恢复协程的运行，告知协程读写的结果。

`co_await awaiter`的在未来应该会有很多种等待体，比如 AIO，异步网络，异步读写数据库等。这也应该是未来 C++ 协程重点反正发展地方。

### await_suspend 的参数
---------------------

这个问题先提前说一下，我曾经疑惑过。`await_suspend`接口的参数，其是调用其的外部协程的句柄。

```
void await_suspend (std::coroutine_handle<result::promise_type> awaiting)
```

但让我疑惑的是 `std::coroutine_handle<>` 里面模板参数理论应该是协程`promise_type`承诺对象。不知道您理解这儿的麻烦没有，如果你要写一个通用的 awaiter，那么难道都要使用模板？让使用者填写其协程对应的`promise_type`。这样开发者，使用者都麻烦。

后面我发现，如果只要你不使用对应的承诺对象，`std::coroutine_handle<promise_type>:: promise ()` 。参数类型写成`std::coroutine_handle<>`也没有问题（<> 中为空，默认为 void）。这样也可以适配各种协程。

### co_await 的呈现形式
------------------

co_await 可以呈现出不少形式，如果你才开始学你会比较疑惑。

```
co_ret = co_await  awaiter;
```

co_await 调用 awaiter 的接口。co_ret 是从 awaiter 里面的 await_resume 接口的返回值。

```
co_ret = co_await  fun ();
```

fun () 函数返回值是 awaiter 对象，co_ret 是从 awaiter 里面的 await_resume 接口的返回值。

## 例子 ：尝试异步 IO（有缺陷）
-------------------

我们尝试一些一个异步的读取文件的操作，封装在 awaiter 对象`await_read_file`里面，在其`await_suspend`接口中，我们尝试使用`std::async`发起了一个异步操作。然后等待返回结果。

协程返回值仍然是 `coro_ret<T>`, 承诺对象还是`coro_ret<T>::promise_type`，这个地方和前面的例子几乎没有差别，只是`initial_suspend`返回的`std:: suspend_never{}`，表示协程在初始化后（刚刚进入时）不进行挂起操作。源代码地址请点击。

```
#include <coroutine>
#include <iostream>
#include <stdexcept>
#include <thread>
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <future>
#include <chrono>
#include <thread>

//! coro_ret 协程函数的返回值，内部定义 promise_type，承诺对象
template <typename T>
struct coro_ret
{
   struct promise_type;
   using handle_type = std::coroutine_handle<promise_type>;
   //! 协程句柄
   handle_type coro_handle_;

   coro_ret (handle_type h)
      : coro_handle_(h)
  {
  }
   coro_ret (const coro_ret&) = delete;
   coro_ret (coro_ret&& s)
      : coro_handle_(s.coro_)
  {
       s.coro_handle_ = nullptr;
  }
   ~coro_ret ()
  {
       //! 自行销毁
       if (coro_handle_)
           coro_handle_. destroy ();
  }
   coro_ret& operator=(const coro_ret&) = delete;
   coro_ret& operator=(coro_ret&& s)
  {
       coro_handle_ = s.coro_handle_;
       s.coro_handle_ = nullptr;
       return *this;
  }

   //! 恢复协程，返回是否结束
   bool move_next ()
  {
       coro_handle_. resume ();
       return coro_handle_. done ();
  }
   //! 通过 promise 获取数据，返回值
   T get ()
  {
       return coro_handle_. promise (). return_data_;
  }
   //! promise_type 就是承诺对象，承诺对象用于协程内外交流
   struct promise_type
  {
       promise_type () = default;
       ~promise_type () = default;

       //! 生成协程返回值
       auto get_return_object ()
      {
           return coro_ret<T>{handle_type:: from_promise (*this)};
      }

       //! 注意这个函数, 返回的就是 awaiter
       //! 如果返回 std:: suspend_never{}，就不挂起，
       //! 返回 std:: suspend_always{} 挂起
       //! 当然你也可以返回其他 awaiter
       auto initial_suspend ()
      {
           return std:: suspend_never{};
           //return std:: suspend_always{};
      }
       //! co_return 后这个函数会被调用
       void return_value (T v)
      {
           return_data_ = v;
           return;
      }
       //!
       auto yield_value (T v)
      {
           std:: cout << "yield_value invoked." << std:: endl;
           return_data_ = v;
           return std:: suspend_always{};
      }
       //! 在协程最后退出后调用的接口。
       //! 若 final_suspend 返回 std:: suspend_always 则需要用户自行调用
       //! handle.destroy () 进行销毁，但注意 final_suspend 被调用时协程已经结束
       //! 返回 std:: suspend_always 并不会挂起协程（实测 VSC++ 2022）
       auto final_suspend () noexcept
      {
           std:: cout << "final_suspend invoked." << std:: endl;
           return std:: suspend_always{};
      }
       //
       void unhandled_exception ()
      {
           std:: exit (1);
      }
       //返回值
       T return_data_;
  };
};

int read_file (const char* filename,
             char* buffer,
             size_t buf_len,
             size_t* read_len,
             std::coroutine_handle<> coro_hdl)
{
   int result = 0;
   size_t len = 0;
   *read_len = 0;
   //打开文件
   FILE* fd = :: fopen (filename, "r+");
   if (nullptr == fd)
  {
       result = -1;
       goto READ_FILE_END;
  }
   //读取内容
   len = :: fread (buffer, 1, buf_len, fd);
  :: fclose (fd);
   if (len <= 0)
  {
       result = -1;
       goto READ_FILE_END;
  }

   *read_len = len;
   result = 0;

   //到了最后一步，这儿用 goto 只是方便写代码。
READ_FILE_END:

   return result;
}


struct await_read_file
{
   await_read_file (const char* filename,
                   char* buffer,
                   size_t buf_len,
                   size_t* read_len)
  {
       filename_ = filename;
       buffer_ = buffer;
       buf_len_ = buf_len;
       read_len_ = read_len;
  };
   ~await_read_file () = default;

   bool await_ready ()
  {
       return false;
  }
   //挂起的操作，发起异步读文件操作，然后等待返回
   void await_suspend (std::coroutine_handle<> awaiting)
  {
       fur_ = std:: async (std::launch:: async,
                         &read_file,
                         filename_,
                         buffer_,
                         buf_len_,
                         read_len_,
                         awaiting);
       result_ = fur_. get ();
       awaiting.resume ();
  }
   //返回结果
   int await_resume ()
  {
       return result_;
  }

   //读文件的参数，返回值
   int result_ = -1;
   const char* filename_ = nullptr;
   char* buffer_ = nullptr;
   size_t buf_len_ = 0;
   size_t* read_len_ = nullptr;

   std::future<int> fur_;

   //! 协程的句柄
   std::coroutine_handle<> awaiting_;
};



//这就是一个协程函数
coro_ret<int> coroutine_await (const char* filename,
                             char* buffer,
                             size_t buf_len,
                             size_t* read_len)
{
   int ret = co_await await_read_file (filename,
                                      buffer,
                                      buf_len,
                                      read_len);
   //这行其实没有执行到。
   std:: cout << "await_read_file ret= " << ret << std:: endl;
   if (ret == 0)
  {
       std:: cout << "await_read_file read_len= " << *read_len << std:: endl;
  }
   co_return 0;
}

int main (int argc, char* argv[])
{
   using namespace std:: chrono_literals;
   //调用协程
   char buffer[1024];
   size_t read_len = 0;
   std:: cout << "Start coroutine_await coroutine\n";
   auto c_r = coroutine_await ("E:/TEST 001/aio_test_001. txt",
                              buffer,
                              1024,
                              &read_len);
   std:: cout << "End coroutine_await coroutine\n";
   return 0;
}
```

最后输出的信息记录是：

```
Start coroutine_await coroutine
await_read_file ret= 0
await_read_file read_len= 20
final_suspend invoked.
End coroutine_await coroutine
```

其实您可以已经发现了。这个实现虽然可以正常运行，但没有起到任何异步操作效果，因为 await_suspend 的接口虽然发起了异步操作 std:: async。但后面又进行了等待操作 result_ = fur_. get ();

```
void await_suspend (std::coroutine_handle<> awaiting)
  {
       fur_ = std:: async (std::launch:: async,
                         &read_file,
                         filename_,
                         buffer_,
                         buf_len_,
                         read_len_,
                         awaiting);
       result_ = fur_. get ();
       awaiting.resume ();
  }
```

你可以认为虽然他发起了异步操作，整个主线程还是阻塞的，没有任何异步效果。

## 例子 ：再次尝试异步 IO（有 bug，多线程的危险）
------------------------------

在部分文章例子代码中，他们会提出一些异步思路。

比如在异步执行的函数 read_file 中去调用 coro_hdl.resume (); 在 await_resume 中执行 result_ = fur_. get (); 效果如何呢？我们先贴出作出改进代码。

```
int read_file (const char* filename,
             char* buffer,
             size_t buf_len,
             size_t* read_len,
             std::coroutine_handle<> coro_hdl)
{
//…………
   //到了最后一步
READ_FILE_END:
   //变化点：在 AIO 的线程里面恢复协程。
   coro_hdl.resume ();
   return result;
}

struct await_read_file
{
//其他代码没改变
   //…………
   //挂起的操作，发起异步读文件操作，然后等待返回
   void await_suspend (std::coroutine_handle<> awaiting)
  {
       fur_ = std:: async (std::launch:: async,
                         &read_file,
                         filename_,
                         buffer_,
                         buf_len_,
                         read_len_,
                         awaiting);
       //不再在这个地方进行等待了
  }
   //返回结果
   int await_resume ()
  {
       result_ = fur_. get ();
       return result_;
  }
}
```

但这无疑是一个错误的改进。最后的输出结果要不就是崩溃，要不就是无法真正完成协程。

```
Start coroutine_await ()
Start coroutine_await ()
End coroutine_await ()
# 协程并没有执行完成
```

为什么？？？这儿又是因为**可恶的多线程陷阱了**。我们贴个时序图，您就会更加理解。

![[thread-coroutine.png]]

您**不能在另外一个线程中去恢复协程的运行。**，切记，切记。

## 吐槽一下 C++ 11 的异步操作设计
-----------------------

那应该如何修正，能异步操作，有能唤醒协程呢？方法还是有的，在我们发起 std:: aysnc 操作，得到一个 std:: future 时，我们可以在主循环里面去等待`std::future`，因为`future`可以等待很短的时间，也可以反复尝试。这样我们的代码主循环就一边等待（反复尝试），一边干点别的事情。

不过我也懒得把这个很丑的模型实现出来了。

这儿我们可以讨论一个问题，C++ 的异步模式，`promise/future`，`async/future`，都需要 future 在后面等待事情的完成。特别是在**服务器类型的开发**，这种方式并不好用。（我注明了服务器类型呀）

首先看，每一个异步操作都（可能）需要启动一个线程，这个消耗过大，其次每一个 future 都需要等待，其实在设计上也很讨厌。如果你设计一个队列保存 future，那么还需要将 future 和需要回调的操作绑定起来。

个人用不太惯，有高人指点一下？在服务器里面怎么

## 异步协程 co_await awaiter 接口设计
------------------------------

上面那个例子很初步，真正用起来很不爽，那么怎么设计能更加好的设计协程的异步 IO。

首先我们回顾一下传统的`libuv`这类传统的 AIO 设计。

这类 AIO 都是通过一个请求消息队列传递请求给线程池，让线程池去真正干活。线程池干完活后，再将结果返回给一个应答消息队列。请求消息中有一个请求者的回调函数指针，随后又会回填给应答消息中。主循环会不断检查应答消息队列里面有没有消息，如果有应答消息，就从消息中取出回调函数调用之。

这种模型才是比较通用的服务器异步模型设计。这种模型也很容易结合到协程`co_await awaiter`设计中来。你只需要在回调函数里面激活挂起的协程就可以了。

做一个简单的时序图给大家。

![[coroutine-timing-diagram.png]]

而如果你想用`libuv`封装，我估计还是改造一下 libuv 的代码。毕竟如果寄希望协程句柄透传回填回来。也需要消息结构进行改变。

至于代码，我自己的代码库 zcelib/dev 分支，aio 目录下的代码有一个测试实现。因为涉及的面有不少（因为功能，代码写在好多 CPP 里面），只贴出部分说明一下吧。

```
//AIO 文件处理相关的 awaiter 等待体
struct await_aiofs
{
    await_aiofs (zce::aio::Worker* worker,
                zce::aio::FS_Handle* fs_hdl)
         worker_(worker),
         fs_hdl_(fs_hdl)
    {
    }
    ~await_aiofs () = default;

    //是否准备好
    bool await_ready ()
    {
        return false;
    }
    //挂起操作
    void await_suspend (std::coroutine_handle<> awaiting);
    {
        //回调函数
        fs_hdl_->call_back_ = std:: bind (&await_aiofs:: resume,
                                        this,
                                        std::placeholders::_1);
        //将一个文件操作句柄放入请求队列
        bool succ_req = worker_->request (fs_hdl_);
        if (succ_req)
        {
            return false;
        }
        else
        {
            return true;
        }
    }
    //! 恢复后返回结果
    FS_Handle await_resume ()
    {
        return return_hdl_;
    }
    //! 回调函数
    void resume (AIO_Handle* return_hdl)
    {
        FS_Handle* fs_hdl = (FS_Handle*) return_hdl;
        return_hdl_ = *fs_hdl;
        awaiting_. resume ();
        return;
    }

    //! 工作者，具有请求，应答管道，处理 IO 多线程的管理者
    zce::aio::Worker* worker_ = nullptr;
    //! 请求的文件操作句柄
    zce::aio::FS_Handle* fs_hdl_ = nullptr;
    //! 完成后返回的句柄
    zce::aio:: FS_Handle return_hdl_;
    //! 协程的句柄（调用者）
    std::coroutine_handle<> awaiting_;
};

//AIO 协程的 co_await 函数
await_aiofs co_read_file (zce::aio::Worker* worker,
                         const char* path,
                         char* read_bufs,
                         size_t nbufs,
                         ssize_t offset)
{
    //从对象池分配一个 FS_Handle
    zce::aio::FS_Handle* aio_hdl = (FS_Handle*)
        worker->alloc_handle (AIO_TYPE::FS_READFILE);
    aio_hdl->path_ = path;
    aio_hdl->read_bufs_ = read_bufs;
    aio_hdl->bufs_count_ = nbufs;
    aio_hdl->offset_ = offset;

    return await_aiofs (worker, aio_hdl);
}
```

最后，我们来剖析一下协程的过程。通过这个剖析，希望达到梳理协程几个重要概念的关系，把这些点串起来。所以在概念参考我们列出了相应的概念文字。

## 协程的创建
---------

C++20 协程在启动前，开始会 new 一个协程状态（`coroutine state`）。然后构造协程的承诺对象（`promise`)。承诺对象（`promise`) 通过`get_return_object ()`构造协程的返回值`result`。这个返回值在协程第一次挂起时，赋值给调用者。然后通过`co_await promise. initial_suspend ()`，决定协程初试完成后的行为。如果返回`std::suspend_always`，初始化就挂起，如果返回`std::suspend_never` ，初始化后就继续运行。(注意 initial_suspend 也可以返回其他协程体)

![[Pasted image 20230720135414.png]]

### 协程的 co_await
----------------

`cw_ret = co_await awaiter` 或者`cw_ret = co_await fun ()`，先计算表达式 fun，fun 返回结果，就是一个等待体`awaiter`。系统先调用`awaiter. await_ready ()`接口，看等待体是否准备好了，没准备好（`return false`）就调用`awaiter. await_suspend ()`。`await_suspend`根据参数可以记录调用其的协程的的句柄。`await_suspend`的返回值为`return true` ，或者 `return void` 就会挂起协程。

后面在外部如果恢复了协程的运行，`awaiter. await_resume ()`接口被调用。其返回结果，作为`co_await`的返回值。

![[Pasted image 20230720135429.png]]

**协程的 co_yield**
----------------

`co_yield cy_ret;`，相当于调用`co_wait promise. yield_value (cy_ret)`，你可以在`yield_value`中记录参数`cy_ret`后面使用，`yield_value`的返回值如果是`std::suspend_always`，协程挂起，如果返回`std::suspend_never` ，协程就继续运行。

![](https://pic4.zhimg.com/v2-c134d45be8f1a5c05426488eb01fe803_r.jpg)

### 协程的 co_return
-----------------

`co_yield cr_ret;`，调用`promise. retun_value (cr_ret)`，如果没有返回值相当于`promise. retun_viod ()`，你可以在`retun_value`中记录参数`cr_ret`后面使用。然后调用`co_await promise. final_suspend (void)`，如果返回值是`std::suspend_always`，你需要自己手动青清理`coroutine handle`，调用`handle.destroy ()`。

![[Pasted image 20230720135451.png]]

> [! question]
 这儿存在一个疑问，`final_suspend`，并没有真正挂起协程。看 C++ 参考, 里面说的也是 `calls promise. final_suspend () and co_awaits the result.`。按说如果返回应该要挂起。但用 VS 2022 测试是不会挂起的，再探 C++20 协程文章中说的是如果返回 `std::suspend_always`，需要你自己清理 `coroutine handle`。存疑吧。

## 概念参考附录
-----------

这些概念在原文第一章都有，附录在此仅供您方便参考。

### 协程状态（coroutine state）

协程状态（`coroutine state`）是协程启动开始时，new 空间存放协程状态，协程状态记录协程函数的参数，协程的运行状态，变量。挂起时的断点。

注意，协程状态 (`coroutine state`) 并不是就是协程函数的返回值 RET。虽然我们设计的 RET 一般里面也有`promise`和`coroutine handle`，大家一般也是通过 RET 去操作协程的恢复，获取返回值。但`coroutine state`理论上还应该包含协程运行参数，断点等信息。而**协程状态** (`coroutine state`) 应该是协程句柄（`coroutine handle`）对应的一个数据，而由系统管理的。

### 承诺对象（promise）

承诺对象的表现形式必须是`result::promise_type`，`result`为协程函数的返回值。

承诺对象是一个实现若干接口，用于辅助协程，构造协程函数返回值；提交传递`co_yield`，`co_return`的返回值。明确协程启动阶段是否立即挂起；以及协程内部发生异常时的处理方式。其接口包括：

*   `auto get_return_object ()` ：用于生成协程函数的返回对象。
*   `auto initial_suspend ()`：用于明确初始化后，协程函数的执行行为，返回值为等待体（`awaiter`），用`co_wait`调用其返回值。返回值为`std::suspend_always` 表示协程启动后立即挂起（不执行第一行协程函数的代码），返回`std::suspend_never` 表示协程启动后不立即挂起。（当然既然是返回等待体，你可以自己在这儿选择进行什么等待操作）
*   `void return_value (T v)`：调用`co_return v`后会调用这个函数，可以保存`co_return`的结果
*   `auto yield_value (T v)`：调用`co_yield`后会调用这个函数，可以保存`co_yield`的结果，其返回其返回值为`std::suspend_always`表示协程会挂起，如果返回`std::suspend_never`表示不挂起。
*   `auto final_suspend () noexcept`：在协程退出是调用的接口，返回`std::suspend_never` ，自动销毁 coroutine state 对象。若 `final_suspend` 返回 `std::suspend_always` 则需要用户自行调用 `handle.destroy ()` 进行销毁。但值得注意的是返回 std:: suspend_always 并不会挂起协程。

前面我们提到在协程创建的时候，会 new 协程状态（`coroutine state`）。你可以通过可以在 `promise_type` 中重载 `operator new` 和 `operator delete`，使用自己的内存分配接口。

### 协程句柄（coroutine handle）

协程句柄（`coroutine handle`）是一个协程的标示，用于操作协程恢复，销毁的句柄。

协程句柄的表现形式是`std::coroutine_handle<promise_type>`，其模板参数为承诺对象（`promise`）类型。句柄有几个重要函数：

*   `resume ()`函数可以恢复协程。
*   `done ()`函数可以判断协程是否已经完成。返回 false 标示协程还没有完成，还在挂起。

协程句柄和承诺对象之间是可以相互转化的。

*   `std::coroutine_handle<promise_type>::from_promise` ：这是一个静态函数，可以从承诺对象（promise）得到相应句柄。
*   `std::coroutine_handle<promise_type>:: promise ()` 函数可以从协程句柄`coroutine handle`得到对应的承诺对象（`promise`）

### 等待体（awaiter）

co_wait 关键字会调用一个等待体对象 (`awaiter`)。这个对象内部也有 3 个接口。根据接口 co_wait 决定进行什么操作。

*   `bool await_ready ()`：等待体是否准备好了，返回 false ，表示协程没有准备好，立即调用 await_suspend。返回 true，表示已经准备好了。
*   `auto await_suspend (std::coroutine_handle<> handle)`如果要挂起，调用的接口。其中 handle 参数就是调用等待体的协程，其返回值有 3 种可能

*   void 同返回 true
*   bool 返回 true 立即挂起，返回 false 不挂起。
*   返回某个协程句柄（`coroutine handle`），立即恢复对应句柄的运行。

*   `auto await_resume ()` ：协程挂起后恢复时，调用的接口。返回值作为 co_wait 操作的返回值。

**等待体**（awaiter）值得用更加详细的笔墨书写一章，我们就放一下，先了解其有 2 个特化类型。

*   `std::suspend_never`类，不挂起的的特化等待体类型。
*   `std::suspend_always`类，挂起的特化等待体类型。

前面不少接口已经用了这 2 个特化的类，同时也可以明白其实协程内部不少地方其实也在使用`co_wait` 关键字。