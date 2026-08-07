---
title : "Kiểm thử chất lượng, bảo mật và chi phí"
date : 2026-08-06
weight : 5
chapter : false
pre : " <b> 5.5 </b> "
---

#### Mục tiêu

Một giao diện hoạt động chưa đủ để kết luận đồ án thành công. Phần này đánh giá toàn bộ chuỗi từ tải tài liệu, lập chỉ mục, truy xuất, tạo câu trả lời và trích dẫn đến dữ liệu bền vững, bảo mật, khả năng xử lý lỗi và chi phí vận hành.

#### 1. Chuẩn bị bộ đánh giá

Chọn một tập tài liệu đại diện cho nội dung nhóm sẽ demo. Loại bỏ dữ liệu nhạy cảm, tệp trùng và phiên bản cũ. Với mỗi tài liệu, ghi tên, phiên bản, số trang, ngôn ngữ và checksum nếu cần tái tạo kết quả.

Xây dựng 20–30 câu hỏi gồm:

- Câu hỏi có câu trả lời xuất hiện rõ trong một đoạn.
- Câu hỏi cần kết hợp hai đoạn trong cùng tài liệu.
- Câu hỏi dễ nhầm giữa các phiên bản hoặc thuật ngữ gần giống.
- Câu hỏi không có đáp án trong bộ tài liệu để kiểm tra việc từ chối suy đoán.
- Câu hỏi tiếng Việt và tiếng Anh nếu hệ thống tuyên bố hỗ trợ song ngữ.

Mỗi câu hỏi có đáp án tham chiếu, tài liệu nguồn, đoạn mong đợi và loại câu hỏi. Không dùng chính câu trả lời do chatbot sinh ra làm đáp án chuẩn.

#### 2. Kiểm thử chức năng

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| F01 | Tải PDF hợp lệ | Tệp được tiếp nhận, trạng thái indexing hoàn tất và không có lỗi âm thầm |
| F02 | Tải tệp không hỗ trợ hoặc tệp lỗi | Hệ thống từ chối rõ ràng, không làm hỏng collection hiện có |
| F03 | Đặt câu hỏi có đáp án | Trả lời đúng trọng tâm và mở được citation đến đoạn nguồn |
| F04 | Đặt câu hỏi ngoài tài liệu | Nêu không đủ thông tin thay vì tạo dữ kiện không có căn cứ |
| F05 | Khởi động lại container | Tài liệu, chỉ mục và lịch sử cần thiết vẫn còn |
| F06 | Dừng và khởi động EC2 | EBS tự gắn, container khởi động và dữ liệu truy cập được |
| F07 | Sao lưu và khôi phục | Checksum đúng, container thử đọc được dữ liệu và trả lời câu hỏi đã biết |

Mỗi test case cần lưu thời gian, commit hash, model, tài liệu, kết quả thực tế và bằng chứng. Nếu lỗi, ghi rõ bước nào thất bại thay vì chỉ đánh dấu “không đạt”.

#### 3. Đánh giá RAG

Với từng câu hỏi, chấm riêng ba lớp:

1. **Retrieval:** đoạn mong đợi có nằm trong các kết quả truy xuất đầu tiên không.
2. **Answer:** câu trả lời có đúng ý chính, không thêm thông tin trái tài liệu không.
3. **Citation:** nguồn được dẫn có thật sự hỗ trợ mệnh đề chính không.

Các chỉ số đề xuất:

```text
Retrieval accuracy = số câu lấy đúng đoạn / tổng số câu có đáp án
Citation support rate = số câu có citation hỗ trợ đúng / tổng số câu được trả lời
Unsupported answer rate = số câu trả lời ngoài tài liệu nhưng vẫn khẳng định / tổng số câu ngoài tài liệu
```

Ngưỡng nghiệm thu của bản thử nghiệm:

- Retrieval accuracy từ 80% trở lên.
- Citation support rate từ 90% trở lên.
- Câu hỏi ngoài tài liệu phải ưu tiên trả lời không đủ thông tin; mọi khẳng định không có nguồn cần được phân tích.

