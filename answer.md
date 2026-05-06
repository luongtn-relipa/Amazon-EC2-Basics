## 🌐 PHẦN 1: APPLICATION LOAD BALANCER (ALB)

---

### ✅ Câu 1 — Đáp án: **C**

**Đáp án đúng:** Dùng một ALB với ba target group, cấu hình listener rules dựa trên path-based routing và header-based routing.

#### 🎯 Vì sao C đúng?

ALB hoạt động ở **Layer 7 (HTTP)**, có thể "nhìn vào" nội dung HTTP request — bao gồm path (`/users`, `/orders`, `/search`), hostname, query string và header. Đây chính xác là use case mà ALB sinh ra để giải quyết:

- **Path-based routing:** `/users` → Target Group Users, `/orders` → Target Group Orders, `/search` → Target Group Search.
- **Header-based routing:** Trong target group Search, có thể thêm rule kiểm tra header `User-Agent` để lọc client mobile.
- **Health check riêng từng target group:** ALB cho phép mỗi target group có endpoint health check khác nhau (ví dụ `/users/health`, `/orders/health`).

Đây là kiến trúc **chuẩn vàng cho microservices** — đặc biệt hữu ích khi kết hợp với ECS hoặc EKS.

#### ❌ Vì sao các đáp án khác sai?

- **A (NLB):** NLB hoạt động ở **Layer 4 (TCP/UDP)** — KHÔNG nhìn được path hay header HTTP. NLB không thể route theo `/users` vs `/orders`. NLB phù hợp khi bạn cần hiệu năng cực cao hoặc traffic non-HTTP.
- **B (3 CLB):** Tốn kém gấp 3 lần, đi ngược yêu cầu "một Load Balancer duy nhất". Hơn nữa, CLB là thế hệ cũ, AWS khuyến nghị không dùng cho dự án mới. Route 53 cũng không route theo path được — nó chỉ làm DNS-level routing.
- **D (GWLB):** Sai hoàn toàn về kiến thức. GWLB hoạt động ở **Layer 3 (IP packets)** với giao thức GENEVE — chuyên dụng cho **virtual security appliance** (firewall, IDS/IPS), KHÔNG phải để route HTTP traffic.

#### 💡 Take-away
> Khi đề bài có từ khóa **"path-based routing"**, **"host-based routing"**, **"microservices"**, **"container"** → **ALB** là câu trả lời gần như chắc chắn.

---

### ✅ Câu 2 — Đáp án: **C**

**Đáp án đúng:** Đọc IP client từ header `X-Forwarded-For` mà ALB tự động chèn vào.

#### 🎯 Vì sao C đúng?

ALB là Layer 7 proxy — nó **terminate kết nối từ client** và **mở kết nối mới đến EC2**. Vì vậy server chỉ thấy IP của ALB. Tuy nhiên, AWS đã giải quyết vấn đề này bằng cách **tự động chèn các header chuẩn** vào mỗi HTTP request:

| Header | Ý nghĩa |
|---|---|
| `X-Forwarded-For` | IP gốc của client |
| `X-Forwarded-Port` | Port mà client kết nối tới |
| `X-Forwarded-Proto` | Giao thức gốc (HTTP hoặc HTTPS) |

Application chỉ cần **đọc các header này** trong code — hầu hết framework hiện đại (Express, Spring Boot, Django, ASP.NET) đều có middleware xử lý sẵn.

#### ❌ Vì sao các đáp án khác sai?

- **A (đổi sang NLB):** Đúng là NLB ở Layer 4 nên bảo toàn IP client gốc, **NHƯNG** đề bài nói rõ team đã đầu tư vào ALB và muốn giữ nó. Đổi sang NLB sẽ mất tính năng path/host routing, mất WebSocket native, mất HTTP/2, mất redirect rules… Đây là giải pháp "đập đi xây lại" không cần thiết.
- **B ("passthrough mode"):** **Tính năng này không tồn tại ở ALB.** Đây là đáp án đánh lừa — nghe rất kỹ thuật nhưng hoàn toàn bịa đặt.
- **D ("Preserve Client IP"):** Tùy chọn này **có ở NLB** (target type IP/instance), KHÔNG có ở ALB. Người ra đề cố ý trộn lẫn tính năng giữa các loại LB để bẫy thí sinh.

#### 💡 Take-away
> Khi dùng ALB và cần IP client → đọc `X-Forwarded-For`. Đừng đổi sang NLB nếu bạn vẫn cần các tính năng Layer 7 của ALB.

---

### ✅ Câu 3 — Đáp án: **B**

