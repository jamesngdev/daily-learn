# Case Study Chi Tiết: Khi Nào Nên Dùng PostgreSQL?

PostgreSQL (Postgres) là một hệ quản trị cơ sở dữ liệu quan hệ mã nguồn mở, nổi bật nhờ **tính năng mở rộng**, **kiến trúc mạnh mẽ**, và **sự tuân thủ chuẩn SQL nghiêm ngặt**. Trong bài viết này, chúng ta sẽ tập trung phân tích các tình huống cụ thể mà PostgreSQL được ưu tiên sử dụng, đặc biệt:

- Hệ thống ngân hàng
- Ứng dụng tài chính với nhiều transaction phức tạp
- Dữ liệu có cấu trúc rõ ràng, yêu cầu schema nghiêm ngặt
- Xử lý truy vấn đa bảng phức tạp

---

## 1. Tổng quan về PostgreSQL

- **ACID Compliance**: Đảm bảo tính nhất quán dữ liệu qua các transaction.
- **Schema Rigid**: Hỗ trợ schema rõ ràng, chế độ ràng buộc dữ liệu chặt chẽ.
- **Khả năng xử lý query phức tạp**: JOIN đa bảng, sub-queries, window functions.
- **Hỗ trợ JSON/JSONB**: Dù tập trung dữ liệu có cấu trúc, PostgreSQL vẫn hỗ trợ lưu trữ dữ liệu phi cấu trúc linh hoạt.
- **Tính mở rộng và bảo mật cao**

---

## 2. Khi nào nên chọn PostgreSQL?

### Tình huống 1: Hệ thống ngân hàng

**Vì sao?**

Ngân hàng yêu cầu:

- Tính nhất quán cực kỳ nghiêm ngặt trong các giao dịch (ACID).
- Ràng buộc schema rõ ràng để đảm bảo kiểu dữ liệu, bắt buộc các trường thông tin quan trọng phải chính xác như số tài khoản, mã khách hàng,...
- Xử lý transaction đa bước phức tạp: chuyển tiền giữa nhiều tài khoản, hạch toán đa chiều.
- Audit, logging dữ liệu.

**Ví dụ cụ thể: Hệ thống chuyển khoản giữa các tài khoản**

```sql
-- Tạo bảng tài khoản với giới hạn số dư không âm
CREATE TABLE accounts (
    account_id SERIAL PRIMARY KEY,
    account_holder VARCHAR(100) NOT NULL,
    balance NUMERIC(15, 2) CHECK (balance >= 0) NOT NULL
);

-- Tạo bảng ghi transaction
CREATE TABLE transactions (
    transaction_id SERIAL PRIMARY KEY,
    from_account INT REFERENCES accounts(account_id),
    to_account INT REFERENCES accounts(account_id),
    amount NUMERIC(15, 2) NOT NULL CHECK (amount > 0),
    transaction_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Thực hiện chuyển tiền trong transaction đảm bảo ACID
BEGIN;

UPDATE accounts
SET balance = balance - 1000.00
WHERE account_id = 1 AND balance >= 1000.00;

UPDATE accounts
SET balance = balance + 1000.00
WHERE account_id = 2;

INSERT INTO transactions(from_account, to_account, amount)
VALUES (1, 2, 1000.00);

COMMIT;
```

**Giải thích:**

- Câu lệnh trên đảm bảo chỉ cho phép chuyển khoản nếu `from_account` còn đủ tiền (`balance >= 1000.00`).
- Nếu một bước lỗi, toàn bộ các thay đổi rollback, đảm bảo dữ liệu luôn nhất quán.
- PostgreSQL cho phép define các ràng buộc kiểm soát dữ liệu chính xác, tránh sai lệch.

---

### Tình huống 2: Ứng dụng có nhiều transaction tài chính liên quan đến thanh toán, đặt hàng

Trong các ứng dụng thương mại điện tử, fintech, xử lý transaction đa bước, đa bảng với tính nhất quán cao rất quan trọng.

**Khó khăn:**

- Thao tác đọc ghi đồng thời với lượng lớn user.
- Nhu cầu rollback khi lỗi phát sinh.
- Cần câu truy vấn phức tạp để tổng hợp dữ liệu (ví dụ: báo cáo doanh thu, phân tích đơn hàng).

**PostgreSQL giải quyết:**

- Hỗ trợ MVCC (Multi-Version Concurrency Control) giúp tăng hiệu năng đọc ghi song song.
- Nhiều kiểu join, window functions giúp xây dựng báo cáo chi tiết.
- Các loại transaction isolation level tùy chỉnh phù hợp yêu cầu nghiệp vụ

**Ví dụ query phức tạp:**

