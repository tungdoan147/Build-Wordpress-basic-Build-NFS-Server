# Project 1: Dựng Web Server chạy WordPress trên server linux (tự chọn Ubuntu hoặc CentOS)

Yêu cầu:
+ Web chạy bằng nginx hoặc Apache
+ Cấu hình ít nhất 2 virtual site
+ Set Basic-Authen để 2 user vào được site
+ Quản lý database bằng phpmyadmin và mysql CLI
+ Cấu hình dùng nginx làm proxy của Apache (chạy chung trên 1 server)
+ Config SSL HTTPS

Chọn CentOS

### 1+2.Web chạy bằng Apache + 2 virtual site

#### + Bước 1: Cài đặt stack LAMP
  + Cài đặt Apache server với các cấu hình căn bản
  
     -Cài đặt gói Apache httpd service bằng lệnh:
    
       `# yum install httpd -y ` 
      
     -Khởi động Apache 
    
       `# systemctl start httpd`
       
       `# systemctl enable httpd `
      
      -Mở dịch vụ http (mở port) trên Firewall
      
      `# firewall-cmd --permanent --add-service=http`
      
      `# firewall-cmd --permanent --add-service=https`
      
      `# systemctl restart firewalld`

	-Truy cập đến địa chỉ IP server với giao thức HTTP qua đường dẫn: http://IP-server để tải nội dung trang mặc định của Apache



   + Cài đặt PHP
   
   -Ở đây em sử dụng PHP 5.6 ( Vì wordpress hiện tại đã chỉ nhận bản PHP 5.6 trở lên)
 
   -Cài đặt và khởi động 2 repo, đó là EPEL và Remi trên CentOS 7

`# yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm`

 `# yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm`
   -Cài yum utils:

  `# yum install yum-utils`

   -Download và cài đặt các gói bổ sung:
 
 `# yum-config-manager --enable remi-php56`

`# yum-config-manager --enable remi-php56   [Install PHP 5.6]`
    
`# yum install php php-mcrypt php-cli php-gd php-curl php-mysql php-ldap php-zip php-fileinfo`

  -Kiểm tra version của php

`#php -v`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/v6vn5pga6r_image.png)

  -Tạo file test php như trên mạng

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/fj2f3cq81e_image.png)

  + Cài đặt và cấu hình MariaDB Database

`yum install mariadb-server mariadb`

  -Khởi động và cấu hình


`systemctl start mariadb`

`systemctl enable mariadb`

`mysql_secure_installation`

  -Để kiểm tra lại hoạt động database server,  đăng nhập vào MariaDB bằng tài khoản root

`mysql -u root -p `

`show databases;`

#### + Bước 2: Cài đặt 2 virtual host 

-Trong bài viết e sẽ sử dụng 2 virtual host( 2 tên miền trên cùng 1 địa chỉ IP máy chủ ) là web1.com và web2.com

-Tạo thư mục và phân quyền

`mkdir -p /var/www/web1.com/`

`mkdir -p /var/www/web2.com/`

`chmod -R 755 /var/www`

-Tạo trang demo

`vi /var/www/web1.com/index.html`

Trong file này ta tạo một đoạn code HTLM đơn giản để trang đó kết nối với site mà ta muốn

`Daylaweb1`

Tương tự với vidu2.com

-Tạo các file Virtual Host mới

Đầu tiên ta cần cài danh mục lưu trữ virtual host cũng như danh mục thông báo cho Apache rằng virtual host đã sẵn sàng phục vụ truy cập. Danh mục  sites-available sẽ chứa các file virtual host, trong khi danh mục sites-enabled sẽ chứ các link tới virtual host mà chúng ta muốn đưa ra. Ta có thể tạo cả hai danh mục này bằng cách:

`mkdir /etc/httpd/sites-available`

`mkdir /etc/httpd/sites-enabled`

-Tạo File Virtual host cho web1.com: ( tương tự với web2.com)

 `vi /etc/httpd/sites-available/web1.com.conf`


