# AWS Database Services — Bài Thuyết Trình Chi Tiết

---

## Slide 1: Trang Bìa — AWS Database Services

Xin chào mọi người! Hôm nay chúng ta sẽ cùng tìm hiểu về **AWS Database Services** — hệ sinh thái cơ sở dữ liệu toàn diện của Amazon Web Services.

AWS cung cấp một danh mục phong phú các dịch vụ cơ sở dữ liệu được quản lý hoàn toàn (fully managed), giúp doanh nghiệp không còn phải lo lắng về việc cài đặt, vá lỗi, sao lưu hay quản lý hạ tầng. Thay vào đó, đội ngũ kỹ thuật có thể tập trung hoàn toàn vào việc xây dựng sản phẩm và tạo ra giá trị kinh doanh.

Trong buổi hôm nay, chúng ta sẽ đi qua toàn bộ hệ sinh thái này — từ cơ sở dữ liệu quan hệ truyền thống cho đến NoSQL, in-memory, data warehouse, cho đến các cơ sở dữ liệu chuyên biệt như graph database.

---

## Slide 2: Database Selection Framework — Khung Chọn Lựa Cơ Sở Dữ Liệu

Trước khi đi vào từng dịch vụ cụ thể, điều quan trọng nhất là **chọn đúng công cụ cho đúng công việc**. AWS cung cấp rất nhiều loại cơ sở dữ liệu, và việc lựa chọn sai có thể dẫn đến hiệu năng kém, chi phí cao hoặc khó mở rộng sau này.

Khung lựa chọn gồm 6 tiêu chí cốt lõi:

**1. Workload Type (Loại khối lượng công việc)**
Đây là câu hỏi đầu tiên: ứng dụng của bạn thiên về **OLTP** (Online Transaction Processing — xử lý giao dịch thời gian thực như mua hàng, đặt vé), **OLAP** (Online Analytical Processing — phân tích dữ liệu lớn, báo cáo kinh doanh), hay kết hợp cả hai? OLTP cần độ trễ thấp và tính nhất quán cao. OLAP cần khả năng xử lý truy vấn phức tạp trên lượng dữ liệu khổng lồ.

**2. Data Structure (Cấu trúc dữ liệu)**
Dữ liệu của bạn có cấu trúc như thế nào? Nếu dữ liệu có quan hệ rõ ràng giữa các bảng thì dùng **Relational**. Nếu dữ liệu dạng JSON linh hoạt thì dùng **Document**. Nếu cần tra cứu nhanh bằng key thì dùng **Key-Value**. Nếu dữ liệu có nhiều mối quan hệ phức tạp như mạng xã hội thì dùng **Graph**.

**3. Scale Requirements (Yêu cầu mở rộng)**
Dự đoán dung lượng Read/Write throughput và nhu cầu lưu trữ. Hệ thống của bạn cần xử lý bao nhiêu request mỗi giây? Dữ liệu sẽ tăng trưởng đến bao nhiêu terabyte?

**4. Performance (Hiệu năng)**
Độ trễ chấp nhận được là bao nhiêu? Millisecond hay microsecond? Mức độ nhất quán dữ liệu cần như thế nào — strong consistency hay eventual consistency?

**5. Availability (Tính sẵn sàng)**
Xác định **RPO** (Recovery Point Objective — tối đa bao nhiêu dữ liệu có thể mất nếu xảy ra sự cố) và **RTO** (Recovery Time Objective — tối đa bao lâu hệ thống được phép gián đoạn). Đây là nền tảng để thiết kế High Availability.

**6. Cost (Chi phí)**
Đánh giá **TCO** (Total Cost of Ownership) bao gồm chi phí vận hành, nhân lực, licensing, và so sánh với chi phí AWS managed service. Managed service thường đắt hơn on-demand EC2, nhưng rẻ hơn rất nhiều khi tính cả chi phí vận hành và nhân lực.

---

## Slide 3: Database Types Overview — Tổng Quan Các Loại Cơ Sở Dữ Liệu

AWS chia hệ sinh thái database thành 6 nhóm chính, mỗi nhóm được tối ưu cho một bài toán khác nhau:

| Loại | Dịch vụ AWS | Phù hợp nhất cho |
|------|-------------|-----------------|
| **Relational (SQL)** | RDS, Aurora | Ứng dụng truyền thống, giao dịch tài chính |
| **Key-Value** | DynamoDB | Web app lưu lượng cao, gaming, IoT |
| **In-Memory** | ElastiCache | Caching, session store, analytics thời gian thực |
| **Data Warehouse** | Redshift | Business intelligence, phân tích dữ liệu lớn |
| **Document** | DocumentDB | Quản lý nội dung, danh mục sản phẩm, hồ sơ người dùng |
| **Graph** | Neptune | Mạng xã hội, phát hiện gian lận, knowledge graph |

**Triết lý cốt lõi của AWS:** "Use the right tool for the right job." Không có một loại database nào là tốt nhất cho mọi tình huống. Một ứng dụng phức tạp thường sử dụng nhiều loại database cùng lúc — ví dụ RDS cho giao dịch, ElastiCache cho caching, và Redshift cho báo cáo.

---

## Slide 4: Amazon RDS — Tiêu Đề Phần

Bây giờ chúng ta bước vào dịch vụ đầu tiên và phổ biến nhất: **Amazon RDS (Relational Database Service)**.

---

## Slide 5: Amazon RDS Overview — Tổng Quan

**Amazon RDS** là dịch vụ cơ sở dữ liệu quan hệ được quản lý hoàn toàn, giúp tự động hóa các tác vụ quản trị tốn thời gian nhất.

**Những gì AWS quản lý thay bạn:**
- Vá lỗi hệ điều hành và cập nhật phần mềm database
- Cài đặt và cấu hình database engine
- Tự động sao lưu theo lịch
- Phát hiện lỗi và khởi động lại tự động
- Cung cấp và quản lý hạ tầng vật lý

**Lợi ích chính:**
- **Automated backups:** Sao lưu tự động hàng ngày, có thể khôi phục về bất kỳ thời điểm nào trong vòng 35 ngày (Point-in-Time Recovery).
- **Multi-AZ deployments:** Triển khai đồng bộ sang Availability Zone dự phòng, đảm bảo High Availability. Khi AZ chính gặp sự cố, hệ thống tự động chuyển sang AZ dự phòng trong 1-2 phút.
- **Read replicas:** Tạo bản sao chỉ đọc để phân tán tải đọc, giảm áp lực cho database chính.
- **Monitoring & metrics:** Tích hợp sẵn với Amazon CloudWatch để theo dõi hiệu năng và cài đặt cảnh báo.

