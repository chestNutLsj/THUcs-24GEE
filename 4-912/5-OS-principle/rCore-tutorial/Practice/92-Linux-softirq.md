## Softirqs

softirqs can not be used by device drivers, they are reserved for various kernel subsystems. Because of this there is a fixed number of softirqs defined at compile time. For the current kernel version we have the following types defined:
```
enum {
    HI_SOFTIRQ = 0,
    TIMER_SOFTIRQ,
    NET_TX_SOFTIRQ,
    NET_RX_SOFTIRQ,
    BLOCK_SOFTIRQ,
    IRQ_POLL_SOFTIRQ,
    TASKLET_SOFTIRQ,
    SCHED_SOFTIRQ,
    HRTIMER_SOFTIRQ,
    RCU_SOFTIRQ,
    NR_SOFTIRQS
};
```
Each type has a specific purpose:

- _HI_SOFTIRQ_ and _TASKLET_SOFTIRQ_ - running tasklets
- _TIMER_SOFTIRQ_ - running timers
- _NET_TX_SOFIRQ_ and _NET_RX_SOFTIRQ_ - used by the networking subsystem
- _BLOCK_SOFTIRQ_ - used by the IO subsystem
- _BLOCK_IOPOLL_SOFTIRQ_ - used by the IO subsystem to increase performance when the iopoll handler is invoked;
- _SCHED_SOFTIRQ_ - load balancing
- _HRTIMER_SOFTIRQ_ - implementation of high precision timers
- _RCU_SOFTIRQ_ - implementation of RCU type mechanisms[^1]


The highest priority is the _HI_SOFTIRQ_ type softirqs, followed in order by the other softirqs defined. _RCU_SOFTIRQ_ has the lowest priority.

Softirqs are running in interrupt context which means that they can not call blocking functions. If the sofitrq handler requires calls to such functions, work queues can be scheduled to execute these blocking calls.

### Tasklets

A tasklet is a special form of deferred work that runs in interrupt context, just like softirqs. The main difference between sofirqs and tasklets is that tasklets can be allocated dynamically and thus they can be used by device drivers. A tasklet is represented by `struct tasklet` and as many other kernel structures it needs to be initialized before being used. A pre-initialized tasklet can defined as following:
```
void handler(unsigned long data);

DECLARE_TASKLET(tasklet, handler, data);
DECLARE_TASKLET_DISABLED(tasklet, handler, data);
```

If we want to initialize the tasklet manually we can use the following approach:
```
void handler(unsigned long data);

struct tasklet_struct tasklet;

tasklet_init(&tasklet, handler, data);
```

The _data_ parameter will be sent to the handler when it is executed.

Programming tasklets for running is called scheduling. Tasklets are running from softirqs. Tasklets scheduling is done with:
```
void tasklet_schedule(struct tasklet_struct *tasklet);

void tasklet_hi_schedule(struct tasklet_struct *tasklet);
```

When using _tasklet_schedule_, a _TASKLET_SOFTIRQ_ softirq is scheduled and all tasklets scheduled are run. For _tasklet_hi_schedule_, a _HI_SOFTIRQ_ softirq is scheduled.

If a tasklet was scheduled multiple times and it did not run between schedules, it will run once. Once the tasklet has run, it can be re-scheduled, and will run again at a later timer. Tasklets can be re-scheduled from their handlers.

Tasklets can be masked and the following functions can be used:
```
void tasklet_enable(struct tasklet_struct * tasklet );
void tasklet_disable(struct tasklet_struct * tasklet );
```

Remember that since tasklets are running from softirqs, blocking calls can not be used in the handler function.

### Timers

A particular type of deferred work, very often used, are timers. They are defined by `struct timer_list`. They run in interrupt context and are implemented on top of softirqs.

To be used, a timer must first be initialized by calling `timer_setup()`:
```
#include <linux / sched.h>

void timer_setup(struct timer_list * timer,
                 void (*function)(struct timer_list *),
                 unsigned int flags);
```

The above function initializes the internal fields of the structure and associates _function_ as the timer handler. Since timers are planned over softirqs, blocking calls can not be used in the code associated with the treatment function.

Scheduling a timer is done with `mod_timer()`:
```
int mod_timer(struct timer_list *timer, unsigned long expires);
```

Where _expires_ is the time (in the future) to run the handler function. The function can be used to schedule or reschedule a timer.

The time unit timers is _jiffie_. The absolute value of a jiffie is dependent on the platform and it can be found using the `HZ` macro that defines the number of jiffies for 1 second. To convert between jiffies (_jiffies_value_) and seconds (_seconds_value_), the following formulas are used:
```
jiffies_value = seconds_value * HZ ;
seconds_value = jiffies_value / HZ ;
```

The kernel mantains a counter that contains the number of jiffies since the last boot, which can be accessed via the `jiffies` global variable or macro. We can use it to calculate a time in the future for timers:
```
#include <linux/jiffies.h>

unsigned long current_jiffies, next_jiffies;
unsigned long seconds = 1;

current_jiffies = jiffies;
next_jiffies = jiffies + seconds * HZ;
```

To stop a timer, use `del_timer()` and `del_timer_sync()`:
```
int del_timer(struct timer_list *timer);
int del_timer_sync(struct timer_list *timer);
```

Thse functions can be called for both a scheduled timer and an unplanned timer. `del_timer_sync()` is used to eliminate the races that can occur on multiprocessor systems, since at the end of the call it is guaranteed that the timer processing function does not run on any processor.

A frequent mistake in using timers is that we forget to turn off timers. For example, before removing a module, we must stop the timers because if a timer expires after the module is removed, the handler function will no longer be loaded into the kernel and a kernel oops will be generated.

The usual sequence used to initialize and schedule a one second timeout is:
```
#include <linux/sched.h>

void timer_function(struct timer_list *);

struct timer_list timer ;
unsigned long seconds = 1;

timer_setup(&timer, timer_function, 0);
mod_timer(&timer, jiffies + seconds * HZ);
```

And to stop it:
```
del_timer_sync(&timer);
```

### Locking

For synchronization between code running in process context (A) and code running in softirq context (B) we need to use special locking primitives. We must use spinlock operations augmented with deactivation of bottom-half handlers on the current processor in (A), and in (B) only basic spinlock operations. Using spinlocks makes sure that we don’t have races between multiple CPUs while deactivating the softirqs makes sure that we don’t deadlock in the softirq is scheduled on the same CPU where we already acquired a spinlock.

We can use the `local_bh_disable()` and `local_bh_enable()` to disable and enable softirqs handlers (and since they run on top of softirqs also timers and tasklets):
```
void local_bh_disable(void);
void local_bh_enable(void);
```

Nested calls are allowed, the actual reactivation of the softirqs is done only when all local_bh_disable() calls have been complemented by local_bh_enable() calls:
```
/* We assume that softirqs are enabled */
local_bh_disable();  /* Softirqs are now disabled */
local_bh_disable();  /* Softirqs remain disabled */

local_bh_enable();  /* Softirqs remain disabled */
local_bh_enable();  /* Softirqs are now enabled */
```

>[!Attention]
>These above calls will disable the softirqs only on the local processor and they are usually not safe to use, they must be complemented with spinlocks.

Most of the time device drivers will use special versions of spinlocks calls for synchronization like `spin_lock_bh()` and `spin_unlock_bh()`:
```
void spin_lock_bh(spinlock_t *lock);
void spin_unlock_bh(spinlock_t *lock);
```

[^1]: [Deferred work — The Linux Kernel documentation](https://linux-kernel-labs.github.io/refs/pull/189/merge/labs/deferred_work.html#id2)