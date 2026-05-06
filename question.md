
## 🌐 PHẦN 1: APPLICATION LOAD BALANCER (ALB)

---

### 🔹 Câu 1 — Routing trong kiến trúc microservices

Một công ty thương mại điện tử đang triển khai kiến trúc microservices trên AWS với ba service riêng biệt:
- Service **Users**: xử lý request tại đường dẫn `example.com/users`
- Service **Orders**: xử lý request tại `example.com/orders`
- Service **Search**: xử lý request tại `example.com/search` nhưng chỉ phục vụ client di động (request có header `User-Agent: Mobile`)

Mỗi service chạy trên một nhóm EC2 instance riêng và có endpoint health check khác nhau. Công ty muốn dùng **một** Load Balancer duy nhất cho cả ba service để giảm chi phí và đơn giản hóa quản trị.

**Giải pháp nào sau đây phù hợp NHẤT?**

**A.** Dùng một Network Load Balancer (NLB) với ba listener khác nhau, mỗi listener trỏ đến một target group.

**B.** Dùng ba Classic Load Balancer (CLB) riêng biệt, mỗi CLB cho một service, sau đó dùng Route 53 để route theo path.

**C.** Dùng một Application Load Balancer (ALB) với ba target group, cấu hình listener rules dựa trên path-based routing và header-based routing, mỗi target group có health check riêng.

**D.** Dùng một Gateway Load Balancer (GWLB) với ba target group, vì GWLB hỗ trợ Layer 7 routing.

---

### 🔹 Câu 2 — Logging IP của client

Đội DevOps phát hiện rằng sau khi triển khai ALB phía trước cụm EC2, **tất cả log của application server đều ghi nhận cùng một IP nguồn** — đó là IP private của ALB. Điều này khiến team Security không thể phát hiện các IP độc hại để chặn ở tầng ứng dụng (ví dụ rate limiting theo IP, blacklist).

**Giải pháp nào dưới đây giúp application server biết được IP THẬT của client?**

**A.** Tắt ALB và dùng Network Load Balancer (NLB) thay thế, vì NLB bảo toàn IP gốc của client.

**B.** Cấu hình ALB ở chế độ "passthrough mode" để forward IP gốc của client.

**C.** Đọc IP client từ header `X-Forwarded-For` mà ALB tự động chèn vào mỗi request, đồng thời có thể đọc thêm `X-Forwarded-Port` và `X-Forwarded-Proto` nếu cần.

**D.** Bật tính năng "Preserve Client IP" trong cấu hình target group của ALB.

> **Lưu ý:** Câu này có một đáp án hấp dẫn nhưng KHÔNG phải là đáp án phù hợp nhất. Hãy đọc kỹ yêu cầu — họ đã đầu tư vào ALB và muốn giữ ALB.

---

### 🔹 Câu 3 — SSL với nhiều domain

Một startup vận hành **5 website khác nhau** (`shop.com`, `blog.com`, `api.com`, `admin.com`, `support.com`), mỗi website có SSL certificate riêng. Họ muốn dùng **một Load Balancer duy nhất** để phục vụ cả 5 website nhằm tiết kiệm chi phí.

Đồng thời, một số khách hàng doanh nghiệp của họ vẫn dùng **trình duyệt cũ** (legacy clients) không hỗ trợ SNI.

**Kiến trúc nào sau đây đáp ứng được CẢ HAI yêu cầu trên?**

**A.** Dùng một Classic Load Balancer (CLB) với 5 listener, mỗi listener gắn một SSL certificate khác nhau.

**B.** Dùng một Application Load Balancer (ALB) với một HTTPS listener: chỉ định một certificate **mặc định** (cho legacy clients không hỗ trợ SNI) và thêm 4 certificate còn lại vào danh sách. Client hiện đại sẽ dùng SNI để chọn đúng certificate.

**C.** Dùng một Network Load Balancer (NLB) ở Layer 4, vì NLB tự động xử lý SSL termination cho mọi domain mà không cần cấu hình.

**D.** Dùng 5 ALB riêng biệt, mỗi ALB cho một domain, sau đó dùng Route 53 weighted routing để phân phối traffic.

---

## ⚙️ PHẦN 2: AUTO SCALING GROUP (ASG)

---

### 🔹 Câu 4 — Scaling Policy phù hợp

