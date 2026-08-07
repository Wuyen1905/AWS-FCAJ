---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# “Dính” lỗi HTTP 429 khi gọi Gemini API trên AWS – Và cách nhóm mình khắc phục

### Thông tin bài viết

* **Nền tảng đăng:** Cộng đồng AWS Study Group VN
* **Người đăng đại diện nhóm:** Phan Thị Hải Vân
* **Thời gian đăng:** 22:42 ngày 01/08/2026
* **Liên kết bài đăng:** [Xem bài viết trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2230866751011618/)
* **Chủ đề:** Xử lý HTTP 429 khi ứng dụng RAG trên AWS gọi Gemini API
* **Từ khóa:** HTTP 429, Gemini API, rate limiting, exponential backoff, jitter, Amazon EC2, Amazon Bedrock, CloudWatch

### Bối cảnh của bài viết

Bài viết mô tả tình huống một chatbot RAG chạy trên Amazon EC2 và gọi Gemini API để xử lý yêu cầu AI. Khi thử nghiệm trên local với một người dùng, số request phát sinh mỗi phút còn thấp nên ứng dụng hoạt động bình thường. Sau khi đưa ứng dụng lên EC2 để nhiều thành viên kiểm thử đồng thời, nhiều worker gửi request trong cùng một khoảng thời gian ngắn và Gemini API bắt đầu trả về:

<code>HTTP 429 – Too Many Requests / RESOURCE_EXHAUSTED</code>

Lỗi này không có nghĩa EC2 bị hỏng. Nó cho biết phía API từ chối request do hệ thống đã chạm một giới hạn áp dụng cho project hoặc API key, chẳng hạn số request, lượng token hoặc mức sử dụng theo usage tier. Vì các worker cùng sử dụng chung một quota, tải đồng thời có thể làm giới hạn bị chạm nhanh hơn nhiều so với lúc kiểm thử local.

### Vì sao local chạy ổn nhưng lên EC2 lại gặp lỗi?

| Môi trường | Đặc điểm tải | Khả năng chạm rate limit |
| --- | --- | --- |
| **Local** | Một người dùng, ít worker, request phân tán | Thấp hơn |
| **EC2 test chung** | Nhiều người dùng, nhiều worker hoặc tiến trình chạy song song | Cao hơn |

Những nguyên nhân nhóm cần kiểm tra gồm:

* Số request đồng thời tăng đột biến khi nhiều người cùng sử dụng.
* Nhiều worker cùng chia sẻ một Gemini API key và một quota.
* Mỗi thao tác RAG có thể tạo nhiều lời gọi model hoặc embedding, không chỉ một request từ trình duyệt.
* Retry tức thời hoặc retry ở nhiều tầng làm lưu lượng tăng thêm, tạo thành retry storm.
* Prompt, context hoặc tài liệu quá lớn làm tốc độ sử dụng token tăng nhanh.
* Quota hoặc usage tier của project thấp hơn tải thử nghiệm thực tế.

### Hệ quả nếu không xử lý đúng

Nếu ứng dụng chỉ bắt lỗi chung hoặc retry không giới hạn, HTTP 429 có thể kéo theo nhiều vấn đề:

* Pipeline bị dừng giữa bước đọc file, gọi API, tạo embedding hoặc ghi dữ liệu.
* Worker đang xử lý có thể thất bại và làm mất trạng thái của tác vụ.
* Retry đồng loạt khiến API tiếp tục quá tải, tăng độ trễ và tiêu tốn tài nguyên EC2.
* Cùng một tác vụ có thể bị ghi trùng nếu thao tác retry không được thiết kế idempotent.
* Người dùng chỉ nhận lỗi HTTP 500 chung chung và không biết cần chờ hay thử lại sau.
* Nhóm khó xác định lỗi xảy ra ở provider, network, worker hay tầng ứng dụng nếu không có log và metric.

### Giải pháp nhóm đề xuất

#### 1. Retry có giới hạn với Exponential Backoff và Jitter

Thay vì retry ngay lập tức, ứng dụng tăng dần thời gian chờ sau mỗi lần gặp lỗi tạm thời. Bài viết minh họa các khoảng chờ cơ sở:

| Lần thất bại | Thời gian chờ cơ sở |
| --- | --- |
| Lần 1 | 2 giây |
| Lần 2 | 4 giây |
| Lần 3 | 8 giây |

Mỗi khoảng chờ được cộng thêm một lượng ngẫu nhiên nhỏ gọi là jitter. Ví dụ, thay vì tất cả worker cùng retry chính xác sau 4 giây, mỗi worker sẽ thử lại tại một thời điểm hơi khác nhau. Điều này giúp tránh hiện tượng thundering herd, khi nhiều tiến trình đồng loạt gửi lại request và tạo ra một đợt nghẽn mới.

