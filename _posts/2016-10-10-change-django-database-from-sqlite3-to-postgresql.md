---
layout: post
title: Django从sqlite3切换数据库服务器到postgresql
category: django
---

# 安装postgresql 服务器

```bash
# ubuntu 

# centos
```

# 安装python依赖

```bash
# ubuntu 环境
sudo apt-get install libpg-dev
pip install psycopg2

# centos7(failed: need path with postgres/version/bin)
yum install postgresql-libs
pip install psycopg2

# centos7
yum install libpqxx-devel
pip instlal psycopg2
```

# 备份当前的数据

```bash
# 忽略session, admin操作记录以及每次自动创建的对象
./manage.py dumpdata --indent=4 -e sessions -e admin -e contenttypes -e auth.Permission > data.json

# 备注
# 如果需要备份contrib.auth.Permission或者contrib.contenttypes.Contenttype 需要使用--natural-foreign
# 如果需要重新开始primary_key,使用--natural-primary
```


# 配置django

```python
# settings.py
DATABASES = {
     'default': {
          'ENGINGE': 'django.db.backends.postgresql',
          'NAME': 'griverdb',
          'USER': 'griver',
          'PASSWORD': 'gstar!@#',
          'HOST': '127.0.0.1',
          'PORT': '5432'
     }
}
```

# 恢复数据库

```bash
./manage.py loaddata data.json
```