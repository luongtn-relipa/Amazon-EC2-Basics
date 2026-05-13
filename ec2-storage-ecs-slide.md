# Bài Thuyết Trình: EC2 Instance Storage & Containers on AWS
> Tài liệu giải thích từng slide — phiên bản tiếng Việt

---

## Slide 1 — Tiêu đề: Amazon EC2 – Instance Storage

**Nội dung cần trình bày:**

Xin chào mọi người! Trong phần này, chúng ta sẽ đi sâu vào chủ đề **lưu trữ của Amazon EC2 — EC2 Instance Storage**.

Đây là một trong những chủ đề quan trọng khi làm việc với AWS, vì bất kỳ ứng dụng nào cũng cần nơi để lưu dữ liệu. Chúng ta sẽ tìm hiểu các loại storage khác nhau mà AWS cung cấp cho EC2, bao gồm EBS, EFS, và EC2 Instance Store, sau đó chuyển sang phần Containers trên AWS với ECS, EKS và các dịch vụ liên quan.

---

## Slide 2 — EBS Volume là gì?

**Nội dung cần trình bày:**

**EBS — Elastic Block Store** là một **ổ đĩa mạng (network drive)** mà bạn có thể gắn vào EC2 instance trong khi instance đang chạy.

Có một vài điểm quan trọng cần ghi nhớ:

- **Dữ liệu được giữ lại** ngay cả khi instance bị xóa (terminate). Đây là điểm khác biệt so với bộ nhớ tạm thời.
- EBS **bị khóa trong một Availability Zone (AZ)** — nghĩa là một EBS ở `us-east-1a` không thể gắn trực tiếp vào instance ở `us-east-1b`.
- Ở cấp độ CCP (Cloud Practitioner), **mỗi EBS chỉ gắn được vào một instance tại một thời điểm**.

Hãy tưởng tượng EBS như một ổ SSD hoặc HDD nhưng kết nối qua mạng thay vì cắm trực tiếp vào máy tính.

---

## Slide 3 — EBS Volume – Ví dụ minh họa

**Nội dung cần trình bày:**

Slide này minh họa cách các EBS Volume hoạt động theo từng Availability Zone.

- Trong **US-EAST-1A**: có các EBS 50 GB, 50 GB, và 10 GB đang không được gắn (unattached).
- Trong **US-EAST-1B**: có các EBS 10 GB và 100 GB.

Điều cần chú ý: các volume **không thể di chuyển trực tiếp** giữa các AZ. Nếu muốn dùng dữ liệu ở AZ khác, bạn cần tạo **Snapshot** rồi khôi phục ở AZ đích — chúng ta sẽ nói đến ở slide tiếp theo.

---

## Slide 4 — EBS Snapshots

**Nội dung cần trình bày:**

**EBS Snapshot** là bản sao lưu (backup) của EBS Volume tại một thời điểm cụ thể.

Các điểm quan trọng:

- Bạn **không cần phải detach** volume khỏi instance trước khi snapshot, nhưng AWS **khuyến nghị** nên làm vậy để đảm bảo tính toàn vẹn dữ liệu.
- Snapshot được lưu trên **S3** và có thể **copy sang AZ khác hoặc Region khác**.

Ứng dụng thực tế: nếu bạn muốn di chuyển dữ liệu từ `us-east-1a` sang `us-east-1b`, hãy tạo snapshot từ volume nguồn, rồi restore thành volume mới ở AZ đích.

---

## Slide 5 — EBS Snapshots – Các tính năng nâng cao

**Nội dung cần trình bày:**

AWS cung cấp thêm ba tính năng đáng chú ý cho EBS Snapshots:

**1. EBS Snapshot Archive**
Cho phép chuyển snapshot sang tầng lưu trữ "archive" — rẻ hơn **75%** so với thông thường. Tuy nhiên, khi cần khôi phục, sẽ mất từ **24 đến 72 giờ**.

**2. Recycle Bin for EBS Snapshots**
Giống như thùng rác trên máy tính — khi bạn xóa snapshot nhầm, nó vẫn được giữ lại trong Recycle Bin. Bạn có thể cấu hình thời gian lưu giữ từ **1 ngày đến 1 năm**.

