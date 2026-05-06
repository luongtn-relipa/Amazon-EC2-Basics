# BẢN THẢO THUYẾT TRÌNH
## Chủ đề: High Availability & Scalability trong Amazon EC2
### (Slide 48 đến Slide 87)

---

## 🎯 Slide 48: Mở đầu chủ đề – High Availability & Scalability

Xin chào mọi người! Đến phần tiếp theo của bài thuyết trình, chúng ta sẽ đi vào một chủ đề rất quan trọng trong AWS, đó là **Tính sẵn sàng cao (High Availability)** và **Khả năng mở rộng (Scalability)**.

Đây là hai khái niệm cốt lõi mà bất kỳ ai làm việc với hệ thống Cloud, đặc biệt là EC2, đều cần phải nắm vững. Trong các slide tiếp theo, mình sẽ giúp các bạn hiểu rõ sự khác biệt giữa hai khái niệm này, cũng như cách AWS hỗ trợ chúng ta đạt được cả hai thông qua các dịch vụ như Load Balancer và Auto Scaling Group.

---

## 📌 Slide 49: Scalability & High Availability – Tổng quan

Trước tiên, chúng ta cần hiểu **Scalability – Khả năng mở rộng** là gì?

Scalability có nghĩa là một ứng dụng hoặc hệ thống có khả năng **xử lý được tải lớn hơn** bằng cách tự thích nghi với điều kiện thay đổi. Ví dụ, khi số lượng người dùng tăng lên, hệ thống vẫn phải đáp ứng được mà không bị sập.

Có hai loại Scalability chính:
- **Vertical Scalability** – Mở rộng theo chiều dọc
- **Horizontal Scalability** – Mở rộng theo chiều ngang, còn gọi là *elasticity*

Lưu ý quan trọng: **Scalability có liên quan nhưng KHÔNG giống High Availability**. Đây là hai khái niệm khác nhau và mình sẽ làm rõ ngay bây giờ.

Để dễ hình dung, chúng ta sẽ dùng ví dụ về một **trung tâm chăm sóc khách hàng (call center)** để minh họa.

---

## 📌 Slide 50: Vertical Scalability – Mở rộng theo chiều dọc

**Vertical Scalability** nghĩa là **tăng kích thước của một instance**.

Hãy tưởng tượng ứng dụng của bạn đang chạy trên một instance loại `t2.micro` – tức là một máy ảo nhỏ. Khi bạn muốn mở rộng theo chiều dọc, bạn sẽ chuyển ứng dụng đó sang chạy trên một máy lớn hơn, ví dụ `t2.large`. Đơn giản là **nâng cấp máy lên cấu hình mạnh hơn**.

Quay lại với ví dụ call center: thay vì để một nhân viên junior xử lý cuộc gọi, ta thay bằng một nhân viên senior có kinh nghiệm xử lý nhanh và nhiều việc hơn. Đó chính là Vertical Scaling.

Loại scaling này **rất phổ biến với các hệ thống không phân tán (non-distributed)**, ví dụ như cơ sở dữ liệu. Trong AWS, các dịch vụ như **RDS** và **ElastiCache** sử dụng Vertical Scaling.

Tuy nhiên, có một hạn chế: bạn **không thể mở rộng vô hạn theo chiều dọc**, vì luôn có giới hạn về phần cứng. Đến một lúc nào đó, bạn sẽ phải dùng tới phương án khác.

---

## 📌 Slide 51: Horizontal Scalability – Mở rộng theo chiều ngang

**Horizontal Scalability** nghĩa là **tăng số lượng instance hoặc hệ thống** cho ứng dụng của bạn.

Quay lại ví dụ call center: thay vì thay một nhân viên giỏi hơn, ta tuyển thêm nhiều nhân viên để cùng nhau xử lý cuộc gọi. Số lượng càng nhiều, năng lực xử lý càng cao.

Horizontal Scaling đòi hỏi hệ thống phải là **distributed system – hệ thống phân tán**. Đây là cách tiếp cận **rất phổ biến với các ứng dụng web hiện đại**.

Và điểm tuyệt vời là: với cloud như **Amazon EC2**, việc mở rộng theo chiều ngang trở nên cực kỳ đơn giản. Bạn chỉ cần vài cú click hoặc một dòng lệnh là có thể tạo thêm hàng chục, hàng trăm instance.

---

## 📌 Slide 52: High Availability – Tính sẵn sàng cao

Bây giờ là **High Availability – Tính sẵn sàng cao**.

High Availability thường **đi đôi với Horizontal Scaling**. Cụ thể, High Availability nghĩa là chạy ứng dụng của bạn ở **ít nhất 2 trung tâm dữ liệu** – trong AWS, ta gọi là **Availability Zones (AZ)**.

