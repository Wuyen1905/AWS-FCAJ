---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Mình từng hardcode API key trong ECS Task Definition – Cách khắc phục bằng Parameter Store

### Thông tin bài viết

* **Trạng thái:** Cộng đồng AWS Study Group VN
* **Nền tảng dự kiến:** Cộng đồng AWS Study Group VN
* **Ngày đăng:** ngày 05/08/2026
* **Liên kết bài đăng:** https://www.facebook.com/groups/awsstudygroupfcj/permalink/2235479093883717/?rdid=0EI9A3H3rdsnAmIa#
* **Chủ đề:** Quản lý Gemini API key an toàn khi triển khai container trên Amazon ECS Fargate
* **Từ khóa:** Amazon ECS, AWS Fargate, Task Definition, Parameter Store, SecureString, AWS KMS, IAM, secret management

### Bối cảnh của bài viết

Trong quá trình triển khai fcaj-rag-chat, một RAG chatbot được tùy biến từ Kotaemon, lên Amazon ECS Fargate, container backend cần Gemini API key để gọi chat model và embedding model. Cách cấu hình nhanh nhất là đặt trực tiếp key vào phần <code>environment</code> của container definition:

    {
      "name": "GEMINI_API_KEY",
      "value": "actual-secret-value"
    }

Cấu hình này giúp ứng dụng chạy được nhưng tạo ra một lỗ hổng nghiêm trọng: secret trở thành dữ liệu plaintext trong ECS task definition. Người có quyền xem task definition hoặc các file cấu hình liên quan có thể đọc key; giá trị cũng có nguy cơ bị sao chép vào repository, CI/CD log, ảnh chụp màn hình hoặc tài liệu báo cáo.

Nhóm quyết định loại bỏ secret khỏi task definition và sử dụng AWS Systems Manager Parameter Store với kiểu SecureString. Task definition chỉ lưu ARN của parameter, còn ECS lấy giá trị thật khi task khởi động.

### Vì sao không nên hardcode secret?

#### 1. Phạm vi lộ thông tin rộng

Task definition không phải nơi lưu bí mật. Plaintext environment variable có thể xuất hiện khi task definition được mô tả qua Console, API hoặc CLI. Nếu JSON được gửi cho thành viên khác để review, key cũng bị chia sẻ theo.

#### 2. Khó rotate và dễ bỏ sót

ECS task definition là tài nguyên có revision. Khi key thay đổi, nhóm phải tạo revision mới và cập nhật service. Giá trị cũ vẫn có thể tồn tại trong revision, file cấu hình hoặc log trước đó nếu không được xử lý.

#### 3. Khó áp dụng least privilege và audit

Khi secret nằm trực tiếp trong cấu hình, nhóm khó tách quyền “được triển khai ứng dụng” khỏi quyền “được đọc secret”. Parameter Store cho phép kiểm soát quyền theo ARN hoặc đường dẫn parameter và ghi nhận các API call thông qua cơ chế audit của AWS.

#### 4. Rủi ro không chỉ nằm ở network

Security Group, private subnet và Application Load Balancer giúp bảo vệ network flow nhưng không bảo vệ một API key đã được ghi plaintext trong cấu hình. Bảo mật cần bao phủ cả network, identity, secret, image, log và dữ liệu.

### Việc cần làm ngay nếu key từng bị hardcode

Chuyển key sang Parameter Store không làm giá trị cũ “chưa từng bị lộ”. Trước khi triển khai cấu hình mới, nhóm cần:

1. Thu hồi hoặc rotate Gemini API key cũ tại provider.
2. Tạo key mới và chỉ lưu trong secret store phù hợp.
3. Xóa key khỏi source code, file <code>.env</code> được commit, CI/CD variable không an toàn và tài liệu chia sẻ.
4. Kiểm tra Git history, build log, CloudWatch Logs, ảnh chụp và ECS task definition revisions có thể chứa giá trị cũ.
5. Dừng task đang sử dụng key cũ và triển khai task mới.
6. Theo dõi bất thường về quota hoặc request để phát hiện key có thể đã bị sử dụng trái phép.

Nếu secret từng được commit vào Git, chỉ xóa ở commit mới là chưa đủ; cần rotate key trước, sau đó xử lý lịch sử repository theo quy trình của nhóm.

### Kiến trúc sau khi thay đổi

![So sánh hardcode API key và sử dụng AWS Systems Manager Parameter Store](/images/3-BlogsPosted/3.3-Blog3/ecs-parameter-store-securestring.png)

*Bên trái: API key được lưu plaintext trong task definition. Bên phải: task definition chỉ tham chiếu ARN của Parameter Store SecureString được bảo vệ bằng AWS KMS.*

Luồng mới gồm:

1. Gemini API key được lưu dưới dạng <code>SecureString</code> trong Parameter Store.
2. Parameter Store sử dụng AWS KMS để mã hóa giá trị khi lưu trữ.
3. ECS Task Definition khai báo ARN parameter trong trường <code>secrets</code>, không chứa giá trị thật.
4. Khi task khởi động, ECS/Fargate dùng Task Execution Role để lấy và giải mã parameter.
5. ECS inject giá trị vào container dưới dạng biến môi trường <code>GEMINI_API_KEY</code>.

