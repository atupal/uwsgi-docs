Python/WSGI 应用快速入门
=======================

这个快速入门指南将会向你展示如何部署简单的 WSGI 应用和普通 web 框架。

Python 在这里特指 CPython，如果你想用 PyPy 你需要使用专门的插件: :doc:`PyPy`, 
Jython 的支持正在开发中。

.. note::

    为了完成这个快速入门确保你的 uWSGI 的版本在 1.4 以上。任何旧的东西
    都不会再维护并且使用它们是非常危险的。

安装带 Python 支持的 uWSGI
**************************

.. tip::

    当你开始学习 uWSGI 的时候，尝试从官方源代码构建：使用发行版提供的包可能会
    让你非常头疼。当事情变得明朗一点的时候，你可以使用模块化构建(就像在你的发行版中提供的一样)。

uWSGI 是一个(巨大的) C 应用，所以你需要一个 C 编译器(比如 gcc 或者 clang)和 Python 开发版头文件。

在 Debian 系的发行版上一条

.. code-block:: sh

   apt-get install build-essential python-dev

命令就够了。

你有多种方式来安装 uWSGI 的 Python 包：

* 使用 pip

  .. code-block:: sh

      pip install uwsgi

* 使用网络安装

  .. code-block:: sh

      curl http://uwsgi.it/install | bash -s default /tmp/uwsgi

  (这将会把 uWSGI 二进制文件安装到 ``/tmp/uwsgi`` 下，你可以随意修改它)。

* 通过下载源代码然后 make 安装

  .. code-block:: sh

      wget http://projects.unbit.it/downloads/uwsgi-latest.tar.gz
      tar zxvf uwsgi-latest.tar.gz
      cd <dir>
      make

  (make 完后你会在你的当前目录下得到一个 ``uwsig`` 的二进制文件)。

通过你的发行版的包管理器安装是不能面面俱到的(不可能让所有人都开心)，但是一般的规则都适用。

当你使用发行版提供的包来测试这个快速入门的时候，一件你可能想重视的事情就是很有可能
你的发行版是用模块化的方式构建的(每个特性都是一个不同的必须被加载的插件)。
为了完成这个快速入门，你必须在前面第一个例子的前面加上 ``--plugin python,http`` 选项，
以及当 HTTP 路由被移除时加上 ``--plugin python`` 选项(这可能对你没什么用，继续阅读就好)。

第一个 WSGI 应用
****************

让我们从一个简单的 "Hello World" 例子开始吧(这是在 Python 2.x 中，Python 3.x 需要
返回字节字符串，看下面)：

.. code-block:: python

   def application(env, start_response):
       start_response('200 OK', [('Content-Type','text/html')])
       return ["Hello World"]

(保存为 ``foobar.py``)。

正如你看到的，它由一个单独的 Python 函数组成。它的名字是 "application"，这是
默认的函数名，uWSGI 的 Python 加载器将会搜索这个名字(但你当然可以修改它)。

Python 3.x 版本如下：

.. code-block:: python

   def application(env, start_response):
       start_response('200 OK', [('Content-Type','text/html')])
       return [b"Hello World"]

把它部署到 HTTP 端口 9090 
*************************

现在运行 uWSGI 来启动一个会把请求传递给你的 WSGI 应用的 HTTP 服务器/路由器。

.. code-block:: sh

   uwsgi --http :9090 --wsgi-file foobar.py

这就是全部了。

.. note:: 当你有前端 web 服务器时不要使用 --http 选项，使用 --http-socket。继续阅读快速入门来理解为什么要这么做。

添加并发和监控
**************

你想做的第一件事可能就是增加并发(uWSGI 默认启动一个单独的进程和一个单独的线程)。

你可以通过 ``--processes`` 选项或者 ``--threads`` (或者两个选项都使用)来增加更多的进程或者线程。

.. code-block:: sh

   uwsgi --http :9090 --wsgi-file foobar.py --master --processes 4 --threads 2

这将会产生 4 个进程(每个进程 2 个线程)，一个主进程(当你的进程死掉时会重新 spawn 一个新的)以及 HTTP 路由器(见前面)。

一个重要的任何就是监控。知道发生了什么在生产环境中是极其重要的。stats 子系统允许你
用 JSON 输出 uWSGI 的内部数据：

