django-mama-cas部署文档
-----------------------

## 开发环境

```
django-admin startproject mysite
```

编辑 mysite/mysite/settings.py 的内容

```
INSTALLED_APP = [
    'mama_cas',
]

MAMA_CAS_ATTRIBUTE_CALLBACKS = ('mama_cas.callbacks.user_info_attributes',)
MAMA_CAS_FOLLOW_LOGOUT_URL = True
MAMA_CAS_ENABLE_SINGLE_SIGN_OUT = True

''' 在 GitHub 帐户页的 setting 里面注册 '''
MAMA_CAS_OAUTH_GITHUB_CLIENT_ID = ''
MAMA_CAS_OAUTH_GITHUB_CLIENT_SECRET = ''

''' 去 http://connect.qq.com 申请 '''
MAMA_CAS_OAUTH_QQ_APP_ID = ''
MAMA_CAS_OAUTH_QQ_APP_KEY = ''

''' 去 http://open.weibo.com 申请 '''
MAMA_CAS_OAUTH_WEIBO_APP_KEY = ''                                        
MAMA_CAS_OAUTH_WEIBO_APP_SECRET = ''
```

编辑 mysite/mysite/urls.py 的内容

```
urlpatterns = [
    url(r'', include('mama_cas.urls')),
]
```

由于***Weibo***、***QQ***需要认证网站域名（GitHub就不需要！），编辑django_mama_cas的模板文件
mama_cas/templates/mama_cas/__base.html

```
<!-- Weibo -->
<meta property="wb:webmaster" content="" />
<!-- QQ -->
<meta property="qc:admins" content="" />
```

在新浪、腾讯的相关页面上会给出对应的 content 值。

由于***Weibo***、***QQ***需要认证网站域名（GitHub就不需要！）在 /etc/hosts 中配置一个假域名

```
python manage.py runserver www.yourdomain.com:8000
```

而且***QQ***的回调URL不支持8000端口，只能在生产环境下调式QQ oauth API。

## 生产环境

nginx + uwsgi

/etc/nginx/sites-available/django 例子

```
upstream django {
    server unix:///data/mysite/mysite.sock;
}

server {
    listen      80;
    server_name www.yourdomain.com;
    charset     utf-8;

    client_max_body_size 75M;

    location /media  {
        alias /data/mysite/media;
    }

    location /static {
        alias /data/mysite/static;
    }

    location / {
        uwsgi_pass  django;
        include     uwsgi_params;
    }
}
```

/etc/uwsgi/django.ini 例子

```
[uwsgi]

chdir           = /data/mysite
module          = mysite.wsgi
chown-socket    = http:http
uid             = http
gid             = http
master          = true
processes       = 10
socket          = /data/mysite/mysite.sock
vacuum          = true
```
