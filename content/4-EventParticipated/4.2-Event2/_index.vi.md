---
title: "AWS for Software & Financial Services Companies Day"
date: 2026-10-08
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---



## Thông tin sự kiện

- **Ngày tổ chức:** 08/10/2026
- **Địa điểm:** Sheraton Saigon Grand Opera
- **Đơn vị tổ chức:** Amazon Web Services (AWS)
- **Workshop đã tham gia:** Track B - Build & Deploy AI Agents on Amazon Bedrock AgentCore Using Vibe Coding with Kiro

## Tổng quan sự kiện

Sự kiện quy tụ các chuyên gia AWS và những người làm công nghệ để chia sẻ về AI Agent hướng đến production, data product chất lượng cao dành cho AI, cloud migration bằng Agentic AI và hoạt động thực hành với Kiro cùng Amazon Bedrock AgentCore. Chương trình gồm ba phiên kỹ thuật, hai workshop diễn ra song song, Customer Spotlight và phần tổng kết.

## Khai mạc

**Diễn giả:** Hung Gia - Manager, Solutions Architecture, AWS

Phần khai mạc giới thiệu chương trình và các chủ đề chính về Agentic AI, nền tảng dữ liệu hiện đại, cloud migration, modernization và hoạt động thực hành.

## Session 1 - From Demo to Deployment: Solving Agentic AI's Toughest Challenges

**Diễn giả:** Dzung Ha - Solutions Architect, AWS

Phiên chia sẻ giải thích khoảng cách giữa một bản AI proof of concept và một hệ thống sẵn sàng vận hành trên production thông qua bốn trụ cột:

- **Operational Excellence:** observability, kiểm soát chi phí, khả năng mở rộng, độ trễ và governance.
- **Data and Context:** chất lượng dữ liệu, retrieval, RAG, grounding và bộ nhớ ngắn hạn, dài hạn.
- **Reliability:** sử dụng code có tính xác định, evaluation, kiểm thử liên tục, monitoring và recovery.
- **Trust:** identity, session isolation, policy, ranh giới bảo mật, quyền riêng tư và guardrails.

Phiên này cũng giới thiệu Amazon Bedrock AgentCore Memory, AgentCore Evaluations và Amazon Bedrock Guardrails. Bài học chính là một model hoạt động được vẫn chưa đủ; Agent trên production còn cần dữ liệu đáng tin cậy, monitoring, evaluation, security và governance.

Điểm tôi thấy đặc biệt hữu ích là cách diễn giả bắt đầu từ một vấn đề quen thuộc - nhiều bản AI proof of concept không thể đi đến production - rồi phát triển thành một khung tư duy có cấu trúc. Thay vì trình bày từng dịch vụ AWS một cách riêng lẻ, diễn giả liên kết mỗi dịch vụ với một câu hỏi vận hành: hệ thống có quan sát được không, ngữ cảnh có đáng tin cậy không, hành vi có đánh giá được không và Agent có hoạt động trong ranh giới đã xác định hay không. Các ví dụ về context window, memory, chất lượng RAG, model evaluation và guardrails giúp tôi nhìn AI Agent như một hệ thống phần mềm hoàn chỉnh, không chỉ là model và prompt.

Tôi cũng học được một nguyên tắc kỹ thuật quan trọng: code có tính xác định vẫn nên được sử dụng cho phép tính, validation và logic dựa trên quy tắc; Agent chỉ nên đảm nhiệm những công việc thực sự cần reasoning. Cách tiếp cận này có thể giảm chi phí và độ trễ, đồng thời giúp hệ thống dễ kiểm thử và xử lý sự cố hơn.

## Session 2 - Building High-Quality Data Products for AI Agents

**Diễn giả:** Binh Tran - Senior Solutions Architect, AWS

Phiên này trình bày năm đặc điểm của dữ liệu sẵn sàng cho AI: trusted, contextual, traceable, governed và operational. Các nội dung chính gồm:

- Quyền sở hữu data product, operating model và yêu cầu về mức độ dịch vụ.
- Data contract bao gồm schema, quy tắc chất lượng, freshness, versioning và access.
- Kiểm soát chất lượng dữ liệu gần nguồn bằng các dịch vụ như AWS Glue Data Quality.
- Technical metadata, business metadata, lineage, semantic layer, business glossary và ontology.
- Kiểm soát truy cập chi tiết bằng IAM, AWS Lake Formation, Cedar policy và agent identity.
- Truy cập liên thông thông qua open table format và AWS MCP Servers.
- RAG, vector retrieval, graph retrieval, Text-to-SQL và guardrails trước khi Agent thay đổi hệ thống thật.

Bài học chính là Agent cần dữ liệu có ý nghĩa nghiệp vụ rõ ràng, đã được kiểm chứng, truy vết được, còn mới và được giới hạn quyền truy cập trước khi đưa ra quyết định hoặc thực hiện hành động.

Cách trình bày của diễn giả trong session này rất hữu ích vì một lỗi dữ liệu được theo dõi xuyên suốt từ source, pipeline, catalog, semantic layer, Agent đến hành động cuối cùng. Điều này làm rõ hậu quả của chất lượng dữ liệu kém: lỗi có thể không khiến hệ thống kỹ thuật dừng hoạt động nhưng vẫn làm Agent đưa ra một quyết định sai với độ tự tin cao. Ví dụ về data contract giúp tôi hiểu cách ownership, schema, ý nghĩa nghiệp vụ, quy tắc chất lượng, freshness, versioning và access cần được xác định cùng nhau thay vì xử lý như những vấn đề tách biệt.

Một số khái niệm mới hoặc trở nên rõ ràng hơn với tôi trong session này. Tôi hiểu vì sao technical metadata chưa đủ cho Agent và cách business glossary, semantic layer, ground-truth examples cùng entity relationships bổ sung những ngữ cảnh mà con người thường tự hiểu. Tôi cũng có góc nhìn rõ hơn về authorization nhiều lớp: Agent cần identity riêng, service role giới hạn, quyền được ủy quyền thay mặt người dùng, quyền truy cập dữ liệu đã được lọc và bước kiểm tra policy trước khi gọi tool hoặc API. Điều này thay đổi cách tôi nhìn việc chuẩn bị dữ liệu, từ quá trình làm sạch và transform đơn thuần thành việc thiết kế một data product có governance để Agent sử dụng an toàn.

## Session 3 - Migration at Scale with Agentic AI: AWS Transform and Beyond

**Diễn giả:** Taiki Dang - Solutions Architect, Migration & Modernize | Security, AWS

Phiên này trình bày quá trình phát triển từ migration truyền thống sang AI-assisted migration và agentic migration. Nội dung giới thiệu AWS Transform và khả năng tích hợp với Kiro, Amazon Bedrock AgentCore, Cloud Migration Factory và AWS Transform for VMware.

Ba mô hình migration được trình bày:

- **Assist:** sử dụng AI trên quy trình thực thi đã được kiểm chứng của Cloud Migration Factory.
- **Automate:** sử dụng AWS Transform xuyên suốt vòng đời migration với các bước phê duyệt của con người.
- **Orchestrate:** kết hợp AWS Transform, Kiro và AgentCore agents để thực hiện cả migration lẫn công việc điều phối.

Phiên chia sẻ cũng đề cập đến rehosting, replatforming, containerization, modernization cho Windows và mainframe, custom transformation cùng các ví dụ từ doanh nghiệp quốc tế và Việt Nam. Bài học chính là Agent có thể tự động hóa các workstream migration, trong khi con người vẫn chịu trách nhiệm cho những quyết định và phê duyệt quan trọng.

Diễn giả giúp một chủ đề migration phức tạp trở nên dễ theo dõi bằng cách giải thích giới hạn của migration truyền thống do con người điều phối, sau đó so sánh với cách tiếp cận AI-assisted và agentic. Ba mô hình Assist, Automate và Orchestrate cho tôi một cách thực tế để hiểu rằng các tổ chức không cần áp dụng cùng một mức độ tự động hóa. Cách phù hợp phụ thuộc vào yêu cầu tuân thủ, công cụ hiện có, mức độ sẵn sàng của đội ngũ và nhu cầu phê duyệt của con người, không chỉ phụ thuộc vào số lượng server hay application.

