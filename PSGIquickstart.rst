perl/PSGI 应用快速入门
======================


下面的说明将会引导你安装运行一个基于 perl 的目的在于运行 PSGI apps 的 uWSGI 发行版。


安装带 Perl 支持的 uWSGI
************************

为了构建 uWSGI 你需要一个 c 编译器(gcc 和 clang 都支持)以及 python 二进制文件(它只会用于运行 uwsgiconfig.py
脚本来执行一些比较复杂的步骤)。
为了构建带有 perl 支持的 uWSGI 二进制文件我们也需要 perl 开发版的头文件(在 debian 系发行版上是
libperl-dev 这个包)。

你可以手动构建 uWSGI：

.. code-block:: sh

   python uwsgiconfig.py --build psgi
   
这和下面一样：


.. code-block:: sh

   UWSGI_PROFILE=psgi make
   
或者使用网络安装：

.. code-block:: sh

   curl http://uwsgi.it/install | bash -s psgi /tmp/uwsgi
   
这将会在 /tmp/uwsgi (你可以随便改变成你想要的路径) 目录下创建一个 uWSGI 二进制文件。

使用发行版包需要注意的地方
**************************

你的发行版很有可能已经包含了一个 uWSGI 的包集合。这些 uWSGI 包趋向于高度模块化，
所以除了 core 你还需要安装需要的插件。
在你的配置中插件必须被加载。在学习阶段我们强烈建议不要使用发行版的包，简单地跟着文档和教程走就可以了。

一旦你对 "uWSGI 方式" 感到习惯了，你可以为你的部署选择最好的途径。

你的第一个 PSGI 应用
*******************

把它以 myapp.pl 文件名保存

.. code-block:: pl

   my $app = sub {
        my $env = shift;
        return [
                '200',
                [ 'Content-Type' => 'text/html' ],
                [ "<h1>Hello World</h1>" ],
        ];
   };

然后以 http 模式运行 uWSGI：

.. code-block:: sh

   uwsgi --http :8080 --http-modifier1 5 --psgi myapp.pl

(如果 uwsgi 不在你当前的 $PATH 里的话记着替换它)

或者如果你使用了模块化安装(比如你的发行版里的包)

.. code-block:: sh

   uwsgi --plugins http,psgi --http :8080 --http-modifier1 5 --psgi myapp.pl
   
.. note:: 当你有一个前段 web 服务器的时候不要使用 -http 选项，使用 --http-socket。继续阅读这个快速入门你就会明白为什么要这么做

'--http-modifier1 5' 是什么鬼？？？
**********************************

uWSGI 支持多种语言和平台。当服务器收到一个请求时它必须知道“路由”它到哪里去。

每一个 uWSGI 插件都有一个分配的数字(modifier)，perl/psgi 的数字是 5。所以 --http-modifier1 5 
表示“路由到 psgi 插件”。

虽然 uWSGI 有一个更“友好”的 :doc:`internal routing system <InternalRouting>` ，但使用
modifier 仍然是最快的方式，所以尽可能地使用他们。


使用一个完整的 web 服务器：nginx
********************************

提供的 http 路由器仅仅就是一个路由器(是的，难以置信)。你可以使用它作为负载均衡器或者代理，
但是如果你需要一个完整的 web 服务器(比如为了高性能地提供静态文件访问或者那些 web 服务器更适合的工作)，
使用 uwsig http 路由器有风险(记住把 --plugins http,psgi 改成 --plugins psgi 如果你是模块化安装的话)，
你应该把你的应用放在 nginx 后面。

为了和 nginx 通信，uWSGI 可以使用多种协议：http，uwsgi，fastcgi，scgi...

性能最高的是 uwsgi。Nginx 提供了开箱即用的 uwsgi 协议支持。

使用 uwsgi socket 运行你的 psgi 应用：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --psgi myapp.pl

然后在你的 nginx 配置中加一个 location 节：


.. code-block:: c

   location / {
       include uwsgi_params;
       uwsgi_pass 127.0.0.1:3031;
       uwsgi_modifier1 5;
   }

重启你的 nginx 服务器，然后它就会启动请求到你的 uWSGI 实例之间的代理。

注意你不需要把你的 uWSGI 配置一个特殊的 modifier，nginx 将会使用 ``uwsgi_modifier1 5;`` 指令。

如果你的代理/web 服务器/路由器 使用 HTTP，你需要告诉 uWSGI 使用 http 协议(这与 --http 不同，后者
会自己 spawn 一个代理):

.. code-block:: sh

   uwsgi --http-socket 127.0.0.1:3031 --http-socket-modifier1 5 --psgi myapp.pl
   
正如你看到的我们需要指定 modifier1，因为 http 协议不能附带这种信息。


添加并发
********

你可以通过多进程，多线程或者各种异步模式来给你的应用添加并发。

要 spawn 更多的进程，使用 --processes 选项

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --psgi myapp.pl --processes 4

要使用更多的线程，使用 --threads

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --psgi myapp.pl --threads 8

或者两者都用

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --psgi myapp.pl --threads 8 --processes 4
   
在 perl 世界中一个非常常见的非堵塞/协程库就是 Coro::AnyEvent 。uWSGi 简单
包含 ``coroae`` 插件就可以使用它了。

要编译一个带有 ``coroae`` 支持的 uWSGI 二进制文件只需运行：

.. code-block:: sh

   UWSGI_PROFILE=coroae make
   
或者