Mục tiêu của High Availability là gì? Là **sống sót được khi một trung tâm dữ liệu bị sập**. Nếu một AZ gặp sự cố, ứng dụng của bạn vẫn chạy ở AZ còn lại.

Ví dụ minh họa: bạn có một tòa nhà ở New York và một tòa nhà ở San Francisco. Nếu New York gặp vấn đề, San Francisco vẫn hoạt động bình thường.

Có hai dạng High Availability:
- **Passive (bị động)**: ví dụ như RDS Multi-AZ, có một bản sao dự phòng sẵn sàng đứng lên thay khi cần.
- **Active (chủ động)**: như trong Horizontal Scaling, tất cả các instance đều đang xử lý request cùng lúc.

---

## 📌 Slide 53: High Availability & Scalability cho EC2

Áp dụng cụ thể vào EC2, ta có ba nhóm chính:

**Vertical Scaling (Scale Up / Scale Down):**
Tăng giảm kích thước instance. Ví dụ từ `t2.nano` (chỉ 0.5 GB RAM, 1 vCPU) lên đến `u-12tb1.metal` – một con quái vật với **12.3 TB RAM và 448 vCPU**!

**Horizontal Scaling (Scale Out / Scale In):**
Tăng giảm số lượng instance. Để làm được điều này, AWS cung cấp **Auto Scaling Group** và **Load Balancer**.

**High Availability:**
Chạy nhiều instance ở các AZ khác nhau – tức là **Auto Scaling Group multi-AZ** và **Load Balancer multi-AZ**.

Đây chính là 3 trụ cột giúp ứng dụng của bạn vừa mở rộng được, vừa có độ tin cậy cao.

---

## 📌 Slide 54: Load Balancing là gì?

Tiếp theo, chúng ta đi vào khái niệm **Load Balancing – Cân bằng tải**.

**Load Balancer là các server có nhiệm vụ chuyển tiếp traffic đến nhiều server phía sau**, ví dụ như các EC2 instance.

Hình dung đơn giản: thay vì người dùng gọi trực tiếp đến từng EC2 instance, họ sẽ gọi đến Load Balancer. Sau đó, Load Balancer sẽ phân phối các request đó cho nhiều EC2 instance phía sau, đảm bảo không có một instance nào bị quá tải.

---

## 📌 Slide 55: Tại sao nên dùng Load Balancer?

Có rất nhiều lý do để sử dụng Load Balancer:

- **Phân phối tải đều** trên nhiều instance phía sau
- **Cung cấp một điểm truy cập duy nhất (DNS)** cho ứng dụng của bạn
- **Xử lý lỗi instance một cách mượt mà** – nếu một instance hỏng, traffic tự động chuyển sang instance khác
- **Thực hiện health check thường xuyên** để kiểm tra tình trạng instance
- **Cung cấp SSL termination (HTTPS)** cho website
- **Hỗ trợ Stickiness với cookies** – đảm bảo cùng một user luôn được route về cùng instance
- **High Availability across zones** – tính sẵn sàng cao trên nhiều AZ
- **Tách biệt traffic public và private**

Tóm lại, Load Balancer giúp hệ thống của bạn vừa mạnh, vừa bền, vừa an toàn.

---

## 📌 Slide 56: Tại sao dùng Elastic Load Balancer (ELB)?

Tại sao nên dùng **Elastic Load Balancer của AWS** thay vì tự dựng?

ELB là một **managed load balancer**, nghĩa là AWS quản lý hộ bạn:
- AWS **đảm bảo nó luôn hoạt động**
- AWS **lo việc nâng cấp, bảo trì, high availability**
- AWS chỉ cung cấp một số tùy chỉnh cấu hình cơ bản

Bạn cũng có thể tự dựng một load balancer, chi phí có thể rẻ hơn, nhưng **bạn sẽ phải tốn rất nhiều công sức** để duy trì.

Một điểm cộng lớn: ELB **tích hợp sẵn với rất nhiều dịch vụ AWS khác** như: EC2, EC2 Auto Scaling Groups, Amazon ECS, AWS Certificate Manager (ACM), CloudWatch, Route 53, AWS WAF, AWS Global Accelerator…

Nói chung, dùng ELB là lựa chọn hợp lý cho hầu hết trường hợp.

---

## 📌 Slide 57: Health Checks – Kiểm tra sức khỏe

**Health Check** là một tính năng cực kỳ quan trọng đối với Load Balancer.

Health Check giúp Load Balancer biết được instance phía sau có **đang hoạt động và sẵn sàng nhận request hay không**.

Cách hoạt động: Health Check được thực hiện trên một **port** và một **route** cụ thể – thường là `/health`.

