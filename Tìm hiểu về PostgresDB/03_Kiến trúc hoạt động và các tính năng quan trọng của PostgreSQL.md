# Kiến trúc hoạt động và các tính năng quan trọng của PostgreSQL

PostgreSQL là một hệ quản trị cơ sở dữ liệu quan hệ mã nguồn mở mạnh mẽ, nổi tiếng với tính ổn định, khả năng mở rộng và tính năng phong phú. Bài viết này sẽ đi sâu vào **kiến trúc hoạt động** của PostgreSQL, đặc biệt là các cơ chế trọng yếu như **MVCC (Multi-Version Concurrency Control)** và **WAL (Write-Ahead Logging)**, đồng thời giới thiệu các tính năng nổi bật như hỗ trợ JSON/JSONB, hệ thống kiểu dữ liệu đa dạng, khả năng mở rộng (extensibility), và trigger.

---

## 1. Kiến trúc hoạt động của PostgreSQL

### 1.1. Tổng quan kiến trúc

PostgreSQL hoạt động theo mô hình server-client, với các thành phần chính:

- **Postmaster**: tiến trình cha quản lý các tiến trình con.
- **Backend process**: mỗi kết nối client được xử lý bởi một tiến trình backend riêng biệt.
- **Shared Buffers**: bộ đệm dùng chung lưu trữ các page dữ liệu để tăng tốc độ đọc/ghi.
- **Write-Ahead Log (WAL)**: ghi nhật ký các thay đổi để đảm bảo tính an toàn dữ liệu.
- **Background process** gồm các tiến trình như: checkpoint, writer, logger, wal writer, autovacuum, v.v.

---

### 1.2. MVCC – Multi-Version Concurrency Control

Điểm nổi bật của PostgreSQL trong xử lý đồng thời giao dịch là sử dụng **MVCC** để giảm thiểu việc khóa tài nguyên (locking) và tăng hiệu năng.

#### Cơ chế MVCC là gì?

- Mỗi bản ghi trong PostgreSQL không bị cập nhật trực tiếp mà sẽ được tạo ra một **phiên bản mới**.
- Các giao dịch đồng thời có thể thấy các phiên bản dữ liệu khác nhau mà không ảnh hưởng lẫn nhau.
- PostgreSQL sử dụng các **Transaction IDs (XIDs)** và thông tin thời gian để xác định phiên bản phù hợp với từng transaction.

#### Lợi ích của MVCC

- Tránh tình trạng **đóng băng (blocking)** do lock.
- Cho phép đọc không bị chặn bởi ghi, do đó tăng độ đồng thời.
- Giữ lại lịch sử các phiên bản để hỗ trợ rollback và các thao tác khác.

---

#### Ví dụ minh hoạ MVCC:

Giả sử hai transaction cùng truy vấn bảng `accounts`:

- Transaction A bắt đầu trước, đọc tài khoản có số dư là 1000.
- Transaction B thực hiện cập nhật, thêm 100 vào tài khoản đó (phiên bản mới).
- Transaction A vẫn nhìn thấy số dư 1000 (phiên bản cũ).
- Transaction B nhìn thấy số dư 1100 (phiên bản mới).

Điều này giúp tránh việc Transaction A bị block hoặc phải chờ Transaction B commit.

---

#### Cấu trúc dữ liệu hỗ trợ MVCC

Mỗi tuple (bản ghi) trong PostgreSQL kèm theo:

- `xmin`: ID của transaction đã tạo tuple.
- `xmax`: ID của transaction đã xoá hoặc cập nhật tuple.
- Trạng thái tuple dựa trên so sánh với ID transaction hiện hành, xác định tuple có hiển thị với transaction đó không.

---

### 1.3. WAL – Write-Ahead Logging

PostgreSQL đảm bảo tính an toàn dữ liệu thông qua cơ chế **Write-Ahead Logging (WAL)**.

#### WAL hoạt động như thế nào?

- Trước khi dữ liệu thay đổi trên đĩa, những thay đổi này bắt buộc phải được ghi vào tập tin log gọi là **WAL**.
- Log này lưu lại đủ thông tin để có thể phục hồi dữ liệu khi có sự cố.
- Vậy nên dù hệ thống bị crash, logs trong WAL có thể được sử dụng để "phục hồi" các giao dịch chưa được ghi hoàn chỉnh ra đĩa.

---

#### Các bước của WAL:

1. Khi có thay đổi, ghi log vào WAL.
2. Ghi dữ liệu thay đổi vào bộ đệm (shared buffers).
3. Thời điểm checkpoint, dữ liệu từ bộ đệm được đẩy xuống đĩa.
4. Khi khởi động lại sau sự cố, hệ thống sẽ replay các log từ WAL để phục hồi trạng thái dữ liệu.

---

#### Ví dụ mô tả lưu log WAL:

Giả sử chúng ta thực hiện lệnh:

```sql
UPDATE accounts SET balance = balance + 100 WHERE id = 1;
```

- Trước khi ghi data page lên đĩa, PostgreSQL ghi lại chi tiết thay đổi vào log WAL.
- Nếu hệ thống crash, khi phục hồi sẽ dựa vào log này để áp dụng đổi thay.

---

### 1.4. Backup và phục hồi trong PostgreSQL

PostgreSQL hỗ trợ các phương pháp backup:

