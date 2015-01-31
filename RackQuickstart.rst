ruby/Rack 应用快速入门
=====================

下面这份使用说明将会引导你安装，运行一个基于 Ruby 的 uWSGI 应用。旨在运行 Rack 应用。

安装带 Ruby 支持的 uWSGI
************************

为了编译 uWSGI 你需要一个 C 编译器(gcc 和 clang 都支持)和一个 Python 解释器(用于运行 uwsgiconfig.py 脚本, 这个脚本
将会执行各种各样的编译步骤)。

为了编译带 Ruby 支持的 uWSGI 二进制文件，我们还需要 Ruby 开发版头文件(在 Debian
系的发行版上即 ``ruby-deb`` 包)。

你可以手动构建 uWSGI -- 所有这些方式都是等价的：

.. code-block:: sh

   make rack
   UWSGI_PROFILE=rack make
   make PROFILE=rack
   python uwsgiconfig.py --build rack
   
如果你够懒的话，你可以一次性下载编译安装一个 uWSGI + Ruby 二进制文件：

.. code-block:: sh

   curl http://uwsgi.it/install | bash -s rack /tmp/uwsgi
   
或者一种更 "Ruby 友好“ 的方法：

.. code-block:: sh

   gem install uwsgi
   
所有这些方法都构建了一个”大一统“的 uWSGI 二进制文件。
uWSGI 项目由许多插件组成。你可以选择构建核心的服务器，然后为每个特性构建单独的插件(需要的时候就加载)。
或者你可以构建一个单独的带有所有你需要的特性的二进制文件。后者被称为 'monolithic' 。

这个快速入门假定你编译了一个大一统的二进制文件(所以你不需要加载插件)。
如果你更喜欢使用你的包管理器(从官方源构建 uWSGI)，请看下面。

Note for distro packages
************************

你的发行版很有可能已经包含了一个 uWSGI 的包集合。
这些 uWSGI 包更倾向于高度模块化(以及偶尔会严重过时)，
所以出了核心你还需要安装需要的插件。插件必须在你的 uWSGI 配置中加载。
在学习阶段我们强烈建议不要使用包管理器提供的包，而是简单跟着文档和教程走就可以了。

一旦你适应了这种 ”uWSGI 方式“，你就可以选择最适合你开发的方法了。

比如说，这个教程会使用到 "http" 和 "rack" 插件。
如果你使用的是模块化编译确保你用 ``--plugins http,rack`` 选项加载了它们。

你的第一个 Rack 应用
********************

Rack 是编写 Ruby web 应用的标准方式。

这是一个标准的 Rack Hello world 脚本(把它取名为 app.ru)：

.. code-block:: rb

   class App

     def call(environ)
       [200, {'Content-Type' => 'text/html'}, ['Hello']]
     end
     
   end
   
   run App.new
   
``.ru`` 后缀名表示 "rackup", 它是 Rack 包包含的一个开发工具。
Rackup 使用了一些 DSL, 所以想在 uWSGI 中使用它的话你需要安装 rack gem：

.. code-block:: sh

   gem install rack
   
现在我们准备好使用 uWSGI 了：

.. code-block:: sh

   uwsgi --http :8080 --http-modifier1 7 --rack app.ru

(记着如果 uwsgi 不在 $PATH 里的话把 'uwsgi' 替换成成相应的路径)

或者如果你使用了模块化安装的话(比如你的发行版提供的包)

.. code-block:: sh

   uwsgi --plugins http,rack --http :8080 --http-modifier1 7 --rack app.ru
   
通过这个命令我们 spawn 了一个 HTTP 代理，它会每一个请求转发到一个进程(叫 'worker') ，worker 会
处理它然后返回一个回复给 HTTP 路由(然后它再发送给客户端)。

如果你问为什么要 spawn 两个进程，那是因为这是在生产环境中最常见的架构(一个前端 web 服务器和一个后端应用服务器)。

如果你真的不想 spawn HTTP 代理而是直接强制 worker 回应 HTTP 请求的话改下命令行就可以了：

.. code-block:: sh

   uwsgi --http-socket :8080 --http-socket-modifier1 7 --rack app.ru
   
现在你就有了一个单一的进程来处理请求(但是记住这样会把应用服务器暴露在公网中，这通常
是很危险的，而且也很少用)。

‘--http-modifier1 7’ 是什么鬼？
******************************

uWSGI 支持多种语言和平台。当服务器收到一个请求时它得知道把它"路由"到哪里去。

