# HAProxy Post Deployment Steps

## 1. Merge the certificate and key into the same file at the HAProxy persistent volume directory

Get the files generated at the "Generating Required Certificates" procedure and merge them as shown in the example below:

```
cat ~/cert_files/server.crt ~/cert_files/server.key > /var/lib/docker/volumes/haproxy_haproxy-volume/_data/laas.crt
```

## Copy the HAProxy configuration file from template into the HAProxy persistent volume directory

```
cp ~/haproxy.cfg /var/lib/docker/volumes/haproxy_haproxy-volume/_data/haproxy.cfg
```

The following content can be used as an example for your file, please consider changing the password of this file at the `Userlist` section:

```
#------------------
# Global settings
#------------------
global
    tune.ssl.default-dh-param   2048
    ssl-default-bind-ciphers    ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS
    ssl-default-bind-options    no-sslv3 no-tlsv10
    # no-tlsv11

#------------------
# common defaults that all the 'listen' and 'backend' sections will
# use- if not designated in their block
#------------------
defaults
    log global
    mode http
    retries 3
    option      http-keep-alive
    option      httplog
    option      dontlognull
    option      forwardfor
    option      http-server-close
#    log-format %ci:%cp\ [%t]\ %ft\ %b/%s\ %Tq/%Tw/%Tc/%Tr/%Tt\ %ST\ %B\ %CC\ %CS\ %tsc\ %ac/%fc/%bc/%sc/%rc\ %sq/%bq\ %hr\ %hs\ %{+Q}r
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
#------------------
# internal statistics
#------------------
listen stats
    bind 0.0.0.0:8998
    mode http
    stats auth admin:admin
    stats enable
    stats realm Haproxy\ Statistics
    stats refresh 20s
    stats uri /admin?stats
#------------------
# frontend instances
#------------------
frontend unsecured
    bind        *:80
    http-request set-header X-Forwarded-Proto https
    redirect    scheme https code 301 if !{ ssl_fc }
    default_backend www-backend
    
frontend www-https
    bind        *:443 ssl crt /usr/local/etc/haproxy/laas.crt
    log-format %ci:%cp\ [%t]\ %ft\ %b/%s\ %Tq/%Tw/%Tc/%Tr/%Tt\ %ST\ %B\ %CC\ %CS\ %tsc\ %ac/%fc/%bc/%sc/%rc\ %sq/%bq\ %hr\ %hs\ %{+Q}r\ ssl_version:%sslv\ ssl_cipher:%sslc
    # passing on that browser is using https
    http-request set-header X-Forwarded-Proto https
    # for Clickjacking
    http-response add-header X-Frame-Options SAMEORIGIN
    # prevent browser from using non-secure
    http-response add-header Strict-Transport-Security max-age=15768000
    default_backend www-backend
    
frontend elasticsearch
    bind        *:9001 ssl crt /usr/local/etc/haproxy/laas.crt
    log-format %ci:%cp\ [%t]\ %ft\ %b/%s\ %Tq/%Tw/%Tc/%Tr/%Tt\ %ST\ %B\ %CC\ %CS\ %tsc\ %ac/%fc/%bc/%sc/%rc\ %sq/%bq\ %hr\ %hs\ %{+Q}r\ ssl_version:%sslv\ ssl_cipher:%sslc
    # passing on that browser is using https
    http-request set-header X-Forwarded-Proto https
    # for Clickjacking
    http-response add-header X-Frame-Options SAMEORIGIN
    # prevent browser from using non-secure
    http-response add-header Strict-Transport-Security max-age=15768000 
    default_backend elasticsearch
    
frontend kibana
    bind        *:9002 ssl crt /usr/local/etc/haproxy/laas.crt
    log-format %ci:%cp\ [%t]\ %ft\ %b/%s\ %Tq/%Tw/%Tc/%Tr/%Tt\ %ST\ %B\ %CC\ %CS\ %tsc\ %ac/%fc/%bc/%sc/%rc\ %sq/%bq\ %hr\ %hs\ %{+Q}r\ ssl_version:%sslv\ ssl_cipher:%sslc
    # passing on that browser is using https
    http-request set-header X-Forwarded-Proto https
    # for Clickjacking
    http-response add-header X-Frame-Options SAMEORIGIN
    # prevent browser from using non-secure
    http-response add-header Strict-Transport-Security max-age=15768000
    default_backend kibana

frontend cerebro
    bind        *:9003 ssl crt /usr/local/etc/haproxy/laas.crt
    log-format %ci:%cp\ [%t]\ %ft\ %b/%s\ %Tq/%Tw/%Tc/%Tr/%Tt\ %ST\ %B\ %CC\ %CS\ %tsc\ %ac/%fc/%bc/%sc/%rc\ %sq/%bq\ %hr\ %hs\ %{+Q}r\ ssl_version:%sslv\ ssl_cipher:%sslc
    # passing on that browser is using https
    http-request set-header X-Forwarded-Proto https
    # for Clickjacking
    http-response add-header X-Frame-Options SAMEORIGIN
    # prevent browser from using non-secure
    http-response add-header Strict-Transport-Security max-age=15768000
    default_backend cerebro
     
#------------------
# Userlist
#------------------
userlist UsersFor_ElasticsearchID
  user kibana insecure-password k1b4n4
#------------------
# backend instances
#------------------
backend www-backend
    balance roundrobin
    cookie WWWID insert indirect nocache
    redirect scheme https if !{ ssl_fc }
    server mypool1 127.0.0.1:8080 cookie www1 check inter 3000
    #server mypool2 127.0.0.1:8080 cookie www2 check inter 3000
    #server mypool3 127.0.0.1:8080 cookie www3 check inter 3000

backend elasticsearch
    mode http
    option forwardfor
    balance source
    option httpclose
    acl AuthOkay_ElasticsearchID http_auth(UsersFor_ElasticsearchID)
    http-request auth realm ElasticsearchID if !AuthOkay_ElasticsearchID
    server elasticsearch1 elasticsearch:9200 weight 1 check inter 1000 rise 5 fall 1

backend kibana
    mode http
    option forwardfor
    balance source
    option httpclose
    acl AuthOkay_ElasticsearchID http_auth(UsersFor_ElasticsearchID)
    http-request auth realm ElasticsearchID if !AuthOkay_ElasticsearchID
    server kibana1 kibana:5601 weight 1 check inter 1000 rise 5 fall 1
    
backend cerebro
    mode http
    option forwardfor
    balance source
    option httpclose
    acl AuthOkay_ElasticsearchID http_auth(UsersFor_ElasticsearchID)
    http-request auth realm ElasticsearchID if !AuthOkay_ElasticsearchID
    server cerebro1 cerebro:9000 weight 1 check inter 1000 rise 5 fall 1

```
