---
title: "First Cloud AI Journey: Community Day"
date: 2026-10-31
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---


## Thông tin sự kiện

- **Ngày tổ chức:** Thứ Bảy, 31/10/2026
- **Địa điểm:** Bitexco Financial Tower, Thành phố Hồ Chí Minh
- **Đơn vị tổ chức:** Cộng đồng AWS Study Group / First Cloud AI Journey (FCAJ)
- **Vai trò:** Người tham dự
- **Hình thức:** Sáu phiên chia sẻ trong buổi sáng từ diễn giả doanh nghiệp và các bạn thực tập sinh của chương trình

## Tổng quan sự kiện

Community Day là buổi gặp mặt của cộng đồng First Cloud AI Journey, nơi các kỹ sư đang đi làm và các bạn thực tập sinh chia sẻ kinh nghiệm thực tế trên AWS. Khác với hai sự kiện trước, nội dung không tập trung vào một dịch vụ hay một use case duy nhất mà trải dài theo hành trình của một người học cloud: bắt đầu học AWS, đưa AI vào quy trình phát triển phần mềm, xây dựng ứng dụng serverless, tối ưu kiến trúc web, xử lý bất đồng bộ và đóng góp mã nguồn mở.

## Session 1 - AWS Roadmap for Beginners

**Diễn giả:** Minh Tuấn - Backend Developer, thực tập sinh FCAJ

Phiên mở đầu giới thiệu lộ trình và tài nguyên để người mới tiếp cận AWS một cách có hệ thống và tiết kiệm chi phí.

- **AWS Study Group:** hands-on lab từng bước bằng tiếng Việt, được các admin cộng đồng cập nhật thường xuyên.
- **AWS Skill Builder:** học lý thuyết theo lộ trình role-based, làm lab và ôn chứng chỉ, ví dụ AWS Certified Cloud Practitioner.
- **AWS Builder Center:** đọc và viết bài kỹ thuật, tham gia hackathon và các Student Builder Group.
- **AWS Cloud Quest:** học qua game 3D với các khóa miễn phí như Cloud Practitioner và AI Foundations.
- **Ưu đãi cho sinh viên:** xác thực sinh viên để nhận gói Skill Builder Subscription (trị giá khoảng $449/năm) và voucher thi chứng chỉ khi tích lũy badge trên Builder Center; tài khoản mới có thể nhận free credit.

Diễn giả nhấn mạnh quy trình làm lab an toàn: **đọc hiểu dịch vụ → thử nghiệm → xử lý lỗi (hỏi AI hoặc admin) → luôn clean up tài nguyên** sau khi làm xong để tránh phát sinh chi phí ngoài ý muốn.

Điều tôi thấy giá trị nhất là bước clean up được xem là một phần bắt buộc của bài lab, không phải việc làm thêm. Đây là thói quen tôi cần giữ khi tự triển khai các project cá nhân trên tài khoản AWS của mình.

## Session 2 - Lessons from Building a Software Factory with AI

**Diễn giả:** Henry Đức Bùi - Head of Engineering, CloudThinker

Phiên này chia sẻ cách một đội ngũ nhỏ tổ chức quy trình phát triển phần mềm xoay quanh AI coding agent.

- **Kết quả thực tế:** đội 5-6 người xử lý gần 3.000 merge request (MR) trong một tháng, khoảng 100 MR mỗi ngày, phần lớn do AI thực thi và merge.
- **Điểm nghẽn mới là con người:** khi ngân sách token không còn là giới hạn, giới hạn chuyển sang khả năng review và approve của con người khi nhiều agent chạy song song.
- **Thảo luận trước khi code:** khoảng 60% thời gian dành cho thảo luận có AI tham gia; kết quả được AI xuất thành dashboard hoặc diagram dạng HTML để cả nhóm review và chốt hướng đi.
- **Divergent prompting:** tránh prompt quá hẹp, để AI có không gian đề xuất giải pháp khác (ví dụ thêm câu "feel free to advise me").
- **Pattern "Folder Control":** một folder điều phối chung chứa file `agent.md` với chỉ dẫn cơ bản để agent làm việc trên nhiều repository và server.
- **Tự động hóa review và merge:** phần lớn MR nhỏ như bảo trì, refactor, kiểm tra security và tối ưu performance được giao cho agent tự kiểm tra, tranh luận và merge.
- **Xác minh bằng bằng chứng:** yêu cầu AI chạy end-to-end test, chụp màn hình before/after hoặc quay lại quá trình chạy thử.
- **Tối ưu CI/CD:** chạy CI trên máy local hoặc server riêng cấu hình mạnh, cache kết quả test (giảm khoảng 76% thời gian chạy test) và dùng chung môi trường dependency.

