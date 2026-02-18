# Tìm hiểu chi tiết các kiểu dữ liệu, chỉ mục và tối ưu truy vấn trong PostgreSQL

PostgreSQL là một hệ quản trị cơ sở dữ liệu quan hệ rất mạnh mẽ với khả năng mở rộng cao và hỗ trợ nhiều kiểu dữ liệu tiên tiến cùng các công cụ tối ưu truy vấn hiệu quả. Bài viết dưới đây sẽ tập trung giúp bạn hiểu rõ về:

- Các kiểu dữ liệu phổ biến và nâng cao trong PostgreSQL
- Các loại chỉ mục thường dùng và cách lựa chọn chỉ mục phù hợp
- Cách tối ưu truy vấn bằng EXPLAIN và các phương pháp viết câu lệnh hiệu quả

---

## 1. Các kiểu dữ liệu trong PostgreSQL

### 1.1 Kiểu số (Numeric types)

PostgreSQL hỗ trợ đa dạng các kiểu số, phù hợp với từng mục đích:

| Kiểu dữ liệu | Mô tả                        | Ví dụ sử dụng                  |
|--------------|-----------------------------|-------------------------------|
| `smallint`   | Số nguyên 2 byte (-32768 đến 32767)  | `age smallint = 25`            |
| `integer`    | Số nguyên 4 byte (-2,147,483,648 đến 2,147,483,647) | `id integer = 123456`          |
| `bigint`     | Số nguyên 8 byte             | `views bigint = 123456789012` |
| `numeric`    | Số thập phân chính xác, định nghĩa độ chính xác và quy mô | `price numeric(10, 2) = 99.99`|
| `real`       | Số thực dấu phẩy động 4 byte (float4)    | `score real = 3.14`             |
| `double precision` | Số thực dấu phẩy động 8 byte (float8) | `balance double precision = 12345.6789`|

**Ví dụ tạo bảng với kiểu số:**
```sql
CREATE TABLE products (
  id serial PRIMARY KEY,
  price numeric(10,2),
  quantity integer
);
```

---

### 1.2 Kiểu chuỗi (String types)

| Kiểu dữ liệu | Mô tả                         | Ví dụ sử dụng                     |
|--------------|-------------------------------|----------------------------------|
| `char(n)`    | Chuỗi kí tự cố định độ dài n  | `code char(3) = 'ABC'`           |
| `varchar(n)` | Chuỗi kí tự có độ dài tối đa n | `name varchar(50) = 'Postgres'`  |
| `text`       | Chuỗi kí tự không giới hạn     | `description text`                |

**Lưu ý:**  
- `varchar` giới hạn độ dài và thường được sử dụng khi cần kiểm soát kích thước.  
- `text` không giới hạn nhưng hiệu năng tương đương với `varchar` không giới hạn.

---

### 1.3 Kiểu ngày tháng (Date/time types)

PostgreSQL hỗ trợ các kiểu thời gian mạnh mẽ:

| Kiểu dữ liệu        | Mô tả                         | Ví dụ sử dụng                             |
|---------------------|-------------------------------|------------------------------------------|
| `date`              | Ngày (năm-tháng-ngày)          | `birthdate date = '1990-01-01'`          |
| `timestamp`         | Ngày giờ (không có múi giờ)    | `created_at timestamp = '2024-06-01 12:30:00'`  |
| `timestamp with time zone (timestamptz)` | Ngày giờ có múi giờ           | `updated_at timestamptz = now()`         |
| `time`              | Giờ, phút, giây (không ngày)   | `start_time time = '08:30:00'`            |
| `interval`          | Khoảng thời gian               | `duration interval = '1 day 2 hours'`    |

Ví dụ:

```sql
CREATE TABLE events (
  id serial PRIMARY KEY,
  event_date date,
  event_timestamp timestamptz DEFAULT now()
);
```

---

### 1.4 Kiểu Boolean

Chỉ nhận hai giá trị: `TRUE` hoặc `FALSE`.

