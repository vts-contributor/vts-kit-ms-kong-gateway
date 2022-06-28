[![][kong-logo]][kong-url]




**Kong** or **Kong API Gateway**   provided functionality for proxying, routing, load balancing, health checking, authentication, Kong serves as the central layer for orchestrating microservices or conventional API traffic with ease.

Kong runs natively on Kubernetes thanks to its official [Kubernetes Ingress Controller](https://github.com/Kong/kubernetes-ingress-controller).

---
---

## Getting Started


We suggest using the docker-compose distribution via the instructions below, but there is also a [docker installation](https://docs.konghq.com/install/docker/) procedure if you’d prefer to run the Kong API Gateway in DB-less mode. 

Whether you’re running in the cloud, on bare metal, or using containers, you can find every supported distribution on our [official installation](https://konghq.com/install/#kong-community) page.

1) To start, load kong and Postgre images and  run the docker-compose.yml file included inside the installation folder.
```cmd
  $ docker load -i kong.tar
  $ docker load -i postgres.tar
  $ docker-compose up -d
```

## PORT

The Gateway will be available on the following ports on localhost:\
`:5432` on which PostgreSQL used to logs all traffic listened by Kong.\
`:8000` on which Kong listens for incoming HTTP traffic from your clients, and forwards it to your upstream services.\
`:8001` on which the Admin API used to configure Kong HTTP listens.\
`:8443` on which Kong listens for incoming HTTPS traffic from your clients, and forwards it to your upstream services.\
`:8444` on which the Admin API used to configure Kong HTTPS listens.\

In order to change  default ports, open up the docker-compose.yml file and change lines 16 and 55 through 58 
```cmd
    - "5432:5432"
    - "0.0.0.0:8001:8001"
    - "0.0.0.0:8000:8000"  
    - "0.0.0.0:8444:8444"
    - "0.0.0.0:8443:8443"
```
Change these value to your desired ports. For example, if you want to change Kong to ports 3000 and Kong Admin API to port 3001, you would change the these above line as follow: 
```cmd
    - "5432:5432"
    - "0.0.0.0:3001:8001"
    - "0.0.0.0:3000:8000"  
    - "0.0.0.0:8444:8444"
    - "0.0.0.0:8443:8443"
```

## Configuring a Service
### Add a Service using the Admin API
Issue the following POST request to add your Service to Kong. This instructs Kong to create a new Service named example-service which will accept traffic at http://mockbin.org. 

```cmd
curl -i -X POST \
  --url http://localhost:8001/services/ \
  --data 'name=example-service' \
  --data 'url=http://mockbin.org'
```
Change localhost port to your Admin API port in previous step or ignore if you use default value. Replace example-service with your service name and  http://mockbin.org with your URL.\
You should receive a response similar to:
```cmd
HTTP/1.1 201 Created
Content-Type: application/json
Connection: keep-alive

{
   "host":"mockbin.org",
   "created_at":1519130509,
   "connect_timeout":60000,
   "id":"92956672-f5ea-4e9a-b096-667bf55bc40c",
   "protocol":"http",
   "name":"example-service",
   "read_timeout":60000,
   "port":80,
   "path":null,
   "updated_at":1519130509,
   "retries":5,
   "write_timeout":60000
}
```
### Add a Route for the Service
Issue the following POST request to add a Route to the previously created service. The following commands instructing Kong to proxy requests with a Host header that contains example.com to the example-service.
```cmd
curl -i -X POST \
  --url http://localhost:8001/services/example-service/routes \
  --data 'hosts[]=example.com'
```
Change localhost port to your Admin API port in previous step or ignore if you use default value.\
Replace example-service with your service name.\
Replace  hosts[]=example.com with your header key-value pair. 

If you want to proxy requests with a certain path, issue the following post.
```cmd
curl -i -X POST \
  --url http://localhost:8001/services/example-service/routes \
  --data 'paths[]=/yourpath'
```
Change localhost port to your Admin API port in previous step or ignore if you use default value.\
Replace example-service with your service name.\
Replace /yourpath with your path. Note that the value must start with `/` 









## License

```
Copyright 2016-2022 Kong Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

[kong-url]: https://konghq.com/
[kong-logo]: https://konghq.com/wp-content/uploads/2018/05/kong-logo-github-readme.png
[kong-benefits]: https://konghq.com/wp-content/uploads/2018/05/kong-benefits-github-readme.png
[kong-master-builds]: https://hub.docker.com/r/kong/kong/tags
[badge-action-url]: https://github.com/Kong/kong/actions
[badge-action-image]: https://github.com/Kong/kong/workflows/Build%20&%20Test/badge.svg

[busted]: https://github.com/Olivine-Labs/busted
[luacheck]: https://github.com/mpeterv/luacheck