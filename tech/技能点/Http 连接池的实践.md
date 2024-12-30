# 关于 Expire 和 Idle

**Expire**

连接过期时间，通常配置指定。当连接过期了，该连接会被连接池移除，并等待连接使用完毕后关闭连接。

定期关闭连接的好处是，避免连接出现如内存泄漏等引起性能下降问题。

**Idle**

空闲连接，指一段时间内没有被使用的连接，空闲连接可以被池内复用。

定期关闭空闲连接，可以提高资源的利用率。

**keep-alive:timeout=time**

配置了 Header `keep-alive:timeout=time`，可以不需要使用 closeIdleConnections。连接到期会关闭。

但是使用 `keep-alive` 时，需要记录连接开始时间，才能够到达指定时间后关闭连接。通常 Nginx 和连接池都有相关实现。

**连接复用**

如果连接不复用的情况下，每次都会创建一个新的连接，连接池相当于是一个信号量控制。

# 参数设置

如[最近学习了Http连接池 - 五月的仓颉 - 博客园](https://www.cnblogs.com/xrq730/p/10963689.html) 所述，连接池数量太小会阻塞连接导致 RT 变高。连接池数量太多，遇到高并发短时延的任务时，每次都要创建新的连接耗时，同时消耗资源也过高。

# 文档博客

[最近学习了Http连接池 - 五月的仓颉 - 博客园](https://www.cnblogs.com/xrq730/p/10963689.html)


# 选型

HttpClient 与 okHttp 中选择，趋向使用 okHttp。

**注意：** Spring RestTemplate 的默认实现是 HttpURLConnection，性能较差。可以通过 `RestTemplateBuilder builder` 实现为 HttpClient。也可以替换成 okhttp。其中 Spring Cloud 的 Feign 调用也是依赖于 RestTemplate。

[七大主流的HttpClient程序比较-阿里云开发者社区](https://developer.aliyun.com/article/1416617)
[Which Java HTTP client should I use in 2024?](https://www.wiremock.io/post/java-http-client-comparison)