Khi kết quả thấp, xem lại chất lượng tài liệu, cách tách đoạn, embedding, top-k và prompt trước khi đổi mô hình lớn hơn.

#### 4. Kiểm thử hiệu năng và lỗi phụ thuộc

Đo ít nhất các thời điểm: nhận câu hỏi, hoàn tất truy xuất, nhận token hoặc câu trả lời đầu tiên và hoàn tất phản hồi. Với bản demo ít người dùng, mục tiêu ban đầu là thời gian phản hồi trung vị không quá 15 giây.

Thử lần lượt 1, 2 và một số nhỏ người dùng đồng thời. Theo dõi:

- CPU, bộ nhớ và dung lượng EC2.
- Latency truy xuất và tổng latency.
- Số lỗi HTTP 429, timeout và retry khi gọi Gemini.
- Thời gian indexing theo kích thước tài liệu.

Retry phải có giới hạn, exponential backoff và jitter. Không gửi lại hàng loạt request vô hạn vì vừa làm tăng lỗi vừa khó kiểm soát quota.

#### 5. Kiểm thử bảo mật

- Tìm `GOOGLE_API_KEY`, mẫu access key và nội dung `.env` trong Git history, Docker image metadata và log.
- Xác nhận bucket S3 vẫn bật Block Public Access và không có object public.
- Dùng IAM Policy Simulator hoặc một thao tác thử để xác nhận role không truy cập được bucket ngoài phạm vi.
- Xác nhận SSH chỉ mở cho IP quản trị; cổng 7860 không được mở ngoài Internet.
- Dùng API key sai và xác nhận ứng dụng báo lỗi có kiểm soát, không in khóa ra log.
- Kiểm tra quyền filesystem của `.env` là 600 và chỉ người vận hành cần thiết đọc được.

Ví dụ kiểm tra tệp môi trường và cổng publish:

```bash
cd /opt/fcaj/app
stat -c '%a %U:%G %n' .env
sudo docker compose ps
sudo ss -lntp | grep -E ':80|:7860'
```

#### 6. Đối chiếu chi phí

Sau buổi thử nghiệm, mở Cost Explorer hoặc Billing để so sánh với giả định:

- Số giờ EC2 thực tế.
- EBS gp3 60 GB còn tồn tại bao lâu.
- Public IPv4 được tính trong khoảng thời gian nào.
- Dung lượng S3, số phiên bản object và log CloudWatch.
- Alarm, metric tùy chỉnh và dữ liệu truyền.
- Chi phí Gemini hoặc API ngoài AWS được theo dõi riêng.

Phương án mục tiêu là `t3.medium` khoảng 60 giờ/tháng, tổng ước tính 11,36 USD đã gồm 15% dự phòng và Budget 15 USD/tháng. Nếu chạy liên tục, ước tính tăng lên khoảng 55,89 USD/tháng; do đó cần dừng EC2 khi không sử dụng.

#### 7. Biên bản nghiệm thu

| Nhóm tiêu chí | Kết quả | Bằng chứng | Kết luận |
|---|---|---|---|
| Chức năng | Số test đạt/tổng số | Ảnh, log, test sheet | Đạt/Không đạt |
| Retrieval | Tỷ lệ lấy đúng đoạn | Bảng đánh giá 20–30 câu | Đạt/Không đạt |
| Citation | Tỷ lệ nguồn hỗ trợ | Bảng đối chiếu nguồn | Đạt/Không đạt |
| Hiệu năng | P50, giá trị lớn nhất, lỗi | Metric và log | Đạt/Không đạt |
| Bền vững | Restart và restore | Archive, checksum, ảnh kiểm tra | Đạt/Không đạt |
| Bảo mật | Secret scan, IAM, SG, S3 | Checklist | Đạt/Không đạt |
| Chi phí | Thực tế so với dự kiến | Billing/Budget | Đạt/Không đạt |

Nếu một tiêu chí quan trọng không đạt, ghi người phụ trách, biện pháp khắc phục và ngày kiểm thử lại. Chỉ hoàn tất workshop khi các lỗi nghiêm trọng về dữ liệu và secret đã được xử lý.
