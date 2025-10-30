# AWS ARCHITECTURE Personal Learning Roadmap
## Thiết kế Kiến trúc AWS theo Well-Architected Framework

---

## **MỤC TIÊU HỌC TẬP**
- Nắm vững 4 trụ cột chính của AWS Well-Architected Framework
- Thiết kế được kiến trúc bảo mật, linh hoạt, hiệu năng cao và tối ưu chi phí
- Hiểu rõ pricing model và deployment patterns của các dịch vụ AWS core

---

## **CHƯƠNG 1: THIẾT KẾ KIẾN TRÚC BẢO MẬT (Security Pillar)**

### **1.1 Lý thuyết cơ bản**
- **Nguyên tắc Defense in Depth**: Nhiều lớp bảo mật
- **Principle of Least Privilege**: Quyền tối thiểu cần thiết
- **Identity-based vs Resource-based policies**

### **1.2 Các dịch vụ bảo mật chính**

#### **IAM (Identity and Access Management)**
- **Mô tả tổng quan**: Dịch vụ quản lý "ai được làm gì" trong AWS. Giống như hệ thống phân quyền trong công ty - xác định nhân viên nào có thể truy cập phòng nào, sử dụng thiết bị gì.
- **Chức năng chính**: Tạo users, gán permissions, quản lý access keys, thiết lập MFA
- **Sử dụng khi nào**: Mỗi khi cần kiểm soát quyền truy cập AWS resources - từ developer cần quyền deploy code đến application cần quyền đọc S3
- **Kết quả trả về**: Allow/Deny decisions cho mỗi API call, temporary credentials cho applications
- **Thành phần**: Users, Groups, Roles, Policies
- **Pricing**: Miễn phí cho core features
- **Vị trí**: Global service (không thuộc VPC)
- **Thực hành**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

#### **MFA (Multi-Factor Authentication)**
- **Mô tả tổng quan**: Lớp bảo mật thứ hai sau password. Giống như ATM cần cả thẻ và mã PIN - AWS cần cả password và mã từ điện thoại/thiết bị.
- **Chức năng chính**: Tạo mã xác thực 6 số thay đổi mỗi 30 giây, yêu cầu nhập khi login
- **Sử dụng khi nào**: Bắt buộc cho Root user, khuyến nghị cho tất cả privileged users, required cho compliance
- **Kết quả trả về**: Token hợp lệ cho phép tiếp tục truy cập, hoặc từ chối nếu mã sai
- **Loại**: Virtual MFA, Hardware MFA, SMS
- **Pricing**: Miễn phí cho Virtual MFA
- **Best Practice**: Bắt buộc cho Root user và privileged users

#### **SCP (Service Control Policies)**
- **Mô tả tổng quan**: "Rào chắn" cao nhất trong tổ chức AWS. Giống như quy định công ty cấm nhân viên truy cập một số trang web - SCP cấm accounts trong Organization sử dụng một số AWS services.
- **Chức năng chính**: Đặt giới hạn tối đa cho permissions, không thể grant thêm quyền
- **Sử dụng khi nào**: Multi-account environment, cần kiểm soát centralized, compliance requirements (VD: cấm tạo EC2 ở regions không được phép)
- **Kết quả trả về**: Deny hoặc cho phép AWS API calls ở organization level
- **Vị trí**: Organization level (không thuộc VPC)
- **Pricing**: Miễn phí với AWS Organizations
- **Ví dụ**: Ngăn chặn tạo EC2 instances ở regions không được phép

#### **Encryption Services**

**AWS KMS (Key Management Service)**
- **Mô tả tổng quan**: "Két sắt điện tử" chứa chìa khóa mã hóa. Giống như ngân hàng giữ chìa khóa két sắt - KMS tạo, lưu trữ và quản lý encryption keys, chỉ cho phép applications được ủy quyền sử dụng.
- **Chức năng chính**: Tạo encryption keys, mã hóa/giải mã data, rotate keys tự động, audit key usage
- **Sử dụng khi nào**: Mã hóa EBS volumes, S3 objects, RDS databases, Lambda environment variables - bất cứ khi nào cần bảo vệ sensitive data
- **Kết quả trả về**: Encrypted data hoặc decrypted plaintext, key metadata, audit logs
- **Type**: Managed service
- **Vị trí**: Regional service (không thuộc VPC)
- **Pricing**: $1/key/month + $0.03/10,000 requests
- **Ví dụ**: Mã hóa EBS volumes, S3 objects

**TLS/ACM (Certificate Manager)**
- **Mô tả tổng quan**: "Nhà cung cấp chứng chỉ SSL miễn phí" của AWS. Giống như CA (Certificate Authority) cấp chứng chỉ cho website - ACM tự động tạo, gia hạn SSL certificates để website hiển thị "khóa xanh" HTTPS.
- **Chức năng chính**: Tạo SSL/TLS certificates, tự động renewal, deploy đến AWS services
- **Sử dụng khi nào**: Website cần HTTPS, Load Balancers, CloudFront distributions, API Gateway - bất cứ khi nào cần secure communication
- **Kết quả trả về**: Valid SSL certificate, automatic renewal notifications, certificate validation
- **Type**: Managed service
- **Pricing**: Miễn phí cho public certificates
- **Vị trí**: Regional (không thuộc VPC)
- **Sử dụng**: Load Balancers, CloudFront, API Gateway

#### **Network Security**

**Security Groups**
- **Mô tả tổng quan**: "Bảo vệ cá nhân" cho từng EC2 instance. Giống như bodyguard kiểm tra từng người muốn gặp VIP - Security Group kiểm tra từng network packet muốn đến instance, chỉ cho phép traffic từ sources được phép.
- **Chức năng chính**: Filter inbound/outbound traffic, stateful connection tracking, allow rules only
- **Sử dụng khi nào**: Mỗi EC2 instance, RDS database, Load Balancer cần network protection - default security layer cho mọi AWS resource
- **Kết quả trả về**: Allow/drop packets, connection state tracking, automatic return traffic allowance
- **Khái niệm**: Stateful firewall cho EC2 instances
- **Vị trí**: VPC level
- **Pricing**: Miễn phí
- **Quy tắc**: Allow rules only, stateful

**NACLs (Network Access Control Lists)**
- **Mô tả tổng quan**: "Bảo vệ khu vực" cho cả subnet. Giống như checkpoint an ninh ở cổng khu phố - kiểm tra tất cả xe ra vào, không nhớ xe nào đã qua, phải kiểm tra lại mỗi lần.
- **Chức năng chính**: Filter traffic ở subnet level, stateless rules evaluation, both allow và deny rules
- **Sử dụng khi nào**: Cần additional security layer, compliance requirements, block specific IP ranges ở subnet level
- **Kết quả trả về**: Allow/deny decisions per packet, no connection state memory, separate inbound/outbound evaluation
- **Khái niệm**: Stateless firewall cho subnets
- **Vị trí**: Subnet level trong VPC
- **Pricing**: Miễn phí
- **Quy tắc**: Allow và Deny rules, stateless

#### **Threat Detection & Protection**

**GuardDuty**
- **Mô tả tổng quan**: "Thám tử AI" giám sát 24/7 tìm hoạt động đáng ngờ. Giống như hệ thống camera an ninh thông minh - phân tích patterns, phát hiện hành vi bất thường như login từ địa điểm lạ, malware communication.
- **Chức năng chính**: Machine learning threat detection, analyze VPC Flow Logs/DNS logs/CloudTrail, generate security findings
- **Sử dụng khi nào**: Cần continuous security monitoring, detect compromised instances, malware, cryptocurrency mining, data exfiltration
- **Kết quả trả về**: Security findings với severity levels, detailed threat intelligence, remediation recommendations
- **Type**: Managed threat detection
- **Pricing**: $4.00/million events analyzed
- **Vị trí**: Regional (không thuộc VPC)
- **Chức năng**: ML-based threat detection

**AWS Shield**
- **Mô tả tổng quan**: "Lá chắn chống DDoS" bảo vệ website khỏi tấn công từ chối dịch vụ. Giống như hệ thống lọc traffic tự động - phát hiện và chặn hàng triệu requests độc hại trước khi chúng làm sập server.
- **Chức năng chính**: DDoS protection, traffic filtering, attack mitigation, real-time monitoring
- **Sử dụng khi nào**: Website/application cần protection khỏi DDoS attacks, đặc biệt public-facing services
- **Kết quả trả về**: Filtered clean traffic, attack reports, mitigation statistics, uptime protection
- **Standard**: Miễn phí, DDoS protection cơ bản
- **Advanced**: $3,000/month, DDoS protection nâng cao
- **Type**: Managed service

