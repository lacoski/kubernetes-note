root@devtest-traefik:~# htpasswd -nb admin secure_password
admin:$apr1$ndF93rnr$F4AtC.kmfxhN8HijGX9L..

root@devtest-traefik:~# cat traefik.toml
[entryPoints]
  [entryPoints.web]
    address = ":80"
    [entryPoints.web.http.redirections.entryPoint]
      to = "websecure"
      scheme = "https"

  [entryPoints.websecure]
    address = ":443"
[api]
  dashboard = true
[certificatesResolvers.lets-encrypt.acme]
  email = "thanhnb@nhanhoa.com.vn"
  storage = "acme.json"
  [certificatesResolvers.lets-encrypt.acme.tlsChallenge]
[providers.docker]
  watch = true
  network = "web"
[providers.file]
  filename = "traefik_dynamic.toml"


root@devtest-traefik:~# cat traefik_dynamic.toml
[http.middlewares.simpleAuth.basicAuth]
  users = [
    "admin:$apr1$ndF93rnr$F4AtC.kmfxhN8HijGX9L.."
  ]
[http.routers.api]
  rule = "Host(`wpc.devopsviet.com`)"
  entrypoints = ["websecure"]
  middlewares = ["simpleAuth"]
  service = "api@internal"
  [http.routers.api.tls]
    certResolver = "lets-encrypt"

root@devtest-traefik:~# docker network create web
19c04a262de4359f8a7ccd4c2b3320b6dda2cfc68307b2ba308a4d40fee3b915
root@devtest-traefik:~# touch acme.json
root@devtest-traefik:~# 
root@devtest-traefik:~# chmod 600 acme.json


https://wpc.devopsviet.com/dashboard/#/


root@devtest-traefik:~# cat docker-compose.yml
version: "3"

networks:
  web:
    external: true
  internal:
    external: false

services:
  blog:
    image: wordpress:4.9.8-apache
    environment:
      WORDPRESS_DB_PASSWORD:
    labels:
      - traefik.http.routers.blog.rule=Host(`wp1.devopsviet.com`)
      - traefik.http.routers.blog.tls=true
      - traefik.http.routers.blog.tls.certresolver=lets-encrypt
      - traefik.port=80
    networks:
      - internal
      - web
    depends_on:
      - mysql
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD:
    networks:
      - internal
    labels:
      - traefik.enable=false

  adminer:
    image: adminer:4.6.3-standalone
    labels:
    labels:
      - traefik.http.routers.adminer.rule=Host(`db-admin.devopsviet.com`)
      - traefik.http.routers.adminer.tls=true
      - traefik.http.routers.adminer.tls.certresolver=lets-encrypt
      - traefik.port=8080
    networks:
      - internal
      - web
    depends_on:
      - mysql


root@devtest-traefik:~# export WORDPRESS_DB_PASSWORD=secure_database_password_demo
root@devtest-traefik:~# export MYSQL_ROOT_PASSWORD=secure_database_password_demo

https://www.digitalocean.com/community/tutorials/how-to-use-traefik-v2-as-a-reverse-proxy-for-docker-containers-on-ubuntu-20-04

https://viblo.asia/p/tong-quan-ve-traefik-XL6lAA8Dlek

curl -H Host:whoami.docker.localhost http://127.0.0.1