```sql
CREATE TABLE users (
  id serial PRIMARY KEY,
  is_active boolean DEFAULT TRUE
);
```

---

### 1.5 Kiểu nâng cao trong PostgreSQL

#### 1.5.1 Kiểu JSON và JSONB

- `json`: lưu trữ dữ liệu JSON dạng text, không được tối ưu cho truy vấn.
- `jsonb`: nhị phân hóa JSON, tối ưu hơn cho các truy vấn, lọc, index.

```sql
CREATE TABLE documents (
  id serial,
  data jsonb
);

INSERT INTO documents(data) VALUES ('{"name": "Alice", "age": 30}');
```

Truy vấn ví dụ trích lọc trường:

```sql
SELECT data->>'name' AS name FROM documents WHERE data->>'age' = '30';
```

#### 1.5.2 Kiểu mảng (Array)

PostgreSQL hỗ trợ lưu mảng trực tiếp trong cột, rất tiện cho các trường hợp cần lưu danh sách.

```sql
CREATE TABLE employees (
  id serial,
  skills text[]
);

INSERT INTO employees(skills) VALUES (ARRAY['Python', 'SQL', 'Docker']);
```

Truy vấn kiểm tra một phần tử có trong mảng:

```sql
SELECT * FROM employees WHERE 'Python' = ANY(skills);
```

#### 1.5.3 Kiểu Hstore (Key-Value store)

Hstore lưu trữ dữ liệu dạng cặp key-value, tiện lợi để lưu dữ liệu không theo cấu trúc cố định.

Cần kích hoạt extension:

```sql
CREATE EXTENSION IF NOT EXISTS hstore;
```

Tạo bảng:

```sql
CREATE TABLE products (
  id serial,
  attributes hstore
);

INSERT INTO products(attributes) VALUES ('color => red, size => M');
```

Truy vấn kiểm tra key:

```sql
SELECT * FROM products WHERE attributes -> 'color' = 'red';
```

#### 1.5.4 Kiểu UUID

Dùng để tạo các mã định danh duy nhất, không phụ thuộc vào số thứ tự.

Cần kích hoạt extension:

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

Tạo bảng:

```sql
CREATE TABLE sessions (
  id uuid DEFAULT uuid_generate_v4(),
  user_id integer,
  created_at timestamptz DEFAULT now()
);
```

---

## 2. Các kiểu chỉ mục trong PostgreSQL

Chỉ mục giúp tăng tốc truy vấn bằng cách giảm số lượng bản ghi cần scan.

### 2.1 B-tree (Balanced Tree)

- Là loại chỉ mục mặc định.
- Hỗ trợ tốt cho các loại so sánh `=`, `>`, `<`, `>=`, `<=`.
- Chủ yếu dùng cho các kiểu dữ liệu như số, chuỗi, ngày tháng, boolean.

Tạo chỉ mục:

```sql
CREATE INDEX idx_users_name ON users (name);
```

### 2.2 Hash index

- Hỗ trợ tốt cho phép so sánh `=`.
- Ít được dùng vì có nhiều hạn chế hơn B-tree.
- Được cải tiến trong các phiên bản PostgreSQL mới.

```sql
CREATE INDEX idx_users_email_hash ON users USING hash (email);
```

### 2.3 GIN (Generalized Inverted Index)

- Dùng cho các kiểu dữ liệu có thể chứa nhiều giá trị, như `array`, `jsonb`, `hstore`.
- Rất hiệu quả cho các truy vấn chứa phép toán kiểm tra phần tử có trong.

Ví dụ chỉ mục GIN cho cột jsonb:

```sql
CREATE INDEX idx_documents_data ON documents USING gin (data);
```

### 2.4 GiST (Generalized Search Tree)

- Dùng cho các cấu trúc dữ liệu phức tạp như dữ liệu địa lý (`PostGIS`), hoặc các truy vấn tìm kiếm gần đúng.
- Hữu ích cho các loại dữ liệu như `geometry`, `box`, `circle`.