**Đáp án đúng:** ALB với một certificate mặc định + 4 certificate bổ sung, dùng SNI cho client hiện đại.

#### 🎯 Vì sao B đúng?

Đây là use case kinh điển của **SNI (Server Name Indication)** trên ALB:

1. **Client hiện đại (hỗ trợ SNI):** Trong quá trình SSL handshake, client gửi kèm hostname mong muốn (ví dụ `shop.com`). ALB tra trong danh sách certificate, tìm đúng cái cho `shop.com` và trả về.

2. **Legacy client (không hỗ trợ SNI):** Không gửi hostname trong handshake → ALB trả về **certificate mặc định**. Đây chính là lý do AWS yêu cầu phải khai báo "default certificate" — để legacy client vẫn hoạt động được, dù chỉ với một domain.

ALB hỗ trợ **nhiều listener với nhiều SSL certificate** thông qua SNI, đúng yêu cầu "một Load Balancer cho 5 website".

#### ❌ Vì sao các đáp án khác sai?

- **A (CLB với 5 listener):** **CLB chỉ hỗ trợ MỘT SSL certificate duy nhất.** Đây là hạn chế lớn nhất khiến CLB lỗi thời. Để dùng CLB cho 5 domain với 5 cert khác nhau, bạn phải dựng 5 CLB riêng — đắt và khó quản lý.
- **C (NLB tự xử lý SSL):** NLB hoạt động ở Layer 4 và **không tự "xử lý SSL termination cho mọi domain mà không cần cấu hình"**. NLB cũng hỗ trợ multi-cert qua SNI nhưng phải cấu hình rõ ràng. Đáp án này sai về mặt mô tả kỹ thuật.
- **D (5 ALB + Route 53):** Tốn kém gấp 5 lần, đi ngược yêu cầu "một Load Balancer". Route 53 weighted routing cũng không phải để route theo domain — nó dùng để phân phối tải hoặc canary deployment.

#### 💡 Take-away
> **Bảng so sánh khả năng SSL multi-domain của ELB:**
> | Loại LB | Multi-cert | SNI |
> |---|---|---|
> | CLB (v1) | ❌ Chỉ 1 cert | ❌ |
> | ALB (v2) | ✅ | ✅ |
> | NLB (v2) | ✅ | ✅ |
>
> Câu hỏi thi rất hay hỏi: "SNI hoạt động trên LB nào?" → Đáp án: **ALB, NLB, CloudFront**. Không hỗ trợ CLB.

---

## ⚙️ PHẦN 2: AUTO SCALING GROUP (ASG)

---

### ✅ Câu 4 — Đáp án: **B**

**Đáp án đúng:** Kết hợp Target Tracking Scaling + Scheduled Scaling.

#### 🎯 Vì sao B đúng?

Đề bài có **hai yêu cầu phân biệt rõ ràng**, cần hai loại scaling khác nhau cho từng yêu cầu:

**Yêu cầu 1: "CPU trung bình luôn ở khoảng 50%"**
→ Đây là pattern điển hình của **Target Tracking Scaling**. Bạn chỉ cần khai báo target value (50%), AWS tự động tính toán và điều chỉnh số instance để giữ metric ở mức đó. **Đơn giản, tự động, ít cấu hình.**

**Yêu cầu 2: "Sẵn sàng năng lực TRƯỚC peak thứ Hai 9 giờ sáng"**
→ Đây là pattern điển hình của **Scheduled Scaling** — biết trước peak nên chuẩn bị trước. Cấu hình: tăng `min capacity` lên 8 vào 8:45 sáng thứ Hai. ASG sẽ tự khởi động instance, đợi 4–5 phút khởi động xong, ấm máy, sẵn sàng nhận traffic lúc 9:00.

**Vì sao phải kết hợp cả hai?** Vì hai loại tải có bản chất khác nhau:
- Target Tracking phản ứng **theo thời gian thực** với metric → tốt cho tải biến động không dự đoán được.
- Scheduled Scaling chuẩn bị **trước thời điểm cụ thể** → tốt cho tải đỉnh có pattern rõ ràng.

#### ❌ Vì sao các đáp án khác sai?

