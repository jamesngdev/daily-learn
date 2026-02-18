# Các kỹ thuật nâng cao trong RabbitMQ: High Availability, Clustering và Tối ưu hiệu suất

---

## Giới thiệu chung

Ngày thứ hai tập trung vào các kỹ thuật và tính năng nâng cao quan trọng để triển khai RabbitMQ trong môi trường production thực tế. Đó là các phương pháp giúp hệ thống:

- Mở rộng quy mô (scalability)
- Tăng tính sẵn sàng (high availability)
- Tối ưu hiệu suất xử lý tin nhắn (performance tuning)
- Đảm bảo an toàn thông tin và giám sát hoạt động (security & monitoring)

Trong bài viết này, chúng ta sẽ lần lượt tìm hiểu:

1. Cấu hình **Clustering** và **Federation** để mở rộng và phân phối tải.
2. Triển khai **High Availability** với Mirror Queues, xử lý failover và cân bằng tải.
3. Tối ưu hiệu suất thông qua tuning các tham số như prefetch, persistent messages và batch processing.
4. Kỹ thuật giám sát và bảo mật, bao gồm xác thực, ủy quyền, TLS/SSL và plugin quản lý.

---

# 1. Clustering và Federation trong RabbitMQ

## 1.1 RabbitMQ Clustering

Clustering là việc kết nối nhiều node RabbitMQ thành một nhóm (cluster) để chia sẻ metadata (user, queue, exchange, binding...) và cung cấp khả năng mở rộng (scale-out) và tăng tính sẵn sàng.

- **Kiến trúc**: Các node trong cluster chia sẻ trạng thái metadata nhưng dữ liệu queue chỉ có trên node nắm giữ queue đó.
- Nếu node giữ queue chết, queue đó không còn sẵn sàng.
- Thích hợp cho các workloads có thể phân bổ queue theo nhóm.

### Cách tạo cluster đơn giản

Giả sử bạn có 2 node RabbitMQ đang chạy:

```bash
# Trên node2, kết nối vào node1
rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@node1
rabbitmqctl start_app
```

Lệnh trên khiến node2 hợp nhất vào cluster với node1.

### Lưu ý

- Tất cả node phải có cùng RabbitMQ version.
- Node name (`rabbit@hostname`) phải xác định đúng.
- Firewall cổng 4369 (Erlang Port Mapper Daemon) phải mở.
- Metadata được đồng bộ, message queue dữ liệu chỉ tồn tại trên node sở hữu queue.

---

## 1.2 Federation

Federation cho phép kết nối các cluster RabbitMQ hoặc broker độc lập ở các vị trí địa lý khác nhau hoặc môi trường khác nhau, thông qua các kênh federated exchanges hoặc queues.

- Khác với cluster, federation không đồng bộ metadata.
- Mỗi broker hoạt động độc lập.
- Phù hợp cho hệ thống phân tán, hạn chế latency và tăng tính khả dụng.

### Ví dụ Federation setup (từ node A đến node B)

```erlang
rabbitmqctl set_parameter federation-upstream my-upstream \
  '{"uri":"amqp://user:password@nodeB","expires":3600000}'
```

Sau đó tạo policy để kích hoạt federation trên exchange cần thiết:

```bash
rabbitmqctl set_policy federate-me "^my-exchange$" \
  '{"federation-upstream":"my-upstream"}' --apply-to exchanges
```

---

# 2. High Availability với Mirror Queues

## 2.1 Mirror Queues là gì?

Mirror Queues (queue nhân bản) giữ các copies của queue trên các node cluster khác nhau để đảm bảo tính sẵn sàng khi node chính chết.

- Mọi queue có policy mirror sẽ có các bản copy trên các node phụ (mirror nodes).
- Message đồng bộ trên tất cả mirror nodes.
- Khi node chủ chết, một mirror node sẽ được promote thành chủ, không mất dữ liệu.

---

## 2.2 Cấu hình Mirror Queues

### Cơ bản: tạo policy mirror queue

```bash
rabbitmqctl set_policy ha-all ".*" '{"ha-mode":"all"}'
```

Ý nghĩa:

- Áp dụng cho tất cả queue.
- `ha-mode: all` tức là nhân bản trên tất cả node của cluster.

Hoặc bạn có thể giới hạn số node:

```bash
rabbitmqctl set_policy ha-two ".*" '{"ha-mode":"exactly","ha-params":2}'
```

Trong đó `"ha-params":2` tạo ra 2 bản mirror.

---

## 2.3 Failover và cân bằng tải

- Khi node chủ queue down, một mirror queue được promote tự động, client có thể reconnect lại node mới mà không mất message.
- Để cân bằng tải, bạn nên push message đến exchange cluster-wide và để queue mirror xử lý.
- Client khi khởi tạo kết nối cần sử dụng danh sách các node để failover.

Ví dụ cấu hình kết nối client trong Java:

```java
ConnectionFactory factory = new ConnectionFactory();
factory.setUsername("guest");
factory.setPassword("guest");
factory.setVirtualHost("/");
factory.setAutomaticRecoveryEnabled(true);
factory.setTopologyRecoveryEnabled(true);

// Thêm danh sách các host cho failover
factory.setHost("node1");
factory.addHost("node2");
factory.addHost("node3");

Connection connection = factory.newConnection();
```

---

# 3. Tối ưu hiệu suất

## 3.1 Tuning prefetch

Prefetch chỉ định số lượng message mà consumer có thể nhận trước khi ack message cũ.

- Giá trị thấp làm giảm hiệu suất do thường xuyên chờ ack.
- Giá trị cao tăng throughput nhưng có thể tăng bộ nhớ đệm.

