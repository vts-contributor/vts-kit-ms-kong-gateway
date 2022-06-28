[![][kong-logo]][kong-url]


**Kong** hay **Kong API Gateway** cung cấp chức năng proxy, routing, cân bằng tải, kiểm tra tình trạng của các Service, chứng thực, Kong đóng vai trò là lớp trung gian tổ chức các microservice hay traffic của API.


 ## Cài Đặt

 Chúng tôi khuyên bạn nên dùng docker-compose để cài đặt theo hướng dẫn dưới đây nhưng nếu bạn muốn chạy Kong API Gateway ở chế độ không có DB, bạn có thể đọc thêm ở [đây](https://docs.konghq.com/install/docker/). Khi nhận được folder cài đặt này, hãy xác nhận rằng folder có đủ những file sau đây:
 ```cmd
 kong.tar
 postgres.tar
 docker-compose.yml
 kong-deck.tar
 ```
 
 1) Để cài đặt Kong, load Kong và Postgre images và chạy file docker-compose.yml kèm theo trong folder cài đặt   
 ```cmd
  $ docker load -i kong.tar
  $ docker load -i postgres.tar
  $ docker load -i kong-deck.tar
  $ docker-compose up -d
```

Trong trường hợp bạn không tim thấy file docker-compose.yml, tạo một file mới, đặt tên là docker-compose.yml và dán nội dung sau vào đó:
```cmd
version: "3.9"
networks:
 kong-net:
  driver: bridge
services:
  kong-database:
    image: postgres:9.6
    restart: always
    networks:
      - kong-net
    environment:
      POSTGRES_USER: kong
      POSTGRES_DB: kong
      POSTGRES_PASSWORD: kong 
    ports:
      - "5432:5432"

  kong-migration:
    image: kong:latest
    command: "kong migrations bootstrap"
    networks:
      - kong-net
    restart: on-failure
    environment:
      KONG_PG_HOST: kong-database
      KONG_DATABASE: postgres
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kong
      KONG_CASSANDRA_CONTACT_POINTS: kong-database
    links:
      - kong-database
    depends_on:
      - kong-database

  kong:
    image: kong:latest
    restart: always
    networks:
      - kong-net
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kong
      KONG_CASSANDRA_CONTACT_POINTS: kong-database
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001, 0.0.0.0:8444 ssl
    depends_on:
      - kong-migration
      - kong-database
    ports:
      - "0.0.0.0:8001:8001"
      - "0.0.0.0:8000:8000"  
      - "0.0.0.0:8444:8444"
      - "0.0.0.0:8443:8443"

  kong-deck:
    image: kong/deck:latest
    networks:
      - kong-net
    depends_on:
      - kong      
```
Sau đó mở terminal ở folder cài đặt và chạy lệnh
```cmd
$ docker load -i kong.tar
$ docker load -i postgres.tar
$ docker load -i kong-deck.tar
$ docker-compose up -d
```

Trong trường hợp thiếu một trong 2 file kong.tar hoặc postgres.tar, máy bạn cần có kết nối internet. Sau đó mở terminal ở folder cài đặt và chạy lệnh

```cmd
$  docker-compose up -d
```
## PORT
Kong Gateway sẽ sử dụng những port sau đây trên localhost:\
`:5432` PostgreSQL sử dụng để lưu lại tất cả các yêu cầu được xử lý bởi Kong.\
`:8000` Kong sử dụng để tiếp nhận mọi HTTP traffic từ khách hàng và chuyển tiếp những traffic đó cho những Service phía trên.\
`:8001` Admin API dùng để thiết lập Kong HTTP\
`:8443` Kong sử dụng để tiếp nhận mọi HTTPS traffic từ khách hàng và chuyển tiếp những traffic đó cho những Service phía trên.\
`:8444` Admin API dùng để thiết lập Kong HTTPS

Nếu bạn muốn thay đổi những cổng mặc định trên, mở file docker-compose.yml và thay đổi các giá trị ở dòng 16 và 55 tới 58.
```cmd
    - "5432:5432"
    - "0.0.0.0:8001:8001"
    - "0.0.0.0:8000:8000"  
    - "0.0.0.0:8444:8444"
    - "0.0.0.0:8443:8443"
```

Thay đổi những giá trị trên thành port mà bạn mong muốn. Ví dụ bạn muốn Kong hoạt động trên cổng 3000 và Kong Admin API hoạt động trên cổng 3001 thì giá trị trên ở file docker-compose sẽ trông như  sau.

```cmd
    - "5432:5432"
    - "0.0.0.0:3001:8001"
    - "0.0.0.0:3000:8000"  
    - "0.0.0.0:8444:8444"
    - "0.0.0.0:8443:8443"
```
## Thiết lập Service
### Thêm Service mới sử dụng Admin API
Gửi lệnh POST sau tới Admin API của Kong. Lệnh này sẽ yêu cầu Kong tạo một Service mới tên là example-service và chấp nhận yêu cầu ở địa chỉ url http://mockbin.org.

```cmd
curl -i -X POST \
  --url http://localhost:8001/services/ \
  --data 'name=example-service' \
  --data 'url=http://mockbin.org'
```

Để đăng kí Service của bạn :
1. Thay port của localhost thành port của Admin API mà bạn thiết lập ở bước trên
2. Thay example-service thành tên Service của bạn
3. Thay  http://mockbin.org thành URL của bạn.

Server sẽ trả về một respond dưới dạng
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

