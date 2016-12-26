# docker-server-deploy
服务器全部使用docker和docker-compose部署应用, 请首先在你的服务器中安装[docker](https://docs.docker.com/)和[docker-compose](https://docs.docker.com/compose/install/)

## 1.[nginx + docker-gen + letsencrypt](https://github.com/ihahoo/docker-server-deploy/tree/master/nginx-dockergen-letsencrypt)
- 通过nginx反向代理，配置虚机共享80和443端口通过域名映射到不同的服务
- 通过docker-gen，配置nginx的自发现服务
- 自动配置 [let's encrypt](https://letsencrypt.org/) https证书，自发现，并且可以自动更新证书。

[详情请点这里查看](https://github.com/ihahoo/docker-server-deploy/tree/master/nginx-dockergen-letsencrypt)
