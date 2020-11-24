---
title: 前后端分离vue+springboot所遇到的跨域问题
comments: true
tags:
  - CORS
  - JAVA
categories:
  - [后台, JAVA]
date: 2020-11-24 20:11:13
updated: 2020-11-24 20:11:13
keywords: CORS, JAVA, 跨域
---
# 跨域
这是我第二次遇到此问题，特此记录解决方法。
跨域资源共享(CORS) 是一种机制，它使用额外的 HTTP 头来告诉浏览器  让运行在一个 origin (domain) 上的Web应用被准许访问来自不同源服务器上的指定的资源。当一个资源从与该资源本身所在的服务器不同的域、协议或端口请求一个资源时，资源会发起一个跨域 HTTP 请求。
<!-- more -->
比如，站点 http://domain-a.com 的某 HTML 页面通过 


出于安全原因，浏览器限制从脚本内发起的跨源HTTP请求。 例如，XMLHttpRequest和Fetch API遵循同源策略。 这意味着使用这些API的Web应用程序只能从加载应用程序的同一个域请求HTTP资源，除非响应报文包含了正确CORS响应头。 

（译者注：这段描述不准确，并不一定是浏览器限制了发起跨站请求，也可能是跨站请求可以正常发起，但是返回结果被浏览器拦截了。）

跨域资源共享（CORS ）机制允许 Web 应用服务器进行跨域访问控制，从而使跨域数据传输得以安全进行。现代浏览器支持在 API 容器中（例如 XMLHttpRequest 或Fetch）使用 CORS，以降低跨域 HTTP 请求所带来的风险。

摘自：https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS

# 实现
## 通过编写中间件（过滤器、适配器）

这里我通过滤器实现：

```
package com.example.redisdemo.config;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletResponse;

import org.springframework.stereotype.Component;

@Component
public class CorsFilter implements Filter {

    /*跨域请求配置*/
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        HttpServletResponse response = (HttpServletResponse) res;
        response.setHeader("Access-Control-Allow-Origin","*");
        response.setHeader("Access-Control-Allow-Credentials", "true");
        response.setHeader("Access-Control-Allow-Methods", "POST, PUT, GET, OPTIONS, DELETE");
        response.setHeader("Access-Control-Max-Age", "5000");
        response.setHeader("Access-Control-Expose-Headers", "Token");
        response.setHeader("Access-Control-Allow-Headers", "Origin, No-Cache, X-Requested-With, If-Modified-Since, Pragma, Last-Modified, Cache-Control, Expires, Content-Type, X-E4M-With,Authorization,Token");
        chain.doFilter(req, res);
    }
    @Override
    public void init(FilterConfig filterConfig) {}
    @Override
    public void destroy() {}
}
```
从该过滤器可看到，它将请求的响应体进行了部分设置，包括：
Access-Control-Allow-Origin：访问源控制
Access-Control-Allow-Credentials：是否返回cookie
Access-Control-Allow-Methods：处理发送请求方式
Access-Control-Expose-Headers：允许暴露给前端的属性
Access-Control-Allow-Headers：允许前端访问携带的属性

## 通过Spring注解的方式

为请求handler或controller添加@CrossOrigin允许跨源请求注解，默认允许所有的域以及允许所有的header。

详见：[https://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html](https://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html)