- **Logical backup**: sử dụng `pg_dump`, xuất toàn bộ dữ liệu dạng SQL hoặc định dạng khác.
- **Physical backup**: copy trực tiếp file dữ liệu và WAL để có thể phục hồi chính xác trạng thái database.

**Khả năng Point-In-Time Recovery (PITR)**:

- Với WAL, PostgreSQL có thể phục hồi database đến trạng thái bất kỳ trong quá khứ (nhờ các log WAL).
- Cơ chế này giúp hệ thống có thể rollback về một thời điểm trước khi có lỗi xảy ra.

---

## 2. Các tính năng quan trọng của PostgreSQL

### 2.1. Hỗ trợ JSON và JSONB

PostgreSQL hỗ trợ lưu trữ và thao tác với dữ liệu JSON một cách mạnh mẽ:

- `JSON`: lưu trữ dưới dạng text, giữ nguyên định dạng.
- `JSONB`: bản nhị phân, hiệu năng cao hơn khi truy vấn.

---

#### Ví dụ:

```sql
CREATE TABLE products (
  id serial PRIMARY KEY,
  data JSONB
);

INSERT INTO products (data) VALUES
('{"name": "Laptop", "specs": {"cpu": "Intel i7", "ram": "16GB"}}'),
('{"name": "Phone", "specs": {"cpu": "Snapdragon 888", "ram": "8GB"}}');

SELECT data->>'name' AS product_name FROM products WHERE data->'specs'->>'ram' = '16GB';
```

Kết quả:

| product_name |
|--------------|
| Laptop       |

---

### 2.2. Hệ thống kiểu dữ liệu phong phú

PostgreSQL hỗ trợ đa dạng kiểu dữ liệu:

- Kiểu chuẩn: `integer`, `text`, `date`, `timestamp`, `boolean`.
- Kiểu mở rộng: `ARRAY`, `UUID`, `XML`, `HSTORE` (key-value store), geometric types.
- Kiểu tùy chỉnh: Người dùng có thể định nghĩa kiểu dữ liệu riêng.

---

### 2.3. Extensibility – Khả năng mở rộng

Một trong những điểm mạnh của PostgreSQL là khả năng mở rộng:

- **Extensions**: có thể cài thêm các gói bổ trợ như `PostGIS` (GIS), `pg_stat_statements` (thống kê), `pg_trgm` (tìm kiếm similarity).
  
  Ví dụ: cài đặt PostGIS để làm việc với dữ liệu địa lý

  ```sql
  CREATE EXTENSION postgis;
  ```

- **Custom functions**: PostgreSQL hỗ trợ viết hàm bằng PL/pgSQL, PL/Python, PL/Perl, PL/Tcl, v.v.
- **Foreign Data Wrapper (FDW)**: kết nối và truy vấn dữ liệu bên ngoài PostgreSQL như file CSV, cơ sở dữ liệu khác.

---

### 2.4. Trigger – Kích hoạt sự kiện

Trigger cho phép tự động thực hiện một hàm khi có sự kiện (INSERT, UPDATE, DELETE) xảy ra trên bảng.

---

#### Ví dụ tạo trigger log sự thay đổi bảng `accounts`:

```sql
CREATE TABLE accounts_log (
  account_id int,
  old_balance numeric,
  new_balance numeric,
  changed_at timestamp DEFAULT now()
);

CREATE OR REPLACE FUNCTION log_balance_change() RETURNS trigger AS $$
BEGIN
  INSERT INTO accounts_log(account_id, old_balance, new_balance)
  VALUES (NEW.id, OLD.balance, NEW.balance);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_log_balance_change
AFTER UPDATE ON accounts
FOR EACH ROW
WHEN (OLD.balance IS DISTINCT FROM NEW.balance)
EXECUTE FUNCTION log_balance_change();
```

Khi thay đổi số dư một tài khoản, trigger sẽ ghi lại lịch sử thay đổi vào bảng `accounts_log`.

---

## 3. Tóm tắt sơ đồ kiến trúc tổng quan (diagram)

```plaintext
+--------------------+
| Client Connections  |
+---------+----------+
          |
+---------v----------+          +----------------------+
|    Postmaster      |<-------->| Background Processes  |
+---------+----------+          +----------------------+
          |
+---------v----------+
| Backend Processes  |  <-- xử lý truy vấn từng client riêng biệt
+---------+----------+  
          |
+---------v----------+
|  Shared Buffers     | <------ dữ liệu đệm cho truy vấn
+---------+----------+
          |
+---------v----------+
|   Data Files on    |
|   Disk (Heap, Index)|
+---------------------+
          |
+---------v----------+
| Write Ahead Log (WAL)|
+---------------------+
```

---

## 4. Kết luận

PostgreSQL với kiến trúc MVCC giúp tăng khả năng xử lý đồng thời hiệu quả, tránh lock giữa các giao dịch; WAL bảo đảm an toàn dữ liệu và khả năng phục hồi đáng tin cậy; cùng với hệ thống kiểu dữ liệu phong phú, hỗ trợ JSON/JSONB, khả năng mở rộng qua extensions và trigger, PostgreSQL là giải pháp cơ sở dữ liệu mạnh mẽ, linh hoạt cho nhiều ứng dụng hiện đại.

---

Nếu bạn cần thêm ví dụ hoặc phần giải thích chi tiết hơn về một phần nào đó, hãy cho tôi biết nhé!