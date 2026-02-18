# Hướng Dẫn Cài Đặt và Làm Quen Với Môi Trường Làm Việc PostgreSQL

PostgreSQL là một hệ quản trị cơ sở dữ liệu quan hệ mã nguồn mở mạnh mẽ, được sử dụng rộng rãi trong nhiều ứng dụng từ nhỏ đến lớn. Bài viết này sẽ hướng dẫn bạn cài đặt PostgreSQL trên các nền tảng phổ biến như Windows, Linux và MacOS. Đồng thời, sẽ giới thiệu các công cụ cơ bản (psql và pgAdmin) giúp bạn làm quen với PostgreSQL, cũng như các thao tác đầu tiên: tạo cơ sở dữ liệu, tạo user, tạo bảng và thao tác với dữ liệu bằng các câu lệnh SQL cơ bản.

---

## 1. Cài Đặt PostgreSQL

### 1.1 Trên Windows

1. **Tải về PostgreSQL Installer:**

Truy cập trang chính thức: https://www.postgresql.org/download/windows/  
Chọn bản phù hợp với hệ điều hành.

2. **Chạy file cài đặt:**

- Chọn các thành phần cần cài: PostgreSQL Server, pgAdmin 4 (giao diện quản lý), Stack Builder (công cụ bổ sung).
- Đặt mật khẩu cho user `postgres` (superuser mặc định).
- Chọn cổng mặc định (5432).
- Hoàn tất cài đặt.

3. **Kiểm tra cài đặt:**

Mở Command Prompt và gõ:

```sh
psql -U postgres
```

Nhập mật khẩu khi được hỏi để đăng nhập vào PostgreSQL shell.

---

### 1.2 Trên Linux (Ubuntu/Debian)

1. **Cài đặt PostgreSQL từ kho lưu trữ:**

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

2. **Khởi động dịch vụ (thường tự động):**

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

3. **Đăng nhập với user mặc định:**

PostgreSQL tạo tài khoản Linux `postgres` tự động, dùng để đăng nhập vào DB:

```bash
sudo -i -u postgres
psql
```

---

### 1.3 Trên MacOS

1. **Sử dụng Homebrew:**

Nếu chưa cài Homebrew, bạn tham khảo [Homebrew](https://brew.sh).

2. **Cài đặt PostgreSQL:**

```bash
brew update
brew install postgresql
```

3. **Khởi động PostgreSQL:**

```bash
brew services start postgresql
```

4. **Kiểm tra bằng cách đăng nhập:**

```bash
psql postgres
```

---

## 2. Cấu Hình Ban Đầu

Sau khi cài xong, bạn nên tạo password cho `postgres` user nếu chưa thiết lập rõ ràng.

- **Trên psql shell:**

```sql
\password postgres
```

Sau đó nhập mật khẩu mới.

---

## 3. Giới Thiệu Công Cụ Làm Việc Với PostgreSQL

### 3.1 psql - Giao diện dòng lệnh

- Là công cụ CLI truy cập và thao tác trực tiếp trên PostgreSQL.
- Cú pháp cơ bản:

```bash
psql -U username -d dbname
```

- Một số lệnh hữu ích trong psql:

| Lệnh           | Chức năng                           |
|----------------|-----------------------------------|
| `\l`           | Hiển thị danh sách cơ sở dữ liệu   |
| `\c dbname`    | Kết nối tới cơ sở dữ liệu khác     |
| `\dt`          | Hiển thị bảng trong cơ sở dữ liệu  |
| `\q`           | Thoát psql                        |

---

### 3.2 pgAdmin - Giao diện GUI

- pgAdmin là công cụ quản lý GUI thân thiện với người dùng.
- Phù hợp để tạo database, user, viết query,... dễ dàng cho người mới.
- Sau khi cài đặt, mở pgAdmin, đăng nhập bằng thông tin `postgres`.

---

## 4. Thao Tác Với PostgreSQL - Ví Dụ Cụ Thể

### 4.1 Tạo cơ sở dữ liệu

Đăng nhập psql (hoặc qua pgAdmin), thực hiện:

```sql
CREATE DATABASE testdb;
```

Kiểm tra cơ sở dữ liệu vừa tạo:

```sql
\l
```

---

### 4.2 Tạo user (role)

Tạo user mới có mật khẩu:

```sql
CREATE USER testuser WITH PASSWORD 'testpass';
```

Cấp quyền cơ bản (thí dụ quyền kết nối và toàn quyền trên cơ sở dữ liệu testdb):

```sql
GRANT ALL PRIVILEGES ON DATABASE testdb TO testuser;
```

---

### 4.3 Kết nối tới cơ sở dữ liệu mới

```bash
psql -U testuser -d testdb
```

---

### 4.4 Tạo bảng và thao tác dữ liệu

```sql
-- Tạo bảng
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    position VARCHAR(50),
    salary NUMERIC
);

-- Thêm dữ liệu
INSERT INTO employees (name, position, salary)
VALUES 
  ('Nguyen Van A', 'Developer', 1200),
  ('Tran Thi B', 'Manager', 1500);

-- Xem dữ liệu
SELECT * FROM employees;

-- Cập nhật dữ liệu
UPDATE employees SET salary = 1300 WHERE name = 'Nguyen Van A';

-- Xoá dữ liệu
DELETE FROM employees WHERE name = 'Tran Thi B';
```

---

### 4.5 Các câu lệnh SQL cơ bản trong PostgreSQL

| Câu lệnh          | Mô tả                                          | Ví dụ                                                            |
|-------------------|------------------------------------------------|-----------------------------------------------------------------|
| `SELECT`          | Truy vấn dữ liệu                               | `SELECT * FROM employees;`                                       |
| `INSERT INTO`     | Thêm dữ liệu                                   | `INSERT INTO employees(name) VALUES ('Le Van C');`               |
| `UPDATE`          | Cập nhật dữ liệu                               | `UPDATE employees SET salary=2000 WHERE id=1;`                   |
| `DELETE`          | Xóa dữ liệu                                    | `DELETE FROM employees WHERE id=2;`                              |
| `CREATE TABLE`    | Tạo bảng mới                                   | `CREATE TABLE example(id SERIAL PRIMARY KEY, data TEXT);`         |
| `DROP TABLE`      | Xóa bảng                                       | `DROP TABLE example;`                                            |
| `ALTER TABLE`     | Thay đổi cấu trúc bảng                         | `ALTER TABLE employees ADD COLUMN birthday DATE;`                 |

---

## Tổng Kết

- PostgreSQL có thể dễ dàng cài đặt trên Windows, Linux và MacOS.
- Sử dụng `psql` giao diện dòng lệnh để thao tác trực tiếp hoặc dùng `pgAdmin` cho giao diện đồ họa.
- Khởi đầu nên tạo database, tạo user, rồi dùng các câu lệnh SQL cơ bản để tạo bảng và thao tác dữ liệu.
- PostgreSQL là công cụ mạnh mẽ phù hợp học tập và phát triển các ứng dụng thực tế.

---

## Tài liệu tham khảo

- https://www.postgresql.org/docs/
- https://www.postgresql.org/download/
- https://www.pgadmin.org/docs/

---

Xin chúc bạn thành công trong việc cài đặt và làm quen với PostgreSQL! Nếu cần trợ giúp thêm, đừng ngần ngại hỏi nhé.