# GIÁO ÁN AWS ARCHITECTURE
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
- **Khái niệm**: Quản lý danh tính và quyền truy cập
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
- **Khái niệm**: Xác thực đa yếu tố
- **Loại**: Virtual MFA, Hardware MFA, SMS
- **Pricing**: Miễn phí cho Virtual MFA
- **Best Practice**: Bắt buộc cho Root user và privileged users

#### **SCP (Service Control Policies)**
- **Khái niệm**: Chính sách kiểm soát dịch vụ trong AWS Organizations
- **Vị trí**: Organization level (không thuộc VPC)
- **Pricing**: Miễn phí với AWS Organizations
- **Ví dụ**: Ngăn chặn tạo EC2 instances ở regions không được phép

#### **Encryption Services**

**AWS KMS (Key Management Service)**
- **Type**: Managed service
- **Vị trí**: Regional service (không thuộc VPC)
- **Pricing**: $1/key/month + $0.03/10,000 requests
- **Ví dụ**: Mã hóa EBS volumes, S3 objects

**TLS/ACM (Certificate Manager)**
- **Type**: Managed service
- **Pricing**: Miễn phí cho public certificates
- **Vị trí**: Regional (không thuộc VPC)
- **Sử dụng**: Load Balancers, CloudFront, API Gateway

#### **Network Security**

**Security Groups**
- **Khái niệm**: Stateful firewall cho EC2 instances
- **Vị trí**: VPC level
- **Pricing**: Miễn phí
- **Quy tắc**: Allow rules only, stateful

**NACLs (Network Access Control Lists)**
- **Khái niệm**: Stateless firewall cho subnets
- **Vị trí**: Subnet level trong VPC
- **Pricing**: Miễn phí
- **Quy tắc**: Allow và Deny rules, stateless

#### **Threat Detection & Protection**

**GuardDuty**
- **Type**: Managed threat detection
- **Pricing**: $4.00/million events analyzed
- **Vị trí**: Regional (không thuộc VPC)
- **Chức năng**: ML-based threat detection

**AWS Shield**
- **Standard**: Miễn phí, DDoS protection cơ bản
- **Advanced**: $3,000/month, DDoS protection nâng cao
- **Type**: Managed service

**AWS WAF (Web Application Firewall)**
- **Type**: Managed service
- **Pricing**: $1/web ACL/month + $0.60/million requests
- **Vị trí**: Edge locations (không thuộc VPC)
- **Sử dụng**: CloudFront, ALB, API Gateway

**Secrets Manager**
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
- **Khái niệm**: Triển khai trên nhiều Availability Zones
- **Pricing**: Không phí thêm cho architecture, chỉ trả cho resources
- **Ví dụ**: RDS Multi-AZ (2x cost), ELB tự động multi-AZ

#### **Multi-Region Deployment**
- **Khái niệm**: Triển khai trên nhiều AWS Regions
- **Pricing**: Data transfer charges giữa regions
- **Use case**: Disaster recovery, compliance, latency optimization

#### **DR (Disaster Recovery) Strategies**
1. **Backup & Restore**: RTO hours, RPO hours, cost thấp nhất
2. **Pilot Light**: RTO 10s minutes, RPO minutes, cost trung bình
3. **Warm Standby**: RTO minutes, RPO seconds, cost cao
4. **Multi-Site Active/Active**: RTO seconds, RPO near-zero, cost cao nhất

#### **Auto Scaling**
- **Type**: Managed service
- **Pricing**: Miễn phí (chỉ trả cho EC2 instances)
- **Vị trí**: Regional, hoạt động trong VPC
- **Scaling Policies**: Target tracking, Step scaling, Simple scaling

#### **Route 53**
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
- **Type**: Cost analysis tool
- **Pricing**: Miễn phí cho basic reports
- **Features**: Cost breakdown, forecasting, rightsizing recommendations

#### **AWS Budgets**
- **Pricing**: $0.02/budget/day (first 2 budgets free)
- **Features**: Cost budgets, usage budgets, alerts

#### **Savings Plans**
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
- **VPC**: Mạng ảo riêng biệt trong AWS Cloud
- **Isolated network**: Hoàn toàn tách biệt với các VPC khác
- **Regional service**: Trải rộng trên tất cả AZ trong region
- **CIDR Block**: Dải IP private (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)

#### **Core Components của VPC**

**Subnets**
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
- **NAT (Network Address Translation)**: Dịch địa chỉ IP private sang public
- **Mục đích**: Cho phép instances trong private subnet truy cập Internet
- **Hướng**: Chỉ outbound, không cho phép inbound connections
- **Managed Service**: AWS quản lý hoàn toàn

#### **Loại NAT Gateway**

