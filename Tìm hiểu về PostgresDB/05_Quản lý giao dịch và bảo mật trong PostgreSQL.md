# Quản lý Giao dịch và Bảo mật trong PostgreSQL

PostgreSQL là một hệ quản trị cơ sở dữ liệu quan hệ mạnh mẽ và phổ biến, nổi bật với tính năng quản lý giao dịch (transaction management) và bảo mật dữ liệu tiên tiến. Bài viết này sẽ tập trung giúp bạn:

- **Tìm hiểu về giao dịch trong PostgreSQL**, các cấp độ isolation, kiểm soát đồng thời, đảm bảo tính toàn vẹn dữ liệu.
- **Hướng dẫn bảo mật PostgreSQL**, bao gồm xác thực người dùng, phân quyền, vai trò và các tùy chọn bảo mật nâng cao như SSL/TLS, mã hóa dữ liệu.

---

## 1. Giao dịch (Transaction) trong PostgreSQL

### 1.1 Giao dịch là gì?

Giao dịch là một tập hợp các thao tác truy cập và thay đổi dữ liệu được thực hiện như một đơn vị logic duy nhất. Nếu có sự cố xảy ra, toàn bộ giao dịch sẽ bị hủy để đảm bảo dữ liệu không bị sai lệch.

### 1.2 Tính chất ACID trong PostgreSQL

- **Atomicity (Nguyên tử):** Toàn bộ giao dịch được thực hiện hoặc không thực hiện.
- **Consistency (Tính nhất quán):** Cơ sở dữ liệu phải được giữ nhất quán trước và sau giao dịch.
- **Isolation (Cô lập):** Các giao dịch chạy đồng thời không làm ảnh hưởng đến nhau.
- **Durability (Bền vững):** Sau khi giao dịch commit, dữ liệu được lưu trữ bền vững, ngay cả khi có lỗi hệ thống.

---

### 1.3 Các cấp độ Isolation trong PostgreSQL

Isolation quyết định cách các giao dịch nhìn thấy dữ liệu của nhau khi chạy đồng thời. PostgreSQL tuân theo chuẩn SQL và hỗ trợ 4 cấp độ isolation chính:

| Cấp độ Isolation         | Đặc điểm                             | Ví dụ vấn đề tránh được          |
|-------------------------|------------------------------------|---------------------------------|
| **Read Uncommitted**     | Cho phép đọc dữ liệu chưa commit (dirty read) | Có thể đọc được dữ liệu chưa hoàn thành giao dịch khác (PostgreSQL thực chất không hỗ trợ cấp độ này, nó sẽ coi như Read Committed)|
| **Read Committed (Mặc định)** | Chỉ đọc dữ liệu đã commit          | Tránh dirty read nhưng không tránh non-repeatable read |
| **Repeatable Read**      | Đảm bảo đọc nhất quán trong một giao dịch | Tránh non-repeatable read, có thể xuất hiện phantom read |
| **Serializable**         | Đảm bảo tính tuần tự tuyệt đối (cao nhất) | Tránh tất cả hiện tượng bất đồng bộ khác |

---

#### Ví dụ: Thay đổi isolation level trong PostgreSQL

```sql
-- Thiết lập cấp độ isolation cho giao dịch hiện tại
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- các câu lệnh SQL bên trong transaction
COMMIT;
```

---

### 1.4 Kiểm soát đồng thời (Concurrency Control)

PostgreSQL sử dụng **Multiversion Concurrency Control (MVCC)**, cho phép nhiều phiên làm việc truy cập dữ liệu đồng thời mà không gây khóa lâu dài.

- Mỗi transaction sẽ có phiên bản snapshot riêng biệt của dữ liệu.
- Transaction có thể đọc dữ liệu ổn định tại thời điểm bắt đầu mà không bị ảnh hưởng bởi giao dịch khác chưa commit.
- Khi có tranh chấp ghi chép, hệ thống xử lý dựa trên thứ tự commit.

---

### 1.5 Đảm bảo tính toàn vẹn dữ liệu

- Qua **ràng buộc (constraints)**: PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK.
- Qua **transaction và rollback** nếu có lỗi.
- Qua **triggers** để tự động kiểm soát và xử lý nghiệp vụ.

---

## 2. Bảo mật trong PostgreSQL

### 2.1 Xác thực người dùng (Authentication)

PostgreSQL hỗ trợ đa dạng các phương thức xác thực trong file `pg_hba.conf`:

| Phương thức       | Mô tả                                            |
|-------------------|-------------------------------------------------|
| `trust`           | Không yêu cầu mật khẩu (chỉ thử nghiệm hoặc môi trường tin cậy)|
| `password`        | Yêu cầu mật khẩu, truyền không mã hóa            |
| `md5`             | Yêu cầu mật khẩu mã hóa MD5 (phổ biến nhất)      |
| `scram-sha-256`   | Xác thực mật khẩu an toàn hơn bằng SCRAM-SHA-256|
| `peer`            | Xác thực theo user OS (trên Linux/Unix)         |
| `gss`/`sspi`      | SSO Kerberos (trên hệ thống hỗ trợ)              |
| `ldap`/`cert`     | Xác thực qua LDAP hoặc certificated SSL          |

