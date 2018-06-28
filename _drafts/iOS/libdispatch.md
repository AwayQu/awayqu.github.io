---
layout: post
title:  "libdispatch source code (GCD)"
date:   2018-05-01 00:00:22 +0800
categories: iOS
tags: [iOS, libdispatch]
---

* TOC
{:toc}

# Managing Dispatch Queues


## Dispatch Queue Types

## Dispatch Queue Label Constants

## dispatch_queue_t

> A dispatch queue is a lightweight object to which your application submits blocks for subsequent execution.

> 轻量派发队列结构体

### define
```c
typedef struct dispatch_queue_s *dispatch_queue_t;
```
### dispatch_queue_s

{% capture code-capture %}

```c
struct dispatch_queue_s {
	DISPATCH_STRUCT_HEADER(dispatch_queue_s, dispatch_queue_vtable_s);
	DISPATCH_QUEUE_HEADER;
	char dq_label[DISPATCH_QUEUE_MIN_LABEL_SIZE]; // must be last
	char _dq_pad[DISPATCH_QUEUE_CACHELINE_PAD]; // for static queues only
};
```

```c
struct dispatch_queue_s {
//regoin expand DISPATCH_STRUCT_HEADER(dispatch_queue_s, dispatch_queue_vtable_s)
    const struct dispatch_queue_vtable_s *do_vtable; // 当前执行的任务类型 栅栏, 异步, 组, 同步慢
    struct dispatch_queue_s *volatile do_next; // 下一个dc
	unsigned int do_ref_cnt;               //引用计数，这是内部gcd内部使用的计数器
	unsigned int do_xref_cnt;             // 外部引用计数，这是gcd外部使用的计数器，两者都为0的时候才能dispose
	unsigned int do_suspend_cnt;          // suspend计数，用作暂停标志
	struct dispatch_queue_s *do_targetq;  // 目标队列
	void *do_ctxt;                        // dc的上下文
	void *do_finalizer;                   // dc销毁时候调用函数
//endregoin
//regoin expand DISPATCH_QUEUE_HEADER
    uint32_t volatile dq_running;  /// 当前执行的任务数
	uint32_t dq_width;  /// 同时可执行的任务数 , serial 为 1, concurrency 为其他值
	struct dispatch_object_s *volatile dq_items_tail; 
	struct dispatch_object_s *volatile dq_items_head; 
	unsigned long dq_serialnum; 
	dispatch_queue_t dq_specific_q;
//endregoin
	char dq_label[DISPATCH_QUEUE_MIN_LABEL_SIZE]; // must be last
	char _dq_pad[DISPATCH_QUEUE_CACHELINE_PAD]; // for static queues only
};

```