**AWS WAF (Web Application Firewall)**
- **Mô tả tổng quan**: "Bảo vệ thông minh" cho web applications. Giống như bouncer club kiểm tra ID và hành vi - WAF kiểm tra HTTP requests, chặn SQL injection, XSS attacks, bot traffic dựa trên rules tùy chỉnh.
- **Chức năng chính**: HTTP/HTTPS request filtering, custom rules creation, bot detection, rate limiting
- **Sử dụng khi nào**: Web applications cần protection khỏi OWASP Top 10 attacks, API protection, bot management
- **Kết quả trả về**: Allow/block decisions, detailed request logs, attack statistics, custom responses
- **Type**: Managed service
- **Pricing**: $1/web ACL/month + $0.60/million requests
- **Vị trí**: Edge locations (không thuộc VPC)
- **Sử dụng**: CloudFront, ALB, API Gateway

**Secrets Manager**
- **Mô tả tổng quan**: "Két sắt mật khẩu thông minh" tự động thay đổi passwords. Giống như password manager cá nhân nhưng cho applications - lưu trữ database passwords, API keys, tự động rotate và cung cấp cho applications khi cần.
- **Chức năng chính**: Store secrets securely, automatic rotation, fine-grained access control, audit logging
- **Sử dụng khi nào**: Applications cần database passwords, API keys, certificates - thay thế hardcoded secrets trong code
- **Kết quả trả về**: Encrypted secrets, rotated credentials, access logs, integration với AWS services
- **Type**: Managed service
- **Pricing**: $0.40/secret/month + $0.05/10,000 API calls
- **Vị trí**: Regional (không thuộc VPC)
- **Chức năng**: Automatic rotation, encryption

### **1.3 Thực hành thiết kế**
```
Internet Gateway
       |
   Public Subnet (Web Tier)
   - ALB + WAF
   - Security Group: 80,443 from 0.0.0.0/0
       |
   Private Subnet (App Tier)  
   - EC2 instances
   - Security Group: 8080 from Web Tier SG only
       |
   Private Subnet (DB Tier)
   - RDS with encryption
   - Security Group: 3306 from App Tier SG only
```

---

## **CHƯƠNG 2: THIẾT KẾ KIẾN TRÚC LINH HOẠT VÀ BỀN VỮNG (Reliability Pillar)**

### **2.1 Lý thuyết cơ bản**
- **RTO (Recovery Time Objective)**: Thời gian phục hồi mục tiêu
- **RPO (Recovery Point Objective)**: Điểm phục hồi dữ liệu mục tiêu
- **Fault tolerance vs High availability**

### **2.2 Strategies cho Resilience**

#### **Multi-AZ Deployment**
- **Mô tả tổng quan**: "Sao lưu tự động trong cùng thành phố". Giống như có 2 văn phòng trong cùng thành phố - nếu văn phòng chính gặp sự cố (mất điện, lũ lụt), văn phòng phụ tự động tiếp quản ngay lập tức.
- **Chức năng chính**: Automatic failover, synchronous data replication, zero-downtime maintenance
- **Sử dụng khi nào**: Critical applications cần high availability, databases không thể downtime, compliance requirements
- **Kết quả trả về**: 99.95%+ uptime, automatic recovery, transparent failover cho applications
- **Khái niệm**: Triển khai trên nhiều Availability Zones
- **Pricing**: Không phí thêm cho architecture, chỉ trả cho resources
- **Ví dụ**: RDS Multi-AZ (2x cost), ELB tự động multi-AZ

#### **Multi-Region Deployment**
- **Mô tả tổng quan**: "Chi nhánh ở nhiều thành phố khác nhau". Giống như công ty có văn phòng ở Hà Nội và TP.HCM - nếu Hà Nội gặp thiên tai, TP.HCM vẫn hoạt động bình thường, khách hàng không bị gián đoạn.
- **Chức năng chính**: Geographic redundancy, disaster recovery, latency optimization, compliance với data residency
- **Sử dụng khi nào**: Global applications, disaster recovery, regulatory compliance, improve user experience
- **Kết quả trả về**: Geographic fault tolerance, reduced latency for global users, regulatory compliance
- **Khái niệm**: Triển khai trên nhiều AWS Regions
- **Pricing**: Data transfer charges giữa regions
- **Use case**: Disaster recovery, compliance, latency optimization

#### **DR (Disaster Recovery) Strategies**
1. **Backup & Restore**: RTO hours, RPO hours, cost thấp nhất
2. **Pilot Light**: RTO 10s minutes, RPO minutes, cost trung bình
3. **Warm Standby**: RTO minutes, RPO seconds, cost cao
4. **Multi-Site Active/Active**: RTO seconds, RPO near-zero, cost cao nhất

#### **Auto Scaling**
- **Mô tả tổng quan**: "Quản lý nhân sự tự động". Giống như có AI quản lý - khi khách hàng đông, tự động thuê thêm nhân viên; khi vắng khách, cho nghỉ bớt để tiết kiệm lương. Luôn đảm bảo đủ người phục vụ mà không lãng phí.
- **Chức năng chính**: Automatic capacity adjustment, health monitoring, cost optimization, performance maintenance
- **Sử dụng khi nào**: Traffic không đoán trước, cost optimization, maintain performance during peaks, replace unhealthy instances
- **Kết quả trả về**: Right-sized capacity, cost savings, consistent performance, automatic recovery
- **Type**: Managed service
- **Pricing**: Miễn phí (chỉ trả cho EC2 instances)
- **Vị trí**: Regional, hoạt động trong VPC
- **Scaling Policies**: Target tracking, Step scaling, Simple scaling

#### **Route 53**
- **Mô tả tổng quan**: "Hệ thống định vị thông minh toàn cầu". Giống như GPS thông minh - không chỉ chỉ đường đến địa chỉ (IP) mà còn chọn đường nhanh nhất, tránh kẹt xe (server down), thậm chí đưa khách đến chi nhánh gần nhất.
- **Chức năng chính**: DNS resolution, health monitoring, traffic routing, domain registration
- **Sử dụng khi nào**: Cần domain name resolution, load distribution, failover automation, geographic routing
- **Kết quả trả về**: Fast DNS responses, automatic failover, optimized routing, high availability
- **Type**: Managed DNS service
- **Pricing**: $0.50/hosted zone/month + $0.40/million queries
- **Vị trí**: Global service (không thuộc VPC)
- **Health Checks**: $0.50/health check/month
- **Routing Policies**: Simple, Weighted, Latency-based, Failover, Geolocation

#### **Load Balancing**

**Application Load Balancer (ALB)**
- **Type**: Managed service
- **Pricing**: $0.0225/hour + $0.008/LCU-hour
- **Vị trí**: VPC, multi-AZ
- **Features**: Layer 7, SSL termination, content-based routing

**Network Load Balancer (NLB)**
- **Type**: Managed service  
- **Pricing**: $0.0225/hour + $0.006/NLCU-hour
- **Vị trí**: VPC, multi-AZ
- **Features**: Layer 4, ultra-low latency, static IP

#### **Backup & Restore**
- **AWS Backup**: Centralized backup service
- **Pricing**: $0.05/GB/month + restore charges
- **EBS Snapshots**: $0.05/GB/month
- **S3 Cross-Region Replication**: Automatic replication

### **2.3 Thực hành thiết kế**
```
Route 53 (Health Checks)
    |
Primary Region          Secondary Region
    |                       |
Multi-AZ ALB           Standby ALB
    |                       |
Auto Scaling Group     Standby instances
(Min:2, Max:10)        (Min:0, Max:10)
    |                       |
RDS Multi-AZ          RDS Read Replica
```

---

## **CHƯƠNG 3: THIẾT KẾ HỆ THỐNG HIỆU NĂNG CAO (Performance Efficiency Pillar)**

### **3.1 Lý thuyết cơ bản**
- **Compute optimization**: Right-sizing, instance types
- **Storage optimization**: IOPS, throughput, latency
- **Network optimization**: Bandwidth, latency, CDN
- **Caching strategies**: In-memory, distributed, edge

### **3.2 Compute Scaling**

#### **EC2 Auto Scaling**
- **Instance Types**: General purpose (t3, m5), Compute optimized (c5), Memory optimized (r5)
- **Pricing Models**:
  - On-Demand: Pay per hour, no commitment
  - Reserved: 1-3 years, up to 75% discount
  - Spot: Up to 90% discount, interruptible
- **Vị trí**: VPC, specific subnets

#### **AWS Lambda**
- **Type**: Serverless compute
- **Pricing**: $0.20/1M requests + $0.0000166667/GB-second
- **Vị trí**: Có thể trong VPC hoặc ngoài VPC
- **Limits**: 15 minutes timeout, 10GB memory max
- **Use case**: Event-driven, microservices

#### **AWS Fargate**
- **Type**: Serverless containers
- **Pricing**: Pay for vCPU and memory used
- **Vị trí**: VPC, managed networking
- **Use case**: Containers without managing EC2

### **3.3 Storage Optimization**

#### **Amazon S3**
- **Type**: Object storage
- **Pricing**: 
  - Standard: $0.023/GB/month
  - IA: $0.0125/GB/month
  - Glacier: $0.004/GB/month
