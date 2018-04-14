# Triển khai ứng dụng Laravel trên shared hosting

* Hiện nay các đơn vị cung cấp host cũng vẫn đang cung cấp các sản phẩm host dùng chung (shared hosting) bên cạnh máy chủ ảo VPS. Đặc điểm của host dùng chung là sử dụng rất đơn giản thông qua các hệ thống quản trị hosting còn gọi là control panel như CPanel, Plesk, ISPConfig… Các ứng dụng Laravel không được thiết kế để sử dụng luôn cho host dùng chung, các ứng dụng có điểm xuất phát đều từ file index.php nằm trong thư mục public. Do đó khi triển khai lên các host dùng chung sẽ phải có một số điều chỉnh.

* Trong bài viết chúng ta sẽ cùng nhau cài đặt ứng dụng Laravel trên host dùng chung sử dụng CPanel (các nhà cung cấp host dùng chung đa phần sử dụng CPanel).


## Bước 1: Chuẩn bị tên miền và shared hosting

* Tên miền quốc tế .com, .net… các bạn có thể mua ở các nhà cung cấp nước ngoài. Tên miền Việt Nam như .vn, .com.vn… bắt buộc phải mua của các đơn vị cung cấp tên miền trong nước như Tenten, PA Vietnam, Matbao, Nhân hòa… Tiếp đến là lựa chọn hosting, mua hosting trong nước hay quốc tế đều có những ưu và nhược điểm:

### Hosting quốc tế:

* Giá rẻ hơn hẳn trong nước, khoảng 4$/ tháng tương đương với gói khoảng 200k/ tháng của các nhà cung cấp trong nước…
* Tài nguyên dùng thoải mái hơn, không giới hạn về dung lượng, băng thông, số lượng database…
* Nhược điểm là các website của chúng ta thường phục vụ cho người Việt Nam, do đó khi tải trang từ máy chủ trong nước sẽ nhanh hơn, tuy nhiên không phải đơn vị cung cấp host nào trong nước cũng cam kết điều này. Thứ nữa là cá mập cũng rất thích cắn cáp quang làm đường truyền ra quốc tế thỉnh thoảng cũng bị gián đoạn.

## Bước 2: Chuẩn bị source code, phần mềm FTP để truyền file lên máy chủ

* Do host dùng chung không thể thực hiện được các câu lệnh kiểu dòng lệnh do đó source code cần phải lấy đầy đủ source code bao gồm cả các thư viện vendor hay các gói javascript cài đặt thông qua npm. Như vậy, bạn nén tất cả source code của ứng dụng sẽ được triển khai với định dạng file .zip (CPanel không hỗ trợ định dạng nén rar).

* Tiếp theo, thực hiện vào CPanel tạo tài khoản FTP và kết nối phần mềm FTP client với máy chủ qua tài khoản đó. Dùng giao thức FTP cho tốc độ nhanh vì nhiều source code rất lớn lên đến hàng GB.

## Bước 3: Cài đặt ứng dụng Laravel trên shared hosting

* Giải nén file zip ra thư mục gốc thường là public_html.
* Copy toàn bộ file trong thư mục public ra thư mục gốc public_html.
* Chỉnh sửa file index.php trong thư mục gốc (đây chính là file được chuyển từ thư mục public ra thư mục gốc).
```php
require __DIR__.'/../vendor/autoload.php';
$app = require_once __DIR__.'/../bootstrap/app.php';
```
Thành
```php
require __DIR__.'/vendor/autoload.php';
$app = require_once __DIR__.'/bootstrap/app.php';
```
* Chỉnh sửa file server.php nằm trong thư mục gốc dự án từ:
```php
if ($uri !== '/' && file_exists(__DIR__.'/public'.$uri)) {
    return false;
}

require_once __DIR__.'/public/index.php';
```
thành
```php
if ($uri !== '/' && file_exists(__DIR__.'/'.$uri)) {
    return false;
}

require_once __DIR__.'/index.php';
```
* Mở file .htaccess và thay đổi nội dung như sau để mọi request đến sẽ trỏ về file index.php ở thư mục gốc.
```js
DirectoryIndex index.php

    Options -MultiViews

RedirectMatch 404 /\.git

RewriteEngine On

RewriteCond %{REQUEST_URI}::$1 ^(/.+)/(.*)::\2$
RewriteRule ^(.*) - [E=BASE:%1]

RewriteCond %{ENV:REDIRECT_STATUS} ^$
RewriteRule ^index\.php(/(.*)|$) %{ENV:BASE}/$2 [R=301,L]

RewriteCond %{REQUEST_FILENAME} -f
RewriteRule .? - [L]

RewriteRule .? %{ENV:BASE}/index.php [L]
```
* Mở file .env và thay đổi một số thông số
```php
APP_URL=http://localhost
```
thành
```php
APP_URL=yourdomain
```
Như vậy, chúng ta đã thiết lập xong ứng dụng Laravel để có thể chạy trên host dùng chung. Các bước thực hiện có ảnh hưởng nhiều đến source code do đó không chuẩn tắc. Nếu bạn có ý định triển khai một ứng dụng Laravel thì tốt nhất nên nghĩ đến máy chủ ảo VPS, sẽ phức tạp hơn khi chúng ta phải tự cài webserver như apache hoặc nginx, cơ sở dữ liệu mysql và một số các ứng dụng khác cho cache…