Tôi học được rằng agentic migration không đơn giản là tạo script nhanh hơn. Quá trình này còn bao gồm discovery, planning, chuẩn bị landing zone và network, truy vết worklog, approval gate, execution, validation và phối hợp giữa nhiều đội ngũ. Các ví dụ về AWS Transform đóng vai trò execution engine, Kiro là conversational interface và AgentCore agents đảm nhiệm công việc điều phối cho thấy nhiều công cụ có thể chia sẻ context, trong khi con người vẫn kiểm soát các quyết định có tác động lớn.

## Học hỏi từ diễn giả và cách tổ chức nội dung

Một trong những mục tiêu chính của tôi khi tham gia sự kiện là quan sát cách các chuyên gia AWS truyền đạt những chủ đề kỹ thuật nâng cao. Các diễn giả không bắt đầu bằng danh sách dài các dịch vụ. Họ xác định một vấn đề thực tế, giải thích vì sao cách tiếp cận hiện tại chưa đủ, rồi mới giới thiệu architecture pattern và khả năng của AWS như lời giải cho vấn đề đó. Nhờ vậy, mỗi session dễ theo dõi hơn và các chi tiết kỹ thuật đều có mục đích rõ ràng.

Tôi ghi nhận một số cách trình bày có thể áp dụng cho công việc kỹ thuật của mình:

- Bắt đầu từ rủi ro nghiệp vụ hoặc vận hành trước khi nói về công nghệ.
- Chia một chủ đề lớn thành số ít trụ cột hoặc pattern dễ ghi nhớ.
- Dùng một tình huống xuyên suốt session để người nghe theo dõi quan hệ nguyên nhân và kết quả.
- So sánh các lựa chọn và trade-off thay vì xem một dịch vụ là lời giải cho mọi vấn đề.
- Liên kết sơ đồ kiến trúc với trách nhiệm, quyền truy cập, data flow và cách xử lý lỗi.
- Kết thúc bằng các bài học cụ thể để người nghe hiểu điều gì cần thay đổi trong thực tế.

Ba session cũng liên kết với nhau một cách tự nhiên. Session 1 trình bày những yêu cầu của một Agent sẵn sàng cho production. Session 2 giải thích nền tảng dữ liệu đáng tin cậy và có governance mà Agent đó cần. Session 3 mở rộng các ý tưởng này sang một use case vận hành quy mô lớn, nơi Agent, tool, policy, approval gate và các đội ngũ con người cùng phối hợp. Trình tự này giúp tôi xây dựng hiểu biết đầy đủ hơn về Agentic AI ở cả cấp độ application và tổ chức.

## Giới thiệu workshop

Sự kiện cung cấp hai workshop diễn ra song song:

- **Track A:** Build with Kiro: From Spec to Working Application.
- **Track B:** Build & Deploy AI Agents on Amazon Bedrock AgentCore Using Vibe Coding with Kiro.

Tôi đã lựa chọn và tham gia **Track B**.

## Track B - Build & Deploy AI Agents on Amazon Bedrock AgentCore Using Vibe Coding with Kiro

**Cấp độ:** Intermediate đến Advanced

### Mục tiêu workshop

- Xây dựng một AI Agent bằng Kiro.
- Phát triển và mở rộng Agent theo từng bước hướng dẫn.
- Chuẩn bị và triển khai Agent lên Amazon Bedrock AgentCore.
- Tìm hiểu những yêu cầu cần thiết để chuyển một AI Agent từ prototype sang môi trường cloud hướng đến production.

### Tiến độ và kết quả cá nhân

Tôi đã hoàn thành khoảng một nửa nội dung Track B theo hướng dẫn. Tôi thực hành các bước đầu trong quá trình xây dựng AI Agent bằng Kiro, đồng thời tìm hiểu quy trình phát triển Agent theo từng bước và chuẩn bị Agent để triển khai lên Amazon Bedrock AgentCore.

