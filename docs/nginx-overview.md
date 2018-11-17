# Introduction to Nginx

## Mục lục

1. Nginx là gì và để làm gì?

2. Quay lại căn bản - HTTP Basic, Thế nào là web server?

3. Tại sao nên sử dụng Nginx?

4. Điểm khác biệt của Nginx và Apache?

--------------------------------------------------

### 1. Nginx là gì và để làm gì?

Khi mà bạn gõ một địa chỉ nào đó vào trình duyệt tức là bạn đã bắt đầu 1 request, request này rời khỏi network local của bạn, trải qua một vài location ngoài kia và reach tới địa chỉ cuối cùng, nó được gọi là web server.

Đó có thể coi là định nghĩa đơn giản về web server và giao thức http. Tuy nhiên, những gì mà chúng ta sẽ tìm hiểu qua series này là việc làm thế nào để những thứ mình nói bên trên giao tiếp với nhau.

Nginx (phát âm là engine-x) là đứa con đẻ của Igor Sysoev. Nó được bắt đầu vào những năm 2002 và chính thức release vào 2 năm sau đó. Nginx đang available trên `www.nginx.org` và theo những gì mà nginx team nói thì đây là "a free, open-source, high-performance HTTP server and reverse proxy, as well as an IMAP/POP3 proxy server". Trải qua từng ấy năm, Nginx được biết đến bởi hiệu năng cao, đi kèm với sự ổn định, nhiều tính năng, cấu hình đơn giản và tiêu tốn ít resource.

### 2. Quay lại căn bản - HTTP Basic, Thế nào là web server?

#### 2.1 HTTP cơ bản

Kể từ khi thời kì của internet bắt đầu trên thế giới này, HTTP (HyperText Transfer Protocol) đã đóng một vai trò rất quan trọng trong việc vận chuyển các nội dung số đến mọi nơi. Về cơ bản, bạn có thể coi HTTP giống như 1 list các rules về việc vận chuyển file. Khi mà bạn truy cập vào website, bạn sẽ thấy một số file được hiển thị trực tiếp trên trình duyệt, một số khác thì sẽ được download về.

Tại sao lại như vậy? Bởi đơn giản đó là cách mà web server và web client hoạt động. HTTP là giao thức, mà giao thức là cách thức để giao tiếp (theo CongTO). Ở đây chính là web client giao tiếp với webserver.

Để mình cho các bạn xem HTTP trông thế nào :D

- Mở Chrome lên

- Nhấn tổ hợp phím CTRL + Shift + I

- Chuyển qua tab network

- Truy cập tới `meditech.vn` và đợi page load xong

- Click vào một vài gói và xem

Mục đích của tấm hình này chỉ để cung cấp cho các bạn cái nhìn tổng quan về việc giao tiếp giữa trình duyệt của bạn và web server.

<img src="https://i.imgur.com/tNumST9.png">

Bạn có thể thấy 1 số thông tin thông qua tấm hình (ở những mũi tên mà mình chỉ vào)

- Để render được trang web này, trình duyệt của mình đã tạo tới 106 requests

- Ở tab Headers, bạn có thể thấy giao tiếp HTTP.

  - Request URL: Yêu cầu được thực hiện bởi trình duyệt cho một nội dung cụ thể.
  - Remote Address : Địa chỉ IP của tài nguyên cùng với thông tin cổng. Cổng mặc định là 80 cho yêu cầu HTTP.
  - Request Method: Trong ví dụ này, nó cho máy chủ biết rằng nó chỉ quan tâm đến việc nhận dữ liệu chứ không phải POST nó. Mã trạng thái = 200 có nghĩa là máy chủ đã nói "Được rồi!"

- Response Headers: Có request headers thì phải có response headers. Trong thực tế, nó đang nói chuyện với trình duyệt. Ví dụ, khi nó nói Content-Encoding là gzip, nó đơn giản có nghĩa là tất cả dữ liệu được nén ở phía máy chủ bằng cách sử dụng thuật toán gzip và trình duyệt được cho là giải nén dữ liệu đó.

- Câu hỏi đặt ra là: làm thế nào và tại sao nó gửi dữ liệu dưới định dạng nén? Phần đó được quản lý bởi các tiêu đề yêu cầu. Trình duyệt đã gửi một tiêu đề được gọi là AcceptEncoding với một giá trị của gzip, deflate, sdch. Đó là cách HTTP nói với máy chủ, "hãy tiết kiệm băng thông! Tôi có thể giải nén dữ liệu bằng cách sử dụng gzip, deflate & sdch. Tôi thích nếu bạn gửi nó bằng gzip (vì gzip là tùy chọn đầu tiên)!” Về cơ bản, tiêu đề yêu cầu đã đi đến máy chủ và máy chủ bắt buộc phải gửi dữ liệu đã được nén bằng gzip.

Bạn có để ý tới server trong ảnh trên? Nó là nginx. Trên thực tế, nginx đang ngày một được sử dụng nhiều, trong top 1000 website bây giờ, có tới hơn 40% là đang sử dụng nginx, ví dụ như : Netflix, Dropbox, Pinterest, Airbnb, WordPress, Box, Instagram, GitHub, SoundCloud, Zappos, và Yandex.

#### 2.2 Thế nào là web server?

Nói một cách đơn giản, một máy chủ web là một máy chủ lưu trữ một ứng dụng lắng nghe các yêu cầu HTTP. Trách nhiệm của máy chủ web là phải hiểu (tức là hiểu HTTP) những gì trình duyệt đang nói và phản hồi một cách thích hợp.

Hiện tại có 3 web server đang chiếm ưu thế đó là Apache , Microsoft Internet Information Services (IIS) , và Nginx. Chỉ riêng 3 web server này đã chiếm 85% thị trường rồi. Tuy nhiên cần cân nhắc trước khi lựa chọn, bởi việc chuyển qua lại giữa 3 web server này có thể mang lại nhiều đau thương đấy :D.

### 3. Tại sao nên sử dụng Nginx?

- Hiệu suất: Nginx cực kì nhanh nagy cả khi có nhiều lượt truy cập.

- Nó có thể tăng tốc cho ứng dụng của bạn: Có thể đặt nginx ở trước backend và nó sẽ giúp bạn xử lí bớt các tác vụ.

- Nó có thể cân bằng tải

- Nginx scale rất tốt

- Nginx có thể được reconfigure và upgrade nhanh chóng mà ko ảnh hưởng tới trải nghiệm người dùng.

- Không tốn nhiều tài nguyên để cài đặt và bảo trì

- Dễ dàng sử dụng

**Tính năng chính của Nginx**

- Nginx có thể làm event-based reverse proxy server.

<img src="https://i.imgur.com/Fj929Qm.png">

- Được thiết kế theo dạng module

Nginx có thể được extend bởi nó support plug-ins. Bạn có thể build nginx từ source vì thế có thể loại bỏ một số module ko cần thiết.