- **Vị trí**: Regional, không thuộc VPC
- **Performance**: 3,500 PUT/COPY/POST/DELETE, 5,500 GET/HEAD per prefix

#### **Amazon EFS (Elastic File System)**
- **Type**: Managed NFS
- **Pricing**: $0.30/GB/month (Standard), $0.045/GB/month (IA)
- **Vị trí**: VPC, multi-AZ
- **Performance**: General Purpose vs Max I/O

#### **Amazon EBS**
- **Types**:
  - gp3: $0.08/GB/month, 3,000 IOPS baseline
  - io2: $0.125/GB/month + $0.065/IOPS/month
- **Vị trí**: Single AZ, attached to EC2 in same AZ

### **3.4 Caching**

#### **ElastiCache**
- **Redis**: In-memory data structure, persistence
- **Memcached**: Simple caching, multi-threading
- **Pricing**: Instance-based, $0.017/hour (cache.t3.micro)
- **Vị trí**: VPC, private subnets

#### **CloudFront**
- **Type**: Global CDN
- **Pricing**: $0.085/GB (first 10TB/month)
- **Vị trí**: Edge locations globally
- **Features**: SSL/TLS, custom origins, Lambda@Edge

### **3.5 Network Optimization**

#### **AWS Global Accelerator**
- **Type**: Network performance service
- **Pricing**: $0.025/hour + data transfer charges
- **Vị trí**: Global network
- **Use case**: Improve performance for global users

### **3.6 Thực hành thiết kế**
```
CloudFront (Edge Caching)
    |
Global Accelerator
    |
ALB (Multi-AZ)
    |
Auto Scaling Group
- c5.large instances (compute optimized)
- Target tracking scaling
    |
ElastiCache Redis Cluster
    |
RDS with Read Replicas
    |
S3 with Transfer Acceleration
```

---

## **CHƯƠNG 4: THIẾT KẾ TỐI ƯU CHI PHÍ (Cost Optimization Pillar)**

### **4.1 Lý thuyết cơ bản**
- **Right-sizing**: Chọn instance type phù hợp
- **Reserved capacity**: Cam kết dài hạn để giảm cost
- **Spot instances**: Sử dụng excess capacity
- **Lifecycle management**: Tự động chuyển đổi storage classes

### **4.2 Cost Management Tools**

#### **AWS Cost Explorer**
- **Mô tả tổng quan**: "Kế toán AI phân tích chi tiêu". Giống như app quản lý tài chính cá nhân - phân tích bạn tiêu tiền vào đâu nhiều nhất, dự đoán chi tiêu tháng sau, gợi ý cách tiết kiệm (như chuyển từ taxi sang xe bus).
- **Chức năng chính**: Cost analysis, spending visualization, forecasting, rightsizing recommendations
- **Sử dụng khi nào**: Monthly cost reviews, budget planning, cost optimization, trend analysis
- **Kết quả trả về**: Detailed cost breakdowns, spending forecasts, optimization recommendations, cost trends
- **Type**: Cost analysis tool
- **Pricing**: Miễn phí cho basic reports
- **Features**: Cost breakdown, forecasting, rightsizing recommendations

#### **AWS Budgets**
- **Mô tả tổng quan**: "Hệ thống cảnh báo chi tiêu thông minh". Giống như set ngân sách hàng tháng trên app banking - khi sắp hết tiền hoặc chi tiêu bất thường sẽ gửi thông báo, thậm chí tự động khóa thẻ (stop resources).
- **Chức năng chính**: Budget setting, cost alerts, usage monitoring, automated actions
- **Sử dụng khi nào**: Cost control, spending limits, department budgets, project cost management
- **Kết quả trả về**: Budget alerts, spending notifications, automated cost controls, usage reports
- **Pricing**: $0.02/budget/day (first 2 budgets free)
- **Features**: Cost budgets, usage budgets, alerts

#### **Savings Plans**
- **Mô tả tổng quan**: "Gói cước trả trước giảm giá". Giống như mua gói cước điện thoại 12 tháng - cam kết dùng một lượng nhất định, được giảm giá đáng kể so với trả theo tháng. Càng cam kết lâu, giảm càng nhiều.
- **Chức năng chính**: Commitment-based discounts, flexible usage, automatic application, significant savings
- **Sử dụng khi nào**: Predictable workloads, long-term projects, cost optimization for steady usage
- **Kết quả trả về**: Up to 72% cost reduction, billing simplification, usage flexibility
- **Compute Savings Plans**: Up to 66% discount, flexible
- **EC2 Instance Savings Plans**: Up to 72% discount, specific instance family
- **Commitment**: 1 or 3 years

### **4.3 Storage Cost Optimization**

#### **S3 Lifecycle Policies**
```json
{
  "Rules": [
    {
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ]
    }
  ]
}
```

#### **Storage Tiering**
- **S3 Intelligent-Tiering**: $0.0025/1,000 objects/month
- **EFS Intelligent-Tiering**: Automatic movement to IA

### **4.4 Network Cost Optimization**

#### **NAT Gateway Optimization**
- **NAT Gateway**: $0.045/hour + $0.045/GB processed
- **NAT Instance**: EC2 pricing + data transfer
- **Optimization**: Use NAT Gateway in single AZ for non-critical workloads

### **4.5 Thực hành thiết kế**
```
Cost-Optimized Architecture:
- Spot instances for batch processing
- Reserved instances for baseline capacity
- S3 lifecycle policies
- Single NAT Gateway for dev environments
- CloudWatch for monitoring and rightsizing
```

---

## **CHƯƠNG 5: KIẾN THỨC CHI TIẾT VỀ VPC VÀ NETWORKING**

### **5.1 Amazon VPC (Virtual Private Cloud) - Khái niệm cốt lõi**

#### **Khái niệm cơ bản**
- **VPC**: "Trung tâm dữ liệu ảo riêng" trong AWS Cloud. Giống như thuê một tầng building riêng - bạn có toàn quyền thiết kế layout, quyết định phòng nào public/private, ai được vào ra.
- **Isolated network**: Hoàn toàn tách biệt với các VPC khác - như có tường bao quanh
- **Regional service**: Trải rộng trên tất cả AZ trong region - như building có nhiều cánh
- **CIDR Block**: Dải IP private (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) - như số nhà trong khu phố riêng

#### **Core Components của VPC**

**Subnets**
- **Mô tả tổng quan**: "Phòng chuyên dụng" trong VPC building. Public subnet như lobby - ai cũng có thể vào từ internet. Private subnet như phòng server - chỉ nhân viên nội bộ truy cập được.
- **Chức năng chính**: Phân chia VPC theo chức năng và security level, định vị resources trong specific AZ
- **Sử dụng khi nào**: Tách biệt web servers (public) và databases (private), implement security layers, distribute resources across AZs
- **Kết quả trả về**: Isolated network segments, controlled internet access, AZ-specific placement
- **Khái niệm**: Phân đoạn VPC theo AZ
- **Public Subnet**: Có route đến Internet Gateway
- **Private Subnet**: Không có route trực tiếp đến Internet
- **CIDR**: Phải nằm trong CIDR của VPC
- **Triển khai**:
```bash
# Tạo VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Tạo Public Subnet
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.1.0/24 --availability-zone us-east-1a

# Tạo Private Subnet  
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.2.0/24 --availability-zone us-east-1a
```

**Internet Gateway (IGW)**
- **Mô tả tổng quan**: "Cổng ra vào internet" duy nhất của VPC. Giống như cổng chính của building - tất cả traffic muốn ra internet phải đi qua đây, IGW tự động "dịch" private IP thành public IP.
- **Chức năng chính**: Enable internet connectivity, perform NAT cho public IP addresses, route traffic bidirectionally
- **Sử dụng khi nào**: VPC cần internet access, public-facing resources như web servers, load balancers
- **Kết quả trả về**: Internet connectivity, automatic IP translation, bidirectional traffic flow
- **Khái niệm**: Cổng kết nối VPC với Internet
- **Chức năng**: NAT cho public IP addresses
- **Quy tắc**: 1 VPC = 1 IGW maximum
- **Pricing**: Miễn phí
- **Triển khai**:
```bash
# Tạo và attach IGW
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id vpc-xxx --internet-gateway-id igw-xxx
```

