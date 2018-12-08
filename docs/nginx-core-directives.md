# Nginx Core Directives

## Mục lục

1. File cấu hình của Nginx

2. Directives là gì?

3. Context Types

----------------

### 1. File cấu hình của Nginx

File cấu hình của Nginx có tên là `nginx.conf`. Tùy vào cách bạn cấu hình mà nó sẽ nằm ở các thư mục khác nhau. Để xem đường dẫn file cấu hình hiện tại, sử dụng câu lệnh sau:

`ps -ax | grep nginx`

### 2. Directives là gì?

Directive được định nghĩa như là `instruction` hoặc để `direct`. Nôm na thì Directives sẽ định nghĩa cách mà nginx chạy trên server của bạn. Directives có 2 loại: simple directives
and block directives.

- Simple directive: directive đơn giản, các parameters cách nhau bởi dấu cách và kết thúc bởi dấu chấm phẩy.
ví dụ:

`worker_processes 1;`

- Block directive : Bao gồm các block được bọc bởi ngoặc nhọn ({}) và bao hàm bên trong các simple directive

Hình sau mô tả cấu trúc của file cấu hình nginx

<img src="https://i.imgur.com/RyHVCkf.png">

Các bạn có thể thấy context ngoài cùng được gọi là main context và chứa simple directive kèm theo các block directive.

### 3. Context Types

Tất cả các module trong nginx đều có những mục đích riêng và được control bởi các directive. Có khá nhiều context trong Nginx như: main, events, HTTP, server,
location, upstream, if, stream, mail, etc. Trong đó HTTP, events, server, and location là được dùng nhiều nhất. Dưới đây là cấu trúc cơ bản của file cấu hình:

```
# main block is not explicitly called as main, it is implied
main {
  simple_directives parameters;
  ...
  events{
      event_directives parameters;
      ...
  }
  http{
      http_directives parameters;
      ...
    server{
        server_directives parameters;
        ...
        location{
          location_directives parameters;
          ...
        }
    }
  }
}
```

### 4. Tìm hiểu cấu hình mặc định

Cấu hình mặc định của Nginx như sau:

<img src="https://i.imgur.com/RFdgwhW.png">

#### 4.1 Simple Directives

Có thể gọi cả đoạn trên là main context. Có một vài simple directive trong main block:

- user directive: Có giá trị mặc định là nobody. Bạn có thể define để chỉ định account nào có quyền chạy nginx worker process. Lưu ý user này phải có trên hệ thống trước đó.
- worker_process: Giá trị mặc định là 1, chỉ định số worker_process sẽ được spawn ra cho Nginx. Nếu là `auto`, nginx sẽ lấy bằng số cores.
- error_log: Có thể dử dụng trong nhiều context khác nhau. Trong ví dụ trên, nó được đặt ở main context. parameter thứ nhất là `/var/log/nginx/error.log` chỉ định path của file log. Trong khi đó parameter thứ 2 chỉ định rằng tất cả các log lớn hơn warning level đều sẽ được ghi lại.
- pid: chỉ định file name sẽ lưu trữ process id của master process là `/var/run/nginx.pid`.

#### 4.2 Events Context

Sau các cấu hình mặc định là simple directive, bạn sẽ thấy `events context`. Context này chỉ được định ở trong main context và ở trong 1 file cấu hình nginx bạn chỉ được phép định nghĩa 1 event context

- worker_connections: cho phép tối đa 1024 worker_connections, đây cũng là con số mặc định. Chúng ta có thể tính concurrent connections bạn có thể có trên webserver theo công thức sau:

(worker_processes x worker_connections x N) / Average Request Time

trong đó N là số lượng connections trung bình trong mỗi rq.