**Tại sao dùng RDS thay vì tự cài trên EC2?** Khi tự cài trên EC2, bạn phải tự quản lý toàn bộ — từ OS, database software, backup, đến failover. RDS loại bỏ hoàn toàn gánh nặng này, cho phép đội ngũ tập trung vào phát triển ứng dụng.

---

## Slide 6: RDS Engine Options — Các Engine Được Hỗ Trợ

RDS hỗ trợ 6 database engine phổ biến nhất:

**MySQL**
Engine open-source phổ biến nhất thế giới. Hỗ trợ phiên bản 5.7 và 8.0. Lý tưởng cho web application, CMS (WordPress, Drupal). Cộng đồng lớn, tài liệu phong phú.

**PostgreSQL**
Database open-source tiên tiến với nhiều tính năng mạnh mẽ: hỗ trợ JSON, full-text search, extensions đa dạng, và các kiểu dữ liệu phức tạp. Hỗ trợ từ phiên bản 12 đến 15. Phù hợp với ứng dụng cần tính năng advanced như GIS, financial modeling.

**MariaDB**
Fork của MySQL với nhiều cải tiến về hiệu năng và bảo mật. Tương thích ngược với MySQL. Hỗ trợ phiên bản 10.5 và 10.6.

**Oracle**
Database thương mại enterprise-grade. AWS hỗ trợ hai mô hình licensing: **BYOL** (Bring Your Own License — mang license có sẵn của bạn) hoặc **License Included** (AWS tính phí license trong chi phí instance). Phù hợp cho doanh nghiệp đã đầu tư vào hệ sinh thái Oracle.

**SQL Server**
Database của Microsoft với 4 phiên bản: Express (miễn phí, giới hạn), Web, Standard, và Enterprise. Lý tưởng cho hệ thống .NET, SharePoint, hay các ứng dụng Microsoft.

**Db2**
Database enterprise của IBM, có từ phiên bản Standard đến Advanced. Phù hợp cho doanh nghiệp đang dùng hệ sinh thái IBM.

---

## Slide 7: RDS Features and Benefits — Tính Năng Chi Tiết

**Multi-AZ Deployments (Triển khai đa Availability Zone)**
Đây là tính năng High Availability cốt lõi của RDS. Khi bật Multi-AZ, AWS tạo một bản sao **đồng bộ** (synchronous) của database tại một AZ khác trong cùng region. Nếu AZ chính gặp sự cố — dù là lỗi phần cứng, mất điện, hay cần maintenance — AWS tự động chuyển traffic sang bản sao dự phòng trong vòng **1-2 phút**, không cần can thiệp thủ công. Lưu ý: bản sao Multi-AZ không dùng để phục vụ read traffic, chỉ dùng cho failover.

**Read Replicas (Bản sao chỉ đọc)**
Khác với Multi-AZ, Read Replicas được dùng để **mở rộng khả năng đọc**. Dữ liệu được sao chép **bất đồng bộ** (asynchronous) từ primary database sang các replicas. Có thể tạo tối đa 5 replicas cho MySQL/PostgreSQL. Ứng dụng đọc nhiều (báo cáo, tìm kiếm, analytics) có thể trỏ trực tiếp đến read replica thay vì tốn tài nguyên primary. Read replica còn có thể được **promote** thành standalone database độc lập khi cần.

**Automated Backups (Sao lưu tự động)**
RDS tự động tạo snapshot của database mỗi ngày trong cửa sổ bảo trì (maintenance window) bạn chỉ định. Cùng với đó, transaction logs được lưu liên tục mỗi 5 phút. Kết hợp lại, bạn có thể khôi phục database về **bất kỳ thời điểm nào** trong vòng 1-35 ngày (cấu hình được). Đây gọi là **Point-in-Time Recovery** — rất hữu ích khi xảy ra lỗi do người dùng như xóa nhầm dữ liệu.

**Encryption (Mã hóa)**
RDS hỗ trợ mã hóa toàn diện:
- **At-rest:** Dữ liệu lưu trên đĩa được mã hóa bằng AWS KMS (Key Management Service).
- **In-transit:** Kết nối giữa ứng dụng và database được mã hóa bằng SSL/TLS.
- Với Oracle và SQL Server còn có **Transparent Data Encryption (TDE)** — mã hóa ở tầng database engine.

---

## Slide 8: RDS Use Cases — Trường Hợp Sử Dụng

**Web và Mobile Applications**
RDS phù hợp hoàn hảo cho các nền tảng thương mại điện tử, hệ thống quản lý nội dung (CMS), và dịch vụ xác thực người dùng. Đây là những ứng dụng cần đảm bảo tính toàn vẹn dữ liệu (ACID transactions) và hỗ trợ SQL query linh hoạt.

**Enterprise Applications**
Các hệ thống ERP (Enterprise Resource Planning), CRM (Customer Relationship Management), quản lý kho hàng, và hệ thống tài chính đều là ứng dụng điển hình. Đây là workload có cấu trúc dữ liệu phức tạp, nhiều quan hệ giữa các bảng, và yêu cầu tính nhất quán cao.

**SaaS Applications**
Các ứng dụng SaaS phục vụ nhiều khách hàng (multi-tenant) có thể dùng RDS với cấu hình cô lập dữ liệu (data isolation) và tuân thủ các tiêu chuẩn bảo mật (compliance).

**Ví dụ kiến trúc thực tế — Nền tảng thương mại điện tử:**
- **Multi-AZ RDS MySQL** cho dữ liệu giao dịch (đơn hàng, thanh toán) — đảm bảo không mất dữ liệu
- **Read replicas** để phục vụ truy vấn danh mục sản phẩm — giảm tải cho primary
- **ElastiCache** để quản lý session người dùng — tốc độ nhanh, không tốn tài nguyên DB
- **Automated backups** trước mỗi lần deploy lớn — an toàn khi có sự cố

---

## Slide 9: Amazon Aurora — Tiêu Đề Phần

Tiếp theo là **Amazon Aurora** — thế hệ database tiếp theo được AWS xây dựng từ đầu cho môi trường cloud.