---

#### Ví dụ cấu hình `pg_hba.conf` với `scram-sha-256`:

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             0.0.0.0/0               scram-sha-256
```

Để sử dụng SCRAM-SHA-256, cần PostgreSQL 10 trở lên.

---

### 2.2 Phân quyền (Authorization)

- PostgreSQL dùng hệ thống **Role** (vai trò) mà các user sẽ được gán quyền.
- Roles có thể là USER hoặc GROUP.
- Các quyền phổ biến: SELECT, INSERT, UPDATE, DELETE, EXECUTE, USAGE, và các cấp quyền quản lý khác.

---

#### Ví dụ tạo role và gán quyền:

```sql
-- Tạo role user mới
CREATE ROLE app_user WITH LOGIN PASSWORD 'strongpassword';

-- Gán quyền SELECT trên schema public
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_user;

-- Tạo role nhóm
CREATE ROLE read_only_group;

-- Gán user vào nhóm
GRANT read_only_group TO app_user;

-- Gán quyền cho nhóm
GRANT SELECT ON ALL TABLES IN SCHEMA public TO read_only_group;
```

---

### 2.3 Vai trò (Roles)

- **Role login:** Có thể đăng nhập, tương đương user.
- **Role không login:** Dùng làm nhóm, gom nhiều quyền lại.
- Bạn có thể thay đổi quyền role hoặc kích hoạt quyền của role trong session:

```sql
SET ROLE role_name;
RESET ROLE;
```

---

### 2.4 Các biện pháp bảo mật dữ liệu nâng cao

#### a) SSL / TLS trong PostgreSQL

- PostgreSQL hỗ trợ mã hóa kênh giao tiếp giữa client - server bằng SSL / TLS.
- Để bật SSL, cấu hình trong `postgresql.conf`:

```conf
ssl = on
ssl_cert_file = '/path/to/server.crt'
ssl_key_file = '/path/to/server.key'
```

- Khi kết nối, client có thể yêu cầu xác thực bằng cách sử dụng SSL, đảm bảo dữ liệu truyền đi không bị nghe lén.

---

#### b) Mã hóa dữ liệu

- PostgreSQL không có tính năng mã hóa dữ liệu ở mức dòng (row-level encryption) trong lõi.
- Bạn cần triển khai mã hóa dữ liệu ở tầng ứng dụng hoặc dùng các extension như:

  - `pgcrypto` để mã hóa dữ liệu cột (ví dụ: mã hóa trường nhạy cảm).
  - Mã hóa ổ đĩa/volume bằng phần mềm OS (LUKS, BitLocker).
  
##### Ví dụ mã hóa dữ liệu sử dụng `pgcrypto`:

```sql
-- Cài đặt extension
CREATE EXTENSION pgcrypto;

-- Mã hóa dữ liệu
INSERT INTO users (username, password_enc)
VALUES ('alice', crypt('mypassword', gen_salt('bf')));

-- Kiểm tra mật khẩu
SELECT username FROM users
WHERE password_enc = crypt('mypassword', password_enc);
```

---

#### c) Tùy chọn bảo mật nâng cao khác

- **Audit Logging:** Ghi lại các truy vấn, thao tác đăng nhập để giám sát.
- **Row Level Security (RLS):** Giới hạn truy cập dữ liệu ở cấp hàng.
- **Logical replication và physical replication với bảo mật**.

---

### 2.5 Ví dụ bật Row Level Security:

```sql
-- Bật RLS cho bảng
ALTER TABLE employees ENABLE ROW LEVEL SECURITY;

-- Tạo policy giới hạn quyền xem dữ liệu theo user
CREATE POLICY emp_policy ON employees
FOR SELECT USING (employee_user = current_user);
```

---

## Kết luận

PostgreSQL cung cấp đầy đủ công cụ để quản lý giao dịch đồng thời và bảo mật dữ liệu một cách hiệu quả:

- Giao dịch với tiêu chuẩn ACID và cấp độ isolation linh hoạt kết hợp MVCC giúp đảm bảo dữ liệu chính xác và không bị nghẽn.
- Hệ thống xác thực đa dạng, bảng phân quyền dựa trên role giúp quản lý user linh hoạt.
- Các tùy chọn bảo mật nâng cao: SSL/TLS, mã hóa, RLS và audit giúp bảo vệ dữ liệu toàn diện.

---

## Tham khảo thêm

- [PostgreSQL Documentation: Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [PostgreSQL Documentation: Client Authentication](https://www.postgresql.org/docs/current/client-authentication.html)
- [PostgreSQL Documentation: Role Management](https://www.postgresql.org/docs/current/user-manag.html)
- [PostgreSQL Documentation: Security](https://www.postgresql.org/docs/current/security.html)
- [pgcrypto extension](https://www.postgresql.org/docs/current/pgcrypto.html)

---

Nếu bạn cần trợ giúp thêm về code hay cấu hình cụ thể, đừng ngần ngại hỏi nhé!