Quy tắc rất đơn giản: nếu instance trả về mã **không phải 200 (OK)**, thì instance đó được coi là **unhealthy**, và Load Balancer sẽ ngừng gửi traffic đến nó.

Ví dụ: Load Balancer gọi vào `http://instance-ip:4567/health`. Nếu nhận về 200, instance OK. Nếu không, instance đó bị "đánh dấu xấu".

---

## 📌 Slide 58: Các loại Load Balancer trên AWS

AWS có **4 loại managed Load Balancer**:

1. **Classic Load Balancer (CLB)** – ra mắt 2009, thế hệ cũ (v1)
   - Hỗ trợ HTTP, HTTPS, TCP, SSL

2. **Application Load Balancer (ALB)** – ra mắt 2016, thế hệ mới (v2)
   - Hỗ trợ HTTP, HTTPS, WebSocket

3. **Network Load Balancer (NLB)** – ra mắt 2017, thế hệ mới (v2)
   - Hỗ trợ TCP, TLS (TCP bảo mật), UDP

4. **Gateway Load Balancer (GWLB)** – ra mắt 2020
   - Hoạt động ở Layer 3 (Network layer) – giao thức IP

Lời khuyên: **nên dùng các thế hệ mới (ALB, NLB, GWLB)** vì chúng có nhiều tính năng hơn.

Một điểm thú vị: một số Load Balancer có thể được cấu hình là **internal (private)** hoặc **external (public)**, tùy theo nhu cầu của bạn.

---

## 📌 Slide 59: Load Balancer Security Groups

Khi dùng Load Balancer, ta cần cấu hình **Security Group** đúng cách.

Sơ đồ điển hình:

- **Security Group của Load Balancer**: cho phép HTTPS hoặc HTTP từ bất kỳ đâu (`0.0.0.0/0`) – vì Load Balancer là điểm tiếp xúc với internet.

- **Security Group của Application (EC2)**: chỉ cho phép traffic **từ Security Group của Load Balancer**, không cho phép truy cập trực tiếp từ internet.

Đây là một best practice rất quan trọng: **EC2 không bao giờ nên expose trực tiếp ra internet**, mà phải đi qua Load Balancer.

---

## 📌 Slide 60: Application Load Balancer (ALB) – Phần 1

Bây giờ ta đi sâu hơn vào **Application Load Balancer**.

ALB hoạt động ở **Layer 7 – tức là tầng HTTP**. Đây là điểm khác biệt quan trọng so với NLB.

Các tính năng nổi bật của ALB:
- **Cân bằng tải cho nhiều ứng dụng HTTP** trên nhiều máy khác nhau – thông qua khái niệm **target group**
- **Cân bằng tải cho nhiều ứng dụng trên cùng một máy** – ví dụ với container
- **Hỗ trợ HTTP/2 và WebSocket**
- **Hỗ trợ redirect** – ví dụ tự động chuyển từ HTTP sang HTTPS

Có thể nói, ALB là sự lựa chọn mặc định cho hầu hết các ứng dụng web hiện đại.

---

## 📌 Slide 61: Application Load Balancer (ALB) – Phần 2: Routing

Một sức mạnh lớn của ALB là **khả năng routing thông minh** đến các target group khác nhau:

- **Routing dựa trên path trong URL**: ví dụ `example.com/users` đi đến một group, `example.com/orders` đi đến group khác.
- **Routing dựa trên hostname**: ví dụ `one.example.com` và `other.example.com` đi đến các group khác nhau.
- **Routing dựa trên Query String hoặc Headers**: ví dụ `example.com/users?id=123&order=false`.

Vì sự linh hoạt này, **ALB rất phù hợp với kiến trúc microservices và container-based application** (ví dụ Docker và Amazon ECS).

ALB còn có **tính năng port mapping** để redirect đến port động trong ECS.

So sánh: với CLB, bạn cần nhiều CLB cho nhiều ứng dụng, còn với ALB chỉ cần một là đủ.

---

## 📌 Slide 62: ALB – HTTP Based Traffic (Sơ đồ minh họa)

Đây là sơ đồ minh họa cách ALB hoạt động với HTTP traffic.

Người dùng từ internet gửi request đến **External Application Load Balancer**:

- Nếu request đi vào path `/user`, ALB sẽ chuyển HTTP traffic đến **Target Group cho ứng dụng Users**, kèm health check.
- Nếu request đi vào path `/search`, ALB sẽ chuyển đến **Target Group cho ứng dụng Search**, cũng có health check riêng.

Mỗi target group có **health check độc lập**, đảm bảo từng nhóm ứng dụng được giám sát riêng biệt.

---

## 📌 Slide 63: ALB – Target Groups

**Target Groups** trong ALB có thể bao gồm những loại nào?