**3. Fast Snapshot Restore (FSR)**
Giúp snapshot sẵn sàng sử dụng ngay lập tức, không có độ trễ khi dùng lần đầu. Tuy nhiên, tính năng này **rất tốn phí ($$$)**, chỉ dùng khi thực sự cần thiết.

---

## Slide 6 — Các loại EBS Volume

**Nội dung cần trình bày:**

EBS có **6 loại volume** chính, chia thành hai nhóm: **SSD** và **HDD**.

| Loại | Công nghệ | Mục đích |
|------|-----------|----------|
| gp2 / gp3 | SSD | Đa năng, cân bằng giá và hiệu năng |
| io1 / io2 Block Express | SSD | Hiệu năng cao, mission-critical |
| st1 | HDD | Throughput cao, truy cập thường xuyên |
| sc1 | HDD | Chi phí thấp nhất, ít truy cập |

Các volume được đặc trưng bởi ba thông số: **Size** (dung lượng), **Throughput** (băng thông), và **IOPS** (số lượng thao tác I/O mỗi giây).

Quan trọng: **Chỉ có gp2/gp3 và io1/io2** mới có thể dùng làm **boot volume** (ổ khởi động hệ điều hành).

---

## Slide 7 — EBS Volume: General Purpose SSD (gp2/gp3)

**Nội dung cần trình bày:**

Đây là loại volume **phổ biến nhất**, phù hợp với hầu hết các use case thông thường.

**gp3** (thế hệ mới hơn):
- Baseline: **3,000 IOPS** và **125 MiB/s** throughput.
- Có thể tăng độc lập lên **16,000 IOPS** và **1,000 MiB/s** — mà không cần tăng dung lượng.

**gp2** (thế hệ cũ):
- IOPS gắn liền với dung lượng: cứ **3 IOPS mỗi GiB**.
- IOPS tối đa là 16,000, đạt được khi volume đạt ~5,334 GiB.

**Lời khuyên**: Nếu tạo mới, hãy chọn **gp3** — linh hoạt hơn và thường rẻ hơn gp2.

---

## Slide 8 — EBS Volume: Provisioned IOPS SSD (io1/io2)

**Nội dung cần trình bày:**

Đây là loại volume dành cho **ứng dụng critical** đòi hỏi IOPS ổn định và cao.

**io1** (4 GiB – 16 TiB):
- Max IOPS: **64,000** cho Nitro EC2 instances, **32,000** cho các loại khác.
- Có thể tăng IOPS độc lập với dung lượng.

**io2 Block Express** (4 GiB – 64 TiB):
- Độ trễ **dưới mili-giây (sub-millisecond)**.
- Max IOPS: **256,000** với tỉ lệ 1,000 IOPS trên mỗi GiB.
- Hỗ trợ **EBS Multi-Attach** — gắn cùng lúc vào nhiều instance.

**Use case điển hình**: Cơ sở dữ liệu lớn như Oracle, SAP HANA — những hệ thống nhạy cảm với hiệu năng I/O.

---

## Slide 9 — EBS Volume: Hard Disk Drives (HDD)

**Nội dung cần trình bày:**

HDD không thể dùng làm **boot volume**, nhưng rất phù hợp cho dữ liệu lớn và chi phí thấp.

**st1 — Throughput Optimized HDD**:
- Dành cho Big Data, Data Warehouses, Log Processing.
- Max throughput: **500 MiB/s**, max IOPS: **500**.

**sc1 — Cold HDD**:
- Dành cho dữ liệu ít được truy cập.
- **Chi phí thấp nhất** trong các loại EBS.
- Max throughput: **250 MiB/s**, max IOPS: **250**.

**Ghi nhớ**: Khi cần lưu dữ liệu lớn mà không cần tốc độ cao, hãy chọn HDD để tiết kiệm chi phí.

---

## Slide 10 — EBS Volume Types – Bảng tóm tắt

**Nội dung cần trình bày:**

