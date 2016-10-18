---
layout: post
title: 使用国内镜像源来加速python pypi包的安装
description: 
keywords: Python, pip, easy_install
category: Python
tags: [pip,easy_install]
---

pipy国内镜像目前有：

* http://pypi.douban.com/  豆瓣

* http://pypi.hustunique.com/  华中科技大学

* http://pypi.sdutlinux.org/  山东理工大学

* http://pypi.mirrors.ustc.edu.cn/  中国科学技术大学

* http://e.pypi.python.org/simple 公网

* http://pypi.tuna.tsinghua.edu.cn/simple 教育网

<!-- more -->

要配制成默认的话，需要创建或修改配置文件（linux的文件在~/.pip/pip.conf，windows在%HOMEPATH%\pip\pip.ini），修改内容为：
{% highlight sh %}
[global]
index-url = http://pypi.douban.com/simple
{% endhighlight %}

如果想手动指定源，可以在pip后面跟-i 来指定源，比如用豆瓣的源来安装web.py框架：
{% highlight sh %}
pip install web.py -i http://pypi.douban.com/simple
{% endhighlight %}
注意后面要有`/simple`目录。

###Windows

修改`%PYTHON_HOME%\Lib\site-packages\pip\cmdoptions.py`
{% highlight py %}
    index_url = OptionMaker(  
        '-i', '--index-url', '--pypi-url',  
        dest='index_url',  
        metavar='URL',  
        #default='https://pypi.python.org/simple/',  
         default='http://mirrors.bistu.edu.cn/pypi/',  
        help='Base URL of Python Package Index (default %default).')  
{% endhighlight %}

和`%PYTHON_HOME%\Lib\site-packages\pip\commands\search.py`
{% highlight py %}
    class SearchCommand(Command):  
        """Search for PyPI packages whose name or summary contains <query>."""  
        name = 'search'  
        usage = """  
          %prog [options] <query>"""  
        summary = 'Search PyPI for packages.'  
      
        def __init__(self, *args, **kw):  
            super(SearchCommand, self).__init__(*args, **kw)  
            self.cmd_opts.add_option(  
                '--index',  
                dest='index',  
                metavar='URL',  
                #default='https://pypi.python.org/pypi',  
                default='http://mirrors.bistu.edu.cn/pypi/',  
                help='Base URL of Python Package Index (default %default)')  
      
            self.parser.insert_option_group(0, self.cmd_opts)  
{% endhighlight %}

###Linux

####1.命令方式临时修改

* easy_install:
{% highlight sh %}
easy_install -i http://e.pypi.python.org/simple fabric
{% endhighlight %}

* pip:
{% highlight sh %}
pip -i http://e.pypi.python.org/simple install fabric
{% endhighlight %}

####2.配置方式修改

* easy_install:
{% highlight sh %}
vi ~/.pydistutils.cfg

[easy_install]
index_url = http://e.pypi.python.org/simple
{% endhighlight %}

* pip:
{% highlight sh %}
vi ~/.pip/pip.conf

[global]
index-url = http://e.pypi.python.org/simple
{% endhighlight %}