每个 uWSGI 插件都有一个给定的数字(即 modifiers), ruby/rack 是 7 。所以 ``--http-modifier1 7`` 表示 "路由到 rack 插件"。

虽然 uWSGI 也有一个更人性化的 :doc:`internal routing system <InternalRouting>` , 但是
使用 modifiers 是处理速度最快的方式，所有尽量使用它们。

使用完整的 web 服务器: nginx
****************************

uWSGI 提供的 HTTP 路由器只是一个路由器。
你可以把它当成负载均衡器或者代理来使用，但是如果你需要一个完整的 web 服务器(为了高效地提供静态文件服务或者
其他类似的 web 服务器擅长的任务),
使用 uwsgi HTTP 路由器有风险(如果你使用了模块化构建的话记着把 --plugins http,rack 改成 --plugins rack)，
你应该把你的应用放在 Nginx 后面。

为了和 Nginx 通讯，uWSGI 可以使用多种协议：HTTP, uwsgi, FastCGI, SCGI 等等。

性能最好的是 uwsgi 协议。Ngxin 包含了 uwsgi 协议，开箱即用。

在 uwsgi socket 上运行你的 rack 应用：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --rack app.ru

然后在你的 nginx 配置添加 location 节：

.. code-block:: c

   location / {
       include uwsgi_params;
       uwsgi_pass 127.0.0.1:3031;
       uwsgi_modifier1 7;
   }

重启你的 nginx 服务器，然后它应该就开始为你的 uWSGI 实例反向代理请求了。

注意你并不需要配置 uWSGI 来使用特定的 modifier, nginx 将会直接使用 ``uwsgi_modifier1 5;`` 。

添加并发
*******

在前面的例子中我们构建了一个一次只能处理一个请求的栈。

为了增加并发我们需要增加更多的进程。
如果你希望有一个魔法方程来计算正确的进程数目，呃，不好意思我们没有。
你需要实验监控你的应用来找到正确的值。
考虑到每一个单独的进程都是你的应用的一份完全的复制，所以内存使用需要被考虑在内。

要添加更多的进程使用 `--processes <n>` 选项就可以了：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --rack app.ru --processes 8
   
这将会 spawn 8 个进程。

混合编译时这个插件会自动编译进去。

添加更多的线程：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --rack app.ru --rbthreads 4
   
或者线程 + 进程

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --rack app.ru --processes --rbthreads 4
   
有一些其他的(通常更高级/复杂)方法来增加并发(比如 'fibers')，但大多数情况下你会
以原先的多进程或者多线程告终。如果你感兴趣的话可以查阅 :doc:`Rack` 的完整文档。

增加鲁棒性：主进程
******************

强烈建议在生产环境中始终使用 uWSGI 主进程。

它会持续地监视你的进程/线程，然后它还会像 :doc:`StatsServer` 一样添加一些有趣的特性。

要使用主进程只要简单地加上 ``--master`` 就可以了

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --rack app.ru --processes 4 --master
   
使用配置文件
***********

uWSGI 提供了好几百个选项(但你通常用到的不会超过几十个)。通过命令行去处理它们是愚蠢的，所以尽量使用配置文件。 

uWSGI 支持多种标准(xml, .ini, json, yaml...)。从一个标准变成另一个非常简单。 
所有你在命令行中可以使用的选项只要去掉 -- 前缀就可以用在配置文件中。

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   rack = app.ru
   processes = 4
   master = true
   
或者 xml:

.. code-block:: xml

   <uwsgi>
     <socket>127.0.0.1:3031</socket>
     <rack>app.ru</rack>
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

等等。

你甚至可以使用管道流式配置(使用 - 强制从标准输入读取)：

.. code-block:: sh

   ruby myjsonconfig_generator.rb | uwsgi --json -
   
当你使用多进程时的 fork() 问题
******************************

uWSGI is "Perlish" in a way, there is nothing we can do to hide that. Most of its choices (starting from "There's more than one way to do it") came from the Perl world (and more generally from classical UNIX sysadmin approaches).

有时候其他语言/平台上使用这些方法会导致不在意料中的行为发生。

当你开始学习 uWSGI 的时候一个你可能会面对的"问题"之一就是它的 ``fork()`` 使用。

默认情况下 uWSGI 在第一个 spawned 的进程里加载你的应用，然后在这个进程里面调用 ``fork()`` 多次。

这意味这你的应用被单独加载一次然后被复制。

虽然这个方法加速了服务器的启动，但有些应用可能会因为这个技术造成一些问题(特别是这些在启动的
时候初始化数据库连接的，因为连接的文件描述符会在字进程中继承)。