Công ty SaaS vận hành một ứng dụng web với hai loại tải:
- **Tải nền (baseline)** thay đổi liên tục trong ngày, không theo pattern rõ ràng — phụ thuộc vào hành vi user thực tế.
- **Tải đỉnh có thể dự đoán**: mỗi thứ Hai 9:00 sáng có buổi training online, lượng request tăng gấp 5 lần trong 30 phút.

Yêu cầu của business:
1. Khi tải biến động, hệ thống phải tự điều chỉnh để **CPU trung bình của ASG luôn ở khoảng 50%**.
2. Vào buổi training thứ Hai, hệ thống phải **sẵn sàng năng lực TRƯỚC** khi traffic peak đến — không được scale-out chậm vì người dùng đầu tiên sẽ gặp lỗi.

**Kiến trúc scaling policy nào tối ưu NHẤT?**

**A.** Chỉ dùng **Simple Scaling** với CloudWatch alarm: CPU > 70% thêm 2 instance, CPU < 30% bớt 1 instance.

**B.** Kết hợp **Target Tracking Scaling** (target CPU = 50%) cho tải nền + **Scheduled Scaling** để tăng `min capacity` lên 8 vào 8:45 sáng thứ Hai (trước peak 15 phút).

**C.** Chỉ dùng **Predictive Scaling**, vì nó dự đoán mọi loại tải dựa trên machine learning.

**D.** Dùng **Step Scaling** với rất nhiều bậc (10 bậc) để xử lý mọi tình huống, không cần Scheduled Scaling.

---

### 🔹 Câu 5 — Hiểu sai về Cooldown

Một kỹ sư mới cấu hình ASG với policy: **CPU > 70% → thêm 2 instance**. Anh ta thấy CPU đang ở 85% nhưng ASG chỉ thêm 2 instance rồi **dừng lại trong 5 phút**, mặc dù CPU vẫn đang ở 85%. Anh ta lo lắng và muốn "fix" vấn đề này.

Đồng thời, kỹ sư phát hiện instance mới mất **gần 4 phút** để khởi động xong (cài đặt phần mềm qua User Data) trước khi sẵn sàng nhận traffic.

**Hành động nào sau đây là ĐÚNG ĐẮN nhất?**

**A.** Đây là bug của AWS — mở support ticket yêu cầu fix.

**B.** Đặt cooldown period = 0 để ASG scale liên tục mà không bị "khóa".

**C.** Đây là hành vi đúng của **cooldown period** (mặc định 300 giây) — ASG tạm dừng để metric ổn định lại sau khi instance mới khởi động xong và bắt đầu chia tải. Giải pháp: tạo **AMI đã chuẩn bị sẵn** (golden AMI) chứa tất cả phần mềm cần thiết để giảm thời gian khởi động instance, từ đó có thể giảm cooldown period một cách an toàn.

**D.** Tăng cooldown period lên 1800 giây (30 phút) để ASG có nhiều thời gian "suy nghĩ" hơn trước khi scale tiếp.

---

### 🔹 Câu 6 — High Availability bị đánh sập bởi cấu hình sai

Một công ty triển khai ứng dụng e-commerce với kiến trúc:
- 1 Application Load Balancer
- 1 Auto Scaling Group: `min=2, desired=4, max=10`
- ASG được cấu hình **chỉ trong 1 Availability Zone** (`us-east-1a`)
- Scaling metric: `RequestCountPerTarget` với target value = 1000 req/instance

Một ngày nọ, **toàn bộ AZ `us-east-1a` bị sự cố**. Trong 45 phút, người dùng không truy cập được website và team CTO yêu cầu giải thích.

**Đâu là nguyên nhân gốc rễ và giải pháp ĐÚNG nhất?**

**A.** Nguyên nhân do `RequestCountPerTarget` không phù hợp; chuyển sang dùng CPUUtilization sẽ khắc phục được sự cố AZ.

**B.** Nguyên nhân do `max=10` quá thấp; nâng `max` lên 50 sẽ giải quyết vấn đề.

**C.** Nguyên nhân là **High Availability không tự đến từ ASG hay ALB** nếu hạ tầng chỉ ở **một AZ duy nhất**. Giải pháp đúng: cấu hình ASG **trải qua nhiều AZ** (ít nhất 2 AZ, ví dụ `us-east-1a` và `us-east-1b`) và đảm bảo ALB cũng được enable trên các AZ đó. Khi đó, nếu một AZ sập, ASG sẽ tự động chạy instance ở AZ còn lại và ALB sẽ route traffic qua đó.

**D.** Nên thay ALB bằng Network Load Balancer vì NLB chịu lỗi tốt hơn ALB ở mức AZ.

---