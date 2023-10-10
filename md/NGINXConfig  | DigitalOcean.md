> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.digitalocean.com](https://www.digitalocean.com/community/tools/nginx#?)

> The easiest way to configure a performant, secure, and stable nginx server.

### /etc/nginx/nginx.conf

```
cd /etc/nginx
```

### /etc/nginx/sites-available/example.com.conf

```
tar -czvf nginx_$(date +'%F_%H-%M-%S').tar.gz nginx.conf sites-available/ sites-enabled/ nginxconfig.io/
```

### /etc/nginx/nginxconfig.io/letsencrypt.conf

```
tar -xzvf nginxconfig.io-example.com.tar.gz | xargs chmod 0644
```

### /etc/nginx/nginxconfig.io/security.conf

```
openssl dhparam -out /etc/nginx/dhparam.pem 2048
```

### /etc/nginx/nginxconfig.io/general.conf

```
mkdir -p /var/www/_letsencrypt
```

### /etc/nginx/nginxconfig.io/php_fastcgi.conf

```
chown www-data /var/www/_letsencrypt
```

### /etc/nginx/nginxconfig.txt

```
sed -i -r 's/(listen .*443)/\1; #/g; s/(ssl_(certificate|certificate_key|trusted_certificate) )/#;#\1/g; s/(server \{)/\1\n    ssl off;/g' /etc/nginx/sites-available/example.com.conf
```