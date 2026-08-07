---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# EC2 tưởng đơn giản mà không đơn giản – Những bài học khi làm lab AWS

### Thông tin bài viết

* **Nền tảng đăng:** Cộng đồng AWS Study Group VN
* **Người đăng đại diện nhóm:** Phan Thị Hải Vân
* **Thời gian đăng:** 09:17 ngày 28/07/2026
* **Liên kết bài đăng:** [Xem bài viết trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2226675271430766/)
* **Chủ đề:** Quản lý vòng đời Amazon EC2 và hạn chế chi phí ngoài ý muốn trong quá trình làm lab
* **Từ khóa:** Amazon EC2, Amazon EBS, AWS Free Tier, Amazon EventBridge, AWS Lambda, AWS Budgets, Cost Optimization
* **Hashtag:** #AWSStudyGroup #FCAJ #EC2 #CostOptimization

### Lý do lựa chọn chủ đề

Amazon EC2 thường là một trong những dịch vụ đầu tiên người mới tiếp cận khi học AWS. Việc khởi tạo một máy ảo có vẻ đơn giản, nhưng nếu chưa hiểu rõ trạng thái của instance, tài nguyên lưu trữ đi kèm và cách tính chi phí, người học rất dễ để lại tài nguyên sau bài lab.

Bài viết được nhóm thực hiện nhằm chia sẻ những nhầm lẫn thực tế đã gặp trong quá trình học, đặc biệt là sự khác nhau giữa Stop và Terminate, cách theo dõi quyền lợi AWS Free Tier, cũng như phương án tự động dừng instance bằng Amazon EventBridge và AWS Lambda.

### Nội dung chính của bài viết

#### 1. Stop không giống Terminate

Khi chọn Stop, EC2 instance chuyển sang trạng thái <code>stopped</code> nhưng vẫn tồn tại trong tài khoản và có thể được khởi động lại. Phí sử dụng compute của instance sẽ dừng, tuy nhiên các tài nguyên liên quan vẫn có thể tiếp tục phát sinh chi phí, đáng chú ý nhất là Amazon EBS volume. Một số tài nguyên mạng hoặc địa chỉ IP công cộng được giữ lại cũng cần được kiểm tra riêng.

Khi chọn Terminate, instance bị xóa vĩnh viễn và không thể khởi động lại. Root EBS volume thường được xóa theo mặc định, nhưng hành vi thực tế phụ thuộc thuộc tính <code>DeleteOnTermination</code>. Các data volume có thể vẫn được giữ lại và tiếp tục phát sinh phí nếu thuộc tính này được đặt là <code>false</code>. Vì vậy, sau khi terminate, người dùng vẫn cần kiểm tra mục Volumes, Snapshots, Elastic IP addresses và các tài nguyên liên quan thay vì cho rằng mọi thứ đã tự động biến mất.

| Trạng thái | Có thể khởi động lại | Phí compute của instance | EBS volume | Trường hợp sử dụng |
| --- | --- | --- | --- | --- |
| Stopped | Có | Không tính khi instance đã dừng | Vẫn tồn tại và có thể phát sinh phí | Tạm dừng để tiếp tục sử dụng sau |
| Terminated | Không | Dừng tính phí instance | Xóa hoặc giữ lại tùy <code>DeleteOnTermination</code> | Xóa hẳn tài nguyên không còn sử dụng |

Bài học rút ra: Stop chỉ tạm dừng compute, không đồng nghĩa với dọn dẹp toàn bộ tài nguyên.

#### 2. Không nên ghi nhớ Free Tier như một con số cố định

Bài đăng ban đầu nhấn mạnh rằng giới hạn giờ sử dụng trong mô hình Free Tier cũ được tính trên tổng thời gian của các instance đủ điều kiện, không phải mỗi instance đều có một hạn mức riêng. Do đó, chạy nhiều instance song song có thể làm mức sử dụng miễn phí hết nhanh hơn dự kiến.

Tuy nhiên, chính sách AWS Free Tier hiện phụ thuộc vào thời điểm tạo tài khoản:

* Với tài khoản được tạo trước ngày 15/07/2025, quyền lợi EC2 Free Tier cũ có thể áp dụng theo giới hạn giờ sử dụng trong 12 tháng đầu.
* Với tài khoản mới được tạo từ 15/07/2025, AWS sử dụng mô hình credit; gói miễn phí kéo dài tối đa 6 tháng hoặc đến khi credit được sử dụng hết.
* Trong cả hai trường hợp, người dùng phải kiểm tra mục Free Tier, Billing and Cost Management và điều kiện áp dụng của chính tài khoản thay vì dựa vào một con số được ghi nhớ từ tài liệu cũ.

**Bài học rút ra:** Free Tier giúp học tập với chi phí thấp hơn nhưng không có nghĩa mọi tài nguyên đều miễn phí hoặc không cần theo dõi.