Slide này hiển thị bảng so sánh tổng hợp tất cả các loại EBS Volume từ tài liệu chính thức của AWS.

Bảng giúp so sánh trực quan các thông số như: dung lượng tối đa/tối thiểu, IOPS tối đa, throughput, và các trường hợp sử dụng phù hợp.

**Tham khảo thêm**: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html

---

## Slide 11 — EBS Multi-Attach – io1/io2

**Nội dung cần trình bày:**

**EBS Multi-Attach** cho phép gắn **cùng một EBS volume** vào **nhiều EC2 instance** trong cùng một AZ.

Mỗi instance đều có **quyền đọc/ghi đầy đủ** vào volume đó.

**Use case**:
- Ứng dụng Linux dạng cluster cần tính sẵn sàng cao (ví dụ: Teradata).
- Tối đa **16 EC2 instances** cùng lúc.

**Lưu ý quan trọng**: Ứng dụng phải tự xử lý các thao tác ghi đồng thời (concurrent writes), và phải dùng **file system cluster-aware** — không dùng XFS hay EXT4 thông thường.

---

## Slide 12 — EBS Encryption (Mã hóa)

**Nội dung cần trình bày:**

Khi bạn tạo một **EBS volume được mã hóa**, AWS tự động xử lý toàn bộ — bạn không cần làm gì thêm.

Những gì được mã hóa:
- **Dữ liệu lưu trữ** bên trong volume.
- **Dữ liệu truyền tải** giữa instance và volume.
- **Tất cả snapshot** tạo từ volume đó.
- **Các volume mới** tạo từ snapshot đó.

Điểm đáng chú ý:
- Mã hóa sử dụng **KMS với chuẩn AES-256**.
- **Ảnh hưởng tới hiệu năng là tối thiểu** — không đáng lo ngại.
- Có thể mã hóa một snapshot không được mã hóa bằng cách copy và bật mã hóa khi copy.

---

## Slide 13 — AMI Overview (Tổng quan về AMI)

**Nội dung cần trình bày:**

**AMI = Amazon Machine Image** — hiểu đơn giản là một **"bản sao hệ thống"** của EC2 instance.

AMI bao gồm: hệ điều hành, phần mềm đã cài, cấu hình, và các thiết lập monitoring. Điều này giúp bạn **khởi động instance mới nhanh hơn** vì mọi thứ đã được đóng gói sẵn.

**AMI được xây dựng cho một Region cụ thể** nhưng có thể copy sang Region khác.

Có ba loại AMI:
1. **Public AMI**: AWS cung cấp sẵn (Amazon Linux, Ubuntu...).
2. **Your own AMI**: Bạn tự tạo và quản lý.
3. **AWS Marketplace AMI**: AMI của bên thứ ba (có thể mất phí).

---

## Slide 14 — AMI Process (Quy trình tạo AMI từ EC2)

**Nội dung cần trình bày:**

Quy trình tạo AMI từ một EC2 instance gồm **4 bước**:

1. **Khởi động EC2 instance** và cài đặt, cấu hình theo nhu cầu.
2. **Dừng instance** (stop) để đảm bảo tính toàn vẹn dữ liệu — không nên tạo AMI khi instance đang chạy.
3. **Build AMI** từ instance đó — quá trình này cũng tự động tạo EBS Snapshots.
4. **Launch instance mới** từ AMI vừa tạo — có thể ở bất kỳ AZ nào trong cùng Region.

Đây là cách phổ biến để **nhân bản môi trường** hoặc **chuẩn bị golden image** cho Auto Scaling Groups.

---

## Slide 15 — EC2 Instance Store

**Nội dung cần trình bày:**

**EC2 Instance Store** là bộ nhớ **gắn trực tiếp vào phần cứng** của server vật lý — không phải qua mạng như EBS.

Ưu điểm:
- **Hiệu năng I/O rất cao** — nhanh hơn EBS đáng kể.

Nhược điểm:
- **Dữ liệu bị mất** khi instance bị stop hoặc terminate (**ephemeral storage**).
- Nếu phần cứng server gặp sự cố, dữ liệu mất hoàn toàn.
- **Bạn tự chịu trách nhiệm** backup và replication.