---

## Slide 10: Amazon Aurora Overview — Tổng Quan

**Amazon Aurora** là database engine được AWS tự phát triển, tương thích hoàn toàn với MySQL và PostgreSQL, nhưng được thiết kế lại từ đầu để tận dụng tối đa kiến trúc cloud.

**Những con số ấn tượng:**
- Hiệu năng gấp **5 lần** MySQL tiêu chuẩn
- Hiệu năng gấp **3 lần** PostgreSQL tiêu chuẩn
- Storage tự động mở rộng lên tới **128 TB**
- Hỗ trợ tới **15 read replicas** với độ trễ dưới 10ms
- Backup liên tục sang S3
- **6-way replication** trên 3 AZs — đây là điểm khác biệt cốt lõi so với RDS

**Hai engine tương thích:**
- **Aurora MySQL:** Tương thích với MySQL 5.7 và 8.0. Migrate từ RDS MySQL sang Aurora MySQL gần như không cần thay đổi code.
- **Aurora PostgreSQL:** Tương thích với PostgreSQL 12, 13, 14, 15. Tương tự, việc chuyển đổi từ RDS PostgreSQL rất đơn giản.

**Fast Database Cloning:** Aurora cho phép tạo bản clone của toàn bộ database trong vài giây mà không cần copy dữ liệu — rất hữu ích cho môi trường dev/test.

**Global Database:** Aurora có thể trải dài trên nhiều AWS Regions, cho phép đọc với độ trễ thấp ở bất kỳ nơi nào trên thế giới.

---

## Slide 11: Aurora Performance and Architecture — Kiến Trúc và Hiệu Năng

**Storage Architecture (Kiến trúc lưu trữ)**
Đây là điểm khác biệt lớn nhất của Aurora. Thay vì lưu trữ theo kiểu truyền thống, Aurora sử dụng kiến trúc lưu trữ **phân tán, tự phục hồi** (distributed, self-healing). Dữ liệu được tự động nhân bản 6 bản sao trên 3 AZs (2 bản/AZ). Storage tự động tăng theo từng bước 10GB, lên đến 128TB — bạn không cần dự phòng dung lượng trước.

**High Availability (Tính sẵn sàng cao)**
Aurora chịu được:
- Mất **2 bản sao** mà vẫn đảm bảo **write** hoạt động bình thường
- Mất **3 bản sao** mà vẫn đảm bảo **read** hoạt động bình thường
- Failover tự động trong dưới **30 giây** — nhanh hơn gấp đôi so với RDS Multi-AZ (1-2 phút)

**Performance Features (Tính năng hiệu năng)**
Aurora sử dụng Parallel Query Processing để phân tán việc xử lý truy vấn phức tạp xuống tầng storage. Điều này cho phép analytics query chạy nhanh hơn mà không ảnh hưởng đến OLTP workload.

**Aurora Serverless**
Phiên bản đặc biệt: tự động scale capacity lên xuống dựa trên nhu cầu thực tế, tính phí theo giây. Lý tưởng cho workload không đều (variable workloads) và môi trường dev/test — không phải trả tiền khi database không có traffic.

**Global Database**
Cho phép một database Aurora trải rộng trên nhiều Regions. Write được thực hiện ở Region chính (primary), và replicated sang các Region phụ (secondary) với độ trễ dưới 1 giây. Trong tình huống disaster recovery, secondary Region có thể được promote thành primary với **RPO (Recovery Point Objective) chỉ 1 giây**.

---

## Slide 12: Aurora Use Cases — Trường Hợp Sử Dụng

**Enterprise Applications (Ứng dụng doanh nghiệp)**
Các ứng dụng mission-critical đòi hỏi high availability, hiệu năng cao, và khả năng mở rộng lớn. Ví dụ điển hình: **nền tảng giao dịch tài chính** yêu cầu độ trễ millisecond, không được phép downtime, và phải xử lý hàng nghìn transaction mỗi giây.

**SaaS Platforms**
Nền tảng SaaS phục vụ khách hàng toàn cầu với workload không đều. Aurora Serverless giúp scale tự động khi có spike traffic, Global Database đảm bảo độ trễ thấp ở mọi khu vực. Ví dụ: **Nền tảng CRM** phục vụ khách hàng ở nhiều châu lục.

**Gaming (Trò chơi trực tuyến)**
Game online cần database có thể xử lý hàng triệu người dùng đồng thời, với dữ liệu về leaderboard, player state, và matchmaking. Ví dụ: **Game multiplayer** với hàng triệu concurrent users cần database vừa nhanh, vừa có thể scale theo giờ cao điểm.

---

## Slide 13: RDS vs Aurora Comparison — So Sánh Chi Tiết

| Tính năng | Amazon RDS | Amazon Aurora |
|-----------|-----------|--------------|
| **Hiệu năng** | Hiệu năng database tiêu chuẩn | Gấp 5x MySQL, 3x PostgreSQL |
| **Storage** | Tối đa 64 TB (tùy engine) | Tối đa 128 TB, tự động mở rộng |
| **Read Replicas** | Tối đa 5 (MySQL/PostgreSQL) | Tối đa 15 với độ trễ thấp hơn |
| **Replication** | Bất đồng bộ sang replicas | 6-way trên 3 AZs |
| **Failover Time** | 1-2 phút | Dưới 30 giây |
| **Chi phí** | Thấp hơn, điểm khởi đầu tốt | Đắt hơn ~20%, hiệu quả hơn ở quy mô lớn |

**Khi nào chọn RDS?**
- Workload nhỏ đến trung bình
- Budget hạn chế
- Không cần hiệu năng đặc biệt cao
- Đang dùng Oracle hoặc SQL Server (Aurora không hỗ trợ)

**Khi nào chọn Aurora?**
- Ứng dụng mission-critical cần uptime cao nhất
- Cần scale lớn và hiệu năng cao
- Muốn Global Database
- Workload không đều (dùng Aurora Serverless)

---

## Slide 14: Amazon DynamoDB — Tiêu Đề Phần

Bước sang thế giới NoSQL với **Amazon DynamoDB** — database serverless hiệu năng cao nhất của AWS.

---

## Slide 15: Amazon DynamoDB Overview — Tổng Quan

**Amazon DynamoDB** là database NoSQL được quản lý hoàn toàn, serverless, cung cấp hiệu năng **single-digit millisecond** (độ trễ dưới 10ms) ở bất kỳ quy mô nào.