- **A (chỉ Simple Scaling):** Không đáp ứng yêu cầu 2 — Simple Scaling chỉ phản ứng *sau khi* metric vượt ngưỡng, mà tại thời điểm đó user đã gặp lỗi 5xx rồi. Hơn nữa, Simple Scaling không tự duy trì CPU ở mức target; nó chỉ "thêm/bớt instance khi alarm trigger" — phải tự tính toán ngưỡng phức tạp.
- **C (chỉ Predictive Scaling):** Predictive Scaling có giá trị nhưng **không phải giải pháp duy nhất cho mọi tình huống**. Predictive cần dữ liệu lịch sử ổn định để học pattern — không phù hợp khi tải nền biến động ngẫu nhiên. Hơn nữa, Predictive thường được khuyến nghị **kết hợp với** Target Tracking, không thay thế.
- **D (Step Scaling 10 bậc):** Step Scaling tốt khi có nhiều ngưỡng phản ứng khác nhau, nhưng **không giải quyết được yêu cầu "scale TRƯỚC peak"**. Dù có 100 bậc đi nữa, Step Scaling vẫn là *reactive* — phản ứng sau khi metric thay đổi.

#### 💡 Take-away
> **Cây quyết định chọn Scaling Policy:**
> - "Giữ metric ở mức X" → **Target Tracking** ✅ (đơn giản nhất, dùng đầu tiên)
> - "Có pattern thời gian rõ ràng" (thứ Hai 9h, cuối tháng, Black Friday) → **Scheduled Scaling**
> - "Pattern phức tạp dài hạn, có dữ liệu lịch sử" → **Predictive Scaling**
> - "Cần phản ứng theo nhiều ngưỡng" → **Step Scaling**
>
> Trong thực tế, **kết hợp nhiều loại** là chuẩn — đề thi thường ưu tiên đáp án nào kết hợp đúng các loại scaling cho các yêu cầu khác nhau.

---

### ✅ Câu 5 — Đáp án: **C**

**Đáp án đúng:** Đây là hành vi đúng của cooldown — giải pháp là dùng AMI chuẩn bị sẵn để giảm thời gian khởi động.

#### 🎯 Vì sao C đúng?

**Cooldown period là một tính năng, không phải bug.** Nó tồn tại để giải quyết một bài toán cụ thể:

> *Khi ASG vừa thêm instance mới, instance đó cần thời gian khởi động + ấm máy + bắt đầu nhận traffic. Trong khoảng thời gian đó, CPU trung bình của ASG VẪN còn cao (vì instance mới chưa chia tải). Nếu ASG cứ thấy CPU cao là scale tiếp, nó sẽ scale-out QUÁ MỨC cần thiết — gây lãng phí và mất ổn định.*

Cooldown period (mặc định 300 giây) là khoảng "**chờ xem**" để metric ổn định lại sau khi instance mới đi vào hoạt động. Kỹ sư trong tình huống đang **hiểu sai cooldown là bug** — thực ra ASG đang làm đúng nhiệm vụ bảo vệ hệ thống.

**Vấn đề thực sự cần fix là gì?** Là **thời gian khởi động instance quá dài (4 phút)**. Khi User Data cài đặt phần mềm mất 4 phút, instance mới chưa kịp phục vụ → cooldown 5 phút trở thành "bottleneck" cho tốc độ phản ứng của hệ thống.

**Giải pháp đúng — Golden AMI:**
1. Khởi động một instance, cài đặt sẵn tất cả phần mềm cần thiết.
2. Tạo AMI từ instance đó (đó là "golden AMI").
3. Cấu hình Launch Template dùng golden AMI thay vì AMI gốc + User Data dài.
4. Instance mới khởi động trong **30–60 giây** thay vì 4 phút.
5. Lúc này có thể an toàn giảm cooldown xuống (ví dụ 120 giây) → hệ thống phản ứng nhanh hơn nhiều.

#### ❌ Vì sao các đáp án khác sai?

- **A (bug AWS):** Hoàn toàn sai về kiến thức nền tảng. Trong môi trường production thật, mở support ticket cho vấn đề này sẽ rất ngại với team AWS.
- **B (cooldown = 0):** Cực kỳ nguy hiểm! Cooldown = 0 nghĩa là ASG sẽ scale-out liên tục mỗi giây khi metric còn cao. Trong 5 phút khởi động instance đầu tiên, có thể ASG đã thêm 50–100 instance không cần thiết → **hóa đơn AWS sốc**, hệ thống dao động không ổn định (thrashing).
- **D (cooldown 30 phút):** Đi ngược hướng. Cooldown càng dài, hệ thống càng phản ứng chậm với tải tăng đột biến. 30 phút là quá nhiều — user sẽ gặp lỗi suốt nửa tiếng trước khi hệ thống scale-out tiếp.

#### 💡 Take-away
> **Quy tắc vàng về cooldown:**
> - **Đừng** đặt cooldown = 0.
> - **Đừng** tăng cooldown vô tội vạ.
> - **Hãy** giảm thời gian khởi động instance (golden AMI, container, warm pool) → từ đó mới có cơ sở để giảm cooldown.