>
       <VirtualHost *:80>
       ServerAdmin admin@web1.com
       ServerName web1.com
       ServerAlias www.web1.com
       DocumentRoot /var/www/web1.com
       DirectoryIndex index.php index.html
       ErrorLog /var/www/web1.com/error.log
       CustomLog /var/www/web1.com/requests.log combined
       </VirtualHost> 

-Kích hoạt Virtual host

`ln -s /etc/httpd/sites-available/web1.com.conf /etc/httpd/sites-enabled/web1.com.conf`

`ln -s /etc/httpd/sites-available/web2.com.conf /etc/httpd/sites-enabled/web2.com.conf`

-Tắt SE Linux: Ngoài firewalld SE Linux cũng cần phải được disabled để thực hiện được nhiệm vụ

`vi /etc/selinux/config`

Đổi trạng thái SELINUX=disabled

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/2g75ufysly_image.png)

Để kiểm tra dùng lệnh `sestatus`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/cvrx8m1opt_image.png)

-Các nguồn tài liệu thường hướng dẫn tới đây. Nhưng thực ra chúng ta cần phải trỏ ip trong file hosts của windows để có thể phân dải 2 tên miền web1.com và web2.com về ip của Web server.

+ Truy cập Notepad với quyền admins 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/sj9h7apo8m_image.png)

+ Dùng Notepad Open theo đường dẫn. Chọn file `hosts`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/ke69bnr2p3_image.png)

+ Thêm vào file địa chỉ IP cũng như 2 tên miền web1.com và web2.com

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/h255m326f_image.png)

Làm tất cả xong truy nhập vào trang web vào file index.html của web1.com và web2.com sẽ thu được kết quả

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/8odgcbjihu_image.png)

#### + Bước 3: Cài đặt Wordpress

+ Ở đây e sẽ cài đặt wordpress cho mỗi trang web vừa tạo ( mỗi trang sẽ sử dụng 1 database riêng)
+ Thực hiện download file wordpress, cài đặt như bình thường. Tiếp đó sử dụng MySQL tạo 2 database. Chuyển file wordpress vào 2 thư mục /var/www/web1 và /var/www/web2 
+ Sau đó vào file cấu hình wordpress của 2 thư mục trên chỉnh đúng dữ liệu database
+ Làm hết các bước chúng ta sẽ thu được kết quả khi truy nhập vào 2 trang web1.com và web2.com như sau

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/3ck6doo9e1_image.png)

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/zl479w0q5z_image.png)

### 3.Set Basic-Authen để 2 user vào được site

+ Quản lý bằng mysql CLI

Sử dụng lệnh `mysql -u root -y ` để truy cập cào MySQL

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/ecpjma00sq_image.png)

Ở đây ta có thao tác để tạo, xem những databases.....

+ Quản lý bằng phpmyadmin

 -Cài đặt: 
 
 `yum install http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el7.rf.x86_64.rpm`
 
 `yum install phpmyadmin`
 
 -Cấu hình 
 
 `vi  /etc/httpd/conf.d/phpMyAdmin.conf`
 
  Sử dụng # để khóa những dòng sau
 
>`# Require ip 127.0.0.1`
`# Require ip ::1`

Thêm nội dung sau vào ngay bên dưới 

`Require all granted`

Copy file mẫu config.sample.inc.php và đổi tên thành config.inc.php trong thư mục /usr/share/phpMyAdmin/

Điều chỉnh lại nội dung file config.inc.php

`vi /usr/share/phpMyAdmin/config.inc.php`

Tìm tất cả các giá trị “cookie” có trong file và thay thế bằng “http“.

  - Truy nhập phpMyAdmin

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/ottv72hazn_image.png)

Tại đây ta có thể thao tác với các databases.

### 4.Quản lý database bằng phpmyadmin và mysql CLI


### 5.Cấu hình SSL  HTTPS


### 6.Cấu hình dùng nginx làm proxy của Apache (chạy chung trên 1 server)


Mô hình 




