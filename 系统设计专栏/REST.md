### 什么是RESTful API？

```text
GET    /classes：列出所有班级
POST   /classes：新建一个班级
```

**RESTful API 可以让你看到 URL+Http Method 就知道这个 URL 是干什么的，让你看到了 HTTP 状态码（status code）就知道请求结果如何。**

REST实际上就是 **“资源”在网络传输中以某种“表现形式”进行“状态转移”** 

1. 每一个 URI 代表一种资源；

2. 客户端和服务器之间，传递这种资源的某种表现形式比如 `json`，`xml`，`image`,`txt` 等等；

3. 客户端通过特定的 HTTP 动词，对服务器端资源进行操作，实现"表现层状态转化"。