---

### ✅ Câu 6 — Đáp án: **C**

**Đáp án đúng:** ASG ở một AZ duy nhất không thể có HA — phải trải ASG (và ALB) qua nhiều AZ.

#### 🎯 Vì sao C đúng?

Đây là một trong những **lỗi kiến trúc phổ biến nhất** mà người mới mắc phải:

> "Tôi có ASG, tôi có ALB, vậy hệ thống của tôi phải High Availability rồi chứ?"

**Sai!** ASG và ALB chỉ là **công cụ** để thực hiện HA — chúng không tự đảm bảo HA. **HA chỉ có khi hạ tầng được trải qua nhiều Availability Zone.**

Lý do: mỗi AZ là một (hoặc nhiều) **trung tâm dữ liệu vật lý độc lập**. Một AZ có thể bị sập do mất điện, cháy nổ, network issue, hoặc các sự cố khác. Nếu **toàn bộ instance của bạn nằm trong 1 AZ**, khi AZ đó sập, **không có gì** trong AWS có thể cứu bạn.

**Giải pháp đúng:**
1. Cấu hình ASG với **nhiều subnet ở nhiều AZ** (ví dụ subnet ở `us-east-1a`, `us-east-1b`, `us-east-1c`).
2. ASG sẽ **tự động phân bổ instance đều giữa các AZ** (best-effort).
3. ALB cũng phải được **enable trên các AZ tương ứng**.
4. Khi một AZ sập, ASG sẽ **tự động khởi động instance thay thế ở các AZ còn sống**, và ALB sẽ ngừng route traffic đến AZ chết → người dùng không bị ảnh hưởng (hoặc chỉ ảnh hưởng vài giây).

**Một chi tiết quan trọng:** Khi đã trải nhiều AZ, hãy bật **Cross-Zone Load Balancing** (mặc định đã bật cho ALB) để traffic được phân phối đều cho mọi instance ở mọi AZ, tránh tình trạng AZ này quá tải còn AZ kia rảnh rỗi.

#### ❌ Vì sao các đáp án khác sai?

- **A (đổi metric):** Hoàn toàn không liên quan đến vấn đề. `RequestCountPerTarget` là metric tốt cho ứng dụng web. Đổi sang CPUUtilization không làm AZ sống lại được.
- **B (tăng max=50):** `max` chỉ là giới hạn trên cho số instance. Nâng max lên 50 cũng vô nghĩa nếu **cả 50 instance đều ở AZ đã sập**. Đây là bẫy kinh điển — người mới hay nghĩ "thêm instance là giải quyết được".
- **D (đổi NLB):** Chuyển sang NLB không giải quyết vấn đề AZ. Cả ALB lẫn NLB **đều có thể HA tốt nếu cấu hình multi-AZ**, và đều **đều fail nếu chỉ ở một AZ**. Vấn đề nằm ở **kiến trúc**, không phải ở loại Load Balancer.

#### 💡 Take-away
> **Công thức HA chuẩn cho EC2 trên AWS:**
> ```
> HA = ALB multi-AZ + ASG multi-AZ + EC2 instances ở ít nhất 2 AZ
> ```
> Thiếu bất kỳ thành phần nào trong công thức này, hệ thống KHÔNG có HA thực sự — bất kể bạn có dùng dịch vụ AWS đắt tiền đến đâu.
>
> **Câu hỏi tự kiểm tra cho mọi kiến trúc:** *"Nếu một AZ trong region của tôi sập trong 1 giờ, hệ thống có còn phục vụ được người dùng không?"* — Nếu trả lời "không" → hệ thống chưa HA.

---

## 📊 Bảng tổng kết kiến thức cốt lõi

| Câu | Chủ đề | Bài học chính |
|---|---|---|
| 1 | ALB Layer 7 routing | Path/host/header routing là sức mạnh của ALB, không có ở NLB/CLB/GWLB |
| 2 | X-Forwarded-For | ALB chèn IP client vào header, đừng nhầm với tính năng "Preserve Client IP" của NLB |
| 3 | SNI + Multi-cert | ALB/NLB hỗ trợ nhiều cert qua SNI; CLB chỉ 1 cert duy nhất |
| 4 | Scaling policy | Target Tracking + Scheduled là combo chuẩn cho tải biến động + tải có pattern |
| 5 | Cooldown | Cooldown là tính năng bảo vệ; fix thời gian khởi động instance, đừng fix cooldown |
| 6 | High Availability | Multi-AZ là điều kiện cần để có HA — ALB/ASG chỉ là công cụ |

---