**Đặc điểm cốt lõi:**
- **Serverless với auto-scaling:** Không cần provision capacity, DynamoDB tự động điều chỉnh theo nhu cầu
- **Single-digit millisecond latency:** Độ trễ nhất quán dù bảng có 1 item hay hàng tỷ items
- **Virtually unlimited throughput:** Không có giới hạn thực tế về khả năng xử lý
- **Built-in security and backup:** Mã hóa mặc định, sao lưu liên tục
- **Global Tables:** Database đa vùng với cấu hình active-active
- **ACID Transactions:** Hỗ trợ giao dịch đảm bảo tính nhất quán

**Những con số ấn tượng ở quy mô thực tế:**
- Xử lý **10 nghìn tỷ request mỗi ngày**
- **20 triệu request mỗi giây** ở thời điểm peak

Đây là những con số từ các hệ thống thực của Amazon — Prime Day, Black Friday. DynamoDB là xương sống của nhiều dịch vụ lớn nhất của Amazon.

---

## Slide 16: DynamoDB Core Concepts — Khái Niệm Cốt Lõi

**Tables and Items (Bảng và mục)**
Không giống SQL database, DynamoDB là **schema-less** — mỗi item trong bảng có thể có các attributes (thuộc tính) khác nhau. Không cần định nghĩa schema trước. Điều này cho phép linh hoạt tuyệt đối trong việc lưu trữ dữ liệu đa dạng.

**Primary Keys (Khóa chính)**
Mỗi item được xác định duy nhất bởi Primary Key, có 2 dạng:
- **Partition Key** (Simple Primary Key): Chỉ một thuộc tính, dùng làm key để phân phối dữ liệu
- **Composite Key** (Partition Key + Sort Key): Kết hợp hai thuộc tính, cho phép nhóm items có cùng partition key và query theo range

**Indexes (Chỉ mục)**
- **GSI (Global Secondary Index):** Index với Partition Key khác, cho phép query theo nhiều chiều khác nhau mà không cần scan toàn bộ bảng
- **LSI (Local Secondary Index):** Cùng Partition Key nhưng Sort Key khác, cho phép sắp xếp theo nhiều tiêu chí

**Capacity Modes (Chế độ dung lượng)**
- **On-demand:** Tự động scale, trả phí theo từng request. Phù hợp cho workload không thể dự đoán
- **Provisioned (RCU/WCU):** Đặt trước Read/Write Capacity Units. Tiết kiệm hơn khi workload ổn định và có thể dự đoán

**Streams (Luồng dữ liệu)**
DynamoDB Streams ghi lại mọi thay đổi ở cấp độ item (item-level changes) theo thời gian gần thực. Dùng để kích hoạt Lambda function, replicate sang database khác, hoặc build analytics pipeline.

---

## Slide 17: DynamoDB Features — Tính Năng Nâng Cao

**Global Tables (Bảng toàn cầu)**
Tạo bảng DynamoDB được nhân bản trên nhiều AWS Regions với cấu hình **active-active** — nghĩa là write có thể thực hiện ở bất kỳ Region nào, và dữ liệu được đồng bộ sub-second (dưới 1 giây) sang các Region khác. Lý tưởng cho ứng dụng toàn cầu cần độ trễ thấp ở mọi nơi.

**Point-in-Time Recovery (PITR)**
Sao lưu liên tục cho phép khôi phục database về bất kỳ thời điểm nào trong **35 ngày qua**, với độ chính xác từng giây. Không ảnh hưởng đến hiệu năng của ứng dụng đang chạy.

**DAX (DynamoDB Accelerator)**
Cache in-memory được quản lý hoàn toàn, tích hợp trực tiếp với DynamoDB. Giảm độ trễ từ **millisecond xuống microsecond** cho read-heavy workloads. API tương thích hoàn toàn với DynamoDB — chỉ cần thay endpoint, không cần thay đổi code.

**Transactions (Giao dịch ACID)**
Hỗ trợ ACID transactions trên nhiều items và nhiều bảng cùng lúc. Đảm bảo "all-or-nothing" — hoặc tất cả thao tác trong transaction đều thành công, hoặc tất cả đều được rollback.

**TTL (Time-to-Live)**
Cho phép đặt thời gian hết hạn cho từng item. Khi đến thời điểm hết hạn, item tự động bị xóa mà **không tốn thêm chi phí** write. Rất hữu ích cho session data, temporary data, hay log cần giữ trong thời gian ngắn.

**Encryption (Mã hóa)**
Mã hóa at-rest mặc định bằng AWS KMS. Toàn bộ traffic được mã hóa in-transit bằng TLS.

---

## Slide 18: DynamoDB Use Cases — Trường Hợp Sử Dụng

**Mobile và Gaming**
Quản lý user profiles, session data, leaderboards, và game state cho hàng triệu người dùng đồng thời. DynamoDB xử lý spike traffic từ gaming events (tournament, launch day) mà không cần over-provision.

**IoT Applications**
Lưu trữ time-series data từ hàng triệu sensor, device metadata, và event tracking ở quy mô lớn. DynamoDB scale tự động khi số lượng device tăng mà không cần thay đổi kiến trúc.

**Real-Time Bidding (Đấu thầu thời gian thực)**
Nền tảng quảng cáo (ad tech) cần response time microsecond và xử lý spike traffic không thể dự đoán. DynamoDB với DAX cung cấp đúng loại hiệu năng này.

**Ví dụ kiến trúc Gaming Platform:**
- **Player profiles:** On-demand mode — traffic bất thường, không cần dự đoán
- **Leaderboards:** GSI với score làm Sort Key — query top players cực nhanh
- **Session data:** TTL tự động xóa session hết hạn — không tốn chi phí quản lý
- **Game events:** DynamoDB Streams → Lambda để xử lý sự kiện real-time (achievement, reward)
- **Global players:** Global Tables để đảm bảo độ trễ thấp ở châu Á, châu Âu, Mỹ

---

## Slide 19: Amazon ElastiCache — Tiêu Đề Phần

Tiếp theo là **Amazon ElastiCache** — dịch vụ caching in-memory giúp giảm thiểu độ trễ và tải cho database backend.

---

## Slide 20: Amazon ElastiCache Overview — Tổng Quan