- **EC2 instances** – có thể được quản lý bởi Auto Scaling Group, dùng giao thức HTTP
- **ECS tasks** – được quản lý bởi chính ECS, dùng HTTP
- **Lambda functions** – HTTP request sẽ được dịch thành một JSON event để Lambda xử lý
- **IP Addresses** – nhưng phải là private IP

Một điểm lưu ý: **một ALB có thể route đến nhiều target group**, và **health check được thực hiện ở cấp target group**, không phải cấp ALB.

---

## 📌 Slide 64: ALB – Query Strings/Parameters Routing

Một tính năng cực kỳ hữu ích: **Routing dựa trên Query String**.

Ví dụ thực tế:
- Khi request có `?Platform=Mobile`, ALB sẽ chuyển đến **Target Group 1** chứa các EC2 instance phục vụ mobile.
- Khi request có `?Platform=Desktop`, ALB sẽ chuyển đến **Target Group 2** – có thể là on-premises với private IP routing.

Tính năng này rất hữu ích khi bạn muốn phục vụ các loại client khác nhau với hạ tầng riêng biệt.

---

## 📌 Slide 65: ALB – Good to Know (Những điều cần biết)

Một số điểm thú vị về ALB:

- ALB có **hostname cố định** dạng `XXX.region.elb.amazonaws.com`.

- **Application server không nhìn thấy IP của client trực tiếp**. Vì sao? Vì traffic đi qua ALB, nên server chỉ thấy IP của Load Balancer (private IP).

- Vậy làm sao biết IP thật của client? AWS chèn vào **header X-Forwarded-For** với IP gốc của client. Ngoài ra còn có:
  - **X-Forwarded-Port** – port của client
  - **X-Forwarded-Proto** – giao thức (HTTP hay HTTPS)

Đây là điểm rất quan trọng khi viết ứng dụng – bạn phải đọc các header này nếu muốn lấy IP thật của user.

---

## 📌 Slide 66: Network Load Balancer (NLB) – Phần 1

Tiếp theo là **Network Load Balancer**, hoạt động ở **Layer 4**.

NLB cho phép:
- **Forward TCP và UDP traffic** đến các instance
- **Xử lý hàng triệu request mỗi giây** – con số rất ấn tượng!
- **Độ trễ cực thấp (ultra-low latency)**

Một đặc điểm quan trọng: NLB có **một static IP cho mỗi AZ**, và hỗ trợ gán **Elastic IP** – rất hữu ích khi cần whitelist IP cụ thể trên tường lửa.

NLB phù hợp khi nào? Khi bạn cần **hiệu năng cực cao**, hoặc traffic là **TCP/UDP** chứ không phải HTTP.

---

## 📌 Slide 67: NLB – TCP (Layer 4) Based Traffic

Sơ đồ minh họa NLB:

Người dùng từ internet gửi TCP traffic đến **External Network Load Balancer**:
- Theo các quy tắc TCP (TCP + Rules), NLB chuyển đến **Target Group** cho ứng dụng Users (giao thức TCP)
- Hoặc chuyển đến **Target Group** cho ứng dụng Search (có thể là HTTP)

Mỗi target group đều có **health check riêng**, đảm bảo độ tin cậy cao.

---

## 📌 Slide 68: NLB – Target Groups

Target Groups của NLB có thể là:

- **EC2 instances**
- **IP Addresses** – phải là private IP
- **Application Load Balancer** – đúng vậy, NLB có thể trỏ đến một ALB ở phía sau!

Health check của NLB hỗ trợ các giao thức: **TCP, HTTP và HTTPS**.

Việc NLB có thể trỏ đến ALB là một mẫu kiến trúc rất hay – kết hợp được cả static IP của NLB với khả năng routing thông minh của ALB.

---

## 📌 Slide 69: Gateway Load Balancer (GWLB)

**Gateway Load Balancer** là loại đặc biệt nhất.

Mục đích: dùng để **triển khai, mở rộng và quản lý đội các thiết bị mạng ảo bên thứ ba (3rd party network virtual appliances)** trong AWS.

Ví dụ: **Firewall, Intrusion Detection và Prevention Systems (IDS/IPS), Deep Packet Inspection Systems, payload manipulation**, v.v.

GWLB hoạt động ở **Layer 3 (Network Layer) – cấp IP packet**.

GWLB kết hợp 2 chức năng:
- **Transparent Network Gateway**: là điểm vào/ra duy nhất cho toàn bộ traffic
- **Load Balancer**: phân phối traffic đến các virtual appliance

GWLB sử dụng **giao thức GENEVE trên port 6081**.

Cách hoạt động: tất cả traffic từ User đến Application sẽ đi qua GWLB, được chuyển đến các virtual appliance bảo mật để kiểm tra, sau đó mới đến đích.