Phiên này liên kết trực tiếp với nguyên tắc **evidence over confidence** tôi đã học ở chuỗi Prove It (Event 1): đầu ra của AI chỉ được chấp nhận khi có test, ảnh chụp hoặc video chứng minh. Tôi cũng nhận ra rằng khi AI viết code nhanh hơn, kỹ năng review, viết yêu cầu rõ ràng và thiết kế quy trình kiểm chứng trở nên quan trọng hơn.

## Session 3 - Three-Month Internship Journey: Building a Real-time Serverless Web Game

**Diễn giả:** Phúc Khang - thực tập sinh FCAJ (Công nghệ phần mềm)

Diễn giả trình bày dự án web game thẻ bài multiplayer real-time được xây dựng trong ba tháng thực tập.

- **Tech stack:** AWS Lambda, Amazon DynamoDB, Amazon SQS, Amazon EventBridge, Amazon API Gateway (WebSocket và HTTP), Amazon Cognito, AWS Amplify và AWS CDK.
- **Bốn serverless pattern được áp dụng:**
  - **Synchronous:** API Gateway + Lambda + DynamoDB cho xác thực và đọc/ghi dữ liệu cơ bản.
  - **Scheduled job:** cập nhật bảng xếp hạng định kỳ.
  - **Asynchronous filtering:** EventBridge xử lý sự kiện bất đồng bộ.
  - **Message queue:** SQS làm buffer cho kết quả trận đấu, tránh quá tải database khi nhiều kết quả đổ về cùng lúc.

**Bài học từ sự cố thực tế:**

- Dùng SQS để đếm giờ lượt chơi khiến cơ chế polling giữa Lambda và SQS tạo ra khoảng 863.000 request dù không có người chơi, làm vượt Free Tier. Nhóm khắc phục bằng cách chuyển sang dùng EventBridge làm "watchdog" theo dõi timeout và trạng thái AFK.
- Không gom mọi quyền vào một IAM role duy nhất vì vi phạm nguyên tắc least privilege.
- Không lưu secret trực tiếp trong environment variable của Lambda; dùng AWS Systems Manager Parameter Store hoặc AWS Secrets Manager kết hợp AWS KMS.
- Chuyển từ thao tác thủ công trên AWS Console sang Infrastructure as Code với AWS CDK và AWS CloudFormation.

Đây là phiên tôi thấy gần với bản thân nhất vì cùng vai trò thực tập sinh. Sự cố 863.000 request cho thấy một dịch vụ dùng sai mục đích có thể gây chi phí dù hệ thống không có người dùng, và việc đặt budget alert, theo dõi metric ngay từ đầu là cần thiết cho cả project nhỏ.

## Session 4 - Re-architecting Web Applications with AWS Lambda Web Adapter

**Diễn giả:** Thịnh Nguyễn - DevOps / Cloud Engineer, Style Solution

Phiên này so sánh cách chạy web application trên Lambda truyền thống với mô hình đóng gói toàn bộ web app thành container image và chạy trên Lambda thông qua Lambda Web Adapter.

**Hạn chế của cách dùng Lambda truyền thống cho web app:**

- Mỗi request chiếm một execution environment, dễ gây cold start hàng loạt khi traffic tăng đột ngột.
- Phụ thuộc API Gateway, làm tăng latency và chi phí.
- Tính phí cả thời gian function chờ database phản hồi I/O.

**Cách tiếp cận được đề xuất:**

