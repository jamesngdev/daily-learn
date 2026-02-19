# Case Study Chi Tiết: Khi Nào Nên Dùng MongoDB?

MongoDB là một cơ sở dữ liệu NoSQL dạng document, nổi bật với khả năng lưu trữ dữ liệu phi cấu trúc hoặc bán cấu trúc dưới dạng JSON-like documents (BSON). Nó phù hợp với nhiều ứng dụng yêu cầu linh hoạt về mô hình dữ liệu, hiệu suất truy vấn cao trên dữ liệu phi quan hệ, và khả năng mở rộng dễ dàng.

Trong bài viết này, chúng ta sẽ phân tích chi tiết các trường hợp **ưu tiên sử dụng MongoDB** thông qua các ví dụ thực tế, giúp bạn hiểu rõ khi nào MongoDB là lựa chọn tối ưu.

---

## 1. Startup cần nhanh chóng thay đổi Schema

### Vấn đề

- Các startup thường phải thay đổi schema dữ liệu nhanh theo yêu cầu thị trường, thử nghiệm tính năng mới.
- Cơ sở dữ liệu quan hệ (SQL) đòi hỏi schema cố định, thay đổi schema thường phức tạp, mất thời gian.

### MongoDB giải quyết như thế nào?

- MongoDB không cần schema cố định, bạn có thể thêm/xóa/trường mới trong document dễ dàng.
- Không cần migration phức tạp, dữ liệu cũ vẫn truy xuất được ngay cả khi document thay đổi cấu trúc.

### Ví dụ

Giả sử startup phát triển ứng dụng quản lý sản phẩm ecommerce:

- Ban đầu bạn chỉ cần lưu thông tin sản phẩm với các trường: `name`, `price`.
- Sau một thời gian, muốn lưu thêm thông tin đánh giá, `ratings`.

**Document ban đầu:**

```json
{
  "_id": ObjectId("..."),
  "name": "Smartphone XYZ",
  "price": 500
}
```

**Document sau khi bổ sung**

```json
{
  "_id": ObjectId("..."),
  "name": "Smartphone XYZ",
  "price": 500,
  "ratings": [5, 4, 3, 5]
}
```

Bạn không cần thay đổi cấu trúc bảng hay thêm cột - chỉ cần ghi nhận document mới với trường mới.

### Mã mẫu thêm trường mới (Node.js với MongoDB driver):

```javascript
const { MongoClient } = require('mongodb');

async function addRatings(productId, ratings) {
  const client = new MongoClient('mongodb://localhost:27017');
  await client.connect();
  const db = client.db('shop');
  const products = db.collection('products');

  await products.updateOne(
    { _id: productId },
    { $set: { ratings: ratings } }
  );

  await client.close();
}
```

---

## 2. Ứng dụng lưu trữ nội dung Web (CMS, Blog, eMagazine…)

### Vấn đề

- Nội dung bài viết đa dạng với nhiều kiểu dữ liệu: văn bản, hình ảnh, metadata, tags, bình luận.
- Khó thiết kế bảng quan hệ bao quát hết được các trường thay đổi thường xuyên.

### MongoDB giải quyết như thế nào?

- Document JSON cho phép lưu trữ nội dung đa dạng dưới cùng một tài liệu.
- Dễ dàng nest các trường con như comments trong bài viết.

### Ví dụ

Document bài viết một blog:

```json
{
  "_id": ObjectId("..."),
  "title": "Học MongoDB căn bản",
  "author": "Nguyen Van A",
  "content": "<p>MongoDB là...</p>",
  "tags": ["NoSQL", "Database", "MongoDB"],
  "comments": [
    {
      "user": "User1",
      "comment": "Bài viết rất hữu ích!",
      "date": "2024-05-01"
    },
    {
      "user": "User2",
      "comment": "Mình thích ví dụ code.",
      "date": "2024-05-02"
    }
  ]
}
```

Chỉ cần một document ghi lại tất cả nội dung, tags và comments, không phải join nhiều bảng.

---

## 3. Hệ thống xử lý dữ liệu phi cấu trúc lớn (Big Data)

### Vấn đề

- Dữ liệu vào có thể đa dạng, từ sensor, mạng xã hội, logs, IoT… thường dưới dạng JSON, không có cấu trúc cố định.
- Dữ liệu khổng lồ cần lưu trữ và truy cập nhanh, mở rộng theo nhu cầu.

### MongoDB giải quyết như thế nào?

- MongoDB tối ưu cho dữ liệu phi cấu trúc, hỗ trợ truy vấn sâu trong nested JSON.
- Hỗ trợ sharding, phân tán dữ liệu tốt để mở rộng quy mô.
- Hệ sinh thái hỗ trợ tốt các công cụ tích hợp big data (ví dụ tích hợp với Apache Spark).