.. code-block:: sh

   curl http://uwsgi.it/install | bash -s coroae /tmp/uwsgi
   
你将会得到一个带有 ``psgi`` 和 ``coroae`` 插件的 uWSGI 二进制文件。

现在用 Coro::AnyEvent 模式来运行你的应用：


.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --psgi myapp.pl --coroae 1000 --processes 4
   
它会运行 4 个进程，每个进程可以管理 1000 个协程(或者 Coro 微线程)。


增加鲁棒性：主进程
******************

非常推荐的做法是在生成环境中的应用全部都运行主进程。

它会持续地监控你的进程/线程，并且会像 :doc:`StatsServer` 一样将会添加更多有趣的特性。

要使用主进程只需要加上 --master 选项

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --psgi myapp.pl --processes 4 --master
   
使用配置文件
************

uWSGI 提供了好几百个选限报告。通过命令行去处理它们是愚蠢的，所以尽量使用配置文件。
uWSGI 支持多种标准(xml, .ini, json, yaml...)。从一个标准变成另一个非常简单。
所有你在命令行中可以使用的选项只要去掉 ``--`` 前缀就可以用在配置文件中。

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   psgi = myapp.pl
   processes = 4
   master = true
   
或者 xml：

.. code-block:: xml

   <uwsgi>
     <socket>127.0.0.1:3031</socket>
     <psgi>myapp.pl</psgi>
     <processes>4</processes>
     <master/>
   </uwsgi>
   
要用配置文件来运行 uWSGI，只需要通过参数来指定它就可以了：

.. code-block:: sh

   uwsgi yourconfig.ini
   
如果出于某种原因你的配置文件不能以正常的拓展名(.ini, .xml, .yml, .js)结尾，
你可以用下面这种方式来强制 uWSGI 使用指定的解析器：

.. code-block:: sh

   uwsgi --ini yourconfig.foo
   
.. code-block:: sh

   uwsgi --xml yourconfig.foo

.. code-block:: sh

   uwsgi --yaml yourconfig.foo

等等

你甚至可以使用管道流式配置(使用 - 强制从标准输入读取)：

.. code-block:: sh

   perl myjsonconfig_generator.pl | uwsgi --json -


自动启动 uWSGI
**************

如果你打算写一些 init.d 脚本来启动 uWSGI，坐下来冷静一下，然后检查你的系统是否
真的没有提供更好的(现代化)的方式。

每一个发行版会选择一个启动系统 (:doc:`Upstart<Upstart>`, :doc:`Systemd`...) 除此之外也许多
进程管理工具 (supervisord, god...) 。

uWSGI will integrate very well with all of them (we hope), but if you plan to deploy a big number of apps check the uWSGI :doc:`Emperor<Emperor>`
uWSGI 与上面列出的那些工具都集成得很好(我们希望如此)，但是如果你想部署大量应用的话，看
看 uWSGI 的 :doc:`Emperor<Emperor>` 。它是每个运维开发的梦想。

安全和可用性
************

永远 不要使用 root 来运行 uWSGI 实例。你可以用 uid 和 gid 选项来降低权限：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   uid = foo
   gid = bar
   chdir = path_toyour_app
   psgi = myapp.pl
   master = true
   processes = 8


web 应用开发一个最常见的问题就是 “stuck requests”(卡住的请求)。你所有的线程/worker 都被卡住(被请求堵塞)， 
然后你的应用再也不能接受更多的请求。

为了避免这个问题你可以设置一个 harakiri 计时器。它是一个监视器(由主进程管理)，
当进程被卡住的时间超过特定的秒数后就销毁这个进程。

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   uid = foo
   gid = bar
   chdir = path_toyour_app
   psgi = myapp.pl
   master = true
   processes = 8
   harakiri = 30

上面的配置会将卡住超过 30 秒的 worker 销毁。慎重选择 harakiri 的值 !!!

另外，从 uWSGI 1.9 起，统计服务器会输出所有的请求变量，所以你可以(实时地)查看你的
实例在干什么(对于每个 worker，线程或者异步 core)。

打开 stats server 很简单：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   uid = foo
   gid = bar
   chdir = path_toyour_app
   psgi = myapp.pl
   master = true
   processes = 8
   harakiri = 30
   stats = 127.0.0.1:5000
   
只需要把它绑定到一个地址(UNIX domain sockt 或者 TCP)然后(你也可以使用 telnet)连接它，然后就会
返回你的实例的一个 JSON 数据。

``uwsgitop`` 应用(你可以在官方的 github 仓库中找到它)就是一个使用 stats 
server 的例子，它和 top 这种实时监控的工具类似(彩色的!!!)


Offloading
**********

:doc:`OffloadSubsystem` 使得你可以在某些模式满足时释放你的 worker，并且把工作委托给一个纯 c 的线程。
这样例子比如有从文件系统传递静态文件，通过网络向客户端传输数据等等。

Offloading 非常复杂，但它的使用对用户来说是透明的。如果你想试试的话加上 ``--offload-threads <n>`` 选项，
这里的 `<n>` 是 spawn 的线程数(以 CPU 数目的线程数启动是一个不错的值)。

当 offload threads 被启用时，所有可以被优化的部分都可以自动被检测到。


那么现在...
***********

有了这些很少的概念你就已经可以进入到生产中了，但是 uWSGI 是一个拥有上百个特性和配置的生态系统。
如果你想成为一个更好的系统管理员，继续阅读完整的文档吧。