如果你确定应不应该使用 uWSGI 野蛮的预-fork方式，那就使用 ``--lazy-apps`` 选项禁用掉它。
它将会强制你的应用在每个 worker 里都会完整加载一次。

部署 Sinatra
************

让我们忘掉 fork(), 然后回到有趣的事情上来。这次我们要部署一个 Sinatra 应用。

.. code-block:: rb

   require 'sinatra'

   get '/hi' do
     "Hello World"
   end

   run Sinatra::Application
   
保存为 ``config.ru`` 然后像前面那样运行：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   rack = config.ru
   master = true
   processes = 4
   lazy-apps = true
   
.. code-block:: sh

   uwsgi yourconf.ini
   
呃，你或许早就发现和前面例子中 app.ru 基本没有发生什么改变。

这是因为基本上所有的现代的 Rack 应用都把它自己暴露成一个 .ru 文件(通常叫 config.ru), 所以
加载应用不需要多种选项(就像 Python/WSGI 世界里的例子一样)。

部署 RubyOnRails >= 3
*********************

从 3.0 开始，Rails 完全兼容 Rack，并且提供了一个你可以直接加载的 cofnig.ru 文件(就像我们在
Sinatra 中做的那样)。

与 Sinatra 唯一的不同就是你的项目的布局/约定你的当前目录包含了项目，所以让我们添加一个 chdir 的选项：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   rack = config.ru
   master = true
   processes = 4
   lazy-apps = true
   chdir = <path_to_your_rails_app>
   env = RAILS_ENV=production
   
.. code-block:: sh

   uwsgi yourconf.ini
   
除了 chdir 之外我们还加上了 'env' 选项，设置了 ``RAILS_ENV`` 环境变量。

从 4.0 起，Rails 支持多线程(仅在 ruby 2.0 中)：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   rack = config.ru
   master = true
   processes = 4
   rbthreads = 2
   lazy-apps = true
   chdir = <path_to_your_rails_app>
   env = RAILS_ENV=production

部署旧版的 RubyOnRails
**********************

旧版的 Rails 不是完全兼容 Rack。基于这个原因所以 uWSGI 有一个专门的选项来加载旧版 Rails 应用(你
也需要 'thin' gem)。

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   master = true
   processes = 4
   lazy-apps = true
   rails = <path_to_your_rails_app>
   env = RAILS_ENV=production
   
所以，简单来说就是，指定 ``rails`` 选项，然后把 rails 应用目录传给它，而不是传一个 Rackup 文件。

Bundler 和 RVM
**************

Bundler 是事实上的标准 Ruby 依赖管理工具。你主要在 Gemfile 文本文件中申明
你的应用需要的 gems，然后用 bundler 来安装它们。

要让 uWSGI 帮你使用 bundler 安装你只需要添加：

.. code-block:: ini

   rbrequire = rubygems
   rbrequire = bundler/setup
   env = BUNDLE_GEMFILE=<path_to_your_Gemfile>

(前一个 require 在 rubty 1.9/2.x 中不需要。)

这些行主要强制 uWSGI 加载 bundler 引擎然后使用由 ``BUNDLE_GEMFILE`` 环境变量
指定的 Gemfile 文件。

当使用 Bundler 的时候(就像现代的框架一样)你通常的开发配置会这样：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   rack = config.ru
   master = true
   processes = 4
   lazy-apps = true
   rbrequire = rubygems
   rbrequire = bundler/setup
   env = BUNDLE_GEMFILE=<path_to_your_Gemfile>
   
除了 Bundler，RVM 是另外一个常用的工具。

它允许你有多个版本(独立的)的 Ruby 安装(以及它们的 gem 集合)在一个单独的系统中。

要让 uWSGI 使用某特定 RVM 版本的 gem 集合只需要使用 `-gemset` 选项：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   rack = config.ru
   master = true
   processes = 4
   lazy-apps = true
   rbrequire = rubygems
   rbrequire = bundler/setup
   env = BUNDLE_GEMFILE=<path_to_your_Gemfile>
   gemset = ruby-2.0@foobar
   
请注意对于每一个 Ruby 版本(是 Ruby 的版本，不是 gemset 的)你需要一个 uWSGI 二进制文件(或者一个插件，如果你使用了模块化构建的话)。

如果你感兴趣，这是用来在 rvm 构建多版本的 Ruby 并各自带有 uWSGI 核心以及一个插件的命令列表：