**Use case phù hợp**: Buffer, cache, scratch data, temporary files — những dữ liệu không cần tồn tại lâu dài.

> Nhớ nguyên tắc: **Cần tốc độ cao + dữ liệu tạm thời → Instance Store. Cần dữ liệu bền vững → EBS.**

---

## Slide 16 — Local EC2 Instance Store – IOPS

**Nội dung cần trình bày:**

Slide này minh họa con số IOPS cực kỳ cao mà EC2 Instance Store có thể đạt được so với EBS thông thường.

Một số instance type đặc biệt (như `i3`, `i4i`) có thể đạt hàng **triệu IOPS** — phù hợp cho các workload cực kỳ nhạy cảm với tốc độ I/O như NoSQL databases (Cassandra, Redis) khi cần throughput tối đa.

---

## Slide 17 — Amazon EFS – Elastic File System (Giới thiệu)

**Nội dung cần trình bày:**

**Amazon EFS** là một **hệ thống file mạng (NFS)** được quản lý hoàn toàn bởi AWS.

Điểm khác biệt lớn so với EBS:
- EFS có thể **gắn đồng thời vào nhiều EC2 instances** trên **nhiều Availability Zones** khác nhau.
- **Highly available và scalable** — tự động mở rộng khi cần.
- **Tốn kém hơn**: giá khoảng **3 lần gp2**, nhưng bạn chỉ trả tiền cho dung lượng thực sự dùng (**pay per use**).

Truy cập được kiểm soát qua **Security Group**.

---

## Slide 18 — Amazon EFS – Chi tiết và Use Cases

**Nội dung cần trình bày:**

**Các trường hợp sử dụng EFS điển hình**:
- Quản lý nội dung (Content Management).
- Web Serving.
- Chia sẻ dữ liệu giữa nhiều server.
- **WordPress** — một use case rất phổ biến trên AWS.

**Chi tiết kỹ thuật**:
- Dùng giao thức **NFSv4.1**.
- Truy cập được kiểm soát bởi **Security Group**.
- **Chỉ tương thích với Linux AMI** (POSIX file system) — **không hỗ trợ Windows**.

---

## Slide 19 — EBS vs EFS – So sánh

**Nội dung cần trình bày:**

Đây là phần so sánh quan trọng giữa hai loại storage:

| Tiêu chí | EBS | EFS |
|----------|-----|-----|
| Gắn vào bao nhiêu instance? | 1 instance (trừ io1/io2 Multi-Attach) | Hàng trăm instances cùng lúc |
| Phạm vi | Trong 1 AZ | Đa AZ |
| OS | Mọi OS | **Chỉ Linux (POSIX)** |
| Chi phí | Thấp hơn | Cao hơn |

**Use case điển hình của EFS**: Chia sẻ file website WordPress giữa nhiều web server chạy ở nhiều AZ khác nhau.

**Tóm lại**: Cần chia sẻ dữ liệu đa-AZ → EFS. Cần lưu trữ riêng cho một instance → EBS.

---

## Slide 20 — Quiz Time!

**Nội dung cần trình bày:**

Đây là slide chuyển tiếp sang phần câu hỏi kiểm tra kiến thức về EC2 Storage.

Hãy dành vài phút ôn lại các điểm chính:
- **EBS**: Network drive, 1 AZ, 1 instance (ngoại trừ Multi-Attach).
- **Snapshot**: Backup, copy được sang AZ/Region khác.
- **Instance Store**: Hiệu năng cao, dữ liệu tạm thời.
- **EFS**: Chia sẻ đa-AZ, chỉ Linux.

---

## Slide 21 — Tiêu đề: Containers on AWS

**Nội dung cần trình bày:**

Chúng ta chuyển sang phần thứ hai của bài: **Containers trên AWS**.

Container đang là công nghệ triển khai ứng dụng phổ biến nhất hiện nay. AWS cung cấp nhiều dịch vụ để chạy và quản lý container — từ việc tự quản lý hạ tầng đến hoàn toàn serverless. Hãy cùng tìm hiểu!