**Amazon ElastiCache** là dịch vụ caching in-memory được quản lý hoàn toàn, hỗ trợ hai engine phổ biến nhất: **Redis** và **Memcached**, cung cấp độ trễ microsecond.

**Tại sao cần caching?**
Hầu hết các web application đều có pattern: 80% traffic là các read request giống nhau lặp đi lặp lại — trang chủ, danh mục sản phẩm, thông tin người dùng. Nếu mỗi request đều truy cập database, database sẽ nhanh chóng trở thành bottleneck. ElastiCache cho phép **giảm tải database đến hơn 80%** bằng cách cache kết quả query.

**Lợi ích chính:**
- **Sub-millisecond response times:** Nhanh hơn 100-1000 lần so với truy cập database trực tiếp
- **Reduce database load 80%+:** Giảm đáng kể tải cho RDS/DynamoDB
- **Fully managed:** Tự động phát hiện node (auto-discovery), tự động failover, tự động backup
- **Scaling với minimal downtime:** Có thể thêm/bớt node mà ứng dụng gần như không bị gián đoạn
- **Enhanced security:** Triển khai trong VPC, hỗ trợ encryption in-transit và at-rest

**Các use case phổ biến:**
- Cache kết quả query database
- Session store cho web application
- Real-time analytics (leaderboard, counter)
- Message queue và pub/sub
- Rate limiting và throttling

---

## Slide 21: Redis vs Memcached — So Sánh Chi Tiết

| Tính năng | ElastiCache for Redis | ElastiCache for Memcached |
|-----------|----------------------|--------------------------|
| **Kiểu dữ liệu** | Strings, Lists, Sets, Sorted Sets, Hashes, Bitmaps, HyperLogLogs | Chỉ key-value strings đơn giản |
| **Persistence** | Snapshots và AOF (Append-Only File) | Không có persistence |
| **Replication** | Multi-AZ với auto failover, tối đa 5 read replicas | Không có replication |
| **Transactions** | Hỗ trợ transactions và Lua scripting | Không có |
| **Pub/Sub** | Có, với pattern matching | Không |
| **Multi-threading** | Single-threaded | Multi-threaded, tốt hơn cho node lớn |
| **Phù hợp nhất** | Cấu trúc dữ liệu phức tạp, persistence, HA | Caching đơn giản, horizontal scaling |

**Khi nào chọn Redis?**
- Cần Sorted Sets cho leaderboard
- Cần persistence — dữ liệu phải sống qua restart
- Cần High Availability với auto failover
- Cần pub/sub messaging
- Cần Lua scripting cho atomic operations phức tạp

**Khi nào chọn Memcached?**
- Cần pure caching đơn giản nhất
- Scale ngang (horizontal) bằng cách thêm node
- Muốn tận dụng multi-threading trên server có nhiều CPU cores
- Không cần persistence hay replication

---

## Slide 22: ElastiCache Use Cases — Trường Hợp Sử Dụng

**Database Caching (Cache kết quả database)**
Pattern phổ biến nhất: cache kết quả query từ RDS hoặc DynamoDB để tránh truy cập database cho mỗi request.
Ba chiến lược caching chính:
- **Cache-aside (Lazy loading):** Ứng dụng check cache trước; nếu miss thì query DB và lưu vào cache
- **Write-through:** Mỗi khi write vào DB, đồng thời cập nhật cache — đảm bảo cache luôn up-to-date
- **Lazy loading:** Chỉ load vào cache khi được request — cache chỉ chứa dữ liệu thực sự được dùng

**Session Management (Quản lý phiên người dùng)**
Lưu user session trong ElastiCache thay vì database để:
- Truy cập cực nhanh (sub-millisecond)
- Application server stateless — có thể scale hoặc restart bất kỳ server nào mà không mất session
- TTL tự động hết hạn session cũ
- Đặc biệt quan trọng khi dùng Auto Scaling Group

**Real-Time Analytics (Phân tích thời gian thực)**
Redis Sorted Sets là công cụ lý tưởng cho:
- **Leaderboard:** `ZADD` để thêm điểm, `ZREVRANK` để lấy rank ngay lập tức
- **Counters:** `INCR` đảm bảo atomic increment — không có race condition
- **Rate limiting:** Kiểm tra và giới hạn số request từ một IP/user trong khoảng thời gian

---

## Slide 23: Amazon Redshift — Tiêu Đề Phần

Bây giờ chuyển sang thế giới analytics với **Amazon Redshift** — data warehouse được quản lý ở quy mô petabyte.

---

## Slide 24: Amazon Redshift Overview — Tổng Quan

**Amazon Redshift** là data warehouse được quản lý hoàn toàn, được tối ưu cho các truy vấn analytics phức tạp và workload Business Intelligence.

**Khác biệt cơ bản so với RDS/Aurora:**
Trong khi RDS tối ưu cho OLTP (nhiều transaction nhỏ, nhanh), Redshift tối ưu cho OLAP — ít transaction nhưng mỗi query xử lý hàng tỷ rows để tổng hợp, so sánh, và phân tích dữ liệu.

**Khả năng chính:**
- **Columnar storage:** Lưu dữ liệu theo cột thay vì theo hàng. Khi query chỉ cần 5 cột trong bảng 100 cột, Redshift chỉ đọc 5 cột đó — tiết kiệm I/O đáng kể
- **Massively Parallel Processing (MPP):** Phân tán query trên nhiều node, xử lý song song
- **Standard SQL:** Tương thích với SQL tiêu chuẩn, dễ dàng kết nối với BI tools
- **Data lake integration:** Truy vấn trực tiếp dữ liệu trong S3 qua Redshift Spectrum

**Hiệu năng và giá trị:**
- Nhanh hơn **10 lần** so với data warehouse truyền thống
- Giá/hiệu năng tốt hơn **3 lần** so với các cloud data warehouse khác
- Mở rộng đến **exabyte** với Redshift Spectrum

---

## Slide 25: Redshift Architecture — Kiến Trúc

**Cluster Architecture (Kiến trúc cluster)**
Redshift hoạt động theo mô hình cluster:
- **Leader Node:** Nhận SQL query từ client, phân tích, tạo execution plan, và điều phối các compute nodes
- **Compute Nodes:** Thực thi query song song, mỗi node xử lý một phần dữ liệu
- Scale từ **1 đến 128 compute nodes** — càng nhiều node, query càng nhanh