Mặc dù chưa hoàn thành toàn bộ workshop, phần thực hành giúp tôi tiếp cận thực tế với quá trình phát triển Agent và các khái niệm triển khai hướng đến production. Kết quả này được ghi nhận là tiến độ workshop một phần và không khẳng định rằng bước triển khai cuối cùng đã hoàn thành.

### Trải nghiệm học tập qua phần thực hành

Workshop giúp chuyển các khái niệm của buổi sáng thành một quy trình phát triển thực tế. Làm việc với Kiro cho tôi thấy cách một môi trường hỗ trợ AI có thể hướng dẫn triển khai thông qua context, instruction và các task tăng dần, thay vì tạo toàn bộ giải pháp trong một bước khó kiểm soát. Tôi thực hành theo trình tự workshop, xem lại phần công việc được tạo ra và tìm hiểu cách quá trình phát triển Agent ở local được thiết kế để kết nối với Amazon Bedrock AgentCore.

Việc chỉ hoàn thành khoảng một nửa track vẫn mang lại giá trị vì giúp tôi nhận ra sự khác nhau giữa hiểu một architecture slide và trực tiếp triển khai. Mỗi giai đoạn đều phụ thuộc vào project context, chuẩn bị môi trường, cấu hình và verification chính xác. Phần chưa hoàn thành cũng tạo cho tôi một mục tiêu học tập rõ ràng hơn: xem lại các bước còn lại, hoàn tất luồng triển khai AgentCore và kiểm tra Agent sau khi deploy thay vì xem việc deploy là kết quả cuối cùng.

## Trải nghiệm học tập tổng thể

Đối với tôi, sự kiện này chủ yếu là một cơ hội học hỏi. Ngoài từng dịch vụ AWS riêng lẻ, tôi tiếp nhận một cách tư duy rộng hơn về hệ thống production: bắt đầu từ vấn đề thực tế, xác định ranh giới dữ liệu và bảo mật, làm cho hành vi có thể quan sát và kiểm thử, duy trì sự tham gia của con người trong các quyết định có tác động lớn và lựa chọn mức độ tự động hóa phù hợp với môi trường vận hành.

Sự kiện cũng giúp tôi xác định những nội dung cần tiếp tục tìm hiểu, bao gồm các thành phần AgentCore runtime, memory và evaluation, data contract, metadata và semantic layer, authorization bằng Cedar, tích hợp MCP và cách sử dụng AWS Transform cho migration và modernization. Việc quan sát cách các diễn giả liên kết những chủ đề này với các tình huống thực tế mang lại cho tôi một nguồn tham khảo hữu ích cho cả quá trình học tập và những bài trình bày kỹ thuật trong tương lai.

## Những nội dung chính học được

- AI Agent trên production cần observability, evaluation, ngữ cảnh đáng tin cậy, security và governance bên cạnh khả năng của model.
- Dữ liệu dành cho AI phải được kiểm chứng, còn mới, truy vết được, có định nghĩa rõ ràng và được bảo vệ bằng quyền truy cập phù hợp.
- Agentic AI có thể hỗ trợ cloud migration và modernization nhưng vẫn duy trì sự phê duyệt của con người đối với các quyết định quan trọng.
- Kiro và Amazon Bedrock AgentCore cung cấp một quy trình thực tế để xây dựng và chuẩn bị AI Agent cho việc triển khai trên cloud.
- Việc hoàn thành một phần Track B giúp tôi hiểu các bước phát triển Agent ban đầu và xác định những bước còn lại để hoàn tất quá trình triển khai.

## Hình ảnh sự kiện

![Ảnh check-in trước backdrop sự kiện, thứ Năm 8/10/2026, TP. Hồ Chí Minh](../../../Image/event%202/checkin.jpg)

*Check-in trước backdrop sự kiện ngày 8/10/2026.*

![Ảnh nhóm cùng quà tặng của AWS tại sự kiện](../../../Image/event%202/anh1.jpg)

*Cùng các bạn thực tập sinh sau các phiên chia sẻ, cầm quà tặng của AWS.*

![Các bạn tham gia workshop cầm hộp quà AWS trước backdrop sự kiện](../../../Image/event%202/anh2.jpg)

*Ảnh chụp chung của những người tham gia vào cuối ngày.*