---

## 📌 Slide 70: Gateway Load Balancer – Target Groups

Target Groups của GWLB hỗ trợ:
- **EC2 instances**
- **IP Addresses** – phải là private IP

Đơn giản hơn so với ALB và NLB, vì GWLB chuyên dụng cho mục đích bảo mật mạng.

---

## 📌 Slide 71: Sticky Sessions (Session Affinity)

**Sticky Sessions**, hay còn gọi là **Session Affinity**, là một tính năng quan trọng.

Ý tưởng: bạn có thể cấu hình để **cùng một client luôn được điều hướng đến cùng một instance** phía sau Load Balancer.

Tính năng này hoạt động với:
- **Classic Load Balancer (CLB)**
- **Application Load Balancer (ALB)**
- **Network Load Balancer (NLB)**

Với CLB và ALB, **cookie** được dùng để duy trì stickiness, và **bạn có thể kiểm soát thời gian hết hạn** của cookie này.

**Use case** điển hình: đảm bảo user **không mất session data** khi đang sử dụng ứng dụng – ví dụ khi đang đăng nhập hoặc đang điền form.

Tuy nhiên, có một nhược điểm: **bật stickiness có thể gây mất cân bằng tải** trên các EC2 instance phía sau, vì một số instance có thể nhận nhiều "khách quen" hơn.

---

## 📌 Slide 72: Sticky Sessions – Cookie Names

Có hai loại cookie cho Sticky Sessions:

**1. Application-based Cookies:**

- **Custom cookie**: 
  - Do **target tự sinh ra**
  - Có thể chứa các attribute tùy chỉnh
  - **Tên cookie phải được đặt riêng cho từng target group**
  - **Lưu ý quan trọng**: KHÔNG được dùng tên `AWSALB`, `AWSALBAPP`, `AWSALBTG` – đây là các tên đã được AWS dành riêng cho ELB.

- **Application cookie**:
  - Do **Load Balancer tự sinh ra**
  - Tên cố định là `AWSALBAPP`

**2. Duration-based Cookies:**
- Do Load Balancer sinh ra
- Tên là `AWSALB` cho ALB, hoặc `AWSELB` cho CLB

Hiểu rõ các loại cookie này rất hữu ích khi bạn debug session.

---

## 📌 Slide 73: Cross-Zone Load Balancing

**Cross-Zone Load Balancing** quyết định cách Load Balancer phân phối traffic giữa các AZ.

Có Cross-Zone: traffic được chia đều cho **tất cả instance trên tất cả AZ**.
Không có Cross-Zone: mỗi node của Load Balancer chỉ chia traffic cho instance **trong cùng AZ**.

Cấu hình mặc định cho từng loại:

**Application Load Balancer:**
- **Bật mặc định** (có thể tắt ở cấp Target Group)
- **Không tính phí** cho data giữa các AZ

**Network Load Balancer & Gateway Load Balancer:**
- **Tắt mặc định**
- **Có tính phí** cho data giữa các AZ nếu bạn bật

**Classic Load Balancer:**
- **Tắt mặc định**
- **Không tính phí** cho data giữa các AZ nếu bật

Đây là chi tiết quan trọng cần nhớ về mặt **chi phí** khi thiết kế hệ thống.

---

## 📌 Slide 74: SSL/TLS – Cơ bản

Chuyển sang chủ đề **SSL/TLS**.

**SSL Certificate** cho phép traffic giữa client và Load Balancer được **mã hóa khi truyền (in-flight encryption)** – đây là điều cực kỳ quan trọng cho bảo mật.

Một chút lịch sử:
- **SSL** là viết tắt của **Secure Sockets Layer**, dùng để mã hóa kết nối.
- **TLS** là **Transport Layer Security**, là phiên bản mới hơn.

Ngày nay, **chứng chỉ TLS được dùng phổ biến**, nhưng người ta vẫn quen gọi là **SSL certificate**.

Public SSL certificate được cấp bởi các **Certificate Authority (CA)** như: Comodo, Symantec, GoDaddy, GlobalSign, Digicert, Let's Encrypt…

Lưu ý quan trọng: SSL certificate **có ngày hết hạn** (do bạn đặt) và phải được **gia hạn định kỳ**.

---

## 📌 Slide 75: Load Balancer – SSL Certificates

Cách hoạt động của SSL trên Load Balancer:

- Phía client: **HTTPS được mã hóa**, đi qua internet.
- Phía sau Load Balancer: traffic là **HTTP không mã hóa**, vì nó đi trong **VPC private** – một môi trường an toàn rồi.

