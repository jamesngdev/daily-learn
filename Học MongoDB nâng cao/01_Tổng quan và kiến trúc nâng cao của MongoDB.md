# Tổng Quan Và Kiến Trúc Nâng Cao Của MongoDB

MongoDB là một cơ sở dữ liệu NoSQL dạng document (tài liệu) mã nguồn mở rất phổ biến hiện nay, được thiết kế để xử lý dữ liệu lớn, phi cấu trúc (semi-structured hoặc unstructured) với khả năng mở rộng và hiệu suất cao. Trong bài viết này, ta sẽ đi sâu vào kiến trúc nâng cao của MongoDB, tập trung vào các thành phần quan trọng như:

- Quá trình lưu trữ dữ liệu
- Cơ chế index nâng cao (compound index, text index,...)
- Phân vùng dữ liệu (sharding)
- Cơ chế replication (sao chép dữ liệu)
- Cách MongoDB đảm bảo tính nhất quán dữ liệu
- Các kỹ thuật tối ưu truy vấn

---

## 1. Kiến Trúc Lưu Trữ Dữ Liệu Trong MongoDB

MongoDB lưu trữ dữ liệu dưới dạng **document BSON** (Binary JSON) trong các collection (tập hợp các document tương tự bảng trong SQL).

- **Document (tài liệu)**: Cấu trúc dữ liệu dạng JSON, linh hoạt, có thể chứa các kiểu dữ liệu phức tạp như mảng, đối tượng nhúng.
- **Collection**: Một tập hợp các document, tương tự table trong cơ sở dữ liệu quan hệ.
- **Database**: Một tập hợp các collection.

### Cách MongoDB lưu trữ:

- MongoDB sử dụng **WiredTiger Storage Engine** làm engine mặc định (từ phiên bản 3.2 trở đi).
- Dữ liệu được lưu trữ trên đĩa theo các **data files** có cấu trúc B-Tree cung cấp khả năng tìm kiếm, chèn, cập nhật nhanh.
- Mỗi document được lưu trữ dưới dạng BSON, hỗ trợ nhiều kiểu dữ liệu, tối ưu cho việc serialize/deserialize.

---

## 2. Cơ Chế Index Nâng Cao

MongoDB hỗ trợ nhiều loại index giúp tăng tốc hiệu suất truy vấn:

### a. Simple Index

- Tạo trên một trường (field) cơ bản.
- Ví dụ:

```js
db.users.createIndex({ username: 1 })  // Index tăng dần theo username
```

### b. Compound Index (Chỉ mục đa trường)

- Index được tạo trên nhiều trường trong document.
- Có lợi khi truy vấn thường xuyên sử dụng nhiều điều kiện lọc.
- MongoDB sẽ ưu tiên sử dụng index nếu truy vấn các trường có thứ tự tương ứng với thứ tự tạo index.

**Ví dụ:**

```js
db.orders.createIndex({ customer_id: 1, order_date: -1 })
```

- Phục vụ các truy vấn tìm kiếm theo `customer_id` và sắp xếp `order_date` giảm dần.

### c. Text Index

- Hỗ trợ tìm kiếm văn bản trong các chuỗi.
- Cho phép tìm kiếm với truy vấn kiểu full-text search như: tìm kiếm từ khóa, cụm từ có chứa trong trường.

**Ví dụ:**

```js
db.articles.createIndex({ content: "text", title: "text" })
```

- Tìm kiếm với truy vấn:

```js
db.articles.find({ $text: { $search: "MongoDB tutorial" } })
```

### d. Các loại index khác

- **Geospatial index**: xử lý dữ liệu địa lý.
- **Hashed index**: hỗ trợ sharding dựa trên hash field.

---

## 3. Phân Vùng Dữ Liệu (Sharding)

Khi lượng dữ liệu quá lớn hoặc yêu cầu đáp ứng tải truy vấn cao, MongoDB sử dụng **sharding** để phân tán dữ liệu trên nhiều máy chủ (shard).

### Kiến trúc sharding gồm:

- **Shard**: Là một replica set chứa một phần dữ liệu (chunk).
- **Config Server**: Lưu trữ metadata về cluster sharding, quản lý các chunk phân bổ ra sao.
- **Query Router (mongos)**: Trung gian nhận truy vấn, phân bổ truy vấn đúng shard.

### Cơ sở để phân tán dữ liệu:

- **Shard key**: Trường hoặc tổ hợp các trường được chỉ định để phân chia dữ liệu thành chunks phân phối trên các shards.
- MongoDB hỗ trợ các chính sách phân vùng:
  - **Ranged Sharding**: Phân vùng dữ liệu dựa trên ranges của shard key.
  - **Hashed Sharding**: Phân vùng dựa trên hash của giá trị shard key (đảm bảo phân phối đồng đều).

### Ví dụ thiết lập sharding (rất đơn giản):

```js
sh.enableSharding("salesDB")                                   // Bật sharding trên database
sh.shardCollection("salesDB.orders", { order_id: "hashed" })  // Shard theo hash của order_id
```

---

## 4. Replication (Nhân bản dữ liệu)

MongoDB bảo đảm tính sẵn sàng và khôi phục dữ liệu nhờ cơ chế replication với mô hình **Replica Set**.

- Một replica set gồm nhiều node (1 primary + nhiều secondary).
- **Primary** nhận yêu cầu ghi và phát tán thay đổi sang các secondary.
- Các **Secondary** sẽ copy dữ liệu từ primary (gọi là replication).
- Khi primary gặp sự cố, một secondary sẽ được bầu làm primary mới để đảm bảo tính sẵn sàng.

### Ví dụ cấu hình replica set:

```js
rs.initiate(
  {
    _id: "rs0",
    members: [
      { _id: 0, host: "mongodb0.example.net:27017" },
      { _id: 1, host: "mongodb1.example.net:27017" },
      { _id: 2, host: "mongodb2.example.net:27017" }
    ]
  }
)
```

---

## 5. Cách MongoDB Đảm Bảo Tính Nhất Quán Dữ Liệu

MongoDB tuân thủ mô hình **eventual consistency** mặc định trong môi trường replica set, tuy nhiên cũng hỗ trợ:

- **Write Concern**: Xác định yêu cầu về việc ghi thành công trên bao nhiêu node.
- **Read Concern**: Xác định mức độ độ nhất quán của kết quả truy vấn.

### Ví dụ write với yêu cầu ghi lên nhiều node:

```js
db.orders.insertOne(
  { order_id: 123, item: "Laptop", qty: 1 },
  { writeConcern: { w: "majority", wtimeout: 5000 } }
)
```

- Đảm bảo dữ liệu được ghi thành công trên đa số node trong replica set.

### Tính nhất quán trong sharded cluster:

- MongoDB dùng **two-phase commit** cho các giao dịch đa tài liệu trên nhiều shards từ phiên bản 4.0 trở lên để đảm bảo atomicity.
- MongoDB hỗ trợ **multi-document transactions**, tương tự như RDBMS.

---

## 6. Các Kỹ Thuật Tối Ưu Truy Vấn

- **Sử dụng Index hợp lý**: tạo index đúng trường truy vấn, tránh full collection scan.
- **Tận dụng compound index**: đặc biệt với các truy vấn nhiều điều kiện.
- **Projection Field**: chỉ chọn các trường cần thiết trong kết quả truy vấn giúp giảm IO.
- **Explain Plan**: sử dụng `db.collection.explain()` để phân tích cách MongoDB thực hiện truy vấn và cải tiến index.

### Ví dụ phân tích truy vấn:

```js
db.orders.find({ customer_id: 100, status: "pending" })
  .explain("executionStats")
```

Output sẽ cho biết việc sử dụng index, số lượng document scan,...

---

## Tóm Tắt

| Thành phần           | Chức năng chính                                                                         |
|---------------------|-----------------------------------------------------------------------------------------|
| **BSON Document**    | Lưu trữ dữ liệu phi cấu trúc, linh hoạt                                               |
| **WiredTiger**       | Storage engine mạnh mẽ, hỗ trợ compression, concurrency cao                            |
| **Index**            | Compound index tăng tốc truy vấn đa điều kiện, text index cho tìm kiếm văn bản         |
| **Sharding**          | Phân vùng dữ liệu theo shard key, mở rộng dữ liệu ngang bằng nhiều node               |
| **Replication**      | Replica set với primary-secondary, bảo đảm sẵn sàng, chịu lỗi cao                     |
| **Write/Read Concern** | Điều chỉnh mức độ nhất quán và độ bền của dữ liệu                                    |
| **Transaction**      | Giao dịch đa document trên nhiều shard                                                |

---

## Diagram: Kiến trúc MongoDB Sharding và Replication

```plaintext
+------------------+
|     Mongos       |  <---- Query Router, điều phối truy vấn đến shards
+--------+---------+
         |
   +-----+--------------+      +------------------+
   |   Shard 1          |      |   Shard 2         |
   |  (Replica Set)     |      |  (Replica Set)    |
   |  Primary           |      |  Primary          |
   |  Secondary(s)      |      |  Secondary(s)     |
   +--------------------+      +------------------+

   +--------------------+
   |  Config Servers     |  <-- Lưu trữ metadata và quản lý sharding
   +--------------------+
```

---

Đây là toàn cảnh kiến trúc nâng cao MongoDB, hy vọng giúp bạn có cái nhìn sâu sắc hơn về cách MongoDB xử lý dữ liệu lớn, đảm bảo hiệu suất và tính nhất quán trong môi trường phân tán.

Nếu bạn cần ví dụ code hoặc giải thích chi tiết hơn về phần nào, vui lòng phản hồi nhé!