The uWSGI project
=================

uWSGI 项目致力于为构建一个全栈式的托管服务。

应用服务器（多种编程语言和协议），代理，进程管理器和监视器
全部都以通用 api 和通用配置风格实现了。

得益于它的可插式架构，它可以被拓展到其他更多的平台和语言。

目前你可以使用 C，C++ 和 Objective-C 来写插件。

名字中的 ”WSGI“ 部分是对 Python 标准中的同名一个东西的致敬，因为它
是这个项目的第一个开发的插件。

多功能，高性能，占用资源少和可靠性是这个项目的优势（也是唯一遵循的规则）。

包含的组件（更新到了最新的稳定发行版）
======================================================

核心 Core （实现了配置，进程管理，创建 socket，监控，日志，共享内存，进程间通信，
集群成员和 :doc:`SubscriptionServer` ）

请求插件 Request plugins （实现了多种语言和平台的应用服务器接口： WSGI，PSGI，Rack，Lua WSAPI，CGI，PHP，Go ...）

网关 Gateways （实现了负载均衡，代理和路由器）

The :doc:`Emperor <Emperor>` （实现了对大量实例的管理和监控）

循环引擎 Loop engines （实现了事件和并发，组件可以以 preforking，threaded，asynchronous/evented 和 
green thread/coroutine 模式运行。支持包括 uGreen，Greenlet，Stackless 多种技术，
:doc:`Gevent <Gevent>` , Coro::AnyEvent, :doc:`Tornado <Tornado>`, Goroutines 和 Fibers）

.. note::

  uWSGI 是一个发布周期非常快的活跃项目。所以代码和文档并不总是同步的。
  我们尽最大的努力来保证文档的质量，但这很难。请原谅。
  如果你遇到了麻烦，邮件列表是解决与 uWSGI 有关问题的最佳地方。
  欢迎为文档（以及代码）贡献。


快速入门
========

.. toctree::
   :maxdepth: 1

   WSGIquickstart
   PSGIquickstart
   RackQuickstart
   Snippets


目录表
======

.. toctree::
   :maxdepth: 1

   下载
   安装
   构建系统
   管理
   语言和平台
   支持的平台
   Web 服务
   FAQ(常见问题和答案)
   需要知道的事
   配置
   Fallback 配置
   配置逻辑
   选项
   定制选项
   解析顺序
   变量
   协议
   守护进程
   MasterFIFO
   超级守护进程(Inetd)
   启动(Upstart)
   Systemd
   Circus
   嵌入
   日志
   日志格式
   日志编码
   钩子
   术语表
   第三方插件
   
教程
====

.. toctree::
   :maxdepth: 1

   tutorials/CachingCookbook
   tutorials/Django_and_nginx
   tutorials/dreamhost
   tutorials/heroku_python
   tutorials/heroku_ruby
   tutorials/ReliableFuse
   tutorials/DynamicProxying
   tutorials/GraphiteAndMetrics
   

Articles
========

.. toctree::
   :maxdepth: 1

   articles/SerializingAccept
   #articles/MassiveHostingWithEmperorAndNamespaces
   articles/TheArtOfGracefulReloading
   articles/FunWithPerlEyetoyRaspberrypi
   articles/OffloadingWebsocketsAndSSE
   
   

uWSGI 子系统
============

.. toctree::
   :maxdepth: 1
   
   报警系统
   缓存
   Web 缓存
   计划任务(Cron)
   快速路由(Fastrouter)
   内部路由(InternalRouting)
   Legion
   锁
   Mules
   卸载子系统(OffloadSubsystem)
   队列
   远程过程调用(RPC)
   共享区域(SharedArea)
   信号
   Spooler
   订阅服务器
   静态文件
   SNI
   GeoIP
   Transformations
   WebSockets
   Metrics
   Chunked

Scaling with uWSGI
==================

.. toctree::
   :maxdepth: 1

   Cheaper
   Emperor
   Broodlord
   Zerg
   DynamicApps
   SSLScaling

让 uWSGI 更安全
==============

.. toctree::
   :maxdepth: 1

   Capabilities
   Cgroups
   KSM
   Namespaces
   FreeBSDJails
   ForkptyRouter
   TunTapRouter


盯着你的应用(Keeping an eye on your apps)
=========================================

.. toctree::
   :maxdepth: 1

   Nagios
   SNMP
   PushingStats
   Carbon
   StatsServer
   Metrics


异步和循环引擎 (Async and loop engines)
=======================================

.. toctree::
   :maxdepth: 1

   Async
   Gevent
   Tornado
   uGreen
   asyncio
   



支持的 Web 服务器
==================
   
.. toctree::
   :maxdepth: 1
 
   Apache
   Cherokee
   HTTP
   HTTPS
   SPDY
   Lighttpd
   Mongrel2
   Nginx


语言支持
========
   
.. toctree::
   :maxdepth: 2
 
   Python
   PyPy
   PHP
   Perl
   Ruby
   Lua
   JVM
   Mono
   CGI
   GCCGO
   Symcall
   XSLT
   SSI
   V8
   GridFS
   GlusterFS
   Rados

其他插件
========

.. toctree::
   :maxdepth: 1

   Pty
   SPNEGO
   LDAP


弃用(Broken/deprecated)特性
==========================

.. toctree::
   :maxdepth: 1

   Erlang
   ManagementFlag
   Go


发布说明
========

稳定版
------

.. toctree::
   :maxdepth: 1

   Changelog-2.0.9
   Changelog-2.0.8
   Changelog-2.0.7
   Changelog-2.0.6
   Changelog-2.0.5
   Changelog-2.0.4
   Changelog-2.0.3
   Changelog-2.0.2
   Changelog-2.0.1
   Changelog-2.0
   Changelog-1.9.21
   Changelog-1.9.20
   Changelog-1.9.19
   Changelog-1.9.18
   Changelog-1.9.17
   Changelog-1.9.16
   Changelog-1.9.15
   Changelog-1.9.14
   Changelog-1.9.13
   Changelog-1.9.12
   Changelog-1.9.11
   Changelog-1.9.10
   Changelog-1.9.9
   Changelog-1.9.8
   Changelog-1.9.7
   Changelog-1.9.6
   Changelog-1.9.5
   Changelog-1.9.4
   Changelog-1.9.3
   Changelog-1.9.2
   Changelog-1.9.1
   Changelog-1.9
   
长期支持版(LTS)
---------------

.. toctree::
   :maxdepth: 1

   Changelog-1.4.10


联系信息
=======

================== =
Mailing list       http://lists.unbit.it/cgi-bin/mailman/listinfo/uwsgi
Gmane mirror       http://dir.gmane.org/gmane.comp.python.wsgi.uwsgi.general
IRC                #uwsgi @ irc.freenode.org. The owner of the channel is `unbit`.
Twitter            http://twitter.com/unbit
Commercial support http://unbit.com/
================== =

.

商业支持
========

你可以从 http://unbit.com 购买商业支持

捐助
====

uWSGI 的开发由意大利互联网服务提供商 `Unbit <http://unbit.it/>`_ 以及它的客户
支持。你可以购买商业支持和许可。如果你不是 Unbit 的客户或者你不想购买一个商业的
uWSGI 许可，你可以考虑捐助。显然你可以在你的捐助中随意询问想要的新特性。

我们将会把支持开发新特性的人加到 credit 里。

请看 `old uWSGI site <http://projects.unbit.it/uwsgi/#Donateifyouwant>`_  来获取捐助链接。
你可以通过 `GitTip <https://www.gittip.com/unbit/>`_ 捐助。


索引和查询
==========

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