```sql
-- Báo cáo tổng doanh thu theo tháng và loại sản phẩm
SELECT p.category,
       DATE_TRUNC('month', o.order_date) AS month,
       SUM(od.quantity * od.unit_price) AS total_revenue
FROM orders o
JOIN order_details od ON o.order_id = od.order_id
JOIN products p ON od.product_id = p.product_id
WHERE o.status = 'completed'
GROUP BY p.category, month
ORDER BY month DESC;
```

---

### Tình huống 3: Dữ liệu có cấu trúc rõ ràng, yêu cầu schema nghiêm ngặt

Các ứng dụng như phần mềm quản lý nhân sự, chăm sóc khách hàng CRM,... bắt buộc dữ liệu tuân theo schema chặt chẽ:

- Kiểu dữ liệu được xác định rõ ràng.
- Ràng buộc dữ liệu như bắt buộc, duy nhất (unique), khóa ngoại...
- Không cho phép dữ liệu lộn xộn, sai cấu trúc.

PostgreSQL hỗ trợ:

- Các kiểu dữ liệu phức tạp (ENUM, composite types, arrays).
- Ràng buộc CHECK có thể viết tùy chỉnh.
- Các schema và quyền truy cập linh hoạt theo từng cấp độ.

**Ví dụ:**

```sql
CREATE TYPE employee_status AS ENUM ('active', 'inactive', 'on_leave');

CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    status employee_status NOT NULL DEFAULT 'active',
    date_joined DATE NOT NULL
);

-- Ràng buộc email phải hợp lệ đơn giản
ALTER TABLE employees
ADD CONSTRAINT email_check CHECK (email LIKE '%_@__%.__%');
```

---

### Tình huống 4: Cần xử lý truy vấn phức tạp đa bảng

PostgreSQL vượt trội trong xử lý truy vấn đa bảng (JOIN), subquery, CTE (Common Table Expressions), window functions phục vụ các báo cáo, phân tích dữ liệu sâu.

**Ví dụ CTE kết hợp window function:**

```sql
WITH monthly_sales AS (
    SELECT
        salesperson_id,
        DATE_TRUNC('month', sale_date) AS month,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY salesperson_id, month
)
SELECT
    salesperson_id,
    month,
    total_sales,
    RANK() OVER (PARTITION BY month ORDER BY total_sales DESC) AS sales_rank
FROM monthly_sales
ORDER BY month, sales_rank;
```

**Ý nghĩa:**

- Tính tổng doanh số theo từng nhân viên và tháng.
- Xếp hạng nhân viên theo doanh thu mỗi tháng.
- Dùng kết hợp CTE và window functions để làm báo cáo dễ đọc và hiệu quả.

---

## 3. So sánh nhanh với một số hệ quản trị khác

| Tiêu chí          | PostgreSQL                           | MySQL                                  | MongoDB                                |
|-------------------|------------------------------------|---------------------------------------|---------------------------------------|
| ACID              | Full ACID support                  | ACID trong InnoDB, hạn chế             | Không hỗ trợ transaction phức tạp     |
| Schema            | Chặt chẽ, rõ ràng                 | Không chặt chẽ như Postgres           | Schema-less (phi cấu trúc)             |
| Query phức tạp    | Hỗ trợ mạnh, window, CTE, join đa bảng | Hỗ trợ join nhưng hạn chế              | Không hỗ trợ join, dùng aggregation    |
| Mở rộng           | Tốt, hỗ trợ nhiều loại index, extension | Tốt nhưng ít tính năng nâng cao hơn    | Tốt cho dữ liệu phi cấu trúc và scale |

---

## **Kết luận**

- **Chọn PostgreSQL khi bạn cần một hệ quản trị cơ sở dữ liệu quan hệ mạnh mẽ, đảm bảo tính nhất quán dữ liệu nghiêm ngặt.**
- **Phù hợp với hệ thống ngân hàng, các ứng dụng tài chính có nhiều transaction phức tạp, dữ liệu dạng bảng rõ ràng, schema chặt chẽ và xử lý truy vấn đa bảng phức tạp.**
- PostgreSQL giúp giảm thiểu nguy cơ mất mát, sai lệch dữ liệu trong các nghiệp vụ quan trọng liên quan đến tiền bạc, thông tin khách hàng.

---

## Tài liệu tham khảo

- [PostgreSQL Official Documentation](https://www.postgresql.org/docs/current/)
- [Building Financial Applications with Postgres (Blog)](https://www.cybertec-postgresql.com/en/postgresql-for-financial-applications/)
- [Postgres ACID Transactions and Isolation Levels](https://severalnines.com/database-blog/postgresql-transaction-isolation-levels-explained)

---

Nếu bạn đang phát triển hệ thống tài chính hoặc ngân hàng, hãy ưu tiên thiết kế với PostgreSQL để đảm bảo dữ liệu đáng tin cậy và khai thác sức mạnh truy vấn phức tạp nhé!