**Public NAT Gateway**
- **Vị trí**: Phải đặt trong Public Subnet
- **Elastic IP**: Bắt buộc phải có
- **Routing**: Private Subnet → NAT Gateway → Internet Gateway → Internet
- **Use case**: Internet access cho private instances
- **Pricing**: $0.045/hour + $0.045/GB processed

**Private NAT Gateway**
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
- **EC2**: Elastic Compute Cloud - Virtual servers trong cloud
- **Instance**: Virtual machine với CPU, memory, storage, networking
- **AMI**: Amazon Machine Image - Template để launch instances
- **Instance Store**: Temporary storage gắn trực tiếp với physical host

#### **Instance Types và Use Cases**

**General Purpose (Balanced)**
- **t3/t4g**: Burstable performance, credit-based CPU
  - Use case: Web servers, small databases, development
  - Pricing: $0.0104/hour (t3.micro)
- **m5/m6i**: Balanced compute, memory, networking
  - Use case: Web applications, microservices
  - Pricing: $0.096/hour (m5.large)

**Compute Optimized**
- **c5/c6i**: High-performance processors
  - Use case: CPU-intensive applications, HPC, gaming
  - Pricing: $0.085/hour (c5.large)

**Memory Optimized**
- **r5/r6i**: High memory-to-vCPU ratio
  - Use case: In-memory databases, real-time analytics
  - Pricing: $0.126/hour (r5.large)
- **x1e**: Extreme memory (up to 3,904 GB)
  - Use case: SAP HANA, Apache Spark

**Storage Optimized**
- **i3**: NVMe SSD-backed instance storage
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
- **Durability**: 99.999999999% (11 9's)
- **Availability**: 99.99%
- **Pricing**: $0.023/GB/month
- **Use case**: Active data, websites, content distribution

**Standard-IA (Infrequent Access)**
- **Pricing**: $0.0125/GB/month + retrieval fee
- **Minimum**: 30 days storage, 128KB object size
- **Use case**: Backups, disaster recovery

**One Zone-IA**
- **Pricing**: $0.01/GB/month
- **Availability**: 99.5% (single AZ)
- **Use case**: Secondary backup copies

**Glacier Storage Classes**
- **Glacier Instant Retrieval**: $0.004/GB/month, milliseconds retrieval
- **Glacier Flexible Retrieval**: $0.0036/GB/month, minutes to hours
- **Glacier Deep Archive**: $0.00099/GB/month, 12 hours retrieval

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
- **Serverless**: Không cần quản lý servers
- **Event-driven**: Chạy code khi có events
- **Function**: Unit of deployment và scaling
- **Runtime**: Execution environment (Node.js, Python, Java, etc.)

#### **Core Features**

**Execution Model**
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
- **Layer**: 7 (Application layer)
- **Features**: Content-based routing, SSL termination, WebSocket
- **Targets**: EC2, IP addresses, Lambda functions
- **Pricing**: $0.0225/hour + $0.008/LCU-hour

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

#### **Network Load Balancer (NLB)**
- **Layer**: 4 (Transport layer)
- **Features**: Ultra-low latency, static IP, TCP/UDP
- **Performance**: Millions of requests per second
- **Pricing**: $0.0225/hour + $0.006/NLCU-hour

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
- **Simple**: Single resource
- **Weighted**: Distribute traffic by percentage
- **Latency-based**: Route to lowest latency
- **Failover**: Primary/secondary setup
- **Geolocation**: Route based on user location
- **Geoproximity**: Route based on geographic proximity
- **Multivalue**: Return multiple IP addresses

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
- **RTO**: Hours
- **RPO**: Hours
- **Cost**: Lowest
- **Implementation**: S3 backups, AMI snapshots

#### **Pilot Light**
- **RTO**: 10s of minutes
- **RPO**: Minutes
- **Cost**: Medium
- **Implementation**: Core services running in DR region

#### **Warm Standby**
- **RTO**: Minutes
- **RPO**: Seconds
- **Cost**: Higher
- **Implementation**: Scaled-down version running

#### **Multi-Site Active/Active**
- **RTO**: Seconds
- **RPO**: Near zero
- **Cost**: Highest
- **Implementation**: Full deployment in multiple regions

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
- **Yêu cầu**: 
  - Multi-AZ deployment
  - Public và Private subnets
  - NAT Gateway cho outbound internet access
  - Security Groups và NACLs
- **CIDR Planning**:
  - VPC: 10.0.0.0/16
  - Public Subnets: 10.0.1.0/24, 10.0.2.0/24
  - Private Subnets: 10.0.10.0/24, 10.0.20.0/24
  - Database Subnets: 10.0.100.0/24, 10.0.200.0/24

### **Bài tập 5**: Microservices với Container
- **Services**: User, Product, Order, Payment
- **Container Platform**: ECS Fargate hoặc EKS
- **Service Communication**: API Gateway + ALB
- **Data Storage**: DynamoDB + RDS + ElastiCache

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
