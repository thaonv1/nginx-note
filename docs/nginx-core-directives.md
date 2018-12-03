# Nginx Core Directives

## Mục lục

1. File cấu hình của Nginx

2. Directives là gì?

3.

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

<img src="">

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

Cấu hình mặc định của Nginx như sau:

<img src="">

**Simple Directives**

Có thể gọi cả đoạn trên là main context. Có một vài simple directive trong main block:

- user directive: Có giá trị mặc định là nobody. Bạn có thể define để chỉ định account nào có quyền chạy nginx worker process. Lưu ý user này phải có trên hệ thống trước đó.
- worker_process: Giá trị mặc định là 1, chỉ định số worker_process sẽ được spawn ra cho Nginx. Nếu là `auto`, nginx sẽ lấy bằng số cores.
- error_log: Có thể dử dụng trong nhiều context khác nhau. Trong ví dụ trên, nó được đặt ở main context. parameter thứ nhất là `/var/log/nginx/error.log` chỉ định path của file log. Trong khi đó parameter thứ 2 chỉ định rằng tất cả các log lớn hơn warning level đều sẽ được ghi lại.
- pid: chỉ định file name sẽ lưu trữ process id của master process là `/var/run/nginx.pid`.

**Events Context**

Sau các cấu hình mặc định là simple directive, bạn sẽ thấy `events context`. Context này chỉ được định ở trong main context và ở trong 1 file cấu hình nginx bạn chỉ được phép định nghĩa 1 event context

- worker_connections: cho phép tối đa 1024 worker_connections, đây cũng là con số mặc định. Chúng ta có thể tính concurrent connections bạn có thể có trên webserver theo công thức sau:

(worker_processes x worker_connections x N) / Average Request Time

trong đó N là số lượng connections trung bình trong mỗi rq.

- multi_accept: mặc định là off, có nghĩa rằng worker proccess sẽ chỉ accept 1 connection mới tại một thời điểm.
- accept_mutex: mặc định là on, nó có nghĩa rằng worker process sẽ get request 1-1.
-  accept_mutex_delay:  chỉ có tác dụng khi mà accept_mutex được bật. Nó sẽ chỉ định khoảng time tối đa mà worker process có thể đợi để chấp nhận 1 connection mới trước khi tạo ra proccess mới để thực thi request.

**HTTP Context**

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
