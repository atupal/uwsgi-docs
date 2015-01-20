The uWSGI project
=================

The uWSGI project aims at developing a full stack for building hosting services.
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

核心（实现了配置，进程管理，创建 socket，监控，日志，共享内存，进程间通信，
集群成员和 :doc:`SubscriptionServer` ）

请求插件（实现了多种语言和平台的应用服务器接口： WSGI，PSGI，Rack，Lua WSAPI，CGI，PHP，Go ...）

网关（实现了负载均衡，代理和路由器）

The :doc:`Emperor <Emperor>` （实现了对大量实例的管理和监控）

循环引擎（实现了事件和并发，组件可以以 preforking，threaded，asynchronous/evented 和 
green thread/coroutine 模式运行。支持包括 uGreen，Greenlet，Stackless 多种技术，
:doc:`Gevent <Gevent>` , Coro::AnyEvent, :doc:`Tornado <Tornado>`, Goroutines 和 Fibers）

.. note::

  uWSGI 是一个发布周期非常快的活跃项目。所以代码和文档并不总是同步的。
  我们尽最大的努力来保证文档的质量，但这很难。请原谅。
  如果你遇到了麻烦，邮件列表是解决与 uWSGI 有关问题的最佳地方。
  欢迎为文档（以及代码）贡献。


Quickstarts
===========

.. toctree::
   :maxdepth: 1

   WSGIquickstart
   PSGIquickstart
   RackQuickstart
   Snippets


Table of Contents
=================

.. toctree::
   :maxdepth: 1

   Download
   Install
   BuildSystem
   Management
   LanguagesAndPlatforms
   SupportedPlatforms
   WebServers
   FAQ
   ThingsToKnow
   Configuration
   FallbackConfig
   ConfigLogic
   Options
   CustomOptions
   ParsingOrder
   Vars
   Protocol
   AttachingDaemons
   MasterFIFO
   Inetd
   Upstart
   Systemd
   Circus
   Embed
   Logging
   LogFormat
   LogEncoders
   Hooks
   Glossary
   ThirdPartyPlugins
   
Tutorials
=========

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
   
   

uWSGI Subsystems
================

.. toctree::
   :maxdepth: 1
   
   AlarmSubsystem
   Caching
   WebCaching
   Cron
   Fastrouter
   InternalRouting
   Legion
   Locks
   Mules
   OffloadSubsystem
   Queue
   RPC
   SharedArea
   Signals
   Spooler
   SubscriptionServer
   StaticFiles
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

Securing uWSGI
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


Keeping an eye on your apps
===========================

.. toctree::
   :maxdepth: 1

   Nagios
   SNMP
   PushingStats
   Carbon
   StatsServer
   Metrics


Async and loop engines
======================

.. toctree::
   :maxdepth: 1

   Async
   Gevent
   Tornado
   uGreen
   asyncio
   



Web Server support
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


Language support
==================
   
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

Other plugins
=============

.. toctree::
   :maxdepth: 1

   Pty
   SPNEGO
   LDAP


Broken/deprecated features
==========================

.. toctree::
   :maxdepth: 1

   Erlang
   ManagementFlag
   Go


Release Notes
=============

Stable releases
---------------

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
   
LTS releases
------------

.. toctree::
   :maxdepth: 1

   Changelog-1.4.10


Contact
=======

================== =
Mailing list       http://lists.unbit.it/cgi-bin/mailman/listinfo/uwsgi
Gmane mirror       http://dir.gmane.org/gmane.comp.python.wsgi.uwsgi.general
IRC                #uwsgi @ irc.freenode.org. The owner of the channel is `unbit`.
Twitter            http://twitter.com/unbit
Commercial support http://unbit.com/
================== =

.

Commercial support
==================

You can buy commercial support from http://unbit.com

Donate
======

uWSGI development is sponsored by the Italian ISP `Unbit <http://unbit.it/>`_ and its customers. You can buy commercial support and licensing. If you are not an Unbit customer, or you cannot/do not want to buy a commercial uWSGI license, consider making a donation. Obviously please feel free to ask for new features in your donation.

We will give credit to everyone who wants to sponsor new features.

See the `old uWSGI site <http://projects.unbit.it/uwsgi/#Donateifyouwant>`_ for the donation link. You can also donate via `GitTip <https://www.gittip.com/unbit/>`_.

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
