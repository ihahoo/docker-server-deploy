## 概述
使用nginx反向代理，可以共享80和443端口通过不同的域名访问不同的服务，实现虚机的配置。使用[jwilder/docker-gen](https://github.com/jwilder/docker-gen)配置nginx的自发现能力。再结合[docker-letsencrypt-nginx-proxy-companion](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion)，可以通过nginx自动发现配置 [let's encrypt](https://letsencrypt.org/) 的https证书，并且可以自动更新。

## 步骤
1.登录到服务器, 这个例子里，nginx的路径是`/srv/nginx`，可以修改成你的目录。

2.下载[nginx.tmpl](https://github.com/jwilder/nginx-proxy/blob/master/nginx.tmpl) ,这是`docker-gen`用到的自发现生成nginx配置文件的模版文件。
```
$ curl https://raw.githubusercontent.com/jwilder/nginx-proxy/master/nginx.tmpl > /srv/nginx/nginx.tmpl
```
`/srv/nginx/`目录如果不存在，请首先创建，也可以用你自定义的路径。

3.docker中建立自定义网络
```
$ docker network create nginx-proxy
```
这里为什么建立`nginx-proxy`这个网络？因为我们把nginx等安装到这个网络，这样其它通过`docker-compose`建立的容器也加入这个网络不会有错误，特别是通过`docker-compose`建立的多容器，比如nodejs + redis + postgres，否则会有错误，这里有坑，也是摸索了很久填出来的。

4.下载这里的 [docker-compose.yml](https://github.com/ihahoo/docker-server-deploy/blob/master/nginx-dockergen-letsencrypt/docker-compose.yml)
```
$ curl https://raw.githubusercontent.com/ihahoo/docker-server-deploy/master/nginx-dockergen-letsencrypt/docker-compose.yml > /srv/nginx/docker-compose.yml
```
5.启动服务
```
$ cd /srv/nginx
$ docker-compose up -d
```
6.自动从docker官方仓库下载镜像，并建立容器。（如果访问docker官方仓库比较慢，可以配置阿里云镜像加速器），不出意外的话，将会成功通过镜像建立容器。
```
$ docker ps
```
成功的话可以看到有3个运行的容器，名称是：`nginx`, `nginx-gen`, `nginx-letsencrypt`。如果遇到错误，或没有正常运行，检查80和443端口是否被服务器其它服务占用了。

7.可以配置其它服务了

[具体docker-compose.yml配置请参考这里](https://github.com/ihahoo/docker-server-deploy/tree/master/nginx-dockergen-web)
```
VIRTUAL_HOST=abc.yourdomain.com
LETSENCRYPT_HOST=abc.yourdomain.com
LETSENCRYPT_EMAIL=email@youremail.com
```
启动其它容器时，主要通过传递上面3个变量，就可以启动nginx的自发现功能，自动配置abc.yourdomain.com的虚机配置，如果要配置https，通过`LETSENCRYPT_HOST`和`LETSENCRYPT_EMAIL`变量，可以自动去下载https证书。酷吧。配置好后，把abc.yourdomain.com解析到服务器ip，就可以访问了。

## 配置
正常启动服务后，可以在/srv/nginx目录下建立下面文件夹：
- `certs`: let's encrypt下载的证书，保存在这里。
- `conf.d`: 自动生成的nginx配置文件在这里
- `htpasswd`: 这里可以配置密码才可以访问的服务
- `log`: nginx日志
- `vhost.d`: 虚机如果需要配置，配置文件放到这里

具体的配置说明请访问这里：https://github.com/jwilder/nginx-proxy

letsencrypt的具体变量及配置说明请访问这里：https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion
