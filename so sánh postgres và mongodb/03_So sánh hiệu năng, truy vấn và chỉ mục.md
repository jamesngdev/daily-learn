# So sánh tối ưu truy vấn, chỉ mục trong PostgreSQL và MongoDB: Hiệu năng, join, aggregation, transaction

Trong bài viết này, chúng ta sẽ cùng nhau phân tích, so sánh cách tối ưu truy vấn cũng như chỉ mục (index) trong **PostgreSQL** - hệ quản trị cơ sở dữ liệu quan hệ (RDBMS) truyền thống và **MongoDB** - hệ quản trị cơ sở dữ liệu NoSQL dạng document. Từ đó đánh giá hiệu năng khi thực hiện các truy vấn phức tạp, xử lý join trong Postgres so với flatten và aggregation pipeline trong MongoDB, cũng như phân tích khả năng truy vấn đa chiều và transaction của từng hệ thống.

---

## 1. Cơ sở dữ liệu và quản lý chỉ mục (Indexing)

### PostgreSQL

- **Loại index phổ biến**: B-tree, Hash, GiST, GIN, BRIN.
- **Cơ chế chỉ mục**: Chỉ mục B-tree rất hiệu quả với truy vấn range, tìm kiếm giá trị cụ thể, sorting.
- **Chỉ mục đa cột**: PostgreSQL hỗ trợ tạo chỉ mục trên nhiều cột để tối ưu các truy vấn đa điều kiện (multi-column index).
- **Partial index**: Chỉ tạo index trên một phần dữ liệu (vd: WHERE status = 'active'), giúp tiết kiệm không gian.
- **Expression index**: Tạo index trên kết quả biểu thức, ví dụ LOWER(column_name).

**Ví dụ tạo chỉ mục đa cột:**
```sql
CREATE INDEX idx_user_email_status ON users (email, status);
```

---

### MongoDB

- **Loại index phổ biến**: Single field, compound, multi-key, text, hashed, geospatial.
- **Multi-key index**: Cho phép index trên mảng, hỗ trợ truy vấn các document có trường giá trị là mảng.
- **Index trên document nested**: Có thể tạo index trên trường con trong document nested.
- **Partial index và TTL index**: Partial index tương tự PostgreSQL, TTL index dùng để tự động xóa document sau khoảng thời gian định sẵn.

**Ví dụ tạo chỉ mục compound với trường nested:**
```js
db.orders.createIndex({ "customer.name": 1, "items.productId": 1 });
```

---

## 2. So sánh tối ưu truy vấn phức tạp và join/aggregation

### 2.1. Join trong PostgreSQL

PostgreSQL hỗ trợ cơ chế join phong phú (INNER, LEFT, RIGHT, FULL, CROSS JOIN), quan trọng trong hệ quan hệ và cho phép kết nối dữ liệu từ nhiều bảng.

- **Tối ưu join**: Sử dụng chỉ mục (đặc biệt index trên khóa ngoại), thống kê thống kê cập nhật (ANALYZE), phân tích EXPLAIN để tối ưu.
- **Truy vấn phức tạp**: PostgreSQL xử lý tốt các subqueries, CTE (WITH), window functions.

**Ví dụ truy vấn join:**
```sql
SELECT u.id, u.name, o.order_date, o.total
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed';
```
**Tối ưu bằng index:**
```sql
CREATE INDEX idx_orders_user_status ON orders (user_id, status);
```

> PostgreSQL sử dụng **hash join**, **merge join** hoặc **nested loop join** tùy vào trường hợp để tối ưu hiệu năng.

---

### 2.2. Aggregation pipeline trong MongoDB

MongoDB không hỗ trợ join truyền thống (trước version 3.2), thay vào đó dùng:

- **Aggregation pipeline**: Chuỗi các stages như `$match`, `$group`, `$lookup` (kể từ v3.2, `$lookup` cho phép join dữ liệu giống RDBMS), `$project`, `$unwind` để xử lý truy vấn phức tạp, flatten document.

**Ví dụ join (lookup) trong aggregation pipeline:**
```js
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "user_id",
      foreignField: "_id",
      as: "user_info"
    }
  },
  { $unwind: "$user_info" },
  { $match: { status: "completed" } },
  {
    $project: {
      _id: 1,
      total: 1,
      "user_info.name": 1
    }
  }
]);
```

- **Flattening**: `$unwind` biến các mảng thành nhiều bản ghi.
- **Nhược điểm**: `$lookup` có thể gây chậm nếu dữ liệu lớn, không hiệu quả bằng join trong SQL khi quan hệ phức tạp và lớn.

---

