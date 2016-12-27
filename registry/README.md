## nginx + docker-gen + letsencrypt 下部署Docker私有仓库registry
先按[nginx-dockergen-letsencrypt](https://github.com/ihahoo/docker-server-deploy/tree/master/nginx-dockergen-letsencrypt)方法安装nginx + docker-gen + letsencrypt

## 步骤
1.生成访问registry的密码：
```
$ docker run --rm --entrypoint htpasswd registry -bn username password > /srv/nginx/htpasswd/registry.yourdomain.com
```
- `username`: 你的用户名
- `password`: 你的密码
- `/srv/nginx/htpasswd`: 建立nginx容器时定义的htpasswd的路径，请换成你自己的。
- `registry.yourdomain.com`: registry仓库的域名

2.下载[vhost](https://github.com/ihahoo/docker-server-deploy/blob/master/registry/vhost.conf) 配置文件
```
$ curl https://raw.githubusercontent.com/ihahoo/docker-server-deploy/master/registry/vhost.conf > /srv/nginx/vhost.d/registry.yourdomain.com
```
- `/srv/nginx/vhost.d`: 建立nginx容器时定义的vhost.d路径
- `registry.yourdomain.com`: registry仓库的域名

```
$ vi /srv/nginx/vhost.d/registry.yourdomain.com
```
编辑刚刚下载的配置文件，将里面的`registry.yourdomain.com`和`http://registry.yourdomain.com`部分替换成你的仓库域名,然后保存。

3.建立registry文件夹，下载docker-compose.yml到此文件夹下：
```
$ mkdir /srv/app/registry
```
- [将仓库存储到本地的模板](https://github.com/ihahoo/docker-server-deploy/blob/master/registry/docker-compose.yml)
- [将仓库存储到阿里云oss的模板](https://github.com/ihahoo/docker-server-deploy/blob/master/registry/docker-compose.oss.yml)

3.1 将仓库存储到本地
```
$ curl https://raw.githubusercontent.com/ihahoo/docker-server-deploy/master/registry/docker-compose.yml > /srv/app/registry/docker-compose.yml
```
- `VIRTUAL_HOST`: registry仓库的域名
- `HTTPS_METHOD`: `nohttp`表示，不允许http，只能https。在本例中，由于使用let's encrypt https证书，所以安全期间，不允许http访问。
- `LETSENCRYPT_HOST`: registry仓库的域名，获取Let's encrypt https证书设置
- `LETSENCRYPT_EMAIL`: 你的emial，获取Let's encrypt https证书设置
```
volumes
  - /srv/app/registry/data:/var/lib/registry
```
这里的`/srv/app/registry/data`为本地存储registry仓库的路径

根据你的情况，修改docker-compose.yml的相关配置后保存。

3.2 将仓库存储到阿里云oss
```
$ curl https://raw.githubusercontent.com/ihahoo/docker-server-deploy/master/registry/docker-compose.oss.yml > /srv/app/registry/docker-compose.yml
```
- `REGISTRY_STORAGE_OSS_ACCESSKEYID`: 访问oss的accesskeyid
- `REGISTRY_STORAGE_OSS_ACCESSKEYSECRET`: 访问oss的accesskeysecret
- `REGISTRY_STORAGE_OSS_REGION`: 访问oss的region, 比如上海是oss-cn-shanghai
- `REGISTRY_STORAGE_OSS_BUCKET`: 访问oss的bucket
- `REGISTRY_STORAGE_OSS_INTERNAL`: 是否使用内部网络，使用请设置为true，使用内部网络，从阿里云ecs到oss不产生流量费。
- `REGISTRY_STORAGE_OSS_SECURE`: *如果使用内部网络，这里要设置为false*, 这里有坑坑坑，以为设置了上面的`REGISTRY_STORAGE_OSS_INTERNAL`为true就可以了，结果就是无法正常使用内部网络。而增加这个为false后，就正常了

4.启动服务

启动服务之前，先把registry仓库域名解析到服务器ip，并保证解析正确可以通过域名正常访问。
```
$ cd /srv/app/registry
$ docker-compose up -d
```
5.测试

```
$ docker login registry.yourdomain.com
$ docker pull hahoo/node-hello
$ docker tag hahoo/node-hello registry.yourdomain.com/node-hello
$ docker push registry.yourdomain.com/node-hello
```
- `registry.yourdomain.com`: registry仓库域名
