# Tổng Quan Về PostgreSQL và MongoDB: Lịch Sử, Bản Chất, Kiến Trúc và So Sánh

Trong thời đại dữ liệu bùng nổ, việc lựa chọn hệ quản trị cơ sở dữ liệu (Database Management System - DBMS) phù hợp đóng vai trò then chốt trong sự thành công của dự án. Hai trong số những DBMS phổ biến nhất hiện nay là **PostgreSQL** - một hệ quản trị cơ sở dữ liệu quan hệ (RDBMS) và **MongoDB** - một hệ quản trị cơ sở dữ liệu NoSQL thuộc dạng document. Bài viết này sẽ giúp bạn:

- Khám phá lịch sử, bản chất và kiến trúc cơ bản của PostgreSQL và MongoDB.
- So sánh khái quát về mô hình dữ liệu, cách lưu trữ, tính nhất quán và khả năng mở rộng.
- Đưa ra các ví dụ minh họa cụ thể để người đọc dễ hình dung.

---

## 1. Giới thiệu tổng quan

| Tiêu chí           | PostgreSQL                                   | MongoDB                                       |
|--------------------|----------------------------------------------|-----------------------------------------------|
| Loại DBMS          | RDBMS (Quan hệ, có cấu trúc)                 | NoSQL (Document-oriented)                      |
| Ngôn ngữ truy vấn   | SQL tiêu chuẩn                               | MongoDB Query API (dựa trên JSON/BSON)        |
| Kiểu dữ liệu       | Bảng (table) với các cột (columns)          | Document (dạng JSON, BSON)                      |
| Mục đích phát triển | Hệ quản trị cơ sở dữ liệu quan hệ nâng cao  | Quản lý dữ liệu phi cấu trúc, linh hoạt       |
| Phát triển ban đầu  | 1986 bởi UC Berkeley (Ingres → POSTGRES)    | 2007 bởi 10gen (hiện MongoDB Inc.)            |

---

## 2. Lịch sử hình thành và bản chất

### PostgreSQL

- **Lịch sử:**
  - PostgreSQL xuất phát từ dự án POSTGRES tại Đại học California, Berkeley vào năm 1986, do Michael Stonebraker lãnh đạo.
  - Mục tiêu ban đầu là xây dựng DBMS quan hệ thế hệ mới với khả năng mở rộng và hỗ trợ đa tính năng.
  - Trước đó, Stonebraker đã phát triển hệ Ingres, PostgreSQL là sự kế thừa và phát triển thêm các tính năng như hoạt động ACID, hỗ trợ đa kiểu dữ liệu phong phú, trigger, stored procedure...
- **Bản chất:**
  - PostgreSQL là một hệ quản trị cơ sở dữ liệu quan hệ, hỗ trợ mô hình dữ liệu dạng bảng.
  - Tuân thủ chuẩn SQL (Structured Query Language).
  - Có khả năng mở rộng, hỗ trợ các loại kiểu dữ liệu phức tạp (JSON, XML, địa lý GIS).
  - Hỗ trợ giao dịch tuân thủ ACID (Atomicity, Consistency, Isolation, Durability).

### MongoDB

- **Lịch sử:**
  - MongoDB được phát triển năm 2007 bởi công ty 10gen (hiện nay là MongoDB Inc.).
  - Được thiết kế để đáp ứng nhu cầu lưu trữ dữ liệu phi cấu trúc, dữ liệu lớn với khả năng mở rộng theo chiều ngang.
  - MongoDB đi đầu trong lĩnh vực NoSQL, là DBMS dạng document phổ biến nhất hiện nay.
- **Bản chất:**
  - MongoDB lưu trữ và truy vấn dữ liệu dưới dạng *document* (tài liệu) sử dụng format BSON (Binary JSON).
  - Không ràng buộc cấu trúc schema cứng ngắc như SQL, giúp linh hoạt trong phát triển ứng dụng.
  - Tập trung vào khả năng mở rộng theo chiều ngang (sharding), tính sẵn sàng cao.
  - Hỗ trợ tính nhất quán cuối cùng (eventual consistency), có thể tùy chỉnh sao cho phù hợp với ứng dụng.

---

## 3. Kiến trúc cơ bản

### PostgreSQL

- **Kiến trúc client-server:**
  - PostgreSQL hoạt động theo mô hình client-server, trong đó:
    - **Postgres Server**: Chịu trách nhiệm quản lý dữ liệu, thực thi truy vấn, logic kinh doanh...
    - **Client**: Các ứng dụng bên ngoài kết nối qua giao thức TCP/IP hoặc socket để truy vấn.
- **Kiến trúc đa tiến trình (Multiprocess):**
  - Mỗi client kết nối sẽ được server tạo một tiến trình riêng biệt để phục vụ.
- **Bộ nhớ và lưu trữ:**
  - Sử dụng WAL (Write-Ahead Logging) để đảm bảo tính an toàn dữ liệu.
  - Lưu trữ dữ liệu vật lý trên đĩa dạng tập tin.
- **Hỗ trợ giao dịch ACID:**
  - Cơ chế MVCC (Multi-Version Concurrency Control) cho phép đồng thời nhiều phiên bản bản ghi được truy cập.

> **Diagram kiến trúc đơn giản của PostgreSQL:**

```plaintext
+-----------------+    TCP/IP    +---------------------+
|    Client App   | <----------> |    PostgreSQL Server |
|  (psql, app)    |              | (Process per Client) |
+-----------------+              +---------------------+
                                      |
                         +-----------------------------+
                         | Storage (Data files, WAL)   |
                         +-----------------------------+
```

