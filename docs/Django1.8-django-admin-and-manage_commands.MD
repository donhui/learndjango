###startproject和startapp
####创建一个项目（Project）
Django使用`django-admin startproject [projectname]`命令生成project。
<br>
django-admin命令行的路径为：`django/bin/django-admin.py`
<br>
以`django-admin startproject mysite`为例，这将会在当前目录下生成一个mysite目录，目录结构如下：
<pre><code>mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
</code></pre>

####创建一个应用（App）
在project中与manage.py相同的目录下使用`python manage.py startapp [appname]`命令生成app。
<br>
以`python manage.py startapp polls`为例，这将创建一个目录polls，目录结构如下：
<pre><code>polls/
    __init__.py
    admin.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
</code></pre>

###django-admin.py与manage.py工作原理
`django-admin.py`源码：
<pre><code>#!/usr/bin/env python
from django.core import management

if __name__ == "__main__":
    management.execute_from_command_line()
</code></pre>

manage.py源码：
<pre><code>#!/usr/bin/env python
import os
import sys

if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")

    from django.core.management import execute_from_command_line

    execute_from_command_line(sys.argv)
</code></pre>

它们都用到了django.core.management这个模块的execute_from_command_line方法。
<br>
主要的区别是`manage.py`比`django-admin.py`多了一行:
<br>
`os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")`
<br>
用来设置DJANGO_SETTINGS_MODULE这个环境变量为当前项目的settings文件。

###命令执行过程分析
通过`execute_from_command_line()`方法，可以看到Django命令主要是通过
<br>
`django.core.management.ManagementUtility`类的入口`execute()`执行。

命令执行过程如下：

1. 解析命令，获得要执行的子命令名称（如startapp）。
<br>
`subcommand = self.argv[1]`
2. 找出subcommand的完整路径
<br> 
`self.fetch_command(subcommand)`
<br>
其中fetch_command调用get_commands从下面几个地方找命令:
	- django/core/management/commands目录下的命令文件
	- project/<INSTALLED_APPS>/management/commands/目录下的命令文件
3. 根据返回的subcommand实例，执行`run_from_argv()`方法：
<br>
`self.fetch_command(subcommand).run_from_argv(self.argv)`
<br>
从`django.core.management.base.BaseCommand`中可知`run_from_argv()`方法的调用过程：
<br>
`run_from_argv() ->  execute() -> handle() `  
`handle()`方法执行的结果就是最后的返回。

###Command命令
在django/core/management/commands可以查看所以的系统命令。

startproject.py和startapp.py都是调用`django.core.management.templates.TemplateCommand`类的`handle()`方法进行处理。
<br>
两者主要的区别是：template_dir不同

- project的template目录：`django.conf.project_template`
- app的template目录：`django.conf.app_template`


###参考
- [http://python.usyiyi.cn/django/intro/tutorial01.html](http://python.usyiyi.cn/django/intro/tutorial01.html)
- [http://yijingping.github.io/2014/03/12/django-1.html](http://yijingping.github.io/2014/03/12/django-1.html)







