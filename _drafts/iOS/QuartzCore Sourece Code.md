---
layout: post
title:  "Quartz Core Source Code"
date:   2017-12-24 00:52:22 +0800
categories: iOS
tags: [iOS, QuartzCore]
---

# Quartz Core Source Code


## commit_transaction

```c++
CA::Transaction::commit_transaction(CA::Transaction *transaction);
```

**使用到的函数**

```c++
CA::Context::retain_all_contexts();
CA::Transaction::get_value();
CA::Transaction::run_commit_handlers();
objc_autoreleasePoolPush();
objc_autoreleasePoolPop();
CA::Layer::layout_if_needed();
CA::Layer::thread_flags_();
pthread_mutex_lock();
pthread_mutex_unlock();
CA::Layer::collect_layers_();
CA::Transaction::unlock();
[CALayer display];
_x_heap_free();
CA::Layer::prepare_commit();
CA::Context::retain_all_contexts();
CA::Context::unref();
mach_port_allocate();
mach_port_insert_right();
CA::Context::retain_render_ctx();
CA::Render::Fence::set();
CA::Transaction::Fence::release_port();
CA::Render::Object::unref();
CA::Render::Encoder::ObjectCache::_lock;
CA::Render::Encoder::ObjectCache::_cache_list;
CA::Render::Encoder::set_object_cache();
CA::Render::Context::set_colorspace();
CA::Render::Encoder::grow();
CA::Render::Encoder::encode_colorspace();
CA::Render::Context::delete_object();
CA::Render::encode_delete_object();
CA::Context::commit_command();
CA::Layer::commit_if_needed();
CA::Context::commit_root();
_x_hash_table_foreach();
CA::Layer::collect_animations_();
CA::Render::Encoder::encode_int32();
CA::Render::Context::add_input_time();
CA::Render::Encoder::send_message();
```

目前已知的信息， 通过每次循环提交commit_transaction, 通知渲染中心， 但是这个频率如何控制，需要阅读，runloop以及  CA::DisplayLink源代码

CA::DisplayLink


__Bridge 究竟做了什么
