# Pythia
Test environment to support hygtfrde's delphi extractor project.

link to delphi: ``https://github.com/hygtfrde/delphi``


## How to install the environment.

### Step 1. setup the Environment.
Using docker.

- Create a docker network:
```bash
docker network create pythiaNetwork
```

- Pull redis image: 
```bash
docker pull redis
```

```bash
docker run --name pythiaOnRedis --network pythiaNetwork -d redis
```

### Step 2. Build the redis client.


From the ``network_components/http_redis_client`` dir.
- Build the image:
```bash
docker build -t img_llvm_redispp_crow .
```

- Then run the container:
```bash
docker run -it --network pythiaNetwork -v .:/workspace --name cont_llvm_redispp_crow img_llvm_redispp_crow /bin/bash
```


### Step 3. Build the nginx load balancer.

From the ``network_components/nginx_load_balancer`` dir:

- Build the nginx image:
```bash
docker build -t my-nginx-load-balancer .
```

- Run the nginx container:
```bash
docker run --name nginx-lb --network pythiaNetwork -p 80:80 my-nginx-load-balancer
```




From the ``network_components/http_redis_client/build`` dir.

- Run Cmake to generate makefile:
``` bash
cmake ..
```

- Run make to compile:
```bash
make
```

- Run the redis client:
```bash
./pythia
```

- Remove current build:
```bash
make clean
```

From the project root dir.
- Remove ``build`` dir before generating a new makefile:
```bash
rm -r build
mkdir build
```
### Architecture
```mermaid
flowchart TD
    UserReq["User Request<br>Get Stats"]
    lb["Load Balancer"]
    subgraph HTTP_Servers
        httpServer1["HTTP Server 1<br><br>Redis Client"]
        httpServer2["HTTP Server 2<br><br>Redis Client"]
        httpServer3["HTTP Server 3<br><br>Redis Client"]
        httpServer4["...<br><br>..."]
    end
    redisServer["Redis Server"]
    gtest["Redis Client<br>Google Test"]
    delphi["Delphi<br>Extractor"]

    UserReq --- lb
    delphi --- lb
    lb --- httpServer1
    lb --- httpServer2
    lb --- httpServer3
    lb --- httpServer4
    httpServer1 --- redisServer
    httpServer2 --- redisServer
    httpServer3 --- redisServer
    httpServer4 --- redisServer
    redisServer --- gtest
```
 
### https/server crow documentation.

https://crowcpp.org/master/guides/routes/

We are using nginx so the connection needs to be non SSL


#### Hot to use with with curl and crow  

let's make some post and get request to our server via the load balancer !

(Don't forget to use the appropriate port)

If you have issues, try to add ``-X POST `` or ``-X GET`` in front of ``curl``, but it should ne be necessary. 

| Category | Redis    | Arguments       | Usage           |
| -------- | -------- | --------------- | --------------- |
| POST     | set      | set             | curl http://localhost:80/set \ <br> -H "Content-Type: application/json" \ <br> -d '{"key":"mykey", "value":"myvalue"}'|
| POST     | lpush    | lpush           | curl http://localhost:80/lpush \ <br> -H "Content-Type: application/json" \ <br> -d '{"key":"mykey", "value":"myvalue"}'|
| POST     | rpush    | rpush           | curl http://localhost:80/rpush \ <br> -H "Content-Type: application/json" \ <br> -d '{"key":"mykey", "value":"myvalue"}'|
| POST     | hmset    | hmset           | curl http://localhost:80/rpush \ <br> -H "Content-Type: application/json" \ <br> -d '{"key":"myhash", "fields_values":{"field1":"value1", "field2":"value2"}}'|
| POST     | hmget    | hmget           | curl http://localhost:80/hmget \ <br> -H "Content-Type: application/json" \ <br> -d '{"key":"myhash", "fields":["field1", "field2"]}'|
| GET      | key      | key/<_string_>  | curl http://localhost:80/key/key_value|
| GET      | get      | key/<_string_>  | curl http://localhost:80/get/key_value|
| GET      | lpop     | lpop/<_string_> | curl http://localhost:80/lpop/key_value|
| GET      | rpop     | rpop/<_string_> | curl http://localhost:80/rpop/key_value|
| GET      | llen     | llen/<_string_> | curl http://localhost:80/llen/key_value|
| GET      | ping     | ping            | curl http://localhost:80/ping|
| GET      | echo     | echo/<_string_> | curl http://localhost:80/echo/msg|
| GET      | flushall | flushall        | curl http://localhost:80/flushall|
| GET      | info     | info            | curl http://localhost:80/info|


post request :

```sh
curl http://localhost:80/post \
    -H "Content-Type: application/json" \
    -d '{"key":"mykey", "value":"myvalue"}'
```

get request:
```sh
curl http://localhost:80/get/mykey
```


### redis-plus-plus documentation

https://github.com/sewenew/redis-plus-plus/blob/master/src/sw/redis%2B%2B/redis.h

### googleTest documentation
https://google.github.io/googletest/
https://github.com/google/googletest/tree/main/googletest/samples
