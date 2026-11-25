---
title: "Worklog Tuần 11"
date: 2026-09-13
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu tuần 11:

* Hiểu kiến thức nền tảng về web API.
* Tìm hiểu các thành phần chính của Amazon API Gateway.
* Hiểu cách API Gateway, Lambda và EventBridge hỗ trợ ứng dụng serverless.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Học API cơ bản: client/server, HTTP method, endpoint, request, response, header, status code và JSON | 07/09/2026 | 07/09/2026 | <https://aws.amazon.com/what-is/api/> |
| 3 | - Đọc và tìm hiểu khái niệm API Gateway: REST API, HTTP API, route, method, stage, deployment và tích hợp Lambda | 08/09/2026 | 08/09/2026 | <https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html> |
| 4 | - Lên văn phòng, kết nối với các bạn thực tập sinh và trao đổi về kiến trúc ứng dụng serverless <br> - Đọc tài liệu API Gateway và xem lại luồng request từ client qua API Gateway đến Lambda <br> - Tìm hiểu route, method, stage, deployment, authorization, throttling, CORS và CloudWatch logging <br> - Xem vai trò của EventBridge trong việc định tuyến event đến target và so sánh với SNS, SQS <br> - Hệ thống ghi chú kiến trúc và chuẩn bị câu hỏi cho phần API security, EventBridge ở những ngày tiếp theo | 09/09/2026 | 09/09/2026 | <https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html> |
| 5 | - Đọc và tìm hiểu khái niệm bảo vệ API cơ bản: IAM authorization, API key, throttling, CORS, input validation và CloudWatch logging | 10/09/2026 | 10/09/2026 | <https://docs.aws.amazon.com/apigateway/latest/developerguide/security.html> |
| 6 | - Tìm hiểu Amazon EventBridge: event bus, event, rule, target và schedule <br> - Phân biệt EventBridge, SNS và SQS | 11/09/2026 | 11/09/2026 | <https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html> |
| 7 | - Tham gia **Day 02: INVESTIGATE · From Signal to Root Cause** cùng Team **HCM15 (Tyrex)** <br> - Điều tra cloud environment được cố ý cấu hình sai và challenge câu trả lời đầu tiên của Agent <br> - Xây dựng competing hypotheses, kiểm chứng claim bằng evidence và truy ngược kết luận về source <br> - Xây dựng Evidence Table và Causal Graph, sau đó tổng hợp kết quả thành **Incident Report** <br> - Thực hành kỹ năng **Agentic Incident Investigator** và đạt **93,6/100 điểm** | 12/09/2026 | 12/09/2026 | <https://docs.cloudthinker.io/learn/workshops/prove-it/day-02-investigate> |

### Kết quả đạt được trong tuần 11:

* Kết nối với các bạn thực tập sinh và trao đổi các thành phần trong một kiến trúc ứng dụng serverless cơ bản.
* Đọc trước luồng request của API Gateway và kiến trúc EventBridge trước khi tiếp tục các nội dung security, integration.
* Hiểu HTTP và API cơ bản.
* Hiểu các thành phần chính của API Gateway.
* Hiểu cách API Gateway tích hợp với Lambda ở mức khái niệm.
* Hiểu cách bảo vệ và theo dõi API ở mức cơ bản.
* Hiểu ở mức cơ bản khi nào dùng EventBridge, SNS hoặc SQS.
* Hoàn thành Evidence Table, Causal Graph và Incident Report thông qua việc kiểm tra competing hypotheses, truy nguồn kết luận về evidence.
* Phát triển kỹ năng Agentic Incident Investigator và đạt **93,6/100 điểm** cho bài nộp Day 02.
