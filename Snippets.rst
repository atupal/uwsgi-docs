代码片段
========

这是一些最"有趣"的 uWSGI 特性使用的合集。

X-Sendfile emulation
--------------------

甚至你的前端代理/web 服务器不支持 X-Sendfile (或者不能访问到你的静态资源)你可以使用 uWSGI 内部的
offloading(你的进程/线程把服务静态文件的实际工作交给 offload 线程) 来模拟它。

.. code-block:: ini

   [uwsgi]
   ...
   ; load router_static plugin (compiled in by default in monolithic profiles)
   plugins = router_static
   ; spawn 2 offload threads
   offload-threads = 2
   ; files under /private can be safely served
   static-safe = /private
   ; collect the X-Sendfile response header as X_SENDFILE var
   collect-header = X-Sendfile X_SENDFILE
   ; if X_SENDFILE is not empty, pass its value to the "static" routing action (it will automatically use offloading if available)
   response-route-if-not = empty:${X_SENDFILE} static:${X_SENDFILE}
   

强制 HTTPS
----------

这会强制在你的整个网站上使用 HTTPS。

.. code-block:: ini

   [uwsgi]
   ...
   ; load router_redirect plugin (compiled in by default in monolithic profiles)
   plugins = router_redirect
   route-if-not = equal:${HTTPS};on redirect-permanent:https://${HTTP_HOST}${REQUEST_URI}
   
只是对 ``/admin`` 强制 https：

.. code-block:: ini

   [uwsgi]
   ...
   ; load router_redirect plugin (compiled in by default in monolithic profiles)
   plugins = router_redirect
   route = ^/admin goto:https
   ; stop the chain
   route-run = last:
   
   route-label = https
   route-if-not = equal:${HTTPS};on redirect-permanent:https://${HTTP_HOST}${REQUEST_URI}
   
最后你可能还想要发送 HSTS(HTTP Strict Transport Security) http 头。

.. code-block:: ini

   [uwsgi]
   ...
   ; load router_redirect plugin (compiled in by default in monolithic profiles)
   plugins = router_redirect
   route-if-not = equal:${HTTPS};on redirect-permanent:https://${HTTP_HOST}${REQUEST_URI}
   route-if = equal:${HTTPS};on addheader:Strict-Transport-Security: max-age=31536000
   
   
Python 自动重新加载(Python auto-reloading)(仅限于在开发中使用！)
----------------------------------------------------------------

在生产环境中你可以检测文件/目录的改动，然后自动重新加载(touch-reload, fs-reload...)。

在开发的时候有一个检测所有加载的/使用的 python 模块改动会非常方便。但是请仅仅在开发过程
中使用它。

检测是通过一个线程以设定的频率扫描模块列表实现的：

.. code-block:: ini

   [uwsgi]
   ...
   py-autoreload = 2
   
这将会以每隔两秒的频率检测 python 模块的改动，然后有改动的话就重新启动实例。

再次说明：

.. warning:: 只能在开发中使用它，不要在线上环境使用。


Full-Stack CGI setup
--------------------

This example spawned from a uWSGI mainling-list thread.
这个例子产生自一个 uWSGI 邮件列表。

我的静态文件在 /var/www 目录下，cgi 在 /var/cgi 下，Cgi 通过 /cgi-bin 路径可以访问到。
所以 /var/cig/foo.lua 会在访问 /cgi-bin/foo.lua 时运行。

.. code-block:: ini

   [uwsgi]
   workdir = /var
   ipaddress = 0.0.0.0
 
   ; start an http router on port 8080
   http = %(ipaddress):8080
   ; enable the stats server on port 9191
   stats = 127.0.0.1:9191
   ; spawn 2 threads in 4 processes (concurrency level: 8)
   processes = 4
   threads = 2
   ; drop privileges
   uid = nobody
   gid = nogroup
   
   ; serve static files in /var/www
   static-index = index.html
   static-index = index.htm
   check-static = %(workdir)/www
   
   ; skip serving static files ending with .lua
   static-skip-ext = .lua

   ; route requests to the CGI plugin
   http-modifier1 = 9
   ; map /cgi-bin requests to /var/cgi
   cgi = /cgi-bin=%(workdir)/cgi
   ; only .lua script can be executed
   cgi-allowed-ext = .lua
   ; .lua files are executed with the 'lua' command (it avoids the need of giving execute permission to files)
   cgi-helper = .lua=lua
   ; search for index.lua if a directory is requested
   cgi-index = index.lua
   
   