- multi_accept: mặc định là off, có nghĩa rằng worker proccess sẽ chỉ accept 1 connection mới tại một thời điểm.
- accept_mutex: mặc định là on, nó có nghĩa rằng worker process sẽ get request 1-1.
-  accept_mutex_delay:  chỉ có tác dụng khi mà accept_mutex được bật. Nó sẽ chỉ định khoảng time tối đa mà worker process có thể đợi để chấp nhận 1 connection mới trước khi tạo ra proccess mới để thực thi request.

#### 4.3 HTTP Context

Mặc định, nó sẽ có 1 số directive sau:

- include /etc/nginx/mine.types: Nó maps file name extension với MIME types (mô tả loại media content và hướng dẫn trình duyệt để nó có thể render chuẩn xác mà ko cần download nó về) của các respond trả về.

```
types {
text/html   html htm shtml;
text/css    css;
text/xml    xml;
image/gif   gif;
image/jpeg  jpeg jpg;
application/javascript js;
...
...
audio/midi  mid midi kar;
audio/mpeg  mp3;
...
...
video/x-flv flv;
video/x-m4v m4v;
}
```

- default_type: nó chỉ định MIME type mặc định nếu như Nginx ko tìm thấy cái nào trong `/etc/nginx/mine.types`.
- log_format: config module `ngx_http_log_module`. Nó sẽ ghi log theo forrmat chỉ định.
- access_log: yêu cầu 1 đường dẫn `/var/log/nginx/access.log` và tên của format ( khai báo ở trên)
- keepalive_timeout: mặc định sẽ có giá trị là 65, khi mà connection được thiết lập, bạn sẽ ko cần phải disconnect bằng tay.
- gzip: mặc định là off, khi được bật, nó sẽ nén dữ liệu giúp giảm tải cho mỗi rq.
- include: nó sẽ load các config được khai báo ở đường dẫn phía sau từ include.

#### 4.4 The conf.d Folder

Folder này có 2 file, `default.conf` và `example_ssl.conf`. Mặc định thì `example_ssl.conf` sẽ được comment toàn bộ trừ khi bạn muốn sử dụng ssl.

Cấu hình mặc định của file `default.conf`:

```
server {
  listen 80;
  server_name localhost;
  #charset koi8-r;
  #access_log /var/log/nginx/log/host.access.log main;
  location / {
    root /etc/nginx/html;
    index index.html index.htm;
}
#error_page 404 /404.html;
# redirect server error pages to the static page /50x.html
#
error_page 500 502 503 504 /50x.html;
location = /50x.html {
    root /usr/share/nginx/html;
}
# proxy the PHP scripts to Apache listening on 127.0.0.1:80
#
#location ~ \.php$ {
#   proxy_pass http://127.0.0.1;
#}
# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
#
#location ~ \.php$ {
#   root html;
#   fastcgi_pass 127.0.0.1:9000;
#   fastcgi_index index.php;
#   fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name;
#   include fastcgi_params;
#}
# deny access to .htaccess files, if Apache's document root
# concurs with nginx's one
#
#location ~ /\.ht {
#   deny all;
#}
}
```

#### 4.5. Server Context

Nginx cho phép lựa chọn server cụ thể dựa vào request.

Đối với `static content`

File `/etc/nginx/nginx.conf` có thể để mặc định, chúng ta sẽ backup file default.conf và tạo ra 1 file mới trong cùng thư mục, mục tiêu sẽ là host được 2 app có domain như sau:

```
http://app1.com or http://www.app1.com
http://app2.com or http://www.app2.com
```

```
server {
  listen 80;
  server_name app1.com www.app1.com;
  location / {
    root /etc/nginx/html/app1;
  }
}
server {
  listen 80;
  server_name app2.com www.app2.com;
  location / {
    root /etc/nginx/html/app2;
  }
}
```

Lưu ý:

- có 2 server block và cả 2 đều đang dùng port 80
- có 2 thư mục khác nhau đặt ở phía server và chúng có cùng thư mục root
- tạo ra thư mục `app1` và `app2` rồi tạo mẫu file html trong mỗi thư mục.
- chạy câu lệnh restart nginx hoặc reload lại (nginx -s reload)

