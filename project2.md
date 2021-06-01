# Project 2: NFS server
1.Lý thuyết 


+ NFS là một cơ chế cho phép lưu trữ và truy vấn dữ liệu từ đĩa thông qua một mạng chia sẻ. Với NFS nó cho phép người dùng truy cập tới các file, thư mục ở xa (remote) như cách đang truy cập hệ thống file ở máy trạm. NFS khởi xướng và phát triển bởi Sun Microsystems

+ Mặc định giao thức NFS làm việc trên cổng 111/udp và 2049/tcp

2.Mô hình sử dụng

+ Mô hình lần này sẽ sử dụng : 1 NFS server và 2 client

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/ywv5kah82p_image.png)

***Phía trên là mô hình tổng quát và địa chỉ IP***          
+ Các địa chỉ IP của các thiết bị cụ thể:

  + IP server


 ![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/voa1v381p7_image.png)

  + IP các client 1, 2

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/369ber391l_image.png)

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/yv8n5rs41h_image.png)

3.Các bước cài đặt

- Trên server

  +Cài đặt:
`yum install nfs-util`
  
  +Chuẩn bị thư mục chia sẻ: Theo đề sẽ là /DATA
`mkdir /share-data`
  
  +Thay đổi permission cho thư mục

`chmod -R 755 /DATA`

`chown nfsnobody:nfsnobody /DATA`
  
  +Kích hoạt và chạy các dịch vụ cần thiết:

`systemctl enable rpcbind`

`systemctl enable nfs-server`
 
`systemctl enable nfs-lock`
 
`systemctl enable nfs-idmap`

`systemctl start rpcbind`

`systemctl start nfs-server`
 
`systemctl start nfs-lock`
 
`systemctl start nfs-idmap`


+Thiết lập quyền truy cập (client truy cập được từ một IP cụ thể của Client), cập nhật quyền này vào file  `/etc/exports`

`/DATA    172.16.1.27(rw,sync,no_root_squash,no_all_squash)`

`/DATA    172.16.1.24(rw,sync,no_root_squash,no_all_squash)`

Với 2 địa chỉ IP lần lượt là địa chỉ client 1 và 2

Nếu IP thay bằng * thì mọi client có quyền truy cập.

+Chạy dịch vụ NFS
`systemctl restart nfs-server`

+Mở cổng cho NFS qua firewall
`firewall-cmd --permanent --zone=public --add-service=nfs` 

`firewall-cmd --permanent --zone=public --add-service=mountd` 

`firewall-cmd --permanent --zone=public --add-service=rpc-bind`

`firewall-cmd --reload`

- Trên các máy client( Làm với client 1 tương tự với 2)

+Cài gói nfs-utils sau cài trên máy client CentOS
`yum install nfs-utils`

+Gán ổ đĩa NFS Server vào máy Client

Thiết lập để thư mục `/DATA` từ NFS Server có IP 172.16.1.26, sẽ gắn vào `/datadcchiase` của máy Client

`mkdir -p /datadcchiase`

`mount -t nfs 172.16.1.26:/DATA /datadcchiase`

Giờ thì từ client truy cập đến `/datadcchiase` tương ứng đang truy cập `/DATA `của Server NFS.

4.Kết quả 
- Trên client1 tạo file ở mục `/datadcchiase`
![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/6qwsgwezft_test1.PNG)

- Kiểm tra trên client2 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/axcjn5pv9c_test2.PNG)

Đã thấy file client1 tạo=> Dữ liệu đã đồng bộ