- Đóng gói web app (Express.js, FastAPI, ...) thành Docker container image (tối đa 10 GB) và deploy lên Lambda.
- Theo diễn giả, một execution environment có thể xử lý nhiều request đồng thời (đến 128 request), nên cold start chủ yếu xảy ra ở request đầu tiên.
- Định tuyến từ Amazon CloudFront đến Lambda Function URL bằng Origin Access Control (OAC), không cần API Gateway và không mở endpoint ra public internet.
- Diễn giả ước tính có thể giảm khoảng 40-70% chi phí compute nhờ không tính phí cho thời gian chờ I/O.

**Tiêu chí lựa chọn:**

- **Lambda truyền thống:** phù hợp với tác vụ nhỏ, đơn lẻ như tạo S3 presigned URL, gửi thông báo, cron job.
- **Lambda với Web Adapter:** phù hợp với full-stack web application hoặc API backend hoàn chỉnh.

Phiên này giúp tôi hiểu rằng serverless không có nghĩa là mỗi route phải là một function riêng. Với backend .NET, tôi có thể cân nhắc đóng gói cả ASP.NET Core API thành container image và so sánh trade-off với cách tách từng function.

## Session 5 - Asynchronous Processing with Amazon SQS

**Diễn giả:** Cẩm Đào - thực tập sinh FCAJ

Phiên này giải thích vì sao các tác vụ tốn thời gian như gọi model AI nên được chuyển từ xử lý đồng bộ sang bất đồng bộ.

- **Đồng bộ:** người dùng gọi API và phải chờ đến khi Amazon Bedrock sinh xong kết quả.
- **Bất đồng bộ:** người dùng gửi job → API trả về trạng thái `Pending` ngay lập tức → job được lưu trong SQS → AI worker lấy job từ queue và xử lý phía sau.

**Lựa chọn loại queue:**

- **Standard queue:** throughput rất cao nhưng không đảm bảo thứ tự message.
- **FIFO queue:** đảm bảo thứ tự xử lý theo message group ID, đổi lại throughput thấp hơn.

**Kỹ thuật xử lý message an toàn:**

- **Visibility timeout và heartbeat:** khi consumer nhận message, message bị ẩn trong khoảng visibility timeout; với tác vụ dài, consumer cần gia hạn timeout để tránh consumer khác xử lý trùng.
- **Long polling:** đặt thời gian chờ (ví dụ 20 giây) khi queue rỗng để giảm số request thừa và tiết kiệm chi phí.
- **Dead-letter queue (DLQ):** lưu các message lỗi sau khi vượt số lần retry để kiểm tra lại, không làm nghẽn luồng chính.
- **Fan-out với SNS + SQS:** một message gửi vào Amazon SNS được phân phối đồng thời đến nhiều SQS queue, phù hợp cho logging hoặc xử lý song song.

Nội dung này áp dụng trực tiếp cho các project logistics của tôi, nơi các sự kiện vận chuyển cần được xử lý bất đồng bộ. Tôi ghi lại ba điểm cần kiểm tra khi dùng SQS: visibility timeout phải dài hơn thời gian xử lý, phải có DLQ, và consumer cần idempotent vì message có thể được giao lại.

## Session 6 - Staying Persistent: Open Source Contribution with AWS CDK

**Diễn giả:** Hưng Trần - Senior Cloud Engineer, NAB; Top 25 AWS CDK Contributor

Phiên cuối chia sẻ thực tế của việc đóng góp mã nguồn mở cho các dự án lớn như Kubernetes, Terraform và AWS CDK, nơi một pull request (PR) có thể mất từ 6 đến 8 tháng để được duyệt.

**Lộ trình ba giai đoạn cho contributor:**

- **0 → 1 PR:** tìm issue có nhãn `good first issue` hoặc tham gia các đợt refactor lớn đã có sẵn template và checklist từ maintainer; giữ PR nhỏ và đơn giản.
- **1 → 20 PR:** thử nhiều repository để tìm dự án và đội maintainer phù hợp.
- **20+ PR:** tự tìm việc để làm, ví dụ theo dõi trang AWS What's New mỗi ngày và cập nhật CDK, Terraform hoặc Pulumi khi AWS ra mắt tính năng hoặc region mới.