.. code-block:: sh

   uwsgi --http :9090 --wsgi-file foobar.py --master --processes 4 --threads 2 --stats 127.0.0.1:9191

向你的应用发送几个请求然后 telnet 到 9191 端口，你将得到大量有趣的信息。你可能想要使用
"uwsgitop" (使用 ``pip install`` 你就能得到它)，这是一个类似 top 的工具，用于监控应用实例。

.. attention::

    将 stats 套接字(socket)绑定到私有地址(除非你知道你在做什么)，否则任何人都可以访问到它！

放到一个完整的 web 服务器后
***************************

即使 uWSGI HTTP 路由器(router)是一个可靠的高性能服务器，你可能还是想把你的应用放到一完整的 web 服务器后。

uWSGI 通常和 HTTP，FastCGI，SCGI 以及它自己特有的协议 "uwsgi" (呃，名字不应该这么取的) 通信。
性能最高的协议显然是 uwsgi，并且早已被 nginx 和 Cherokee 支持 (同时 Apache 也有许多可用的模块)。

一个普通的 nginx 配置如下：

.. code-block:: c

   location / {
       include uwsgi_params;
       uwsgi_pass 127.0.0.1:3031;
   }

这个意思是说 "把每个请求传递到服务器绑定的端口 3031，并且使用 uwsgi 协议通信"。

现在我们可以 spawn 一个 uWSGI 进程来天然地以 uwsgi 协议通信：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --wsgi-file foobar.py --master --processes 4 --threads 2 --stats 127.0.0.1:9191

如果你运行 ``ps aux`` ，你将会看到少了一个进程。HTTP 路由器(router)已经从我们的 "workers" (分配给 uWSGI 的进程)
中被移除了，这些 worker 便是天然地用来以 uwsgi 协议形式通信的。

如果你的代理/web 服务器/路由器使用 HTTP 协议，你必须告诉 uWSGI 使用 HTTP 协议(这与通过
--http spawn 一个它自己的代理是不一样的)：

.. code-block:: sh

   uwsgi --http-socket 127.0.0.1:3031 --wsgi-file foobar.py --master --processes 4 --threads 2 --stats 127.0.0.1:9191

开机自启动 uWSGI
****************

如果你打算打开 vi 写一个 init.d 脚本来启动 uWSGI，坐下来冷静一下然后先确保
你的系统没有提供一个更好(更现代化)的方式。

没一个发行版会选择一个启动系统 (:doc:`Upstart<Upstart>`, :doc:`Systemd`...)，除此之外也有许多
进程管理工具(supervisord, god, monit, circus...)。

uWSGI 与上面列出的那些工具都集成得很好(我们希望如此)，但是如果你想部署大量应用的话，
看看 uWSGI 的 :doc:`Emperor<Emperor>` - 它或多或少是每个开发运维工程师的梦想。

部署 Django
***********

Django 可能是使用得最多的 Python web 框架了。部署它非常简单(我们仍然使用 4 个进程，2 个线程的配置)。

假定你的 Django 项目在 ``/home/foobar/myproject`` 下：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --chdir /home/foobar/myproject/ --wsgi-file myproject/wsgi.py --master --processes 4 --threads 2 --stats 127.0.0.1:9191

(通过 ``--chdir`` 选项我们可以移动一个特定的目录)。在 Django 中为了正确的加载模块这是必须的。

啊！这是什么鬼？！是的，你是对的，你是对的。。。处理这么长的命令行是不实际的，又蠢又容易出错。
不要怕！ uWSGI 提供多种配置风格。在这个快速入门里我们将使用 .ini 文件。

.. code-block:: ini

    [uwsgi]
    socket = 127.0.0.1:3031
    chdir = /home/foobar/myproject/
    wsgi-file = myproject/wsgi.py
    processes = 4
    threads = 2
    stats = 127.0.0.1:9191

更好一点了！

尽管运行它：

.. code-block:: sh

   uwsgi yourfile.ini