##### Sẽ có những routing như sau:

- Nếu bạn truy cập `http://localhost` thì nginx sẽ trả về app1, tại vì nginx sẽ cố map với domain mà bạn khai báo (server_name). còn nếu bạn không đưa domain vào request thì nó sẽ map với block đầu tiên. Nếu bạn đổi port của app1 sang 81 và thực hiện lại request thì nginx sẽ trả về app2.

- Bạn có thể chỉ định đâu là server mặc định bằng cách thêm tùy chọn `default_server` vào sau cấu hình (listen 80 default_server;)

- Nếu bạn ko muốn bất cứ block nào làm default block, thêm đoạn sau vào đầu file

```
server {
  listen 80;
  server_name "" localhost 127.0.0.1;
  return 444;
}
```

Lúc này khi bạn truy cập localhost hoặc 127.0.0.1 thì sẽ nhận được output sau : `Empty reply from server`

- Nếu nginx listen nhiều ip, thì nó sẽ xem trước ip và port, sau đó là header field. Vì thế nếu bạn có cấu hình sau và requesst tới địa chỉ app3.com trên node 10.0.0.1 thì bạn sẽ nhận được output của app1 vì nó ko thể tìm thấy app3.com trên 10.0.0.1 và vì thế mặc định nó lấy block đầu tiên

```
server{
  listen 10.0.0.1:80;
  server_name app1.com www.app1.com;
}
server{
  listen 10.0.0.1:80;
  server_name app2.com www.app2.com;
}
server{
  listen 10.0.0.2:80;
  server_name app3.com www.app3.com;
}
```

- Lưu ý: Với 1 ip và port bạn chỉ có thể có 1 default_server

Hình sau mô tả quá trình routing

<img src="https://i.imgur.com/VEIZQ66.png">

#### 4.6. Location Context

Cần lưu ý về `index`, đây sẽ là file mặc định gửi tới client nếu ko có cái gì được define. Ví dụ bạn add `index index.html index.htm` thì nginx sẽ hiểu là phải trả lại `index.html hoặc index.htm` nếu url chỉ là `http://app.com`

Thử thực hành chút nhé, đầu tiên là chuẩn bị server. Truy cập vào thư mục `/etc/nginx/html`, tạo ra folder `common` và trong đó có cấu trúc như sau:

```
html
|-- 50x.html
|-- app1
| `-- index.html
|-- app2
| |-- home.html
| `-- index.html
|-- common
| |-- app.js
| |-- index.html
| |-- nginx.png
| `-- nginx.PNG
`-- index.html
```

Thay đổi cấu hình của file `/etc/nginx/conf.d/virtual_server.conf` mà ta đã tạo mẫu ở phía trên.

```
server {
  listen 80;
  server_name 127.0.0.1 localhost;
  location /app1/ {
    root /etc/nginx/html;
    index index.html;
  }
  location /app2/ {
    root /etc/nginx/html;
    index home.html;
  }
}
```

Reload lại nginx `nginx -s reload`.

Nếu bạn truy cập: `http://localhost/app1` thì sẽ thấy file `index.html`
Nếu truy cập: `http://localhost/app2` thì sẽ thấy file `home.html`

Hoặc bạn cũng có thể khai báo root ở bên ngoài như sau:

```
server {
  listen 80;
  server_name 127.0.0.1 localhost;
  root /etc/nginx/html;
  location /app1/ {
    index index.html;
  }
  location /app2/ {
    index home.html;
  }
}
```

#### 4.7. Location Context Special case

`try_files`

Nó sẽ check xem file đã tồn tại trong thư mục hay chưa. Hãy xem ví dụ sau:


```
server {
  listen 80;
  server_name 127.0.0.1 localhost;
  root /etc/nginx/html;

  location ~* \.(jpg|jpeg)$ {
      root /etc/nginx/html/common;
  }
  location ~* \.(png)$ {
      root /etc/nginx/html/common;
      try_files $uri $uri/ /nginx.png;
  }
}
```

Cả hai block trông khá giống nhau, cùng apply một rule đối với các định dạng jpeg và png. Tuy nhiên ở block thứ 2 có `try_files`. Nó sẽ tìm kiếm trong `$uri`, nếu ko có nó sẽ chuyển qua `$uri/`. Nếu vẫn không có, nó sẽ trả về `/nginx.png`

`rewrite`

Rewrite directive có thể redirect request của bạn. Nó hỗ trợ các flag parameter sau:

- `last` : Dừng process hiện tại và bắt đầu tìm kiếm location mới match với url đã được thay đổi
- `break` : Dừng tiến trình
- `redirect` redirect tạm thời sử dụng status code 302
- `permanent` redirect dài hạn sử dụng status code 301

Ở một số trường hợp cụ thể ví dụ như việc thay đổi ở phía backend hoặc thay đổi framework làm cho `rewrite` hữu ích hơn. Ví dụ như cấu trúc thư mục của bạn đã thay đổi và `common` bây giờ được chuyển thành `vendor_assets`

```
server {
      listen 80;
      server_name 127.0.0.1 localhost;
      root /etc/nginx/html;

      location /common/ {
          rewrite ^(/common/)(.*) /vendor_assets/$2 last;
      }
}
```

- `error_page`

`error_page` giúp nginx trả lại một error page sử dụng internal redirect.

```
error_page 404 /404_not_found.html;                 #Applies to status code 404.
error_page 500 502 503 504 /50x_server_error.html;  #Applies to status code 500, 502, 503 & 504.
error_page 404 =200 /funnypic.png;                  #Instead of sending 404, it will send a png with status code 200.
```

#### 4.8. Kiểm tra cấu hình

Option `-t` và `-c` được sử dụng để check cấu hình của nginx

`nginx -t -c /etc/nginx/nginx.conf`

#### 4.9. Allow Directory Listing

Việc Listing Directory được disable mặc định, để sử dụng, bạn phải có directive `autoindex`:

```
server {
    listen 80;
    server_name 127.0.0.1 localhost;
    root /etc/nginx/html;

    location /common/ {
        root /etc/nginx/html;
        index NON_EXISTENT_FILE;
        autoindex on;
    }
}
```

#### 4.10. Deny Access to Any Specific Location

```
server {
      listen 80;
      server_name 127.0.0.1 localhost;
      root /etc/nginx/html;
      location /vendor_assets/ {
          deny all;
      }
}
```

Truy cập `http://localhost/vendor_assets/` -> bạn sẽ nhận được 403

### 5. Nginx Variables

Nginx có một số biến rất tiện dụng, tuy nhiên ko nên sử dụng nó một cách bừa bãi. Lý do là bởi nó sẽ được cpu evaluated real time nên có thể sẽ gây ảnh hưởng tới máy chủ nếu dùng quá nhiều.

Ví dụ, chỉnh sửa file `nginx.conf`:

`log_format main '$remote_addr - $remote_user [$time_local] "$request" '' $status $body_bytes_sent "$http_referer"';`

Tạo thử một request, truy cập vào file log `/var/log/nginx/access.log`


`127.0.0.1 - - [14/Sep/2015:06:18:22 -0400] "GET / HTTP/1.1" 200 23 "-"`

Bạn có thể lấy thêm một số thông tin

`log_format main '$remote_addr - $remote_user [$time_local] "$document_root$document_uri"'
'"$request" $status $body_bytes_sent "$http_referer"';`

Và đây là log bạn nhận được

`127.0.0.1 - - [14/Sep/2015:06:25:48 -0400] "/home/nginx_new_home/index.html""GET / HTTP/1.1"
200 23 "-"`