**Route Tables**
- **Mô tả tổng quan**: "Bản đồ chỉ đường" cho network traffic. Giống như GPS - khi có packet muốn đi đến địa chỉ nào đó, route table chỉ đường "đi qua cổng nào, theo đường nào".
- **Chức năng chính**: Define traffic routing rules, associate với subnets, determine packet destinations
- **Sử dụng khi nào**: Control traffic flow trong VPC, direct internet traffic qua IGW, route private traffic qua NAT Gateway
- **Kết quả trả về**: Traffic routing decisions, packet forwarding to correct targets, network path determination
- **Khái niệm**: Bảng định tuyến cho subnets
- **Main Route Table**: Default cho tất cả subnets
- **Custom Route Table**: Tạo riêng cho specific subnets
- **Routes**: Destination CIDR + Target
- **Triển khai**:
```bash
# Tạo route table
aws ec2 create-route-table --vpc-id vpc-xxx

# Thêm route đến IGW
aws ec2 create-route --route-table-id rtb-xxx --destination-cidr-block 0.0.0.0/0 --gateway-id igw-xxx

# Associate với subnet
aws ec2 associate-route-table --subnet-id subnet-xxx --route-table-id rtb-xxx
```

### **5.2 NAT Gateway - Chi tiết kỹ thuật**

#### **Khái niệm và chức năng**
- **NAT (Network Address Translation)**: "Người phiên dịch địa chỉ" cho private instances. Giống như receptionist - nhận requests từ nhân viên bên trong (private IP), gọi ra ngoài bằng số điện thoại công ty (public IP), rồi chuyển response về đúng người.
- **Mục đích**: Cho phép instances trong private subnet truy cập Internet mà không expose private IP
- **Hướng**: Chỉ outbound (từ trong ra ngoài), không cho phép inbound connections từ internet
- **Managed Service**: AWS quản lý hoàn toàn - không cần maintain, patch, scale

#### **Loại NAT Gateway**

**Public NAT Gateway**
- **Mô tả tổng quan**: "Cổng ra internet" cho private resources. Đặt trong public subnet, có Elastic IP, cho phép private instances download updates, gọi APIs bên ngoài mà không bị expose.
- **Chức năng chính**: Translate private IPs to public IP, enable outbound internet access, block inbound connections
- **Sử dụng khi nào**: Private instances cần internet access (software updates, API calls, downloading packages)
- **Kết quả trả về**: Successful outbound connections, blocked inbound attempts, translated IP addresses
- **Vị trí**: Phải đặt trong Public Subnet
- **Elastic IP**: Bắt buộc phải có
- **Routing**: Private Subnet → NAT Gateway → Internet Gateway → Internet
- **Use case**: Internet access cho private instances
- **Pricing**: $0.045/hour + $0.045/GB processed

**Private NAT Gateway**
- **Mô tả tổng quan**: "Cầu nối nội bộ" kết nối với networks khác (không phải internet). Giống như switchboard nội bộ - chỉ kết nối với các chi nhánh khác hoặc data center on-premises.
- **Chức năng chính**: Enable connectivity to other VPCs/on-premises, translate IPs for internal communication
- **Sử dụng khi nào**: Multi-VPC architecture, hybrid cloud setup, internal service communication
- **Kết quả trả về**: Successful connections to other networks, IP translation for internal traffic
- **Vị trí**: Đặt trong Private Subnet
- **Elastic IP**: Không cần
- **Routing**: Chỉ đến VPC khác hoặc on-premises qua Transit Gateway/VPN
- **Use case**: Kết nối với networks khác, không phải Internet

#### **Triển khai NAT Gateway**
```bash
# Tạo Public NAT Gateway
aws ec2 create-nat-gateway \
    --subnet-id subnet-xxx \
    --allocation-id eipalloc-xxx \
    --connectivity-type public

# Cập nhật route table của Private Subnet
aws ec2 create-route \
    --route-table-id rtb-private \
    --destination-cidr-block 0.0.0.0/0 \
    --nat-gateway-id nat-xxx
```

#### **NAT Gateway vs NAT Instance**
| Feature | NAT Gateway | NAT Instance |
|---------|-------------|--------------|
| Availability | Managed, HA trong AZ | Tự quản lý |
| Bandwidth | Up to 45 Gbps | Phụ thuộc instance type |
| Maintenance | AWS quản lý | Tự maintain |
| Cost | $0.045/hour + data | EC2 pricing |
| Security Groups | Không hỗ trợ | Hỗ trợ |

### **5.3 Security Groups vs NACLs - So sánh chi tiết**

#### **Security Groups**
- **Level**: Instance level (ENI level)
- **State**: Stateful (return traffic tự động allow)
- **Rules**: Chỉ Allow rules
- **Evaluation**: Tất cả rules được evaluate
- **Default**: Deny all inbound, Allow all outbound
- **Triển khai**:
```bash
# Tạo Security Group
aws ec2 create-security-group \
    --group-name web-sg \
    --description "Web server security group" \
    --vpc-id vpc-xxx

# Thêm rule
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

#### **Network ACLs (NACLs)**
- **Level**: Subnet level
- **State**: Stateless (phải config cả inbound và outbound)
- **Rules**: Allow và Deny rules
- **Evaluation**: Rules theo thứ tự số thấp đến cao
- **Default**: Allow all traffic
- **Triển khai**:
```bash
# Tạo NACL
aws ec2 create-network-acl --vpc-id vpc-xxx

# Thêm rule
aws ec2 create-network-acl-entry \
    --network-acl-id acl-xxx \
    --rule-number 100 \
    --protocol tcp \
    --port-range From=80,To=80 \
    --cidr-block 0.0.0.0/0
```

## **CHƯƠNG 6: DỊCH VỤ TRỌNG TÂM AWS - CORE FEATURES**

### **6.1 Amazon EC2 - Compute Service**

#### **Khái niệm cơ bản**
- **EC2**: "Máy tính ảo cho thuê". Giống như thuê máy tính trong internet cafe - chọn cấu hình phù hợp (CPU mạnh, RAM nhiều), trả tiền theo giờ sử dụng, có thể tắt bật tùy ý, cài đặt software tùy thích.
- **Instance**: Virtual machine với CPU, memory, storage, networking được định sẵn
- **AMI**: "Ảnh chụp máy tính" - Template chứa OS và software để tạo instance mới
- **Instance Store**: "Ổ cứng tạm thời" - mất data khi tắt máy, tốc độ cao

#### **Instance Types và Use Cases**

**General Purpose (Balanced)**
- **t3/t4g**: "Máy tính tiết kiệm điện thông minh". Giống như laptop có chế độ tiết kiệm pin - bình thường chạy chậm để tiết kiệm, khi cần thì tăng tốc (burst), phù hợp công việc văn phòng không đều.
  - **Chức năng chính**: Burstable CPU performance, credit-based system, cost-effective
  - **Sử dụng khi nào**: Web servers nhỏ, development environments, low-traffic applications
  - **Kết quả trả về**: Cost savings, adequate performance for variable workloads
  - Use case: Web servers, small databases, development
  - Pricing: $0.0104/hour (t3.micro)
- **m5/m6i**: "Máy tính văn phòng cân bằng". Giống như PC desktop chuẩn - CPU, RAM, network đều ở mức vừa phải, phù hợp đa số công việc thông thường.
  - **Chức năng chính**: Balanced compute, memory, networking resources
  - **Sử dụng khi nào**: General web applications, microservices, enterprise applications
  - **Kết quả trả về**: Consistent performance, good price-performance ratio
  - Use case: Web applications, microservices
  - Pricing: $0.096/hour (m5.large)

**Compute Optimized**
- **c5/c6i**: "Máy tính gaming cao cấp". Giống như PC gaming với CPU mạnh nhất - xử lý tính toán phức tạp, render video, chạy game nặng mượt mà.
  - **Chức năng chính**: High-performance processors, optimized for CPU-intensive tasks
  - **Sử dụng khi nào**: Scientific computing, gaming servers, high-performance web servers, machine learning inference
  - **Kết quả trả về**: Superior CPU performance, fast processing, low latency
  - Use case: CPU-intensive applications, HPC, gaming
  - Pricing: $0.085/hour (c5.large)

**Memory Optimized**
- **r5/r6i**: "Máy tính với RAM khủng". Giống như workstation chuyên dụng với 64GB+ RAM - xử lý datasets lớn, cache nhiều data trong memory, phù hợp databases lớn.
  - **Chức năng chính**: High memory-to-vCPU ratio, fast memory access
  - **Sử dụng khi nào**: In-memory databases, real-time analytics, big data processing
  - **Kết quả trả về**: Fast data access, large dataset processing, reduced I/O bottlenecks
  - Use case: In-memory databases, real-time analytics
  - Pricing: $0.126/hour (r5.large)
- **x1e**: "Siêu máy tính memory". Giống như server chuyên dụng với RAM lên đến 4TB - xử lý enterprise databases cực lớn.
  - **Chức năng chính**: Extreme memory capacity (up to 3,904 GB)
  - **Sử dụng khi nào**: SAP HANA, Apache Spark, large enterprise databases
  - **Kết quả trả về**: Massive in-memory processing capability
  - Use case: SAP HANA, Apache Spark

**Storage Optimized**
- **i3**: "Máy tính với SSD siêu tốc". Giống như workstation gắn nhiều SSD NVMe - đọc ghi data cực nhanh, phù hợp applications cần I/O cao.
  - **Chức năng chính**: NVMe SSD-backed instance storage, high IOPS
  - **Sử dụng khi nào**: Distributed file systems, data warehousing, high-frequency trading
  - **Kết quả trả về**: Ultra-fast storage access, high throughput
  - Use case: Distributed file systems, data warehousing

#### **Pricing Models**
- **On-Demand**: Pay per hour, no commitment
- **Reserved Instances**: 1-3 years, up to 75% discount
- **Spot Instances**: Up to 90% discount, can be interrupted
- **Dedicated Hosts**: Physical server dedicated to you

#### **Core Features và Triển khai**

**Auto Scaling**
```bash
# Tạo Launch Template
aws ec2 create-launch-template \
    --launch-template-name web-template \
    --launch-template-data '{
        "ImageId": "ami-xxx",
        "InstanceType": "t3.micro",
        "SecurityGroupIds": ["sg-xxx"]
    }'

