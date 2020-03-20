# Graylog Post Deployment Configuration Steps

## 1. Copy certificates and Truststore

The following will copy the certs to the dir that is mounted inside the container

```
[root@graylogvm config]# cp ~/cert_files/server* /var/lib/docker/volumes/graylog_graylog-volume/_data/config
[root@graylogvm config]# chown 1100:1100 /var/lib/docker/volumes/graylog_graylog-volume/_data/config/server*
[root@graylogvm config]# cd /var/lib/docker/volumes/graylog_graylog-volume/_data/config/
```

Note the full path of server.crt and server.key file. **You will need this information later**.

As the directory is mounted at the container persistent volume, the the following files should be present inside the container:

```
/usr/share/graylog/data/config/server.crt
/usr/share/graylog/data/config/server.key
```

## 2. Create the Graylog required inputs (importing the extractors)

Inside the Graylog UI, you'll need to specify the syslog input ports listed below and also import the required extractors provided on [graylog_extractors.json](../../config_files/graylog_extractors.json).

```
GRAYLOG INPUT, CREATE ONE FOR SYSLOG_UDP port 515
GRAYLOG INPUT, CREATE ONE FOR SYSLOG_TCP port 1514
GRAYLOG INPUT, CREATE ONE FOR SYSLOG_TLS port 1515. INFORM THE KEY AND CRT. USE NOAUTH
GRAYLOG INPUT, CREATE ONE FOR BEATS port 1516
```

**If using BEATS you can configure TLS the same way as SYSLOG. When installing Winlogbeat on Windows clients, don't forget to "LOGSTASH" output to Graylog IP on port 1516**.

## 3. Adding IP Geo-location to your messsages on Graylog

```
[root@graylogvm config]# cd /var/lib/docker/volumes/graylog_graylog-volume/_data/config/
[root@graylogvm config]# wget https://geolite.maxmind.com/download/geoip/database/GeoLite2-City.tar.gz
[root@graylogvm config]# tar -zxvf GeoLite2-City.tar.gz
```

As the directory is mounted at the container persistent volume, the the following files should be present inside the container:

```
#/usr/share/graylog/data/config/GeoLite2-City_20190618/GeoLite2-City.mmdb
```

## 3.1. Create Lookup Tables and Adapters

Through the Graylog UI at the Lookup Tables create a new data adapter named `Geolocation`, inform the `GeoLite2-City.mmdb` path, and set `DBTYPE= City Database`.

Create a cache with default values named `cache_01`, then create a lookup table named `Geolocation_table`, and select the cache and adapter you've just created.

## 3.2. Create a Pipeline

Through the Graylog UI at PIPELINES page, select `CREATE PIPELINE` and name it as `Geolocation`, then add the following rule to extract the Geo-IP:

```
rule "message_ip geoip lookup"
when
  has_field("message_ip")
then
  let geo = lookup("geolocation_table", to_string($message.message_ip));
  set_field("message_ip_geolocation", geo["coordinates"]);
  set_field("message_ip_country_code", geo["country"].iso_code);
  set_field("message_ip_country_name", geo["country"].names.en);
  set_field("message_ip_city_name", geo["city"].names.en);
end
```

Then attach the pipeline that we just created to the "All messages" stream.

## 3.2. Enable the Geo-Location Processor

Through the Graylog UI, at the `SYSTEM CONFIGURATION` page, select `ENABLE Geo-Location Processor` and inform the path for the `GeoLite2-City.mmdb` file.
