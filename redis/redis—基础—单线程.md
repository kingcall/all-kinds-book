
## 单线程
- ALL in-memory 100ns；
- 瓶颈不在CPU而在于network IO，可启多实例来提升CPU利用率；
- 避免多线程的上下文切换和竞态条件的开销，实现简单

## 高性能
- 绝大部分请求都是在内存中完成；
- 单线程，避免多线程带来的损耗；
- 多路复用非阻塞IO模型：epoll+AeEventLoop