# Tạo Auto Scaling Group
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name web-asg \
    --launch-template LaunchTemplateName=web-template \
    --min-size 2 \
    --max-size 10 \
    --desired-capacity 2 \
    --vpc-zone-identifier "subnet-xxx,subnet-yyy"
```

**Placement Groups**
- **Cluster**: Low latency, high throughput trong single AZ
- **Partition**: Spread across logical partitions
- **Spread**: Spread across underlying hardware

### **6.2 Amazon S3 - Object Storage**

#### **Khái niệm cơ bản**
- **Object Storage**: Lưu trữ files dưới dạng objects trong buckets
- **Bucket**: Container cho objects, globally unique name
- **Object**: File + metadata, up to 5TB per object
- **Key**: Unique identifier cho object trong bucket

#### **Storage Classes và Use Cases**

**Standard (Frequent Access)**
- **Mô tả tổng quan**: "Kho hàng cao cấp 24/7". Giống như kho hàng trong siêu thị - hàng được đặt ở vị trí dễ lấy nhất, nhân viên luôn sẵn sàng, lấy hàng trong vài giây, nhưng chi phí thuê mặt bằng cao nhất.
- **Chức năng chính**: Instant access, high durability, frequent retrieval optimization
- **Sử dụng khi nào**: Website assets, mobile apps, content distribution, frequently accessed data
- **Kết quả trả về**: Millisecond access, 99.99% availability, immediate retrieval
- **Durability**: 99.999999999% (11 9's)
- **Availability**: 99.99%
- **Pricing**: $0.023/GB/month
- **Use case**: Active data, websites, content distribution

**Standard-IA (Infrequent Access)**
- **Mô tả tổng quan**: "Kho hàng ít dùng". Giống như kho hàng ở tầng 2 - hàng ít lấy, thuê rẻ hơn nhưng mỗi lần lấy phải trả phí thêm cho nhân viên lên lầu.
- **Chức năng chính**: Lower storage cost, retrieval fee, same durability as Standard
- **Sử dụng khi nào**: Backups, disaster recovery, long-term storage với occasional access
- **Kết quả trả về**: Cost savings, quick access when needed, retrieval charges apply
- **Pricing**: $0.0125/GB/month + retrieval fee
- **Minimum**: 30 days storage, 128KB object size
- **Use case**: Backups, disaster recovery

**One Zone-IA**
- **Mô tả tổng quan**: "Kho hàng giá rẻ một địa điểm". Giống như thuê kho ở vùng ven, giá rẻ nhưng rủi ro cao hơn - nếu vùng đó ngập lụt thì mất hàng.
- **Chức năng chính**: Lower cost than Standard-IA, single AZ storage, same access speed
- **Sử dụng khi nào**: Secondary backup copies, reproducible data, cost-sensitive storage
- **Kết quả trả về**: Lowest IA cost, single point of failure risk
- **Pricing**: $0.01/GB/month
- **Availability**: 99.5% (single AZ)
- **Use case**: Secondary backup copies

**Glacier Storage Classes**
- **Glacier Instant Retrieval**: "Kho lạnh cao cấp". Giống như kho lạnh công nghệ cao - hàng đông lạnh để bảo quản lâu, nhưng có robot lấy hàng tức thì khi cần.
  - **Chức năng chính**: Long-term storage, instant retrieval, very low cost
  - **Sử dụng khi nào**: Archive data cần access ngay lập tức nhưng hiếm khi dùng
  - **Kết quả trả về**: Instant access, significant cost savings, long-term preservation
  - $0.004/GB/month, milliseconds retrieval
- **Glacier Flexible Retrieval**: "Kho lạnh thông thường". Giống như kho lạnh truyền thống - hàng đông cứng, cần thời gian rã đông trước khi dùng được.
  - **Chức năng chính**: Flexible retrieval options (minutes to hours), very low storage cost
  - **Sử dụng khi nào**: Archive data, compliance, long-term backups
  - **Kết quả trả về**: Ultra-low storage cost, configurable retrieval time
  - $0.0036/GB/month, minutes to hours
- **Glacier Deep Archive**: "Kho lạnh sâu". Giống như hầm trữ đông sâu - hàng đông cứng như đá, cần nửa ngày để rã đông, nhưng chi phí cực thấp.
  - **Chức năng chính**: Lowest cost storage, long retrieval time, compliance-focused
  - **Sử dụng khi nào**: Long-term compliance, rarely accessed archives, tape replacement
  - **Kết quả trả về**: Minimum storage cost, 12-hour retrieval time
  - $0.00099/GB/month, 12 hours retrieval

#### **Core Features và Triển khai**

**Versioning**
```bash
# Enable versioning
aws s3api put-bucket-versioning \
    --bucket my-bucket \
    --versioning-configuration Status=Enabled
```

**Lifecycle Policies**
```json
{
  "Rules": [
    {
      "ID": "TransitionRule",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ]
    }
  ]
}
```

**Cross-Region Replication**
```bash
# Enable CRR
aws s3api put-bucket-replication \
    --bucket source-bucket \
    --replication-configuration file://replication.json
```

**S3 Transfer Acceleration**
- **Chức năng**: Sử dụng CloudFront edge locations để upload nhanh hơn
- **Pricing**: $0.04/GB transferred
- **Use case**: Upload files từ xa đến S3

### **6.3 Amazon RDS - Managed Database**

#### **Khái niệm cơ bản**
- **RDS**: Relational Database Service - Managed database
- **DB Instance**: Database server trong cloud
- **DB Engine**: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora
- **Multi-AZ**: Synchronous replication cho high availability

#### **Core Features**

**Multi-AZ Deployment**
- **Mô tả tổng quan**: "Database có phó bản tự động". Giống như có 2 kế toán trưởng - người chính ghi sổ, người phụ copy y hệt ngay lập tức. Nếu người chính ốm, người phụ lên thay ngay không cần training.
- **Chức năng chính**: Synchronous replication, automatic failover, zero data loss, maintenance without downtime
- **Sử dụng khi nào**: Production databases, critical applications, compliance requirements, zero-downtime maintenance
- **Kết quả trả về**: High availability, automatic recovery, data protection, seamless failover
- **Chức năng**: Automatic failover đến standby instance
- **RTO**: 1-2 minutes
- **Pricing**: 2x cost của single-AZ
- **Triển khai**:
```bash
aws rds create-db-instance \
    --db-instance-identifier mydb \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --master-username admin \
    --master-user-password mypassword \
    --allocated-storage 20 \
    --multi-az
