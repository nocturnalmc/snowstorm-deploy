## SNOMED CT TERMINOLOGY SERVER - SNOWSTORM (PRODUCTION READY DEPLOYMENT FOR DOCKER SWARM)

- Snowstorm repository here https://github.com/IHTSDO/snowstorm
- This production ready deployment is tailor made for docker swarm, if you're not familiar with docker swarm please give a read here https://docs.docker.com/engine/swarm/
- Step by step for production ready deployment are given below
- The final setup of this SNOMED CT Terminology Server cluster will consist of
  - 3 node clusters Elasticsearch with high availability
  - 2 Snowstorm terminology servers with read only config connected to all 3 node cluster Elasticsearch
  - 1 Snowstorm terminology server with read & write config connected to all 3 node cluster Elasticsearch (for maintenance, not for production use)
  - 1 SNOMED CT browser connected to Snowstorm terminology server with read & write config (for maintenance, not for production use)

### DEPLOYMENT STEPS

#### 1. Make sure to set vm.max_map_count as Elasticsearch will require it to be set for it to run in docker container. Docs can be followed here https://github.com/IHTSDO/snowstorm/blob/master/docs/using-docker.md

#### 2. Create overlay network with --attachable flag so all the containers can connect to it later

```bash
docker network create --driver=overlay --attachable snowstorm-network
```

#### 3. Deploy 3 node Elasticsearch clusters one by one in each VM if you have many of them

- Make sure your Elasticsearch container status health is in (healthy) state before you spawn another Elasticsearch container

```bash
docker ps -a
897eeb01f01f   docker.elastic.co/elasticsearch/elasticsearch:8.11.1   "/bin/tini -- /usr/l…"   27 minutes ago   Up 27 minutes (healthy)   9200/tcp, 9300/tcp                          snowstorm-es01
```

```bash
docker compose -f snowstorm-es01/docker-compose.yml up -d
```

```bash
docker compose -f snowstorm-es02/docker-compose.yml up -d
```

```bash
docker compose -f snowstorm-es03/docker-compose.yml up -d
```

#### 4. Deploy read & write Snowstorm terminology server & SNOMED CT browser

```bash
docker compose -f snowstorm-write-browser/docker-compose.yml up -d
```

- Browse to http://your-server-ip:5000 for Snowstorm Swagger API and upload your RF2 file following this docs https://github.com/IHTSDO/snowstorm/blob/master/docs/loading-snomed.md

- Your can check the logs of your upload progress

```bash
docker logs -fn 10 snowstorm-server-write
```

- SNOMED CT browser will be on http://your-server-ip:9090

#### 5. Deploy read only Snowstorm terminology server

```bash
cat snowstorm-docker-service
# copy, paste and run the content
```

- Be sure to update your docker service snowstorm-server to connect to your application network so your application can interact with snowstorm-server

```bash
docker service update --network-add <your-application-network> snowstorm-server
```

#### 6. Remove read & write Snowstorm terminology server & SNOMED CT browser

```bash
docker compose -f snowstorm-write-browser/docker-compose.yml down -v
```

- -v flag here is to remove anonymous volume that snowstorm-server-write create

- If you ever need to do maintenance eg: upload new RF2 specification, deploy again read & write Snowstorm terminology server & SNOMED CT browser following step no 4

HAPPY HACKING !