Chi tiết kỹ thuật:
- Load Balancer dùng **chứng chỉ X.509** (SSL/TLS server certificate)
- Bạn có thể quản lý chứng chỉ thông qua **ACM (AWS Certificate Manager)** – rất tiện lợi
- Hoặc bạn có thể tự upload chứng chỉ riêng

Với **HTTPS listener**, bạn cần:
- Chỉ định một **chứng chỉ mặc định**
- Có thể thêm **danh sách chứng chỉ tùy chọn** để hỗ trợ nhiều domain
- Client có thể dùng **SNI (Server Name Indication)** để báo hostname họ muốn truy cập
- Có thể đặt **security policy** để hỗ trợ các phiên bản SSL/TLS cũ (cho legacy client)

---

## 📌 Slide 76: SSL – Server Name Indication (SNI)

**SNI – Server Name Indication** là một tính năng quan trọng.

Vấn đề mà SNI giải quyết: làm sao **load nhiều SSL certificate trên cùng một web server** để phục vụ **nhiều website** khác nhau?

SNI là một giao thức mới, **yêu cầu client phải báo hostname của server đích trong giai đoạn SSL handshake ban đầu**. Sau đó, server sẽ tìm chứng chỉ phù hợp, hoặc trả về chứng chỉ mặc định.

Ví dụ minh họa: client nói "Tôi muốn truy cập `www.mycorp.com`". ALB nhận được, nhìn vào kho chứng chỉ, tìm thấy cái cho `www.mycorp.com`, và trả về đúng chứng chỉ đó.

**Điểm quan trọng cần nhớ:**
- SNI **CHỈ hoạt động** với **ALB và NLB (thế hệ mới), và CloudFront**
- SNI **KHÔNG hoạt động** với **CLB (thế hệ cũ)**

Đây là một câu hỏi rất hay xuất hiện trong đề thi chứng chỉ AWS.

---

## 📌 Slide 77: ELB – SSL Certificates Summary

Tổng kết khả năng SSL của các loại ELB:

**Classic Load Balancer (v1):**
- Chỉ hỗ trợ **một** SSL certificate
- Nếu cần nhiều hostname với nhiều chứng chỉ, **phải dùng nhiều CLB**

**Application Load Balancer (v2):**
- Hỗ trợ **nhiều listener với nhiều SSL certificate**
- Sử dụng **SNI** để hoạt động

**Network Load Balancer (v2):**
- Hỗ trợ **nhiều listener với nhiều SSL certificate**
- Cũng dùng **SNI**

Tóm lại: nếu bạn muốn quản lý nhiều domain trên một Load Balancer, hãy dùng ALB hoặc NLB.

---

## 📌 Slide 78: Connection Draining

**Connection Draining** là một tính năng rất hữu ích để xử lý các kết nối đang hoạt động khi instance bị remove.

Đầu tiên là về tên gọi:
- Trong **CLB**: gọi là **Connection Draining**
- Trong **ALB và NLB**: gọi là **Deregistration Delay**
- Bản chất giống nhau, chỉ khác tên.

Connection Draining là gì? Là **khoảng thời gian cho phép các request đang xử lý được hoàn thành**, trong khi instance đang bị **deregister hoặc bị đánh dấu unhealthy**.

Cơ chế hoạt động:
- Load Balancer **ngừng gửi request mới** đến instance đang bị deregister.
- Nhưng các request **đang xử lý dở** thì vẫn được hoàn thành.
- Trong khi đó, **request mới sẽ được chuyển đến các instance khác**.

Cấu hình:
- Giá trị từ **1 đến 3600 giây**, mặc định là **300 giây** (5 phút).
- Có thể **tắt** bằng cách đặt giá trị 0.
- Lời khuyên: **đặt giá trị thấp nếu request của bạn ngắn**, và đặt cao nếu request có thể chạy lâu (ví dụ upload file lớn).

Tính năng này giúp **tránh việc cắt đột ngột** kết nối của user, mang lại trải nghiệm mượt mà hơn.

---

## 📌 Slide 79: Auto Scaling Group là gì?

Bây giờ chuyển sang phần quan trọng tiếp theo: **Auto Scaling Group (ASG)**.

Trong thực tế, **tải trên website hoặc ứng dụng của bạn có thể thay đổi liên tục** – ban ngày đông, ban đêm vắng, ngày lễ tăng vọt, v.v.

Trong cloud, ta có lợi thế là **có thể tạo và xóa server rất nhanh**. Auto Scaling Group sinh ra để khai thác lợi thế này.

Mục tiêu của ASG:
- **Scale Out** – thêm EC2 instance khi tải tăng
- **Scale In** – bớt EC2 instance khi tải giảm
- **Đảm bảo số lượng tối thiểu và tối đa** của instance đang chạy
- **Tự động đăng ký instance mới với Load Balancer**
- **Tự tạo lại instance khi instance cũ bị terminate** (ví dụ vì unhealthy)

