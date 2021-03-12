---
title:  "ch64: alternative io models"
categories: 
  - Linux
---

non-blocking io是基础，下边的几种模型，内部都是用nio进行的读写
nio就是 "return error" instead of "blocking"

## select/poll:level-triggered notification 
**虽然叫notification,其实是userspace的程序主动调用kernel api。signal-driven的（edge-triggered）才是kernel由主动通知userspace的程序**  
来自电路，high/low level, 表示的是电压的一个状态。对于fd来说就是可用/不可用，所以叫level-triggered, 因为只返回当前fd的状态  

用户代码主动调用select或poll请求一堆fd的状态，所以也叫io-multiplexing（同时请求了多个fd）。  
对于这n个fd需要一个个判断。是O(n)  
每次请求都要把这堆fd从userspace传给kernelspace, 即内核不会记忆用户代码之前select或poll了哪些fd

## signal-driven io:edge-triggered notification
是level transition,  falling edge 或 rising edge, 表示的电压的一个变化。类比到fd上，就是fd的状态发生变化。   
内核代码记住了fd列表，以及每个fd的owner是谁、fd状态变化时调用的handle是啥。是O(1)  

## epoll
同时支持level和edege形式的通知，event poll，  
内核也记录了fd列表（interest list 和 ready list）, 每当有io操作使fd状态变为ready时，内核就把fd加到ready list, 用户代码请求时内核直接返回readylist  


[关于level-triggered和edge-triggered](https://en.wikipedia.org/wiki/Interrupt)

## posix aio
socket io还是用signal-driven这种epoll实现，效率很好；posix aio是后来出来的，主要目的是解决asynchronous disk io  
POSIX AIO allows a process to queue an I/O operation to a file and then later be notified when the operation is complete.  
linux目前是在glibc中用userspace的线程实现的posix aio；kernel中有一个其他的方法是直接在driver层实现的aio  
参见[what-is-the-status-of-posix-asynchronous-i-o-aio](https://stackoverflow.com/questions/87892/what-is-the-status-of-posix-asynchronous-i-o-aio/5307557#5307557)
