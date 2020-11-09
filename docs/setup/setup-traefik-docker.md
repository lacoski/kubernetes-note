# Hướng dẫn cài đặt Traefix trên Ubuntu 20.04

## Tổng quan

Traefik là một Reverse-proxy đời mới, và cũng là load balancer để làm cho việc deploy hệ thống microservice được trở lên dễ dàng hơn. Tích hợp rất nhiều các thành phần infrastructure như Docker, Swarm mode, Kubernetes, Marathon, Consul, Etcd, Rancher, Amazon ECS... Và tính tự động là điểm quan trọng nhất trong các config với traefik.

Xem thêm:
- https://viblo.asia/p/tong-quan-ve-traefik-XL6lAA8Dlek
- https://doc.traefik.io/traefik/

## Chuẩn bị

Chuẩn bị VM đáp ứng các yêu cầu sau:
- Sử dụng OS Ubuntu 20.04
- Cài dịch vụ Docker và Docker Compose
- Có IP Public
- Cấu hình 2 Core - 2 GB Ram - 25GB Disk
- Cần trỏ 3 domain tới địa chỉ IP Public của VM với các tiền tố: monitor, wp1, db-admin

VM có địa chỉ IP Public là: 103.124.94.89 và đã trỏ `monitor.devopsviet.com, wp1.devopsviet.com, db-admin-wp1.devopsviet.com, wp2.devopsviet.com, db-admin-wp2.devopsviet.com`

## Phần 1: Chuẩn bị các file cấu hình Traefix

### Bước 1: Tạo thư mục chưa các config
```
mkdir -p /root/traefix
cd /root/traefix
```

### Bước 2: Sinh mật khẩu login http-auth Traefix

```
sudo apt-get -y install apache2-utils
htpasswd -nb admin Cloud365a@123
```

Kết quả
```
root@devtest-traefik:~/traefix# htpasswd -nb admin Cloud365a@123
admin:$apr1$jfEBf99n$/usOROpFWDvM5.U/Gz3oY0
```

Lưu ý:
- `admin:$apr1$jfEBf99n$/usOROpFWDvM5.U/Gz3oY0` - Giá trị này sẽ được sử dụng ở các bước tiếp theo

### Bước 3: Tạo File cấu hình `traefik.toml`

Tạo mới file `traefik.toml` với với nội dung
```
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
```

Lưu ý:
- Tại Section `[certificatesResolvers.lets-encrypt.acme] - email`: Địa chỉ Email xác thực let encrypt

Kết quả
```
root@devtest-traefik:~/traefix# cat traefik.toml 
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
```

### Bước 3: Tạo File cấu hình `traefik_dynamic.toml`

Tạo mới file `traefik_dynamic.toml` với với nội dung
```
[http.middlewares.simpleAuth.basicAuth]
  users = [
    "admin:$apr1$jfEBf99n$/usOROpFWDvM5.U/Gz3oY0"
  ]
[http.routers.api]
  rule = "Host(`monitor.devopsviet.com`)"
  entrypoints = ["websecure"]
  middlewares = ["simpleAuth"]
  service = "api@internal"
  [http.routers.api.tls]
    certResolver = "lets-encrypt"
```

Lưu ý:
- `users = [ <User>:<Password> ]`: Khai báo tài khoản Http auth sử dụng login Traefix Dashboard
  - Sử dụng kết quả có được từ `Bước 2` 
- `monitor.devopsviet.com` - Địa chỉ Domain sử dụng để truy cập Traefix Dashboard

Kết quả
```
root@devtest-traefik:~/traefix# cat traefik_dynamic.toml 
[http.middlewares.simpleAuth.basicAuth]
  users = [
    "admin:$apr1$jfEBf99n$/usOROpFWDvM5.U/Gz3oY0"
  ]
[http.routers.api]
  rule = "Host(`monitor.devopsviet.com`)"
  entrypoints = ["websecure"]
  middlewares = ["simpleAuth"]
  service = "api@internal"
  [http.routers.api.tls]
    certResolver = "lets-encrypt"
```

