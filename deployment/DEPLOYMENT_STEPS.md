# Deployment Steps

## Requirements and Considerations

- Before you can start you need a Linux server with Docker using a version that supports **SWARM MODE**. Please refer to docker documentation for more information about it.
- You'll also need a reasable amount of space on `/var/lib/docker/volumes` to start, you'll need to estimate the size of your daily ingestion of logs in order to avoid running into lack of disk space issues.

## Install Process

### 1. Initialize the swarm

```
[root@laas-lab-local-1 ~]# docker swarm init
Swarm initialized: current node (a8pml3unconooa7t7qnsw7knp) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5jk292hj9b12by0byz4xfnntq07nd58ozrqxxyjx91kx03oqhw-8pl8cm17pj1lam8lq10gmuih0 192.168.122.60:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

### 2. Create an overlay network

```
[root@laas-lab-local-1 ~]# docker network create --driver overlay swarm-overlay-1

[root@laas-lab-local-1 ~]# docker network list
NETWORK ID          NAME                DRIVER              SCOPE
73b277c1972a        bridge              bridge              local
f417626507a7        docker_gwbridge     bridge              local
c713290dd2c9        host                host                local
sfedh8kug91b        ingress             overlay             swarm
a3a6a6de924b        none                null                local
j28vui73v0hl        swarm-overlay-1     overlay             swarm

```

### 3. Customize the environment variables file

Update `~/LaaS/deployment/docker-swarm-compose/01-environment` according to your setup as shown in the example below:

```
[root@laas-lab-local-1 ~]# cat 01-environment
SWARMNODE=graylogvm
SWARMOVERLAYNETWORK=swarm-overlay-1
GRAYLOGPWSECRET=somepasswordpepper
GRAYLOGPWSHA2ITSELF=4bba0654175e3bf16cb6ea4398b082a483477b0bec4e3f977963193c496e8417
GRAYLOGURI=http://skullkid.dlinkddns.com:3300/
GRAYLOGTRUSTSTOREPW=setyourkeystorepw
GRAYLOGTZ=America/Sao_Paulo
```

To generate the value for `GRAYLOGPWSHA2ITSELF` variable which will be your Graylog admin password, perform the following command:

```
[root@laas-lab-local-1 ~]# echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```

**Its very important to notice that the environment variables file should be properly updated before starting the installation process or else the whole operation will fail.**

After updating the file with your information, you'll need to export the variables located on `~/LaaS/deployment/docker-swarm-compose/01-environment`, as shown in the example below:

```
[root@laas-lab-local-1 ~]# export $(cat 01-environment)
```

### 4. Deployment Steps

 Now with your setup information properly customized and environment variables already exported, you're ready to start the deployment.  Execute the following to start deploying each of the different tools of the stack:

#### 4.1. Grafana

```
[root@laas-lab-local-1 ~]# docker stack deploy --compose-file docker-stack-grafana.yml grafana
```

To reset the admin password, execute the following:

```
[root@laas-lab-local-1 ~]# docker exec -it $(docker ps | grep grafana | awk '{ print $1 }') /bin/bash
$ grafana-cli admin reset-admin-password <password>
```

**Optional**: After deployment you can import one Dashboard to serve as example from file [grafana-graylog-dashboard-example.json](../config_files/grafana-graylog-dashboard-example.json).

#### 4.2. Elasticsearch

Before you start, you'll need to increase the `vm.max_map_count`, as shown in the example below:

```
[root@laas-lab-local-1 ~]# sysctl -w vm.max_map_count=262144
```

Them start the deployment with the following command:

```
[root@laas-lab-local-1 ~]# docker stack deploy --compose-file docker-stack-elasticsearch.yml elasticsearch
```

#### 4.3. Graylog

```
[root@laas-lab-local-1 ~]# docker stack deploy --compose-file docker-stack-graylog.yml graylog
```

Please notice that in order to complete the Graylog deployment, you'll need to perform some further actions after all the applications are properly started and running. See detailed information at the  GRAYLOG post deployment steps indicated below:

- [Graylog Post-Install Configuration Steps](configuration/03-graylog_postinstall.md)

#### 4.4. HAProxy

```
[root@laas-lab-local-1 ~]# docker stack deploy --compose-file docker-stack-haproxy.yml haproxy
```

Please notice that in order to complete the HAProxy deployment, you'll need to perform some further actions after all the applications are properly started and running. See detailed information at the  HAPRoxy post deployment steps indicated below:

- [HAProxy Post-Install Configuration Steps](./configuration/02-haproxy.md)


It is expected that the HAProxy container will keep restarting by itself untill the post deployment steps are completed.

#### 4.5. Checking the entire Deployment Status

After performing the steps indicated before, your deployment should look like this:

```
root@graylogvm _data]# docker stack ls
NAME                SERVICES            ORCHESTRATOR
elasticsearch       3                   Swarm
grafana             1                   Swarm
graylog             2                   Swarm
haproxy             1                   Swarm
[root@graylogvm _data]# docker service ls
ID                  NAME                          MODE                REPLICAS            IMAGE                                                 PORTS
yjcaxx1s3fo6        elasticsearch_cerebro         replicated          1/1                 lmenezes/cerebro:0.8.3                                
tmlun41r702s        elasticsearch_elasticsearch   replicated          1/1                 docker.elastic.co/elasticsearch/elasticsearch:6.6.2   *:9200->9200/tcp, *:9300->9300/tcp
izcgsiq6hzt4        elasticsearch_kibana          replicated          1/1                 docker.elastic.co/kibana/kibana:6.6.2                 *:5601->5601/tcp
j2w33xsedo7c        grafana_grafana               replicated          1/1                 grafana/grafana:6.2.4                                 *:3000->3000/tcp
xpnam3xxgpm7        graylog_graylog               replicated          1/1                 graylog/graylog:3.0                                   *:1514-1516->1514-1516/tcp, *:9000->9000/tcp, *:12201->12201/tcp, *:514->514/udp, *:12201->12201/udp
s13rxq427ueu        graylog_mongo                 replicated          1/1                 mongo:3                                               
rms3jjy9j0n9        haproxy_haproxy               replicated          1/1                 haproxy:latest                                        *:9001-9003->9001-9003/tcp
[root@graylogvm _data]# 
```

### 5. Configuration Steps

After completing the deployment steps, the following post-install procedures should be executed:

1. [Generating Required Certificates](./configuration/01-certificates.md)
2. [HAProxy Post Deployment Steps](./configuration/02-haproxy.md)
3. [Graylog Post Deployment Configuration Steps](./configuration/03-graylog_postinstall.md)
4. [RSYSLOG Client Configuration Steps](./configuration/04-rsyslog_client_config.md)