**Tư duy cốt lõi:** dành khoảng 15 phút mỗi ngày cho những việc nhỏ như nâng cấp phiên bản, và không chờ đọc hết tài liệu hay hiểu toàn bộ kiến trúc mới bắt đầu; cách học nhanh nhất là sửa một issue cụ thể.

Phiên này thay đổi cách tôi nhìn về open source: không cần bắt đầu bằng một đóng góp lớn, mà cần sự đều đặn và kiên nhẫn. Đây cũng là một cách thực tế để xây dựng portfolio có thể kiểm chứng công khai.

## Học hỏi từ diễn giả và cách tổ chức nội dung

Điểm khác biệt của Community Day là một nửa số diễn giả là thực tập sinh. Các phiên của thực tập sinh tập trung vào sự cố cụ thể, con số đo được và cách khắc phục, trong khi các phiên của kỹ sư doanh nghiệp đưa ra góc nhìn về quy trình, chi phí và tư duy dài hạn. Hai góc nhìn này bổ sung cho nhau.

Một số cách trình bày tôi muốn áp dụng khi chia sẻ project của mình:

- Kể lại sự cố thật kèm con số cụ thể (số request, chi phí, thời gian) thay vì chỉ mô tả kiến trúc lý tưởng.
- So sánh "trước và sau" khi thay đổi kiến trúc để người nghe thấy rõ trade-off.
- Đưa ra tiêu chí lựa chọn rõ ràng (khi nào dùng cách A, khi nào dùng cách B) thay vì khẳng định một giải pháp luôn đúng.
- Kết thúc bằng hành động nhỏ người nghe có thể làm ngay.

## Trải nghiệm học tập tổng thể

Sự kiện giúp tôi nhìn lại toàn bộ hành trình học cloud từ góc nhìn của những người đi trước một bước. Các phiên về serverless, Lambda Web Adapter và SQS cung cấp pattern kỹ thuật có thể áp dụng ngay cho project hiện tại; phiên về Software Factory cho thấy cách AI đang thay đổi quy trình làm việc của một đội kỹ thuật; còn phiên về open source và roadmap cho người mới nhắc tôi rằng tiến bộ đến từ những việc nhỏ được làm đều đặn.

Sau sự kiện, tôi xác định một số nội dung cần thực hành thêm: thử Lambda Web Adapter với một API .NET đóng gói container, bổ sung DLQ và cấu hình visibility timeout cho các queue trong project, rà soát IAM role và cách lưu secret, và tìm một `good first issue` trên repository AWS CDK để bắt đầu đóng góp.

## Những nội dung chính học được

- Luôn clean up tài nguyên và theo dõi chi phí khi làm lab hoặc project cá nhân trên AWS.
- AI có thể tăng mạnh năng suất phát triển, nhưng đầu ra của AI cần được kiểm chứng bằng test và bằng chứng cụ thể.
- Chọn dịch vụ theo đúng mục đích: dùng SQS sai cách có thể tạo hàng trăm nghìn request không cần thiết; EventBridge phù hợp hơn cho việc theo dõi thời gian.
- Áp dụng least privilege cho IAM, lưu secret trong Parameter Store hoặc Secrets Manager và quản lý hạ tầng bằng IaC.
- Lambda Web Adapter là lựa chọn đáng cân nhắc khi cần chạy toàn bộ web app trên Lambda với chi phí và cold start thấp hơn.
- Xử lý bất đồng bộ với SQS cần visibility timeout phù hợp, long polling, DLQ và consumer idempotent.
- Đóng góp open source bắt đầu từ PR nhỏ và cần sự kiên trì trong nhiều tháng.

## Hình ảnh sự kiện

![Ảnh check-in cùng các bạn thực tập sinh mặc áo First Cloud AI Journey, đeo thẻ khách, trong hội trường tại Bitexco Financial Tower](../../../Image/event%203/checkin.jpg)

*Check-in cùng các bạn thực tập sinh FCAJ tại Community Day ngày 31/10/2026.*