### Quy trình cấu hình chi tiết

#### Bước 1: Tạo SecureString parameter

Nhóm tạo parameter theo cấu trúc phân cấp để thể hiện ứng dụng, môi trường và mục đích:

<code>/fcaj-rag-chat/prod/gemini-api-key</code>

Các thuộc tính quan trọng:

* **Type:** <code>SecureString</code>.
* **Tier:** Standard phù hợp với một API key có kích thước nhỏ; cần kiểm tra pricing và quota hiện hành.
* **KMS key:** Có thể dùng AWS managed key <code>aws/ssm</code> hoặc customer managed symmetric KMS key nếu cần kiểm soát key policy và audit chi tiết hơn.
* **Description/Tags:** Ghi rõ owner, environment, application và mục đích sử dụng nhưng không đưa secret vào tag hoặc description.

Không nên đưa giá trị thật vào shell history, log pipeline hoặc ảnh chụp khi tạo parameter. Với quy trình tự động, secret cần đi qua kênh nhập bảo mật của hệ thống CI/CD.

#### Bước 2: Cấp quyền cho Task Execution Role

Khi ECS inject Parameter Store value qua trường <code>secrets</code>, Task Execution Role được ECS agent/Fargate sử dụng để lấy secret trước khi container khởi động. Role này cần quyền đọc đúng parameter:

    {
      "Effect": "Allow",
      "Action": "ssm:GetParameters",
      "Resource": "arn:aws:ssm:<region>:<account-id>:parameter/fcaj-rag-chat/prod/gemini-api-key"
    }

Nếu parameter dùng customer managed KMS key, role còn cần <code>kms:Decrypt</code> trên đúng key và key policy phải cho phép role sử dụng key:

    {
      "Effect": "Allow",
      "Action": "kms:Decrypt",
      "Resource": "arn:aws:kms:<region>:<account-id>:key/<key-id>"
    }

Không nên cấp <code>Resource: "*"</code> nếu có thể giới hạn theo parameter ARN và KMS key ARN.

#### Bước 3: Phân biệt Task Execution Role và Task Role

| Role | Được sử dụng bởi | Mục đích trong tình huống này |
| --- | --- | --- |
| **Task Execution Role** | ECS agent/Fargate infrastructure | Pull image, gửi log và lấy Parameter Store secret trước khi container chạy |
| **Task Role** | Mã ứng dụng bên trong container | Gọi AWS API trong lúc ứng dụng đang chạy |

Với cơ chế inject qua <code>secrets</code>, quyền <code>ssm:GetParameters</code> thuộc Task Execution Role. Nếu ứng dụng được viết để tự gọi Parameter Store tại runtime, quyền đọc parameter phải gắn vào Task Role thay vì dựa vào execution role.

#### Bước 4: Tham chiếu parameter trong Task Definition

Nhóm xóa API key khỏi <code>environment</code> và khai báo trong <code>secrets</code>:

    {
      "name": "GEMINI_API_KEY",
      "valueFrom": "arn:aws:ssm:<region>:<account-id>:parameter/fcaj-rag-chat/prod/gemini-api-key"
    }

Nếu parameter cùng Region với task, ECS cho phép dùng tên hoặc ARN; dùng ARN đầy đủ giúp cấu hình rõ ràng hơn. Parameter Store là dịch vụ theo Region, vì vậy cần kiểm tra Region của task, parameter và KMS key.

#### Bước 5: Đăng ký revision và triển khai task mới

Sau khi cập nhật task definition:

1. Đăng ký task definition revision mới.
2. Cập nhật ECS service sử dụng revision mới.
3. Chọn Force new deployment để tạo task mới.
4. Theo dõi ECS service events và CloudWatch Logs.
5. Xác nhận health check thành công và Gemini request hoạt động.
6. Dừng task cũ sau khi task mới ổn định.

Parameter được inject tại thời điểm container bắt đầu chạy. Nếu giá trị SecureString thay đổi, task đang chạy không tự nhận key mới; cần launch task mới hoặc force new deployment.

### Kết quả sau khi chuyển đổi

* Task definition chỉ hiển thị parameter ARN, không còn hiển thị plaintext Gemini API key.
* Quyền đọc secret được tách khỏi quyền chỉnh sửa task definition và giới hạn bằng IAM.
* Khi rotate key, nhóm cập nhật một parameter rồi force new deployment thay vì sửa giá trị thật trong task definition.
* Task definition JSON có thể được review an toàn hơn vì không chứa secret thật.
* Tên parameter phân cấp giúp quản lý theo application và environment.
* Việc sử dụng KMS và IAM tạo nền tảng tốt hơn cho audit và kiểm soát truy cập.

### Giới hạn và lưu ý bảo mật

Parameter Store SecureString cải thiện đáng kể cách lưu secret nhưng không làm secret “vô hình” trong mọi giai đoạn:

* Sau khi inject, giá trị tồn tại dưới dạng plaintext environment variable bên trong container và có thể bị đọc bởi process hoặc công cụ có quyền truy cập phù hợp.
* Không được log toàn bộ environment, request header hoặc exception có chứa API key.
* Quyền ECS Exec, quyền debug container và quyền xem log phải được giới hạn.
* Với ECS Fargate, cần dùng platform version hỗ trợ secret injection; nên sử dụng phiên bản hiện hành phù hợp.
* Có thể tạo VPC interface endpoint cho Systems Manager để task private truy cập dịch vụ mà không đi qua public internet.
* KMS encryption bảo vệ dữ liệu at rest; IAM, network, logging và runtime isolation vẫn phải được cấu hình đúng.

Với yêu cầu nghiêm ngặt hơn, AWS khuyến nghị cân nhắc sidecar ghi secret vào volume tạm của task hoặc để ứng dụng đọc secret bằng SDK tại runtime, thay vì giữ secret lâu dài trong environment variable.

### Parameter Store hay Secrets Manager?

| Tiêu chí | Parameter Store SecureString | AWS Secrets Manager |
| --- | --- | --- |
| Lưu key/config được mã hóa | Có | Có |
| Phân cấp tên parameter | Có | Không theo cùng mô hình path |
| Automatic rotation tích hợp | Không phải chức năng chính | Có |
| Random secret generation | Không phải chức năng chính | Có |
| Cross-account secret sharing | Hạn chế hơn | Được thiết kế hỗ trợ tốt hơn |
| Phù hợp | Secret/config đơn giản, cập nhật thủ công | Secret cần lifecycle và rotation nâng cao |

Với Gemini API key chỉ cần lưu, đọc và rotate thủ công, Parameter Store SecureString có thể đáp ứng yêu cầu. Nếu cần rotation tự động, quản lý version/lifecycle nâng cao hoặc chia sẻ cross-account, Secrets Manager phù hợp hơn. Nhóm vẫn cần kiểm tra pricing và quota hiện hành trước khi lựa chọn.

### Checklist kiểm thử

* Xác nhận task definition mới không chứa API key plaintext.
* Xác nhận Task Execution Role chỉ có <code>ssm:GetParameters</code> trên parameter cần thiết.
* Nếu dùng customer managed KMS key, kiểm tra <code>kms:Decrypt</code> và key policy.
* Kiểm tra task mới khởi động thành công và không có lỗi <code>AccessDeniedException</code>.
* Xác nhận container nhận được biến <code>GEMINI_API_KEY</code> nhưng ứng dụng không in giá trị ra log.
* Rotate parameter thử nghiệm, force new deployment và xác nhận task mới nhận giá trị cập nhật.
* Xác nhận task cũ vẫn giữ giá trị cũ cho đến khi bị thay thế.
* Kiểm tra service rollback/health check khi parameter ARN sai hoặc role thiếu quyền.
* Dò repository, task definition revision, CI/CD log và tài liệu để bảo đảm key cũ không còn được sử dụng.
* Thu hồi key cũ và theo dõi usage bất thường.

### Những bài học nhóm rút ra

* **Chạy được chưa có nghĩa an toàn:** secret management phải được kiểm tra trước khi chia sẻ cấu hình hoặc triển khai.
* **Task Execution Role và Task Role có mục đích khác nhau:** gắn quyền sai role là nguyên nhân phổ biến khiến task không khởi động hoặc quyền bị cấp quá rộng.
* **Mã hóa phải đi cùng kiểm soát truy cập:** KMS bảo vệ dữ liệu at rest, còn IAM quyết định ai được phép giải mã.
* **Rotate secret đã lộ là bắt buộc:** di chuyển key sang nơi an toàn không thu hồi bản plaintext đã xuất hiện trước đó.
* **Secret update cần redeploy:** container đang chạy không tự nhận giá trị mới từ Parameter Store.
* **Chọn dịch vụ theo lifecycle:** Parameter Store phù hợp với nhu cầu đơn giản; Secrets Manager phù hợp hơn khi cần automatic rotation và quản trị secret nâng cao.

### Liên kết và tài liệu tham khảo

* [Pass sensitive data to an Amazon ECS container](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/specifying-sensitive-data.html)
* [Pass Systems Manager parameters through ECS environment variables](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-ssm-paramstore.html)
* [Parameter Store SecureString parameters](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-securestring.html)
* [KMS encryption for SecureString parameters](https://docs.aws.amazon.com/systems-manager/latest/userguide/secure-string-parameter-kms-encryption.html)
* [Creating a Parameter Store parameter using AWS CLI](https://docs.aws.amazon.com/systems-manager/latest/userguide/param-create-cli.html)
* [AWS Systems Manager Parameter Store overview](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)

> **Kết luận:** Loại bỏ API key khỏi ECS task definition là một thay đổi nhỏ nhưng quan trọng. Parameter Store SecureString, AWS KMS và IAM least privilege giúp giảm nguy cơ lộ secret, đơn giản hóa việc cập nhật key và làm cấu hình dễ review hơn. Tuy nhiên, giải pháp chỉ hoàn chỉnh khi nhóm rotate key cũ, kiểm soát quyền runtime, tránh ghi secret vào log và triển khai task mới mỗi khi giá trị thay đổi.