```

**Read Replicas**
- **Mô tả tổng quan**: "Nhân viên tư vấn chuyên đọc". Giống như có thêm nhân viên chỉ chuyên trả lời câu hỏi khách hàng (read queries) để nhân viên chính tập trung xử lý đơn hàng (write operations). Khách hỏi gì cũng được, nhưng không thể đặt hàng qua nhân viên tư vấn.
- **Chức năng chính**: Asynchronous replication, read scaling, cross-region support, offload read traffic
- **Sử dụng khi nào**: Read-heavy workloads, reporting, analytics, geographic distribution
- **Kết quả trả về**: Improved read performance, reduced load on primary, global data distribution
- **Chức năng**: Asynchronous replication cho read scaling
- **Limit**: Up to 5 read replicas
- **Cross-Region**: Supported
- **Use case**: Read-heavy workloads, reporting

**Automated Backups**
- **Retention**: 1-35 days
- **Point-in-time recovery**: Down to second
- **Backup window**: Configurable
- **Pricing**: Free up to DB instance storage size

**Parameter Groups**
- **Chức năng**: Database configuration settings
- **Default**: AWS-provided defaults
- **Custom**: Tạo custom configurations

### **6.4 AWS Lambda - Serverless Compute**

#### **Khái niệm cơ bản**
- **Serverless**: "Thuê lập trình viên theo giờ". Giống như gọi thợ sửa ống nước - chỉ trả tiền khi thực sự cần, không phải nuôi thợ cả tháng. AWS lo tất cả, bạn chỉ cần viết code.
- **Event-driven**: "Nhân viên chỉ làm khi có chuông báo". Chỉ chạy khi có events (file upload, HTTP request, database change), không chạy thì không tốn tiền.
- **Function**: "Đoạn code nhỏ làm 1 việc cụ thể". Giống như app mini - nhận input, xử lý, trả output, rồi tắt.
- **Runtime**: "Môi trường chạy code" - hỗ trợ nhiều ngôn ngữ lập trình

#### **Core Features**

**Execution Model**
- **Mô tả tổng quan**: "Nhân viên làm việc theo ca". Mỗi lần có việc, AWS thuê nhân viên mới (cold start - hơi chậm lần đầu), nhưng nếu liên tục có việc thì nhân viên đó ở lại (warm - nhanh hơn). Tối đa 15 phút/ca, sau đó nghỉ.
- **Chức năng chính**: On-demand execution, automatic scaling, pay-per-use, managed infrastructure
- **Sử dụng khi nào**: Event processing, APIs, data transformation, automation tasks
- **Kết quả trả về**: Processed events, API responses, transformed data, automated actions
- **Timeout**: Maximum 15 minutes
- **Memory**: 128MB to 10,240MB
- **Concurrent executions**: 1,000 default limit
- **Cold start**: Initial latency khi function chưa warm

**Pricing Model**
- **Requests**: $0.20 per 1M requests
- **Duration**: $0.0000166667 per GB-second
- **Free tier**: 1M requests + 400,000 GB-seconds/month

**VPC Configuration**
```bash
aws lambda create-function \
    --function-name my-function \
    --runtime python3.9 \
    --role arn:aws:iam::account:role/lambda-role \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip \
    --vpc-config SubnetIds=subnet-xxx,SecurityGroupIds=sg-xxx
```

**Event Sources**
- **API Gateway**: HTTP requests
- **S3**: Object events
- **DynamoDB**: Stream events
- **CloudWatch Events**: Scheduled events
- **SQS**: Message queue events

### **6.5 Amazon CloudWatch - Monitoring & Observability**

#### **Khái niệm cơ bản**
- **Monitoring Service**: Thu thập và theo dõi metrics, logs, events
- **Metrics**: Numerical data points over time
- **Alarms**: Notifications dựa trên metric thresholds
- **Logs**: Centralized log management

#### **Core Features**

**CloudWatch Metrics**
- **Mô tả tổng quan**: "Đồng hồ đo hiệu suất tự động". Giống như dashboard xe hơi - liên tục hiển thị tốc độ (CPU), nhiên liệu (memory), nhiệt độ engine, cảnh báo khi có vấn đề. AWS tự động thu thập, bạn chỉ cần xem biểu đồ.
- **Chức năng chính**: Automatic metric collection, custom metric support, data visualization, historical analysis
- **Sử dụng khi nào**: Monitor application performance, track resource utilization, troubleshoot issues, capacity planning
- **Kết quả trả về**: Performance graphs, trend analysis, threshold alerts, operational insights
- **Default Metrics**: CPU, Network, Disk (không có Memory)
- **Custom Metrics**: Application-specific metrics
- **Pricing**: $0.30/metric/month
- **Resolution**: Standard (5 minutes) hoặc High (1 minute)
- **Triển khai**:
```bash
# Gửi custom metric
aws cloudwatch put-metric-data \
    --namespace "MyApp" \
    --metric-data MetricName=PageViews,Value=100,Unit=Count
```

**CloudWatch Alarms**
- **Mô tả tổng quan**: "Hệ thống báo động thông minh". Giống như báo cháy trong building - theo dõi liên tục, khi nhiệt độ (CPU) vượt ngưỡng thì kêu báo, gửi tin nhắn cho bảo vệ (SNS), thậm chí tự động bật sprinkler (Auto Scaling).
- **Chức năng chính**: Threshold monitoring, automated actions, notification sending, state tracking
- **Sử dụng khi nào**: Proactive monitoring, automated scaling, incident response, SLA monitoring
- **Kết quả trả về**: Real-time alerts, automated responses, state change notifications, operational actions
- **States**: OK, ALARM, INSUFFICIENT_DATA
- **Actions**: SNS notifications, Auto Scaling, EC2 actions
- **Pricing**: $0.10/alarm/month
- **Triển khai**:
```bash
aws cloudwatch put-metric-alarm \
    --alarm-name cpu-high \
    --alarm-description "CPU usage high" \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --statistic Average \
    --period 300 \
    --threshold 80 \
    --comparison-operator GreaterThanThreshold
```

**CloudWatch Logs**
- **Log Groups**: Container cho log streams
- **Log Streams**: Sequence of log events
- **Retention**: 1 day to 10 years
- **Pricing**: $0.50/GB ingested, $0.03/GB stored

**CloudWatch Events/EventBridge**
- **Event-driven architecture**: React to AWS service events
- **Rules**: Route events to targets
- **Targets**: Lambda, SQS, SNS, EC2, etc.

### **6.6 Amazon CloudFront - Content Delivery Network**

#### **Khái niệm cơ bản**
- **CDN**: Global network of edge locations
- **Edge Location**: Cache content gần users
- **Origin**: Source of content (S3, ALB, EC2, custom)
- **Distribution**: Configuration for content delivery

#### **Core Features**

**Caching Behavior**
- **Mô tả tổng quan**: "Hệ thống kho hàng toàn cầu thông minh". Giống như chuỗi cửa hàng tiện lợi - lưu hàng bán chạy ở gần khách hàng, tự động bổ sung khi hết, quyết định hàng nào nên giữ bao lâu dựa trên tần suất mua.
- **Chức năng chính**: Global content caching, automatic cache management, intelligent content delivery, edge optimization
- **Sử dụng khi nào**: Website acceleration, media streaming, API acceleration, global content distribution
- **Kết quả trả về**: Faster load times, reduced origin load, improved user experience, global availability
- **TTL**: Time To Live cho cached objects
- **Cache-Control Headers**: Control caching behavior
- **Query String Parameters**: Include/exclude from cache key
- **Triển khai**:
```bash
aws cloudfront create-distribution \
    --distribution-config '{
        "CallerReference": "my-distribution-2024",
        "Origins": {
            "Quantity": 1,
            "Items": [{
                "Id": "S3-my-bucket",
                "DomainName": "my-bucket.s3.amazonaws.com",
                "S3OriginConfig": {
                    "OriginAccessIdentity": ""
                }
            }]
        },
        "DefaultCacheBehavior": {
            "TargetOriginId": "S3-my-bucket",
            "ViewerProtocolPolicy": "redirect-to-https"
        }
    }'
```

**Security Features**
- **OAI (Origin Access Identity)**: Restrict S3 access to CloudFront only
- **Signed URLs/Cookies**: Private content access
- **AWS WAF Integration**: Web application firewall
- **SSL/TLS**: Free certificates via ACM

**Performance Features**
- **Gzip Compression**: Automatic compression
- **HTTP/2**: Modern protocol support
- **Lambda@Edge**: Run code at edge locations

### **6.7 Elastic Load Balancing - Load Distribution**

#### **Application Load Balancer (ALB)**
- **Mô tả tổng quan**: "Nhân viên phân luồng thông minh". Giống như receptionist cao cấp - không chỉ phân khách đến quầy trống mà còn hiểu nội dung (đọc được HTTP headers), chuyển khách VIP đến phòng đặc biệt, khách thường đến quầy bình thường.
- **Chức năng chính**: Layer 7 load balancing, content-based routing, SSL termination, WebSocket support
- **Sử dụng khi nào**: Web applications, microservices, cần routing dựa trên URL/headers, SSL offloading
- **Kết quả trả về**: Distributed traffic, improved performance, SSL termination, health-based routing
- **Type**: Managed service
- **Pricing**: $0.0225/hour + $0.008/LCU-hour
- **Vị trí**: VPC, multi-AZ
- **Features**: Layer 7, SSL termination, content-based routing

**Core Features**:
- **Path-based routing**: Route based on URL path
- **Host-based routing**: Route based on hostname
- **Health checks**: Monitor target health
- **Triển khai**:
```bash
aws elbv2 create-load-balancer \
    --name my-alb \
    --subnets subnet-xxx subnet-yyy \
    --security-groups sg-xxx

aws elbv2 create-target-group \
    --name my-targets \
    --protocol HTTP \
    --port 80 \
    --vpc-id vpc-xxx \
    --health-check-path /health
