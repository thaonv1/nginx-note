# Nginx Modules

Như ta đã biết, Nginx được xây dựng theo kiến trúc modular. Module chính là thứ giúp chúng ta define xem nginx này sẽ làm nhiệm vụ gì.

Mặc định thì Nginx sẽ có những core modules và các core module này thì không thể bị disable

<img src="https://i.imgur.com/kSu88In.png">

## Module làm việc như thế nào?

### Module structure

Modules thường chứa thông tin, cấu hình, handlers, filters và load balancer functions. 