{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats" button-text="Toggle Code" toggle-text=code-capture %}

## dispatch_get_main_queue

> Returns the serial dispatch queue associated with the application’s main thread.

>  获取主线程关联的 **串行 dispatch queue**.

### define

> extern 修饰的一个全局 结构体 `dispatch_queue_s`

```c
__OSX_AVAILABLE_STARTING(__MAC_10_6,__IPHONE_4_0)
DISPATCH_EXPORT struct dispatch_queue_s _dispatch_main_q;
#define dispatch_get_main_queue() (&_dispatch_main_q)
```

## dispatch_get_global_queue

> Returns a system-defined global concurrent queue with the specified quality of service class.

### define

```c
dispatch_queue_t
dispatch_get_global_queue(long priority, unsigned long flags)
{
	/**
	 * DISPATCH_QUEUE_OVERCOMMIT = 0x2ull
	 * 即 flags 不等于 0 或者 2时 会返回null
     * 因为会无视目前的计算机状况，后续会创建一个新的线程来关联一个新的queue进行处理任务。
	 */
	if (flags & ~DISPATCH_QUEUE_OVERCOMMIT) {
		return NULL;
	}
	/**
	 * 当flags 为 2时， 会启用 overcommit
     * 当flags 为 0时， 不会启用 overcommit
	 */
	return _dispatch_get_root_queue(priority,
			flags & DISPATCH_QUEUE_OVERCOMMIT);
}
```

```c
// priority
#define DISPATCH_QUEUE_PRIORITY_HIGH 2
#define DISPATCH_QUEUE_PRIORITY_DEFAULT 0
#define DISPATCH_QUEUE_PRIORITY_LOW (-2)
#define DISPATCH_QUEUE_PRIORITY_BACKGROUND INT16_MIN
```

### usage

> flags: Flags that are reserved for future use. Always specify 0 for this parameter. --- From apple doc

> flags 是保留参数， 目前都是传0， `也就是说都是使用不启用overcommit的queue`。 (支持传 **0**或者**2**)


```c
dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
```

> TODO: 测试传2会发生什么

## dispatch_get_current_queue

> Returns the queue on which the currently executing block is running. (Deprecated)

> 优先通过 dispatch_key 获取， 获取失败，再从`_dispatch_root_queues`这个全局数组中获取。

### define
```c
dispatch_queue_t
dispatch_get_current_queue(void)
{
	/**
	 * _dispatch_queue_get_current()
	 *  通过dispatch_queue_key 获取 当前线程关联的数据 dispatch_queue_t
	 *  在TSD启用下 使用_pthread_getspecific_direct
     *  不启用下 使用pthread api（pthread_getspecific）
	 * _dispatch_get_root_queue()
	 *  上一步获取不到，说明还未初始化，
     *  调用此方法获取默认生成的队列, 传入DISPATCH_QUEUE_PRIORITY_DEFAULT 0
     *  overcommit true
	 */
	return _dispatch_queue_get_current() ?: _dispatch_get_root_queue(0, true);
}
```

### 相关实现 

**_dispatch_queue_get_current**
```c
DISPATCH_ALWAYS_INLINE
static inline dispatch_queue_t
_dispatch_queue_get_current(void)
{
	return _dispatch_thread_getspecific(dispatch_queue_key);
}
```

**dispatch_queue_key**

> dispatch_queue_key : `unsigned long`

```c
#if DISPATCH_USE_DIRECT_TSD
static const unsigned long dispatch_queue_key		= __PTK_LIBDISPATCH_KEY0;
#else 
pthread_key_t dispatch_queue_key;
#endif

**_dispatch_get_root_queue**

> 获取 `_dispatch_root_queues : dispatch_queue_s[]` 全局数组中的队列

```c

DISPATCH_ALWAYS_INLINE DISPATCH_CONST
static inline dispatch_queue_t
_dispatch_get_root_queue(long priority, bool overcommit)
{
	// overcommit 过载使用 (八种队列)
	if (overcommit) switch (priority) {
	case DISPATCH_QUEUE_PRIORITY_LOW:
		return &_dispatch_root_queues[
				DISPATCH_ROOT_QUEUE_IDX_LOW_OVERCOMMIT_PRIORITY];
	case DISPATCH_QUEUE_PRIORITY_DEFAULT:
		return &_dispatch_root_queues[
				DISPATCH_ROOT_QUEUE_IDX_DEFAULT_OVERCOMMIT_PRIORITY];
	case DISPATCH_QUEUE_PRIORITY_HIGH:
		return &_dispatch_root_queues[
				DISPATCH_ROOT_QUEUE_IDX_HIGH_OVERCOMMIT_PRIORITY];
	case DISPATCH_QUEUE_PRIORITY_BACKGROUND:
		return &_dispatch_root_queues[
				DISPATCH_ROOT_QUEUE_IDX_BACKGROUND_OVERCOMMIT_PRIORITY];
	}
	switch (priority) {
	case DISPATCH_QUEUE_PRIORITY_LOW:
		return &_dispatch_root_queues[DISPATCH_ROOT_QUEUE_IDX_LOW_PRIORITY];
	case DISPATCH_QUEUE_PRIORITY_DEFAULT:
		return &_dispatch_root_queues[DISPATCH_ROOT_QUEUE_IDX_DEFAULT_PRIORITY];
	case DISPATCH_QUEUE_PRIORITY_HIGH:
		return &_dispatch_root_queues[DISPATCH_ROOT_QUEUE_IDX_HIGH_PRIORITY];
	case DISPATCH_QUEUE_PRIORITY_BACKGROUND:
		return &_dispatch_root_queues[
				DISPATCH_ROOT_QUEUE_IDX_BACKGROUND_PRIORITY];
	default:
		return NULL;
	}
}
```

## dispatch_set_target_queue (TODO)

> Sets the target queue for the given object.

> http://www.cnblogs.com/denz/p/5214297.html

### define

```c
void dispatch_set_target_queue(dispatch_object_t object, dispatch_queue_t queue);
```

**object**

> The object to modify. This parameter cannot be NULL.

**queue**

>The new target queue for the object. The queue is retained, and the previous one, if any, is released. This parameter cannot be NULL.


## dispatch_async

> Submits a block for asynchronous execution on a dispatch queue and returns immediately.

### define

```c
void dispatch_async(dispatch_queue_t queue, dispatch_block_t block);
```

```c
void
dispatch_async(dispatch_queue_t dq, void (^work)(void))
{
	// 拷贝block, 使用_dispatch_call_block_and_release 函数, 调用block
	dispatch_async_f(dq, _dispatch_Block_copy(work),
			_dispatch_call_block_and_release);
}
```

### 相关实现

```c
dispatch_block_t
_dispatch_Block_copy(dispatch_block_t db)
{
	dispatch_block_t rval;
	// 调用 usr/include/Block.h 下的 _Block_copy
	// 阻塞拷贝block 直到拷贝成功 TODO:为什么需要阻塞
	while (!(rval = Block_copy(db))) {
		sleep(1);
	}
	return rval;
}

void
_dispatch_call_block_and_release(void *block)
{
	void (^b)(void) = block;
	b();
	// 调用 usr/include/Block.h 下的 _Block_release
	Block_release(b);
}

```

## dispatch_async_f

> Submits an application-defined function for asynchronous execution on a dispatch queue and returns immediately.

### define

```c
void dispatch_async_f(dispatch_queue_t queue, void *context, dispatch_function_t work);
```

### 相关实现

```c
DISPATCH_NOINLINE
void
dispatch_async_f(dispatch_queue_t dq, void *ctxt, dispatch_function_t func)
{
	dispatch_continuation_t dc;

	// No fastpath/slowpath hint because we simply don't know
	if (dq->dq_width == 1) {
		/// 说明是串行
		/// 使用栅栏异步
		return dispatch_barrier_async_f(dq, ctxt, func);
	}

	/// 从当前线程中 获取dc, 并且重新设置线程中的dc
	dc = fastpath(_dispatch_continuation_alloc_cacheonly());
	if (!dc) {
		/// 说明当前线程未设置dc, 创建dc并设置, 然后再异步处理
		return _dispatch_async_f_slow(dq, ctxt, func);
	}

	dc->do_vtable = (void *)DISPATCH_OBJ_ASYNC_BIT;
	dc->dc_func = func;
	dc->dc_ctxt = ctxt;

	// No fastpath/slowpath hint because we simply don't know
	if (dq->do_targetq) {
		/// 如果有do_targetq, 则优先使用do_targetq来进行入队dc
        /// TODO: _dispatch_async_f2 内部实现
		return _dispatch_async_f2(dq, dc);
	}

	/// 原子性, 线程安全的将 dc 加入dq
	_dispatch_queue_push(dq, dc);
}
```

```c
static inline dispatch_continuation_t
_dispatch_continuation_alloc_cacheonly(void)
{
	dispatch_continuation_t dc;
	/// 从线程的关联数据中 获取 dc
	dc = fastpath(_dispatch_thread_getspecific(dispatch_cache_key));
	if (dc) {
		/// 如果获取到了 dc 将 do_next链中的下一个dc, 设为当前线程中的dc, 使用当前dc
		_dispatch_thread_setspecific(dispatch_cache_key, dc->do_next);
	}
	return dc;
}
```
`_dispatch_queue_push`内部调用的函数
```c
DISPATCH_ALWAYS_INLINE
static inline void
_dispatch_queue_push_list(dispatch_queue_t dq, dispatch_object_t _head,
		dispatch_object_t _tail)
{
	struct dispatch_object_s *prev, *head = _head._do, *tail = _tail._do;
	/// 将原来dq_items_tail 换为 tail, 并清空 tail的 do_next
	/// 然后将原来dq_items_tail 链上 head
	tail->do_next = NULL;
	dispatch_atomic_store_barrier();
	/// 原子性 交换值 (也就是dq 需要保证线程安全)
	prev = fastpath(dispatch_atomic_xchg2o(dq, dq_items_tail, tail));
	if (prev) {
		
		// if we crash here with a value less than 0x1000, then we are at a
		// known bug in client code for example, see _dispatch_queue_dispose
		// or _dispatch_atfork_child
		prev->do_next = head;
	} else {
		_dispatch_queue_push_list_slow(dq, head);
	}
}
```





## dispatch_sync_f

> Submits an application-defined function for synchronous execution on a dispatch queue.


### define

```c
void dispatch_sync_f(dispatch_queue_t queue, void *context, dispatch_function_t work);
```

```c

DISPATCH_NOINLINE
void
dispatch_sync_f(dispatch_queue_t dq, void *ctxt, dispatch_function_t func)
{
	if (fastpath(dq->dq_width == 1)) {
		/// 串行情况下使用, 栅栏同步
		return dispatch_barrier_sync_f(dq, ctxt, func);
	}
	if (slowpath(!dq->do_targetq)) {
		// the global root queues do not need strict ordering
		(void)dispatch_atomic_add2o(dq, dq_running, 2);
		return _dispatch_sync_f_invoke(dq, ctxt, func);
	}
	_dispatch_sync_f2(dq, ctxt, func);
}

```

### 相关实现

**_dispatch_sync_f_invoke**

```c

```
**_dispatch_function_invoke**

```c
```

**_dispatch_wakeup**

```c
```


# Using Dispatch Barriers

## dispatch_barrier_async

## dispatch_barrier_async_f

> Submits a barrier function for asynchronous execution and returns immediately.

### define

```c

void dispatch_barrier_async_f(dispatch_queue_t queue, void *context, dispatch_function_t work);

```

### 内部实现
```c
DISPATCH_NOINLINE
void
dispatch_barrier_async_f(dispatch_queue_t dq, void *ctxt,
		dispatch_function_t func)
{
	dispatch_continuation_t dc;
    /// 从线程关联数据中获取dc, 并且设置下一个dc
	dc = fastpath(_dispatch_continuation_alloc_cacheonly());
	if (!dc) {
		/// 没有创建dc, 则创建dc, 再将dc 入队列
		return _dispatch_barrier_async_f_slow(dq, ctxt, func);
	}

	dc->do_vtable = (void *)(DISPATCH_OBJ_ASYNC_BIT | DISPATCH_OBJ_BARRIER_BIT);
	dc->dc_func = func;
	dc->dc_ctxt = ctxt;

	/// 原子性将dc 入队列
	_dispatch_queue_push(dq, dc);
}

```

### dispatch_barrier_async_f 和 dispatch_async_f 区别

```c

    dispatch_continuation_t dc = _dispatch_continuation_alloc_from_heap();

//region async implement
	dc->do_vtable = (void *)DISPATCH_OBJ_ASYNC_BIT;
//otherregion  barrier async implement
    dc->do_vtable = (void *)(DISPATCH_OBJ_ASYNC_BIT | DISPATCH_OBJ_BARRIER_BIT);
//endregion
	dc->dc_func = func;
	dc->dc_ctxt = ctxt;

```


# 所需要的知识点

TSD(Thread Specific Data)

volatile

overcommit priority。/

pthread_workqueue

pthread_workqueue_create_np

ULL (unsigned long long ) 例子 0x2ull,

～ 按位取反运算符

fastpath/slowpath

__sync*

mach port

asm("pause") (https://stackoverflow.com/questions/10159796/what-does-asmpause-do-and-why-to-use-it?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

## 相关解释

### fastpath/slowpath

https://yq.aliyun.com/articles/61328

* fastpath表示条件更可能成立
* slowpath表示条件更不可能成立


### overcommit priority
```swift
Global Dispatch Queue(High Priority)
Global Dispatch Queue(Default Priority)
Global Dispatch Queue(Low Priority)
Global Dispatch Queue(Background Priority)
Global Dispatch Queue(High Overcommit Priority)
Global Dispatch Queue(Default Overcommit Priority)
Global Dispatch Queue(Low Overcommit Priority)
Global Dispatch Queue(Background Overcommit Priority)
```

优先级中附有Overcommit的Global Dispatch Queue使用在Serial Dispatch Queue中。如Overcommit 这个名称所示，不管系统状态如何，都会强制生成线程的Dispatch Queue。 这8种Global Dispatch Queue各使用1个pthread_workqueue。GCD初始化时，使用pthread_workqueue_create_np函数生成pthread_workqueue。

### fastpath/slowpath

### ___sync*

#### __sync_bool_compare_and_swap

https://gcc.gnu.org/onlinedocs/gcc-4.1.0/gcc/Atomic-Builtins.html

#### ___sync_swap

https://clang.llvm.org/docs/LanguageExtensions.html


> __sync_swap is used to atomically swap integers or pointers in memory.

##### Syntax:

```c
type __sync_swap(type *ptr, type value, ...)
```

##### Example of Use:

```c
int old_value = __sync_swap(&value, new_value);
```

##### Description:

The __sync_swap() builtin extends the existing __sync_*() family of atomic intrinsics to allow code to atomically swap the current value with the new value. More importantly, it helps developers write more efficient and correct code by avoiding expensive loops around __sync_bool_compare_and_swap() or relying on the platform specific implementation details of __sync_lock_test_and_set(). The __sync_swap() builtin is a full barrier.