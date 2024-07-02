# Hướng dẫn cài đặt hệ thống chấm điểm trực tuyến VNOJ sử dụng Docker
Với Docker, quá trình cài đặt của bạn sẽ giảm thiểu được xung đột với những phần mềm khác được cài trên máy, giúp hệ thống hoạt động ổn định hơn.
## Chuẩn bị
### Một số yêu cầu về hệ thống:

✅ OS: Ubuntu 20.04 trở lên

✅ Storage: 20 GB trở lên

✅ CPU: 1 core trở lên

✅ RAM: 1 GB trở lên

Tùy theo thực tế và nhu cầu sử dụng, cấu hình và các thông số có thể thay đổi. Ở đây, mình sử dụng 02 máy với cấu hình như sau: 

* Máy chủ (tạm gọi là Local Server) - Cài đặt webserver và chạy 02 máy chấm song song (judge01 và judge02)
   
✅ Ubuntu 22.04 Server/2 Core/4 GB RAM/60 GB SSD

✅ Username: devsmile

✅ IP: 192.168.1.60/24

✅ Judgename: judge01, judge02

✅ MySQL password: greenhat1998

* Máy chấm từ xa (tạm gọi là Remote Judge) - Cài đặt 01 máy chấm (judge03) và kết nối đến Local Server, áp dụng trong trường hợp bạn muốn tăng tốc độ chấm khi Local Server quá tải
  
✅ Ubuntu 20.04 Server/1 Core/2 GB RAM/60 GB SSD

✅ Username: judger

✅ IP: 192.168.1.4/24

✅ Judgename: judge03

✅ Kết nối được đến Local Server qua SSHFS

