# Ứng dụng thực tế của từng hệ quản trị cơ sở dữ liệu (DBMS) qua các case study

Trong phát triển phần mềm hiện đại, việc lựa chọn hệ quản trị cơ sở dữ liệu phù hợp với loại ứng dụng, tính chất dữ liệu và yêu cầu nghiệp vụ là hết sức quan trọng. Mỗi loại DBMS có những thế mạnh riêng, phù hợp với những bài toán và trường hợp sử dụng cụ thể.

Bài viết này sẽ tập trung phân tích các **case study thực tế** của hai hệ quản trị dữ liệu phổ biến và đại diện cho hai “hướng tiếp cận” khác nhau:

- **PostgreSQL (Postgres):** hệ quản trị quan hệ (relational) mạnh mẽ, hỗ trợ nghiệp vụ transaction, độ nhất quán cao.
- **MongoDB:** hệ cơ sở dữ liệu NoSQL dạng document, phù hợp với dữ liệu phi cấu trúc, schema linh động.

---

## 1. PostgreSQL: Ứng dụng trong các hệ transactional, tài chính, ERP, báo cáo phân tích

### **Tính chất nổi bật của Postgres:**

- Tuân thủ chặt chẽ ACID (Atomicity, Consistency, Isolation, Durability), đảm bảo độ nhất quán cao trong giao dịch.
- Hỗ trợ truy vấn phức tạp với ngôn ngữ SQL chuẩn.
- Các tính năng nâng cao: stored procedures, triggers, views, partitioning, indexing, hỗ trợ JSON.
- Mở rộng tốt với các loại dữ liệu phức tạp, SQL window functions, CTEs...

### **Case Study 1: Hệ thống quản lý tài chính ngân hàng**

#### Bối cảnh:
Một ngân hàng cần hệ thống quản lý giao dịch tài khoản, với hàng triệu giao dịch mỗi ngày, đòi hỏi tăng tính toàn vẹn và nhất quán dữ liệu tài khoản, đảm bảo không xảy ra lỗi mất mát dữ liệu hoặc trùng lặp.

#### Tại sao chọn PostgreSQL?
- Hỗ trợ transaction để đảm bảo rằng các thao tác ghi hoặc update trên tài khoản được thực hiện toàn vẹn.
- Cơ chế locking và MVCC giúp xử lý các giao dịch đồng thời hiệu quả mà không bị deadlock.
- Hỗ trợ quy trình audit (ghi nhật ký giao dịch) qua trigger, stored procedures.
- Tự nhiên hỗ trợ các tính toán toán học và phân tích dữ liệu trên SQL.

#### Ví dụ SQL đơn giản minh họa transaction trong Postgres:

```sql
BEGIN;

-- Trừ tiền từ tài khoản A
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';

-- Cộng tiền vào tài khoản B
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'B';

COMMIT;
```

Trong trường hợp lỗi (ví dụ thiếu tiền), transaction sẽ rollback để đảm bảo dữ liệu không bị sai lệch.

---

### **Case Study 2: ERP và báo cáo phân tích nghiệp vụ**

#### Bối cảnh:
Một nhà sản xuất sử dụng hệ ERP tích hợp quản lý sản xuất, nhân sự và kho hàng, yêu cầu báo cáo phân tích thường xuyên trên dữ liệu với các phép JOIN phức tạp và tổng hợp số liệu.

#### Tại sao Postgres?
- Khả năng chạy các truy vấn JOIN nhanh, phức tạp giúp khai thác dữ liệu liên quan từ nhiều bảng khác nhau.
- Các hàm window (partition by, row_number(), rank()) thuận tiện cho báo cáo thứ bậc, phân nhóm.
- Hỗ trợ JSON và JSONB để lưu trữ một số dữ liệu không cố định trong khi vẫn truy vấn SQL được.
- Phân mảnh (partitioning) table giúp quản lý dữ liệu lớn hiệu quả.

Ví dụ truy vấn báo cáo đơn giản dùng hàm window:

```sql
SELECT employee_id, department, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
FROM employees;
```

---

## 2. MongoDB: Ứng dụng trong hệ thống dữ liệu phi cấu trúc, big data, IoT, web linh hoạt

### **Tính chất nổi bật của MongoDB:**

- Lưu trữ dữ liệu dạng document (JSON-like BSON), schema linh hoạt.
- Thích hợp với dữ liệu phi cấu trúc, có thể thay đổi schema theo thời gian.
- Khả năng mở rộng theo chiều ngang (sharding) rất tốt.
- Hỗ trợ truy vấn rich query và aggregation framework.
- Dễ phát triển nhanh, thích hợp với các ứng dụng web, mobile hay Big Data.