Ví dụ RabbitMQ Java client config prefetch:

```java
channel.basicQos(50);  // consumer nhận tối đa 50 message chưa ack lần lượt
```

---

## 3.2 Persistent messages

- Messages persistent được ghi vào disk, giúp không mất message khi broker restart.
- Tuy nhiên ghi đĩa chậm hơn message transient, ảnh hưởng throughput.

Bạn có thể quyết định khi publish message:

```java
AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
    .deliveryMode(2)   // 2 = persistent
    .build();

channel.basicPublish(exchange, routingKey, props, messageBody);
```

Nếu cần tối ưu throughput:

- Batch gửi message persistent.
- Sử dụng batch confirm (Publisher confirms).

---

## 3.3 Batch processing và Publisher Confirms

### Batch gửi message

Tích trữ nhiều message rồi gửi cùng lúc giúp giảm overhead.

### Publisher Confirms

Cho phép producer biết message đã nhận và lưu thành công bởi broker, tránh mất message.

Ví dụ Java:

```java
channel.confirmSelect();

for (byte[] message : messages) {
    channel.basicPublish(exchange, routingKey, props, message);
}

channel.waitForConfirmsOrDie(5000); // chờ confirm trong 5s
```

---

# 4. Giám sát và bảo mật RabbitMQ

## 4.1 Giám sát với Plugin quản lý

RabbitMQ Management Plugin cho phép:

- Dashboard theo dõi queue, exchange, message rates, node health.
- Thực hiện cấu hình, restart node,...
- API REST cho tự động hóa.

Kích hoạt plugin:

```bash
rabbitmq-plugins enable rabbitmq_management
```

Thường truy cập qua URL: `http://localhost:15672`

---

## 4.2 Bảo mật: xác thực, ủy quyền và TLS/SSL

### Xác thực và ủy quyền

- RabbitMQ hỗ trợ nhiều backend xác thực: internal, LDAP, external auth.
- Quyền truy cập được quản lý bằng `vhost`, quyền đọc ghi queue/exchange.

### Cấu hình user

```bash
rabbitmqctl add_user user1 strongpassword
rabbitmqctl set_permissions -p / user1 ".*" ".*" ".*"
```

### TLS/SSL

- Mã hóa kết nối client <-> broker giúp bảo vệ dữ liệu qua mạng.
- Kích hoạt TLS trên RabbitMQ config (`rabbitmq.conf`):

```ini
listeners.tcp = none
listeners.ssl.default = 5671

ssl_options.cacertfile = /path/to/ca_certificate.pem
ssl_options.certfile   = /path/to/server_certificate.pem
ssl_options.keyfile    = /path/to/server_key.pem
ssl_options.verify     = verify_peer
ssl_options.fail_if_no_peer_cert = true
```

Client kết nối:

```java
ConnectionFactory factory = new ConnectionFactory();

factory.useSslProtocol();
factory.setHost("rabbitmq.example.com");
factory.setPort(5671);

Connection conn = factory.newConnection();
```

---

## Tổng kết kiến thức

| Kỹ thuật          | Mục đích                         | Ví dụ/Config                                                 |
|-------------------|---------------------------------|--------------------------------------------------------------|
| **Clustering**    | Mở rộng, chia sẻ metadata        | `rabbitmqctl join_cluster rabbit@node1`                      |
| **Federation**    | Liên kết nhiều broker độc lập    | set_parameter, set_policy federation                          |
| **Mirror Queues** | High Availability                | `set_policy ha-all ".*" '{"ha-mode":"all"}'`                 |
| **Prefetch QoS**  | Tối ưu performance consumer      | `channel.basicQos(50)`                                        |
| **Persistent Msg**| Đảm bảo message không mất        | `BasicProperties.deliveryMode = 2`                           |
| **Publisher Confirms** | Tối ưu throughput, tránh mất message | `channel.confirmSelect() + waitForConfirmsOrDie()`            |
| **Management Plugin** | Giám sát, quản lý live          | `rabbitmq-plugins enable rabbitmq_management`                |
| **TLS/SSL**       | Bảo mật kết nối                  | cấu hình ssl_options trong `rabbitmq.conf`                    |
| **Xác thực, ủy quyền** | Kiểm soát truy cập              | `rabbitmqctl add_user` và `set_permissions`                   |

---

# Phụ lục: sơ đồ đơn giản Clustering + Mirror Queues

```mermaid
graph LR
    subgraph Cluster
    N1[node1 (Master Queue)]
    N2[node2 (Mirror Queue)]
    N3[node3 (Mirror Queue)]
    end

    Client --> Exchange
    Exchange --> N1
    N1 <--> N2
    N1 <--> N3

    style N1 fill:#9fdfbf
    style N2 fill:#d3eecd
    style N3 fill:#d3eecd
```

---

# Kết luận

Việc triển khai RabbitMQ trong môi trường production yêu cầu nắm vững các kỹ thuật nâng cao như clustering, federation, mirror queues để tăng khả năng mở rộng và tính ổn định. Đồng thời, tuning các tham số giúp tối ưu hiệu suất xử lý, kết hợp giám sát và bảo mật giúp hệ thống hoạt động hiệu quả và an toàn hơn.

Hy vọng bài viết giúp bạn có cái nhìn tổng thể và kỹ năng cần thiết để vận hành RabbitMQ ở scale lớn và môi trường nghiêm ngặt thực tế. Nếu cần, mình có thể cung cấp thêm ví dụ code, hướng dẫn chi tiết từng bước khác nhé!