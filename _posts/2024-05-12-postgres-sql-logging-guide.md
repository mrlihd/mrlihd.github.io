---
layout: post
title: Hướng dẫn bật ghi log truy vấn SQL trên PostgreSQL
---
# Guide to enable SQL query logging (PostgreSQL)

Status: Drafting
Language: Vietnamese
Visuals: No
Type of content: Blog, Research

### Mở đầu

Trong quá trình research các lỗ hổng, đặc biệt đối với các lỗ hổng SQLi thì việc log lại mọi query được ứng dụng thực thi là rất quan trọng để theo dõi các bất thường trong quá trình xử lý. Hôm nay mình sẽ chia sẻ cách để cấu hình PostgreSQL logging trên một vài môi trường thường gặp.

### Windows/Linux

Việc cần làm là thay đổi cấu hình trong file `postgresql.conf`, có thể xác định vị trí của file này bằng query trong PosgreSQL:

![Untitled](Guide%20to%20enable%20SQL%20query%20logging%20(PostgreSQL)%20a086c00ed2404348a280babdb19dc648/Untitled.png)

Vị trí thường gặp của file config này trên các OS như sau:

- Windows: `C:\Program Files\PostgreSQL\<version>\data\postgresql.conf`
- Linux: `/var/lib/postgresql/data/postgresql.conf`

Sau khi xác định vị trí file, cần enable và set nội dung cấu hình tại các entry sau:

1. `log_statement = ‘all’` 
2. `logging_collector = on`
3. `log_directory = ‘log’` (chọn folder sẽ ghi các file log, lưu ý về quyền write của postgresql tại folder đó)
4. `log_filename` (enable và custom tên file log sẽ tạo)

Sau khi lưu lại file cấu hình, cần restart service PostgreSQL để apply cấu hình mới

- Windows:
    - sử dụng service.msc
    - command sc
        - `sc stop <SERVICE_NAME>`
        - `sc start <SERVICE_NAME>`
- Linux: `systemctl restart <SERVICE_NAME>`

### Container

Đối với container, có thể trực tiếp sửa config khi container đang chạy hoặc apply config mới ngay khi khởi chạy container.

**Khi đang chạy**

Truy cập shell trong container

`docker exec -it <container_name/id> bash`

Sử dụng text editor (vi/nano) để thay đổi config như phần trên và lưu lại

Restart container

`docker restart <container_name/id>`

**Khi khởi tạo bằng docker-compose**

Thêm entry command trong file docker-compose.yml

```yaml
version: "3"
services:
  db:
    image: postgres:12-alpine
    command: ["postgres", "-c", "log_statement=all", "-c", "log_destination=stderr", "-c", "log_directory=log", "-c", "logging_collector=on"]
```

Sau đó khởi tạo container: `docker-compose up -d`

Kết quả

![Untitled](Guide%20to%20enable%20SQL%20query%20logging%20(PostgreSQL)%20a086c00ed2404348a280babdb19dc648/Untitled%201.png)

DB đã ghi lại mọi query được thực thi, từ đây researcher có thể dễ dàng theo dõi các hành vi bất thường của ứng dụng do mình gây ra 😎

### Kết luận

Trong blog này mình đã hướng dẫn các bạn một số cách để enable logging trên PostgreSQL. Hy vọng post này sẽ giúp ích cho các bạn và cả chính mình trong tương lai khi chẳng may quên mất 😵‍💫
