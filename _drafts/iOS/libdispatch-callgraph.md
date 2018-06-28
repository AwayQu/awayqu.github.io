---
layout: post
title:  "libdispatch call graph (GCD)"
date:   2018-05-06 00:00:22 +0800
categories: iOS
tags: [iOS, libdispatch]
---

### 调用dispatch_async(global_queue, block)


```shell
#1	0x000000010695a3f7 in _dispatch_call_block_and_release () libdispatch.dylib
#2	0x000000010695b43c in _dispatch_client_callout ()         libdispatch.dylib
#3	0x0000000106960352 in _dispatch_queue_override_invoke ()  libdispatch.dylib
#4	0x00000001069671f9 in _dispatch_root_queue_drain ()       libdispatch.dylib
#5	0x0000000106966e97 in _dispatch_worker_thread3 ()         libdispatch.dylib
#6	0x0000000106e1d1ca in _pthread_wqthread ()                libsystem_pthread.dylib
#7	0x0000000106e1cc4d in start_wqthread ()                   libsystem_pthread.dylib
```

### 调用dispatch_async(main_queue, block)

```shell

#1	0x0000000106be73f7 in _dispatch_call_block_and_release ()
#2	0x0000000106be843c in _dispatch_client_callout ()
#3	0x0000000106bf36f0 in _dispatch_main_queue_callback_4CF ()
#4	0x00000001034cfef9 in __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__ ()

```

### 空闲线程

```
#0	0x000000010cd6c6da in __workq_kernreturn ()
#1	0x000000010cda226f in _pthread_wqthread ()
#2	0x000000010cda1c4d in start_wqthread ()
```


## 处理任务主入口

### 主线程

```c

static void _dispatch_main_queue_drain(void);

```

### 子线程

```c
static void _dispatch_queue_drain(dispatch_queue_t dq);

```



### pthread_queue.h

#### pthread_workqueue_additem_np

```c
int pthread_workqueue_additem_np(pthread_workqueue_t workq, void ( *workite     m_func)(void *), void * workitem_arg, pthread_workitem_handle_t * itemhandl     ep, unsigned int *gencountp);
```


### 
```c
int 
pthread_workqueue_additem_np(pthread_workqueue_t workq, void ( *workitem_func)(void *), void * workitem_arg, pthread_workitem_handle_t * itemhandlep, unsigned int *gencountp)
{
	pthread_workitem_t witem;

	if (valid_workq(workq) == 0) {
		return(EINVAL);
	}

	workqueue_list_lock();

	/*
	 * Allocate the workitem here as it can drop the lock.
	 * Also we can evaluate the workqueue state only once.
	 */
	witem = alloc_workitem();
	witem->func = workitem_func;
	witem->func_arg = workitem_arg;
	witem->flags = 0;
	witem->workq = workq;
	witem->item_entry.tqe_next = 0;
	witem->item_entry.tqe_prev = 0;

	/* alloc workitem can drop the lock, check the state  */
	if ((workq->flags & (PTHREAD_WORKQ_IN_TERMINATE | PTHREAD_WORKQ_DESTROYED)) != 0) {
		free_workitem(witem);
		workqueue_list_unlock();
		*itemhandlep = 0;
		return(ESRCH);
	}

	if (itemhandlep != NULL)
		*itemhandlep = (pthread_workitem_handle_t *)witem;
	if (gencountp != NULL)
		*gencountp = witem->gencount;
#if WQ_TRACE
	__kdebug_trace(0x9008090, witem, witem->func, witem->func_arg,  workq, 0);
#endif
	TAILQ_INSERT_TAIL(&workq->item_listhead, witem, item_entry);
#if WQ_LISTTRACE
	__kdebug_trace(0x90080a4, workq, &workq->item_listhead, workq->item_listhead.tqh_first,  workq->item_listhead.tqh_last, 0);
#endif
	
	if (((workq->flags & PTHREAD_WORKQ_BARRIER_ON) == 0) && (workq->queueprio < wqreadyprio))
			wqreadyprio = workq->queueprio;

	pick_nextworkqueue_droplock();

	return(0);
}
```