```

**Network Load Balancer (NLB)**
- **Mô tả tổng quan**: "Bảo vệ siêu tốc". Giống như traffic cop chuyên nghiệp - không cần đọc giấy tờ, chỉ nhìn biển số xe (IP/Port) là phân luồng ngay lập tức, tốc độ cực nhanh, độ trễ cực thấp.
- **Chức năng chính**: Layer 4 load balancing, ultra-low latency, static IP support, TCP/UDP handling
- **Sử dụng khi nào**: High-performance applications, gaming, IoT, cần static IP, extreme low latency
- **Kết quả trả về**: Ultra-fast routing, consistent IP addresses, millions of requests/second
- **Type**: Managed service  
- **Pricing**: $0.0225/hour + $0.006/NLCU-hour
- **Vị trí**: VPC, multi-AZ
- **Features**: Layer 4, ultra-low latency, static IP

#### **Gateway Load Balancer (GWLB)**
- **Use case**: Third-party virtual appliances
- **Protocol**: GENEVE on port 6081
- **Features**: Transparent network gateway + load balancer

### **6.8 Amazon Route 53 - DNS Service**

#### **Khái niệm cơ bản**
- **DNS Service**: Domain Name System
- **Hosted Zone**: Container for DNS records
- **Record Types**: A, AAAA, CNAME, MX, TXT, etc.
- **Global Service**: Không thuộc region cụ thể

#### **Core Features**

**Routing Policies**
- **Simple**: "Chỉ đường đơn giản". Giống như biển báo đường - chỉ 1 hướng duy nhất, tất cả đều đi theo đường đó.
  - **Chức năng**: Single resource mapping, basic DNS resolution
  - **Sử dụng khi nào**: Simple websites, single server applications
  - **Kết quả**: All traffic goes to one destination
- **Weighted**: "Phân chia theo tỷ lệ". Giống như chia khách hàng - 70% đến cửa hàng A, 30% đến cửa hàng B để test hiệu quả.
  - **Chức năng**: Distribute traffic by percentage, A/B testing support
  - **Sử dụng khi nào**: Blue-green deployments, A/B testing, gradual rollouts
  - **Kết quả**: Controlled traffic distribution, testing capabilities
- **Latency-based**: "Chọn đường gần nhất". Giống như GPS chọn đường ít kẹt xe - tự động chỉ đến server gần nhất để giảm độ trễ.
  - **Chức năng**: Route to lowest latency endpoint, performance optimization
  - **Sử dụng khi nào**: Global applications, performance-critical services
  - **Kết quả**: Improved user experience, reduced latency
- **Failover**: "Đường dự phòng". Giống như có đường chính và đường phụ - bình thường đi đường chính, khi tắc đường thì tự động chuyển sang đường phụ.
  - **Chức năng**: Primary/secondary setup, automatic failover
  - **Sử dụng khi nào**: High availability requirements, disaster recovery
  - **Kết quả**: Automatic failover, business continuity
- **Geolocation**: "Chỉ đường theo địa chỉ". Giống như hệ thống phân phối - khách Hà Nội đến kho Hà Nội, khách TP.HCM đến kho TP.HCM.
  - **Chức năng**: Route based on user geographic location, content localization
  - **Sử dụng khi nào**: Content localization, compliance requirements, regional services
  - **Kết quả**: Localized content, regulatory compliance
- **Geoproximity**: "Chỉ đường theo khoảng cách có điều chỉnh". Giống như GPS có thể điều chỉnh - server gần nhưng đang quá tải thì chuyển hướng đến server xa hơn nhưng rảnh hơn.
  - **Chức năng**: Route based on geographic proximity with bias adjustment
  - **Sử dụng khi nào**: Load balancing with geographic considerations, capacity management
  - **Kết quả**: Optimized geographic distribution, flexible traffic control
- **Multivalue**: "Đưa nhiều lựa chọn". Giống như GPS đưa 3 tuyến đường khác nhau, để thiết bị tự chọn tuyến nào đi.
  - **Chức năng**: Return multiple IP addresses, client-side load balancing
  - **Sử dụng khi nào**: Simple load balancing, multiple healthy endpoints
  - **Kết quả**: Multiple options for clients, basic redundancy

**Health Checks**
- **Endpoint monitoring**: HTTP, HTTPS, TCP
- **Calculated health checks**: Combine multiple checks
- **CloudWatch integration**: Metrics and alarms
- **Pricing**: $0.50/health check/month

**Triển khai**:
```bash
# Tạo hosted zone
aws route53 create-hosted-zone \
    --name example.com \
    --caller-reference 2024-01-01

# Tạo A record
aws route53 change-resource-record-sets \
    --hosted-zone-id Z123456789 \
    --change-batch '{
        "Changes": [{
            "Action": "CREATE",
            "ResourceRecordSet": {
                "Name": "www.example.com",
                "Type": "A",
                "TTL": 300,
                "ResourceRecords": [{"Value": "1.2.3.4"}]
            }
        }]
    }'
```

## **CHƯƠNG 7: KIẾN TRÚC PATTERNS VÀ BEST PRACTICES**

### **7.1 Multi-Tier Architecture Pattern**

#### **3-Tier Architecture**
- **Mô tả tổng quan**: "Tòa nhà 3 tầng chuyên dụng". Tầng 1 (Presentation) đón khách, tầng 2 (Application) xử lý công việc, tầng 3 (Database) lưu trữ hồ sơ. Mỗi tầng có bảo vệ riêng, chỉ tầng trên mới liên lạc được với tầng dưới.
- **Chức năng chính**: Separation of concerns, scalability per tier, security isolation, maintainability
- **Sử dụng khi nào**: Traditional web applications, enterprise systems, clear business logic separation
- **Kết quả trả về**: Scalable architecture, security layers, maintainable codebase, fault isolation
```
Presentation Tier (Public Subnet)
├── ALB + CloudFront
├── Auto Scaling Group (Web Servers)
└── Security Group: 80,443 from Internet

Application Tier (Private Subnet)
├── Auto Scaling Group (App Servers)  
├── ElastiCache (Session Storage)
└── Security Group: 8080 from Web Tier only

Data Tier (Private Subnet)
├── RDS Multi-AZ (Primary Database)
├── RDS Read Replicas (Read Scaling)
└── Security Group: 3306 from App Tier only
```

#### **Triển khai Best Practices**
- **Separation of Concerns**: Mỗi tier có responsibility riêng
- **Security Groups**: Least privilege access
- **Auto Scaling**: Horizontal scaling cho từng tier
- **Load Balancing**: Distribute traffic evenly

### **7.2 Microservices Architecture Pattern**

#### **Container-based Microservices**
- **Mô tả tổng quan**: "Khu phố các cửa hàng chuyên môn". Thay vì 1 siêu thị lớn, có nhiều cửa hàng nhỏ chuyên về 1 lĩnh vực (User shop, Product shop, Payment shop). Mỗi shop tự quản lý, có thể mở rộng riêng, nếu 1 shop đóng cửa thì các shop khác vẫn hoạt động.
- **Chức năng chính**: Service independence, technology diversity, fault isolation, independent scaling
- **Sử dụng khi nào**: Large applications, multiple teams, rapid development, technology flexibility needs
- **Kết quả trả về**: Faster development, better fault tolerance, independent deployments, technology choices
```
API Gateway
├── Authentication Service (Lambda)
├── User Service (ECS Fargate)
├── Product Service (EKS)
├── Order Service (Lambda)
└── Payment Service (ECS Fargate)

Data Layer
├── DynamoDB (User data)
├── RDS (Product catalog)
├── ElastiCache (Session cache)
└── S3 (File storage)
```

#### **Service Communication**
- **Synchronous**: API Gateway, ALB
- **Asynchronous**: SQS, SNS, EventBridge
- **Service Discovery**: AWS Cloud Map
- **Circuit Breaker**: Implement in application code

### **7.3 Event-Driven Architecture Pattern**

#### **Serverless Event Processing**
- **Mô tả tổng quan**: "Hệ thống phản ứng chuỗi tự động". Giống như domino - khi có sự kiện xảy ra (file upload, user click, timer), nó kích hoạt chuỗi phản ứng tự động (resize image, send email, update database) mà không cần ai điều khiển.
- **Chức năng chính**: Event-driven processing, loose coupling, automatic scaling, real-time responses
- **Sử dụng khi nào**: Real-time processing, IoT applications, workflow automation, reactive systems
- **Kết quả trả về**: Real-time processing, automatic responses, scalable event handling, loose coupling
```
Event Sources
├── S3 (Object events)
├── DynamoDB (Stream events)
├── API Gateway (HTTP events)
└── CloudWatch Events (Scheduled)

Processing Layer
├── Lambda Functions
├── Step Functions (Workflow)
└── SQS (Queue processing)