### MongoDB

- **Kiến trúc client-server:**
  - MongoDB chạy dưới dạng một hoặc nhiều server (mongod processes).
  - Client kết nối thông qua MongoDB Driver, gửi truy vấn theo API Mongo.
- **Kiến trúc mô hình document:**
  - Dữ liệu được lưu trữ trong **collections** (tương tự bảng) chứa nhiều **documents** (tài liệu), mỗi document là một cấu trúc JSON/BSON.
- **Sharding và Replica Sets:**
  - Hỗ trợ sharding phân phối dữ liệu theo nhiều server để tăng khả năng mở rộng.
  - Replica sets giúp tăng độ sẵn sàng và độ bền của dữ liệu.
- **Không hỗ trợ giao dịch đa document lâu dài (phiên bản cũ), hiện nay MongoDB cũng hỗ trợ ACID cho multi-document transaction (phiên bản 4.0+).**

> **Diagram kiến trúc MongoDB (Replica Set + Sharding):**

```plaintext
Client Drivers
      |
+-----------------+
|    mongos       |  <-- Router (đối với sharded cluster)
+-----------------+
   |           |
+------+    +------+
|Shard1|    |Shard2|   <-- Shards chứa dữ liệu phân tán
+------+    +------+
   |           |
Replica Sets (primary + secondary nodes)
```

---

## 4. So sánh khái quát

| Tiêu chí             | PostgreSQL                          | MongoDB                          |
|----------------------|-----------------------------------|---------------------------------|
| **Mô hình dữ liệu**    | Quan hệ (bảng, hàng, cột)         | Document (JSON/BSON)             |
| **Schema**             | Cứng, schema cố định              | Linh hoạt, schema động          |
| **Cách lưu trữ dữ liệu** | File hệ thống, phân mảnh theo bảng| Document trong collections       |
| **Ngôn ngữ truy vấn**  | SQL tiêu chuẩn                    | MongoDB Query API (JSON-like)   |
| **Tính nhất quán**     | ACID (đảm bảo mạnh)               | Eventual consistency (mặc định), hỗ trợ ACID multi-document transaction phiên bản 4.0+ |
| **Khả năng mở rộng**   | Mở rộng dọc (tăng cấu hình server)| Mở rộng ngang dễ dàng (sharding)|
| **Giao dịch**          | Đầy đủ, mạnh mẽ                   | Multi-document transaction từ bản 4.0, chủ yếu thiết kế cho tốc độ và mở rộng |
| **Tính năng đặc biệt** | Hỗ trợ truy vấn phức tạp, store procedures, trigger, JSON, GIS | Lưu trữ dữ liệu phi cấu trúc, thay đổi schema linh hoạt, tốt với dữ liệu đa dạng |

---

## 5. Ví dụ minh họa

### 5.1 Ví dụ tạo bảng và truy vấn trong PostgreSQL

```sql
-- Tạo bảng
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Thêm dữ liệu
INSERT INTO users (name, email) VALUES ('Nguyen Van A', 'a@example.com');

-- Truy vấn dữ liệu
SELECT * FROM users WHERE name LIKE 'Nguyen%';
```

### 5.2 Ví dụ lưu trữ và truy vấn document trong MongoDB

```js
// Thêm document vào collection users
db.users.insertOne({
  name: "Nguyen Van A",
  email: "a@example.com",
  created_at: new Date()
});

// Truy vấn document
db.users.find({ name: /^Nguyen/ });
```

---

## 6. Kết luận

| Ưu điểm nổi bật của PostgreSQL  | Ưu điểm nổi bật của MongoDB          |
|---------------------------------|------------------------------------|
| - Hỗ trợ chuẩn SQL, phù hợp với ứng dụng yêu cầu tính nhất quán cao. | - Linh hoạt schema, thích hợp dữ liệu phi cấu trúc, thay đổi nhanh. |
| - Mạnh mẽ, bảo vệ dữ liệu an toàn với ACID. | - Mở rộng theo chiều ngang tốt, thích hợp big data, phân tán.  |
| - Hỗ trợ truy vấn phức tạp, dữ liệu đa dạng (JSON, GIS). | - Triển khai đa vùng và đám mây dễ dàng, khả năng scale-out hiệu quả. |

Tuỳ thuộc vào nhu cầu thực tế (cấu trúc dữ liệu, tính nhất quán, khả năng mở rộng), bạn có thể lựa chọn sử dụng PostgreSQL hoặc MongoDB.

---

Nếu bạn cần triển khai hệ thống quản lý dữ liệu truyền thống, có schema rõ ràng và yêu cầu giao dịch phức tạp, **PostgreSQL** là lựa chọn phù hợp.

Nếu bạn cần hệ thống linh hoạt về schema, lưu trữ dữ liệu đa dạng, cần mở rộng nhanh theo chiều ngang, đặc biệt trong các ứng dụng web, big data, thì **MongoDB** là giải pháp tối ưu.

---

Hy vọng bài viết đã giúp bạn hiểu rõ về PostgreSQL và MongoDB, từ lịch sử phát triển, bản chất, kiến trúc cho đến cách sử dụng và so sánh tổng quát. Nếu bạn muốn tôi hỗ trợ viết đoạn code cụ thể cho ứng dụng hoặc tạo diagram chi tiết hơn, hãy cho tôi biết nhé!