#### 3. Tự động dừng EC2 thay vì phụ thuộc vào trí nhớ

Trong các bài lab ngắn hạn, người học có thể quên dừng instance sau khi hoàn thành. Bài viết đề xuất sử dụng lịch chạy cố định để tự động gọi thao tác <code>StopInstances</code>. Sơ đồ của nhóm sử dụng luồng:

**Amazon EventBridge → AWS Lambda → Amazon EC2**

![Kiến trúc tự động dừng EC2 bằng EventBridge và Lambda](/images/3-BlogsPosted/3.1-Blog1/eventbridge-lambda-stop-ec2.png)

*Amazon EventBridge kích hoạt theo lịch 23:00 hằng ngày, Lambda xử lý logic và gọi EC2 StopInstances API.*

Luồng xử lý được hiểu như sau:

1. EventBridge kích hoạt lịch vào 23:00 hằng ngày.
2. Lịch gọi một Lambda function.
3. Lambda xác định các EC2 instance cần dừng, ưu tiên lọc theo tag như <code>Environment=Lab</code> hoặc <code>AutoStop=true</code>.
4. Lambda gọi EC2 <code>StopInstances</code> API bằng một IAM role chỉ có các quyền cần thiết.
5. Kết quả thực thi được ghi vào CloudWatch Logs để kiểm tra khi lịch chạy không thành công.

Đối với thiết kế mới, Amazon EventBridge Scheduler là lựa chọn được AWS khuyến nghị thay cho scheduled rule cũ. Scheduler có thể gọi AWS API theo lịch; Lambda chỉ cần thiết khi cần thêm logic như lọc theo tag, loại trừ một số instance, gửi thông báo hoặc xử lý nhiều điều kiện.

**Bài học rút ra:** Automation giúp giảm phụ thuộc vào thao tác thủ công, nhưng vẫn phải cấu hình IAM theo nguyên tắc least privilege và kiểm tra log định kỳ.

#### 4. Thiết lập cảnh báo chi phí

Tự động dừng EC2 chỉ giải quyết một phần rủi ro. Người học cũng nên tạo AWS Budget và cấu hình cảnh báo theo chi phí thực tế hoặc ngưỡng dự kiến. Cảnh báo giúp phát hiện sớm tài nguyên bị bỏ quên, nhưng dữ liệu billing không cập nhật theo thời gian thực, vì vậy không thể thay thế việc kiểm tra tài nguyên sau mỗi buổi lab.

### Checklist sau mỗi buổi lab

* Kiểm tra EC2 Dashboard và xác nhận không còn instance ngoài dự kiến ở trạng thái <code>running</code>.
* Đặt tag hoặc tên rõ ràng cho instance, bao gồm mục đích, người tạo và ngày tạo.
* Dừng instance nếu còn sử dụng lại; terminate nếu chắc chắn không cần nữa.
* Kiểm tra <code>DeleteOnTermination</code> và xóa EBS volume hoặc snapshot không còn cần thiết.
* Kiểm tra Elastic IP, Load Balancer, NAT Gateway và các tài nguyên liên quan có thể phát sinh chi phí.
* Thiết lập EventBridge Scheduler hoặc EventBridge–Lambda cho các môi trường lab cần tự động dừng.
* Bật AWS Budget và theo dõi Billing Dashboard/Free Tier usage thường xuyên.
* Thực hiện cleanup theo đúng hướng dẫn của workshop.

### Giá trị và bài học từ bài viết

Bài viết giúp nhóm chuyển từ suy nghĩ “khởi tạo được EC2 là hoàn thành” sang tư duy quản lý toàn bộ vòng đời tài nguyên. Một bài lab chỉ thực sự kết thúc khi kết quả đã được kiểm tra, evidence đã được lưu và các tài nguyên không còn sử dụng đã được dọn dẹp.

Những kiến thức này có thể áp dụng trực tiếp cho các worklog sau: đặt naming convention, sử dụng tag để quản lý tài nguyên, thiết lập cảnh báo chi phí và luôn bổ sung bước cleanup vào checklist. Đây cũng là nền tảng của tư duy FinOps và cost optimization khi làm việc trên cloud.

### Liên kết và tài liệu tham khảo

* [Bài viết gốc trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2226675271430766/)
* [Vòng đời Amazon EC2 instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html)
* [Theo dõi mức sử dụng EC2 Free Tier](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-free-tier-usage.html)
* [Amazon EventBridge Scheduler](https://docs.aws.amazon.com/eventbridge/latest/userguide/using-eventbridge-scheduler.html)
* [Best practices cho AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-best-practices.html)

> **Kết luận:** Amazon EC2 dễ bắt đầu nhưng cần được quản lý cẩn thận. Hiểu đúng Stop–Terminate, kiểm tra chính sách Free Tier theo tài khoản, tự động hóa có kiểm soát và duy trì thói quen cleanup là những bước đơn giản nhưng quan trọng để học AWS an toàn và hiệu quả.