如果 ``/home/foobar/myproject/myproject/wsgi.py`` (或者其他你的项目的名字) 这个文件不存在，你很有可能
使用的是老的版本的 Django (1.4 以下)。在这种情况下你需要配置更多一点的东西：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --chdir /home/foobar/myproject/ --pythonpath .. --env DJANGO_SETTINGS_MODULE=myproject.settings --module "django.core.handlers.wsgi:WSGIHandler()" --processes 4 --threads 2 --stats 127.0.0.1:9191

或者，使用 .ini 文件：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   chdir = /home/foobar/myproject/
   pythonpath = ..
   env = DJANGO_SETTINGS_MODULE=myproject.settings
   module = django.core.handlers.wsgi:WSGIHandler()
   processes = 4
   threads = 2
   stats = 127.0.0.1:9191

老版(1.4 以下)的 Django 发行版需要设置 ``evn``, ``module`` 和 ``pythonpath`` (``..`` 使得我们可以访问
``myproject.settings`` 模块)。


部署 Flask
**********

Flask 是一个流行的 Python web 微框架。

保存下面这个例子到 ``myflaskapp.py`` :

.. code-block:: python

   from flask import Flask

   app = Flask(__name__)

   @app.route('/')
   def index():
       return "<span style='color:red'>I am app 1</span>"

Flask 把它的 WSGI 函数(就是我们在之前快速入门里称作 "application" 即应用的东西)暴露成 "app", 所以
我们需要告诉 uWSGI 去使用它。
我们仍然使用 4 个进程/2 个线程，以及 uwsgi socket :

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --wsgi-file myflaskapp.py --callable app --processes 4 --threads 2 --stats 127.0.0.1:9191

(唯一增加的选项便是 ``--callable`` 选项)。

部署 web2py
***********

又是一个流行的选择。你可以选择把 web2py 的发行版源代码解压到一个目录然后写一个 uWSGI 配置文件：

.. code-block:: ini

   [uwsgi]
   http = :9090
   chdir = path_to_web2py
   module = wsgihandler
   master = true
   processes = 8

.. note::

    On recent web2py releases you may need to copy the ``wsgihandler.py`` script out of the ``handlers`` directory.

我们再次使用 HTTP 路由器(router)。用你的浏览器访问 9090 端口然后你就可以看到 web2py 的欢迎页面了。

点击管理页面然后...哎呀，它需要 HTTPS。不要担心，uWSGI 路由器(router)可支持 HTTPS (确保你
有 OpenSSL 开发版的头文件：安装它们然后重新构建 uWSGI，build 系统会自动检测到它)。

First of all generate your key and certificate:
首先生成你的秘钥(key)和证书(certificate)：

.. code-block:: sh

   openssl genrsa -out foobar.key 2048
   openssl req -new -key foobar.key -out foobar.csr
   openssl x509 -req -days 365 -in foobar.csr -signkey foobar.key -out foobar.crt

现在你有两个文件(算上 ``foobar.csr`` 的话就是三个了), ``foobar.key``  和 ``foobar.crt`` 。修改 uWSGI 配置：

.. code-block:: ini

   [uwsgi]
   https = :9090,foobar.crt,foobar.key
   chdir = path_to_web2py
   module = wsgihandler
   master = true
   processes = 8

重新运行 uWSGI 然后使用 ``https://`` 用你的浏览器连接到 9090 端口。

Python 线程小贴士
*****************

如果你没有使用线程启动 uWSGI，Python 的 GIL 将不会被开启，所以你的应用产生的线程
将永远不会运行。你可能不会喜欢这个选择，但是记住 uWSGI 是一个语言无关的服务器，所以它的
大部分选择都是尽可能维持它 "agnostic"。

但是不用担心，基本上不存在不能通过选项来改变的由 uWSGI 开发者决定的选项。

如果你想维持 Python 的线程支持同时应用又不启动多个线程，只需要加上
``--enable-threads`` 选项 (或者 ``enable-threads = true`` 在 ini 风格配置文件中)。

Virtualenvs
***********

uWSGI 可以被配置成在某个特定的 virtualenv 中搜索 Python 模块。

只要添加 ``virtualenv = <path>`` 到你的选中中就可以了。

安全和可用性
************

**永远** 不要使用 root 来运行 uWSGI 实例。你可以用 ``uid`` 和 ``gid`` 选项来降低权限：