Một tin tốt: **ASG hoàn toàn miễn phí** – bạn chỉ trả tiền cho EC2 instance bên dưới.

---

## 📌 Slide 80: Auto Scaling Group trong AWS – Sơ đồ

Đây là sơ đồ tổng quan của Auto Scaling Group.

Trong một ASG, có 3 mức quan trọng:

- **Minimum Capacity (sức chứa tối thiểu)**: số instance ít nhất luôn phải chạy. Ví dụ 2 instance.

- **Desired Capacity (sức chứa mong muốn)**: số instance hiện tại mong muốn. Ví dụ 4.

- **Maximum Capacity (sức chứa tối đa)**: giới hạn trên – ASG không bao giờ vượt quá con số này. Ví dụ 7.

Khi tải tăng, ASG sẽ **scale out** từ Desired Capacity lên đến Maximum Capacity.
Khi tải giảm, ASG **scale in** xuống đến Minimum Capacity, không bao giờ thấp hơn.

Hệ thống này đảm bảo bạn luôn có đủ tài nguyên, nhưng cũng không lãng phí.

---

## 📌 Slide 81: Auto Scaling Group với Load Balancer

Đây là kiến trúc kinh điển kết hợp **ASG + Load Balancer**.

User truy cập vào **Elastic Load Balancer**. ELB có khả năng **kiểm tra health của các EC2 instance** trong Auto Scaling Group.

ASG quản lý các EC2 instance: nếu một instance unhealthy, ELB ngừng gửi traffic, ASG terminate và tạo instance mới. Khi tải tăng, ASG tạo thêm instance, và ELB tự động phát hiện và phân phối traffic.

Đây là **mô hình chuẩn vàng** cho mọi ứng dụng web hiện đại trên AWS: **Load Balancer + Auto Scaling Group + EC2 đa AZ**.

---

## 📌 Slide 82: Auto Scaling Group – Attributes (Thuộc tính)

ASG có những thuộc tính gì?

**Launch Template** (Launch Configuration cũ đã deprecated):
Đây là "công thức" để tạo EC2 instance, bao gồm:
- **AMI và Instance Type**
- **EC2 User Data** (script chạy khi khởi động)
- **EBS Volumes**
- **Security Groups**
- **SSH Key Pair**
- **IAM Roles cho EC2 Instance**
- **Network và Subnet Information**
- **Load Balancer Information**

Ngoài ra:
- **Min Size / Max Size / Initial Capacity**
- **Scaling Policies** – chính sách tự động mở rộng

Khi ASG cần tạo instance mới, nó dùng Launch Template này để biết phải tạo instance như thế nào.

---

## 📌 Slide 83: Auto Scaling – CloudWatch Alarms & Scaling

ASG có thể được **tự động scale dựa trên CloudWatch Alarm**.

Cách hoạt động:
- Một **alarm giám sát một metric** – ví dụ Average CPU, hoặc một metric tùy chỉnh
- Metric như **Average CPU được tính trên toàn bộ instance trong ASG**
- Dựa trên alarm:
  - Có thể tạo **scale-out policy**: tăng số instance
  - Có thể tạo **scale-in policy**: giảm số instance

Ví dụ thực tế: nếu Average CPU vượt 70%, alarm bật → ASG thêm 2 instance. Nếu CPU dưới 30%, alarm bật → ASG bớt 1 instance.

Đây là cách phổ biến nhất để tự động hóa scaling.

---

## 📌 Slide 84: Auto Scaling Group – Scaling Policies (Phần 1)

Các loại **Scaling Policies** của ASG:

**1. Dynamic Scaling – Mở rộng động:**

- **Target Tracking Scaling**:
  - **Đơn giản, dễ cấu hình nhất**
  - Ví dụ: "Tôi muốn Average CPU của ASG luôn ở khoảng 40%". AWS tự lo việc còn lại.

- **Simple / Step Scaling**:
  - Khi CloudWatch alarm trigger (ví dụ CPU > 70%), thêm 2 instance
  - Khi alarm khác trigger (CPU < 30%), bớt 1 instance

**2. Scheduled Scaling – Mở rộng theo lịch:**
- **Dùng để dự đoán scaling dựa trên pattern sử dụng đã biết**
- Ví dụ: tăng min capacity lên 10 vào 5 giờ chiều thứ Sáu hàng tuần (vì biết lúc đó traffic tăng).

Bạn có thể **kết hợp các policy này** để tối ưu chi phí và hiệu năng.

---

## 📌 Slide 85: Auto Scaling Group – Scaling Policies (Phần 2)

**Predictive Scaling – Mở rộng dựa trên dự đoán:**