.. code-block:: sh

   # build the core
   make nolang
   # build plugin for 1.8.7
   rvm use 1.8.7
   ./uwsgi --build-plugin "plugins/rack rack187"
   # build for 1.9.2
   rvm use 1.9.2
   ./uwsgi --build-plugin "plugins/rack rack192"
   # and so on...
   
然后如果你想使用 ruby 1.9.2 并使用 @oops gemset:

.. code-block:: ini

   [uwsgi]
   plugins = ruby192
   socket = 127.0.0.1:3031
   rack = config.ru
   master = true
   processes = 4
   lazy-apps = true
   rbrequire = rubygems
   rbrequire = bundler/setup
   env = BUNDLE_GEMFILE=<path_to_your_Gemfile>
   gemset = ruby-1.9.2@oops

自动启动
********

如果你打算打开 vi 写一个 init.d 脚本来启动 uWSGI，
坐下来冷静一下然后先确保你的系统没有提供一个更好(更现代化)的方式。

没一个发行版会选择一个启动系统 (:doc:`Upstart<Upstart>`, :doc:`Systemd`...) ，
除此之外也有许多 进程管理工具(supervisord, god, monit, circus...)。

uWSGI 与上面列出的那些工具都集成得很好(我们希望如此)，但是如果你想部署大量应用的话，
看看 uWSGI 的 :doc:`Emperor<Emperor>` - 它或多或少是每个开发运维工程师的梦想。

安全和可用性
************

**永远** 不要使用 root 来运行 uWSGI 实例。你可以用 uid 和 gid 选项来降低权限：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   uid = foo
   gid = bar
   chdir = path_toyour_app
   rack = app.ru
   master = true
   processes = 8


web 应用开发一个最常见的问题就是 “stuck requests”(卡住的请求)。你所有的线程/worker 都被卡住(被请求堵塞)， 然后你的应用再也不能接受更多的请求。

为了避免这个问题你可以设置一个 ``harakiri`` 计时器。它是一个监视器(由主进程管理)，当进程被卡住的时间超过特定的秒数后就销毁这个进程。

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   uid = foo
   gid = bar
   chdir = path_toyour_app
   rack = app.ru
   master = true
   processes = 8
   harakiri = 30

上面的配置会将卡住超过 30 秒的 worker 销毁。慎重选择 harakiri 的值!

另外，从 uWSGI 1.9 起，统计服务器会输出所有的请求变量，所以你可以(实时地)查看你的实例在干什么(对于每个 worker，
线程或者异步 core)。

打开 stats server 很简单：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   uid = foo
   gid = bar
   chdir = path_to_your_app
   rack = app.ru
   master = true
   processes = 8
   harakiri = 30
   stats = 127.0.0.1:5000
   
只需要把它绑定到一个地址(UNIX domain sockt 或者 TCP)然后(你也可以使用 telnet)连接它，
然后就会返回你的实例的一个 JSON 数据。

``uwsgitop`` 应用(你可以在官方的 github 仓库中找到它)就是一个使用 stats server 的例子，
它和 top 这种实时监控的工具类似(彩色的!!!)

内存使用
********

低内存消耗是真个 uWSGI 项目的一个买点之一。

不幸的是默认的苛刻内存使用可能(注意：是可能)会导致性能问题。

uWSGI Rack 插件默认在每个请求完成后调用 Ruby GC(垃圾回收器)。
如果你想减少 gc 的频率只需要添加上 ``--rb-gc-freq <n>`` 选项，
n 是多少个请求完成后才调用 GC。

如果你计划对 uWSGI 做基准测试(或者与其他的解决方案比较)请注意它的 GC 使用。

Ruby 有时可能会真的是一头内存怪兽，所以我们更倾向于默认的苛刻内存使用，
而不是为了得到 hello-world 类的基准测试(benchmarkers)高分。

Offloading
**********

The uWSGI :doc:`OffloadSubsystem` 使得你可以在某些模式满足时立即释放你的 worker，
并且把工作委托给一个纯 c 的线程。
这样例子包括从文件系统传递静态文件，通过网络向客户端传输数据等等。

Offloading 非常复杂，但它的使用对终端用户来说是透明的。
如果你想试试的话加上 ``--offload-threads <n>`` 选项，
这里的 <n> 是 spawn 的线程数(以 CPU 数目的线程数启动是一个不错的值)。

当 offload threads 被启用时，所有可以被优化的部分都可以自动被检测到。


那么现在...
***********

有了这些很少的概念你就已经可以进入到生产中了，
但是 uWSGI 是一个拥有上百个特性和配置的生态系统。 如果你想成为一个更好的系统管理员，继续阅读完整的文档吧。

欢迎！
