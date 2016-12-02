---
layout: post
title: ISSO 评论系统试用总结!
tags:
 - python
 - isso
 - nginx
categoreis: python
---

项目：https://github.com/posativ/isso.git
文档：https://posativ.org/isso/

# 简介

介绍部分在文档里面都有，概括就是基于python语言开发的评论系统，使用轻量级数据库SQLite，支持Disqus、WordPress评论的导入，可配置的js客户端。

# 服务器的安装
以linux（ubuntu）系统为例。

1.安装python的虚拟环境，创建目录isso, 创建虚拟环境`.isso_venv`，并安装`isso`软件包。

```bash
mkdir isso
cd isso
virtualenv .isso_venv
source .iss_venv/bin/activate #进入虚拟环境。
pip install isso
```

2.isso配置文件
简单的配置如下：

````ini
# isso.cfg
[general]
dbpath = comments.db  # 评论数据库
host = http://localhost  # 评论系统挂在的主机
log-file = isso.log

[moderation]
enabled = false  # 是否开启评论审核。如果为true, 用户的评论在审核通过后才可以被其他用户看到。简单服务器可以向disabled = false。

[server]
listen = http://localhost:8080  # 评论系统的所在的服务器器。也可以使用uwsgi或者其他工具启动服务器。详情见官方文档描述。
reload = on

[guard]
enabled = true  # 开启限制选项
direct-relay = 50  # 最大允许的一级回复的数目
retelimit = 10  # 一分钟允许最大创建5个回复
reply-to-self = fase  # 不允许回复自己
```

在配置完成后，启动服务前可以试着导入Disqus, WordPress的评论，既可以验证配置，也可以为后面提供测试数据。
测试导入的文件在：
Disqus: https://github.com/posativ/isso/blob/master/isso/tests/disqus.xml
WordPress: https://github.com/posativ/isso/blob/master/isso/tests/wordpress.xml
当然，也可以使用自己的文件导入。
导入命令为：

```bash
isso -c isso.cfg import disqus.xml
```

3.启动服务器。

```bash
isso -c isso.cfg run
```

测试是否启动成功。

```bash
curl http://localhost/ # 如果返回，Bad Request，missing uri query等消息，表示服务启动成功。
```

测试之前导入的数据。

```bash
curl http://localhost/?uri=/  # uri=/ 表示查询uri=/， 根目录下的评论。测试文件Disqus.xml中的正是根目录下的评论。
```

测试js文件的加载。

```bash
curl http://localhost/js/embed.min.js  # 测试服务器的js文件
```

在配置中，我们使用isso自身来启动localhost:8080的服务器，也可以是用uwsgi或者其他便于管理的启动工具，来管理服务器。
成功启动服务器，相当于评论系统的后台服务搭建完成。
前端展示部分还需要使用nginx代理，可以避免跨域问题等等等。

# 客户端加载
1.Nginx代理。
评论系统大多是准备集成在已有的web server上，站点本身已经有特定的首页，评论后台为其他提供API服务即可。
其中一个做法，将isso服务器挂在在nginx的子目录，代理isso服务。
Nginx配置如下：

```nginx
	....defautl.....
	# comments api.
	location /isso {
		proxy_pass http://localhost:8080;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Script-Name /isso;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
	}
```

当然可以可以使用虚拟主机的，详细做法官网有介绍。见https://posativ.org/isso/docs/quickstart/#id3

2.客户端加载评论。
在页面上添加脚本。

```html
<div id="wrapper" style="width: 900px; text-align: left; margin-left: auto; margin-right: auto;">
    <script 
    	data-isso="/isso/"
        src="/isso/js/embed.min.js"
        data-isso-lang="zh"
        data-isso-vote="true">
    </script>

    <section id="isso-thread" data-title="APIDoc"></section>
</div>

```js
data-isso: isso服务的url，本例中因为挂在在80端口的/isso，可以使用绝对的地址如，http://exampe.tld/isso/
src: isso js路径
data-isso-lang: 语言
data-isso-vote: 允许投票
其他配置项，详见，https://posativ.org/isso/docs/configuration/client/

# 其他的功能

1.评论审查功能
开启审查功能，启用邮件通知。

```ini
# iss.cfg
[general]
 ...
notify = smtp
purge-after = 30d  # 30天后，没有审查通过的评论自动删除。
 ...

[moderation]
enabled = true
 ...
[smtp]
username = xxxxx
password = xxxxx
host = smtp.email.com
port = 465
to = yyyyy@email.com
from = IssoGhost<xxx@email.com>
security = starttls
timeout = 30
 ...
```

开启评论审查功能后，用户的新评论会触发isso发送通知邮件到管理员。邮件内容包含新用户评论信息，删除评论的链接和激活评论的链接。审查通过的评论才能被其他用户看到。

2.多个页面的评论
在多个页面添加scritps,创建评论时会自动带上当前的相对URI, 并保存在评论表中。

3.评论排列问题：
client scripts 中：

```js
data-isso-max-comments-top  // 第一层评论最大显示数。
data-isso-max-comments-nested  // 嵌套评论最大显示数。
data-isso-reveal-on-click  // 点击显示隐藏的评论。
data-isso-vote  // 允许评论
```

isso的加载配置和客户端scrips加载均有配置是否同意评论自身（自己回复自己），但是在vote（投票）上，却是不可以自己给自己投票。
即是相同的IP显示为一个用户，同一个IP不允许upvote或者downvote。（我目前试用是这样的情况）。

4.评论的导出csv。

```bash
echo -e '"page: URI","page: title","ID","mode","created on","modified on","author: name","author: email","author: website","author: IP","likes","dislikes","voters","text"\n'"$(sqlite3 comments.db -csv 'SELECT threads.uri, threads.title, comments.id, comments.mode, datetime(comments.created, "unixepoch", "localtime"), datetime(comments.modified, "unixepoch", "localtime"), comments.author, comments.email, comments.website, comments.remote_addr, comments.likes, comments.dislikes, comments.voters,comments.text FROM comments INNER JOIN threads ON comments.tid=threads.id')" > export.csv
```

或者

```bash
echo -e '"page: URI","page: title","ID","mode","created on","modified on","author: name","author: email","author: website","author: IP","likes","dislikes","voters","text"\n'"$(sqlite3 /path/to/your/isso.db -csv 'SELECT threads.uri, threads.title, comments.id, comments.mode, comments.created, comments.modified, comments.author, comments.email, comments.website, comments.remote_addr, comments.likes, comments.dislikes, comments.voters,comments.text FROM comments INNER JOIN threads ON comments.tid=threads.id')" > export.csv
```

# 一些坑。
1.客户端放置script脚本的html文件要求。

```python
# isso/utils/parse.py, line 29~30
    html = html5lib.parse(data, treebuilder="dom")

    assert html.lastChild.nodeName == "html"
    html = html.lastChild
```

要求html文件的最后一个node必须是html，也就是在html文件最后来个注释块是不行的。（这里还没有弄懂为什么）

2.react中加载客户端的方法。
不能直接粘贴上面的客户端scripts要相应页面。应该document.createElemen("script"),document.createElement("section"),并给之赋予相对应的属性，"isso-thread", "src", "data-isso", "data-isso-lang", "data-isso-vote"等。