---

## Slide 22 — Docker là gì?

**Nội dung cần trình bày:**

**Docker** là nền tảng phát triển phần mềm dùng để **đóng gói và triển khai ứng dụng dưới dạng container**.

Ưu điểm của Docker:
- Ứng dụng **chạy giống nhau trên mọi môi trường** — không còn lỗi "chạy được trên máy tôi nhưng không chạy được trên server".
- **Không có vấn đề tương thích** giữa các môi trường.
- **Dễ bảo trì và triển khai**.
- Hoạt động với **mọi ngôn ngữ, OS, và công nghệ**.

**Use case trên AWS**:
- Kiến trúc **microservices**.
- Di chuyển ứng dụng on-premises lên AWS (**lift-and-shift**).

---

## Slide 23 — Docker vs. Virtual Machines

**Nội dung cần trình bày:**

Docker thường được so sánh với Virtual Machine (VM), nhưng chúng hoạt động rất khác nhau.

**Virtual Machine (VM)**:
- Mỗi VM có **Guest OS riêng** → tốn nhiều tài nguyên hơn.
- Khởi động chậm hơn.

**Docker Container**:
- **Chia sẻ Host OS** (không cần Guest OS riêng).
- Nhẹ hơn, khởi động nhanh hơn nhiều.
- Nhiều container có thể chạy song song trên cùng một server.

**Kết luận**: Container hiệu quả hơn về tài nguyên so với VM, nhưng VM cung cấp sự cô lập tốt hơn.

---

## Slide 24 — Bắt đầu với Docker

**Nội dung cần trình bày:**

Quy trình làm việc cơ bản với Docker:

1. **Viết Dockerfile** — file cấu hình mô tả cách build image.
2. **Build** → tạo ra **Docker Image**.
3. **Push** image lên **Docker Repository** (Docker Hub hoặc Amazon ECR).
4. **Pull** image xuống khi cần triển khai.
5. **Run** image → tạo ra **Container** đang chạy.

**Amazon ECR** (Elastic Container Registry) là nơi AWS lưu trữ Docker images của bạn — tương tự như Docker Hub nhưng private và tích hợp sẵn với các dịch vụ AWS.

---

## Slide 25 — Docker trên một OS

**Nội dung cần trình bày:**

Slide này minh họa cách nhiều Docker containers cùng chạy trên một server (ví dụ: EC2 instance).

Mỗi container là một tiến trình độc lập, chia sẻ kernel của Host OS nhưng được cô lập về process, network, và file system. Đây là lý do Docker nhẹ và nhanh hơn VM.

---

## Slide 26 — Docker Images được lưu ở đâu?

**Nội dung cần trình bày:**

Docker images được lưu trữ trong **Docker Repository**. AWS có hai tùy chọn:

**Docker Hub** (https://hub.docker.com):
- **Kho công khai** lớn nhất thế giới.
- Chứa các base image phổ biến: Ubuntu, MySQL, Node.js, Python...

**Amazon ECR** (Elastic Container Registry):
- **Private repository** — chỉ bạn và những người được cấp quyền mới truy cập được.
- Cũng có **Public Gallery** tại https://gallery.ecr.aws.
- Tích hợp sẵn với ECS, EKS, và các dịch vụ AWS khác.

---

## Slide 27 — Quản lý Docker Containers trên AWS

**Nội dung cần trình bày:**

AWS cung cấp **bốn dịch vụ chính** để quản lý containers:

| Dịch vụ | Mô tả |
|---------|-------|
| **Amazon ECS** | Nền tảng container riêng của AWS |
| **Amazon EKS** | Kubernetes được quản lý bởi AWS (mã nguồn mở) |
| **AWS Fargate** | Serverless container — không cần quản lý server |
| **Amazon ECR** | Lưu trữ Docker images |

**Fargate** có thể kết hợp với cả ECS lẫn EKS, giúp bạn chạy containers mà không cần lo về hạ tầng bên dưới.

---

## Slide 28 — Amazon ECS – EC2 Launch Type

**Nội dung cần trình bày:**

**ECS (Elastic Container Service)** là dịch vụ chạy containers của AWS.

Với **EC2 Launch Type**:
- Bạn **phải tự tạo và quản lý** các EC2 instances trong ECS Cluster.
- Mỗi EC2 instance cần cài **ECS Agent** để đăng ký vào Cluster.
- AWS sẽ tự động **start/stop containers** trên các instance đó.

**Tóm lại**: Bạn kiểm soát hạ tầng (EC2), AWS kiểm soát việc lập lịch chạy containers.

**Khi nào dùng**: Khi bạn cần kiểm soát chi tiết về loại instance, cấu hình phần cứng, hoặc muốn tối ưu chi phí với Reserved Instances.

---

## Slide 29 — Amazon ECS – Fargate Launch Type

**Nội dung cần trình bày:**

Với **Fargate Launch Type**, bạn **không cần quản lý bất kỳ EC2 instance nào** — hoàn toàn **Serverless**!

Cách hoạt động:
- Bạn chỉ cần định nghĩa **Task Definition** (bao nhiêu CPU, RAM cần).
- AWS tự tìm hạ tầng phù hợp và chạy container cho bạn.
- Muốn scale up? Chỉ cần **tăng số lượng Tasks** — không cần lo thêm server.

**Ưu điểm**: Đơn giản, không cần quản lý hạ tầng, tự động scale.

**Nhược điểm**: Ít kiểm soát hơn và thường tốn kém hơn EC2 Launch Type ở quy mô lớn.

---

## Slide 30 — Amazon ECS – IAM Roles

**Nội dung cần trình bày:**

Bảo mật trong ECS được thực hiện qua **IAM Roles** — có hai loại:

**EC2 Instance Profile** (chỉ áp dụng cho EC2 Launch Type):
- Được dùng bởi **ECS Agent**.
- Cho phép ECS Agent gọi API, gửi log lên CloudWatch, và pull image từ ECR.

**ECS Task Role**:
- Mỗi Task (container) có **role riêng** với quyền khác nhau.
- Ví dụ: Task A cần truy cập S3, Task B cần truy cập DynamoDB — mỗi Task nhận một role phù hợp.
- Task Role được định nghĩa trong **Task Definition**.

**Nguyên tắc Least Privilege**: Chỉ cấp quyền tối thiểu cần thiết cho mỗi Task.

---

## Slide 31 — Amazon ECS – Load Balancer Integration

**Nội dung cần trình bày:**

ECS hỗ trợ tích hợp với ba loại Load Balancer:

| Load Balancer | Khuyến nghị | Ghi chú |
|---------------|-------------|---------|
| **Application Load Balancer (ALB)** | ✅ Khuyến nghị | Phù hợp cho hầu hết use case |
| **Network Load Balancer (NLB)** | ⚡ Cho throughput cao | Dùng khi cần hiệu năng cực cao hoặc AWS Private Link |
| **Classic Load Balancer** | ❌ Không khuyến nghị | Không có tính năng nâng cao, không hỗ trợ Fargate |

**Cách hoạt động**: User gửi request đến Load Balancer → Load Balancer phân phối đến các ECS Tasks đang chạy trên EC2 instances.

---

## Slide 32 — Amazon ECR (Elastic Container Registry)

**Nội dung cần trình bày:**

**Amazon ECR** là dịch vụ lưu trữ và quản lý Docker images trên AWS.

Các tính năng chính:
- Hỗ trợ cả **Private** lẫn **Public** repository.
- **Tích hợp sẵn với ECS** — ECS có thể pull image trực tiếp từ ECR.
- Dữ liệu được lưu trên **Amazon S3** phía sau.
- Truy cập được kiểm soát qua **IAM** — nếu bị lỗi permission, hãy kiểm tra IAM policy.
- Hỗ trợ **image vulnerability scanning**, versioning, image tags, và lifecycle policies.

**Lưu ý bảo mật**: Luôn dùng **ECR Private** cho production images để tránh lộ code và cấu hình.

---

## Slide 33 — Amazon EKS Overview

**Nội dung cần trình bày:**

**Amazon EKS = Elastic Kubernetes Service** — dịch vụ chạy **Kubernetes được quản lý** trên AWS.

**Kubernetes** là hệ thống mã nguồn mở dùng để tự động hóa việc triển khai, scale, và quản lý containers.

**Khi nào nên dùng EKS thay vì ECS?**
- Công ty bạn **đang dùng Kubernetes on-premises** và muốn chuyển lên AWS.
- Muốn **cloud-agnostic** — Kubernetes chạy được trên AWS, Azure, GCP.
- Cần tích hợp với hệ sinh thái Kubernetes rộng lớn.

**Lưu ý triển khai**:
- Hỗ trợ cả **EC2 worker nodes** lẫn **Fargate** (serverless).
- Mỗi Region nên triển khai **một EKS cluster riêng**.
- Thu thập logs và metrics qua **CloudWatch Container Insights**.

---

## Slide 34 — AWS App Runner

**Nội dung cần trình bày:**

**AWS App Runner** là dịch vụ **fully managed** giúp triển khai web apps và APIs cực kỳ đơn giản — **không cần kiến thức về hạ tầng**.

Cách hoạt động:
1. Cung cấp **source code** hoặc **container image**.
2. Cấu hình: vCPU, RAM, Auto Scaling, Health Check.
3. App Runner tự **build và deploy**.
4. Truy cập ngay qua **URL**.

Tính năng tích hợp sẵn:
- Auto scaling, Load Balancer, encryption.
- Hỗ trợ VPC access để kết nối database, cache, message queue.

**Use case**: Web apps, APIs, microservices — đặc biệt khi cần **deploy nhanh** mà không muốn cấu hình phức tạp.

---

## Slide 35 — AWS App2Container (A2C)

**Nội dung cần trình bày:**

**AWS App2Container (A2C)** là công cụ CLI dùng để **containerize** ứng dụng **Java** và **.NET** đang chạy trên hạ tầng cũ.

Cho phép bạn **lift-and-shift** từ:
- On-premises bare metal
- Virtual machines
- Bất kỳ Cloud provider nào

Sang AWS — **không cần thay đổi code**.

Quy trình tự động hóa:
1. Phân tích ứng dụng hiện tại.
2. Tạo **Docker container** từ ứng dụng đó.
3. Đăng ký image lên **ECR**.
4. Tạo **CloudFormation templates** cho compute và network.
5. Deploy lên **ECS, EKS, hoặc App Runner**.

**A2C giúp hiện đại hóa legacy apps một cách nhanh chóng**, không cần refactor toàn bộ.

---

## Tổng kết bài thuyết trình

### Phần 1: EC2 Instance Storage

| Dịch vụ | Đặc điểm chính |
|---------|----------------|
| **EBS** | Network drive, 1 AZ, bền vững, nhiều loại (gp/io/st/sc) |
| **EBS Snapshot** | Backup, copy AZ/Region, Archive/Recycle Bin/FSR |
| **AMI** | Bản sao hệ thống EC2, khởi động nhanh |
| **EC2 Instance Store** | Hiệu năng cao nhất, dữ liệu tạm thời |
| **EFS** | NFS, đa-AZ, nhiều instance, chỉ Linux |

### Phần 2: Containers on AWS

| Dịch vụ | Đặc điểm chính |
|---------|----------------|
| **Docker** | Nền tảng container, nhất quán mọi môi trường |
| **Amazon ECS** | Container platform của AWS (EC2 hoặc Fargate) |
| **Amazon EKS** | Kubernetes được quản lý trên AWS |
| **AWS Fargate** | Serverless containers, không quản lý EC2 |
| **Amazon ECR** | Lưu trữ Docker images (private/public) |
| **AWS App Runner** | Deploy web app cực đơn giản, fully managed |
| **AWS App2Container** | Containerize ứng dụng Java/.NET legacy |

---

*Tài liệu biên soạn từ slide deck của Stephane Maarek — datacumulus.com*