## Phần 2: Khởi tạo Traefix Container

### Bước 1: Tạo mới dải Network có tên `web`
```
docker network create web
```

Kết quả
```
root@devtest-traefik:~/traefix# docker network create web
ca17ae38811aab786d9672a0139a268b59656afa8484dcc521cfb9b3bff46e5c
```

### Bước 2: Tạo mới file `acme.json`
```
touch acme.json
chmod 600 acme.json
```

Kết quả
```
root@devtest-traefik:~/traefix# ll
total 16
drwxr-xr-x 2 root root 4096 Nov  7 15:13 ./
drwx------ 5 root root 4096 Nov  7 15:11 ../
-rw------- 1 root root    0 Nov  7 15:20 acme.json
-rw-r--r-- 1 root root  487 Nov  7 14:56 traefik.toml
-rw-r--r-- 1 root root  314 Nov  7 15:11 traefik_dynamic.toml
```

### Bước 3: Tạo Traefix container
```
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $PWD/traefik.toml:/traefik.toml \
  -v $PWD/traefik_dynamic.toml:/traefik_dynamic.toml \
  -v $PWD/acme.json:/acme.json \
  -p 80:80 \
  -p 443:443 \
  --network web \
  --name traefik \
  traefik:v2.2
```

Kết quả
```
root@devtest-traefik:~/traefix# docker run -d \
>   -v /var/run/docker.sock:/var/run/docker.sock \
>   -v $PWD/traefik.toml:/traefik.toml \
>   -v $PWD/traefik_dynamic.toml:/traefik_dynamic.toml \
>   -v $PWD/acme.json:/acme.json \
>   -p 80:80 \
>   -p 443:443 \
>   --network web \
>   --name traefik \
>   traefik:v2.2
Unable to find image 'traefik:v2.2' locally
v2.2: Pulling from library/traefik
cbdbe7a5bc2a: Pull complete 
f16506d32a25: Pull complete 
605303653d66: Pull complete 
a9005a35b171: Pull complete 
Digest: sha256:ea0aa8832bfd08369166baecd40b35fc58979df8f5dc5182e4e63ee6adbe66db
Status: Downloaded newer image for traefik:v2.2
479ea854f0b004f52a51de19df7db46b8163e67d66e137a7b4d5b1e85807cd0f
```

### Bước 4: Truy cập giao diện Traefix Dashboard

Truy cập đường dẫn `https://monitor.your_domain/dashboard/`, trong bài đường dẫn của mình là ``

![](/images/setup/setup-traefik-docker/pic1.png)

Nhập mật khẩu `admin / Cloud365a@123` (xem lại `Phần 1`) và đăng nhập

Kết quả

![](/images/setup/setup-traefik-docker/pic2.png)

## Phần 3: Sử dụng Traefix làm Reverse Proxy cho Service WordPress

### Bước 1: Khởi tạo Site WP1

Tạo mới thư mục cho site Wordpress 1
```
mkdir -p /root/traefix/wp1
cd /root/traefix/wp1
```

Tạo mới file docker-compose `docker-compose.yml`
```
version: "3"

networks:
  web:
    external: true
  internal1:
    external: false

services:
  wp1:
    image: wordpress:4.9.8-apache
    environment:
      WORDPRESS_DB_PASSWORD:
    labels:
      - traefik.http.routers.wp1.rule=Host(`wp1.devopsviet.com`)
      - traefik.http.routers.wp1.tls=true
      - traefik.http.routers.wp1.tls.certresolver=lets-encrypt
      - traefik.port=80
    networks:
      - internal1
      - web
    depends_on:
      - mysql
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD:
    networks:
      - internal1
    labels:
      - traefik.enable=false

  adminerwp1:
    image: adminer:4.6.3-standalone
    labels:
    labels:
      - traefik.http.routers.adminerwp1.rule=Host(`db-admin-wp1.devopsviet.com`)
      - traefik.http.routers.adminerwp1.tls=true
      - traefik.http.routers.adminerwp1.tls.certresolver=lets-encrypt
      - traefik.port=8080
    networks:
      - internal1
      - web
    depends_on:
      - mysql
```

