# 作业题：网络编程实践题

## 题目 - 手写一个 Web 容器

**需求：工程师自定义一个 Web 容器提供给码农使用，码农只需要按照规定步骤，即可编写出自己的应用程序发布到 Web 容器**

### 01- 详细要求：

1. 要求：Web 容器支持发布静态资源到指定目录下
2. 要求：Web 容器支持动态资源 Servlet

### 02- 设计思路：

**Web 容器角色：**

- Web 容器开发者：工程师
- Web 容器使用者：码农
- 用户

**码农使用 Web 容器的步骤：**

- 码农编写应用程序：
  - 导入 Web 容器依赖坐标，并编写启动类
  - 将 index.html、js、image、css 等静态资源部署在 resources 目录下
  - 将自定义 Servlet 放置到指定包下：例如`com.hero.webapp`
- 码农发布自己的服务：
  - 码农将自己的接口 URL 按照固定规则发布：
    - 按照后缀，`.do`、`.action`、`无后缀`
  - 不管用何种规则：都将映射到自定义的 Servlet（类名映射，忽略大小写）举例

复制代码

```
   http://localhost:8080/aaa/bbb/userservlet?name=xiong
```

- 用户在访问应用程序：
  - 按照 URL 地址访问服务
  - 如果没有指定的 Servlet，则访问默认的 Servlet
  - 如果访问的是静态资源，则读取文件输出给浏览器

**工程师实现 Web 容器思路：**

- 第一步：创建 Web 容器工程，导入依赖坐标
- 第二步：复制 HeroCat 中的简版 Servlet 规范：HeroRequest、HeroResponse、HeroServlet
- 第三步：实现 Servlet 规范
  - HttpCustomRequest
  - HttpCustomResponse
  - DefaultCustomServlet
- 第四步：编写 Web 容器核心代码：
  - CustomServer 基于 Netty 实现：Servlet 容器
  - CustomHandler 处理请求，映射到 Servlet 的容器的自定义 Servlet 中去
- 第五步：打包发布 Web 容器
- 第六步：测试容器

​    

**代码位置为**：

https://github.com/pzsdev/home-work-for-java-lesson-code 地址 herocat 和 herocat-app 目录下。