### Ví dụ

Giả sử hệ thống giám sát cảm biến nhiệt độ của một nhà máy với dữ liệu thu về mỗi giây:

```json
{
  "sensorId": "sensor01",
  "timestamp": 1685558400,
  "metrics": {
    "temperature": 27.5,
    "humidity": 58
  }
}
```

Dữ liệu có thể thay đổi thêm trường mới mà không cần bảo trì hệ thống.

---

## 4. Hệ thống dữ liệu thời gian thực (Real-time analytics, IoT, games)

### Vấn đề

- Ứng dụng dành cho game, IoT hoặc phân tích dữ liệu trực tiếp cần cập nhật và truy vấn dữ liệu rất nhanh.
- Dữ liệu thời gian thực thường thay đổi liên tục, đa dạng và có định dạng phức tạp.

### MongoDB giải quyết như thế nào?

- MongoDB có khả năng ghi nhanh, hỗ trợ các thao tác insert/update nhiều, đồng thời truy vấn dữ liệu hiệu quả.
- Mô hình document cho phép lưu dữ liệu realtime rất linh hoạt.
- Hỗ trợ change streams để đẩy dữ liệu update realtime cho frontend hoặc xử lý streaming.

### Ví dụ

Hệ thống theo dõi vị trí xe tải:

```json
{
  "truckId": "TX123",
  "location": {
    "lat": 10.762622,
    "lng": 106.660172
  },
  "status": "moving",
  "lastUpdate": ISODate("2024-06-01T10:00:00Z")
}
```

Bạn có thể dùng Change Streams để phát hiện cập nhật mới của xe ngay lập tức:

```javascript
const changeStream = collection.watch();
changeStream.on('change', (next) => {
  console.log("Document changed:", next);
});
```

---

## 5. Dữ liệu đa dạng với định dạng document JSON

### Vấn đề

- Khi hệ thống cần lưu trữ nhiều loại tài liệu JSON khác nhau.
- Các trường dữ liệu khác nhau không đồng nhất giữa các documents nhưng vẫn muốn lưu trong cùng một collection.

### MongoDB giải quyết như thế nào?

- MongoDB là database dạng tài liệu (document database) gốc, rất linh hoạt với JSON/BSON.
- Không cần thiết kế schema cứng nhắc, dễ dàng lưu tài liệu có cấu trúc khác biệt dưới cùng một collection.

### Ví dụ

Hệ thống CRM quản lý khách hàng và lịch sử tương tác, các tài liệu có thể khác nhau:

**Khách hàng**

```json
{
  "_id": ObjectId("..."),
  "type": "customer",
  "name": "Nguyen Van B",
  "email": "nguyenvanb@example.com",
  "phone": "0123456789"
}
```

**Lịch sử tương tác**

```json
{
  "_id": ObjectId("..."),
  "type": "interaction",
  "customerId": ObjectId("..."),
  "date": ISODate("2024-06-01T08:00:00Z"),
  "mode": "phone call",
  "notes": "Khách hàng quan tâm sản phẩm mới"
}
```

Cùng lưu trong collection `crm_data`, dễ dàng truy vấn theo trường `type` mà không phải join phức tạp.

---

# Kết luận

| Trường hợp sử dụng                          | Ưu điểm MongoDB                                                            |
|--------------------------------------------|----------------------------------------------------------------------------|
| Startup nhanh thay đổi schema               | Schema linh hoạt, không cần migration phức tạp                             |
| Ứng dụng lưu trữ nội dung Web               | Lưu trữ tài liệu đa dạng, dễ nest dữ liệu                                  |
| Xử lý dữ liệu phi cấu trúc lớn              | Hỗ trợ dữ liệu JSON đa dạng, mở rộng tốt với sharding                      |
| Hệ thống dữ liệu thời gian thực              | Ghi/đọc nhanh, hỗ trợ change streams để streaming realtime                 |
| Dữ liệu đa dạng, nhiều định dạng JSON       | Lưu trữ đa dạng document trong cùng collection, xử lý JSON thuận tiện      |

MongoDB là lựa chọn tối ưu mỗi khi bạn cần sự linh hoạt về cấu trúc dữ liệu, hiệu năng truy cập cao trên dữ liệu dạng document và khả năng mở rộng linh hoạt.

---

## Tham khảo thêm tài nguyên

- [MongoDB Official Documentation](https://docs.mongodb.com/)
- [Change Streams in MongoDB](https://www.mongodb.com/docs/manual/changeStreams/)
- [MongoDB Node.js Driver Tutorial](https://www.mongodb.com/docs/drivers/node/current/)

Nếu bạn cần hỗ trợ thêm về cách triển khai cụ thể dự án với MongoDB, hãy cho tôi biết nhé!