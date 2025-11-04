🧩 1. Chức năng chính của API Gateway
•	Gateway của bạn hiện tại chỉ định tuyến (reverse proxy), hay có thêm chức năng khác như authentication, rate limiting, request transformation?
•	Gateway này có đảm nhận logging/tracing và monitoring cho toàn hệ thống không?
•	Nó có thực hiện load balancing giữa nhiều instance của Core Agent không?
________________________________________
🔒 2. Bảo mật & xác thực
•	Cách xác thực request từ client: JWT, OAuth2, API key, hay cơ chế riêng của bạn?
•	Gateway có xác minh role/permission hay chỉ kiểm tra token rồi forward cho Core Agent?
•	Bạn có dùng TLS/HTTPS termination ngay tại Gateway không?
________________________________________
⚙️ 3. Hạ tầng triển khai
•	Gateway được triển khai bằng gì? (ví dụ: Nginx, Spring Cloud Gateway, Kong, Traefik, API Gateway của AWS, v.v.)
•	Bạn định chạy nó trong môi trường nào: Kubernetes, EC2, hay on-premises?
•	Có sử dụng service discovery (Consul, Eureka, v.v.) để tìm Core Agent không?
________________________________________
📈 4. Routing và kiến trúc
•	Gateway có mapping tĩnh (ví dụ /core-agent/**) hay động theo intent/request type?
•	Có hỗ trợ WebSocket hoặc streaming request-response không?
•	Có cần fallback (ví dụ khi Core Agent down thì redirect sang server dự phòng)?
________________________________________
🧰 5. Quản lý request
•	Gateway có ghi log chi tiết (body, header, latency) không?
•	Có cơ chế caching (VD: Redis cache layer) hay không?
•	Có giới hạn kích thước request hoặc timeout cụ thể không?
