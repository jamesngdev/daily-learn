# Thực hành cơ bản với PostgreSQL và MongoDB  
_Hướng dẫn cài đặt, cấu hình cơ bản, tạo database, bảng/collection, và thao tác CRUD (Create, Read, Update, Delete) với ví dụ cụ thể. So sánh cách viết truy vấn và thao tác dữ liệu giữa hai hệ thống._

---

## 1. Giới thiệu ngắn gọn

- **PostgreSQL**: Hệ quản trị cơ sở dữ liệu quan hệ (RDBMS) mạnh mẽ, hỗ trợ SQL chuẩn, ACID, nhiều tính năng nâng cao.
- **MongoDB**: Cơ sở dữ liệu NoSQL dạng document, lưu trữ dữ liệu dưới dạng JSON-like (BSON), linh hoạt schema và mở rộng dễ dàng.

---

## 2. Hướng dẫn cài đặt và cấu hình cơ bản

### 2.1 Cài đặt PostgreSQL

- Trang chủ: https://www.postgresql.org/download/

#### Với Ubuntu (Debian-based):

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

#### Kiểm tra trạng thái dịch vụ:

```bash
sudo systemctl status postgresql
```

#### Đăng nhập PostgreSQL shell (psql):

```bash
sudo -u postgres psql
```

---

### 2.2 Cài đặt MongoDB

- Trang chủ: https://www.mongodb.com/try/download/community

#### Với Ubuntu (ví dụ MongoDB 6.0):

```bash
curl -fsSL https://pgp.mongodb.com/server-6.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg --dearmor

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg ] \
  https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | \
  sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

sudo apt update
sudo apt install -y mongodb-org

sudo systemctl start mongod
sudo systemctl enable mongod
```

#### Kiểm tra:

```bash
mongosh
```

---

## 3. Cấu hình cơ bản & tạo database

### 3.1 PostgreSQL

Mặc định sau cài đặt, PostgreSQL tạo user `postgres`.

#### Tạo database mới:

```sql
CREATE DATABASE testdb;
```

#### Tạo user mới (optional):

```sql
CREATE USER testuser WITH PASSWORD '123456';
GRANT ALL PRIVILEGES ON DATABASE testdb TO testuser;
```

---

### 3.2 MongoDB

MongoDB tự động tạo database khi bạn thao tác với nó. Không cần câu lệnh tạo database riêng biệt.

Ví dụ trong `mongosh`:

```javascript
use testdb;  // chuyển sang database testdb, nếu không tồn tại nó sẽ được tạo khi bạn thêm dữ liệu
```

---

## 4. Thực hành tạo bảng/collection và thao tác CRUD

---

### 4.1 Trong PostgreSQL

#### 4.1.1 Tạo bảng

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    age INTEGER,
    email VARCHAR(100) UNIQUE
);
```

#### 4.1.2 Thao tác CRUD

- **Create (Insert):**

```sql
INSERT INTO users (name, age, email)
VALUES ('Nguyen Van A', 30, 'a.nguyen@example.com');
```

- **Read (Select):**

```sql
SELECT * FROM users WHERE age > 25;
```

- **Update:**

```sql
UPDATE users SET age = 31 WHERE name = 'Nguyen Van A';
```

- **Delete:**

```sql
DELETE FROM users WHERE name = 'Nguyen Van A';
```

---

### 4.2 Trong MongoDB

#### 4.2.1 Tạo collection và tài liệu (document)

Trong `mongosh`:

```javascript
use testdb;
db.createCollection("users");
```

(Thực tế `createCollection` không bắt buộc, collection được tạo khi bạn insert document đầu tiên)

#### 4.2.2 Thao tác CRUD

- **Create (Insert):**

```javascript
db.users.insertOne({
  name: "Nguyen Van A",
  age: 30,
  email: "a.nguyen@example.com"
});
```

- **Read (Find):**

```javascript
db.users.find({ age: { $gt: 25 } }).pretty();
```

- **Update:**

```javascript
db.users.updateOne(
  { name: "Nguyen Van A" },
  { $set: { age: 31 } }
);
```

- **Delete:**

```javascript
db.users.deleteOne({ name: "Nguyen Van A" });
```

---

## 5. So sánh cách viết truy vấn và thao tác dữ liệu

| **Khía cạnh**              | **PostgreSQL**                  | **MongoDB**                                |
|---------------------------|--------------------------------|--------------------------------------------|
| Kiểu dữ liệu              | Quan hệ (bảng, dòng, cột)       | Document (dữ liệu JSON-like)                |
| Ngôn ngữ truy vấn         | SQL chuẩn                      | MongoDB Query Language (JavaScript-like)   |
| Tạo DB và bảng           | Cần tạo rõ ràng                | Tạo tự động khi insert                      |
| Câu lệnh tạo bảng/collection | `CREATE TABLE`                 | Tùy chọn, collection tạo khi insert document |
| Thao tác CRUD            | Câu lệnh SQL: `INSERT`, `SELECT`, `UPDATE`, `DELETE` | Các phương thức: `insertOne()`, `find()`, `updateOne()`, `deleteOne()` |
| Truy vấn tìm kiếm phức tạp | SQL với JOIN, GROUP BY, HAVING, v.v... | Aggregation pipeline hoặc truy vấn nâng cao khác |
| Schema                   | Cố định, định nghĩa trước     | Linh hoạt, không ràng buộc schema          |

---

## 6. Tóm tắt demo với cùng dữ liệu

### PostgreSQL - SQL

```sql
-- Tạo database testdb
CREATE DATABASE testdb;

-- Kết nối vào testdb
\c testdb

-- Tạo bảng users
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    age INTEGER,
    email VARCHAR(100) UNIQUE
);

-- Thêm 1 bản ghi
INSERT INTO users (name, age, email)
VALUES ('Nguyen Van A', 30, 'a.nguyen@example.com');

-- Lấy người dùng tuổi trên 25
SELECT * FROM users WHERE age > 25;

-- Cập nhật tuổi
UPDATE users SET age = 31 WHERE name = 'Nguyen Van A';

-- Xóa bản ghi
DELETE FROM users WHERE name = 'Nguyen Van A';
```

---

### MongoDB - JavaScript (mongosh)

```javascript
// Chuyển hoặc tạo database testdb
use testdb;

// Tạo collection users (tùy chọn)
db.createCollection('users');

// Thêm 1 document
db.users.insertOne({
  name: 'Nguyen Van A',
  age: 30,
  email: 'a.nguyen@example.com'
});

// Tìm document có age > 25
db.users.find({ age: { $gt: 25 } }).pretty();

// Cập nhật age
db.users.updateOne(
  { name: 'Nguyen Van A' },
  { $set: { age: 31 } }
);

// Xóa document
db.users.deleteOne({ name: 'Nguyen Van A' });
```

---

## 7. Kết luận

- **PostgreSQL** phù hợp cho các ứng dụng cần cấu trúc dữ liệu rõ ràng, tuân thủ quy tắc quan hệ, phức tạp, đa dạng truy vấn như JOIN, transaction phức tạp.
- **MongoDB** nổi bật với tính linh hoạt cao, schema tự do, dễ dàng mở rộng theo chiều ngang, thích hợp với dữ liệu phi cấu trúc hoặc nửa cấu trúc, các ứng dụng thời gian thực.

---

## 8. Tài nguyên tham khảo

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [MongoDB Manual](https://docs.mongodb.com/manual/)
- Các khóa học trực tuyến như [freeCodeCamp](https://www.freecodecamp.org), [MongoDB University](https://university.mongodb.com/)

---

Nếu bạn cần mình hướng dẫn thêm về các câu truy vấn nâng cao, index, tối ưu hiệu năng, hoặc kết nối từ các ngôn ngữ lập trình, hãy cho mình biết nhé!