Đây là loại scaling thông minh nhất.

**Predictive Scaling liên tục dự báo tải và lên lịch scaling trước**.

AWS sử dụng machine learning để phân tích pattern lịch sử, dự đoán khi nào tải sẽ tăng, và **chuẩn bị instance trước khi tải thực sự đến**.

Đây là tính năng cực kỳ mạnh mẽ cho các ứng dụng có pattern sử dụng có thể dự đoán được – ví dụ trang thương mại điện tử với traffic peak vào ngày Black Friday.

---

## 📌 Slide 86: Good metrics to scale on – Các metric tốt để scale

Các metric tốt để dùng cho scaling:

- **CPUUtilization**: Average CPU trên các instance. **Đây là metric phổ biến nhất**.

- **RequestCountPerTarget**: đảm bảo **số request trên mỗi EC2 instance ổn định**. Ví dụ: bạn muốn mỗi instance xử lý khoảng 3 request – nếu vượt, scale out; nếu thấp hơn, scale in.

- **Average Network In / Out**: dùng khi ứng dụng của bạn **bị giới hạn bởi mạng (network bound)**, ví dụ ứng dụng truyền video.

- **Custom metric**: bất kỳ metric tùy chỉnh nào bạn **push lên CloudWatch**. Ví dụ: số user đang đăng nhập, độ dài hàng đợi xử lý, v.v.

Lời khuyên: **chọn metric phản ánh đúng bottleneck của ứng dụng**. Nếu app bị nghẽn vì CPU, dùng CPU. Nếu nghẽn vì I/O, dùng metric mạng. Nếu logic phức tạp, dùng custom metric.

---

## 📌 Slide 87: Auto Scaling Group – Scaling Cooldowns

Cuối cùng, ta nói về **Scaling Cooldowns – Thời gian nghỉ giữa các lần scaling**.

Vấn đề: nếu ASG scale liên tục mỗi giây, sẽ rất bất ổn và tốn kém. Nên cần có khoảng "nghỉ".

Cơ chế hoạt động:
- Sau khi xảy ra một hoạt động scaling, ASG vào **cooldown period (mặc định 300 giây = 5 phút)**.
- Trong thời gian cooldown, **ASG sẽ KHÔNG launch hay terminate thêm instance** – để **các metric có thời gian ổn định lại**.

Sơ đồ logic:
- Có hành động scaling cần thực hiện?
- Đang trong cooldown? **Có** → bỏ qua (Ignore Action). **Không** → thực hiện launch hoặc terminate.

**Lời khuyên hữu ích:** dùng **AMI đã chuẩn bị sẵn (ready-to-use)** để **giảm thời gian cấu hình instance**, từ đó:
- Instance mới phục vụ request nhanh hơn
- Có thể giảm cooldown period xuống

Đây là một tip rất quan trọng để tối ưu scaling trong production.

---

## 🎯 Tổng kết phần High Availability & Scalability

Vậy là chúng ta đã đi qua toàn bộ phần High Availability & Scalability cho EC2. Tổng kết nhanh:

- **Scalability** có hai loại: Vertical (tăng kích thước) và Horizontal (tăng số lượng).
- **High Availability** đạt được bằng cách triển khai trên nhiều AZ.
- **Load Balancer** phân phối traffic, có 4 loại: CLB, ALB, NLB, GWLB.
- **Auto Scaling Group** tự động tăng giảm số lượng instance dựa trên metric.
- Kết hợp **ALB + ASG multi-AZ** là kiến trúc chuẩn cho web app trên AWS.

Cảm ơn mọi người đã lắng nghe! Mọi người có câu hỏi gì không ạ?

---

## 📝 GHI CHÚ KHI THUYẾT TRÌNH

**Mẹo trình bày:**
- Mỗi slide nên thuyết trình trong khoảng **1-2 phút**, tổng thời gian khoảng **45-60 phút** cho 40 slides.
- Khi đến các slide có sơ đồ (slide 54, 62, 67, 80, 81), hãy **chỉ tay vào sơ đồ** và giải thích từng phần.
- Các slide so sánh (slide 58 – 4 loại LB, slide 73 – Cross-Zone, slide 77 – SSL summary) là những nội dung **thường ra trong đề thi AWS**, nên nhấn mạnh.
- Hãy hỏi audience câu hỏi tương tác ở các điểm chuyển: ví dụ "Theo các bạn, khi nào nên dùng NLB thay vì ALB?"
- Chuẩn bị **2-3 câu chuyện thực tế** để minh họa, ví dụ: "Có một dự án mình từng làm, lưu lượng tăng đột ngột vào dịp khuyến mãi, nhờ ASG mà hệ thống không sập…"

**Chúc bạn thuyết trình thành công!** 🎉