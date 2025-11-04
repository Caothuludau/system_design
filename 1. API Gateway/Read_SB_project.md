Với một **Spring Boot Gateway project riêng**, bạn chỉ cần đọc **5 nhóm chính trong codebase** để hiểu toàn bộ logic và trả lời các câu hỏi kỹ thuật tôi nêu lúc trước.
Dưới đây là hướng dẫn chi tiết, cực kỳ thực hành:

---

## 🧭 Mục tiêu: Hiểu được 5 khía cạnh của API Gateway

| Nhóm                               | Cần đọc ở đâu                                               | Câu hỏi bạn sẽ trả lời được                              |
| ---------------------------------- | ----------------------------------------------------------- | -------------------------------------------------------- |
| 1️⃣ Routing & Filter               | `application.yml` / `GatewayConfig.java`                    | Request đi từ đâu đến đâu, qua những filter nào          |
| 2️⃣ Authentication & Security      | `SecurityConfig.java`, `JwtFilter.java`                     | Gateway xác thực thế nào, có kiểm quyền không            |
| 3️⃣ Request Flow                   | Các class implements `GlobalFilter`, `GatewayFilterFactory` | Gateway có log, transform, chặn, hay retry request không |
| 4️⃣ Observability & Error Handling | `LoggingFilter.java`, `ExceptionHandler.java`               | Làm gì khi lỗi, có metrics, tracing, retry không         |
| 5️⃣ Infra & Performance            | `Dockerfile`, `application.yml` (cổng, rate-limit, timeout) | Triển khai ở đâu, giới hạn tải, caching                  |

---

## 1️⃣ **Routing & Filter**

👉 Bắt đầu ở `src/main/resources/application.yml` (hoặc `.properties`)

Tìm đoạn như sau:

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: core-agent
          uri: http://core-agent:8080
          predicates:
            - Path=/core/**
          filters:
            - AddRequestHeader=X-Gateway, true
```

Nếu thấy:

* `routes:` → cho biết đường đi request → các service (Core Agent, Intent API, …)
* `filters:` → cho biết có transform, add header, rewrite path, …
* `default-filters:` → áp dụng cho toàn hệ thống
* `globalcors:` → cấu hình CORS cho client

> 💡 Gợi ý đọc:
>
> * `spring.cloud.gateway.routes`
> * `spring.cloud.gateway.globalcors`
> * `spring.cloud.gateway.default-filters`

---

## 2️⃣ **Authentication & Security**

Tìm các file có tên:

* `SecurityConfig.java`
* `JwtAuthenticationFilter.java`
* `AuthenticationManager.java`
* hoặc package `security/`, `config/`

Đọc để xác định:

* Gateway **nhận token từ client** rồi **xác thực tại Gateway** (filter JWT)
  hay chỉ **chuyển tiếp token sang Core Agent**?
* Có dùng `ReactiveAuthenticationManager` (nếu WebFlux) hay `OncePerRequestFilter` (nếu MVC)?
* Có loại trừ route nào khỏi bảo mật (`permitAll`) không?

Ví dụ:

```java
http.authorizeHttpRequests()
    .requestMatchers("/auth/**").permitAll()
    .anyRequest().authenticated()
    .and()
    .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
```

---

## 3️⃣ **Request Flow**

Tìm trong `src/main/java/.../filter/` hoặc tìm bằng IntelliJ:

```
Ctrl+Shift+F → "implements GlobalFilter"
```

hoặc

```
Ctrl+Shift+F → "GatewayFilterFactory"
```

Những class này cho biết **Gateway xử lý request như thế nào**:

* Ghi log đầu/cuối request
* Chỉnh sửa header/body
* Giới hạn tốc độ, chống spam
* Gắn thêm trace-id, correlation-id
* Retry request khi service lỗi

> 💡 Đọc `filter()` hoặc `apply()` method trong các class này.

---

## 4️⃣ **Observability & Error Handling**

Tìm:

* `@ControllerAdvice` class (xử lý exception)
* `LoggingFilter`, `TraceFilter`, `ErrorHandler`
* `spring-boot-starter-actuator` trong `pom.xml` → có nghĩa là có endpoint `/actuator/health`, `/metrics`
* Cấu hình Prometheus / Zipkin / Sleuth trong `application.yml` (nếu có)

→ Phần này giúp bạn hiểu Gateway có theo dõi log, error, và performance không.

---

## 5️⃣ **Infrastructure & Performance**

Mở:

* `Dockerfile` hoặc `docker-compose.yml` → biết cổng expose (`8080`, `8081`, …)
* `application.yml` → tìm:

  ```yaml
  server:
    port: 8080
  spring:
    main:
      web-application-type: reactive
  ```

  → Nếu có `reactive`, project dùng WebFlux (Spring Cloud Gateway mới)
* Tìm thêm:

  ```yaml
  spring:
    cloud:
      gateway:
        httpclient:
          connect-timeout: 2000
          response-timeout: 5s
  ```

  → Có giới hạn thời gian kết nối, chứng tỏ chú trọng performance.

---

## ✅ Kết luận: Thứ tự bạn nên đọc

1. `application.yml` — hiểu routing và filter.
2. `SecurityConfig.java` / `JwtFilter.java` — hiểu auth.
3. Các `Filter` hoặc `GatewayFilterFactory` — hiểu logic xử lý.
4. `ErrorHandler` + `Actuator` — hiểu log/monitor.
5. `Dockerfile` / `pom.xml` — hiểu runtime & dependency.

---

Nếu bạn muốn, bạn có thể **copy phần `application.yml` (ẩn domain, token)** và **tên class trong package `config/` hoặc `filter/`** gửi qua đây — tôi sẽ chỉ bạn **nên đọc class nào trước và dòng nào quan trọng nhất**, để không phải mở lung tung.

Bạn có muốn tôi giúp bạn **lên checklist cụ thể cho chính project Gateway của bạn (dựa trên Spring Cloud Gateway)** để bạn tick dần khi đọc không?