---

### 2.5 Lựa chọn chỉ mục phù hợp

| Loại chỉ mục | Khi nào sử dụng                                   |
|--------------|--------------------------------------------------|
| B-tree       | Truy vấn so sánh (=, >, <, ...) trên kiểu dữ liệu chuẩn (int, text, date, bool) |
| Hash         | Truy vấn so sánh bằng (=) không yêu cầu sắp xếp |
| GIN          | Tìm kiếm trên dữ liệu chứa nhiều giá trị: jsonb, arrays, hstore |
| GiST         | Dữ liệu phức tạp hay không gian (GIS)            |

---

## 3. Tối ưu truy vấn trong PostgreSQL

### 3.1 Sử dụng EXPLAIN để phân tích câu truy vấn

Câu lệnh `EXPLAIN` để xem kế hoạch thực thi câu truy vấn (query plan). Kế hoạch này cho biết PostgreSQL sẽ lấy dữ liệu như thế nào (quét bảng, sử dụng chỉ mục, join,...)

Ví dụ:

```sql
EXPLAIN SELECT * FROM products WHERE price > 100;
```

Kết quả sẽ cho biết có dùng chỉ mục hay scan toàn bộ bảng.

Để xem chi tiết hơn:

```sql
EXPLAIN ANALYZE SELECT * FROM products WHERE price > 100;
```

`ANALYZE` chạy câu truy vấn thật và trả về thời gian, chi phí thực tế.

---

### 3.2 Một số mẹo tối ưu truy vấn

- **Dùng chỉ mục đúng cách:** áp dụng chỉ mục cho các cột thường dùng trong điều kiện `WHERE`, `JOIN` hoặc để sắp xếp `ORDER BY`.
- **Tránh `SELECT *`:** Chỉ lấy những cột cần thiết để giảm tài nguyên mạng và bộ nhớ.
- **Sử dụng phân trang (LIMIT OFFSET):** Khi lấy dữ liệu lớn.
- **Tránh truy vấn lồng nhau phức tạp:** Dùng `JOIN`, `WITH (CTE)` hiệu quả hơn.
- **Cẩn thận với LIKE và biểu thức chính quy:** Có thể thiếu chỉ mục và chậm.
- **Dùng `EXPLAIN ANALYZE` để kiểm tra chi phí thực thi.**

---

### 3.3 Ví dụ tối ưu

Bảng `orders` có nhiều bản ghi, bạn muốn tìm các đơn hàng có trạng thái `"completed"` và giá trị `total > 100`.

```sql
CREATE INDEX idx_orders_status_total ON orders (status, total);
```

Truy vấn:

```sql
SELECT id, total FROM orders WHERE status = 'completed' AND total > 100 ORDER BY total DESC LIMIT 10;
```

Dùng `EXPLAIN ANALYZE` để xem truy vấn dùng chỉ mục như mong muốn.

---

## Tổng kết

| Chủ đề                   | Gợi ý học tập                                                             |
|--------------------------|---------------------------------------------------------------------------|
| **Kiểu dữ liệu**          | Hiểu các kiểu số, chuỗi, ngày tháng, boolean; khám phá kiểu nâng cao JSONB, arrays, hstore, UUID |
| **Kiểu chỉ mục**           | Biết phân biệt B-tree, Hash, GIN, GiST; chọn chỉ mục phù hợp theo kiểu dữ liệu và truy vấn |
| **Tối ưu truy vấn**        | Sử dụng EXPLAIN để phân tích; tạo chỉ mục phù hợp; viết truy vấn lọc hiệu quả, tránh truy vấn thừa thãi |

---

Hy vọng bài viết sẽ giúp bạn nắm bắt kỹ năng làm việc với dữ liệu, chỉ mục và tối ưu truy vấn hiệu quả trên PostgreSQL! Nếu bạn cần, tôi có thể cung cấp thêm các ví dụ nâng cao hoặc các bài tập thực hành!