**Columnar Storage (Lưu trữ theo cột)**
Thay vì lưu theo hàng (row-based như RDS), Redshift lưu dữ liệu theo cột. Lợi ích:
- **Compression cao hơn** vì cùng một cột thường có giá trị tương tự nhau
- **Đọc chỉ cột cần thiết** — query `SELECT revenue, date` không cần đọc 50 cột khác
- **Aggregate functions** (SUM, AVG, COUNT) cực kỳ nhanh trên columnar data

**Redshift Spectrum**
Tính năng đặc biệt: cho phép query trực tiếp **exabytes dữ liệu trong S3** mà không cần load vào Redshift cluster. Mở rộng khả năng phân tích ra ngoài giới hạn của cluster storage.

**Concurrency Scaling**
Khi có nhiều user chạy query cùng lúc (spike), Redshift tự động thêm cluster capacity. **1 giờ đầu mỗi ngày miễn phí** — tiết kiệm chi phí đáng kể.

**Node Types:**
- **RA3:** Tách biệt compute và storage — linh hoạt nhất, khuyến nghị cho hầu hết workload
- **DC2:** Dense compute — tốt cho workload cần compute mạnh với hot data
- **DS2:** Dense storage — legacy, không khuyến nghị cho project mới

---

## Slide 26: Redshift Use Cases — Trường Hợp Sử Dụng

**Business Intelligence (Phân tích kinh doanh)**
Query phức tạp trên dataset lớn, tạo dashboard và báo cáo. Tích hợp tự nhiên với **Tableau, Power BI, Amazon QuickSight** và hầu hết BI tools qua JDBC/ODBC.

**Data Lake Analytics**
Kết hợp structured data trong Redshift cluster với semi-structured data (JSON, Parquet, ORC) trong S3 qua Spectrum. Tạo unified analytics layer trên toàn bộ data lake.

**Operational Analytics**
Phân tích near real-time trên dữ liệu vận hành, kết hợp với streaming data từ Kinesis để có insights gần thời gian thực.

**Ví dụ kiến trúc Retail Analytics:**
- **Sales data:** Nhiều năm lịch sử giao dịch trong Redshift
- **Log data:** Raw clickstream (hành vi người dùng) trong S3, query qua Spectrum — tiết kiệm chi phí
- **Real-time:** Amazon Kinesis → Redshift Streaming để cập nhật gần thời gian thực
- **Reporting:** Amazon QuickSight dashboard cho business users
- **Machine Learning:** Amazon SageMaker tích hợp để chạy ML model trên data trong Redshift

---

## Slide 27: Specialized Databases — Tiêu Đề Phần

Ngoài các dịch vụ phổ biến trên, AWS còn cung cấp các database chuyên biệt cho những bài toán cụ thể.

---

## Slide 28: Amazon DocumentDB — Database Tài Liệu JSON

**Amazon DocumentDB** là database document được quản lý hoàn toàn, tương thích với MongoDB, được thiết kế cho workload JSON.

**Tương thích MongoDB:**
- Hỗ trợ MongoDB 3.6, 4.0, và 5.0 API
- Code viết cho MongoDB chạy được trên DocumentDB mà không cần thay đổi nhiều
- Migrate từ MongoDB lên DocumentDB dễ dàng bằng AWS DMS

**Tính năng kỹ thuật:**
- Fully managed với auto-scaling storage
- Tối đa **15 read replicas** để scale read workload
- Sao lưu liên tục lên S3
- **6-way replication trên 3 AZs** — tương tự Aurora
- Mã hóa at-rest và in-transit

**Use cases:**
- **Content Management Systems:** Nội dung có cấu trúc linh hoạt, thay đổi theo thời gian
- **Product Catalogs:** Sản phẩm với nhiều loại attributes khác nhau — giày dép vs điện tử vs thực phẩm
- **User Profiles:** Thông tin người dùng đa dạng, không đồng nhất
- **Mobile App Backends:** Lưu trữ app data dạng JSON
- **Real-time Big Data:** Ingestion và query dữ liệu lớn dạng JSON

**Tại sao không tự cài MongoDB trên EC2?**
DocumentDB cung cấp fully managed experience: không cần quản lý OS, tự động backup, HA tốt hơn với 6-way replication, và tách biệt compute/storage để scale độc lập.

---

## Slide 29: Amazon Neptune — Graph Database

**Amazon Neptune** là database graph được quản lý hoàn toàn, hỗ trợ hai query language phổ biến nhất trong thế giới graph database.

**Hai mô hình graph:**
- **Property Graph** với **Apache TinkerPop Gremlin** — phổ biến trong enterprise, dễ học
- **RDF (Resource Description Framework)** với **W3C SPARQL** — chuẩn semantic web

**Tính năng kỹ thuật:**
- ACID transactions — đảm bảo tính nhất quán của graph
- Tối đa **15 read replicas**
- Point-in-time recovery
- Multi-AZ High Availability
- Failover nhanh (dưới 30 giây)

**Tại sao cần Graph Database?**
Với relational database, để tìm "bạn bè của bạn bè của tôi" cần nhiều vòng JOIN cực kỳ tốn kém. Graph database lưu trữ dữ liệu dưới dạng nodes (đỉnh) và edges (cạnh), cho phép **traverse relationships hàng triệu bước** với hiệu năng cao hơn **nhiều bậc độ lớn** so với JOIN trong SQL.

**Use cases:**
- **Social Networking Graphs:** Gợi ý kết bạn, phân tích mạng lưới quan hệ
- **Fraud Detection:** Phát hiện pattern gian lận qua mạng lưới giao dịch phức tạp
- **Recommendation Engines:** "Người mua sản phẩm này cũng mua..."
- **Knowledge Graphs:** Lưu trữ và truy vấn kiến thức có cấu trúc
- **Network and IT Operations:** Map topology mạng, phân tích dependency
- **Life Sciences Research:** Phân tích mạng lưới protein, drug interaction

---

## Slide 30: Database Comparison Matrix — Ma Trận So Sánh

Tổng hợp toàn bộ các dịch vụ trong một bảng so sánh:

| Dịch vụ | Loại | Độ trễ | Quy mô | Phù hợp nhất cho |
|---------|------|---------|--------|-----------------|
| **RDS** | Relational (SQL) | Single-digit ms | Vertical, tối đa 64TB | Ứng dụng truyền thống, ACID transactions |
| **Aurora** | Relational (SQL) | Single-digit ms | Tối đa 128TB, 15 replicas | Hiệu năng cao, cloud-native apps |
| **DynamoDB** | Key-Value/Document | Single-digit ms | Gần như không giới hạn | Web app quy mô lớn, gaming, IoT |
| **ElastiCache** | In-Memory | Sub-millisecond | Horizontal scaling | Caching, session store, real-time |
| **Redshift** | Data Warehouse | Vài giây | Petabytes | Analytics, BI, complex queries |
| **DocumentDB** | Document | Single-digit ms | Tối đa 64TB | Quản lý nội dung, catalogs |
| **Neptune** | Graph | Single-digit ms | Hàng tỷ relationships | Mạng xã hội, fraud detection |

**Nhận xét tổng quan:**
- Không có database nào "tốt nhất" cho mọi tình huống
- Latency giảm dần khi đi từ disk-based → in-memory
- Scale tăng dần từ relational → NoSQL
- Bài toán nào, công cụ đó — đây là triết lý cốt lõi

---

## Slide 31: When to Use Each Database — Khi Nào Dùng Gì

Sơ đồ quyết định nhanh:

**→ Dùng RDS khi:**
- Cần ACID transactions
- Dữ liệu có cấu trúc rõ ràng với nhiều JOINs
- Ứng dụng đã viết SQL, cần migrate lên cloud
- Team quen với SQL, Oracle, SQL Server

**→ Dùng Aurora khi:**
- Cần hiệu năng SQL cao hơn RDS
- Ứng dụng mission-critical, cần failover nhanh
- Cần Global Database cho người dùng toàn cầu
- Workload lớn, quy mô đến 128TB

**→ Dùng DynamoDB khi:**
- Cần scale không giới hạn (hàng triệu request/giây)
- Serverless, không muốn quản lý capacity
- Workload NoSQL — key-value, document
- Gaming, mobile, IoT, real-time web

**→ Dùng ElastiCache khi:**
- Cần caching để giảm tải database
- Session management cho stateless app servers
- Cần sub-millisecond latency
- Leaderboard, rate limiting với Redis

**→ Dùng Redshift khi:**
- Business Intelligence và analytics
- Query phức tạp trên hàng tỷ rows
- Báo cáo, dashboard cho business users
- Data warehouse, data lake analytics

**→ Dùng DocumentDB khi:**
- Đã có code MongoDB muốn đưa lên cloud
- Dữ liệu JSON linh hoạt, schema thay đổi
- Content management, product catalogs

**→ Dùng Neptune khi:**
- Dữ liệu về mối quan hệ (social graph, fraud network)
- Cần traverse nhiều bước quan hệ hiệu quả
- SQL JOINs quá chậm cho bài toán của bạn

---

## Slide 32: Migration Strategies — Tiêu Đề Phần

Bây giờ chúng ta nói về **làm thế nào để đưa database hiện tại lên AWS**.

---

## Slide 33: Migration Strategies Overview — Ba Chiến Lược Di Chuyển

**1. Rehost (Lift-and-Shift) — Chuyển nguyên trạng**
Di chuyển database "as-is" lên AWS mà không thay đổi gì.

- **Tools:** AWS Application Migration Service, backup/restore
- **Tốc độ:** Nhanh nhất, rủi ro thấp nhất
- **Phù hợp:** Khi cần migrate nhanh, budget hạn chế, hoặc giai đoạn đầu của cloud journey
- **Hạn chế:** Không tận dụng được lợi ích của cloud managed services, vẫn phải tự quản lý OS và database

**2. Replatform — Di chuyển lên managed service**
Chuyển từ self-managed lên AWS managed service với thay đổi tối thiểu.

- **Ví dụ:** Oracle trên EC2 → RDS Oracle; MySQL on-premise → RDS MySQL
- **Tools:** AWS DMS (Database Migration Service), native replication
- **Tốc độ:** Trung bình
- **Lợi ích:** Giảm đáng kể TCO (Total Cost of Ownership), AWS quản lý OS và database software
- **Phù hợp:** Hầu hết doanh nghiệp muốn cân bằng giữa tốc độ và lợi ích

**3. Refactor — Thiết kế lại cho cloud**
Thiết kế lại kiến trúc để tận dụng tối đa cloud-native capabilities.

- **Ví dụ:** Oracle monolith → Aurora PostgreSQL + DynamoDB + ElastiCache
- **Tools:** AWS DMS, Schema Conversion Tool (SCT), viết lại application code
- **Tốc độ:** Chậm nhất, đòi hỏi nhiều nỗ lực nhất
- **Lợi ích:** Tối đa — hiệu năng cao nhất, chi phí tối ưu nhất, scale tốt nhất
- **Phù hợp:** Ứng dụng chiến lược, long-term investment

---

## Slide 34: AWS Database Migration Service (DMS) — Dịch Vụ Di Chuyển

**AWS DMS** là dịch vụ giúp migrate database lên AWS với **minimal downtime** (gần như không có downtime).

**Cơ chế hoạt động:**
DMS tạo một replication instance, kết nối đến source database và target database. Nó đọc dữ liệu từ source, chuyển đổi nếu cần, và ghi vào target. Trong suốt quá trình này, **source database vẫn hoạt động bình thường** — người dùng không bị ảnh hưởng.

**Khả năng chính:**
- **Homogeneous migrations (cùng engine):** MySQL → RDS MySQL, Oracle → RDS Oracle — đơn giản, ít rủi ro
- **Heterogeneous migrations (khác engine):** Oracle → PostgreSQL, SQL Server → Aurora — phức tạp hơn, cần Schema Conversion Tool
- **Ongoing replication:** Sau khi migrate xong, có thể tiếp tục sync changes từ on-premise lên AWS — dùng cho hybrid scenarios
- **Automatic schema conversion:** AWS SCT (Schema Conversion Tool) hỗ trợ chuyển đổi schema và code

**Ba pattern migration phổ biến:**
- **Homogeneous:** Cùng engine, đơn giản nhất
- **Heterogeneous:** Khác engine, cần schema conversion
- **Dev/Test replication:** Copy production data sang môi trường dev/test thường xuyên

---

## Slide 35: Migration Best Practices — Thực Hành Tốt Nhất

**Bước 1: Assessment (Đánh giá)**
- Kiểm kê toàn bộ database: engine, version, size, số connection
- Đánh giá compatibility với target service
- Ước tính chi phí và timeline
- Xác định dependencies: ứng dụng nào kết nối database này?

