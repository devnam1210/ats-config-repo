# ATS Config Repository

Kho cấu hình cho project:
https://github.com/hauhoa123r/MSS301-Presentation-Config-Service

Chứa cấu hình ba service nghiệp vụ và ba profile dev/test/prod.
Code Config Server và Config Client thuộc repository ATS.
Được chuẩn bị từ mã nguồn nhóm, chưa kiểm chứng tích hợp chạy thực tế.

## Cấu trúc

- application.yml: Eureka client, Actuator health/info và dấu nhận biết kho cấu hình.
- application-{profile}.yml: địa chỉ Eureka, log và tên môi trường dùng chung.
- {service}.yml: driver PostgreSQL, dialect và quy tắc đặt tên bảng của service.
- {service}-{profile}.yml: cổng, database, Hibernate và giá trị demo.
- application-service-{profile}.yml còn chứa URL Feign tới Job/Candidate.

Service: job-service, candidate-service, application-service.
Profile: dev, test, prod. Nhánh: main.
Giữ nguyên tên application-service theo mã nguồn; kiểm tra propertySources cho
cả ba service khi tích hợp, không tự đổi tên service trong kho này.
Tên service và profile do ứng dụng client chọn khi khởi động.

## Ba môi trường

| Thuộc tính | dev | test | prod |
|---|---|---|---|
| Database mặc định | *_sdb của project | *_sdb_test riêng biệt | bắt buộc qua biến môi trường |
| Hibernate ddl-auto | update | validate | validate |
| show_sql | true | false | false |
| Log fu.ats | DEBUG | INFO | INFO |
| Eureka | localhost hoặc biến môi trường | localhost hoặc biến môi trường | bắt buộc qua biến môi trường |

Cổng mặc định 8081/8082/8083. Demo các môi trường lần lượt.
Chạy dev và test đồng thời cần cổng, URL Feign và registry Eureka tách biệt để
tránh xung đột hoặc định tuyến lẫn môi trường.
Profile không tự tạo database, schema hoặc máy chủ.
Prod là mẫu cần cung cấp tài nguyên thực tế, không phải production đã triển khai.

## Biến môi trường trên máy chạy service

| Service | URL database | Tài khoản | Mật khẩu bắt buộc | Cổng tùy chọn |
|---|---|---|---|---|
| job-service | JOB_DB_URL | JOB_DB_USERNAME | JOB_DB_PASSWORD | JOB_SERVER_PORT |
| candidate-service | CANDIDATE_DB_URL | CANDIDATE_DB_USERNAME | CANDIDATE_DB_PASSWORD | CANDIDATE_SERVER_PORT |
| application-service | APPLICATION_DB_URL | APPLICATION_DB_USERNAME | APPLICATION_DB_PASSWORD | APPLICATION_SERVER_PORT |

Dev/test mặc định username postgres; URL xem YAML.
Prod bắt buộc URL, username. Không có mật khẩu mặc định ở bất kỳ profile nào.
Mỗi service chỉ cần biến có tiền tố tương ứng.

- EUREKA_DEFAULT_ZONE: URL đầy đủ như http://localhost:8761/eureka/.
- JOB_SERVICE_URL và CANDIDATE_SERVICE_URL: đặt tại application-service.
  Dev/test mặc định localhost:8081/8082; prod bắt buộc cung cấp.
- SPRING_PROFILES_ACTIVE: dev, test hoặc prod, đặt tại client.

Đặt biến trong Run Configuration của IDE hoặc môi trường tiến trình.
Không commit bí mật hoặc đặt mật khẩu trong URL database.
Không cần đặt mật khẩu database tại Config Server chỉ để trả cấu hình.
Placeholder được client giải quyết từ môi trường của nó.
Spring Boot không tự nạp .env nếu chưa có cơ chế hỗ trợ.

## Database

Project nguồn có database/job-service/schema.sql,
database/candidate-service/schema.sql và database/application-service/schema.sql.
Đây là script tạo bảng, không chứng minh database đã được tạo hoặc script đã chạy.

Test cần database riêng: job_sdb_test, candidate_sdb_test, application_sdb_test.
Kết nối đúng từng database trống rồi chạy schema tương ứng trước khi chạy service.
Không chạy lại script tạo bảng trên database đã có bảng.
validate kiểm tra schema, không tự tạo hay xóa bảng.
Nếu validation lỗi, đối chiếu entity/schema; không tự chuyển sang create/drop.

## Bàn giao Config Server cho Thuận

Tạo module Config Server ở repository ATS với dependency phù hợp và
@EnableConfigServer. Kiểm tra tương thích Boot/Cloud trước khi chạy.
Các giá trị cấu hình server:

- server.port: 8888
- spring.application.name: config-server
- spring.cloud.config.server.git.uri: https://github.com/devnam1210/ats-config-repo.git
- spring.cloud.config.server.git.default-label: main
- spring.cloud.config.server.git.clone-on-start: true

File nằm ở gốc repo nên không cần search-paths.
Kiểm tra http://localhost:8888/job-service/dev/main,
http://localhost:8888/job-service/test/main,
http://localhost:8888/job-service/prod/main.
Thay tên bằng candidate-service và application-service để kiểm tra đủ 9 trường hợp.
Đọc name, profiles, version, propertySources, ats.environment, ats.demo.message.
HTTP 200 chưa đủ: phải có đúng nguồn file và giá trị.
Placeholder mật khẩu còn nguyên trong JSON không phải lỗi thiếu mật khẩu trong Git.

## Bàn giao Config Client cho Tiến Nam và Sơn

Thêm dependency Config Client phù hợp trong service, giữ cấu hình kết nối tại client:

- spring.application.name: tên service tương ứng
- spring.config.import: configserver:http://localhost:8888
- SPRING_PROFILES_ACTIVE: dev hoặc test hoặc prod

Đặt biến mật khẩu của service trước khi chạy.
Import không có optional để lỗi kết nối hiện rõ trong demo.
Bỏ các giá trị cũ đã chuyển sang Git khỏi application.yml của client, đặc biệt
mật khẩu viết trực tiếp; kiểm tra thứ tự ưu tiên khi còn nhiều nguồn cấu hình.
Thay localhost nếu server chạy trên máy khác.
Không có Config Client thì service chưa tự lấy cấu hình từ kho này.

Chưa bật refresh/env/busrefresh trong cấu hình cơ bản.
Refresh, rollback, failure handling thuộc phần tích hợp của Hòa.

## Demo của Nam

1. Giới thiệu file chung và riêng.
2. So sánh job-service-dev/test/prod: database, log, ddl-auto, thông điệp demo.
3. Profile chọn môi trường; label chọn phiên bản Git.
4. Dùng git log và git show minh họa lịch sử.
5. Thuận tiếp tục demo server đọc Git.

Hòa có thể sửa ats.demo.message, commit/push, restart client rồi kiểm tra.
Push Git không có nghĩa mọi ứng dụng đang chạy tự cập nhật.

## Kiểm tra

Kiểm tra cú pháp YAML, đủ 3 service x 3 profile, database dev/test tách biệt,
placeholder bắt buộc ở prod và không chứa mật khẩu literal.
Chưa chạy Config Server/ATS hoặc xác thực schema PostgreSQL.