## Cài đặt Docker và Docker-Compose
Thực hiện trên Local Server và Remote Judge (nếu có)
### Cài đặt Docker 
> Tham khảo cách cài đặt từ [document chính thức của Docker](https://docs.docker.com/engine/install/ubuntu/)

Set up Docker's apt repository.
```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Install the Docker packages.
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io 
```
> Note: Hiện tại, chỉ có sudo mới có thể chạy các lệnh của Docker. Để các user khác cũng chạy được, cần thêm `sudo` vào trước các câu lệnh. Các lỗi như "docker: Got permission denied while trying to connect to the Docker daemon.." thường là do thiếu sudo trước câu lệnh.

### Cài đặt Docker-Compose
> Có thể tham khảo thêm tại [Install the Compose plugin](https://docs.docker.com/compose/install/linux/)
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
## Cài đặt site
Bước này chỉ cần thực hiện trên Local Server
### Tải về mã nguồn VNOJ Docker
```
git clone --recursive https://github.com/VNOI-Admin/vnoj-docker.git
cd vnoj-docker/dmoj
```
Kể từ lúc này, các câu lệnh đằng sau sẽ có thư mục hiện hành là `/dmoj`
### Cấu hình môi trường để sử dụng Docker
Thay đổi các thông số cài đặt nhằm phù hợp với mục đích sử dụng và tăng tính bảo mật cho webserver.
Có 3 nơi mà bạn có thể chỉnh sửa:
192.168.253.131
1. `dmoj/environment/`
cd environment/

Nơi này chứa các biến môi trường để build Docker image.
> Note: Đổi tên các file .example tương ứng thành: mysql-admin.env, mysql.env, site.env
nano mysql.example >Copy và đổi tên File thành mysql.env
* mysql.env
```
MYSQL_DATABASE=dmoj
MYSQL_USER=dmoj
MYSQL_PASSWORD=Thuocnv1202@			#thay doi password
```
>Đổi tên file mysql-admin.example thành mysql-admin.env bằng lệnh
nano mysql-admin.example
* mysql-admin.env
```
MYSQL_ROOT_PASSWORD=Thuocnv1202@		#thay doi password
```
* site.env
Đổi tên file site.example thành file site.env bằng lệnh
nano site.example
```
HOST=192.168.1.60				#thay bang IP cua Local Server
SITE_FULL_URL=http://192.168.1.60/
MEDIA_URL=http://192.168.1.60/
DEBUG=0
SECRET_KEY=abcdefghijklmnopqrstuvwxyz		#thay doi bang ma khoa tuy y
```
Dùng lệnh Cd để chuyển đến thư mục Conf.d bằng lệnh
cd dmoj/nginx/conf.d/
dùng lệnh nano để sử nội dung file nginx.conf
2. `dmoj/nginx/conf.d/nginx.conf`

Cấu hình hình tên server_name thành 192.168.1.60 hoặc domain name
Sau khi luu xong dung lenh cd ../.. để về thư mục dmoj
3. `dmoj/local_settings.py`

Hầu hết các thông số đã được cấu hình sẵn. Nếu muốn thêm tính năng nào thì bỏ dấu comment tính năng đó. 

Ví dụ: Để người dùng tự đăng ký tài khoản, tiến hành thêm các thông tin cấu hình email để thực hiện xác thực đăng ký tài khoản qua email (cần tạo mật khẩu ứng dụng cho email)

### Build Docker Image
Khởi tạo trước khi build
Dùng lệnh CD .. để về thư mục như đường dẫn sau: ~/vnoj-docker/dmoj$ 
copy lệnh dưới vào sau dòng lệnh trên
```
./scripts/initialize
```
Build Docker Image
```
sudo docker-compose build
```
Khởi động thành phần site để thực hiện cấu hình
```
sudo docker-compose up -d site
```
Khởi tạo dữ liệu cho Database
```
sudo ./scripts/migrate
```
Khởi tạo các file static
```
sudo ./scripts/copy_static
```
Load các dữ liệu cần thiết cho Website
```
sudo ./scripts/manage.py loaddata navbar
sudo ./scripts/manage.py loaddata language_small
sudo ./scripts/manage.py loaddata demo
```
### Sử dụng VNOJ Site
Quá trình cài đặt đến đây đã hoàn tất. Chạy câu lệnh bên dưới để khởi động tất cả các docker container.
```
sudo docker-compose up –d

```
nếu lệnh trên bị lỗi, ta sử dụng 2 câu lệnh dưới đây:
```
sudo service docker restart
sudo docker compose up -d
```
Truy cập http://192.168.1.60 để kiểm tra kết quả (IP Local Server).

Truy cập bài tập mẫu A+B và tiến hành upload testcase để kiểm tra.
## Cài đặt judge
Để đơn giản trong quá trình cài đặt, chúng ta sẽ tiếp tục cài đặt judge sử dụng Docker
> Note:<br>
	- Nếu cài đặt judge chạy trên Local Server, không cần cài đặt lại Docker.<br>
	- Nếu cài đặt judge trên Remote Judge, tiến hành cài đặt Docker theo hướng dẫn bên trên (không cần cài Docker-Compose)
> 
### Thiết lập cấu hình judge trên admin site
Truy cập http://192.168.1.60/admin/judge/

Tạo các judge, lưu lại tên judge id và key (ví dụ ở đây tạo 03 judge là judge01, judge02, judge03)
### Tạo môi trường biên dịch 
Tải về môi trường biên dịch (thực hiện trên Local Server và Remote Judge)
Trở về thư mục gốc bằng lệnh CD
```
git clone https://github.com/VNOI-Admin/judge-server
cd judge-server/.docker
sudo apt install make
sudo make judge-tier1
```
###sudo make judge-tiervnoj
Có thể thay thế `tiervnoj` bằng `tier1`, `tier2`, `tier3` (tier càng cao thì dung lượng càng lớn, càng được tích hợp nhiều ngôn ngữ hơn).

### Tạo judge trên Local Server
Tạo các file cấu hình tương ứng với mỗi judge có dạng là `judge_name.yml` (tên judge) và ghi những thông tin sau vào file:
```
id: <judge name>
key: <judge authentication key>
problem_storage_globs:
  - /problems/*
```
Ở đây, ta sẽ chạy 2 máy chấm `judge01` và `judge02` trên Local Server
Dùng lệnh CD để về thư mục gốc, rồi dùng lệnh CD để chuyển đến thư mục vnoj-docker/dmoj/problems 
để tạo file judge01.yml dùng lệnh 
```
nano judge01.yml
Copy nội dung: 
id: <judge name>
key: <judge authentication key>
problem_storage_globs:
  - /problems/*

và lưu lại
nano judge02.yml
nano judge03.yml
Build Docker Image
```
sudo docker run \
    --name judge01 \
    --network="host" \
    -v /home/devsmile/vnoj-docker/dmoj/problems:/problems \
    --cap-add=SYS_PTRACE \
    -d \
    --restart=always \
    vnoj/judge-tiervnoj:latest \
    run -p 9999 -c /problems/judge01.yml 192.168.1.60 -A 0.0.0.0 -a 9111
```

Kiểm tra trạng thái của máy test
sudo docker logs -ft judge01.yml
> Note: <br>
	- Với mỗi judge, cần thay thế judge01 (judge name), judge01.yml (judge config), 9111 (PID) tương ứng khác nhau.<br>
	- Các judge chạy trên cùng Local Server phải có ID khác nhau (thay 9111 thành 9112, 9113, ...)

### Tạo judge trên remote
Tạo folder để mount dữ liệu từ thư mục `problems` trên Local Server
```
mkdir -p /home/judger/problems
sudo chmod 775 –R problems
```
Mount dữ liệu từ thư mục `problems` trên Local Server về thư mục `problems` vừa tạo trên Remote Judge
```
sudo apt install sshfs 
sudo addgroup judger root
sudo sshfs devsmile@192.168.1.60:/home/devsmile/vnoj-docker/dmoj/problems /home/judger/problems -o allow_other		#IP và Username tren Local Server
cd /home/judger/problems
```
Tạo file cấu hình tương ứng với judge có dạng là judge_name.yml (tên judge) và ghi những thông tin sau vào file:
```
id: <judge name>
key: <judge authentication key>
problem_storage_globs:
  - /problems/*
```
Build Docker Image
```
sudo docker run \
    --name judge03 \
    --network="host" \
    -v /home/judger/problems:/problems \
    --cap-add=SYS_PTRACE \
    -d \
    --restart=always \
    vnoj/judge-tier1:latest \
    run -p 9999 -c /problems/judge03.yml 192.168.1.60 -A 0.0.0.0 -a 9113
```
### Kiểm tra trạng thái của máy chấm
Mở Docker logs để kiểm tra kết quả cài đặt Judge
```
sudo docker logs -ft judge01
```
Kiểm tra ở mục STATUS trên website để xem trạng thái kết nối của Judge đến Site. Sau đó thử nộp bài với các máy chấm khác nhau để kiểm tra kết quả.
Chúc các bạn thành công.