.. code-block:: ini

   [uwsgi]
   https = :9090,foobar.crt,foobar.key
   uid = foo
   gid = bar
   chdir = path_to_web2py
   module = wsgihandler
   master = true
   processes = 8

如果你需要绑定到一个特权端口(比如 HTTPS 的443)，使用共享套接字(shared sockets)。它们在权限降低之前被创建，可以
使用 ``=N`` 语法来引用，这里的 ``N`` 指 socket 编号(从0开始)：

.. code-block:: ini

   [uwsgi]
   shared-socket = :443
   https = =0,foobar.crt,foobar.key
   uid = foo
   gid = bar
   chdir = path_to_web2py
   module = wsgihandler
   master = true
   processes = 8

web 应用开发一个最常见的问题就是 "stuck requests"(卡住的请求)。你所有的线程/worker 都被卡住(被请求堵塞)，
然后你的应用再也不能接受更多的请求。
为了避免这个问题你可以设置一个 ``harakiri`` 计时器。它是一个监视器(由主进程管理)，当
进程被卡住的时间超过特定的秒数后就销毁这个进程(慎重选择 ``harakiri`` 的值)。比如，你可能
想把卡住超过 30 秒的 worker 销毁掉：

.. code-block:: ini

   [uwsgi]
   shared-socket = :443
   https = =0,foobar.crt,foobar.key
   uid = foo
   gid = bar
   chdir = path_to_web2py
   module = wsgihandler
   master = true
   processes = 8
   harakiri = 30

另外，从 uWSGI 1.9 起，统计服务器会输出所有的请求变量，所以你可以(实时地)查看你的
实例在干什么(对于每个 worker，线程或者异步 core)。


Offloading
**********

:doc:`OffloadSubsystem` 使得你可以在某些模式满足时释放你的 worker，并且把工作委托给一个纯 c 的线程。
这样例子比如有从文件系统传递静态文件，通过网络向客户端传输数据等等。

Offloading 非常复杂，但它的使用对用户来说是透明的。如果你想试试的话加上 ``--offload-threads <n>`` 
选项，这里的 <n> 是 spawn 的线程数(以 CPU 数目的线程数启动是一个不错的值)。

当 offload threads 被启用时，所有可以被优化的部分都可以自动被检测到。

Bonus: 多版本 Python 使用同一个 uWSGI 二进制文件
************************************************

正如我们已经看到的，uWSGI 由一个很小的核心和许多插件组成。插件可以被嵌入到二进制文件中
或者动态加载。当你为 Python 构建 uWSGI 的时候，许多插件包括 Python 在内的插件都被嵌入到了最终的二进制文件中。

当你使用多个 Python 版本但是没有为每一个版本构建一个二进制文件时这可能会造成问题。

最好的方法可能是弄一个没有内置语言特性的小二进制文件，然后每个 Python 版本有一个
插件，可以动态地加载。

在 uWSGI 的源代码目录中：

.. code-block:: sh

   make PROFILE=nolang
   
这将会构建一个包含除了 Python 之外的所有默认内置插件的 uwsgi 二进制文件。

现在，在相同的目录下，我们开始构建 Python 插件：

.. code-block:: sh

   PYTHON=python3.4 ./uwsgi --build-plugin "plugins/python python34"
   PYTHON=python2.7 ./uwsgi --build-plugin "plugins/python python27"
   PYTHON=python2.6 ./uwsgi --build-plugin "plugins/python python26"

你最后会得到这些文件： ``python34_plugin.so``, ``python27_plugin.so``, ``python26_plugin.so`` 。复制
这些文件到你的目录中。(uWSGI 默认在当前的工作目录中搜索插件。)

现在你只需要在你的配置文件中(在文件最上面)简单加上 `plugins-dir` 和 `plugin` 选项就可以了。

.. code-block:: ini

   [uwsgi]
   plugins-dir = <path_to_your_plugin_directory>
   plugin = python26
   
这将会从你复制插件到的那个目录中加载 ``python26_plugin.so`` 插件。

那么现在...
***********

有了这些很少的概念你就已经可以进入到生产中了，但是 uWSGI 是一个拥有上百个特性和配置的生态系统。
如果你想成为一个更好的系统管理员，继续阅读完整的文档吧。
