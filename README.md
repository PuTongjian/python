# Python编码规范
- [PEP 8规范](https://github.com/PuTongjian/python-stack/blob/master/doc/PEP%208.md)
- [Python语言规范 — Google 开源项目风格指南](https://github.com/PuTongjian/python-stack/blob/master/doc/Python%E8%AF%AD%E8%A8%80%E8%A7%84%E8%8C%83%5BGoogle%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE%E9%A3%8E%E6%A0%BC%E6%8C%87%E5%8D%97%5D.pdf)
- [Python编程惯例 — 十分钟让你的代码更加Pythonic](https://github.com/PuTongjian/python-stack/blob/master/doc/Python%E7%BC%96%E7%A8%8B%E6%83%AF%E4%BE%8B.pdf)

---

# 一些你一定要知道的常识
- [TCP/IP协议栈](https://developer.51cto.com/art/201906/597961.htm)
- [HTTPS的故事——浅显易懂](https://juejin.im/post/5a66ba596fb9a01cb64eed6f)
- [HTTP验证大法(Basic Auth,Session, JWT, Oauth, Openid)](https://segmentfault.com/a/1190000008481722)
- [RESTful API](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)
- [I/O模式详解](https://github.com/PuTongjian/python-stack/blob/master/doc/IO%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3.md)
- [I/O多路复用之select、poll、epoll](https://www.cnblogs.com/aspirant/p/9166944.html)

---

# 人生苦短，我用Python
## Python指南
- [Python魔法方法指南.pdf](https://github.com/PuTongjian/python-stack/blob/master/doc/Python%E9%AD%94%E6%B3%95%E6%96%B9%E6%B3%95%E6%8C%87%E5%8D%97.pdf)
- [Python装饰器的理解](https://github.com/PuTongjian/python-stack/blob/master/doc/Python%E8%A3%85%E9%A5%B0%E5%99%A8%E7%9A%84%E7%90%86%E8%A7%A3.md)

## 轮子
- [Requests](https://cn.python-requests.org/zh_CN/latest/)  Requests唯一的一个非转基因的 Python HTTP 库，人类可以安全享用
- [AIOHTTP](https://www.cntofu.com/book/127/readme.html)  提供异步HTTP客户端/服务端编程，基于asyncio的异步库

---

# Web 应用框架
## Flask
### 参考
- [Flask 文档（中文版）](https://dormousehole.readthedocs.io/en/latest/)
- [SQLAlchemy实现读写分离](https://github.com/PuTongjian/python/blob/master/doc/SQLAlchemy%E5%AE%9E%E7%8E%B0%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB.md)
- [Flask-SQLAlchemy分表分库](http://www.pythondoc.com/flask-sqlalchemy/binds.html)
### Flask插件
- [Flask-Login](http://www.pythondoc.com/flask-login/)  使用Cookie进行用户会话管理
- [Flask-Principal](https://flask-principal-cn.readthedocs.io/zh_CN/latest/)  主要的组件包括身份(Identity)，需求(Needs)，权限(Permission)，和身份上下文(IdentityContext)
- [Flask-JWT-Extended](https://flask-jwt-extended.readthedocs.io/en/latest/) 实现基于Json Web Token的用户认证授权
- [Flask-HTTPAuth](https://flask-httpauth.readthedocs.io/en/latest/)  封装了以几种简单的认证方式：基本认证（Basic Authentication），摘要认证（Digest Authentication），标志认证（Token Authentication）
- [SQLAlchemy](https://www.osgeo.cn/sqlalchemy/) ORM框架，帮助你实现数据持久化
- [Flask-SQLAlchemy](http://www.pythondoc.com/flask-sqlalchemy/)  基于SQLAlchemy（ORM框架）的扩展，支持MySQL，PostgreSQL和Oracle
- [Flask-RESTful](http://www.pythondoc.com/Flask-RESTful/index.html)  帮助你快速构建RESTful API
- [Flask-Caching](https://pythonhosted.org/Flask-Caching/)  它为任何Flask应用程序增加了对各种后端的缓存支持
- [Flask-Marshmallow ](https://flask-marshmallow.readthedocs.io/en/latest/)  序列化和反序列化
- [Flask-Limiter](https://flask-limiter.readthedocs.io/en/stable/)  它为Flask路由提供速率限制功能。它支持使用内存，redis和memcache的当前实现来存储的可配置后端
- [Flask-Migrate](https://flask-migrate.readthedocs.io/en/latest/)  它是用于对Flask程序中的SQLAlchemy进行数据库的迁移
- [flask-wtf](http://www.pythondoc.com/flask-wtf/)  基于WTForms的扩展，用于参数验证
- [Flask-Cors](https://flask-cors.readthedocs.io/en/latest/)  用于处理跨源资源共享，使得AJAX跨域请求成为可能
- [Flask-Mail](https://pythonhosted.org/Flask-Mail/)  将发送邮件功能集成到你的应用程序
- [Flask-SocketIO](https://flask-socketio.readthedocs.io/en/latest/)  Flask-SocketIO为flask应用提供了一个客户端与服务器之间低延迟的双向通信

以上是Flask应用程序一些常用的优秀插件（持续更新）


## Tornado
### 参考
- [Tornado文档（英文）](http://www.tornadoweb.org/en/stable/)  可参考中文文档阅读
- [Tornode文档（中文）](https://tornado-zh.readthedocs.io/zh/latest/)  更新至4.3
### 轮子
- [Tornado-Redis](https://github.com/leporo/tornado-redis)  它为Tornado应用程序提供Redis缓存服务
- [Peewee](https://www.osgeo.cn/peewee/)  种简单而小的ORM框架
- [peewee-async](https://peewee-async.readthedocs.io/en/latest/)  基于Peewee实现的异步ORM框架
- [WTForms](https://wtforms.readthedocs.io/en/stable/) 表单参数验证

## Sanic | Build fast. Run fast.
### 参考
- [Sanic文档（中文）](https://sanic.dev/zh/)
---
