# 并发模型

业界将如何实现并发编程总结归纳为各式各样的并发模型，常见的并发模型有以下几种：

- 线程&锁模型
- Actor模型
- CSP模型
- Fork&Join模型

Go语言中的并发程序主要是通过基于 `CSP（communicating sequential processes）` 的 `goroutine` 和 `channel`来实现，当然也支持使用传统的多线程共享内存的并发方式。