### Thêm Route cho Service
Gửi lệnh POST sau tới Admin API của Kong. Lệnh này sẽ hướng dẫn Kong proxy mọi request mà header có chứa cặp key value hosts:examples.com tới Service example-service
```cmd
curl -i -X POST \
  --url http://localhost:8001/services/example-service/routes \
  --data 'hosts[]=example.com'
```
Sau khi chạy dòng lệnh này, mọi request đến địa chỉ
```cmd
http://localhost:8000
```
với cặp key-value hosts:example.com ở header sẽ được điều hướng tới tới API
```cmd
http://mockbin.org
``` 

Để đăng kí Route cho Service của bạn :
1. Thay port của localhost thành port của Admin API mà bạn thiết lập ở bước trên.
2. Thay example-service thành tên Service của bạn.
3. Thay hosts[]=example.com thành cặp key-value ở header mà bạn mong muốn.

Nếu bạn muốn proxy những request tới một path cụ thể, sử dụng dòng lệnh sau
```cmd
curl -i -X POST \
  --url http://localhost:8001/services/example-service/routes \
  --data 'paths[]=/yourpath'
```
  Sau khi chạy dòng lệnh này, mọi request đến địa chỉ
```cmd
http://localhost:8000\yourpath
```
sẽ được điều hướng tới tới API
```cmd
http://mockbin.org
``` 
1. Thay port của localhost thành port của Admin API mà bạn thiết lập ở bước trên.
2. Thay example-service thành tên Service của bạn.
3. Thay /yourpath thành đường dẫn từ URL mà bạn mong muốn. Lưu ý rằng đường dẫn phải bắt đầu bằng `/`.

### Xóa bỏ một Service đã tồn tại 
Để xóa bỏ một Service đã tồn tại trên Kong API, chạy dòng lệnh sau đây 
```cmd
curl -X DELETE http://localhost:8001/services/serviceFoo
```
thay thế serviceFoo với tên Service mà bạn muốn xóa bỏ. 

### Xóa bỏ một Route trong Service 
Để xóa bỏ một Route trong Service đã tạo trên Kong API, trước hết bạn cần biết ID của Route mình muốn xóa. Để hiển thị Route của tất cả các Service, sử dụng dòng lệnh 
```cmd
curl -X GET "http://localhost:8001/routes"
```
Nếu bạn biết Route mình muốn xóa nằm trong Service nào, sử dụng dòng lệnh 
```cmd
curl -X GET "http://localhost:8001/services/foo-service/routes"
```
và thay thế foo-service bằng tên Service mà bạn mong muốn. Sau khi chạy dòng lệnh trên, 
Server sẽ trả về một respond tương tự như sau :
```cmd
 "next":null,
 "data":[
          {
            "id":"a714d55e-a168-4efc-ae1a-6c380b0d5c84",
            "path_handling":"v0",
            "paths" => [
                         "/example/endpoint"
                       ],
            "destinations":null,
            "headers":null,
            "protocols":["http","https"],
            "methods":null,
            "snis":null,
            "service":{
                        "id":"fbb57bfd-124b-451c-841c-dbfea2b9ee8b"
                      },
            "name":null,
            "strip_path":true,
            "preserve_host":false,
            "regex_priority":0,
            "updated_at":1584417746,
            "sources":null,
            "hosts":[
                      "example.com"
                    ],
            "https_redirect_status_code":426,
            "tags":null,
            "created_at":1584417746
          }
        ]
}
```
Copy lại ID của Route mà bạn muốn xóa, trong trường hợp này chính là `a714d55e-a168-4efc-ae1a-6c380b0d5c84`. 
Sau đó thực thi chạy dòng lệnh 
```cmd
curl -X DELETE "http://localhost:8001/services/foo-service/routes/a714d55e-a168-4efc-ae1a-6c380b0d5c84"
``` 
1. Thay port của localhost thành port của Admin API mà bạn thiết lập ở bước trên.
2. Thay example-service thành tên Service của bạn.
3. Thay route ID ở cuối thành route ID mà bạn tìm thấy ở bước trước.

## Config Service sử dụng Deck
Bạn cũng có thể sử dụng Deck, một library của Kong để quản lý tất cả các Service trên Kong API. Để sử dụng Deck, bạn cần file `kong.yaml` (đã được bao gồm trong folder cài đặt). Trong trường hợp file `kong.yaml` không tồn tại, tạo một file mới và đặt tên là `kong.yaml` và thêm vào đó 
```cmd
_format_version: "1.1"
services:
```
### Config Service và Route 
Để thêm Service và Routes, thêm vào file yaml dưới dạng: 
```cmd
- name: svc1
  host: mockbin.org
  routes:
  - name: r1
    paths:
    - /r1
  - name: test-header 
    headers: 
      forward-this-request: ["this data"]
``` 
Thay thế giá trị của name và host thành giá trị mà bạn mong muốn. Lặp lại tương tự với routes. Nếu bạn muốn routing sử dụng path cụ thể, sử dụng biến paths tương tự như route name r1 ở ví dụ trên. Nếu bạn muốn routing sử dụng header, làm tương tự ví dụ test-header. \
Thêm tương tự các Service và Route khác. Sau khi đã thêm hết các Route và Service, từ folder chứa file `kong.yaml`, dùng lệnh sau để cập nhật các Service:
```cmd
$ deck sync
```
Tương tự, sau mỗi lần cập nhật file `kong.yaml` cần sử dụng lệnh trên để lưu các thay đổi.


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