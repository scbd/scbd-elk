
# Build Docker images

* Elastic search
    ```
        docker image build -f ./elasticsearch/Dockerfile . -t scbd/elastic-search
    ```
* Kibana
    ```
        docker image build -f ./kibana/Dockerfile . -t scbd/kibana
    ```
* Logstash
    ```
        docker image build -f ./logstash/Dockerfile . -t scbd/logstash
    ```


# Make sure below directories exists on server and have write access

```
    ssh ubuntu@logs.infra.cbd.int " mkdir ~/elasticsearch;
                                mkdir ~/elasticsearch/data;
                                mkdir ~/kibana;
                                mkdir ~/kibana/plugins;
                                mkdir ~/kibana/optimize;
                                mkdir ~/kibana/plugins/logtrail;
                                mkdir ~/logstash;
                                sudo chown 1000:1000 ./elasticsearch/data/;
                                sudo chown 1000:1000 ./kibana/optimize/;"
```

# Copy conf files to server if required

```
    scp -r ./elasticsearch/config/  ubuntu@logs.infra.cbd.int:~/elasticsearch
    scp ./elasticsearch/logstash-template.json ubuntu@logs.infra.cbd.int:~/elasticsearch/
    
    scp -r ./kibana/config/         ubuntu@logs.infra.cbd.int:~/kibana
    scp ./kibana/plugins/logtrail/config.json ubuntu@logs.infra.cbd.int:~/kibana/plugins/logtrail

    scp -r ./logstash/config/       ubuntu@logs.infra.cbd.int:~/logstash
    scp -r ./logstash/pipeline/     ubuntu@logs.infra.cbd.int:~/logstash

    scp    ./.env                   ubuntu@logs.infra.cbd.int:
    scp -r ./services               ubuntu@logs.infra.cbd.int:
```
# Network
```
    sudo docker network create --attachable --driver overlay elk 
```

# Start proxy and portainer stack
```
    sudo docker stack deploy --compose-file=./services/proxy-portainer.yml proxy-portainer
```

# Start Elasticsearch and Kibana stack
 * update elastic search user `elastic` password in .env file `elastic_password`
 * ```
    sudo docker stack deploy --compose-file=./services/elastic-search-kibana.yml elastic-kibana
   ```
* create logstash user with write permissions ref https://www.elastic.co/guide/en/logstash/master/ls-security.html#ls-http-auth-basic
    * Role name `logstash_writer`
    ```
        CURL -H "Content-Type:application/json" -u 'elastic:${xxxxxxxxxxx}' --request POST http://localhost:9200/_xpack/security/role/logstash_writer -d '
        {
            "cluster": ["manage_index_templates", "monitor", "manage_ilm"], 
            "indices": [
                {
                "names": [ "scbd-*" ], 
                "privileges": ["write","delete","create_index","manage","manage_ilm"]  
                }
            ]
        }'
    ```
    * User name `logstash_writer`
    ```
        CURL -H "Content-Type:application/json" -u 'elastic:${xxxxxxxxxxx}' --request POST  http://localhost:9200/_xpack/security/user/${logstash-user} -d '
        {
            "password" : "${xxxxxxxxxxx}",
            "roles" : [ "logstash_writer"],
            "full_name" : "Internal Logstash User"
        }'
    ```
* update all remaining evn variable in .env file

# Deploy logstash stack
```
    sudo docker stack deploy --compose-file=./services/logstash.yml logstash
```


<!-- sudo docker stack rm elastic-kibana 
sudo docker stack rm logstash
sudo docker stack rm proxy-portainer  

sudo docker stack deploy --compose-file=./services/proxy-portainer.yml proxy-portainer
sudo docker stack deploy --compose-file=./services/elastic-search-kibana.yml elastic-kibana
sudo docker stack deploy --compose-file=./services/logstash.yml logstash -->