在不同的 url 路径下使用多个 flask 应用
--------------------------------------

让我们写三个 flask 应用：

.. code-block:: py

   #app1.py
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def hello():
       return "Hello World! i am app1"
       

.. code-block:: py

   #app2.py
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def hello():
       return "Hello World! i am app2"
       
       
.. code-block:: py

   #app3.py
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def hello():
       return "Hello World! i am app3"

每个会被相应地挂载到 /app1, /app2, /app3

在 uWSGI 中要把一个应用挂载到一个特定的"key"，需要使用 --mount 选项：

```
--mount <mountpoint>=<app>
```

在我们的例子中我们想要挂载三个 python 应用，每一个以相应的 WSGI 脚本名字作为 key：

.. code-block :: ini
   
   [uwsgi]
   plugin = python
   mount = /app1=app1.py
   mount = /app2=app2.py
   mount = /app3=app3.py
   ; generally flask apps expose the 'app' callable instead of 'application'
   callable = app

   ; tell uWSGI to rewrite PATH_INFO and SCRIPT_NAME according to mount-points
   manage-script-name = true

   ; bind to a socket
   socket = /var/run/uwsgi.sock



现在直接把你的 webserver.proxy 指向你的实例 socket (不需要任何其他的配置)

Note: 每个应用默认会启动一个新的 python 解释器(这意味着每个应用的名字空间是相互隔离的)。
如果你希望所有的应用都运行同一个 python 虚拟机上的话，使用 --single-interpreter 选项。

Another note: 你可能已经看到 "modifier1 30" 这个明显的陷阱了。它已经被弃用了，而且它相当丑陋。uWSGI 有许多的方式来重写请求的变量。

Final note: 第一个加载的应用默认为是缺省挂载应用。当没有挂载点匹配时那个应用便会起作用。


在 OSX 上使用 rbenv (也应该能在其他的平台上工作)
-------------------------------------------------

安装 rbenv

.. code-block:: sh

   brew update
   brew install rbenv ruby-build
   
(不要在 .bash_profile 中设置 magic line，因为我们不想污染系统环境并且导致 uWSGI 异常)

获取一个 uWSGI 源码包，然后编译成 'nolang' 版本(即一个没有编译任何语言插件进去的版本)

.. code-block:: sh

   wget http://projects.unbit.it/downloads/uwsgi-latest.tar.gz
   tar zxvf uwsgi-latest.tar.gz
   cd uwsgi-xxx
   make nolang
   
现在开始安装你需要的 ruby 版本

.. code-block:: sh

   rbenv install 1.9.3-p551
   rbenv install 2.1.5
   
然后安装你需要的 gems(即 sinatra):

.. code-block:: sh

   # set the current ruby env
   rbenv local 1.9.3-p551
   # get the path of the gem binary
   rbenv which gem
   # /Users/roberta/.rbenv/versions/1.9.3-p551/bin/gem
   /Users/roberta/.rbenv/versions/1.9.3-p551/bin/gem install sinatra
   # from the uwsgi sources directory, build the rack plugin for 1.9.3-p551, naming it rack_193_plugin.so
   # the trick here is changing PATH to find the right ruby binary during the build procedure
   PATH=/Users/roberta/.rbenv/versions/1.9.3-p551/bin:$PATH ./uwsgi --build-plugin "plugins/rack rack_193"
   # set ruby 2.1.5
   rbenv local 2.1.5
   rbenv which gem
   # /Users/roberta/.rbenv/versions/2.1.5/bin/gem
   /Users/roberta/.rbenv/versions/2.1.5/bin/gem install sinatra
   PATH=/Users/roberta/.rbenv/versions/2.1.5/bin:$PATH ./uwsgi --build-plugin "plugins/rack rack_215"
   
现在切换到另外一个 ruby，只需要改变插件就可以了：

.. code-block:: ini

   [uwsgi]
   plugin = rack_193
   rack = config.ru
   http-socket = :9090
   
或者

.. code-block:: ini

   [uwsgi]
   plugin = rack_215
   rack = config.ru
   http-socket = :9090

请确保插件存储在当前的工作目录中，或者直接设置插件目录，或者指定绝对路径，就像这样：

.. code-block:: ini

   [uwsgi]
   plugin = /foobar/rack_215_plugin.so
   rack = config.ru
   http-socket = :9090