Storage Layer
├── DynamoDB (NoSQL)
├── S3 (Object storage)
└── ElastiSearch (Search/Analytics)
```

### **7.4 Disaster Recovery Patterns**

#### **Backup and Restore**
- **Mô tả tổng quan**: "Sao lưu đơn giản như chụp ảnh". Giống như chụp ảnh tài liệu quan trọng lưu trong két sắt - khi cần thì lấy ra photo lại, mất vài giờ nhưng chi phí thấp nhất.
- **Chức năng chính**: Data backup, long-term storage, cost-effective recovery
- **Sử dụng khi nào**: Non-critical systems, cost-sensitive environments, acceptable downtime
- **Kết quả trả về**: Data protection, lowest cost, longer recovery time
- **RTO**: Hours, **RPO**: Hours, **Cost**: Lowest

#### **Pilot Light**
- **Mô tả tổng quan**: "Lò sưởi dự phòng". Giống như có lò sưởi backup luôn có lửa nhỏ - khi lò chính hỏng, chỉ cần thêm củi vào lò backup là có lửa lớn ngay, nhanh hơn đốt từ đầu.
- **Chức năng chính**: Minimal running infrastructure, quick scale-up capability
- **Sử dụng khi nào**: Critical systems with moderate RTO requirements
- **Kết quả trả về**: Faster recovery than backup/restore, moderate cost
- **RTO**: 10s of minutes, **RPO**: Minutes, **Cost**: Medium

#### **Warm Standby**
- **Mô tả tổng quan**: "Xe dự phòng luôn nổ máy". Giống như có xe backup luôn nổ máy sẵn - khi xe chính hỏng, nhảy sang xe backup và tăng tốc ngay lập tức.
- **Chức năng chính**: Scaled-down version always running, quick scale-up
- **Sử dụng khi nào**: Business-critical applications, short RTO requirements
- **Kết quả trả về**: Quick recovery, higher availability, higher cost
- **RTO**: Minutes, **RPO**: Seconds, **Cost**: Higher

#### **Multi-Site Active/Active**
- **Mô tả tổng quan**: "Hai văn phòng hoạt động song song". Giống như có 2 văn phòng cùng làm việc - nếu văn phòng này gặp sự cố, văn phòng kia tiếp tục ngay không gián đoạn.
- **Chức năng chính**: Full redundancy, zero downtime, load distribution
- **Sử dụng khi nào**: Mission-critical systems, zero downtime requirements
- **Kết quả trả về**: No downtime, immediate failover, highest cost
- **RTO**: Seconds, **RPO**: Near zero, **Cost**: Highest

## **CHƯƠNG 8: COST OPTIMIZATION STRATEGIES**

### **8.1 Right-Sizing Strategies**

#### **EC2 Right-Sizing**
- **CloudWatch metrics**: CPU, Memory, Network utilization
- **AWS Compute Optimizer**: ML-based recommendations
- **Instance families**: Match workload characteristics
- **Burstable instances**: t3/t4g for variable workloads

#### **Storage Right-Sizing**
- **EBS volume types**: gp3 vs io2 based on IOPS requirements
- **S3 storage classes**: Lifecycle policies for cost optimization
- **EFS storage classes**: Standard vs IA based on access patterns

### **8.2 Reserved Capacity Strategies**

#### **EC2 Reserved Instances**
- **Standard RI**: Up to 75% discount, specific instance type
- **Convertible RI**: Up to 54% discount, can change instance type
- **Scheduled RI**: For predictable recurring schedules

#### **RDS Reserved Instances**
- **Up to 69% discount** compared to On-Demand
- **Multi-AZ deployment**: Separate reservation needed
- **Size flexibility**: Within same instance family

### **8.3 Spot Instance Strategies**

#### **Spot Instance Best Practices**
- **Fault-tolerant workloads**: Batch processing, CI/CD
- **Diversification**: Multiple instance types and AZ
- **Spot Fleet**: Automatic diversification
- **Mixed instance types**: Combine On-Demand + Spot

#### **Spot Instance Implementation**
```bash
aws ec2 request-spot-fleet \
    --spot-fleet-request-config '{
        "IamFleetRole": "arn:aws:iam::account:role/fleet-role",
        "AllocationStrategy": "diversified",
        "TargetCapacity": 10,
        "SpotPrice": "0.05",
        "LaunchSpecifications": [
            {
                "ImageId": "ami-xxx",
                "InstanceType": "m5.large",
                "KeyName": "my-key",
                "SubnetId": "subnet-xxx"
            }
        ]
    }'
```

## **CHƯƠNG 9: THỰC HÀNH TỔNG HỢP**

### **9.1 Kịch bản: E-commerce Website**

#### **Yêu cầu**:
- High availability (99.9% uptime)
- Auto scaling based on traffic
- Secure payment processing
- Global content delivery
- Cost optimization

#### **Architecture Design**:
```
Route 53 (DNS + Health Checks)
    |
CloudFront (Global CDN)
    |
WAF (Web Application Firewall)
    |
ALB (Multi-AZ Load Balancer)
    |
Auto Scaling Group (Web Tier)
- EC2 instances in private subnets
- Security Group: 80,443 from ALB only
    |
ElastiCache (Session storage)
    |
Auto Scaling Group (App Tier)
- EC2 instances in private subnets  
- Security Group: 8080 from Web Tier only
    |
RDS Multi-AZ (Database)
- Private subnets
- Encrypted at rest
- Security Group: 3306 from App Tier only
    |
S3 (Static content + backups)
- Lifecycle policies
- Cross-region replication
```

#### **Cost Estimation**:
- ALB: ~$20/month
- EC2 (4 x t3.medium): ~$120/month
- RDS (db.t3.medium Multi-AZ): ~$80/month
- S3: ~$10/month
- CloudFront: ~$15/month
- **Total: ~$245/month**

### **9.2 Best Practices Summary**

#### **Security**:
- Enable MFA cho tất cả users
- Sử dụng IAM roles thay vì access keys
- Encrypt data at rest và in transit
- Regular security audits với GuardDuty

#### **Reliability**:
- Multi-AZ deployment cho critical components
- Automated backups và disaster recovery testing
- Health checks và auto-recovery
- Chaos engineering practices

#### **Performance**:
- Right-size instances based on monitoring
- Use caching layers (ElastiCache, CloudFront)
- Optimize database queries và indexing
- Network optimization với placement groups

#### **Cost Optimization**:
- Regular rightsizing reviews
- Use Reserved Instances cho predictable workloads
- Implement lifecycle policies
- Monitor và alert on cost anomalies

---

## **CHƯƠNG 10: BÀI TẬP THỰC HÀNH**

### **Bài tập 1**: Thiết kế kiến trúc 3-tier web application
- Yêu cầu: HA, scalable, secure
- Budget: $200/month
- Traffic: 1000 concurrent users

### **Bài tập 2**: DR strategy cho critical application
- RTO: 30 minutes
- RPO: 5 minutes  
- Primary region: us-east-1
- DR region: us-west-2

### **Bài tập 3**: Cost optimization cho existing architecture
- Current cost: $1000/month
- Target: Reduce 30% without performance impact
- Workload: Batch processing + web application

### **Bài tập 4**: VPC Design Challenge
- **Mô tả tổng quan**: "Thiết kế khu đô thị thông minh". Tạo một khu phố có đầy đủ tiện ích - khu dân cư (private subnets), khu thương mại (public subnets), hệ thống an ninh (security groups), cổng ra vào (gateways).
- **Yêu cầu**: 
  - Multi-AZ deployment cho high availability
  - Public và Private subnets với proper routing
  - NAT Gateway cho outbound internet access từ private subnets
  - Security Groups và NACLs cho defense in depth
- **CIDR Planning**:
  - VPC: 10.0.0.0/16 (65,536 IPs)
  - Public Subnets: 10.0.1.0/24, 10.0.2.0/24 (256 IPs each)
  - Private Subnets: 10.0.10.0/24, 10.0.20.0/24 (256 IPs each)
  - Database Subnets: 10.0.100.0/24, 10.0.200.0/24 (256 IPs each)
- **Kết quả mong đợi**: Secure, scalable network foundation

### **Bài tập 5**: Microservices với Container
- **Mô tả tổng quan**: "Xây dựng khu phố dịch vụ hiện đại". Tạo hệ thống các dịch vụ độc lập, mỗi service như một cửa hàng chuyên môn, có thể scale riêng biệt.
- **Services**: User (quản lý tài khoản), Product (catalog sản phẩm), Order (xử lý đơn hàng), Payment (thanh toán)
- **Container Platform**: ECS Fargate (serverless containers) hoặc EKS (Kubernetes)
- **Service Communication**: API Gateway làm cổng chính + ALB cho internal routing
- **Data Storage**: DynamoDB (NoSQL), RDS (relational), ElastiCache (caching)
- **Kết quả mong đợi**: Scalable, maintainable microservices architecture

---

## **TÀI LIỆU THAM KHẢO**
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Pricing Calculator](https://calculator.aws)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [AWS Documentation](https://docs.aws.amazon.com/)

---

## **ĐÁNH GIÁ**
- **Lý thuyết (40%)**: Multiple choice về services, pricing, best practices
- **Thực hành (40%)**: Thiết kế architecture diagram
- **Case study (20%)**: Phân tích và tối ưu existing architecture

---

*Giáo án được cập nhật theo AWS Documentation mới nhất (November 2024)*