## 3. Hiệu năng truy vấn đa chiều và tối ưu trong thực tế

| Tiêu chí                        | PostgreSQL                                         | MongoDB                                               |
|-------------------------------|---------------------------------------------------|------------------------------------------------------|
| Mô hình dữ liệu                | Quan hệ, schema chặt chẽ                           | Document-oriented, schema linh hoạt                  |
| Tối ưu truy vấn phức tạp       | Tốt với join, subqueries, window functions        | Aggregation pipeline mạnh, hỗ trợ join bằng `$lookup` |
| Chỉ mục đa chiều               | Multi-column index, expression index              | Compound index, multi-key index                       |
| Truy vấn dữ liệu nested        | Phức tạp, cần dùng JSONB + GiST / GIN index       | Tự nhiên, dễ dàng truy vấn nested                    |
| Truy vấn phân tán             | Cần extension như CitusDB                         | Hỗ trợ phân tán tự nhiên bằng sharding              |
| Hiệu năng join đa bảng lớn    | Hiệu quả, đặc biệt với chỉ mục phù hợp            | Kém hơn khi join lớn, cần denormalization            |

---

## 4. Transaction và xử lý mượt mà

### PostgreSQL

- Hỗ trợ **ACID hoàn chỉnh**.
- Transaction mạnh mẽ với khả năng rollback, isolation levels.
- Multi-statement transaction dễ dàng và hiệu quả, dùng MVCC để tránh lock lâu.
- Thích hợp ứng dụng cần **độ nhất quán mạnh**, ví dụ tài chính, đặt hàng.

**Ví dụ transaction:**
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

---

### MongoDB

- Trước đây chỉ hỗ trợ transaction ở mức document (atomic trên 1 document).
- Từ phiên bản 4.0 có hỗ trợ **multi-document ACID transaction** trong replica set, 4.2 mở rộng cho sharded cluster.
- Transaction thường nặng hơn Postgres do bản chất NoSQL, nên khuyên dùng cho workload cần tính nhất quán cao.
- Thường ưu tiên denormalization hoặc thiết kế schema để giảm usage transaction.

**Ví dụ transaction multi-document:**
```js
const session = client.startSession();
try {
  session.startTransaction();
  await db.collection('accounts').updateOne({ _id: 1 }, { $inc: { balance: -100 } }, { session });
  await db.collection('accounts').updateOne({ _id: 2 }, { $inc: { balance: 100 } }, { session });
  await session.commitTransaction();
} catch (e) {
  await session.abortTransaction();
} finally {
  await session.endSession();
}
```

---

## 5. Kết luận và lời khuyên

| Đặc điểm                      | Khi dùng PostgreSQL                                         | Khi dùng MongoDB                                          |
|------------------------------|------------------------------------------------------------|----------------------------------------------------------|
| **Tối ưu truy vấn & chỉ mục** | Sử dụng multi-column index, thống kê, phân tích EXPLAIN    | Dùng compound, multi-key index, index trường nested     |
| **Truy vấn phức tạp, join**   | Tối ưu, nhanh, join đa bảng phức tạp                      | Aggregation pipeline, `$lookup` tốt cho join đơn giản   |
| **Xử lý transaction**          | ACID mạnh, lock control, nhanh                            | Hỗ trợ transactional từ 4.0, nặng hơn, dùng ít hơn      |
| **Schema và dữ liệu nested**   | Hỗ trợ JSONB nhưng không mạnh về nested                   | Thiết kế document dễ dàng, query nested tốt hơn          |
| **Quy mô mở rộng**             | Vertical scaling tốt, horizontal cần extension            | Hỗ trợ sharding native, scale out dễ hơn                 |

---

# Tóm tắt:

- PostgreSQL thích hợp cho ứng dụng cần dữ liệu liên quan, quan hệ phức tạp, transaction mượt mà và hiệu năng join tốt.
- MongoDB phù hợp với dữ liệu linh hoạt, document nested, truy vấn aggregation pipeline, và mở rộng quy mô cao.
- Tối ưu chỉ mục trong PostgreSQL chú trọng multi-column, partial, expression index. MongoDB tập trung compound, multi-key và index trên trường nested.
- Khi xây dựng hệ thống cần đánh giá nhu cầu dữ liệu, tính liên quan, và tập trung vào thiết kế schema phù hợp để tối ưu hiệu năng.

---

**Hy vọng bài phân tích này giúp bạn hiểu rõ hơn về cách tối ưu truy vấn và chỉ mục cũng như đánh giá hiệu năng giữa PostgreSQL và MongoDB trong các trường hợp dữ liệu và truy vấn thực tế!**