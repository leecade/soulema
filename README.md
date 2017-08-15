# soulema

## online

https://soulema.com

## usage

```sh
$ docker run -d --name soulema -p 8888:80 leecade/soulema
```

> see more `docker-compose.yml`

## Production steps

### Install nginx

```sh
$ wget https://nginx.org/packages/mainline/centos/7/x86_64/RPMS/nginx-1.13.4-1.el7.ngx.x86_64.rpm
$ rpm -ivh nginx-1.13.4-1.el7.ngx.x86_64.rpm
$ systemctl enable nginx
$ systemctl start nginx
```

### Generate certificates

```sh
mkdir -p /data/letsencrypt

docker run --rm -ti -v /data/letsencrypt:/etc/letsencrypt/ -p 443:443 certbot/certbot certonly --standalone -d soulema.com -d www.soulema.com

# renew
# docker run --rm -ti -v /data/letsencrypt:/etc/letsencrypt/ -p 443:443 certbot/certbot renew
```

### Configure proxy_pass

```nginx
upstream soulema {
  server 127.0.0.1:8898;
}

server {
  listen 80;
  listen 443 ssl http2;
  server_name soulema.com www.soulema.com;
  add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; preload";

  if ($host = "soulema.com") {
    return 301 https://www.soulema.com$request_uri;
  }

  location / {
    proxy_pass http://soulema;
  }

  ssl_certificate     /data/letsencrypt/live/soulema.com/fullchain.pem;
  ssl_certificate_key /data/letsencrypt/live/soulema.com/privkey.pem;
  ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
}
```
