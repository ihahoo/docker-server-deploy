## nginx + docker-gen下部署站点

[nginx-dockergen-letsencrypt](https://github.com/ihahoo/docker-server-deploy/tree/master/nginx-dockergen-letsencrypt) 部署好之后，可以部署站点，实现nginx的自动发现及设置反向代理。如果在配置某一个服务，需要nginx反向代理的话，参考以下docker-compose的模板文件。

单容器模板：[docker-compose.yml](https://github.com/ihahoo/docker-server-deploy/blob/master/nginx-dockergen-web/docker-compose.yml)

多容器模板：[docker-compose.muti.yml](https://github.com/ihahoo/docker-server-deploy/blob/master/nginx-dockergen-web/docker-compose.muti.yml)

```
...

    environment:
      - VIRTUAL_HOST=www.yourdomain.com
      - LETSENCRYPT_HOST=www.yourdomain.com
      - LETSENCRYPT_EMAIL=mail@emaildomain.com
    networks:
      - nginx-proxy

...

networks:
  nginx-proxy:
    external:
      name: nginx-proxy
```
通过变量`VIRTUAL_HOST`让nginx可以发现，自动进行反向代理配置。如果需要https证书，可以设置`LETSENCRYPT_HOST`和`LETSENCRYPT_EMAIL`变量。

设置了networks为`external`名称为`nginx-proxy`, 这个名称是在配置 [nginx-dockergen-letsencrypt](https://github.com/ihahoo/docker-server-deploy/tree/master/nginx-dockergen-letsencrypt) 的时候，设置的网络名称。这样就可以通过docker-gen发现并自动配置nginx反向代理设置了。如果不用这种方式，启动多容器联合的docker-compose启动的时候，会遇到错误，指定了外部网络是nginx所在的网络`nginx-proxy`就没问题了。

更多变量和配置，请查看[https://github.com/jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy)

如果设置了`LETSENCRYPT_HOST`和`LETSENCRYPT_EMAIL`，但是certs下没有证书，请检查域名是否正确解析到了服务器。