Đặt mật khẩu database
```
export WORDPRESS_DB_PASSWORD=cloud3652020
export MYSQL_ROOT_PASSWORD=cloud3652020
```

Khởi động WP 1
```
docker-compose up -d
```

Kết quả
```
root@devtest-traefik:~/traefix/wp1# docker-compose up -d

Creating network "wp1_internal1" with the default driver
Creating wp1_mysql_1 ... done
Creating wp1_adminerwp1_1 ... done
Creating wp1_wp1_1        ... done
```

Truy cập site wp1 và setup Wordpress cơ bản. Đường dẫn https://wp1.devopsviet.com/

![](/images/setup/setup-traefik-docker/pic3.png)

Kết quả

![](/images/setup/setup-traefik-docker/pic4.png)

Truy cập trang quản trị DB https://db-admin-wp1.devopsviet.com/

Lưu ý: 
- Server: mysql
- Username: root
- Password: cloud3652020
- Database: Để trắng

![](/images/setup/setup-traefik-docker/pic5.png)

Kết quả

![](/images/setup/setup-traefik-docker/pic6.png)

### Bước 2: Khởi tạo Site WP2

Tạo mới thư mục cho site Wordpress 2
```
mkdir -p /root/traefix/wp2
cd /root/traefix/wp2
```

Tạo mới file docker-compose `docker-compose.yml`
```
version: "3"

networks:
  web:
    external: true
  internal2:
    external: false

services:
  wp2:
    image: wordpress:4.9.8-apache
    environment:
      WORDPRESS_DB_PASSWORD:
    labels:
      - traefik.http.routers.wp2.rule=Host(`wp2.devopsviet.com`)
      - traefik.http.routers.wp2.tls=true
      - traefik.http.routers.wp2.tls.certresolver=lets-encrypt
      - traefik.port=80
    networks:
      - internal2
      - web
    depends_on:
      - mysql
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD:
    networks:
      - internal2
    labels:
      - traefik.enable=false

  adminerwp2:
    image: adminer:4.6.3-standalone
    labels:
    labels:
      - traefik.http.routers.adminerwp2.rule=Host(`db-admin-wp2.devopsviet.com`)
      - traefik.http.routers.adminerwp2.tls=true
      - traefik.http.routers.adminerwp2.tls.certresolver=lets-encrypt
      - traefik.port=8080
    networks:
      - internal2
      - web
    depends_on:
      - mysql
```

Khởi động site WP2
```
docker-compose up -d
```

Kết quả
```
root@devtest-traefik:~/traefix/wp2# docker-compose up -d

Creating network "wp2_internal2" with the default driver
Creating wp2_mysql_1 ... done
Creating wp2_adminerwp2_1 ... done
Creating wp2_wp2_1        ... done

```

Truy cập site wp2 và setup Wordpress cơ bản https://wp2.devopsviet.com/

![](/images/setup/setup-traefik-docker/pic7.png)

Kết quả

![](/images/setup/setup-traefik-docker/pic8.png)

Truy cập trang quản trị DB https://db-admin-wp2.devopsviet.com/

Lưu ý: 
- Server: mysql
- Username: root
- Password: cloud3652020
- Database: Để trắng

![](/images/setup/setup-traefik-docker/pic9.png)

Kết quả

![](/images/setup/setup-traefik-docker/pic10.png)

## Nguồn

https://www.digitalocean.com/community/tutorials/how-to-use-traefik-v2-as-a-reverse-proxy-for-docker-containers-on-ubuntu-20-04