Một retry policy an toàn cần có:

* Chỉ retry lỗi tạm thời như HTTP 429, timeout hoặc một số lỗi 5xx.
* Không retry lỗi xác thực, sai API key, payload không hợp lệ hoặc lỗi permission cho đến khi cấu hình được sửa.
* Tôn trọng <code>Retry-After</code> nếu provider trả về hướng dẫn thời gian chờ.
* Đặt số lần thử tối đa hoặc tổng time budget cho toàn bộ request.
* Chỉ triển khai retry tại một tầng phù hợp, tránh SDK, worker và API gateway cùng retry một request.
* Ghi lại attempt number, delay, error code và request ID để troubleshooting.

Retry chỉ giúp xử lý lỗi tạm thời. Nếu quota đã cạn hoặc tải liên tục vượt giới hạn, retry nhiều hơn sẽ làm tình hình xấu đi.

#### 2. Kiểm soát tải bằng giới hạn đồng thời, Batching và Chunking

Để giảm áp lực lên API, nhóm đề xuất kiểm soát lượng công việc trước khi request được gửi đi:

* **Concurrency limit:** Giới hạn số worker được phép gọi Gemini đồng thời bằng semaphore, worker pool hoặc request queue.
* **Batching:** Gom nhiều mục nhỏ vào một request khi API và use case cho phép, nhờ đó giảm tổng số lần gọi.
* **Chunking:** Chia dữ liệu lớn thành phần có kích thước phù hợp với giới hạn input/token và khả năng xử lý.
* **Payload optimization:** Loại bỏ trường dữ liệu, context hoặc lịch sử hội thoại không cần thiết.
* **Caching/deduplication:** Không gọi lại model cho những nội dung giống nhau đã có kết quả hợp lệ.

Cần phân biệt rằng batching có thể giảm số request, trong khi chunking chủ yếu kiểm soát kích thước dữ liệu. Nếu chia quá nhiều chunk nhưng không giới hạn concurrency, tổng số request có thể tăng và làm lỗi 429 xuất hiện thường xuyên hơn.

#### 3. Bảo vệ trạng thái và tính idempotent

Một tác vụ RAG thường gồm nhiều bước: đọc tài liệu, chunking, tạo embedding, gọi model và ghi dữ liệu. Hệ thống nên lưu trạng thái của từng bước để có thể tiếp tục hoặc thử lại mà không thực hiện trùng toàn bộ pipeline.

Các thao tác ghi dữ liệu cần có idempotency key, unique constraint hoặc cơ chế kiểm tra trạng thái trước khi ghi. Việc này giúp cùng một task được retry nhiều lần nhưng không tạo bản ghi, index hoặc kết quả trùng lặp.

#### 4. Phương án dự phòng bằng Amazon Bedrock

Sau khi vượt quá số lần retry cho phép, bài viết đề xuất chuyển request sang Amazon Bedrock như một provider dự phòng:

![Kiến trúc retry Gemini API và fallback sang Amazon Bedrock](/images/3-BlogsPosted/3.2-Blog2/http-429-gemini-bedrock-fallback.png)

*EC2 gọi Gemini API; lỗi HTTP 429 được retry với exponential backoff. Khi vượt quá số lần retry, hệ thống có thể chuyển sang model dự phòng trên Amazon Bedrock.*

Để fallback hoạt động đúng, ứng dụng cần:

1. Xây dựng một provider interface chung để chuẩn hóa input và output giữa Gemini và Bedrock.
2. Chọn model Bedrock có khả năng đáp ứng use case và kiểm thử lại chất lượng câu trả lời.
3. Cấu hình Region, model access và IAM permission tối thiểu như <code>bedrock:InvokeModel</code>.
4. Chuyển đổi prompt, tham số inference và định dạng response phù hợp với model đích.
5. Chỉ fallback với lỗi được phân loại là tạm thời hoặc provider unavailable; không dùng fallback để che lỗi dữ liệu hay lỗi lập trình.
6. Theo dõi riêng số lần fallback, latency, chi phí và chất lượng kết quả.

Bedrock cũng có quota, giới hạn và chi phí riêng. Vì vậy, đây là phương án tăng khả năng phục hồi và giảm phụ thuộc vào một provider, không phải giải pháp “không giới hạn”.

#### 5. Theo dõi lỗi bằng Amazon CloudWatch

Nhóm đưa application log từ EC2 vào CloudWatch Logs, sau đó tạo metric hoặc Log Alarm để theo dõi:

* Số lỗi HTTP 429 và timeout.
* Tổng số retry và số request vượt quá max retries.
* Tỷ lệ request thành công sau retry.
* Số lần chuyển sang Bedrock.
* Latency của Gemini và Bedrock.
* Số task thất bại hoặc còn nằm trong queue.

Khi metric vượt ngưỡng trong một khoảng thời gian, CloudWatch Alarm có thể gửi thông báo qua Amazon SNS đến email của nhóm. Log nên ở dạng có cấu trúc và bao gồm timestamp, provider, model, status code, retry count, latency cùng correlation ID. Không được ghi API key, credential, toàn bộ tài liệu hoặc dữ liệu nhạy cảm vào log.

#### 6. Trả lỗi rõ ràng cho người dùng

Ứng dụng không nên chuyển mọi lỗi provider thành HTTP 500. Với tình trạng quá tải tạm thời, hệ thống có thể trả thông báo dễ hiểu như “Hệ thống đang bận, vui lòng thử lại sau”, kèm mã 429 hoặc 503 phù hợp. Đối với tác vụ dài như indexing, có thể đưa task vào queue và cho phép người dùng theo dõi trạng thái thay vì giữ request chờ quá lâu.

### Checklist kiểm thử khả năng chịu lỗi

* Xác định rate limit và usage tier thực tế của Gemini project trong Google AI Studio.
* Tạo load test có kiểm soát để tái hiện HTTP 429 mà không ảnh hưởng dữ liệu thật.
* Xác nhận retry chỉ áp dụng cho lỗi tạm thời và dừng sau số lần tối đa.
* Kiểm tra jitter giúp các worker không retry cùng một thời điểm.
* Giới hạn concurrency và theo dõi độ dài queue.
* Kiểm tra task retry không tạo dữ liệu hoặc index trùng lặp.
* Giả lập Gemini unavailable và xác minh Bedrock fallback hoạt động đúng điều kiện.
* Kiểm tra IAM role chỉ có quyền gọi đúng model Bedrock cần thiết.
* Xác nhận CloudWatch metric/alarm và SNS notification hoạt động.
* So sánh latency, chất lượng và chi phí trước và sau khi thêm retry/fallback.
* Kiểm tra log không chứa API key, prompt hoặc tài liệu nhạy cảm.

### Những bài học nhóm rút ra

* **Chạy tốt trên local không đồng nghĩa chạy tốt khi có tải:** concurrency và số lượng người dùng làm thay đổi hoàn toàn hành vi của hệ thống.
* **Retry không phải “cây đũa thần”:** chỉ retry lỗi tạm thời, có backoff, jitter và giới hạn rõ ràng.
* **Thiết kế retry phải đi cùng idempotency:** nếu không, hệ thống có thể tạo dữ liệu trùng hoặc sai trạng thái.
* **Cần giảm tải trước khi retry:** giới hạn concurrency, queue, batching và tối ưu payload giúp xử lý nguyên nhân thay vì chỉ xử lý triệu chứng.
* **Luôn chuẩn bị phương án cho external dependency:** fallback, graceful degradation hoặc thông báo rõ ràng giúp ứng dụng không phụ thuộc hoàn toàn vào một provider.
* **Observability phải được thiết kế từ đầu:** log, metric và alarm giúp phát hiện lỗi sớm và đánh giá giải pháp có thực sự hiệu quả.

### Liên kết và tài liệu tham khảo

* [Bài viết gốc trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2230866751011618/)
* [Gemini API – Rate limits](https://ai.google.dev/gemini-api/docs/rate-limits)
* [Gemini API – Troubleshooting](https://ai.google.dev/gemini-api/docs/troubleshooting)
* [Timeouts, retries and backoff with jitter – Amazon Builders’ Library](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)
* [REL05-BP03 – Control and limit retry calls](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/rel_mitigate_interaction_failure_limit_retries.html)
* [Retry behavior – AWS SDKs and Tools](https://docs.aws.amazon.com/sdkref/latest/guide/feature-retry-behavior.html)
* [Identity-based policy examples for Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/security_iam_id-based-policy-examples.html)
* [InvokeModel API – Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_InvokeModel.html)
* [Alarming on logs – Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Alarm-On-Logs.html)
* [Creating a metric filter – Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CreateMetricFilterProcedure.html)

> **Kết luận:** HTTP 429 không chỉ là một lỗi cần retry vài lần mà là dấu hiệu hệ thống phải quản lý tải và dependency tốt hơn. Kết hợp retry có giới hạn, exponential backoff, jitter, concurrency control, idempotency, fallback có điều kiện và CloudWatch observability giúp luồng gọi AI API có khả năng phục hồi tốt hơn khi chuyển từ local sang môi trường có nhiều người sử dụng.