---

### **Case Study 3: Ứng dụng IoT - thu thập dữ liệu cảm biến**

#### Bối cảnh:
Một hệ thống IoT thu thập dữ liệu từ hàng triệu cảm biến khác nhau, kích thước mẫu dữ liệu rất lớn và không cố định kiểu dữ liệu (temperature, humidity, vị trí GPS,...). Dữ liệu nhanh và liên tục được đưa về.

#### Tại sao chọn MongoDB?
- Dữ liệu IoT thường phi cấu trúc, MongoDB lưu trữ trực tiếp tài liệu JSON rất phù hợp.
- Có thể lưu mỗi cảm biến với schema khác nhau, dễ mở rộng sửa đổi mà không cần migration schema phức tạp.
- Khả năng scale ra nhiều node để chịu tải dữ liệu lớn.
- Hỗ trợ truy vấn aggregations để tổng hợp và phân tích dữ liệu.
- Tích hợp dễ với các ứng dụng realtime, stream.

Ví dụ một document dữ liệu IoT trong MongoDB:

```json
{
  "sensor_id": "sensor_123",
  "timestamp": "2024-06-01T10:00:00Z",
  "readings": {
    "temperature": 26.5,
    "humidity": 40,
    "gps": {"lat": 10.762622, "lon": 106.660172}
  }
}
```

Lấy ví dụ truy vấn tổng hợp nhiệt độ trung bình mỗi ngày:

```js
db.sensor_data.aggregate([
  {
    $group: {
      _id: { $dateToString: { format: "%Y-%m-%d", date: "$timestamp" } },
      avgTemp: { $avg: "$readings.temperature" }
    }
  }
]);
```

---

### **Case Study 4: Ứng dụng Web có schema linh hoạt**

#### Bối cảnh:
Ứng dụng mạng xã hội hoặc CMS cho phép người dùng tạo các bài viết, bình luận, các trường dữ liệu có thể thay đổi theo thời gian, hoặc lưu các trường metadata khác nhau.

#### Tại sao chọn MongoDB?
- Schema không bắt buộc cố định, phù hợp với các loại đối tượng đa dạng, không yêu cầu cấu trúc cứng.
- Dễ tăng tốc phát triển khi yêu cầu thường xuyên thay đổi.
- Hỗ trợ hệ thống cache, tìm kiếm văn bản, và dữ liệu đa dạng.
- Khả năng mở rộng và tích hợp tốt với kiến trúc microservices.

Ví dụ document người dùng:

```json
{
  "_id": ObjectId("..."),
  "username": "john_doe",
  "profile": {
    "age": 30,
    "location": "Hanoi",
    "interests": ["music", "sports"]
  },
  "posts": [
    {
      "title": "Hello world",
      "content": "This is my first post!",
      "tags": ["intro", "hello"]
    }
  ]
}
```

---

## **Tóm tắt so sánh và lưu ý lựa chọn DBMS**

| Tiêu chí        | PostgreSQL                           | MongoDB                                          |
| ----------------|----------------------------------- |-------------------------------------------------|
| Mô hình dữ liệu | Quan hệ (row, bảng)                | Document (cấu trúc JSON linh hoạt)               |
| Tính nhất quán   | ACID cao, phù hợp hệ thống tài chính| BASE (eventual consistency), thích hợp Big Data  |
| Schema          | Cố định, mạnh về tính liên kết     | Linh hoạt, thích hợp thay đổi thường xuyên       |
| Ứng dụng điển hình| Transaction ngân hàng, ERP, báo cáo phân tích| IoT, big data, web app phát triển nhanh, dữ liệu phi cấu trúc|
| Khả năng mở rộng | Vertical scaling                   | Horizontal scaling (sharding)                    |

---

# Kết luận

- **PostgreSQL** là lựa chọn tuyệt vời cho các trường hợp cần sự chính xác, đồng nhất dữ liệu cao, các nghiệp vụ transaction quan trọng, như hệ thống ngân hàng, ERP, báo cáo phân tích.
- **MongoDB** phù hợp với các ứng dụng cần lưu trữ dữ liệu phi cấu trúc, linh hoạt schema, lượng dữ liệu lớn, như hệ thống IoT, big data, web app phát triển nhanh.

Việc lựa chọn DBMS phải dựa trên phân tích kỹ các yêu cầu nghiệp vụ, tính chất dữ liệu và tốc độ phát triển để tối ưu hiệu quả ứng dụng.

---

Nếu bạn muốn, tôi có thể hỗ trợ thêm cách thiết kế dữ liệu hoặc code mẫu chi tiết hơn với từng trường hợp. Bạn có muốn không?