**Bước 2: Proof of Concept (Thử nghiệm)**
- Test migration với một phần dữ liệu đại diện
- Kiểm tra performance của target database
- Xác nhận tính đúng đắn của dữ liệu sau migration
- Đo lường thực tế latency và throughput

**Bước 3: Migration Execution (Thực hiện migration)**
- Dùng DMS để sync liên tục, minimize downtime
- Chuẩn bị cutover window (cửa sổ chuyển đổi) vào giờ thấp điểm
- Validation dữ liệu sau migration bằng automated scripts
- Có rollback plan rõ ràng nếu có vấn đề

**Bước 4: Optimization (Tối ưu hóa)**
- Right-size instance dựa trên actual usage (không phải estimate)
- Enable auto-scaling phù hợp
- Thêm caching layer (ElastiCache) cho hot data
- Optimize query cho đặc điểm của managed service mới

**Bước 5: Monitoring and Operations (Giám sát vận hành)**
- Cài đặt CloudWatch alarms cho CPU, memory, connections, latency
- Enable Enhanced Monitoring và Performance Insights để deep-dive troubleshoot
- Xác lập backup và retention policy
- Viết runbook cho các tình huống sự cố phổ biến

---

## Slide 36: Cost Optimization Tips — Tối Ưu Chi Phí

Database thường là một trong những mục chi phí lớn nhất trên AWS. Đây là 5 nhóm chiến lược tối ưu:

**Right-Sizing (Chọn đúng kích thước)**
- Monitor CloudWatch metrics để biết CPU, memory thực tế đang ở mức nào
- Dùng **Performance Insights** để xác định query bottleneck
- Bắt đầu nhỏ, scale up dựa trên real data — đừng over-provision từ đầu
- Cân nhắc **Burstable instances (T3/T4g)** cho workload không liên tục

**Reserved Instances (Đặt trước)**
- Cam kết 1 hoặc 3 năm để tiết kiệm tới **69%** so với On-Demand
- **Convertible Reserved Instances** cho phép thay đổi instance type trong kỳ hạn
- **Savings Plans** cũng áp dụng được cho RDS

**Storage Optimization (Tối ưu lưu trữ)**
- Aurora: storage tự động scale, không cần over-provision
- Giảm backup retention period nếu không cần đến 35 ngày
- Chọn đúng IOPS tier — đừng mua IOPS cao hơn nhu cầu thực tế
- Archive dữ liệu lịch sử sang S3 (rẻ hơn nhiều)

**Read Scaling (Mở rộng đọc)**
- Dùng read replicas để giảm tải primary, thay vì scale primary lên
- ElastiCache giảm database read đến 80%+ — đầu tư nhỏ, tiết kiệm lớn
- Aurora Serverless cho workload không đều
- Offload analytics queries sang Redshift, không dùng production RDS

**DynamoDB Specific**
- On-demand mode cho traffic không thể dự đoán
- Provisioned mode + Reserved Capacity cho workload ổn định — tiết kiệm đáng kể
- TTL tự động xóa dữ liệu cũ — giảm storage cost và không tốn WCU

**Monitoring Chi Phí**
- Set up **AWS Billing Alerts** để nhận cảnh báo khi chi phí vượt ngưỡng
- Dùng **Cost Explorer** để phân tích trend và tìm anomaly
- **Tag tất cả resources** theo project/team/environment để phân bổ chi phí
- Review cost hàng tháng — chi phí cloud có thể thay đổi nhanh

---

## Slide 37: Summary — Tóm Tắt

**5 điểm cốt lõi cần ghi nhớ:**

**1. Chọn đúng công cụ**
Không có database nào phù hợp cho mọi tình huống. Luôn bắt đầu bằng câu hỏi: workload là gì? Cấu trúc dữ liệu như thế nào? Yêu cầu scale và latency ra sao?

**2. Ưu tiên managed services**
RDS, Aurora, DynamoDB, ElastiCache — tất cả đều là fully managed. Điều này giải phóng đội ngũ kỹ thuật khỏi việc quản lý hạ tầng, để tập trung vào business logic.

**3. Thiết kế cho High Availability ngay từ đầu**
Multi-AZ, read replicas, backup — đây không phải "nice to have" mà là "must have" cho production workload. Chi phí HA nhỏ hơn nhiều so với chi phí downtime.

**4. Lập kế hoạch migration cẩn thận**
Assess → PoC → Execute → Optimize. Đừng migrate không có chuẩn bị. DMS giúp migrate với minimal downtime nhưng vẫn cần planning kỹ lưỡng.

**5. Tối ưu liên tục**
Môi trường cloud linh động — right-size thường xuyên, thêm caching khi cần, monitor chi phí và điều chỉnh. Cloud không phải set-and-forget.

**Quick Decision Guide:**
- **Transactions + SQL?** → RDS hoặc Aurora
- **Massive scale + NoSQL?** → DynamoDB
- **Caching?** → ElastiCache
- **Analytics?** → Redshift
- **Documents JSON?** → DocumentDB
- **Relationships/Graph?** → Neptune

---

## Slide 38: Next Steps — Các Bước Tiếp Theo

Để bắt đầu hành trình AWS Database của bạn:

**1. Assess your current database workloads**
Kiểm kê toàn bộ database hiện tại: engine, version, size, SLA, dependencies. Hiểu rõ bức tranh hiện tại trước khi lên kế hoạch.

**2. Review AWS documentation and best practices**
AWS cung cấp tài liệu phong phú tại docs.aws.amazon.com. Mỗi dịch vụ đều có best practices guide và well-architected guidance.

**3. Set up a proof-of-concept environment**
Tạo môi trường sandbox miễn phí (AWS Free Tier) để thử nghiệm. Hands-on experience là cách học hiệu quả nhất.

**4. Engage with AWS Solutions Architects**
AWS cung cấp dịch vụ tư vấn miễn phí từ Solutions Architects — đặc biệt hữu ích khi lên kế hoạch migration phức tạp.

**5. Plan your migration roadmap**
Xây dựng roadmap rõ ràng: database nào migrate trước (quick wins), database nào cần refactor (long-term), timeline, và budget cho từng giai đoạn.

---

*Kết thúc bài thuyết trình. Cảm ơn mọi người đã theo dõi!*